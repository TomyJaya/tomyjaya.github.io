---
layout:     post
title:      "Checklist for Restful API Design"
subtitle:   "Things You Must Know as a Backend API Developer"
date:       2018-02-01 12:00:00
author:     "Tomy Jaya"
header-img: "img/post-bg-checklist.jpg"
prefix:     "[Popular 📈] "
tags:
- design
- api
- rest
---

**NOTE:** Last Updated on **11 Jan 2020**

## Introduction: 

The following are best practices I curated when designing RESTful APIs. They should seem very familiar to seasoned backend developers; nevertheless, it's always good to have good old checklist you can rely on. 

A disclaimer: with the advent of GraphQL, most of these principles might seem redundant. In fact, I see GraphQL as an evolution of REST (to quote the doc, "a better REST"). E.g. Things like projection and query are baked into the specifications. That said, for some commoners who don't have the luxury to jump to the bandwagon yet, without further ado, here comes the list: 

---

### 1. Use resources endpoint and HTTP Verbs

Instead of having, the below endpoints: 

```
GET /getAllUsers
GET /getUser?id=1
POST /createUser?id=1&name=a&password=b
POST /deleteUser?id=1
POST /updateUser?id=1&name=c&password=d
```

follow the convention to have resources (nouns) with the corresponding HTTP vebs: 

| HTTP VERB   | CRUD Action | 
| ------------- | ------------- | 
| GET  |  Read |
| POST | Create  |
| PUT | Update  |
| DELETE | Delete  |

You should design API with as follows: 

```
GET /users      // Get All Users
GET /users/1    // Get User 1
POST /users     // Create a new user with the JSON Body
PUT /users/1    // Update User 1 with the values in the JSON body
DELETE /users/1 // Delete User 1
```

Things to note: 
1. resources name should be plural (e.g. `users` instead of `user`)
2. `PUT` can support upsert and should be idempotent
3. `GET` shouldn't alter state
4. If you have nested items, it should be traverseable via the main resource. e.g. `GET /users/1/wallets/2` should get User 1's 2nd wallet. 

### 2. Return correct HTTP codes

The below are some of the commonly neglected ones: 

| HTTP Status Code  | When to use |
| ------------- | ------------- |
| 201 Created  | After a successful POST call |
| 204 No Content  | After a successful DELETE call  |
| 304 Not Modified | if GET request result can be cached |

While `200` & `500` are common, ensure that you correctly return `404`, `400`, `401` instead of wrongly returning the cryptic dreaded `500`. 

### 3. Support for filtering

Expose simple ways to filter your resources by field values. E.g. 

```
GET /users?name=Bob    // Returns a list of users with name "Bob"
```

I recommend making this exact-case by default and refrain from supporting more advanced features just as `startsWith`, wild cards, or even number comparison operators (e.g. `<=`). Simple filtering should do. 


### 4. Support for sorting

Implement sort by field and a toggle for direction:

```
GET /users?age=20&sort=name,desc  //  returns a list of users of age 20 and descending name
```


### 5. Support for fields projection

Implement ways to select fields to project: 

```
GET /users?fields=name,age
```

Might save some bandwidth. 


### 6. Support for pagination

Allow clients to define page size limit and select which page. 

```
GET /users?page=1size=2
```

Gels well with HATEOAS explained later.  


### 7. Support for search

A global text-indexed search the resource will come in handy:

```
GET /users/search?criteria=sky // returns any item with word sky (e.g. user with name Anakin Skywalker)
```

Chances are, your front-end dude will want this. You either can build a dedicated solution such as ElasticSearch or just roll-up your sleeves to offer an alternative simpler solution. Move on to the more complex solution if our simple search becomes a bottleneck. 

### 8. API versioning

I can't stress enough: your API will morph. Version it upfront to cover your ass: 

```
GET /api/v1/users   // supports [name, age, id]

// 3 years later

GET /api/v2/users   // supports [name, id]


```

*Note*: Don't use fancy version semantics which is not URL friendly. Integer should work . Also, don't rely on header for versioning. Most backend solution offers easier way to route based on path than based on header. 

