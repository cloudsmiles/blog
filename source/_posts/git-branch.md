title: git基础详解(一) --- 对象模型
tags:
	- git
date: 2020-03-07 11:00:00
---
终于又开始我自己的技术之旅，这一次会为大家带来git的基础研究。程序员第一次朝圣之旅大概都会发生在github上，一个基于git而开发出来的代码托管平台。而git大家也是眼熟的不能再眼熟，用的也是滚瓜烂熟，但是我自己用了那么久，始终有个疑惑，git究竟是怎么实现的？

这绝不是妄言，作为技术人员，第一个大忌就是对事物产生了一种模糊的印象，不管你是新手还是老手，也希望看完我这篇文章都可以带来正面的反馈

<hr/>
<!--more-->

## git的知识领域

此目录内容会定期更新，因为git的内容很丰富，所以我选择了分开几篇文章介绍（也给我自己喘息的机会~）

* git的对象模型
* git的引用

## git的对象模型

### 创建一个版本库
git的核心部分是一个键值对数据库，意思就是你可以向git仓库中插入任意类型的内容，它会返回一个唯一的key，通过该key可以再次访问该内容。

这样说起来会有点不明不白，所以这里会从一个空白的git的版本库开始演示

	$git init
	Initialized empty Git repository in /xxx/.git/
	$ls .git
	HEAD		config		hooks		objects
	branches	description	info		refs

.git目录就是git的版本库，而我们需要关注的是objects目录，因为这是git存储对象的地方

### 存储对象
接下来会开始存储对象，因为个人的篇幅关系，这里使用的命令不会详细解释。

	$echo 'pokemon happy' | git hash-object -w --stdin
	01236f12ec549e2b1fe4c6107d91057778789b58
	$find .git/objects -type f
	.git/objects//01/236f12ec549e2b1fe4c6107d91057778789b58
	
相信你会有点惊讶，因为也会跟我建立同样的目录层次，接下来我会慢慢解释。

首先是`git hash-object`，它会接受你传给它的东西，然后返回存储在数据库的key，`-w`选项会指示该命令不要只返回key，还要把对象写入数据库中，而存入数据库的操作就是在objects目录下创建文件。git会把返回的key切分成两部分，前部分是两个字符，用于创建子目录以用于索引，后部分是38个字符，作为文件名。

###  读取对象
既然我们已经存储了对象，那我们试着读取这个对象，如果直接用`cat`命令读取文件内容，会发现是编码过的文件内容，其实是因为git做了手脚，在原来的文件内容的基础上附加上`header`，然后再进行编码，不过还好git自带了读取对象的命令。

	$git cat-file -p 01236f12ec549e2b1fe4c6107d91057778789b58
	pokemon happy

可以看出来，我们就顺利读取到之前存储的内容了，但这跟我们认识的git的功能大相径庭，它不是一个版本管理系统吗？版本管理系统的核心是文件内容的存储，需要存储一个文件的多个版本，同时也需要存储目录结构的变更，同时git也需要存储commit信息，以保证分支的合并和对比。

**所以git的对象模型奥秘就在git把这些内容都以不同类型的对象方式存储在这个key-value数据库，而探索它的奥秘，就先要认识它不同类型的对象的区别和使用。**

## 不同类型的对象
git可以使用`git cat-file -t`的命令去读取存储对象的类型

### blob对象
以上我们存储的对象，其实就是一个blob对象，我们可以试着去查看它
	
	$git cat-file -t 01236f12ec549e2b1fe4c6107d91057778789b58
	blob
	
可以看到是类型为**blob**，我们平常在工作目录（working directory）新建文件，然后添加到暂存区时（index），其实就会往git的数据库存储一个对象，也就是在objects目录会建立对应的文件，而存储的对象包含了文件内容（所以读取的对象时可以还原文件内容）。

>blob对象有什么用？
>
>blob对象包含了文件内容，而key的生成，是利用SHA1算法加密对象内容而得出的，所以key的不同，代表了文件内容的不同，也就是为什么存储上文的“pokemon happy”会得出相同的key。git只对文件内容进行校验，与其他的版本管理系统不一样，这样会更加快速比较文件的不同，如果文件在一次提交内没有改变，提交记录就会引用相同的blob对象，而无需再引用新的blob对象。

