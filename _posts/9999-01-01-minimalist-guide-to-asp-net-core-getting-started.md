---
layout: post
title: "The Minimalist Guide to ASP.NET Core: Getting Started"
comments: true
description: "An minimalistic overview of how to setup a new ASP.NET Core app"
keywords: "asp.net core, asp.net core 2.1, csharp, web"
---

ASP.NET Core has really arrived.  With the release of ASP.NET Core 2.1, the framework has reached a level
of maturity and adoption that it cannot be overlooked when starting a new project.  It has been a bit of 
a bumpy road getting to this point.  The API surface for ASP.NET Core apps has evolved rapidly.  Early
adopters of ASP.NET Core were unfortunately caught in the shuffle, while the ASP.NET Core team refined
and improved the framework.  Now that 2.1 is out, it seems the framework is in a place where it is really
starting to stabalize.  So, I think this is the perfect time to kick the tires on Microsoft's new hot rod,
and take a closer look at what ASP.NET Core has to offer.

### The Minimalist Guide
Some developers are happy to follow whatever recipe a framework may give you
(File > New Project > ASP.NET Web Application > With MVC, With Identity Auth, With Bootstrap, With...).  While
there is nothing wrong with that approach, in fact that is often the right approach, that is not what this guide is
about.  In this guide, I want to take a minimalist approach to a variety of topics.  Cut out all the cruft,
and try and find the simplest possible way for implementing various features.  The hope is that exploring
ASP.NET Core in this way I (and you) will gain a better understanding of how the bits and bytes that make up
the framework fit together, leading to a better intuition for how to best use the framework to build web apps.
