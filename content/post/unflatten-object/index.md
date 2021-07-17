+++
author = "Jeff Chang"
title= "Unflatten Object with Javascript" 
date= "2021-07-15"
description= "Last article, we have discussed about how to flatten an object in any level. In this article, we will be going through how to unflatten an object" 
tags = [
    "javascript","algorithm & data-structure"
]
categories = [
    "Algorithm","Javascript"
]
image="cover.jpg"
+++
Let's recap the flatted object we have created in the [last article](/p/flatten-object-with-javascript)
{{< highlight js >}}
const flattedObject ={
  'user.name': 'Jeff',
  'user.age': 24,
  'user.hobbies': [ 'Piano', 'Software Development', 'Foods' ],
  'user.address.country': 'Malaysia',
  'user.address.state': 'Kuala Lumpur'
}
{{< /highlight >}}

## Unflatten Object
In this example, we will be utilizing the [`Array.prototype.reduce`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) methods to unflatten the object
{{< highlight js  "linenos=table,linenostart=1">}}
function unflattenObject(object) {
    let res = {}
    for (key in object) {
        const keys = key.split(".")
        keys.reduce((acc, value, index) => {
            return acc[value] || (acc[value] = keys.length - 1 === index ? object[key] : {})
        }, res)
    }
    return res
}
{{< /highlight >}}

### Result
{{< highlight js >}}
{
  user: {
    name: 'Jeff',
    age: 24,
    hobbies: [ 'Piano', 'Software Development', 'Foods' ],
    address: { 
        country: 'Malaysia',
        state: 'Kuala Lumpur' }
  }
}
{{< /highlight >}}

### Explanation
1. Firstly, we created an object for us to return later. 
2. Iterate the object by using [for...in](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...in) which allow us to acess all keys from the object
3. However, the key of the object might flatted with nested object by full stop **.**. For instance,  `'user.name'` should be equavalent to user:{name :{ `<value>` }}
4. As we can see from the `flattedObject` variable. The object keys are combined with the full stop **.**. So we can separate them by using [split method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split).
5. As [split method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split) will return us an array. Now, we could iterate this array with [`Array.prototype.reduce`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce) method
6. We first assign the object variable => `res` as it initial value. Then we can start to iterate the object keys and create the following conditions 
    - assign the value to current iterated object key if current index is equal to the last index of the object keys
    - Otherwise, create an empty object and continue iterate for the remaining keys