*Another Note*: Some API designers decide to put versioning in the header. I have no strong preference on whether to use URL-based or header-based versioning. But the bottom line remains, you need to have versioning!

### 9. Control serialization format via HTTP header

When working with JSON: 

```
Accept: application/json
Content-Type: application/json
```

if by a fluke of nature, you're still using XML: 

```
Accept: text/xml
Content-Type: text/xml
```

*Note*: I know some people who advocate having a specific format query parameter to handle serialization format. Don't listen to them. They probably didn't understand `Accept` and `Content-Type` headers. 


### 10. Use HATEOAS

This one is controversial. If you don't know what HATEOAS is, it's basically a specifications to embedded navigation information (i.e. links) in your payload response. 

For example: 

```json
{
   "firstName": "Anakin",
   "lastName": "Skywalker",
   "_links": {
       "self": {
           "href": "http://localhost:8080/users/1"
       },
       "spouse" : {
           "href": "http://localhost:8080/users/1"
       }
   }
}
```

I know there is some debate that HATEOAS is unnecessarily cluttering your response, but I believe if we all start to follow these standards, front-end developers can develop better libraries which support APIs following this standard out of the box. Pagination is a good example. 

```
{
 "_links" : {
    "self" : {
      "href" : "http://localhost:8080/users{&sort,page,size}", (1)
    },
    "next" : {
      "href" : "http://localhost:8080/users?page=1&size=5{&sort}", (2)
    }
  },
  "page" : { 
    "size" : 5,
    "totalElements" : 50,
    "totalPages" : 10,
    "number" : 0
  }
}
```

*Final note*: I recommmend HAL for the HATEOAS implementation. 


### 11. Use camelCase

camelCase your payloads

```
{ 
    "firstName": "Tomy",
    "lastName": "Jaya"
}
```

instead of:


```
{ 
    "first_name": "Tomy",
    "last_name": "Jaya"
}
```

Why? It's the convention recommended by the JSON spec. 

### 12. Use error object

Having error payloads single-handedly separates the novice and the erudite API designers. Returning a `500` status code with the server stacktrace is just not cool anymore.

Sample error object you can start with : 

```
{
  "code" : 1234,
  "message" : "Error will saving user",
  "description" : "User has already exists"
}
```

### 13. Pretty print by default

People say pretty print wastes bandwidth, but we're at the age where data is cheap and internet connection is blazing fast. You won't feel the difference due to those few extra whitespaces. What makes the difference is when you have to start typing `python -m json.tool` or `jq` when debugging. 

### 14. Have a versioned documentation of your API with plenty of examples

I can't stress enough the importance of having API documentation. This documentation should: 

