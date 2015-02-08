---
layout: post
title: "Grails-Gradle Plugin 2.0.0 Released"
date: 2014-01-27T00:00:00-06:00
---

I am happy to announce that the 2.0.0 release for the Grails-Gradle plugin has finally been released.

This release has been a long time coming, and offers some significant improvements over the 1.1.0 release. Additionally, many people have been relying on the 2.0.0-SNAPSHOT release and this locks-in a stable version that is now available in Maven Central and the Grails Central repositories.

**ChangeLog**

* Automatic configuration of Grails repositories using `grails.central()` in your `repositories` block
* Automatic configuration of Grails, Groovy, and Spring Loaded versions
* Implementation of a build task structure more similar to a standard Java project (i.e. clean, build, assemble, test, check ,etc.)
* Better integration with IntelliJ IDEA (plugin source is now available)
* Definition of Grails source sets to allow for task input/output configuration
* Various bug fixes and improvements in Grails command spawning.

Additional information can be found on the plugin's Github [page](https://github.com/grails/grails-gradle-plugin).