### tree对象
tree对象的查看，可以在已有项目的git仓库去体验查看

	$git cat-file -p master^{tree}
	100644 blob a906cb2a4a904a152e80877d4088654daad0c859      README
	100644 blob 8f94139338f9404f26296befa88755fc2598c289      Rakefile
	040000 tree 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0      lib

`master^{tree}`意思就是`master`分支上最新的提交所指向的tree对象，这里我们看到tree对象有多条item，每个item的字段意义分别是文件mode，对象类型，对象key名称，文件名。文件mode是指文件模式，与权限模式有关。而我们可以看到这个tree对象底下也有一个文件名是lib的tree对象，我们也可以看一下：

	$git cat-file -p 99f1a6d12cb4b6f19c8655fca46c3ecf317074e0
	100644 blob 47c6340d6459e05787f644c2447d2595f5d3a54b      simplegit.rb
	
tree对象通过这种方式，关联了blob对象和tree对象，而底下的tree对象又会递归关联，这就有点像我们的文件目录结构了，blob对象是文件，tree对象是目录。从概念上讲，git内部数据的存储方式像这样组织起来：

<img src="https://s1.ax1x.com/2020/04/18/Jeisu4.png"/>

>tree对象有什么用？
>
>tree对象的key也是根据对象内容进行加密生成，所以影响key值的因素就是目录下的文件和文件名的改动，即使是子目录改动（子tree对象的key值一样会改动），也会被父目录感知，导致key值变更，**所以两个key值相同的tree对象，代表了目录结构一致，目录内的文件内容一致，tree对象为我们记录了一个目录结构的快照，而key的对比可以快速比较快照，从而知道目录是否变更**。每个提交记录指向一个tree对象，提交记录的代码对比本质就是所指tree对象的对比。

### commit对象
上文提到，一个提交记录指向一个tree对象，而commit对象的作用其实就是存储提交记录，而我们也来看看commit对象是怎样存储信息的：
	
	$git cat-file -p master
	tree f46486556b70f646a6140d89fbea9690efb08718
	parent 2b6cdc298516a39a832aad66ccd39662e83c6a56
	author xxxx <xxxx@gmail.com> 1583574395 +0800
	committer xxxx<xxxx@gmail.com> 1583574395 +0800

	提交message

对象内容有指向的tree对象，指向上一个commit对象的parent节点，还有作者和提交者信息，最后就是提交的comment。

因为commit对象里面的提交者和作者的信息附带了时间戳，所以每次提交的key值会不相同，而commit对象指向的tree对象，反映了整个目录快照，方便不同的commit进行对比。parent节点指向上一个commit对象，所以就形成了git的commit流，每次生成commit对象，都会根据数据库的blob对象和tree对象生成tree对象快照，然后生成新的commit对象指向该tree对象，这就是git历史提交，暂存区，工作目录的底层原理。

## 对象key和内容
虽然已经介绍了各种类型的对象，但是我觉得还需要更加深入理解对象，就需要解开key的生成和内容这层面纱。

首先，对象的存储内容由对象的内容和header（头部）组成，头部的格式为`{type} {content.length}\0`，其中type就是指各个对象类型，content.length是指对象内容的长度。最后对组合的字符串进行SHA1加密，得出key，对组合的字符串进行zlib压缩，存储到文件中。


<hr/>
如果想要更清晰了解git提交的整个流程，可以浏览以下文章：

* [Git-内部原理-对象](https://git-scm.com/book/zh/v2/Git-%E5%86%85%E9%83%A8%E5%8E%9F%E7%90%86-Git-%E5%AF%B9%E8%B1%A1)
* [这才是真正的Git——Git内部原理揭秘！](https://www.jiqizhixin.com/articles/2019-12-20)
* [图解Git](https://marklodato.github.io/visual-git-guide/index-zh-cn.html)
*  [图解git原理与日常实用指南](https://juejin.im/post/5c714d18f265da2d98090503#heading-0)