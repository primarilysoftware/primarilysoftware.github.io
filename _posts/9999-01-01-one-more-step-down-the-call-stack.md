---
layout: post
title: "Try to Take One More Step Down the Call Stack"
comments: true
description: "Taking one more step down the call stack can make all the difference"
keywords: "general, software development, tips"
---

![It doesn't work.  I don't know why...](https://primarilysoftware.github.io/downloads/2017-09-04-one-more-step-down-the-call-stack/it-doesnt-work-i-dont-know-why.png)
[Source](https://neoteric.eu/what-do-programmers-hate)

Software is rarely a black box.  It can be easy to forget that though.  Modern software development involves so many libraries
and frameworks, building apps can feel more like assembling IKEA furniture than carpentry.  This can be a blessing and a curse.
All these libraries and frameworks can help make developers very productive.  But at a certain point, I find myself revering
these components as something more than they really are.  Every now and then, I need to remind myself that the code in these
libraries and frameworks is hardly different than the code I am writing in my own apps.

This mindset is esspecially useful when things start to go wrong.  Inevitably, one of these components is going to start
misbehaving.  It may be you have stepped on a bug.  Or oftentimes, the strange behavior you are seeing turns out to be
perfectly reasonable given a slightly deeper understanding of the system.

As a .NET developer, I have long admired Scott Hanselmann.  Many moons ago, he gave
[a talk about ASP.NET MVC 2](https://channel9.msdn.com/Blogs/matthijs/ASPNET-MVC-2-Basics-Introduction-by-Scott-Hanselman).
Ok, its 2017, why would I share a link about ASP.NET MVC 2?  Well starting around the 11 minute mark of that talk, he
shares some really brilliant insights about how digging a little deeper into a call stack can really broaden your understanding
of a system.  Parts that at first glance are "indistinguishable from magic", can actually be explained and reasoned about.  I highly
recommend watching that video, not for the MVC bits, but for how he demonstrates how to walk through a stack trace to peek inside
these libraries that we tend to view of as block boxes.

This insight is something that has really stuck with me, and helped me to become the developer that I am today.  I find myself
often thinking back to this video whenever I am having trouble working with a new library, or even when [facing a difficult
bug](http://blog.primarilysoftware.com/2017/a-bugs-tale/).  So, thank you Mr. Hanselman.  Hopefully, anyone who may happen to
stumble across this blog will be equally inspired by your words.

In my own career, I have found that being willing to take that extra step down the stack trace can do wonders.  Not only has
this willingness helped me to be a more productive developer, but it has also helped establish a reputation for being the guy
who can figure out the toughest problems.  Some of my peers at work are dumbfounded when I can take a look at a
problem they have been stuggling with for hours, and quickly diagnose the issue.  While I wish I had real super powers, the
truth is I have spent a lot of time investigating stack traces, and from that practice learned the libraries and frameworks
we use at work just a tiny bit more deeply than my peers.  That little bit of extra insight can make a big difference.

So the next time you are stuck battling a new library, or perplexed by a tough bug, try and take an extra step down the stack
trace.  It may just hold the answer you are looking for.
