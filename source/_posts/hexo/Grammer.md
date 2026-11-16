---
title: hexo 用法
permalink: /hexo/grammer
date: 2026-06-11 15:51:29
tags: Hexo
---

## 简单创建
{% codeblock lang:javascript highlight:true %}
hexo new [layout] <title>  用于创建一个新的名为title的文章
{% endcodeblock %}
示例：(在config.yaml中配置了之后，就会如下生成一个文件夹，每个文章的资源加载路径也变成了对应的文件夹名)
{% asset_img hexo01.png %}

## 带有相对路径的方式创建
{% codeblock lang:javascript highlight:true %}
hexo new -p QT/test01   在当前相对路径下创建QT/test01.md的文章(如果QT文件夹不存在则创建，存在则只创建文章)
{% endcodeblock %}
{% asset_img hexo02.png %}

hexo 有三种默认布局：**post** 、**page** 和 **draft**。新建一篇文章，如果没有设置**layout**的话，默认使用 `_config.yaml` 中的 **default_layout** 参数代替。使用布局 **draft** 来创建草稿。

### 静态论坛
配合Github项目，Aciton的自解析。将Hexo生成的资源文件夹(/public)上传到github。

将源代码整体上传到另一条线
示例：

{% img /main01.png %}

[Hexo+Next+Github](https://io-oi.me/tech/hexo-next-optimization/)
