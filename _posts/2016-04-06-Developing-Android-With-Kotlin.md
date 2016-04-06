---
layout: post_layout
title: 使用 Kotlin 开发 Android
time: 2016年04月06日 星期三
location: 上海
pulished: true
excerpt_separator: "#"
---

学习和使用 [Kotlin](https://kotlinlang.org/) 来开发安卓已经有一段时间了, 总体来说, 我还是很喜欢 kotlin 这个语言的,
比 java 用起来感觉轻便很多, 特别是支持 lambda, 委托, 隐式类型转换, 让代码显得特别的精炼.

最主要的是 kotlin 很大程度避免了 java 里面的空指针异常, 而且可以和 java 自由混合.

用 Kotlin 来开发 Android 也极好的, 因为 Kotlin 支持 Java 6+  :)

可以看一下 java 和 kotlin 的语言对比:

```java
// java
view.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        v.post(new Runnable() {
            @Override
            public void run() {
                //
            }
        });
    }
});

// kotlin, lambda, 虽然 Java 8 也支持 lambda 了, 但是 Android 不支持 Java8 ...
view.setOnClickListener {
    it.post {
        // run
    }
}

// java, 这里如果 view 为 null, 则会抛出Null异常
void invisibleView(View view) {
    view.setVisibility(View.INVISIBLE);
}

// kotlin, 这是不允许传入null, 所以不会出现 Null 异常
fun invisibleView(view: View) {
    view.visibility = View.INVISIBLE
}
// 即使允许传入 null, kotlin 也可以很轻松的避免 Null 异常
fun invisibleView(view: Vieww?) {
    view?.visibility = View.INVISIBLE
}
```

不过kotlin也有一些不如人意的地方, 比如 泛型感觉没有java的好用 (可能是我还没掌握好吧!),

但总的来说 kotlin 是很值得学习的！

## 配置 Android 的工程 (Android Studio)

- 使用 Kotlin 开发 Android, 首先得安装 Kotlin 的插件, 直接在 `Settings->Plugins->Install JetBrains Plugins`搜索
Kotlin, 搜索出来应该会有两个一个是 Kotlin, 还有一个是 `Kotlin Extensions For Android`, Kotlin 是必须安装的,
Extensions 可选. 安装完成后重启 Android Studio.


- 现在 Android Studio 已经安装好了 Kotlin, 拥有了 Kotlin 的编译器, 在菜单栏 Tools 下面多了一个 Kotlin 菜单,
一般会用到第一个选项 `Configure Kotlin in Project`, 用来自动配置项目工程, 支持 Kotlin,
另外在 Code 菜单里面也多了一个 `Convert Java File to Kotlin File` 选项, 这个选项可以将 Java 代码自动转换为 Kotlin 代码


- 现在你可以新建一个Android工程, 然后使用 `Tools->Kotlin->Configure Kotlin in Project` 配置你的这个工程,
配置完成后你的项目工程 `app.gradle` 应该是这样的

```groovy
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
android {
    ...
    sourceSets {
        main.java.srcDirs += 'src/main/kotlin'
        // 可以用 kotlin 写测试, 很方便
        test.java.srcDirs += 'src/test/kotlin'
        androidTest.java.srcDirs += 'src/androidTest/kotlin'
    }
    ...
}

dependencies {
    ...
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
}

buildscript {
    ext.kotlin_version = '1.0.1-2'
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
    }
}
repositories {
    jcenter()
}
```

因为 Kotlin 也是基于 JVM 的, 他可以和 Java 代码文件自由混合, 所以其实你可以直接在 src/main/java/ 下面
创建 Kotlin 类, 然后调用 Java 类/方法, 或者 Java 里面调用 Kotlin 类/方法 :), 上面的配置文件中
`main.java.srcDirs += 'src/main/kotlin'` 只是为了将 kotlin 和 Java 文件分开

- 现在你已经用Kotlin来开发Android的了:), 可以试着将 MainActivity.java 使用 菜单` code->Convert Java File to Kotlin File` 选项转换成 MainActivity.kt,
然后运行工程. 或者你也可以直接新建一个 Kotlin 类, 然后在 MainActivity.java 里面进行调用.

```java
package cn.kejin.android

import android.support.v7.app.AppCompatActivity

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main)
    }
}
```


## 使用 Kotlin Extensions For Android

Kotlin 为 Android 的开发提供了一个扩展包, 这个扩展包可以简化很多的 Android 代码, 他会自动将 xml布局文件中
带有 id 的控件映射为对应名字的控件变量, 这样就不再需要我们手动去 findViewById() 了 XD. 例如:

你有一个 activity_main.xml 布局文件:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
              android:orientation="vertical"
              android:layout_width="match_parent"
              android:layout_height="match_parent">
    <TextView
        android:id="@+id/helloText"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```

然后你可以在你的 activity 中这样写

```java
package cn.kejin.android

import android.support.v7.app.AppCompatActivity
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main)

        helloText.text = "Hello, World"
    }
}
```

这种扩展和直接 findViewById()  性能上是一样的, 不过注意使用这种扩展必须要在 layout 被设置好了才能使用, 在 fagment 里面, 要等到 onViewCreated() 之后才能使用.


Kotlin的配置里面, 默认没有加这个扩展, 所以需要自己手动加入, 将 app.gradle 修改为

```groovy
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'

android {
    ...
    sourceSets {
        main.java.srcDirs += 'src/main/kotlin'
        // 可以用 kotlin 写测试, 很方便
        test.java.srcDirs += 'src/test/kotlin'
        androidTest.java.srcDirs += 'src/androidTest/kotlin'
    }
    ...
}

dependencies {
    ...
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
}

buildscript {
    ext.kotlin_version = '1.0.1-2'
    repositories {
        mavenCentral()
        jcenter()
    }
    dependencies {
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        classpath "org.jetbrains.kotlin:kotlin-android-extensions:$kotlin_version"
    }
}
repositories {
    jcenter()
}
```

现在, 享受 Kotlin 带给你的快感吧(虽然一开始会感觉不习惯) :)
