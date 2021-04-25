+++
author = "Jeff Chang"
title= "Speed UP Your Application with Redis Cache" 
date= "2021-04-24"
description= "Speed is everything! Imagine if an user needs to wait around 2 seconds for the desired information. He/ She or at least me myself would already close the application and try to search for another website." 
tags = [
    "redis","nodejs","javascript"
]
categories = [
	"NodeJs", "Javascript","Redis"
]
image = "cover.jpg"
+++
Source code is available at >> [https://github.com/Jeffcw96/node-redis](https://github.com/Jeffcw96/node-redis) <br/>
In this article, let's explore a little bit of how we can improve our application response time by using redis-cache approach.
* [Import necessary packages and setup server](#SETUP)
* [Call API from GITHUB job listing](#GITHUB)
* [Implement Redis-Cache approach in middleware](#MIDDLEWARE)

### Prerequisites:
* [Install Redis server in your machine](/p/setup-redis).
* [NodeJs](/p/setup-node-server-with-express).
* Packages needed to be install in this article: [Express](https://www.npmjs.com/package/express), [Axios](https://www.npmjs.com/package/axios), [Redis](https://www.npmjs.com/package/redis).

## Import necessary packages and setup server<a name="SETUP"></a>
First of all, let's import all necessary packages and start out server at PORT 5000

{{< highlight js >}}
const express = require('express');
const axios = require('axios');
const cors = require('cors');
const redis = require('redis');
const app = express()
const util = require('util');

app.use(express.json({ extended: false }));
app.use(express.urlencoded({ extended: false }));

app.listen(process.env.PORT || 5000, () => {
    console.log("server is running at 5000")
})
{{< /highlight >}}

Now, let's initialize our redis client in port 6379 (default) <br/>
Let's bind the redis-client command into promise to implement asynchronously via nodejs built-in method [util.promisify](https://nodejs.org/dist/latest-v8.x/docs/api/util.html#util_util_promisify_original) instead of infinite callback 
{{< highlight js >}}
const client = redis.createClient({
    host: 'localhost',
    port: 6379,
});

client.on('error', err => {
    console.log('Error ' + err);
});

client.on('connect', () => {
    console.log('Redis is connected');
});

//Recommend to use
const clientLRANGE = util.promisify(client.LRANGE).bind(client);
const clientTTL = util.promisify(client.TTL).bind(client);
const clientRPUSH = util.promisify(client.RPUSH).bind(client);
const clientEXPIRE = util.promisify(client.EXPIRE).bind(client);


//Not recommend to use
client.LRANGE(< key >, (err, result)=>{ // do something here })
client.TTL(< key >, (err, result)=>{ // do something here })
client.RPUSH(< key >, (err, result)=>{ // do something here })
client.EXPIRE(< key >, (err, result)=>{ // do something here })

{{< /highlight >}}

## Call API from GITHUB job listing<a name="GITHUB"></a>
Github offer an API where it will return the current job listing in their page. <br/>
**Link >>**[https://jobs.github.com/positions.json?description=api](https://jobs.github.com/positions.json?description=api)
<br/>

Now, let's create an API which will grab all the information from this Github API via axios.
{{< highlight js >}}
app.get("/getGithubJobListing", middleware, async (req, res) => {
    try {
        const url = `https://jobs.github.com/positions.json?description=api`
        const response = await axios.get(url)

        await clientRPUSH("github:jobList", JSON.stringify(response.data)) //JSON stringify the data and store inside redis list data type
        await clientEXPIRE("github:jobList", 120) //Set expiry time to 120 seconds

        res.json({ data: response.data })

    } catch (error) {
        console.error(error.message)
        res.status(500).send('Server Error')
    }
})
{{< /highlight >}}

### Result
<img src="/p/speed-up-your-application-with-redis-cache/original_response_time.jpg">

### Explanation
1. Create an API and make an HTTP Request to [https://jobs.github.com/positions.json?description=api](https://jobs.github.com/positions.json?description=api)
2. JSON.stringify the response data and store them under [Redis list](/p/redis-list) with [RPUSH](https://redis.io/commands/rpush) command.
3. Set the expiry time to 120 seconds for the particular redis key via [EXPIRE](https://redis.io/commands/expire) command.
4. As we can observe the response time from the image above, it's around 2.7seconds which is really slow.

## Implement Redis-Cache approach in middleware<a name="MIDDLEWARE"></a>
Let's implement a middleware where we will directly get the data from the redis `<github:jobList>` if it hasn't expire yet.
{{< highlight js >}}
const middleware = async (req, res, next) => {
    try {
        const redisData = await clientLRANGE("github:jobList", 0, -1);

        if (redisData.length === 0) {
            next()
            return
        }

        const data = JSONParse(redisData);
        res.json({ data })

    } catch (error) {
        console.error(error.message)
        res.status(500).send('Server Error')
    }
}
{{< /highlight >}}
### Result
<img src="/p/speed-up-your-application-with-redis-cache/redis-cache.jpg">

### Explanation
1. Try to get the data stored inside the key where we just placed the github job listing api data previously `github:jobList`
2. If we managed to get the data which is *Not Yet Expire*. Response back with this data. 
3. Otherwise, use [next()](https://expressjs.com/en/guide/writing-middleware.html) keyword to continue execute the command like what we did in [last section](#GITHUB)

## Extra
We can use redis [TTL](https://redis.io/commands/TTL) command to check how much time left for the expiry redis key. <br/>
The result will be **-2** once it has completely expired
{{< highlight js >}}
function checkExpiryTime(key) {
    try {
        const interval = setInterval(async () => {
            let timer = await clientTTL(key)
            console.log("timer ", timer)

            if (timer === -2) {
                clearInterval(interval)
            }
        }, 1000);
    } catch (error) {
        console.error(error.message)
    }
}
{{< /highlight >}}



