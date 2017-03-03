---
layout: post
title: Introduction to NServiceBus
date: 2017-03-06
comments: true
categories:
- Development
tags:
- NServiceBus
- book
- career
- announcements
---

If you're looking at my blog right now (hint: you are) then you might already know that I am the author of [*Learning NServiceBus*](https://www.packtpub.com/application-development/learning-nservicebus) which focused on NServiceBus 4, and [*Learning NServiceBus - Second Edition*](https://www.packtpub.com/application-development/learning-nservicebus-second-edition) which focused on NServiceBus 5.

Now NServiceBus 6 has been released, and with the change to a fully async API, quite a bit has changed. In fact, enough has changed that if you tried to use my latest book, which is called *Learning NServiceBus*, to actually, you know, *learn NServiceBUs*, it might not be the greatest experience in the world.

Now that I work for the [makers of NServiceBus](https://particular.net), we've decided to go in a bit of a different direction. I've been working on a new [Introduction to NServiceBus](https://docs.particular.net/tutorials/intro-to-nservicebus/) tutorial, hosted within our documentation site.

<!-- more -->

In 5 lessons, each designed to be accomplished in a half hour or less, you'll learn the basics of working with NServiceBus. Of course this includes the techical aspecs like how to use the API to send messages, but also contains the messaging theory to help you get your NServiceBus projects off on the right foot.

Each lesson contains an exercise that guides you through each step of creating a sample messaging system in the retail/ecommerce domain. By the end of the tutorial, you'll have 4 messaging endpoints all exchanging messages with each other, and also learn how to use the tools in the Particular Service Platform to replay messages when the system suffers from temporary failures.

The first tutorial is introductory, and introduces the concepts of messaging, sending messages, and using Publish/Subscribe. As it is introductory, there's a bit of code duplication. I would love to build additional tutorials in the future that will extend on this, showing how to organize NServiceBus projects, centralize repeated configuration, and show how to get the most out of NServiceBus Sagas.

So if you're interested, give the [Introduction to NServiceBus](https://docs.particular.net/tutorials/intro-to-nservicebus/) tutorial a try, and then ping me on Twitter [@DavidBoike](https://twitter.com/DavidBoike) to let me know what you think.
