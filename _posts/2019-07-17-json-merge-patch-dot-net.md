---
layout: post
title: "JSON Merge Patch for .NET"
comments: true
description: "A pattern for implementing the JSON Merge Patch Spec in .NET"
keywords: ".net, web api, asp.net, json merge patch"
---

At my day job, I have been working on a new microservice that sits behind a REST API.  I got to the point where I needed to
support updating some resources, and was looking for the "right" RESTful way to accept update requests.  After some research,
I found two options that seemed to fit my needs well, [JSON Patch](https://tools.ietf.org/html/rfc6902),
and [JSON Merge Patch](https://tools.ietf.org/html/rfc7386).

###JSON Patch
I think of JSON Patch like assembly code for updating a resource.  The JSON Patch document contains an array of
operations to perform of the resource.  Each operation describes some change that should be made.  Here is the example
from the RFC:

```
PATCH /my/data HTTP/1.1
 Host: example.org
 Content-Length: 326
 Content-Type: application/json-patch+json
 If-Match: "abc123"

 [
   { "op": "test", "path": "/a/b/c", "value": "foo" },
   { "op": "remove", "path": "/a/b/c" },
   { "op": "add", "path": "/a/b/c", "value": [ "foo", "bar" ] },
   { "op": "replace", "path": "/a/b/c", "value": 42 },
   { "op": "move", "from": "/a/b/c", "path": "/a/b/d" },
   { "op": "copy", "from": "/a/b/d", "path": "/a/b/e" }
 ]
 ```
