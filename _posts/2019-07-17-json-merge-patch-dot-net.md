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

### JSON Patch
I think of JSON Patch like assembly code for updating a resource.  The JSON Patch document contains an array of
operations to perform on the specified resource.  Each operation describes some change that should be made.  Here is an example
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

This example shows the different types of operations defined in the RFC.  The idea is not very complex.  There are only
a handful of operations, and if I squint my eyes, I can see what is going on.  But, something about the format just
feels unnatural.  Some of the cool things about REST are how human friendly it is, how discoverable a well designed API
can be.  If I want to get a resource, I send a GET request, and the URL generally maps nicely to the thing I am looking
for.  If I want the `thing` with `ID=1` I ask for `/thing/1`.  When I look at a JSON Patch request, I see assembly code,
and I just don't like it.

FWIW, .NET does has some support for [JSON Patch](https://docs.microsoft.com/en-us/dotnet/api/microsoft.aspnetcore.jsonpatch.jsonpatchdocument-1?view=aspnetcore-2.2).  And if you combine that with some
[client side libraries](https://www.npmjs.com/package/fast-json-patch), generating the patch document would be feasible.
But I just wasn't sold on the approach.

### JSON Merge Patch
