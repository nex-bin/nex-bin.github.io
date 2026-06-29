---

title: hexo 资源文件夹
date: 2026-05-28 15:28:06
tags:  资源文件夹的使用
---

## 前提
{% codeblock config.yaml https://hexo.nodejs.cn/docs/asset-folders.html "hexo官网链接" lang:json line_number:true highlight:true %}
post_asset_folder: true
{% endcodeblock %}

启用之后，每次使用`hexo new [layout] <title>` 会创建一个与markdown文件同名的文件夹。将与你的帖子相关的所有资源放入关联文件夹中，你可以使用相对路径引用他们。从而使工作流程更轻松。

## 用于相对路径引用的标签插件

使用普通markdown语法或其他资源可能会导致存档或者索引页面上的内容显示不正确。`hexo 3`中添加了新的标签插件。可以更轻松的在帖子中引用你的资源。
### 代码如下
{% codeblock%}
{ % asset_path slug %}
{ % asset_img slug [title] %}
{ % asset_link slug [title] % }
{% endcodeblock %}
例如，在启用帖子资源文件夹的情况下，如果你将图片，`example.jpg`放入资源文件夹，如果使用常规应用`![](example.jpg)`markdown的语法的相对路径引用它，他将不会出现在索引页面上。

## 图片的引用方法。
图片存放路径{% img /0601.png 400 300 %}
![](0530.png)
### 代码如下
{% codeblock lang:objc highlight:true %}
![](0530.png)
{%endcodeblock%}