+++
author = "Jeff Chang"
title= "Closure with Javascript" 
date= "2021-12-05"
description= "Have you ever wonder how you can actually access the variable from a parent function which is stored in memory and declare in somewhere before you execute the current code? Closure is one of the best way to handle this situation" 
tags = [
    "javascript"
]
categories = [
    "Javascript"
]
image = "cover.jpg"
+++
First and foremost, when we talked about closure, most likely it means a function is returning with another function. <br/>
We have used a `greater than validation` as our closure in example below:
{{< highlight js >}}
function greaterThanValidation(minNumber){
    return function (x, y){
        if(typeof(x) !== 'number' || typeof(y) !== 'number')
            throw new Error("Invalid Argument Type")

        if((x+y) > minNumber)
            return true
        else
            return false
    }
}

try {
    const validation = greaterThanValidation(10)
    const result = validation(2,10)
    console.log("result >> ", result)
} catch (error) {
    console.error(error.message)
}
{{< /highlight >}}

### Explanation
1. As you can see, we have declared a outer function called `greaterThanValidation` which takes in 1 argument and returning a new function wich accepts 2 argument.
2. Inside the inner function, we will first check the type of the input and throw an `Error` if either one is not `number` type
3. Once it pass the inner function condition, it will execute the functionality of the function which is:
    - return `true` if sum of x and y **is** greater than the minumum number <small><em>which is pass from the outer function</em></small>
    - return `false` if sum of x and y **is not** greater than the minumum number.
4. For execute the closure, it's pretty straight forward:
    - First, we need to declare the parameter or value to be passed in the outer closure function
    - Since the outer function is returning a new function, we can then execute the code from inner function by calling the `validation` <small><em>outer function</em></small>


### Why is it useful
Based on the example above, we might not realized how useful the closure is. However, one of the thing I like closure the most is, it actually store the **outer function** parameter value in the memory and be ready for any moment you want to access it in the **inner function** in anytime. Especially when the **outer function** parameter value is actually get from an **API** or not **hardcoded / declared inside the outer function**. <br/>

In short, closure most of the time is saying a function is returning another function. Once we declared and passed in the value for **outer function**, it will actually store in the memory and the **inner function** will able to access all of the variable from the **outer function**

#### Beautify the code with ES6 Arrow function 
We also beautify the code by converting them into ES6 Arrow function manner
{{< highlight js >}}
const anotherGreaterThanValidation = minNumber => (x,y) => {
    if(typeof(x) !== 'number' || typeof(y) !== 'number')
    throw new Error("Invalid Argument Type")

    if((x+y) > minNumber)
        return true
    else
        return false
}
{{< /highlight >}}
