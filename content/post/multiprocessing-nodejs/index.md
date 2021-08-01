+++
author = "Jeff Chang"
title = "Scaling Node.js Application with Multiprocessing"
date = "2021-08-01"
description = "Javascript is a single threaded language. This means it has one call stack and one memory heap and it will executes code in order and must finish executing a piece code before moving onto the next. It will become a problem when the application slowly scaled up. As a result, we could use the child_process to provide multiprocessing on handling CPU-Intensive Tasks.  "
tags = [
    "nodejs", "javascript"
]
categories = [
	"NodeJs", "Javascript"
]
image = "cover.jpg"
+++
## Problem
As we mentioned above Javascript is single threaded and it will executes the code in sequence order. Here will be the problem when user trying to access two HTTP requests which one of it is handling CPU intensive task
<div style="max-width:50%; margin:0 auto">
    <img src="slow.gif" alt="intensive CPU task problem" style="width:100%"/>
</div>

{{< highlight js >}}
const express = require('express')
const app = express()

app.get("/normal", async (req, res) => {
    res.send(`normal request`)
})

app.get("/slow", (req, res) => {
    const startTime = new Date()
    const result = mySlowFunction(req.query.number)
    const endTime = new Date()
    res.json({
        result,
        time: endTime.getTime() - startTime.getTime() + "ms",
    })
})

function mySlowFunction(baseNumber) {
    let result = 0;
    for (let i = Math.pow(baseNumber, 5); i >= 0; i--) {
        result += Math.atan(i) * Math.tan(i);
    };
    return result
}

app.listen(process.env.PORT || 5000, () => console.log('server is running at port 5000'))

{{< /highlight >}}

As we can see, when Node.js server is handling intensive CPU task, other non-intensive HTTP request will get affected as well. To resolve this, we can use `child_process` module to handling it by multiprocessing

## What is child_process
The child_process module provides the ability to spawn new processes which has their own memory. The communication between these processes is established through IPC (inter-process communication) provided by the operating system.

There are total of 4 main methods under this module which are:
1. [child_process.exec()](https://nodejs.org/api/child_process.html#child_process_child_process_exec_command_options_callback)
2. [child_process.execFile()](https://nodejs.org/api/child_process.html#child_process_child_process_execfile_file_args_options_callback)
3. [child_process.fork()](https://nodejs.org/api/child_process.html#child_process_child_process_fork_modulepath_args_options)
4. [child_process.spawn()](https://nodejs.org/api/child_process.html#child_process_child_process_spawn_command_args_options)

## child_process.fork()
In this article, we will be only focus on the fork method to handle the intensive CPU task. <br/>

fork method takes in 3 arguments: `fork(<path to module>,<array of arguments>, <optionsObject>)`<br/>
The **child_process.fork()** method able to spawn new Node.js processes. It allow an additional communication channel built-in that allows messages to be passed back and forth between the parent and child by using [process.send](https://nodejs.org/api/process.html#process_process_send_message_sendhandle_options_callback) and process.on. So the parent process won't be blocked and can continue responding to requests.

**main.js**
{{< highlight js >}}
const express = require('express')
const app = express()
const { fork } = require('child_process')

app.get("/normal", async (req, res) => {
    res.send(`normal request`)
})

app.get("/forkProcess", (req, res) => {
    const startTime = new Date()
    const child = fork('./longComputation') //access the module path in fork method
    child.send({ number: parseInt(req.query.number) }) //pass custom variable into the particular file

    //Execute the following code when the child response has responded back 
    child.on('message', (result) => {
        const endTime = new Date()
        res.json({
            result,
            time: endTime.getTime() - startTime.getTime() + "ms",
        })
    })
})

app.listen(process.env.PORT || 5000, () => console.log('server is running at port 5000'))
{{< /highlight >}}


**longComputation.js**
{{< highlight js >}}
function mySlowFunction(baseNumber) {
    let result = 0;
    for (let i = Math.pow(baseNumber, 5); i >= 0; i--) {
        result += Math.atan(i) * Math.tan(i);
    };
    return result
}

process.on('message', message => {
    const result = mySlowFunction(message.number)
    process.send(result)
    process.exit() //terminate process after it's done
})
{{< /highlight >}}

## Result
<div style="max-width:50%; margin:0 auto">
    <img src="fast.gif" alt="resolve intensive CPU task problem" style="width:100%;"/>
</div>

### Explanation
* main.js
    1. Import the `fork` method from child process
    2. Passing the file path into the `fork` method argument
    3. Send custom variable into separated nodejs process (forked) by using process.send
    4. Respond back to the HTTP request after receiving the child process response by process.on

* longComputation.js
    1. Relocate the high intensive CPU task code into separate file
    2. Execute the high intensive code when receiving the signal/ message from parent process by using `process.on` and return once the task is finised by using `process.send()`


