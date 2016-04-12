---
layout: post_layout
title: 发布 Kotlin 库到 Maven/JCenter仓库
time: 2016年04月10日 星期日
location: 上海
pulished: true
excerpt_separator: ""
---

之前的文章 [Android Studio 发布项目到Maven/JCenter仓库](https://liungkejin.github.io/2016/03/27/Publish-AAR-jcenter.html),
将 Java 库发布到 Maven/JCenter 上.

但是今天我打算将我的 Kotlin 库发布上去的时候, 发现了一些问题. 因为 Kotlin 文件不能用 java 来生成 Javadoc,
导致了在执行 `bintrayUpload` 任务的时候 `javadocJar` `sourcesJar` 两个任务都不能 build 成功,
虽然我可以把这两个任务去掉， 没有 \*-sources.jar 和 \*-javadoc.jar 一样也是可以发布到 Maven 上的,
但是如果要发布到 JCenter 上就需要这两个文件了，如果没有则审核过不了

> JCenter hosts Java applications that follow the Maven convention.
In addition to the existing files, you will need a \*-sources.jar, and optionally a \*-javadoc.jar. Your files should be under a Maven path layout.
(see https://bintray.com/docs/usermanual/uploads/uploads_includingyourpackagesinjcenter.html)

>Once those files are added, we will be glad to include your package in JCenter.

然后我在 Kotlin 社区找了一下关于生成 Javadoc 的讨论，最后发现 Kotlin 的 KDoc 项目已经没有了,
但是 Kotlin 开发了 [Dokka](https://github.com/Kotlin/dokka) , Dokka 是 kotlin 的一个文档生成引擎.

> Dokka is a documentation engine for Kotlin, performing the same function as javadoc for Java. Just like Kotlin itself, Dokka fully supports mixed-language Java/Kotlin projects. It understands standard Javadoc comments in Java files and KDoc comments in Kotlin files, and can generate documentation in multiple formats including standard Javadoc, HTML and Markdown.
