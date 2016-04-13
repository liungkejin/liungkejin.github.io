---
layout: post_layout
title: Android Studio 发布Java项目到Maven/JCenter仓库
time: 2016年03月27日 星期天
location: 上海
pulished: true
excerpt_separator: "##"
---

为了方便自己和他人引用我所写的java库，发布到 Maven/JCenter 上是最好不过了，利己利人，本文主要参考了

[使用Gradle发布项目到JCenter仓库](http://blog.csdn.net/maosidiaoxian/article/details/43148643)

[Git项目: Gradle Publish](https://github.com/msdx/gradle-publish)

虽然这篇已经很详细的，但是我在实际操作中还是碰到了一些问题，所以自己再整理一遍，方便以后查阅

## 前言(MavenCentral和JCenter的区别)

- [maven中央仓库](http://repo1.maven.org/maven2/) 是由Sonatype公司提供的服务，
它是Apache Maven、SBT和其他构建系统的默认仓库，并能很容易被Apache Ant/Ivy、Gradle和其他工具所使用。
开源组织例如Apache软件基金会、Eclipse基金会、JBoss和很多个人开源项目都将构件发布到中央仓库。
maven中央仓库已经将内容浏览功能禁掉了，可在 http://search.maven.org/ 查询构件。


- [JCenter](https://jcenter.bintray.com) 是由JFrog公司提供的Bintray中的Java仓库。
它是当前世界上最大的Java和Android开源软件构件仓库。 所有内容都通过内容分发网络（CDN）使用加密https连接获取。
JCenter是Goovy Grape内的默认仓库，Gradle内建支持（jcenter()仓库），非常易于在（可能除了Maven之外的）其他构建工具内进行配置。

## 第一步

*[注意：你的工程必须为 library, 即："apply plugin: 'com.android.library'", 否则后面的 bintray.gradle 会 build 失败]*

第一步当然是注册 [bintray](https://bintray.com) 的账号, 得到 user 和 apiKey,
apiKey 在你的 profile 里面，编辑你的 Profile，就能看到 API Key,
把你的 user 和 apiKey, 写入到工程目录下的 local.properties 文件中

```
project/
    \_ local.properties
```

写入

```groovy
bintray.user=XXX
bintray.apikey=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

因为 apiKey 是一个非常隐私的东西，所以写在 local.properties 文件里面，避免和工程一起上传到VCS中，
因为一般 local.properties 文件是被写在 .gitignore 里面的，不过如果使用其他的 VCS， 要注意隐藏这个文件

## 第二步

在你的 Module 目录下新建一个 bintray.gradle


```
project/
    \_ module/
        \_ bintray.gradle
```

然后在这个 Module 的 build.gradle 里面追加下面代码

```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.2'
        classpath "org.jfrog.buildinfo:build-info-extractor-gradle:3.1.1"
    }
}

apply from: 'bintray.gradle'
```

## 第三步

将以下代码写入到 bintray.gradle 文件中

```groovy
group = PROJ_GROUP
version = PROJ_VERSION
project.archivesBaseName = PROJ_ARTIFACTID

apply plugin: 'com.jfrog.bintray'
apply plugin: "com.jfrog.artifactory"
apply plugin: 'maven-publish'

task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}

task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += configurations.compile
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}

task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}

javadoc {
    options{
        encoding "UTF-8"
        charSet 'UTF-8'
        author true
        version true
        links "http://docs.oracle.com/javase/7/docs/api"
        title "$PROJ_NAME $PROJ_VERSION"
    }
}


def pomConfig = {
    licenses {
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
        mavenJava(MavenPublication) {
            artifactId PROJ_ARTIFACTID
            artifact javadocJar
            artifact sourcesJar

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
    publishing.publications.mavenJava.artifact(bundleRelease)
}

bintray {
    Properties properties = new Properties()
    properties.load(project.rootProject.file('local.properties').newDataInputStream())
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")

    publications = ['mavenJava']
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
            publications('mavenJava')
            publishArtifacts = true
        }
    }
}
```

## 第四步

配置 gradle.properties 文件， 这个文件再工程根目录下

```
project/
    \_ gradle.properties
```

写入

```properties
# 库的包名
PROJ_GROUP=cn.kejin.android.views
# 库的ID
PROJ_ARTIFACTID=XImageView
# 库的版本
PROJ_VERSION=1.0.0
### 最后 gradle引用的形式就是 $PROJ_GROUP:$PROJ_ARTIFACTID:$PROJ_VERSION

# 库名
PROJ_NAME=XImageView
# 库的项目主页
PROJ_WEBSITEURL=https://github.com/liungkejin/XImageView
# 问题跟踪地址
PROJ_ISSUETRACKERURL=https://github.com/liungkejin/XImageView/issues
# VCS 地址
PROJ_VCSURL=https://github.com/liungkejin/XImageView.git
# 库的简单描述
PROJ_DESCRIPTION=Android View for show large image

# 开发者的信息, 可以随意
DEVELOPER_ID=Kejin
DEVELOPER_NAME=Liang Ke Jin
DEVELOPER_EMAIL=liungkejin
```

## 第五步

运行发布，在我参考的文章中，是直接运行

```bash
gradle bintrayUpload # 发布到bintray.com
gradle artifactoryPublish  # 发布到oss.ifrog.org
```

但是这样做是很容易出错的，因为如果 javadoc 或者 javadocJar 等任务没有运行成功，
他会跳过，最后还是会将最后的 aar 发布上去。

所以我觉得，先单独运行各个子任务，都 Build 成功后，再来进行 publish和Upload

运行子任务可以使用 Android Studio 右边的 gradle 面板，如果什么都没有，可以点击面板上面的刷新按钮，出来后，
进入到你的库模块，找到这个几个子任务，双击执行

子任务有:

- javadoc  (生成 java 文档)

- javadocJar (将java文档打包)

- generatePomFileForMavenJavaPublication (生成 pom.xml)

- bintrayUpload  (发布到 bintray.com)

- artifactoryPublish (发布到 oss.ifrog.org)


## 最后

当成功上传到 bintray 上后， 可以进入到你的首页， 可以在左下角看到 My Recent Package 下面有你刚上传的库
点击进入，然后点击右下方有一个 Add to JCenter 按钮， 填写 comments （不写也可以），点击 Send, 然后等待审核通过,

最后，通过之后，就可以方便的使用这种方式来引用你的库了

```groovy
compile 'cn.kejin.android.views:XImageView:1.0.0'
```
