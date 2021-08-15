+++
author = "Jeff Chang"
title = "1) MongoDB Query of Advanced Filtering, Sorting, Limit Field and Pagination with mongoose"
date = "2021-08-15"
description = "By the end of this article, you will learn how to query a data from MongoDB by various features"
tags = [
    "nodejs", "mongodb"
]
categories = [
	"NodeJs", "MongoDB"
]
image = "cover.jpg"
+++

First and foremost, before we get started, kindly ensure you have installed MongoDB Driver and imported the mongoose module in your local program.

Besides that, the data going to be used will be at [here](/p/1-mongodb-query-of-advanced-filtering-sorting-limit-field-and-pagination-with-mongoose/tours.json). <br/>
And this will be the Model of the data

{{< highlight js >}}
const mongoose = require('mongoose')
const tourSchema = new mongoose.Schema({
    name: {
        type: String,
        required: [true, 'A tour must have a name'],  // first argument determined whethere it's required, 2nd argument determine err message
        unique: true,   // only allow unique value in this document/collection
        trim: true // only works for String, remove white space in start and end
    },
    durations: {
        type: Number,
        required: [true, 'A tour must have a duration']
    },
    maxGroupSize: {
        type: Number,
        required: [true, 'A tour must have a group size']
    },
    difficulty: {
        type: String,
        required: [true, 'A tour should have difficulty']
    },
    ratingsAverage: {
        type: Number,
        default: 4.5
    },
    ratingsQuantity: {
        type: Number,
        default: 0
    },
    price: {
        type: Number,
        required: [true, 'A tour must have a price']
    },
    priceDiscount: Number,
    summary: {
        type: String,
        required: [true, 'A tour must have a description'],
        trim: true
    },
    description: {
        type: String,
        trim: true
    },
    imageCover: {
        type: String,
        required: [true, 'A tour must have a cover image'],
    },
    images: [String], //Array of string data type
    createdAt: {
        type: Date,
        default: Date.now(),
        // select:false      //excluded this field to be return during http req
    },
    startDates: [Date]
});

const Tour = mongoose.model('Tour', tourSchema)
module.exports = Tour
{{< /highlight >}}

### Load Data (optional)
*Hints:* If you have no idea how to import the data into your local/ online mongodb. Kindly copy and run the code below: 
{{< highlight js >}}
//READ JSON FILE
const tours = JSON.parse(fs.readFileSync(`${__dirname}/tours-simple.json`, 'utf-8'))

//IMPORT DATA INTO DB
const importData = async () => {
    try {
        await Tour.create(tours)
        console.log("Data successfully loaded")
    } catch (error) {
        console.log(error)
    }
}
importData()
{{< /highlight >}}

Let's straight away start by writing the controller of the route, which we define it as `getAllTours`

### Initialization
{{< highlight js >}}
exports.getAllTours = async (req, res) => {
  try {
    let query = Tour.find()
    const tours = await query
    res.status(200).json({
        status: 'success',
        results: tours.length,
    })

  }catch (error) {
    res.status(400).json({
      status: 'fail',
      message: error
    })
  }
}
{{< /highlight >}}

### Advanced Filtering<a name="FILTER"></a>
{{< highlight js "linenos=table,linenostart=1">}}
exports.getAllTours = async (req, res) => {
  try {
    //BUILD QUERY
    // 1A) Filtering
    const queryObj = { ...req.query }
    const excludedFields = ['page', 'sort', 'limit', 'fields']
    excludedFields.forEach(el => delete queryObj[el])

    //1B) Advanced filtering
    let queryString = JSON.stringify(queryObj)
    queryString = queryString.replace(/\b(gte|gt|lte|lt)\b/g, match => `$${match}`)    
    let query = Tour.find(JSON.parse(queryString))

    // 2) Sorting
    if (req.query.sort) {
      const sortBy = req.query.sort.split(',').join(' ')
      query = query.sort(sortBy)
    } else {
      query = query.sort('-createdAt')
    }

    //EXECUTE QUERY
    const tours = await query
    res.status(200).json({
      status: 'success',
      results: tours.length,
      data: {
        tours
      }
    });
  } catch (error) {
    res.status(400).json({
      status: 'fail',
      message: error
    })
  }
};
{{< /highlight >}}

### Test Case
1. `http://localhost:3000/api/v1/tours?ratingsAverage=4.7`, Query the data which ratingsAverage is exactly equal to 4.7
2. `http://localhost:3000/api/v1/tours?ratingsAverage[gte]=4.7`, Query the data which ratingsAverage is greater or equal than 4.7
3. `http://localhost:3000/api/v1/tours?sort=ratingsAverage`, Query the data to sort by ratingsAverage field with ascending order
4. `http://localhost:3000/api/v1/tours?sort=-ratingsAverage`, Query the data to sort by ratingsAverage field with descending order
5. `http://localhost:3000/api/v1/tours?sort=ratingsAverage,ratingsQuantity`, Query the data to sort by ratingsAverage field with ascending order. lower `ratingsQuantity` field data will be listed first if the query has same `ratingsAverage` value

### Explanation
#### Filter
1. Get all query parameters from the url
2. Line 6 ~ 7 is deleting and extracting out some special query params which is not from the data field itself but it's indicating some operational actions keyword like `'page', 'sort', 'limit', 'fields'`
    - `page` and `limit` will be used as Pagination (which we will be covered in the next article)
    - `sort` will be used for sorting
    - `fields` will be used as limit fields which only show certain field from the response body
3. Line 10 ~ 11 is to first convert the query params object into string, then we can use `String.replace` method and regular expression to replace some special operator such as `gte`: Greater or Equal Than, `gt`: Greater Than,  `lte`: Less or Equal Than, `lt`: Less Than
    - Basically we need to convert into this format so that MongoDb will able to recognize these operators 
    - `{durations:{gte: '5' }}` map into => `{durations:{$gte: '5' }}`
4. Then we can convert back the query data type from string back into object by using `JSON.parse` and create our query `Tour.find(< query >)`

#### Sort
1. Sort the field `createdAt` in descending order by default *if the sort param is not included in the url*
2. Line 16 is handling if there are more than 1 params need to be sorted
    * URL will be look something like this => `http://localhost:3000/api/v1/tours?sort=ratingsAverage,ratingsQuantity`
    * MongoDB sort multiple fields will needed this structure `.sort('firstParam secondParam')`
    * We can split the query string from url by comma then join them with a white space in between
