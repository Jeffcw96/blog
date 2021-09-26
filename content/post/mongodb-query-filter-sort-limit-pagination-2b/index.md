+++
author = "Jeff Chang"
title = "2) MongoDB Query of Advanced Filtering, Sorting, Limit Field and Pagination with mongoose"
date = "2021-08-28"
description = "By the end of this article, you will learn how to query a data with Pagination and Field Limiting methods"
tags = [
    "nodejs", "mongodb"
]
categories = [
	"NodeJs", "MongoDB"
]
image = "cover.jpg"
+++
Prerequisite
1. Kindly ensure you've gone through the earlier chapter of this article. [mongodb-query-filter-sort-limit-pagination](/p/1-mongodb-query-of-advanced-filtering-sorting-limit-field-and-pagination-with-mongoose)
2. You can also get the example data used in this article. [tours.json](/p/1-mongodb-query-of-advanced-filtering-sorting-limit-field-and-pagination-with-mongoose/tours.json)


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

    //3) Field Limiting
    // Select pattern  .select("firstParam secondParam"), it will only show the selected field, add minus sign for excluding (include everything except the given params)
    if (req.query.fields) {
      const fields = req.query.fields.split(',').join(' ')
      query = query.select(fields)
    } else {
      query = query.select('-__v')
    }

    // 4) Pagination
    // page=2&limit=10, 1-10 page 1, 11-20 page 2, 21-30 page 3
    const page = req.query.page * 1 || 1;
    const limit = req.query.limit * 1 || 100;
    const skip = (page - 1) * limit;

    query = query.skip(skip).limit(limit);

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
1. `http://localhost:3000/api/v1/tours?fields=name,durations,price`, Query the data which only return name, durations and price
2. `http://localhost:3000/api/v1/tours?limit=10&page=2`, Query the data which skip the first 10 results

### Explanation
#### Field Limiting (start from line 22)
1. If the 'fields' query paramater is existed, split the parameter from comma to white-space so that it can read and limit multiple fields
2. Apply mongoose `select` methods to only return the selected field.
3. Exclude out the mongo version field '__v' by simply put a negative sign **(-)** infront

#### Pagination (start from line 31)
1. Pagination is very useful implementation when you are dealing with a very large dataset.
2. Combining `.skip` and `.limit` methods allow us to achieve pagination.
    - `.skip`: total records we wanted to skip from the query
    - `.limit`: total records we wanted to show from the query
3. As we are passing the **Page Number** `page` and **Total Records** `limit` from the query parameters. We can then come out with the formula
    - `const skip = (page - 1) * limit`. This will let us know how mnay records we are required to skip and paginate to our desire page number
