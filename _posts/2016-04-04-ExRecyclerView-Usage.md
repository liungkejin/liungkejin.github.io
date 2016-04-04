---
layout: post_layout
title: ExRecyclerView的使用介绍
time: 2016年04月04日 星期一
location: 上海
pulished: true
excerpt_separator: "#"
---
*ExRecyclerView 使用 Kotlin 编写*

很多时候我们在使用 RecyclerView 时, 总是会碰到设置一个 header 或者 footer 的情况,
比如我们要加一个显示加载更多的footer，跟随 RecyclerView 一起滑动的 header,
这种情况如果是 ListView 我们可以简单的使用 `addHeaderView()` 或者 `addFooterView()`
就可以解决, 但是 RecyclerView 就需要我们自己来进行处理. 虽然说不困难,
但是每次都要重新实现一遍就很麻烦了.

所以我写了ExRecyclerView, ExRecyclerView一共实现了3个功能:

1. 能添加和删除 header 和 footer
2. 滑到底部时回调加载更多
3. 支持 Drag 和 Swipe 拖动 item 或者 swipe 删除 item (当然你也可以自定义)


## ExRecyclerView

### Usage

```xml
<cn.kejin.android.views.ExRecyclerView
    android:id="@+id/exRecycler"
    android:layout_margin="50dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    app:header="@layout/layout_header"
    app:footer="@layout/layout_footer"/>
```

```java
exRecycler.layoutManager = LinearLayoutManager(this)
exRecycler.adapter = Adapter(this)
exRecycler.addHeader(header)
exRecycler.addFooter(footer)

exRecycler.setOnLoadMoreListener {
    // do load more, 结束之后调用 exRecycler.endLoadMore() 结束loading more的状态
}

exRecycler.itemActionListener = adapter
exRecycler.enableItemTouchHelper()
```

### Header & footer

| 方法/属性 | 简介 |
| ----------  | ---- |
| `app:header` | 在 xml 布局中定义 header view |
| `app:footer` | 在 xml 布局中定义 footer view |
| `getHeader()` | 获取定义在 xml 中的 header view |
| `getFooter()` | 获取定义在 xml 中的 footer view |
| `removeHeader()`  | 移除定义在 xml 中的 header |
| `removeFooter()` | 移除定义在 xml 中的 footer |
| | |
| `getHeaderSize()` | header size |
| `hasHeader(view)` | 判断是否有此header |
| `addHeader(view)` | 加入一个 header, 并返回它的 hashcode, 可以根据这个 hashcode 找到或者删除这个header |
| `getHeader(hashcode)` | 根据 view 的 hashcode 找到这个header |
| `removeHeader(view)` `removeHeader(hashcode)` | 根据 view 或者他的 hashcode 移除掉这个 header |
| | |
| `getFooterSize()` | footer size |
| `hasFooter(view)` | 判断是否由此footer |
| `addFooter(view)` | 加入一个 footer, 并返回它的 hashcode, 这个根据这个 hashcode 获取或者删除这个footer |
| `getFooter(hashcode)`  | 根据 view 的 hashcode 找到这个footer |
| `removeFooter(view)` `removeFooter(hashcode)` | 根据 view 或者他的 hashcode 移除掉这个 footer |

这些 header 和 footer 都是不会被 RecyclerView 进行回收的, 所以注意避免加入过多的 header 或者 footer.


### LoadMoreListener

当 ExRecyclerView 滑动到底部(倒数第二个可见) 时，就会回调可以进行 load more 操作.
但是要注意的是当 回调了 load more 之后, ExRecyclerView 会把状态置为 loadingMore 状态,
即不允许在正在执行 load more 操作时再回调 load more 事件, 当 load more 结束之后,
就必须要调用 `exRecycler.endLoadMore()` 来结束 loadingMore 状态.

### Drag & Swipe
