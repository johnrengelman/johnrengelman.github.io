---
layout: post
title: "Reproducible Multi-Project Gradle Builds (Part 2)"
date: 2013-12-20T00:00:00-06:00
---
Follow up to my previous post - [Reproducible Multi-Project Gradle builds]({% post_url 2013-12-17-reproducible-multi-project-gradle-builds %})

I got around to doing some testing on this using a simple Gradle project. Very quickly did I find out that this particular strategy was not going to be easy...and potentially impossible.

## Problem 1 - Setting Timestamp value on Jar Entry's

The first problem I ran into was actually setting the timestamp value of a Jar Entry. My first attempt looked something like this:

```groovy
jar {
  doLast {
    long ts = fileTree('.') {
      exclude 'build/**'
    }.files.sort {
      -(it.lastModified())
    }.first().lastModified()
    JarFile jf = new JarFile(archivePath)
    jf.entries().each { entry ->
      entry.time = ts
    }
    jf.close()
  }
}
```

Yup. That didn’t work. Jar’s are more or less read-only. So in order to change the time stamp, I needed to write out a new Jar file with the modified entries. Basically, create a new jar file, iterate over the entries from the first jar file, clone them, modified the time stamp, add the cloned entry to the new jar file and then binary copy the data from the first jar file to the second. It looks something like this:

<!-- more -->

```groovy
jar {
  doLast {
    long ts = fileTree('.') {
      exclude 'build/**'
    }.files.sort {
      -(it.lastModified())
    }.first().lastModified()
    File newJar = new File(archivePath.parent, 'new-' + archiveName)
    JarOutputStream jos = new JarOutputStream(new FileOutputStream(newJar))
    JarFile jf = new JarFile(archivePath)
    jf.entries().each { entry ->
      cloneAndCopyEntry(jf, entry, jos, ts)
    }
    jos.finish()
    jf.close()
  }
}

void cloneAndCopyEntry(JarFile originalFile, JarEntry original, JarOutputStream jos, long newTimestamp) {
  JarEntry clone = new JarEntry(original)
  clone.time = newTimestamp
  def entryIs = originalFile.getInputStream(original)
  jos.putNextEntry(clone)
  copyBinaryData(entryIs, jos)
}

void copyBinaryData(InputStream is, JarOutputStream jos) {
  byte[] buffer = new byte[1024]
  int len = 0
  while((len = is.read(buffer)) != -1) {
    jos.write(buffer, 0, len)
  }
}
```

## Problem 2 - JAR “Magic” Extra Byte

The next issue I encountered was that my new Jar file always had an entry that didn’t match the original Jar for it’s ‘extra’ property. `JarEntry.geExtra()` returns a `byte[]`. In the orignal file, this was null, but in my copied file it was set to some data. I finally found that it is from the implementation of [JarOutputStream](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7-b147/java/util/jar/JarOutputStream.java) in Java.

This Jar MAGIC byte gets added to the first entry in the Jar file. I haven’t been able to find any documentation on what it is for, but a friend thought it was likely for the Jar tool itself to determine if an archive file is actually a Jar.

Curiously, Jar files produced by the Gradle Jar task, do **NOT** have this magic byte. Digging into their code, I found that they use a `ZipOutputStream` to write out the Jar file which doesn’t have this magic byte code. Using `ZipOutputStream` and `ZipEntry` works just as fine and avoids this, so we update the build to do the same:

```groovy
jar {
  doLast {
    long ts = fileTree('.') {
      exclude 'build/**'
    }.files.sort {
      -(it.lastModified())
    }.first().lastModified()
    File newJar = new File(archivePath.parent, 'new-' + archiveName)
    ZipOutputStream zos = new ZipOutputStream(new FileOutputStream(newJar))
    JarFile jf = new JarFile(archivePath)
    jf.entries().each { entry ->
      cloneAndCopyEntry(jf, entry, zos, ts)
    }
    zos.finish()
    jf.close()
  }
}

void cloneAndCopyEntry(JarFile originalFile, JarEntry original, ZipOutputStream zos, long newTimestamp) {
  ZipEntry clone = new ZipEntry(original)
  clone.time = newTimestamp
  def entryIs = originalFile.getInputStream(original)
  zos.putNextEntry(clone)
  copyBinaryData(entryIs, zos)
}

void copyBinaryData(InputStream is, ZipOutputStream zos) {
  byte[] buffer = new byte[1024]
  int len = 0
  while((len = is.read(buffer)) != -1) {
    zos.write(buffer, 0, len)
  }
}
```

