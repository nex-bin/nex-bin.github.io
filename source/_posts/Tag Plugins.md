---
title: Tag Plugins
date: 2026-05-28 09:36:21
tags: 标签插件是从Octopress移植的，旨在提供一种快速向帖子添加特定内容的有效方法。
---
## 引用块

{% blockquote 过零丁洋 ,文天祥 %}
人生自古谁无死，留取丹心照汗青。
{% endblockquote %}
```bash
{% blockquote   过零丁洋,文天祥 %}
人生自古谁无死，留取丹心照汗青。
{% endblockquote %}
```
## 代码块
{% codeblock hello world https://www.runoob.com/cprogramming/c-function-printf.html Runoob.com  lang:objc first_line:5  line_number:true  highlight:true %}
#include <stdio.h>
 
int main ()
{
   int ch;
 
   for( ch = 75 ; ch <= 100; ch++ ) {
      printf("ASCII 值 = %d, 字符 = %c\n", ch , ch );
   }
 
   return(0);
}
{% endcodeblock %}
```bash
{% codeblock hello world  https://www.runoob.com/cprogramming/c-function-printf.html lang:objc first_line:5  line_number:true  highlight:true %}
#include <stdio.h>
 
int main ()
{
   int ch;
 
   for( ch = 75 ; ch <= 100; ch++ ) {
      printf("ASCII 值 = %d, 字符 = %c\n", ch , ch );
   }
 
   return(0);
}
{% endcodeblock %}
```
## iframe

{% iframe https://pic.netbian.com/ 800 400 %}
```bash
{% iframe https://pic.netbian.com/ 800 400 %}
```

## 图像
{% img /20260420.jpg 400 600 '"title text" "运动是生命的热浪" ' %}
```bash
{% img [class names] /20260420.jpg  '"title text" "运动是生命的热浪" ' %}
```
## 链接
{% link 4K高清壁纸 https://pic.netbian.com/ [external] [title] %}
```bash
{% link 4K高清壁纸 https://pic.netbian.com/ [external] [title] %}
```
## 代码示例

{%  include_code lang:javascript from:3 to:5 test.js %}

{% codeblock lang:javascript %}
{ %  include_code lang:javascript from:3 to:5 test.js %}
{% endcodeblock %}



