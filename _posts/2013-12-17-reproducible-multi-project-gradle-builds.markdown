---
layout: post
title: "Reproducible Multi-Project Gradle Builds"
date: 2013-12-17T00:00:00-06:00
---
Lately I've been working on a way to create a reproducible build of my application source using Gradle. And by reproducible, I mean, when I run the md5 or sha256 checksum on the resulting files, I get the same results.

This is a bit more complicate then I originally thought. The issue is that inherently, a Java Jar file includes the timestamp information for the files contained within. So, if you build a Jar and checksum it and then clean and rebuilt the Jar, even though the files are semantically identical, the resulting checksum's will be different.

This is a problem in Continuous Integration/Deployment environments where you may be building from a clean VCS checkout every time but want to know only about the things that changed.

<!-- more -->

Working around this in Gradle is a bit of mind twister. After thinking about this for a couple days, I've come up with this list of items that need to addressed to successfully accomplish what I'm trying to do.

1.  Calculate a single timestamp for each artifact that Gradle produces
2.  Timestamp calculation needs to be sensitive to the Gradle project that produces the artifact.
3.  Timestamp calculation needs to be sensitive to any Gradle project dependencies for the project producing the artifact.
4.  Modify the resulting artifact to set the timestamp information for contents uniformly.

After a quick Twitter conversation with [Luke Daley](http://twitter.com/ldaley), item 4 was knocked out by using Java's `ZipFile` API to iterate over the jar's contents and set the `lastModified` timestamp to a value, but that leaves the more critical timestamp calculations.

Let's start with a mind experiment to figure out what we need. Let's assume we have the following Gradle project structure:

```
myApp
  |- utils
  |- app
```

This is a simple Gradle multi-project build where we have the application as one project and a utility library as another.

We check this code out and run a `./gradlew build` and calculate an MD5 checksum on `utils.jar` and `app.jar`:

| File | Checksum |
|:----:|:--------:|
| utils.jar | A |
| app.jar | B |

If I make a change to the `app` project and do a `./gradlew clean build`, we want the following to happen:

| File | Checksum |
|:----:|:--------:|
| utils.jar | A |
| app.jar | B' |

In this scenario, only the `app.jar` checksum would change because the `utils` project is functionally equivalent.

However, if we start from our initial commit and change the `utils` project and do a `./gradlew clean build`, we want the following to happen:

| File | Checksum |
|:----:|:--------:|
| utils.jar | A' |
| app.jar | B' |

In this case with only a change to `utils`, we want both checksums to change because the `app` project is dependent upon the changed code in the utils library.

My initial thought was to use my VCS to get a timestamp for the latest change to the Gradle project being built. This would couple my build to the VCS being used, but that's an okay dependency in my opinion. Alternatively, the build could scan over the source contents and get the latest `lastModified` timestamp of any file contained within.

Neither of these solutions would solve the issue with upstream libraries however. Somehow, we need to propagate the upstream project's information.

I'm not sure there is a _nice_ way to do this, but my current thinking is to simulate this meta information by enforcing a commit to downstream projects when an upstream is modified. In my particular case, I would likely change the project's version of the upstream library by modifying the `build.gradle` file in that project. I could add a task that verifies that the downstream project has a version that is _at least_ equivalent to any upstream projects' versions. This could run as part of the standard build process. This would force a change to the downstream's `build.gradle` thus indirectly updating the timestamp the VCS would report for that project.

All of this is theoretical at this point, I've only started working on an implementation. I'll update this post as I learn more.
