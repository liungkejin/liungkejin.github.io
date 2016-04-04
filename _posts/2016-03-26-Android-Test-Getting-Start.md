---
layout: post_layout
title: Android Test 学习基础
time: 2016年03月26日 星期六
location: 上海
pulished: true
excerpt_separator: "##"
---

开发安卓也有很长一段时间了，但是很少用到测试框架， 平时写用都靠打 log 进行调试和测试，
运行的对不对，异常情况的处理，边界条件的处理，全靠自己的想象，写出来的代码和应用健壮性可想而知！

所以学习测试框架是很有必要的一件事，虽然在公司可能很少会用到，因为你可能没有时间去写一个测试案例！
其工作量不亚于实现一个应用功能， 但是我觉得测试是对自己所写代码的一种负责！

## 基础

Android 的测试种类:


- **Unit Test** (单元测试)
  - **JUnit Test**

    这个只能用来测试无关Android平台的功能代码, 只能在本地运行
  - **Instrumentation Unit Test**

    这种单元测试运行在 Android 系统中, 这些测试可以获取到测试应用的上下文信息，用来测试有 Android API 的代码


- **Integration Tests** (集成测试)
  - **Components within your app only**

    这种类型的测试用来验证当用户进行了一个特定的操作或者特定的输入，目标应用的行为是否和预期一样。
    像 `Espresso` 这种UI测试框架就能允许你模拟用户的动作，能测试复杂的应用交互

  - **Cross-app Components**

    这种测试就是用来验证多个不同的应用间或者 应用和系统应用间的正确交互


## 配置测试 (Android Studio/Java/Kotlin)

在 Android Studio 中 Junit Test 的默认的测试文件夹在 `src/test/java`, Instrumentation Test 的文件夹在`src/androidTest/java`
也可以再 buid.gradle 里面重新设置目录，或者加入新的目录, 比如我想用 kotlin 来写单元测试


```groovy
android {
    sourceSets {
        main.java.srcDirs += 'src/main/kotlin'
        test.java.srcDirs += 'src/test/kotlin'
        androidTest.java.srcDirs += 'src/androidTest/kotlin'
    }
}
```


## JUnit Tests

JUnit Test 测试配置比较简单, 他的依赖只有一个, 使用的是 JUnit4


```groovy
dependencies {
    testCompile 'junit:junit:4.12'
}
```
然后在 `src/test/xx.xx.xx/`新建一个 MainUnitTest.kt(名字随意)

```kotlin
class MainUnitTest {
    @Test
    fun testSample() {
        println("hello")
    }
}
```

然后在菜单里点击 `buid` -> `select build variants`, 在弹出的面板里面，选择`Test Artifact`为 `Unit Tests`,

然后再 Project 面板里面，鼠标右击 MainUnitTest.kt, 在弹出的菜单中选择 run MainUnitTest,
就可以运行MainUnitTest 里面所有的 @Test 声明的方法


## Instrumentation Tests

Instrumentation Tests 所需要依赖的东西就比较多一点


```groovy
dependencies {
    androidTestCompile 'com.android.support.test:runner:0.4'
    androidTestCompile 'com.android.support.test:rules:0.4'
    androidTestCompile 'com.android.support.test.espresso:espresso-core:2.2.1'
    // 这个需要应用的 api > 18, 所以低api可以把它注释掉
    androidTestCompile 'com.android.support.test.uiautomator:uiautomator-v18:2.1.2'

    // 如果出现 (Warning:Conflict with dependency 'com.android.support:support-annotations'...)
    // 这种警告可以添加这一条,
    androidTestCompile 'com.android.support:support-annotations:23.1.0'
}
```

设置 `AndroidJUnitRunner` 作为默认的 test instrumentation runner, 配置 build.gradle


```groovy
android {
    defaultConfig {
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
}
```

在 `src/androidTest/XX.XX.XX` 下添加 MainActivtyTest.kt


```kotlin
class MainActivityTest : ActivityInstrumentationTestCase2<MainActivity>(MainActivity::class.java) {

    @Before
    public override fun setUp() {
        super.setUp();

        // Injecting the Instrumentation instance is required
        // for your test to run with AndroidJUnitRunner.
        injectInstrumentation(InstrumentationRegistry.getInstrumentation());
    }

    @Test
    fun pressBackTwice() {
        activity // getActivity()
        // 测试连续两次点击返回是否退出应用
        Espresso.pressBack()
        Thread.sleep(1000)
        Espresso.pressBack()
    }

    @After
    public override fun tearDown(){
        super.tearDown();
    }
}
```

然后在菜单里点击 `buid` -> `select build variants`, 在弹出的面板里面，选择`Test Artifact`为 `Android Instrumentation Tests`,

然后再 Project 面板里面，鼠标右击 MainUnitTest.kt, 在弹出的菜单中选择 run MainUnitTest,
就可以运行MainUnitTest 里面所有的 @Test 声明的方法


## 其他
有一个简便的添加单元测试类的方法， 就是打开需要测试的类，在文件里面点击右键选择 `Go To` -> `Test Subject`
在弹出的对话框中选择 Junit4, 其他的选择随意，点击确定即可创建单元测试类，


*注意创建的单元测试类的位置是根据 build variants面板中所选择的 Test Artifact 来定的，
即如果选择的 Unit Tests，则文件位置为 src/test/XX 下， 如果选择的 Instrumentation Tests
则位置为 src/androidTest/XX 下*
