---
title: QT-basicGrammer
permalink: QT/basicGrammer/
date: 2026-11-15 09:27:09
tags: C++
---

## 类的实例化
### 类的继承以及类承接的参数
在一个类的声明文件中，比如KNXnetip如果需要使用QT内部的信号和槽机制，一般都会选择继承自QObject。
{% codeblock C++/basicGrammer KNXnetip.h   lang:objc first_line:5  line_number:true  highlight:true %}
class KNXnetip : public QObject 		#这里是表明继承关系。
{
	Q_OBJECT
public:
	explicit KNXnetip(QObject * parent = nulllptr);		#这里是表明接收的参数。
	~KNXnetip():
	
}
{% endcodeblock %}

{% codeblock C++/basicGrammer KNXnetip.cpp   lang:objc first_line:5  line_number:true  highlight:true %}
KNXnetip::KNXnetip(QObject * parent)
:QObject(parent)     #这里是表明继承关系。指出QObject是KNXnetip的父类，实例化时需要先实例父类。
{

}
{% endcodeblock %}
