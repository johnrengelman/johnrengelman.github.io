---
layout: post
title: "[Updated] Eclipse STS vs IntelliJ IDEA for Grails Development"
date: 2011-04-20T00:00:00-05:00
---

So, I have become increasingly frustrated with IntelliJ for doing Grails development. I know. Everyone swears by it being "THE" IDE for Grails work, but I just can't get on that boat. Not only do I find the UI a total cluster-fuck and it nearly impossible to find the information that I want on the screen when I want it, but it is also a serious resource hog.</p>

<!-- more -->

I have a brand new Sandy Bridge, Quad-Core Macbook Pro with 4 GB of RAM and I'm constantly fighting with IntelliJ when it's doing something. This is my machine on any given day: iTunes, Twitter Client, Skype Client, Chrome (w/ 4 open tabs), Firefox (maybe), and IntelliJ...that's it. That's all I have running. There are a couple background apps (Dropbox, TunnelBlick SSH) but that's really all that I have running...yet I still have to constantly wait for IntelliJ to do things...I get the stupid beach ball, wait-while-I-inexplicably-tie-up-4-core-and-4-GB-of-RAM-for-no-damn-reason icon. UGH. So frustrating. Not to mention, IntelliJ's inability to perform the simplest in task in Subversion without you having to walk away for 10 mins while it does it.

So I've decided on a little experiment. SpringSource recently released version 2.6.0-SR1 of their STS tool set. STS is build on top of the Eclipse...what I consider the premiere Java IDE. So I'm installing. And I'm going to see if I can get some work done in STS for the rest of the week. And next week, I'll come back to this post with some thoughts and a final decision on what I'm going to use.

## Initial Impressions

I installed STS without any problems on my machine, but quickly found that I needed some items to get up and running. Surprise, surprise. First, you need to install/activate the Grails support for STS. This is done from the STS Dashboard when the application first starts up. I should have remembered this since a co-worker wrote an article about STS/Grails not to long ago: [Start Using Eclipse with Grails Support via STS](http://www.objectpartners.com/2011/02/15/start-using-eclipse-with-grails-support-via-sts/).

Done. Next step, get my project from SVN. Fail. Apparently STS doesn't ship with Subclipse installed by default. Really not that big of a surprise, since Eclipse doesn't either. Just another annoyance.

## Development

After finally getting my project checked out and configuring some additional source folders so Eclipse could find all the Groovy classes, I got up a running. Yet there was one nagging issue I couldn't seem to resolve. Apparently STS applies Java Code checks to Groovy files. In most cases this is valid, since Groovy is built on top of Java. However, STS kept finding one error that really annoyed me (disclaimer: I know the following code example is actually bad code. It's not mine. I also know how to fix it, but I'm not going to since it's production code. Plus, it's valid in Groovy for whatever reason).

## End Game

It appears that STS still has some work to do with Groovy syntax support. Or maybe some way to adjust which Java Checks are applied to .groovy files. I'm still playing, but in the meantime, I've upgraded my machine to 8GB of RAM and IntelliJ is performing much better&hellip;crazy, that I need 8GB to get good performance, but oh well.
