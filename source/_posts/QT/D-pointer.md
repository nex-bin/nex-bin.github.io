---
title: D-pointer
permalink: QT/D-pointer
date: 2026-11-16 09:26:31
tags:
	- QT
---

## QT中的D-pointer模式分析
### 概述
类中数据有多种组织方式，通常放在类本身中。有时，也会将数据部分分离出来，放在另外一个结构体或类中，只在类本身中存放一个指向该结构体的指针。如下图所示：

{% asset_img d-pointer01.png %}

这样做的好处是：
- 隐藏实现，可以直接发布头文件作为接口文档而不必担心源码泄露；
- 降低编译依赖，加快了编译速度；

这样做的缺点是：
- 每次函数调用都涉及到指针操作，运行速度可能变慢；
- 需要在堆上开辟空间。

这种模式一般被称为 `pimpl`(pointer to implementation),QT中引入这种模式，在QT文档中通常被称为 `D-pointer`，在QT中主要用来维护QT库的二进制兼容性，同时， `D-pointer` 在隐式数据共享（copy-on-write）也可以发挥重要的作用。

### d-pointer 实现的原理
一般的pimpl模式的实现如下，QT的参考文档中给了一个简洁的例子：
{% codeblock Widget Cpp  lang:objc first_line:1  line_number:true  highlight:true %}
class WidgetPrivate;
class Widget
{
	Rect geometry() const;
private:
	WidgetPrivate * d_ptr;
}
{% endcodeblock %}

{% codeblock WidgetPrivate h  lang:objc first_line:1  line_number:true  highlight:true %}
struct widgetPrivate
{
	Rect geometry;
	String stylesheet;
}
{% endcodeblock %}

widget_p.h,widget类内部使用的私有头文件

{% codeblock Widget h  lang:objc first_line:1  line_number:true  highlight:true %}
#include "widget_p.h"
Widget::Widget() : d_ptr(new WidgetPrivate)
{
	//creation of private data
}
Rect Widget::geometry() const
{
	//the d-ptr is only accessed in the library code
	return d_ptr->geometry;
}
{% endcodeblock %}

上面的例子中，所有的数据都存在WidgetPrivate中，在Widget类中，通过指针d_ptr来访问这些数据。
对于有继承出现的情况，上面的方法同样适用：
{% codeblock label h  lang:objc first_line:1  line_number:true  highlight:true %}
class Label : public Widget
{
	String text();
private:
	LabelPrivate * d_ptr;
}
{% endcodeblock %}

{% codeblock label cpp  lang:objc first_line:1  line_number:true  highlight:true %}
struct LabelPrivate
{
	String text;
};
Label::Label() : d_ptr(new LabelPrivate)
{
}
String Label::text()
{
	return d_ptr->text;
}
{% endcodeblock %}

这种方式实现了pimpl模式，但是它存在两个很明显的缺点：
- 在类的私有数据部分（LabelPrivate）不能访问Label或Widget的公共函数：例如LabelPrivate可能存着这样的接口setTextAndUpdateWidget，当用户设置了某个Label后，需要调用Widget的update函数。
- 每个类都有自己的私有数据成员，随着继承层次的加深，创建一个继承层次为N的继承类时，将会有N+1次昂贵的内存申请和释放。

对于第一个缺点，我们可以传递公有类的指针到私有数据类，作为私有数据类的一个成员，这样我们就可以在私有数据类调用公有类的方法。
在私有数据类中指向公有类的指针一版被称作q-pointer.

{% codeblock widget h  lang:objc first_line:1  line_number:true  highlight:true %}
class widgetPrivate;
class widget
{
	Rect geometry() const;
private:
	WidgetPrivate * d_ptr;
}
{% endcodeblock %}

{% codeblock widget_p h  lang:objc first_line:1  line_number:true  highlight:true %}
struct WidgetPrivate
{
	widgetPrivate(widget * q) : q_ptr(q){ };
	widget * q_ptr;
	Rect geometry;
	string stylesheet;
}
{% endcodeblock %}

对于第二个缺点，在继承体系中各子类中私有数据类各自申请独立内存空间其实不是必须的，一个比较好的办法是让子类的私有数据部分也形成继承关系，这样各私有数据类就可以共享一片内存区域。

{% asset_img d-pointer02.png %}
这样，当我们创建一个Label对象时，将会调用Label的构造函数创建一个LabelPrivate对象，并用该对象的地址初始化其父类的私有数据成员，从而避免了多次堆上内存申请。
这样做的缺点是Label类中的d_ptr和LabelPrivate中的q_ptr分别继承自Widget类与WidgetPrivate，它们的属性分别是WidgetPrivate和Widget，这样，我们在Label类中使用d_ptr和q_ptr时，必须做相应的类型转换。

{% codeblock Label cpp  lang:objc first_line:1  line_number:true  highlight:true %}
void Label::setText(const String &text)
{
    LabelPrivate *d = static_cast<LabelPrivate*>(d_ptr); // cast to our private type
    d->text = text;
}
{% endcodeblock %}

在这种方式下，代码中经常会使用static_cast，是很难看，而且也很容易犯错，在QT的源码 `corelib/global/qglobal.h` 中，定义了两个宏 `Q_D` 和 `Q_Q` ，分别用来获取转型后的d_ptr和q_ptr，需要使用d_ptr和q_ptr的地方，只要简单的用d和q代替即可。

### 采用d-pointer维护QT的二进制兼容性

二进制兼容的好处是当新库更新后，我们可以直接用新库替换旧库而不用重新编译、部署应用程序。当然这只限于动态链接库。编写代码时，保证库的二进制兼容的一条基本规则是：对于公有类，一旦发布，后期的版本只能在其上添加新的静态数据成员；不可以添加、删除非静态数据成员，不可以更改非静态数据成员的定义顺序和类型等。对于私有类，则没有任何的限制。主要原因是C++编译器采用offset来获取数据，当我们改变类的大小或数据相对位置时，同一offset处对应的数据成员会不同。

### 采用d-pointer实现隐式共享（copy-on-write）
