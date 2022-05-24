+++
author = "Jeff Chang"
title = "Build Your Backend Server with GOLANG"
date = "2020-08-21"
description = "In this article, we will be creating our first restful server with Go and the package called Gorilla Mux"
tags = [
    "go",
]
categories = [
    "GO",
]
metakeywords = "Golang, Server"
codeMaxLines = 1
+++

Firstly, we need to import the [Gorilla Mux package](https://github.com/gorilla/mux) into our web application and initialize our server router.

<!-- prettier-ignore -->
{{< highlight go>}}
package main

import (
    "github.com/gorilla/mux"
    "text/template"
)

var templates *template.Template

func main() {
    templates,_ = template.ParseFiles("template/index.html")

    router := mux.NewRouter()
    router.HandleFunc("/getProductDetails", GetProductDetails).Methods("POST")
    router.HandleFunc("/",HomePage).Methods("GET")

    fmt.Println("Server is at Port 8080")
    http.ListenAndServe(":8080", router)

}

func GetProductDetails(w http.ResponseWriter, r \*http.Request){
    fmt.Println("Get product details")
}

func HomePage(w http.ResponseWriter, r \*http.Request){
    templates.ExecuteTemplate(w, "index.html", nil)
}

{{< /highlight >}}

Mux router will execute the function respectively whenever the request is matched with the end point. Example below showing a simple **POST** request with gorilla mux, when the request path is matched with the end point **(“/getProductDetails”)** The **GetProductDetails** function will be executed. We will discuss more on how we can response back to the front end and other Methods such as _“GET, OPTIONS, PUT, DELETE”_.

Before we start to serve our serve, we need to create a homepage for it. We can use template from golang to serve a HTML template so that when we visit our server, this particular HTML file will be served and displayed. We can create a folder which will hold all template file.

![index.html](golang_server_05.png)

Next, we will import the template library and create a variable for storing the html template.

We can now parse the html template file into our variable and serve as our website home page. As we only serve one html file for our template, So we can use template.ParseFiles. In case there are many template file to serve, we can use template.ParseGlob to point to the file directiory. The end point **“/”** means our home page. For example, we set our local host server in port **8080**. Then when we visit that page, the **index.html** will be served.

Finally, we can start and serve our server.

![](golang_server_11.png)

{{% endLine %}}
