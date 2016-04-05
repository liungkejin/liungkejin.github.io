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
原本这个 pageIndex 属于一个数据模型, 不应该在 View 控制中进行处理!

所以, PageController 就是用来管理控制分页加载时的 page 的, 并方便管理整个分页加载的处理流程

## PageController
