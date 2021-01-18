+++
author = "Jeff Chang"
title = "Upload File with Go"
date = "2021-01-17"
description = "Upload file to server is a very popular use case in any application. However, it somehow can be very complicated or difficult to understand how it works. In this article, we will be studying how to upload any type of file with Golang and Javascript."
tags = [
    "javascript", "go"
]
categories = [
	"Javascript", "GO"
]
image = "cover.jpg"
+++
Let's get started with the code and it's explanation, [HTML](#html), [Javascript](#js), [Go](#go)
## HTML code <a name="html"></a>
{{< highlight html >}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Upload File</title>
    <link rel="stylesheet" href="/static/style.css">
</head>
<body>
    <h1>Upload File</h1>
    <div class="upload-file-modal" style="margin-top: 50px;">
            <input type="file" id="inputFileToLoad">
            <button onclick="doUpload(document.getElementById('inputFileToLoad'))">Upload</button>
    </div>
</body>
<script src="/static/script.js"></script>
</html>
{{< /highlight >}}

#### Explanation
Pretty straight forward here, one **file** input for uploading the file and a button to pass the file input DOM into the function **"doUpload"**

## Javascript code <a name="js"></a>
{{< highlight js >}}
function doUpload(fileElements) {
    let formData = new FormData();
    let file = fileElements.files; //return a JSON array aka FileList which contains of information like filename, size, type and etc
    formData.append("file", file[0]);

    var xhr = new XMLHttpRequest();

    xhr.onload = function () {
        console.log("Message from backend", xhr.response);
    };

    xhr.open("POST", "http://localhost:3000/uploadFile", true);
    xhr.send(formData);
}
{{< /highlight >}}

#### Explanation
1. Create a FormData object so that we can append our file later.
2. Get the *files* attribute from the file input DOM.
3. Since now we are demonstrating to upload **single file**, so we only append the first element of the FileList json array from the file input DOM.
4. Make a **POST** request by using [XMLHttpRequest](/p/xmlhttprequest-vs-fetch-api/)
    * The reason I used XHR request instead of Fetch API is because we will be studying how to track the progress of upload in future article.

## Golang code <a name="go"></a>
{{< highlight go >}}
func UploadFile(w http.ResponseWriter, r *http.Request) {

	//Path for storing image in server
	path := "C://Users/User/Desktop/Upload/"

	mr, err := r.MultipartReader()
	if err != nil {
		fmt.Println("err", err)
		return
	}
	fileName := ""

	for {
		part, err := mr.NextPart()

		//Break, if no more file
		if err == io.EOF {
			break
		}
        //Get File Name attribute
		fileName = part.FileName()

		//Open the file, Create file if it's not existing
		dst, err := os.OpenFile(path+fileName, os.O_WRONLY|os.O_CREATE, 0644)
		if err != nil {
			return
		}

		for {
			//Create and read the file into temparory buffer
			buffer := make([]byte, 100000)
			cBytes, err := part.Read(buffer)

			//break when the reading process is finished
			if err == io.EOF {
				break
			}

			//Write into the file from buffer
			dst.Write(buffer[0:cBytes])
		}
		defer dst.Close()
	}

	w.Header().Set("Content-Type", "application/json; charset=UTF-8")
	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode("done")
}
{{< /highlight >}}

#### Explanation
1. Declare the final destination of the uploaded file in our server. This case will be *"C://Users/User/Desktop/Upload/"*
2. Get the file info from **Request.MultipartReader()**
3. Read the part of the file inside an infinity loop. Break if it has reached End Of File (EOF).
4. We first create and open the file as a empty file according to the path we declared in **point 1**.
    * *Meaning to say it just having the same filename without any data/ information from the actual file*
5. Then we create a temporary buffer and keep reading the file data in a infinity for loop.
6. Meanwhile, we then write the data from temporary buffer into the file we just created and openned from **point 4**
7. Break the for loop once it has successfully read and write the file in our server.