## Problem 3 - Zip timestamp spec

After all this, I still wasn’t getting subsequent builds of the Jar to have matching checksums. I wrote a method that at the end of producing the timestamp adjusted Jar iterated over all the entries and compared all the fields to the original except the time field which I compared to the expected time stamp. Looks like this:

```groovy
jar {
  doLast {
    long ts = fileTree('.') {
      exclude 'build/**'
    }.files.sort {
      -(it.lastModified())
    }.first().lastModified()
    File newJar = new File(archivePath.parent, 'new-' + archiveName)
    ZipOutputStream zos = new ZipOutputStream(new FileOutputStream(newJar))
    JarFile jf = new JarFile(archivePath)
    jf.entries().each { entry ->
      cloneAndCopyEntry(jf, entry, zos, ts)
    }
    zos.finish()
    jf.close()
    compareJars(archivePath, newJar, ts)
  }
}

void compareJars(File original, File copy, long ts) {
  def jf = new JarFile(original)
  def cjf = new JarFile(copy)
  jf.entries().each { entry ->
    def centry = cjf.getJarEntry(entry.name)
    compareEntries(entry, centry, ts)
  }
}

void compareEntries(JarEntry entry, JarEntry centry, long ts) {
  assert entry.name == centry.name
  assert entry.comment == centry.comment
  assert entry.compressedSize == centry.compressedSize
  assert entry.crc == centry.crc
  assert entry.extra == centry.extra
  assert entry.method == centry.method
  assert entry.size == centry.size
  assert ts == centry.time
  assert entry.hashCode() == centry.hashCode()
}

void cloneAndCopyEntry(JarFile originalFile, JarEntry original, ZipOutputStream zos, long newTimestamp) {
  ZipEntry clone = new ZipEntry(original)
  clone.time = newTimestamp
  def entryIs = originalFile.getInputStream(original)
  zos.putNextEntry(clone)
  copyBinaryData(entryIs, zos)
}

void copyBinaryData(InputStream is, ZipOutputStream zos) {
  byte[] buffer = new byte[1024]
  int len = 0
  while((len = is.read(buffer)) != -1) {
    zos.write(buffer, 0, len)
  }
}
```

