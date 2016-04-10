---
layout: post_layout
title: PageController的使用介绍
time: 2016年04月05日 星期二
location: 上海
pulished: true
excerpt_separator: "#"
---

在开发 App 的过程中, 很多时候我们都要用到分页加载, 每次在处理分页操作我都感觉到很麻烦(可能是我太笨..),
因为每次都得维护一个 pageIndex, 刷新时重置为0, 加载更多时+1, 加载失败, 刷新失败...等等都得进行处理,
所以导致我们在写分页加载页面的时候, 很容易就会漏处理了一个逻辑或者逻辑混乱而导致出现一下小 bug.

PageController 就是用来管理控制分页加载时的 page 的, 并方便维护整个分页加载的处理流程,
PageDriver 继承自 PageController, 用 view 来驱动页面控制器

# PageController

PageController 的条件:

- page 按顺序进行加载, 第一页为刷新页
- 页面加载结果只有三种 SUCCESS, FAILED, NO_MORE(没有更多数据了)
- 一次只允许一页正在加载

一般来说, 分页加载的顺序为

```
refresh -> endRefresh -> loadmore -> endloadmore -> no more
    \_ refreshFailed        \_ loadmoreFailed
        (重新refresh)             (加载失败,点击)
```

- 我们进行刷新
    - 如果刷新失败, 如果页面没有数据则显示异常页面 (此时可以点击重新刷新)
    - 刷新成功但是没有更多的数据 (此时不能进行loadmore, 只能进行refresh)
    - 刷新成功且还有更多的数据 (此时可以进行loadmore, refresh)

- 然后进行加载更多
    - 如果加载失败, 则需要显示重新加载按钮 (此时可以进行 点击loadmore本页,或者refresh)
    - 如果加载成功但是没有更多数据了 (此时不能进行loadmore, 只能进行refresh)
    - 如果加载成功且还有更多的数据 (此时可以进行loadmore, refresh)

PageController 维护了3个变量

```java
/**
 * 保存上一次请求的结果
 */
var lastResult = Result.SUCCESS

/**
 * 已经加载好的页面号
 */
var loadedPageIndex = -1

/**
 * 当前正在加载的页面, 在页面加载完成后, 就会被置为 -1
 * 所以用它来判断是否正在加载,
 */
var loadingPageIndex = -1
```
