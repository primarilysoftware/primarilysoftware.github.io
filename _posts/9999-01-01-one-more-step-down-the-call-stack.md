---
layout: post
title: "Try to Take One More Step Down the Call Stack"
comments: true
description: "Taking one more step down the call stack can make all the difference"
keywords: "general, software development, tips"
---

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
recommend watching that video, not for the MVC bits, but how he demonstrates how to walk through a stack trace to peek inside
these libraries that we tend to view of as block boxes.

This is something that really stuck with me as a young developer, and
something that I have carried with me to this day.  This particular bug is just another reminder of how powerful Scott's notion of
taking one more step down the call stack is.  Prior to working this bug, SSL/TLS was mostly magic to me.  I was aware that
there were a variety of different protocols, and trusted that some smart people kept these things secure.  In my .NET apps,
I just give an https URL to a WCF configuration, or `WebClient`, and magic happens.  Having gone through this ordeal, I have
a new appreciation for how all the pieces to a secure TCP connection come together.
2. Open source software is awesome.  I have long used many different open source projects, but this incident has given me a
new found appreciation for the whole open source movement.  I love that Microsoft and .NET are now on board, and cannot wait
to see where this new path leads.
