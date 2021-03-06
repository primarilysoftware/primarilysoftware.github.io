---
layout: post
title: "I Contributed to .NET Core, and You Can Too"
comments: true
description: "A short post about how easy it was to contribute a pull request to .NET core"
keywords: ".net, dotnet, core, open source, contribute"
---

My first job out of college was working as a .NET developer, and I have continued to work
with .NET ever since, going on 7 years.  While I wasn't around for the dark ages of .NET 1.0 or 1.1 (can you
imagine a world without generics or iterators?), I have been around long enough to see quite a bit evolve.  From
LINQ, to `dynamic`, to `async`, Microsoft has continued to push .NET forward from a technology perspective,
borrowing and pioneering new features and patterns for developing software.  That doesn't come as much of a
surprise through.  *Microsoft's business is software*.  They know a thing or two about how to write code.

What has been most surprising to me and many others about how .NET has evolved is Microsoft's recent pivot
toward open source.  *Microsoft's business is software*.  This is M$ we are talking about right?  The
anti-competitive monolith that has made bagillions of dollars selling its software?  Are they really going to turn
around and start giving away their source code?  I was very skeptical as I watched this all unfold.  Microsoft did
not seem to fit with the open source community.

![Hello fellow kids](https://primarilysoftware.github.io/downloads/2017-07-03-i-contributed-to-dotnet-core-and-you-can-too/hello-fellow-kids.gif)

But domino after domino has fallen.  From Node.js support in Windows, to open sourcing ASP.NET MVC, then all of
.NET, and even migrating to GitHub, the software development world we now live in feels very different than the
one I started in.  Something happened recently that really drove this change home for me.  I submitted
a pull request to the [dotnet/corefx repo on GitHub](https://github.com/dotnet/corefx).

### It started with a bug
It's a long story (one that I hope to tell in a future post).  The short version is we started getting reports of
issues connecting to our WCF backend from our desktop app.  I was able to trace the issues back to a change that
was made in a [.NET security update](https://portal.msrc.microsoft.com/en-US/security-guidance/advisory/CVE-2017-0248).
Because all the .NET framework code is open source now, and available on GitHub, not only was I able to see exactly
what caused the problem, but I was able to submit a [pull request with a fix](https://github.com/dotnet/corefx/pull/21320),
and someone actually paid attention to it!

I was very impressed by how quickly my PR was responded to, and how helpful and friendly everyone involved in the
process was.  A couple of suggestions were made by team members, which I was more than happy to implement, and before you
know it, my PR was accepted.

### A Brave New World
This whole incident drove home the fact that Microsoft isn't just spewing marketing speak when it comes to open source.
They are going all in.  I was able to see first hand how as an organization they are building processes around
developing software in the open, willing to take feedback, and even contributions from the community.

I am very excited to see how all this continues to evolve over the coming years.  As much as I have enjoyed being
a .NET developer for the last 7 years, I think things are only going to get better.

And, I have to admit, it feels kind of cool that I can now say I contributed to the .NET framework.  What's ever cooler
is you can too!  Anyone can! And that is the beauty of this whole open source thing.
