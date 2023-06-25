---
title: "使用Jekyll的时候配置的记录"
categories:
  - Edge Case
tags:
  - content
  - css
  - edge case
  - lists
  - markup
toc: true
---

Nested and mixed lists are an interesting beast. It's a corner case to make sure that

* Lists within lists do not break the ordered list numbering order
* Your list styles go deep enough.
### 一个重要文档
jekyll中文站：http://jekyllcn.com/docs
### 每篇博文的头信息格式配置
每篇post中要带头信息，格式像下面这样子，头信息中全局变量会被解析出来
``` 
---
layout: post
title: Blogging Like a Hacker
date: YYYY-MM-DD HH:MM:SS +/-TTTT
category: Jekyll
---
```

提示: 不要重复
如果你不想重复设置你常用的头信息变量，只需为它们定义默认值，仅在必要的时候（或者从不）覆盖它们即可。
这种方式对预定义变量和自定义变量都有效。

预定义好的全局变量有
layout 如果设置的话，会指定使用该模板文件。指定模板文件时候不需要文件扩展名。模板文件必须放在 _layouts 目录下。
permalink 如果你需要让你发布的博客的 URL 地址不同于默认值 /year/month/day/title.html，那你就设置这个变量，然后变量值就会作为最终的 URL 地址。
published 如果你不想在站点生成后展示某篇特定的博文，那么就设置（该博文的）该变量为 false。
date 这里的日期会覆盖文章名字中的日期。这样就可以用来保障文章排序的正确。日期的具体格式为YYYY-MM-DD HH:MM:SS +/-TTTT；时，分，秒和时区都是可选的。
category/categories 除过将博客文章放在某个文件夹下面外，你还可以指定博客的一个或者多个分类属性。这样当你的站点生成后，这些文章就可以根据这些分类来阅读。categories 可以通过 YAML list，或者以逗号隔开的字符串指定。
tags 类似分类 categories，一篇文章也可以给它增加一个或者多个标签。同样，tags 可以通过 YAML 列表或者以逗号隔开的字符串指定。
### 代码高亮显示
高亮代码片段
你可以在代码片段中增加关键字 linenos 来显示行数。这样完整的高亮开始标记将会是: 
{% highlight ruby linenos %}
def show
@widget = Widget(params[:id])
respond_to do |format|
format.html # show.html.erb
format.json { render json: @widget }
end
end
{% endhighlight %}
{% highlight ruby %}
def show
@widget = Widget(params[:id])
respond_to do |format|
format.html # show.html.erb
format.json { render json: @widget }
end
end
{% endhighlight %}

### 其它的页面的位置Permalink
将 HTML 文件或者 Markdown 放在哪里取决于你想让它们如何工作。有两种方式可以创建页面：

将为页面准备的命名好的 HTML 文件或者 Markdown 文件放在站点的根目录下。
在站点的根目录下为每一个页面创建一个文件夹，并把 index.html 文件或者 index.md 放在每个文件夹里。
这两种方法都可以工作（并且可以混合使用），它们唯一的区别就是访问的 URL 样式不同。

命名 HTML 文件Permalink
增加一个新页面的最简单方法就是把给 HTML 文件起一个适当的名字并放在根目录下。一般来说，一个站点下通常会有：主页 (homepage), 关于 (about), 和一个联系 (contact) 页。根目录下的文件结构和对应生成的 URL 会是下面的样子：
``` shell
|-- _config.yml
|-- _includes/
|-- _layouts/
|-- _posts/
|-- _site/
|-- about.html    # => http://example.com/about.html
|-- index.html    # => http://example.com/
|-- other.md      # => http://example.com/other.html
└── contact.html  # => http://example.com/contact.html
```
干净的 URLs 也可以通过在头信息中定义　permalink　实现。就上边的第一个例子而言，在 other.md 文件的头信息中定义：permalink: /other，你就能得到 URL http://example.com/other