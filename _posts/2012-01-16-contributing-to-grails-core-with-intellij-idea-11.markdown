---
layout: post
title: "Contributing to Grails-Core With IntelliJ IDEA 11"
date: 2012-01-16T00:00:00-06:00
---

I've recently started to work on contributing patches to the Grails Framework project. I've been lucky enough to be working for a client that is ok with using the latest versions of Grails for its back-end applications, so I've been able to play around a lot with Grails 2.0.0 since it's release at the end of last year. However, this has resulted in me encountering a number of unexpected/irritating bugs that have sometimes prevented code from being migrated "as-is". So, I've decided to try and learn and contribute back by working on some bugs myself.

If you're looking to contribute yourself, start here [http://grails.org/Contribute.](http://grails.org/Contribute.) I didn't have any issues getting up and running with my own fork of grails-core. I did have to execute a `./gradlew compile` task though before I was able to get any test to execute...and don't worry, when this takes a while. There are **A LOT** of dependencies to download.

My next step was to start using IntelliJ 11 since it would allow me to quickly debug failing tests, so I again returned to the grails.org website and found the executing the `./gradlew idea` task will produce the necessary IntelliJ project files...Excellent. But this is where the problems started.

First, I had to open the Module Settings for `grails-script` and remove the entry under 'Source Folders'. This was causing IntelliJ to not save any edits I made to the module properties. It appears that this entry isn't needed anyway (and I haven't run into any issues with it yet).

Second, the project wouldn't compile because the following 2 classes couldn't be found:`com.springsource.loaded.ReloadEventProcessorPlugin` and `com.springsource.loaded.Plugins`. After some searching, I found the `springloaded-core-1.0.2.jar` in my Gradle User Home (`~/.gradle`) directory (look under `~/.gradle/caches/artifacts-7`, it maybe be something different then `artifact-7`). I copied this jar to my home directory, opened the Module Settings for `grails-core-grails-core` and added a Jar Dependency that pointed to the `springloaded-core-1.0.2.jar` in my home directory.

Third, the `grails-plugin-validation` module won't compile because it was missing a dependency to the `grails-web` module. Again, I opened the Module Settings for `grails-plugin-validation` and added a Module Dependency to `grails-web`.

Now, I can finally run and debug my tests and get working on some patches.
