+++
author = "Jeff Chang"
title = "MongoDB join collection"
date = "2021-10-22"
description = "Have you ever wonder how do we customize our own response field based on the MongoDB Query? Mongo Aggregation Pipeline allows us easily to achieve that Goal"
tags = [
    "nodejs", "mongodb"
]
categories = [
	"NodeJs", "MongoDB"
]
+++

<a href="/p/mongodb-join-collection/response.json" target="_blank">[response.json]</a>
<a href="/p/mongodb-join-collection/filtered_response.json" target="_blank">[filtered_response.json]</a>


## Code

{{< highlight js "linenos=table,linenostart=1">}}
{ 
"_id" : ObjectId("6173e5fe7fc8622cbc56a108"),
"name" : "Jeff",
"age" : 20,
"ref_id" : "457935c0-33ed-11ec-aac1-dd97bcc28a8a",
"**v" : 0
},
{
"_id" : ObjectId("6173e6067fc8622cbc56a10a"),
"name" : "Jason",
"age" : 25,
"ref_id" : "4a4bcc70-33ed-11ec-aac1-dd97bcc28a8a",
"**v" : 0
}
{{< /highlight >}}
