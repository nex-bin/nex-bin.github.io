---
layout: 
title: Modbus RTU CRC 校验
date: 2026-06-30 15:02:53
tags: Modbus
---

## About Modbus RTU
Modbus协议被广泛应用于工业自动化系统中，核心优势在于结构简单，易于实现，且兼容性强，适用于多种物理层（如RS-485，TCP/IP等）。在实际应用中，Modbus主要有两种传输模式：ASCll和RTU。其中，Modbus RTU因其紧凑高效的数据格式和较强的抗干扰能力，在工业现场得到了更广泛的应用。    
为了确保数据在传输过程中不被破坏，Modbus RTU采用CRC（Cyclic Redundancy Check，循环冗余校验）机制进行数据完整性校验。CRC-16算法作为其校验标准，能够在接收端检测出绝大多数传输错误，从而保障通信的可靠性。

## Modbus RTU CRC校验机制
### Modbus RTU 帧结构组成
Modbus RTU帧结构由多个字段组成，包括地址域、功能码、数据域和CRC校验码。
![](https://image.modbus.cn/wp-content/uploads/2024/05/20240508211317493.png)

#### 地址域与功能码解析
Modbus RTU帧的的第一个字节是地址域（Address Field），用于标识目标从站设备的地址。地址范围是0x00到0xFF(即0~255)。其中：
* 0x00为广播地址，表示该请求将被所有从站接收并处理（通常不回复响应）;
* 0x01至0xF7为有效从站地址范围；
* 0xF8至0xFF保留或未使用。
紧接着地址域的是功能码(Function Code)，用于指示主站请求从站执行的具体操作。
{% asset_img modbus01.png [Modbus 功能码] %}
**帧结构示例：**
{% codeblock %}
0x01 0x03 0x00 0x00 0x00 0x02 [CRC1] [CRC2]
{% endcodeblock %}
**响应帧示例**
{% codeblock %}
0x01 0x03 0x04 0x00 0x0A 0x01 0x0B [CRC1] [CRC2]
{% endcodeblock %}

#### 数据域的格式与长度规范
数据域在Modbus RTU帧中紧跟在功能码之后，其内容和长度根据功能码的不同而变化。例如，在功能码0x03的响应帧中，数据域的第一个字节为返回数据的字节数，后续字节为实际数据。

####CRCC校验码的插入与作用
CRC校验码是Modbus RTU帧的最后一个字段，占据两个字节。用于校验整个帧中除CRC字段外的全部数据是否在传输过程中发生错误。CRC计算基于帧中从地址域到数据域的所有字节，通过CRC-16算法  生成一个16位的校验值，并附在帧末尾。
**CRC字段示意图**
{% codeblock %} [地址域] [功能码] [数据域] ... [CRC高字节] [CRC低字节]   <---- 大端
{% endcodeblock %}

## CRC校验码
CRC校验是Modbus RTU协议中保障通信可靠性的核心技术。其工作机制分为发送端的CRC计算和附加，以及接收端的校验流程，任何一方出错都会导致通信失败。
{% codeblock %}
V(x)=A(x)g(x)=xRm(x)+r(x);
{% endcodeblock %}
`
类似于一次线性函数，y = ax + b；每一个y都有唯一的x与之对应。而CRC校验中就是根据余数b来进行判断数据传输是否有出错的。其中a为生成多项式g(x)。  
计算CRC校验的过程就变成了计算余数的过程。
`
### 生成多项式
在CRC校验中存在多种变式。不同的变式对应不同的g(x)。在Modbus中使用的g(x)是：x^16 + x^15 + x^2 + 1。
`
x^16 + x^15 + x^2 + 1可以理解为是x^16 + x^15 + 0^14 + 0^13 + 。。。。 + x^2 + 0^1 + 1。其中，0的任何此幂都为0.
`
x^16 + x^15 + x^2 + 1 对应代码为： 0x 1100 0000 0000 0010 + 1 ====> 在实际使用中会截去最高位 ====> 0x 1000 0000 0000 0101 = 0x8005  ----> g(x)

### 模二除法
模二除法是二进制运算中基于异或逻辑的除法方法，其特点为运算过程不借位、不比较数值大小。仅仅通过相同位数相除完成计算。
{% asset_img modbus02.png /[Modbus] %}
图片左侧为正常运算过程，右侧为镜像运算过程。发现，镜像完全不影响运算过程。也分别对应了网络上的两种算法。**正向算法**和**反向算法**。
正向算法g(x)对应0xA001，反向算法g(x)对应0x8005。
{% asset_img modbus03.png /[modbus]%}
以反向算法为例：
{% codeblock CRC-16  lang: objc line_number: true highlight: true %}
unsigned short crc16_modbus(unsigned char *data, int len) {
    unsigned short crc = 0xFFFF;
    for (int i = 0; i < len; i++) {
        crc ^= data[i];
        for (int j = 0; j < 8; j++) {
            if (crc & 0x0001) {
                crc >>= 1;
                crc ^= 0xA001;
            } else {
                crc >>= 1;
            }
        }
    }
    return crc;
}
{% endcodeblock %}

逐行解释：

* unsigned short crc = 0xFFFF; ：初始化CRC寄存器；
* crc ^= data[i]; ：将当前字节与CRC低8位异或；
* for (int j = 0; j < 8; j++) ：对每个字节进行8位移位；
* if (crc & 0x0001) ：判断最低位是否为1；
* crc ^= 0xA001 ：若为1，则异或多项式；
* return crc; ：返回最终CRC值。

### 影响Modbus CRC-16设备间兼容性的参数：
|参数|值|
|---|---|
|多项式|0xA001（X^16+X^15+X^2+1）|
|初始值|0xFFFF|
|输入输出是否反转|不反转|
|输出异或值|0x0000|