1) have versioning with "What's New" type of release note (include breaking changes and deprecation)
2) have plenty of examples (Read [Badass](http://amzn.to/2qf9JqS) to understand the importance of examples)

Personally, I am *not* a big fan of tools like Swagger. It's probably because most of the APIs I worked on have huge payloads and Swagger UI runs very slowly with huge payload. Also, there are lots of minor customizations you can make in a wiki-based API which could make a lot of difference to your API user-friendliness. For instance, you can add "info", "warning", "danger" markers as reminder to your API users. Ergo, Swagger is a good starting point, but definitely not the final ideal state I want to be in.  

## Public API

The below are more relevant for public API designers:

1. Use Rate Limiter
2. Use OAuth
3. Always support https
4. Support Gzip 
5. Use HTTP Override Method 

---

## References: 

1. https://hackernoon.com/restful-api-designing-guidelines-the-best-practices-60e1d954e7c9
2. http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api
3. https://blog.mwaysolutions.com/2014/06/05/10-best-practices-for-better-restful-api/


# Update 11 Jan 2020

## General Guidelines

More API design tips I learn along my journey:

### 1. Converting Synchronous to Asynchronous
If your API has a long processing time, redesign your API to immediately return a response with status PENDING with some sort of tracking ID. The client can later use this tracking ID to query the status of the process and final result when the processing completes. Alternatively, the client can expose a webhook which can receive notification in case the processing completes. This trick is incredibly useful to convert the synchronous and timeout-based nature of HTTP to match some processes which are inherently asynchronous (i.e. long-running).  

Instead of: 
```
  Client               Server
    |                     |
    | ----- request  ---> |
    |                     | } 
    |                     | } // Long running process (e.g. 2 mins)
    |                     | } 
    | <- - - response - - |
```

Do this:

```
  Client                   Server
    |                             |
    | ----- request  ------->     | 
    | <- - - jobId: 7 - - - - - - | 
    |                             | 
    |                             | //  process in background
    | // in 1 min                 | 
    | -- query jobId 7 status ->  |
    | <- - - PENDING - - - - - -  | 
    |                             | 
    | // in another 1 min         |  
    | -- query jobId 7 status ->  |
    | <- - - DONE & response - -  | 
```

Or this: 

```
  Client                          Server
    |                                 |
    | ----- request & webhook ----->  | 
    | <- - - jobId: 7 - - - - - - - - | 
    |                                 | 
    |                                 | //  process in background
    |                                 | 
    |                                 | 
    | <--- DONE & response ---------  | // via webhook 
```

### 2. Use Wrapper Object instead of Primitives
Never respond with a plain a primitive `String` or `Array`. Always wrap it in an object. Your API might evolve in the future and commmitting to a primitive early is a sure way to the treacherous highway of no coming back. 

For instance, for the simple API to create a new blog post below:

`POST` `/api/posts` 

payload: 

```
{
  "title": "My first blog post",
  "content": "The content of my first blog post"
}
```

you might be tempted to just return a status response as string:

`SUCCESS` or `ERROR`

It's okay for a POC, but a better response would be something like the below: 

```
{
  "status": "SUCCESS"
}
```

Or even better, 

```
{
  "status": {
    "code": "SUCCESS"
  } 
}
```

because in case when there's an error, a description of the error will be tremendously useful: 

```
{
  "status": {
    "code": "ERROR",
    "message": "Duplicate title not allowed"
  } 
}
```

Same goes with returning a plain array. Suppose you have an API which returns list of books: 

`GET` `api/books`

```
[
  { 
    "author": "Josh Bloch",
    "title": "Effective Java"
  },
  {
    "author": "Doug Crockford",
    "title": "JavaScript: The Good Parts"
  }
]
```

And one day, your API client needs more meta-data information about the result, e.g. the count of the records. It would then be impossible to extend your previous API to accomodate this requirement. Instead, if you had started with: 

```
{
  "books": [
    { 
      "author": "Josh Bloch",
      "title": "Effective Java"
    },
    {
      "author": "Doug Crockford",
      "title": "JavaScript: The Good Parts"
    }
  ]
}
```

You can easily make backward compatible enhancements: 

```
{
  "_meta": {
    "_count": 2
  },
  "books": [
    // ...removed for brevity
  ]
}
```


### 3. Entity-oriented, but not Exactly the DB schema

In the first item of the original blog post, I recommended a resource-style or entity-oriented approach instead of procedural style for designing API. This is now a well-known de facto recommendation. 

However, the question now is should your entity/ resource exactly mimic your database schema? The answer is no. Your database schema might be highly normalized. For example, you have a `User` relation/ table and it can have multiple `UserRole`s. The below API design is **not** recommended:

`GET` `/api/users/1`

```
{
  "id": "/api/users/1",
  "username": "TJ",
  "roles": ["/api/users-roles/1","/api/users-roles/2"]
}
```

`GET` `/api/user-roles/1`

```
{
  "id": "/api/user-roles/1",
  "role": "ADMINISTRATOR"
}
```

`GET` `/api/user-roles/2`

```
{
  "id": "/api/user-roles/2",
  "role": "SUPER_USER"
}
```

Imagine how much work your API client has to do with just that one trivial example above. 

Instead, your API should return a response which is self-contained. 

```
{
  "id": "/api/users/1",
  "username": "TJ",
  "roles": ["ADMINISTRATOR","SUPER_USER"]
}
```

This approach also results in simpler caching e.g. you can just flush the entire "contructed" object into your Redis cache. Double whammy!

### 4. Don't add versioning in the URI

I previously didn't have any strong preference on where to put API versioning in. But [this talk at 35:32](https://www.youtube.com/watch?v=P0a7PwRNLVU) convinced me HETEOAS together with API versioning in the URI just don't gel well together. The example of given by the presenter was:

**WRONG**:

```
{
  "id": "/v1/dogs/12345",
  "name": "Lassie",
  "owner": "/v1/persons/1"
}
```

**CORRECT**:

```
{
  "id": "/dogs/12345",
  "name": "Lassie",
  "owner": "/persons/1"
}
```

This is because you might use the newer version of `dogs` API, but at the same time, **not** yet ready to use the newer version of `persons` API yet. The better recommendation is to add a couple of extra properties: 

```
{
  "contentVersion": "v1",
  "contentLocation": "/v1/dogs/12345",
  "id": "/dogs/12345",
  "name": "Lassie",
  "owner": "/persons/1"
}

```

You can then allow clients to use `Accept-Version` header to ask for a specific version. 

**DISCLAIMER**: I have yet to apply this technique in a real production API yet.

### 5. Trade-off between Indexed vs flat API responses

Below is an example of an indexed collection:

```
{
  "<user-id-a>": {
    "data": [{
      "id": "12345",
      "picture": "<photo-url>",
      "created_date": "2015-01-01" 
    },
    // More photos...
    ]
  },
  "<user-id-b>": {
    "data": [{
      // Removed for brevity
    }]
  }
}
```

The above representation favors a certain access pattern. E.g. by the userId. 

A more flexible representation would be a flat array: 

```
[
  {
    "id": "12345",
    "owner": "<user-id-a>", // Notice here
    "picture": "<photo-url>",
    "created_date": "2015-01-01"
  },
  // more photos...
]
```

This generic representation still allows you to conveniently filter by `owner` to get the list of photo objects per user. You can even filter by other properties, e.g. get all photos with certain `created_date`. In contrary, it will be non-trivial to achieve the same in an index biased collection.  


### 6. gRPC (tight-coupling vs performance)

Should your REST API use gRPC? A simple guiding principle is decide what you really care about. 

a) If you want efficient communication between tightly-coupled components, gRPC is a great choice. 

b) If you want to decouple components for future changes and integrate multiple systems, use HTTP/ JSON. 