This showed me that some entries still didn’t have the same timestamp. Curiously, they were always off by 1 second (1000 milliseconds in the script). Digging back into the Java source code for [ZipEntry](http://grepcode.com/file/repository.grepcode.com/java/root/jdk/openjdk/7-b147/java/util/zip/ZipEntry.java#ZipEntry.setTime%28long%29), I found it was doing some sort of unix - DOS conversion and was losing the resolution of the time.

After a call out to the Twitterverse, a friend pointed out that the [Zip Spec](http://ant.apache.org/manual/Tasks/zip.html) specifies that Zip entries have a time resolution of 2 seconds:

> The `update` parameter controls what happens if the ZIP file already exists. When set to `yes`, the ZIP file is updated with the files specified. (New files are added; old files are replaced with the new versions.) When set to `no` (the default) the ZIP file is overwritten if any of the files that would be added to the archive are newer than the entries inside the archive. Please note that ZIP files store file modification times with a granularity of two seconds. If a file is less than two seconds newer than the entry in the archive, Apache Ant will not consider it newer.

So, we need to mimic this same behavior when producing our new Jar. Basically, we need to convert our timestamp to a resolution of 2 seconds. It looks like this:

```groovy
jar {
  doLast {
    long ts = fileTree('.') {
      exclude 'build/**'
    }.files.sort {
      -(it.lastModified())
    }.first().lastModified()
    File newJar = new File(archivePath.parent, 'new-' + archiveName)
    ZipOutputStream zos = new ZipOutputStream(new FileOutputStream(newJar))
    JarFile jf = new JarFile(archivePath)
    jf.entries().each { entry ->
      long adjustedTs = convertTimestamp(ts)
      cloneAndCopyEntry(jf, entry, zos, adjustedTs)
    }
    zos.finish()
    jf.close()
    compareJars(archivePath, newJar, ts)
  }
}

long convertTimestamp(long timestamp) {
  long seconds = timestamp / 1000
  if (seconds % 2 != 0) {
    seconds += 1
  }
  return 1000 * seconds
}

void compareJars(File original, File copy, long ts) {
  def jf = new JarFile(original)
  def cjf = new JarFile(copy)
  long adjustedTs = convertTimestamp(ts)
  jf.entries().each { entry ->
    def centry = cjf.getJarEntry(entry.name)
    compareEntries(entry, centry, adjustedTs)
  }
}

void compareEntries(JarEntry entry, JarEntry centry, long ts) {
  assert entry.name == centry.name
  assert entry.comment == centry.comment
  assert entry.compressedSize == centry.compressedSize
  assert entry.crc == centry.crc
  assert entry.extra == centry.extra
  assert entry.method == centry.method
  assert entry.size == centry.size
  assert ts == centry.time
  assert entry.hashCode() == centry.hashCode()
}

void cloneAndCopyEntry(JarFile originalFile, JarEntry original, ZipOutputStream zos, long newTimestamp) {
  ZipEntry clone = new ZipEntry(original)
  clone.time = newTimestamp
  def entryIs = originalFile.getInputStream(original)
  zos.putNextEntry(clone)
  copyBinaryData(entryIs, zos)
}

void copyBinaryData(InputStream is, ZipOutputStream zos) {
  byte[] buffer = new byte[1024]
  int len = 0
  while((len = is.read(buffer)) != -1) {
    zos.write(buffer, 0, len)
  }
}
```

## Problem 4 - Groovy `_timestamp` static field

This got my Jars closer, but they still weren’t checksumming the same. At this point, I started using a hex viewer to compare the two files. I used [Hex Fiend](http://ridiculousfish.com/hexfiend/) because it can do a side by side diff of files and is free

Using a Hex viewer on a Jar file doesn’t do much since the data is compressed, but it can get you pointed in the right direction. In my cause I could see some byte differences around what appeared to be some class declarations

![Hex View 1]({{ root_url }}/assets/hex1.png)

Next step was to explode the Jars and compare each of the files. This should a pretty apparent difference

![Hex View 2]({{ root_url }}/assets/hex2.png)

There was a binary difference in the class file produced by subsequent compilations. The difference was related to a filed named `__timestamp_239_neverHappen1387492963633`

That’s an interesting field, because there’s nothing like that in my source file:

```groovy
package john.app

import john.lib.EchoUtil

class EchoApp {

  public static void main(String[] args) {
    args.each {
      EchoUtil.echo(it)
    }
  }
}
```

Opening the `.class` file with a Java Decompiler (JD-GUI in this case), we see this for the class:

```java
package john.app;

import groovy.lang.Closure;
import groovy.lang.GroovyObject;
import groovy.lang.MetaClass;
import john.lib.EchoUtil;
import org.codehaus.groovy.runtime.GeneratedClosure;
import org.codehaus.groovy.runtime.callsite.CallSite;

public class EchoApp
  implements GroovyObject
{
  public EchoApp()
  {
    EchoApp this;
    CallSite[] arrayOfCallSite = $getCallSiteArray();
    MetaClass localMetaClass = $getStaticMetaClass();
    this.metaClass = localMetaClass;
  }

  public static void main(String[] args)
  {
    CallSite[] arrayOfCallSite = $getCallSiteArray(); arrayOfCallSite[0].call(args, new _main_closure1(EchoApp.class)); }
  static { __$swapInit();
    long l1 = 0L;
    __timeStamp__239_neverHappen1387493347537 = l1;
    long l2 = 1387493347537L; }
  class _main_closure1 extends Closure implements GeneratedClosure { public _main_closure1(Object _thisObject) { super(_thisObject); }
    public Object doCall(Object it) { CallSite[] arrayOfCallSite = $getCallSiteArray(); return arrayOfCallSite[0].call(EchoUtil.class, it);
    }

    public Object doCall()
    {
      CallSite[] arrayOfCallSite = $getCallSiteArray();
      return doCall(null);
    }

    static
    {
      __$swapInit();
    }
  }
}
```

There is in fact a private variable being initialized. Opening the other copy of this class revealed that the name of this variable was changing:

```
__timeStamp__239_neverHappen1387493347537
```

vs.

```
__timeStamp__239_neverHappen1387492963633
```

Hmmm. That's odd. A little more digging lead me to this [Groovy-6308](https://jira.codehaus.org/browse/GROOVY-6308) - Timestamps in bytecode prevents baselining of code. The title of this bug is "Timestamps in bytecode prevents baselining of code". Yup, this seems to be my problem.

It appears this field is injected as some sort of addition to `SerialVersionUUID`, though the corresponding conversations of that bug and the related bugs seems to indicate that it's not really used for anything.

Unfortunately, the bugs are listed as being targeted for Groovy 3.0 which doesn't help me in the near term.

At this point, since it's a compiler function that's stopping me, I'm not sure I have a path forward. This will probably get back burnered for a while since it's not critical and more of a thought experiment than anything.
