+++
author = "Jeff Chang"
title = "Concurrency & Parallelism with Go"
date = "2021-04-08"
description = "Have you ever wonder how to speed up your program especially when you trying to receive a quick reponse from HTTP Request as soon as possible while there are bunch of code need to be executed in the API. In this article, we will be going through how to increase our performance via concurrency and parallelism"
tags = [
    "go"
]
categories = [
	"GO"
]
image = "cover.jpg"
+++
Before we get started. Please make sure you have imported the necessary modules as listed below
{{< highlight go >}}
package main

import (
	"encoding/json"
	"fmt"
	"log"
	"net/http"
	"sync"
	"time"

	"github.com/gorilla/mux"
)
{{< /highlight >}}

In this article, I will be create 2 API which one of them will be using concurrency method while the other will be using parallelism approach. Below is showing the main function with end points and API function.
{{< highlight go >}}
func main() {
	r := mux.NewRouter()
	r.HandleFunc("/concurrency", ConcurrencyAPI).Methods("GET")
	r.HandleFunc("/parallelism", ParallelismAPI).Methods("GET")

	log.Fatal(http.ListenAndServe(":8080", r))
}
{{< /highlight >}}

* [CONCURRENCY](#CONCURRENCY)
* [PARALLELISM](#PARALLELISM)

## CONCURRENCY<a name="CONCURRENCY"></a>
Before we started to look at how powerful concurrency is. Let's run a simple program that will print out the index until 9999 of the for loop
{{< highlight go >}}
func ConcurrencyAPI(w http.ResponseWriter, r *http.Request) {
	ConcurrencyFunc()

	status := Status{RespStatus: "OK"}
	fmt.Println("---------------------------------------------")
	fmt.Println("DONE")
	fmt.Println("---------------------------------------------")

	w.Header().Set("Content-type", "application/json; charset=UTF-8")
	w.WriteHeader(http.StatusOK)
	json.NewEncoder(w).Encode(status)
}

func ConcurrencyFunc() {
	for i := 0; i <= 9999; i++ {
		fmt.Println("current i", i)
	}
}
{{< /highlight >}}

### Result
<div style="max-width:60%; margin:0 auto">
<img src="normal_program.gif" alt="normal program print in for loop" style="width:100%">
</div>

##### Response Time
![Response Time](concurrency_1.jpg)

### Explanation
As it's just like a code nature where the code will be executed each line by line. However, it will be very bad if we need to take around 13 seconds to complete a HTTP Request. <br/>
This is where [concurrency with goroutine](https://www.geeksforgeeks.org/goroutines-concurrency-in-golang/#:~:text=A%20Goroutine%20is%20a%20function,like%20a%20light%20weighted%20thread.) come into play. We can let the **wrap the blocks of code into goroutine** so that it will be executed asynchronously without blocking or slowing down our response time.

### Concurrency with Goroutine
Kindly add a keyword **"go"** infront of the function needed to be execute asynchronously as shown in below
{{< highlight go >}}
	go ConcurrencyFunc()
{{< /highlight >}}

### Result
<div style="max-width:60%; margin:0 auto">
<img src="concurrency.gif" alt="concurrency with goroutine" style="width:100%">
</div>

##### Response Time
![Response Time](concurrency_2.jpg)