### 7. Homogenous collections only

This one is very intuitive, but sometimes forgotten. In a nutshell, you should never a collection with mixed types. E.g. 

```
// Bad example:
{
  "results": [ 1,2,"divide_by_zero_error"]
}
```

This is because your API client might be a strongly typed language and dealing with the above heterogeneous example will be hugely inconvenient. Instead, you can normalize your response as follows:

```
{
  "results": [{
    "status": "SUCCESS",
    "result": 1
  }, {
    "status": "SUCCESS",
    "result": 2
  }, {
    "status": "ERROR",
    "errorMessage": "divide_by_zero_error"
  }]
}
```


## Public APIs
1. **On API Compression**: there's now a superior compression format than gzip, called brotli (`content-encoding: br`). The resulting compressed data can be around 20% smaller than gzip as it's optimized using a dictionary. As a good rule-of-thumb, only set up high `brotli_comp_level` (aka compression level) for static assets as higher compression level will require more time to compress and in the case of dynamic data, it will backfire by increasing latency. 
2. For public APIs with performance in mind, prefer coarse-grained API responses. More granular responses will result in more chatty communication and therefore, increased undesirable network roundtrips. That being said, always return "just-enough" response, nothing more, nothing less. This is especially true for APIs supporting mobile devices. Though counter-intuitive, invest in building mobile device-specific API response formats. At first, this defeats the purpose of RESTful API standardization, but actually, nowadays, there are numerous technologies available to easily cater to API response of different shapes.   
3. Abuse (or rather, make full use of) the `Cache-control` header! 

## References
1. [Designing Quality APIs (Cloud Next '18)](https://www.youtube.com/watch?v=P0a7PwRNLVU)
2. [Brotli vs Gzip Compression. How we improved our latency by 37%](https://medium.com/oyotech/how-brotli-compression-gave-us-37-latency-improvement-14d41e50fee4)





