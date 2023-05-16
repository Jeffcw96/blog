+++
author = "Jeff Chang"
title = "Understanding GraphQL Query and Mutation in Apollo Server"
date = "2023-05-15"
description = "In this article, we will be going through the fundamental of GraphQL in Apollo Server with NodeJs"
tags = [
    "nodejs", "graphql"
]
categories = [
    "Node","Javascript", "GraphQL"
]
+++

## Table of contents

- [Introduction](#introduction)
- [Schema and Types](#schema-type)
- [Create your own Type and Query](#create-type-and-query)
- [Create Mutations](#mutations)
- [Link multiple query type](#link-types)
- [Final outcome](#final-outcome)

### Introduction<a name="introduction"></a>

Before we get started, it's high recommend to clone the [Github repo](https://github.com/Jeffcw96/graphlq-learning-journey) and we can follow along with the article contents.

The first step after we downloaded the repo is to checkout the the **starter** folder and run `npm install` and `npm run start` to start our Apollo GraphQL server locally.

We can now navigate to http://localhost:4000/ and here is how it should look like

![apollo-graphql-server](apollo-graphql-server.png)

We could then click on the **Query your server** button and it shall brings us to the GraphQL editor

### Schema and Types<a name="schema-type"></a>

There are total of 5 scalar types available in GraphQL

- `ID`: The ID scalar type represents a unique identifier, often used to refetch an object or as the key for a cache. The ID type is serialized in the same way as a String; however, defining it as an ID signifies that it is not intended to be human‐readable.
- `Int`: A signed 32‐bit integer.
- `Float`: A signed double-precision floating-point value.
- `String`: A UTF‐8 character sequence.
- `Boolean`: `true` or `false`.

Here is one of the GraphQL example type in our Github repo:

```graphql
type User {
  id: ID!
  name: String!
  age: Int!
  nationality: String!
  friends: [User!]
}
```

We now structed our `User` type to have a `id` field to be an `ID` scalar type which most of the time will be our unique identifier from the database. We also have the rest of the fields called `name`, `age`, `nationality` and `friends`.

The exclamation mark right beside of the scalar type is stands for **non-nullable** field. In other words, these field must be existed from our data source when we are querying from the GraphQL command.

For the `friends` field, it's a little special as it's an **Array** type. `[User!]` scalar type means the field will now contains whatever we have specified for the `User` type. It also means it's a **nullable** field **BUT** when it also means it cannot be a empty array

Checkout the [official docs](https://graphql.org/learn/schema/) if you are keen to learn more
