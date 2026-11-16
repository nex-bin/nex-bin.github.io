---
title: signalsAndSlots
permalink: QT/signalsAndSlots
date: 2026-11-15 11:53:33
tags:
---

## 信号和槽
信号和槽是QT中很重要，很有特性的一部分。是QT独创的一种 **对象间通信机制** ，用于解耦对象。当一个对象发生改变时（发信号），它不需要知道谁会处理，谁敢兴趣，谁来连接（槽）。

### 核心概念
- 信号（signals）：由 `signals:` 修饰，只有声明没有实现。它相当于广播，不管有没有人听，只要被`emit`触发，就会发出。
- 槽（slot）：由 `slots:`修饰，本质是普通的成员函数。它可以接收信号传递过来的参数。
- `connect` 函数： 把信号和槽绑定在一起。
### 特点：
- 送耦合：发送者不知道接收者的存在。
- 类型安全：QT5 之后推荐使用函数指针形式（`&class::signal`），编译器就能检查参数类型是否匹配。
- 多对多：一个信号可以连接多个槽；一个槽也可以连接多个信号。

## 元对象系统（Meta-Object System）
它是QT底层对C++进行的扩展，赋予了C++对象在运行时进行自我检查和动态交互的能力。

### 核心组成部分
- QObject 基类。任何想要使用元对象系统的类，都必须直接或者间接继承自QObject。
- Q_OBJECT宏。它告诉编译器，这个类需要元对象支持。
- moc(元对象编译器)。QT独有的预处理器。它在编译前会扫描所有包含Q_OBJECT宏的头文件，自动补全C++代码。里面包含该类的元数据。
- QMetaObject:生成的元数据类，内部包含了类的所有信息（类名、有哪些信号、有哪些槽、有哪些属性、枚举等）。

### 功能体现
- 运行时类型信息（RTTI）：通过OBJ->metaObject()->className()获取类名，或使用qobject_cast安全转换类型。
- 动态属性：可以用OBJ->setProperty("myProp",123)在运行时给对象动态加属性。
- 跨线程通信：通过QMetaObject::invokeMethod()实现。

## `信号与槽` 和 `元系统` 是如何协同工作的？
信号与槽的底层运转，完全依赖元对象系统。以一个典型的连接为例。
{% codeblock Cpp   lang:objc first_line:1  line_number:true  highlight:true %}
//假设发送者是 `sender` ,接收者是`receiver`
connect(sender, &sender::mysignal, receiver, &receiver::mySlot);
{% endcodeblock %}

- 元数据生成阶段：`moc` 扫描到 `Sender` 类时，发现它声明了  `mySignal` ，会在生成的 `moc_sender.cpp` 中为这个信号自动生成一段实现代码。这段代码的核心逻辑就是调用 `QMetaObject::activate()`，并带上信号的索引ID。 
- 连接阶段：当你调用  `connect` 时，QT不会去记函数指针，而是通过元对象系统，查出  `mySignal` 和 `mySlot` 在各自类中的字符串签名和索引ID，并在内部维护一张映射表。
- 触发阶段：当你的代码执行 `emit sender->mySignal()`时：
	- 实际上调用的是  `moc` 生成的那个实现代码。
	- 该代码向元对象系统发出激活请求 。
	- 元对象系统查阅内部的映射表，发现这个信号连接了 `Receiver` 的槽。
	- 最后，元对象系统根据之前记录的槽的索引，动态调用 `Receiver` 对象的那个函数。  

### 跨线程的魔法：
如果你用了 `QT::QuenedConnection` ，元对象系统会把这个函数调用的信息（包括参数），打包成一个 `QMetaCallEvent` （事件），投递到接收者所在线程的事件循环中。接收者线程从事件循环里取出这个事件，再动态执行这个槽。  
     

