+++
author = "Jeff Chang"
title= "Flatten Object with Javascript" 
date= "2021-07-04"
description= "This article, we will be going through how to flatten an object with recursively methods by Javascript" 
tags = [
    "javascript","algorithm & data-structure"
]
categories = [
    "Algorithm","Javascript"
]
image="cover.jpg"
+++
Below will be the object to be used and tested in both flatten and unflatten algorithm.
{{< highlight js >}}
const object = {
    user: {
        name: "Jeff",
        age: 24,
        hobbies: ["Piano", "Software Development", "Foods"],
        address: {
            country: "Malaysia",
            state: "Kuala Lumpur"
        },
    },
}
{{< /highlight >}}

## Flatten Object
In this example, we are flattening the object by using recursive method
{{< highlight js  "linenos=table,linenostart=1">}}
function flattenObj(obj, parentKey = null, res = {}) {
    for (let key in obj) {
        const propName = parentKey ? parentKey + "." + key : key
        if (typeof (obj[key]) === "object" && !Array.isArray(obj[key])) {
            flattenObj(obj[key], propName, res)
        } else {
            res[propName] = obj[key]
        }
    }
    return res
}
{{< /highlight >}}

### Result
{{< highlight js >}}
{
  'user.name': 'Jeff',
  'user.age': 24,
  'user.hobbies': [ 'Piano', 'Software Development', 'Foods' ],
  'user.address.country': 'Malaysia',
  'user.address.state': 'Kuala Lumpur'
}
{{< /highlight >}}

### Explanation
1. Create a function which takes in 3 arguments. (1 mandatory and 2 optional)
2. Iterate the object by using [for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) which allow us to acess all keys from the object
3. As we are now trying to flatten the object keys, so it needed to combine with it's previous key. If there is no previous key  *"only 1 layer / not nested object "* we will then use the current key as the flatten object key
4. Check if the current value is an object by using the methods `typeof()`, this case we added another condition `!Array.isArray()` is because `typeof(Array)` will still returning true but we only wanted to check object type.
    - If the value is an object type, we can then called the function recursively `flattenObj(obj[key], propName, res)` by assigning the argument of 
        * curreny value object `obj[key]` 
        * combined key name with previous key 
        * response object
    - else it will be now at the endpoint / last key of the nested object. And we can assign the actual value to it `res[propName] = obj[key]`