---
layout: post_layout
title: Jekyll语法高亮问题
time: 2016年04月03日 星期天
location: 上海
pulished: true
excerpt_separator: "#"
---


Jekyll 的语法高亮在 Jekyll 3.0 以上默认使用的是 [Rouge](http://rouge.jneen.net/),
如果使用的是 Jekyll 2, 需要在配置文件里面设置 `highlighter` 为 `rouge`,
 并确保 rouge 被正确的安装了

__[表示此方法我没成功]__

---

另外, 就是使用 [Pygments](http://pygments.org/), 这个需要使用 Python 来安装

如果你不想麻烦, 可以直接使用我的这个 [syntax.css](/assets/css/syntax.css),
 这个就是最后使用 pygments 生成的默认的代码高亮 css 文件

- 把这个 syntax.css 放在 `/assets/css/syntax.css`

- 在 `_include/head.html` 或者你自己的通用头文件里面添加 link

```html
<link rel="stylesheet" href="/assets/css/highlight.css">
```

- 然后再 `_config.yml` 设置 `highlight: true`


### 安装和使用 [Pygments](https://pypi.python.org/pypi/Pygments)

```bash
pip install pygments
```

安装完成后, 你需要用它来生成一个 css 文件, 放在你的 jekyll 项目中, 生成命令为

```bash
pygmentize -S default -f html > style.css
# 这个 -S 就是 style, 默认的style 为 firendly 具体得 style 可以参考 [Styles](http://pygments.org/docs/styles/)
```

这个 style.css 就是我们所需要的.
详细的 Pygments 的使用请参考其 [官方文档](http://pygments.org/docs/)


### PS -\_-!!

#### [Python](https://www.python.org) 和 [pip](https://pypi.python.org/pypi/pip/#downloads) 安装和使用

- 下载安装 [Python](https://www.python.org/downloads)

- 将 Python 的安装路径加入到环境变量 path

- 下载 [pip](https://pypi.python.org/pypi/pip/#downloads) 安装包, 解压后进入到其目录, 使用此命令安装

```bash
python setup.py install
```

- 完成安装后, 在把 Python 安装目录下的 scripts 目录也加入到环境变量 path 里面

然后就可以用 pip 安装 Python 包了
