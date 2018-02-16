title: How to set up a scaloid project from scratch
date: 2014-07-07 01:14:13
tags: Scala
---

TL;DR; It's hard to set up android development environment without the aid of IDE. And it's harder for scaloid.

1. Download standalone SDK

2. Download Android SDK tools/ SDK platform tools/ SDK build tools
  NB> f@ck GFW, I got around that bastard by modifying `/etc/hosts`.

3. Install sbt, reference for GFW: [here](http://freewind.me/blog/20140509/2619.html)

4. `android create project --target <target-id> --name scaloidApp --path <path>/scaloidApp --activity MainActivity --package com.example.scaloidapp`

5. Create a directory named project within your project and add the file project/plugins.sbt, in it, add the following line:
    addSbtPlugin("com.hanhuy.sbt" % "android-sdk-plugin" % "1.2.20")

6. Create project/build.properties and add the following line:

  sbt.version=0.12.4 # newer versions may be used instead

7. Create `build.sbt` in root directory([example](https://gist.github.com/pfn/5872691)). Remember to import `android.Keys._`

```scala
import android.Keys._

android.Plugin.androidBuild

name := "scaloidApp"

scalaVersion := "2.11.0"

platformTarget in Android := "android-20"

libraryDependencies += "org.scaloid" %% "scaloid" % "3.4-10"

```

> UPDATE: `scaloid-android-plugin` fixed the building =.=
