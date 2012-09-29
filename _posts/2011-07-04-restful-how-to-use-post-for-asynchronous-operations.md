--- 
layout: post
title: "RESTful: How to use POST for Asynchronous operations"
category: 
tags: [English posts, RESTful]
---
{% include JB/setup %}

I'm writing this post inspired by the book [RESTful Web Services Cookbook](http://www.amazon.com/dp/0596801688/). This book has many useful examples on how to use [RESTful Web Services](http://en.wikipedia.org/wiki/Representational_State_Transfer#RESTful_web_services). If you are looking for a book on RESTFul I will highly recommend this book.

This post is an example of an Asynchronous POST using RESTful. I chose this example, because it covers good principles of RESTful, for example the proper usage of HTTP codes.

## Introduction

The Asynchronous POST will be used on this example to create an invitation to a friend. Typically used on social networks. This operation must be async, users can't wait for the operation to complete.

The are many ways to do async operations that fit in our case, for example using [Resque](https://github.com/blog/542-introducing-resque) and [JMS](http://en.wikipedia.org/wiki/Java_Message_Service) etc. These tools will not be covered on this post.

Mapping resources and operations in RESTful is done by using nouns as resources and verbs (HTTP) as operations, similar to the technique used in OOP mapping.

Although RESTful hasn't an official standard for HTTP verbs into operations. We will follow the convention bellow to operate resources.

* Create = POST
* Retrieve = GET
* Update = PUT
* Delete = DELETE

## Create an invitation

    # REQUEST
    POST /invitation HTTP/1.1
    Host: www.example.org
    Content-Type: application/json
    
    {
      "from": "flanders@simpsons.com",
      "to": "homer@simpsons.com",
      "message": "Okilly-dokilly!"
    }
    
    # RESPONSE
    HTTP/1.1 202 Accepted
    Content-Type: application/json
    Content-Location: http://www.example.org/invitations/tasks/1
    Date: Mon, 4 July 2011 17:25:10 GMT
    
    {
      "status": "pending"
      "links": [
         "rel": "self"
         "href": "http://www.example.org/invitations/tasks/1"
       ]
    }

### Content-Type: application/json

I didn't specify the charset, because JSON uses [UTF-8 by default](http://www.ietf.org/rfc/rfc4627.txt).

### HTTP/1.1 202 Accepted

The response `202 Accepted` was used to represent that the operation wasn't completed yet. The operation was only accepted, kind of "We are working on it. Come back later".

## Consulting the invitation status

By using the previous `link rel` received in the last response, we can consult the invitation status.

    # REQUEST
    GET /invitations/tasks/1 HTTP/1.1
    Host: www.example.org
    
    # RESPONSE
    HTTP/1.1 200 OK
    Content-Type: application/json
    Content-Location: http://ww.example.org/invitations/tasks/1
    Date: Mon, 4 July 2011 17:25:10 GMT
    
    {
      "status": "pending"
      "links": [
         "rel": "self"
         "href": "http://ww.example.org/invitations/tasks/1"
       ]
    }

Even though the invitation wasn't yet sent, the response code was `200 OK` because the operation "Consulting the invitation status" was completed. It isn't a pending operation.

## The server successfully completes the processing

    # REQUEST
    GET /invitations/tasks/1 HTTP/1.1
    Host: www.example.org
    
    # RESPONSE
    HTTP/1.1 302 See Other
    Content-Type: application/json
    Location: http://ww.example.org/invitations/1
    Content-Location: http://ww.example.org/invitations/taks/1
    Date: Mon, 4 July 2011 17:25:10 GMT
    
    {
      "status": "done"
      "links": [
         "rel": "self"
         "href": "http://ww.example.org/invitations/1"
       ]
    }

After the server successfully completes the processing, the response code will be `303 See Other` and the `Content-Location` header will contain the url to retrieve the generated invitation.


