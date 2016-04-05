---
layout: post_layout
title: 使用 Kotlin 开发 Android
time: 2016年04月05日 星期二
location: 上海
pulished: true
excerpt_separator: "#"
---

学习和使用 Kotlin 来开发安卓已经有一段时间了, 总体来说, 我还是很喜欢 kotlin 这个语言的,
比 java 用起来感觉轻便很多, 特别是支持 lambda, 委托, 隐式类型转换, 让代码显得特别的精炼.

最主要的是 kotlin 很大程度避免了 java 里面的空指针异常, 这也是 Kotlin 的设计初衷之一

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

// kotlin
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

## 配置 Android 的工程

使用 Kotlin 开发 Android, 首先得安装 Kotlin 的插件
