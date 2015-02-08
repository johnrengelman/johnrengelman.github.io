---
layout: post
title: "Creating Self-contained, Executable Jars With Gradle and Shadow"
date: 2013-10-08T13:43:54-05:00
---
I originally posted this over on [objectpartners.com](http://www.objectpartners.com) (2013/07/16)

Deploying Java applications that are not hosted in an application container (such as a WAR or EAR) can be a pain due to classpath dependencies. This is especially true when using a micro-service framework such as Dropwizard. Using Gradle and the Shadow plugin, applications can be packaged up into a single jar and distributed to your production environments with one file.
