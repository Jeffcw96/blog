+++
author = "Jeff Chang"
title = "ES6 Destructuring Array and Object"
date = "2020-10-24"
description = "Destructuring assignment syntax is a JavaScript expression that makes it possible to unpack values from arrays, or properties from objects, into distinct variables."
tags = [
    "javascript"
]
categories = [
  "Javascript","Algorithm","Front-end"
]
metakeywords = "destructure array, destructure object, ES6, Javascript, Spread Operator"
image = "cover.jpg"
+++

## Array Destructuring
{{< highlight js>}}
const alphabet = ['A', 'B', 'C', 'D', 'E'];
const number = [1, 2, 3, 4, 5];
{{< /highlight >}}

Let's say we want to get the first and second element of the array. The very normal way is by doing this.
{{< highlight js>}}
let first = alphabet[0]; //return 'A'
let second = alphabet[1]; //return 'B'
{{< /highlight >}}

However, we're trying to get more values from the array, the code will be long and difficult to read. 
Let's see how we can handle this by better way. 
{{< highlight js>}}
let [first,second] = alphabet;
console.log(first); //return 'A'
console.log(second); //return 'B'
{{< /highlight >}}

### Skip certain value 
If we want to avoid from getting some value or skip certain indexes from the array. It can be easily done by putting empty value there. For example, now we wanted to get the first index value **'A'** and the fourth index value **'D'**
{{< highlight js>}}
let [first, , ,fourth] = alphabet
console.log(first); //return 'A'
console.log(fourth); //return 'D'
{{< /highlight >}}

### Spread Operator ...
If now we want to get the all value from array **except** the second index **'B'**. This is where **Spread Operator** come into play.
index value **'D'**
{{< highlight js>}}
let [first, ,...rest] = alphabet
console.log(first); //return 'A'
console.log(rest); //return ['C','D','E']
{{< /highlight >}}

### Combine Array
Normal way to combine 2 array is by using the method call "concat"
{{< highlight js>}}
let combineArray = number.concat(alphabet);
console.log(combineArray); //return [1,2,3,4,5,'A','B','C','D','E']
{{< /highlight >}}


And this is how it can be done by using spread operator
{{< highlight js>}}
let combineArray = [...number,...alphabet];
console.log(combineArray); //return [1,2,3,4,5,'A','B','C','D','E']
{{< /highlight >}}

### Working spread operator with function
Let's say we have one function which takes 2 parameters and return an array with diffrent math operation. <br/>
Normal way usually will be like this. 
{{< highlight js>}}
function mathOperation(x,y){
     return [x+y,x-y];
}

let mathArray = mathOperation(10,3);
let sum = mathArray[0];
let subtract = mathArray[1];

console.log(sum); //return 13
console.log(substract); //return 7
{{< /highlight >}}

With array destructuring, we can shorten down the code
{{< highlight js>}}
let [sum,subtract] = mathOperation(10,3);

console.log(sum); //return 13
console.log(substract); //return 7
{{< /highlight >}}

we also can set the default value that will be used when there is no parameters to pass in
{{< highlight js>}}
let [sum,subtract, multiply = "No multiplication"] = mathOperation(10,3);
console.log(multiply); //return 'No Multiplication'

//Edit function
function mathOperation(x,y){
     return [x+y,x-y,x*y];
}

console.log(multiply); //return 30
{{< /highlight >}}

## Object Destructuring
{{< highlight js>}}
const CompanyOne = {
    name: 'Google',
    age: 22,
    headquarter: {
        city: 'Mountain View',
        state: "California"
    }
}

const CompanyTwo = {
    name: 'Amazon',
    age: 26,
    headquarter: {
        city: 'Seattle',
        state: "Washington"
    }
    founder: 'Jeff Bezos'
}
{{< /highlight >}}

Destructuring the object is almost similar with the array destructring. <br/>
Let's say now we want to get the properties **name** and **age** from the object.
{{< highlight js>}}
let {name, age} = CompanyOne;
console.log(name); //return 'Google'
console.log(age); //return 22
{{< /highlight >}}
<small style="display:block"><em>Note: the variable inside the curly braces need to be <strong>exactly matched</strong> with the properties name.</em></small>


### Customize variable name from getting object value
By adding the colon **:** symbol we can then customize our variable name.
{{< highlight js>}}
// let { name: companyName, age: companyAge } = CompanyOne;
console.log(companyName); //return 'Google'
console.log(companyAge); //return 22
{{< /highlight >}}

we also can set the default value that will be used when there is no parameters to pass in
{{< highlight js>}}
let { name: companyName = "Others", age: companyAge } = CompanyOne;
console.log(companyName) //return 'Others' when Company One object doesn't have the property of name
{{< /highlight >}}

### Spread Operator in object
For example we want the first property into one variable and the rest will be another variable. <br/>
It's works quite similar like array destructuring with spread operator <br/>
The variable **otherInfo** is an object data type and holding the rest of the properties value except **name**
{{< highlight js>}}
let { name: companyName, ...otherInfo } = CompanyOne;
console.log(companyName); //return 'Google'
console.log(otherInfo); 
/* { age: 22,
     headquarter: {
        city: 'Mountain View',
        state: "California"
    }}*/  
{{< /highlight >}}

### Desturcutre the object within an object
{{< highlight js>}}
let { name: companyName, headquarter: { city, state } } = CompanyOne;
console.log(city) //return 'Mountain View'
console.log(state) // return 'California'
{{< /highlight >}}

### Concatenate and Overwrite object
By using spread operator with 2 object together, it will be first check if they have the same properties field. If **Yes** the value from second object will overwrite the first object. If **No** it will then create that field which the value respectively.
{{< highlight js>}}
var otherCompany = { ...CompanyOne, ...CompanyTwo }
console.log(otherCompany)
/* { name: 'Amazon'
     age: 22,
     headquarter: {
        city: 'Mountain View',
        state: "California"
     founder:'Jeff Bezos'
    }}*/  
{{< /highlight >}}

### Object Destructuring in function
Object destructure in function is very useful when we have a large json data which holding many properties but there are only few value we are interested. <br/>
Regular way is to first pass in the entire object then only start access the property value by using [dot notation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Property_accessors)

{{< highlight js>}}
function checkCompany(company) {
    console.log(company.name); //return 'Google'
    console.log(company.age); //return 22
}

checkCompany(CompanyOne)
{{< /highlight >}}

With object destructuring, we can straight away pass in the value we want according to their object name in the curly braces.
{{< highlight js>}}
function checkCompany({ name, age }) {
    console.log(name); //return 'Google'
    console.log(age); //return 22
}

checkCompany(CompanyOne)
{{< /highlight >}}

{{% endLine %}}

<div class="fb-comments" data-href="https://jeffdevslife.com/p/destructuring-array-object/" data-numposts="5" ></div>