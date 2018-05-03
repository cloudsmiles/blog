title: IP初全认识
date: 2018-05-04 22:43
tags: [计算机网络]
---
最近在通读[《TCP/IP详解 卷一》](https://book.douban.com/subject/1088054/)，对于计算机网络有了新的认识，而且大学的时候也没好好弄懂个中的原理，趁着这个机会重新整理所学。这本书可以称作教科书之典范，内容从网络的起源到现在广泛使用的技术，从表面的精炼解释到本质的运作，不论从事计算机何个职业，我觉得也应该拜访一下这本书，因为网络是计算机的发展必不可缺的，等同于人类需要交流协作才形成了世界，甚至是更远的未来。

<hr/>
<!--more-->

## 1. IP地址结构
Internet中使用的网络层地址，又称为IP地址。IP地址我们都很熟悉， 它是用于定位整个Internet系统中的设备，所以一台设备连接到全球性的Internet时，为它们分配地址需要协调，这样就不会重复使用网络中的其他地址。而通常个人用户是由[ISP](https://baike.baidu.com/item/ISP/10152#viewPageContent)分配地址，通过支付费用来获取地址。

IP地址分为IPv4和IPv6（本文只针对广泛应用的IPv4作说明，以下IP专有名词也默认指代IPv4）。IP地址由32位二进制数组成，它也可以用**点分四组**来表示，如下图所示：

|  点分四组表示 |  二进制表示 |
| ------------ |:------------:|
| 1.2.3.4  | 00000001 00000010 00000011 00000100 |
| 10.0.0.255  | 00001010 00000000 00000000 11111111  |

大部分的IP地址用于识别连接Internet或某些专用内联网的计算机网络接口，这些地址被称为**单播地址**。除了单播地址之外，其他类型的地址包括广播，组播和任播地址，它们可能涉及到多个接口，代表多个地址。
<br/>

## 2. 分类寻址
最初定义Internet地址结构的时，每个单播IP地址都有一个网络部分，用于识别哪个网络，以及一个主机地址，用于识别该网络下的特定主机。

| - | - |
|:------------ |:------------:|
|  网络位（x） | 主机位 （32-x）  |



由于实际使用中，不同的网络需要不同数量的主机，而每台主机需要唯一的IP地址，所以就根据不同大小的IP地址空间划分成不同的类，以适应不同的网络和站点，如下图所示：

|类 | 高序位  | 网络号位  | 主机号位  | 地址范围  | 用途  |
| ------------ |:------------:|:------------:|:------------:|:------------:|:------------:|
| A  | 0  | 8  | 24  | 0.0.0.0 ~ 127.255.255.255  | 单播/特殊  |
| B | 10  | 16  | 16  | 128.0.0.0 ~ 191.255.255.255  | 单播/特殊  |
|  C |  110 | 24  | 8  |  192.0.0.0 ~ 223.255.255.255 |  单播/特殊 |
|  D |  1110 | 0  | 0  | 224.0.0.0 ~ 239.255.255.255  | 组播  |
| E  | 1111  | 0  | 0  | 240.0.0.0 ~ 255.255.255.255  |  保留 |

比如C类的网络，因为主机号位是8，最多只能分配256台主机，但是网络位有24位，有超过200万的C类网络号可以使用。同理一个站点分配了一个A类网络号18.0.0.0，其中有2^24个地址分配给主机，（范围18.0.0.0 ~ 18.255.255.255）,但整个网络只有127个A类地址，所以A，B类地址尤为短缺。
<br/>

## 3. 子网寻址
Internet发展初期遇到了一个困难，就是很难为接入的新网段分配一个新的网络号，而且随着局域网（LAN）的增加，这个问题更为棘手，所以人们自然而然地想更加好的利用原来分配的网络号的主机地址空间，由站点人员进行进一步划分子网。

| - | - |-|
|:------------ |:------------:|
| 网络位 （x） | 子网位 （y） | 主机位 （32-x-y） |


子网的概念只是相对一个站点自身而言，Internet的其他部分还是只看到网络号，所以本质上来说，子网寻为IP地址结构增加了一个额外部分，但它没有为地址增加长度。