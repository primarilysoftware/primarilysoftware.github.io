---
layout: post
title: "JSON Merge Patch for .NET"
comments: true
description: "A pattern for implementing the JSON Merge Patch Spec in .NET"
keywords: ".net, web api, asp.net, json merge patch"
---

> Source code related to this post is available [here](https://primarilysoftware.visualstudio.com/_git/JsonMergePatch).
> A nuget package is also available - [JsonMergePatch](https://www.nuget.org/packages/JsonMergePatch/1.0.0-CI-20190822-054746).

At my day job, I have been working on a new microservice that sits behind a REST API.  I got to the point where I needed to
support updating some resources, and was looking for the "right" RESTful way to accept update requests.  After some research,
I found two options that seemed to fit my needs well, [JSON Patch](https://tools.ietf.org/html/rfc6902)
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
JSON Merge Patch attempts to solve a similar problem as JSON Patch but takes a different approach.  While I see JSON Patch
documents as assembly code, a JSON Merge Patch looks more like a Diff between two documents.  Again, lets look an example
from the RFC:

```
PATCH /target HTTP/1.1
Host: example.org
Content-Type: application/merge-patch+json

{
  "a":"z",
  "c": {
    "f": null
  }
}
```

In a JSON Merge Patch request, the shape of the JSON object is the same as that of the resource you want to update,
however, you only specify the fields you want to change.  In this case, we should update the `a` field, assigning it
the value `z`.  Also, `c.f` should be set to `null`.  Imagine the target resource for the above request had some
field `b = 123`.  Since it was ommitted from the patch request, that field should not be changed.

I find a certain elegance in this approach.  No special syntax, just send the fields you want to update. Admittedly,
it is not as flexible as the JSON Patch format.  For instance, there is not a succinct way to express that you want to remove a
single item from an array.  To modify an array, the merge patch request should include the entire modified array.  Contrast
that with JSON Patch, where you could send an operation like `{"op": "remove", "path": "/things/1"}`.

Even with the constraints, I still prefer the JSON Merge Patch approach.  It just feels right.  Unfortunately, .NET
does not currently have any support for it.  At the time of this writing, there is an
[open issue](https://github.com/aspnet/AspNetCore/issues/2436) on the ASP.NET Core GitHub repo, but there hasn't been
any movement.  A couple of community members have offerred solutions, but they didn't quite work for my needs.  So,
lets see if we can come up with a workable solution.

### A Naive Approach
Let's start with the simplest thing that could possibly work.  Imagine in our app, we have a `Person` class.  A
person has a `Name` and an `Address`.

```csharp
public class Person
{
    public Guid Identifier { get; set; }
    public string? Name { get; set; }
    public Address? Address { get; set; }
}

public class Address
{
    public string? Street { get; set; }
    public string? City { get; set; }
    public string? State { get; set; }
    public string? ZipCode { get; set; }
}
```

We need to expose a way to update a person.  So we might create an endpoint like this:

```csharp
[ApiController]
public class ApiController
{
    [HttpPatch]
    [Route("naivepatch/person/{personId}")]
    public IActionResult NaivePatch(Guid personId, [FromBody]Person data)
    {
        // TODO update person
    }
}
```

Given a request like:

```
PATCH /naivepatch/person/1c8ff3f2-16d5-4a5a-94d1-52f25f2a80e6 HTTP/1.1
Content-Type: application/json
Host: localhost:5000
Content-Length: 33

{
    "name": "test person"
}
```

According to the JSON Merge Patch spec, we should only update the fields that are supplied in the request (in this case, update `Name`
setting the value to "test person").  We might try and write some code like:

```csharp
public IActionResult NaivePatch(Guid personId, [FromBody]Person data)
{
    var person = /* load from data store using personId */;

    if (data.Name != null)
    {
        person.Name = data.Name;
    }

    if (data.Address != null)
    {
        person.Address = data.Address;
    }

    /* save updated person */
    
    return new OkResult();
}
```

This should handle our request just fine.  We are done then, right?  Not quite.  How would we handle a request like:

```
PATCH http://localhost:5000/naivepatch/person/1c8ff3f2-16d5-4a5a-94d1-52f25f2a80e6 HTTP/1.1
Content-Type: application/json
Host: localhost:5000
Content-Length: 26

{
   "address": null
}
```

The intent of this request is to clear this person's address, setting it to `null`.  However, after model binding has completed,
there is no way for us to tell in our controller action whether Person.Address is `null` because `address` was not included in
the request, or `null` because `address` was set to `null` in the request.

We are going to need to do a bit more work.

### A Better Design
Simple csharp classes do not give us a way to distinguish between a `null` value and an `undefined` value, so we will need
to wrap the model being passed into our action in a way that allows for this distinction.  Imagine if instead of accepting a
`Person` in our controller action, we did something like:

```csharp
[HttpPatch]
[Route("betterpatch/person/{personId}")]
public IActionResult BetterPatch(Guid personId, [FromBody]JsonMergePatch<Person> request)
{
    var person = /* load from data store using personId */;

    if (request.IsDefined(x => x.Name, out var name))
    {
        person.Name = name;
    }

    if (request.IsDefined(x => x.Address, out var address))
    {
        person.Address = address;
    }

    /* save updated person */

    return new OkResult();
}
```

This hypothetical `JsonMergePatch<T>` class would allow us to first inspect whether a value was supplied in the request,
and if so, extract the value and do something with it.  Sadly this class does not actually exist... yet.  Let's build it.

### JsonMergePatch<T>
Given this is **Json**MergePatch, we will be starting from a JSON document.

> For a long time, if you were dealing with JSON in .NET, you would use [Json.NET](https://www.newtonsoft.com/json).
> While this library has served us well for many years, it wasn't something that shipped "in the box" with .NET.
> With .NET core 3.0, we have a new toy to play with,
> [System.Text.Json](https://devblogs.microsoft.com/dotnet/try-the-new-system-text-json-apis/).
> Let's take this as a chance to play with these new bits.

We will construct our class as follows:

```csharp
public class JsonMergePatch<T>
{
    private readonly JsonDocument json;

    private JsonMergePatch(JsonDocument json)
    {
        this.json = json;
    }
    
    public static JsonMergePatch<T> Create(JsonDocument json)
    {
        return new JsonMergePatch<T>(json);
    } 
}
```

> You might be wondering why the static `Create` method.  This is a pattern I have been smitten with somewhat recently.
> I like it because it allows me to have a single constructor, which makes it easier for me
> to reason about the invariants of my class upon construction.
>
> While we haven't gotten there yet, you could imagine
> wanting to construct a `JsonMergePatch` object from a `string`, or maybe a `Stream`.  Instead of
> having several different constructors, I can create overloads for `Create` that would convert the supplied
> parameters to the `JsonDocument` type that my class really depends upon.
>
> Also, this pattern allows for things
> like `async` construction.  Maybe I want an overload like
> `Task<JsonMergePatch<T>> Create(Uri uri)` that fetches the specified resource asynchronously and converts it to a
> `JsonMergePatch<T>`.  Can't do that with constructors.

That is the easy part.  The interesting code will be in the `IsDefined` method.

```csharp
public bool IsDefined<TProperty>(Expression<Func<T, TProperty>> propertyExpression, out TProperty value)
{
    ...
}  
```

We want to enforce some level of type safety when pulling from the JSON document.  To help with that we are taking
a `propertyExpression` to tell us which property to search for.  Similar to the various `TryParse` methods in the 
BCL, we take an `out` parameter of `TProperty`, and return a `bool`.  If we find the specified property in the 
JSON document, we will set the `out` parameter, and return true.  Otherwise, we will return false.

The `JsonDocument` we are working with is a tree of `JsonElement`s.  We can use the 
[JsonElement.TryGetProperty(string, out JsonElement)](https://docs.microsoft.com/en-us/dotnet/api/system.text.json.jsonelement.trygetproperty?view=netcore-3.0)
method to test if a property exists in the JSON document, and if so get its value.  However, this API takes a `string`,
while our method takes an `Expression`, so first we will need to convert our property expression to the name of a property.

Parsing expression trees to get property names is a common problem.  There are *alot* of code snippets out there
showing how it can be done.  Here is how I am going to approach it:

```csharp
private Stack<string> GetPropertyPath<TProperty>(Expression<Func<T, TProperty>> pathExpression)
{
    var memberExpression = pathExpression.Body as MemberExpression;
    var names = new Stack<string>();

    while (memberExpression != null)
    {
        names.Push(memberExpression.Member.Name);
        memberExpression = memberExpression.Expression as MemberExpression;
    }

    return names;
}
```

One thing to note here is I am returning a `Stack<string>`.  I wanted to be able to support accessing nested properties
directly, like `request.IsDefined(x => x.Address.City, out var city)`.  To do that, we will need to keep track of the
*property path* from the root of the JSON document down to the value we are interested in.

Now that we have a function to get the property names, let's use it to start searching the `JsonDocument`.

```csharp
public bool IsDefined<TProperty>(Expression<Func<T, TProperty>> propertyExpression, out TProperty value)
{
    var propertyPath = this.GetPropertyPath(propertyExpression);
    var jsonElement = this.json.RootElement;

    while (propertyPath.Count > 0)
    {
        var propertyName = propertyPath.Pop();
        if (jsonElement.TryGetProperty(propertyName, out var childElement))
        {
            jsonElement = childElement;
            continue;
        }
        
        // if we got here, the property didn't exist in the JsonDocument, so return false
        value = default!;
        return false;
    }
    
    // TADA! jsonElement should now contain the value we were searching for, now what?
}
```

This code is a little gnarly because we are trying to support nested properties.  We loop through the
property names in our `propertyPath` one by one.  Each time we find the specified property, we
descend deeper into the `JsonDocument` until either we don't find the specified property, or we
reach the end of `propertyPath`.  If we get out of the loop without returning, we have the value
we are looking for.

At this point we have a `JsonElement`.  We need to convert this to a value of `TProperty`.  For that, we can use
[JsonSerializer.Deserialize](https://docs.microsoft.com/en-us/dotnet/api/system.text.json.jsonserializer.deserialize?view=netcore-3.0#System_Text_Json_JsonSerializer_Deserialize__1_System_String_System_Text_Json_JsonSerializerOptions_)

```csharp
public bool IsDefined<TProperty>(Expression<Func<T, TProperty>> propertyExpression, out TProperty value)
{
    ...
    value = JsonSerializer.Deserialize<TProperty>(jsonElement.GetRawText());
    return true;
}
```

> One of the points of emphasis in the new `System.Text.Json` APIs was to improve performance and minimize
> allocations.  It is unfortunate that there does not appear to be a way to deserialize a `JsonElement`
> without first converting it to a `string`.  There is a [issue](https://github.com/dotnet/corefx/issues/37564)
> on github suggesting a fix for this, but for now I think this is the best we can do.

Beautiful!  With what we have so far, our `JsonMergePatch` class should work as intended.  There is one small issue
that you will run into pretty quickly though.  The `TryGetProperty` method we are using to find properties in the
`JsonDocument` is case sensitive.  Quite often, the property names of our csharp classes will be `PascalCase`,
while the property names in the JSON will be `camelCase`.  Let's tweek the code a bit so that we can match properties
whether they are `PascalCase` or `camelCase` in the JSON document.

```csharp
public bool IsDefined<TProperty>(Expression<Func<T, TProperty>> propertyExpression, out TProperty value)
{
    var propertyPath = this.GetPropertyPath(propertyExpression);
    var jsonElement = this.json.RootElement;
    var camelCase = false;

    while (propertyPath.Count > 0)
    {
        var propertyName = propertyPath.Pop();
        if (jsonElement.TryGetProperty(propertyName, out var childElement))
        {
            jsonElement = childElement;
            continue;
        }

        if (jsonElement.TryGetProperty(CamelCase(propertyName), out childElement))
        {
            camelCase = true;
            jsonElement = childElement;
            continue;
        }

        value = default!;
        return false;
    }

    value = JsonSerializer.Deserialize<TProperty>(
        jsonElement.GetRawText(),
        new JsonSerializerOptions
        {
            PropertyNamingPolicy = camelCase ? JsonNamingPolicy.CamelCase : null
        });

    return true;
}

private static string? CamelCase(string name)
{
    if (name == null || name.Length == 0)
    {
        return name;
    }

    if (name.Length == 1)
    {
        return name.ToLower();
    }

    return char.ToLower(name[0]) + name.Substring(1);
}
```

Now we will first search for a property that exactly matches the name from the property expression.  If one
is not found, we will try to convert the property name to `camelCase` and try again.  Also note that when
deserializing the `JsonElement` to `TProperty`, we need to specify whether the source document was `camelCase`
so that things are deserialized correctly.

### Model Binding
We had updated our controller action to accept a `JsonMergePatch` parameter:

```csharp
[HttpPatch]
[Route("betterpatch/person/{personId}")]
public IActionResult BetterPatch(Guid personId, [FromBody]JsonMergePatch<Person> request)
```

This is not going to work quite yet.  We need to tell ASP.NET how to bind to the `JsonMergePatch` parameter.
For that, we will need to write a [custom model binder](https://docs.microsoft.com/en-us/aspnet/core/mvc/advanced/custom-model-binding?view=aspnetcore-2.2).
First we will implement an `IModelBinder`:

```csharp
public class JsonMergePatchModelBinder<T> : IModelBinder
{
    public async Task BindModelAsync(ModelBindingContext bindingContext)
    {
        if (bindingContext == null)
        {
            throw new ArgumentNullException(nameof(bindingContext));
        }

        var jsonDocument = await JsonDocument.ParseAsync(bindingContext.HttpContext.Request.Body);
        var model = JsonMergePatch<T>.Create(jsonDocument);
        bindingContext.Result = ModelBindingResult.Success(model);
    }
}
```

All this needs to do is create a `JsonMergePatch` instance from the the supplied `ModelBindingContext`.

Next, we need to write an `IModelBindingProvider`.  Its job will be to return our model binder whenever we find
a parameter that is of type `JsonMergePatch`.

```csharp
public class JsonMergePatchModelBindingProvider : IModelBinderProvider
{
    public IModelBinder GetBinder(ModelBinderProviderContext context)
    {
        if (context == null)
        {
            throw new ArgumentNullException(nameof(context));
        }

        if (context.Metadata.ModelType.GetTypeInfo().IsGenericType &&
            context.Metadata.ModelType.GetGenericTypeDefinition() == typeof(JsonMergePatch<>))
        {
            var typeArgs = context.Metadata.ModelType.GetGenericArguments();
            var closedGenericType = typeof(JsonMergePatchModelBinder<>).MakeGenericType(typeArgs);
            return (IModelBinder)Activator.CreateInstance(closedGenericType)!;
        }

        return null!;
    }
}
```

There is a bit of reflection going on here, but it is not terribly complicated.  Finally, we need to register the
model binding provider when our site starts up.  Somewhere inside of `ConfigureServices`, we need to add:

```csharp
services.AddControllers(config =>
{
    config.ModelBinderProviders.Insert(0, new JsonMergePatchModelBindingProvider());
});
```

And that should do it.  We can now create controller actions with `JsonMergePatch<T>` parameters, and use the
class to patch up our backend models.

### Wrapping up
All in all, I am pretty happy with how this turned out.  In not very much code, I was able to get a decent
JSON Merge Patch implementation working.  I have used this approach on a couple of different projects, and it
has served me well so far.

Potential enhancements might be to automatically apply the patch to some target object.  I am not a huge fan
of that idea however.  Typically, my controller actions will take some anemic DTO type, and the value I want to patch
will be a full domain model.  For instance, my `Person` domain model may not have a setter for `Address`, but
instead have an `UpdateAddress(Address)` that has some business logic in it.

```csharp
public class PersonDomainModel
{
    public Guid Identifier { get; }
    public string? Name { get; }
    public Address? Address { get; private set; }
    
    public void UpdateAddress(Address newAddress)
    {
        // perhaps some checks to validate the address
        if (newAddress != null && newAddress.State == null)
        {
            // just a silly rule, but hopefully you get the idea
            throw new InvalidOperationException("A person's address must have a State");
        }
        
        Address = newAddress;
        
        // maybe changing the address should also raise a [DomainEvent](https://docs.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/domain-events-design-implementation)        
        Raise(new AddressUpdated(Identifier, newAddress);
    }
}
```

With that in mind, it is easy enough to use the `JsonMergePatch` class as is to apply the correct behavior like so:

```csharp
public IActionResult BetterPatch(Guid personId, [FromBody]JsonMergePatch<Person> request)
{
    ...
    
    if (request.IsDefined(x => x.Address, out var address))
    {
        personDomainModel.UpdateAddress(address);
    }
    
    ...
}
```

You can view all the code [here](https://primarilysoftware.visualstudio.com/_git/JsonMergePatch).  I have a pre-release
[nuget package](https://www.nuget.org/packages/JsonMergePatch/1.0.0-CI-20190822-054746) published as well.  Feel free
to check it out, and reach out if you have any questions.

