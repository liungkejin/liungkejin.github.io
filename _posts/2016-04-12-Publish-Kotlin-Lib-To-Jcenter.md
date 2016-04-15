---
layout: post_layout
title: 发布 Kotlin 库到 Maven/JCenter仓库
time: 2016年04月12日 星期日
location: 上海
pulished: true
excerpt_separator: "```"
---
>
> 这篇文章主要是记录我如果解决了这个发布问题, 具体的解决方案文件和教程我已经上传至 Github
> [https://github.com/liungkejin/GradlePublish](https://github.com/liungkejin/GradlePublish)
>

之前的文章 [Android Studio 发布项目到Maven/JCenter仓库](https://liungkejin.github.io/2016/03/27/Publish-AAR-jcenter.html),
将 Java 库发布到 Maven/JCenter 上.

但是今天我打算将我的 Kotlin 库发布上去的时候, 发现了一些问题. 因为 Kotlin 文件不能用 javadoc工具来生成 Javadoc,
导致了在执行 `bintrayUpload` 任务的时候 `javadocJar` `sourcesJar` 两个任务都不能 build 成功,
虽然我可以把这两个任务去掉， 没有 \*-sources.jar 和 \*-javadoc.jar 一样也是可以发布到 Maven 上的,
但是如果要发布到 JCenter 上就需要这两个文件了，如果没有则审核过不了

> JCenter hosts Java applications that follow the Maven convention.
In addition to the existing files, you will need a \*-sources.jar, and optionally a \*-javadoc.jar. Your files should be under a Maven path layout.
(see [https://bintray.com/docs/usermanual/uploads/uploads_includingyourpackagesinjcenter.html]())
>
>Once those files are added, we will be glad to include your package in JCenter.

然后我在 Kotlin 社区找了一下关于生成 Javadoc 的讨论，最后发现 Kotlin 的 KDoc 项目已经没有了,
但是 Kotlin 开发了 [Dokka](https://github.com/Kotlin/dokka) , Dokka 是 kotlin 的一个文档生成引擎.

> Dokka is a documentation engine for Kotlin, performing the same function as javadoc for Java. Just like Kotlin itself, Dokka fully supports mixed-language Java/Kotlin projects. It understands standard Javadoc comments in Java files and KDoc comments in Kotlin files, and can generate documentation in multiple formats including standard Javadoc, HTML and Markdown.

但是又很不幸的是, 目前 dokka 的 `dokka-android-gradle-plugin` 目前还没有 release, 而且 dokka 项目的 README
也都是坑, 虽然这个 Android gradle plugin 没有 release, 不过他已经在上面写上用法了, 也没说明有没有 release,
直至我在 bintray 上查看了他们的版本文件才知道确实没有这个 plugin...., 然后我试了一下使用 `dokka-gradle-plugin`,
但是我也没有弄成功, 可他的 README 上面明明就是这样写的

```groovy
dokka {
    moduleName = 'data'
    outputFormat = 'javadoc'
    outputDirectory = "$buildDir/javadoc"
    processConfigurations = ['compile', 'extra']
    includes = ['packages.md', 'extra.md']
    samples = ['samples/basic.kt', 'samples/advanced.kt']
    linkMapping {
        dir = "src/main/kotlin"
        url = "https://github.com/cy6erGn0m/vertx3-lang-kotlin/blob/master/src/main/kotlin"
        suffix = "#L"
    }
    sourceDirs = files('src/main/kotlin')
}
```

但是 gradle 却提示没有 sourceDirs 这个属性, 而且即使我把它注释掉, 最后又报错说 linkMapping 无法实例化!
然后我在他的源代码看, 发现确实是没有 sourceDirs 属性的.....

```java
open class DokkaTask : DefaultTask() {
    ....
    @Input
    var moduleName: String = ""
    @Input
    var outputFormat: String = "html"
    var outputDirectory: String = ""
    @Input
    var processConfigurations: List<Any?> = arrayListOf("compile")
    @Input
    var includes: List<Any?> = arrayListOf()
    @Input
    var linkMappings: ArrayList<LinkMapping> = arrayListOf()
    @Input
    var samples: List<Any?> = arrayListOf()
    @Input
    var jdkVersion: Int = 6

    fun linkMapping(closure: Closure<Any?>) {
        val mapping = LinkMapping()
        closure.delegate = mapping
        closure.call()

        if (mapping.dir.isEmpty()) {
            throw IllegalArgumentException("Link mapping should have dir")
        }
        if (mapping.url.isEmpty()) {
            throw IllegalArgumentException("Link mapping should have url")
        }

        linkMappings.add(mapping)
    }

    ....
}
```

最后没办法了, 只能用它的 [dokka-fatjar.jar](/assets/files/dokka-fatjar.jar)
(这个是我下载下来的, 0.9.7版本, 可以去 [Dokka](https://github.com/Kotlin/dokka)  下载最新版本) 了, 这回终于没有被坑了!

```bash
java -jar dokka-fatjar.jar <source directories> <arguments>

Dokka supports the following command line arguments:

    -output - the output directory where the documentation is generated
    -format - the output format:
        html - HTML (default)
        markdown - Markdown
        jekyll - Markdown adapted for Jekyll sites
        javadoc - Javadoc (showing how the project can be accessed from Java)
    -classpath - list of directories or .jar files to include in the classpath (used for resolving references)
    -samples - list of directories containing sample code (documentation for those directories is not generated but declarations from them can be referenced using the @sample tag)
    -module - the name of the module being documented (used as the root directory of the generated documentation)
    -include - names of files containing the documentation for the module and individual packages
    -nodeprecated - if set, deprecated elements are not included in the generated documentation
```

不过这里要注意一下, 当生成的格式为 javadoc 时, dokka-fatjar.jar 有引用到 `com.sun.javadoc` 包,
这个包在 `jdkX.X.X/lib/tools.jar` 里面, 所以如果你的 java CLASS_PATH 没有把这个路径包含进来的话,
需要你手动把这个路径包含进来, 否则就会报这个异常

```java
Caused by: java.lang.ClassNotFoundException: com.sun.javadoc.DocErrorReporter
        at java.net.URLClassLoader.findClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        ... 17 more
```

我是将 tools.jar 直接拷贝到了当前目录下, 所以最后我的这个命令为

```bash
java -Djava.ext.dirs=. -jar dokka-fatjar.jar exrecyclerview/src/main/kotlin/ -format javadoc -output exrecyclerview/build/doc
```

当这个命令成功生成了 javadoc 文档之后, 我就想既然命令能成功生成, 那用 gradle 来执行这条命令也是可以的,
最后再用 gradle 打包发布至 bintray 上 岂不是更方便些.

所以我将原来发布 java 库的gradle 文件改造了一下

```groovy
group = PROJ_GROUP
version = PROJ_VERSION
project.archivesBaseName = PROJ_ARTIFACTID

apply plugin: 'com.jfrog.bintray'
apply plugin: "com.jfrog.artifactory"
apply plugin: 'maven-publish'


/**
 * 这个就是执行命令的任务
 */
//"java -Djava.ext.dirs=. -jar dokka-fatjar.jar exrecyclerview/src/main/kotlin/ -format javadoc -output exrecyclerview/build/doc"
task dokkaJavadoc(type: org.gradle.api.tasks.Exec) {
    workingDir '.'
    println "WorkingDir: $workingDir"
    ext.toolJarPath = "$workingDir"
    ext.dokkaJarPath = "$workingDir"+File.separator+"dokka-fatjar.jar"
    ext.sourceDirs = "$workingDir"+File.separator+"src"+File.separator+"main"
    ext.outputFormat = 'javadoc'
    ext.outputDirectory = "$buildDir" + File.separator+ "javadoc"

    /**
     *
     * 这里输出格式可以为: html , markdown, jekyll, javadoc
     * 如果是 javadoc 格式, 他会用到 javadoc 的库,
     * 如果你的 PATH 没有包含 JDK x.x.x/lib 路径的话, 就会报 'java.lang.ClassNotFoundException: com.sun.javadoc.DocErrorReporter' 异常
     * 所以需要你主动将这个路径加进来, 或者将 JDK x.x.x/lib/tools.jar 文件拷贝出来, 下面这个命令我就是拷贝到了当前目录
     *
     * dokka-fatjar.jar 这个jar就是从 dokka 项目上下载下来的
     */
    commandLine "cmd" , "/c", "java -Djava.ext.dirs=$toolJarPath -jar $dokkaJarPath $sourceDirs -format $outputFormat -output $outputDirectory"

    /**
     * 如果你是 Linux 系统, 就用这个
     */
//    commandLine "java -Djava.ext.dirs=$toolJarPath -jar $dokkaJarPath $sourceDirs -format $outputFormat -output $outputDirectory"

}

/**
 * 这里将生成的文档打包成 xxxx-javadoc.jar
 */
task kotlinDocJar(type: Jar, dependsOn: dokkaJavadoc) {
    classifier = 'javadoc'
    from dokkaJavadoc.outputDirectory
}

/**
 * 这里将源码打包成 xxx-sources.jar
 */
task sourceJar(type: Jar) {
    classifier "sources"
    from android.sourceSets.main.java.srcDirs
}

def pomConfig = {
    licenses {
        /**
         * 这个License 可以自行修改
         */
        license {
            name "The Apache Software License, Version 2.0"
            url "http://www.apache.org/licenses/LICENSE-2.0.txt"
            distribution "repo"
        }
    }
    developers {
        developer {
            id DEVELOPER_ID
            name DEVELOPER_NAME
            email DEVELOPER_EMAIL
        }
    }
}

publishing {
    publications {
        mavenKotlin(MavenPublication) {
            artifact kotlinDocJar

            artifact sourceJar

            pom.withXml {
                def root = asNode()
                root.appendNode('description', PROJ_DESCRIPTION)
                root.children().last() + pomConfig

                def dependenciesNode = root.appendNode('dependencies')
                configurations.compile.allDependencies.each {
                    if (it.group && it.name && it.version) {
                        def dependencyNode = dependenciesNode.appendNode('dependency')
                        dependencyNode.appendNode('groupId', it.group)
                        dependencyNode.appendNode('artifactId', it.name)
                        dependencyNode.appendNode('version', it.version)
                    }
                }
            }
        }
    }
}

afterEvaluate {
    publishing.publications.mavenKotlin.artifact(bundleRelease)
}

bintray {
    Properties properties = new Properties()
    properties.load(project.rootProject.file('local.properties').newDataInputStream())
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")

    publications = ['mavenKotlin']
    publish = true

    pkg {
        repo = 'maven'
        name = PROJ_NAME
        desc = PROJ_DESCRIPTION
        websiteUrl = PROJ_WEBSITEURL
        issueTrackerUrl = PROJ_ISSUETRACKERURL
        vcsUrl = PROJ_VCSURL
        licenses = ['Apache-2.0']
        publicDownloadNumbers = true
    }
}


artifactory {
    contextUrl = 'http://oss.jfrog.org/artifactory'
    resolve {
        repository {
            repoKey = 'libs-release'
        }
    }
    publish {
        repository {
            repoKey = 'oss-snapshot-local' //The Artifactory repository key to publish to
            username = bintray.user
            password = bintray.key
            maven = true
        }
        defaults {
            //这里的名字和上面红色的名字一致即可，会将其包含的输出上传到jfrog上去
            publications('mavenKotlin')
            publishArtifacts = true
        }
    }
}
```

最后我成功了！ 成功生成了 xxx-sources.jar, xxxx-javadoc.jar

点击 `publishMavenKotlinPublicationMavenLocal` 任务, build 成功, 再执行 `bintrayUpload` 任务
成功将项目库发上传到了 jfrog bintray!

# Tips

在解决问题过程中的小知识点, 记录一下


- cmd命令行模式下，我们要运行一个java类，一般的方法是：

    ```
    java -classpath xxx.jar Test  
    ```

    引用多个 jar 文件


    ```
    java -Djava.ext.dirs=./lib Test
    ```

- gradle 运行外部命令

  ```groovy
  /**
   * 注意windows 和 linux 的路径分隔符的不同
   */
task execCommand(type:Exec) {
    workingDir './build'

    //on windows:
    commandLine 'cmd', '/c', 'stop.bat'

    //on linux
    commandLine './stop.sh'

    //store the output instead of printing to the console:
    standardOutput = new ByteArrayOutputStream()

    //extension method stopTomcat.output() can be used to obtain the output:
    ext.output = {
        return standardOutput.toString()
    }
}
  ```
