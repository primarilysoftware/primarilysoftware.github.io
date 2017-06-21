---
layout: post
title: "C# Value Objects, Let Me Count the Ways"
comments: true
description: "A brief exploration of the various ways to implement value objects in C#"
keywords: "code snippet, il weaving, aop, value object, ddd, c#, visual studio, .net"
---

Value objects are a good thing.  I find myself using them more and more to harded the code I write, moving away from Primitive Obsession.  The value of Value Objects has already been covered quite thoroughly by much more [credible](https://lostechies.com/jimmybogard/2007/12/03/dealing-with-primitive-obsession/) [sources](http://blog.ploeh.dk/2011/05/25/DesignSmellPrimitiveObsession/) [than myself](http://enterprisecraftsmanship.com/2015/03/07/functional-c-primitive-obsession/), so I do not feel the need to expand on why you may want to use Value Objects.

What I do want to share are a few ways to make using Value Objects a little less painful in C# (at least until we get [record types](https://github.com/dotnet/csharplang/blob/master/proposals/records.md)).

### Generic Base Class
A lot of guidance out there leads you down the path of creating a generic base type for Value Objects, and that cat has been [skinned](http://haacked.com/archive/2012/09/30/primitive-obsession-custom-string-types-and-self-referencing-generic-constraints.aspx/) [many](https://lostechies.com/jimmybogard/2007/06/25/generic-value-object-equality/) [times](https://elegantcode.com/2009/06/07/generic-value-object/).  My favorite varient of this approach looks like this (inspired by [this post](http://enterprisecraftsmanship.com/2017/06/15/value-objects-when-to-create-one/)):

```csharp
public abstract class ValueObject<T>
    where T : ValueObject<T>
{
    public bool Equals(T other)
    {
        if (ReferenceEquals(other, null))
        {
            return false;
        }

        return this.EqualsCore(other);
    }

    public override bool Equals(object obj)
    {
        return this.Equals(obj as T);
    }

    public override int GetHashCode()
    {
        return base.GetHashCode();
    }

    public static bool operator ==(ValueObject<T> a, ValueObject<T> b)
    {
        if (ReferenceEquals(a, null) && ReferenceEquals(b, null))
        {
            return true;
        }

        if (ReferenceEquals(a, null) || ReferenceEquals(b, null))
        {
            return false;
        }

        return a.Equals(b);
    }

    public static bool operator !=(ValueObject<T> a, ValueObject<T> b)
    {
        return !(a == b);
    }

    protected abstract bool EqualsCore(T other);
    protected abstract int GetHashCodeCore();
}
```

To use this base class you would end up writing something like this:

```csharp
public class ZipCode : ValueObject<ZipCode>
{
    public ZipCode(string value)
    {
        this.Value = value;
    }

    public string Value { get; }

    protected override bool EqualsCore(ZipCode other)
    {
        return this.Value == other?.Value;
    }

    protected override int GetHashCodeCore()
    {
        return this.Value?.GetHashCode() ?? 0;
    }
}
```

While this approach makes writing new value objects easier, it still bugged me for a couple of reasons:
1. The self referencing generic constraint, `where T : ValueObject<T>`, [hurts my brain](https://blogs.msdn.microsoft.com/ericlippert/2011/02/03/curiouser-and-curiouser/)
2. That is still more code than I would like to write

Can we do better? With a base class, I think not.  So what other options are there?

### IL Weaving

If the term IL Weaving is new to you, briefly, it is the process of manipulating the IL of an assembly generated by a build to inject new behaviors.  One use case for IL Weaving is to inject code for the equality behaviors we expect of our value objects.  I know of 2 prominent IL Weavers in the .NET space, [Fody](https://github.com/Fody/Fody) and [PostSharp](https://www.postsharp.net/aspects).

Personally, I have more experience with Fody (it is free and open source).  Fody is built around an add-in model.  Fody itself is really a framework for writing IL weavers, where specific types of weavers are implemented as add-ins.  One such add-in is [Equals.Fody](https://github.com/Fody/Equals).  It is surprisingly simple to start using Fody in your projects, so simple that it almost feels like magic.  To use the Equals add-in, just install the [Equals.Fody Nuget package](https://www.nuget.org/packages/Equals.Fody/).  Then, when you want a class to implement value equality, just decorate it with the `[Equals]` attribute like so:

```csharp
[Equals]
public class ZipCode
{
    public ZipCode(string value)
    {
        this.Value = value;
    }

    public string Value { get; }
}
```

Thats really all there is to it.  After building, Fody will scan the assembly generated by the build looking for types decorated with `[Equals]`.  For any types it finds, it will inject the implementation for `IEquatable<>`, as well as override `GetHashCode()`, `Equals(object obj)`, `==`, and `!=` (you can reference the [docs](https://github.com/Fody/Equals) for more details on exactly what gets generated).

So whats the catch?  Well not much from my experience.  The concept of IL weaving is a bit scary to some people, so you may have some difficulty convincing the rest of your team to buy in.  The other hang up at the moment is .NET core support is still a work in progress.

Maybe an IL Weaver is a bridge too far.  Are there any other good choices?

### Code Snippets
I love code snippets.  I use the ones that ship with Visual Studio all the time.  However, I often forget how easy it is to create your own snippets.  My favorite way to create snippets is using [Snippet Designer](https://github.com/mmanela/SnippetDesigner).  This Visual Studio plugin provides a nice editor for creating new code snippets, giving you some basic syntax highlighting and helping you manage the replacements in your snippet.

Since so much of creating a value object is boilerplate, I think a code snippet is a great option for simplifiying its creation.  Here is the meat of the snippet I use (note that things wrapped in $$ are replacements):

```csharp
public class $ValueTypeName$ : IEquatable<$ValueTypeName$>
{
    public $ValueTypeName$($PropertyType$ $PropertyName$)
    {
        this.$PropertyName$ = $PropertyName$;
    }

    public $PropertyType$ $PropertyName$ { get; }

    public override int GetHashCode()
    {
        return this.$PropertyName$?.GetHashCode() ?? 0;
    }

    public override bool Equals(object obj)
    {
        return this.Equals(obj as $ValueTypeName$);
    }

    public bool Equals($ValueTypeName$ obj)
    {
        if (ReferenceEquals(obj, null))
        {
            return false;
        }

        return obj.Value == this.Value;
    }

    public static bool operator ==($ValueTypeName$ a, $ValueTypeName$ b)
    {
        if (ReferenceEquals(a, null) && ReferenceEquals(b, null))
        {
            return true;
        }

        if (ReferenceEquals(a, null) || ReferenceEquals(b, null))
        {
            return false;
        }

        return a.Equals(b);
    }

    public static bool operator !=($ValueTypeName$ a, $ValueTypeName$ b)
    {
        return !(a == b);
    }
}
```

> Here is a link to download the full code snippet, just save that to `%USERPROFILE%\Documents\Visual Studio 2017\Code Snippets\Visual C#\My Code Snippets`.

To use it, type the shortcut `valueo`, specify a couple options, and you are in business.  Now this particular snippet will only work for value objects that have a single backing field.  You may find yourself needing to create value objects with multiple backing fields.  For example, something like a `PersonName` value object, that consists of both a `FirstName` and a `LastName`.  Here is a similar snippet that will generate a value object with 2 backing fields:

```csharp
public class $ValueTypeName$ : IEquatable<$ValueTypeName$>
{
    public $ValueTypeName$($PropertyType1$ $PropertyName1$, $PropertyType2$ $PropertyName2$)
    {
        this.$PropertyName1$ = $PropertyName1$;
        this.$PropertyName2$ = $PropertyName2$;
    }

    public $PropertyType1$ $PropertyName1$ { get; }
    public $PropertyType2$ $PropertyName2$ { get; }

    public override int GetHashCode()
    {
        unchecked
        {
            var hash = 17;
            hash = hash * 23 + this.$PropertyName1$?.GetHashCode() ?? 0;
            hash = hash * 23 + this.$PropertyName2$?.GetHashCode() ?? 0;            
            return hash;
        }
    }

    public override bool Equals(object obj)
    {
        return this.Equals(obj as $ValueTypeName$);
    }

    public bool Equals($ValueTypeName$ obj)
    {
        if (ReferenceEquals(obj, null))
        {
            return false;
        }

        return obj.$PropertyName1$ == this.$PropertyName1$ &&
            obj.$PropertyName2$ == this.$PropertyName2$;
    }

    public static bool operator ==($ValueTypeName$ a, $ValueTypeName$ b)
    {
        if (ReferenceEquals(a, null) && ReferenceEquals(b, null))
        {
            return true;
        }

        if (ReferenceEquals(a, null) || ReferenceEquals(b, null))
        {
            return false;
        }

        return a.Equals(b);
    }

    public static bool operator !=($ValueTypeName$ a, $ValueTypeName$ b)
    {
        return !(a == b);
    }
}
```
> Again, here is a link to download the full code snippet, just save that to `%USERPROFILE%\Documents\Visual Studio 2017\Code Snippets\Visual C#\My Code Snippets`.

Of note is the implementation of `GetHashCode()` (inspired by [Jon Skeet](https://stackoverflow.com/questions/263400/what-is-the-best-algorithm-for-an-overridden-system-object-gethashcode)) which must take into consideration both backing values to generate a good hash code.  I use this snippet with the shortcut `valueo2`.  I think you can see how you could continue this pattern for creating snippets up to a sufficiently large `n`.

### Wrapping Up
In practice, I have grown particularly fond of the Code Snippet approach to creating value objects.  It is very easy to tweak as your taste for how value objects should be implemented evolves.  Also, being a code snippet, once it has been setup on your machine, you can use it in any project you may be working on.  With the other approaches I mentioned, you need to copy and paste your base class implementation around, or add some additional dependencies to your projects.

Well thats my 2 cents at least, would love to hear other peoples thoughts!