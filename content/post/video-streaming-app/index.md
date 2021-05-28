+++
author = "Jeff Chang"
title= "Video Streaming" 
date= "2021-05-28"
description= "Last article, we have gone through how to how to read and write the file into a stream. In this article, we are going to build a simple video streaming app." 
tags = [
    "nodejs", "javascript"
]
categories = [
	"NodeJs", "Javascript"
]
+++
If you not sure how stream and buffer works. Make sure you checkout my [previous article](/p/node-stream-and-buffer) which talks about various methods from stream. <br/>
Stream a video can help us speed up the waiting process which will naturally improves the user experience as well. However, it might having some downsides if we didn't manage it properly. <br/>

First of all, let's start a simple express server in port 3000 which will serve the index.html file.<br/>

*index.html*
{{< highlight html >}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Video Streaming App</title>
    <style>
        body{
            margin: 40px auto;
            max-width: 650px;
            background-color: azure;
            font-family: sans-serif;
        }
    </style>
</head>
<body>
    <h1>Video Streaming App</h1>
    <video id='videoPlayer' width="650" controls muted="muted" autoplay>
        <source src="/video" type="video/mp4"/>
    </video>
</body>
</html>
{{< /highlight >}}

## Result
![video demo](video-demo.jpg)

This article uses a Big Buck Banny video for demo. You may get more public test video from [https://gist.github.com/jsturgis/3b19447b304616f18657](https://gist.github.com/jsturgis/3b19447b304616f18657)

*server.js*
{{< highlight js >}}
const express = require('express');
const app = express();
const fs = require('fs');

//Serve static file
app.get("/", (req, res) => {
    res.sendFile(__dirname + "/index.html")
})

app.get("/video", (req, res) => {
    // Need range header for telling client which part of the video we want to send back
    const range = req.headers.range;
    if (!range) {
        res.status(400).send("Require Range Header");
        return
    }

    const videoPath = 'bigbuck.mp4';
    const videoSize = fs.statSync(videoPath).size;

    const CHUNK_SIZE = 10 ** 6 // 10 power of 6 is 1MB
    const start = Number(range.replace(/\D/g, ""));// replace all non-digit characters to empty string and parse into Number

    // calculate the ending byte we want to response back to client. If it reached the end of file (total video size)
    //Then send back the video size instead
    const end = Math.min(videoSize - 1, start + CHUNK_SIZE);
    const contentLength = end - start + 1;
    const header = {
        //https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Range
        "Content-Range": `bytes ${start}-${end}/${videoSize}`,

        //https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Ranges
        //What type we send back
        "Accept-Range": "bytes",

        "Content-Length": contentLength, // How many byte we are send back
        "Content-Type": "video/mp4",
    };

    //Status 206 = sending partial content. Means the response is not yet finished
    res.writeHead(206, header)
    const readStream = fs.createReadStream(videoPath, { start, end });
    readStream.pipe(res)
})

const PORT = process.env.PORT || 3000
app.listen(PORT, () => { console.log(`Server is Running at PORT ${PORT}`) })
{{< /highlight >}}

### Result
Kindly open the url http://localhost:3000 and open Developer tool(F12) -> Network Tab -> Filter by Media


<video controls autoplay style="max-width:100%">
    <source src="video-stream.mp4" type="video/mp4">
    <source src="video-stream.ogg" type="video/ogg">
    </video>
</div>