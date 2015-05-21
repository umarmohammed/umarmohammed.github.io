---
layout: page
status: publish
published: true
title: Presentations
author:
  display_name: David Boike
  email: david.boike@gmail.com
  url: http://www.make-awesome.com
date: '2011-04-09 15:57:19 -0500'
date_gmt: '2011-04-09 20:57:19 -0500'
categories: []
tags: []
comments: true
---
## Distributed Application Development with NServiceBus

 **[Distributed Application Development with NServiceBus](/downloads/dad-nsb.zip) - Click here to download the presentation PowerPoint and source code.**

Building reliable, scalable, maintainable distributed applications is next to impossible without the right tools. [NServiceBus](http://www.nservicebus.com) is an open-source .NET framework that will help you to do just that.

This code-focused presentation covers the fundamentals of NServiceBus development, including one-way messaging, publish and subscribe, and implementing long-running business processes.

In the code example, we start with fairly simple code for a user to create an account on a website and expand it to include an email verification feature using NServiceBus.

In the first phase, we'll move the work of creating the user into the service layer, so that we can get an example of how NServiceBus works at its most fundamental level.

In the second phase, we will up the ante significantly, using a Saga (a long running business process) to implement an email verification check.  This includes sending an email with a verification code to the user, and then verifying that code before finally creating the user.

*Note: The SlideShare version does not include the animations, which demonstrate how the NServiceBus endpoint works in detail. For a fully functional version, please download the PowerPoint.*

**[Distributed Application Development with NServiceBus](http://www.slideshare.net/davidboike/distributed-application-development-with-nservicebus "Distributed Application Development with NServiceBus")**

View more [presentations](http://www.slideshare.net/) from [David Boike](http://www.slideshare.net/davidboike).

