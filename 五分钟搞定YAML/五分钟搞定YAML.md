---
title: 五分钟搞定YAML
url: https://mp.weixin.qq.com/s?__biz=MzU2MTgxODgwNA==&mid=2247486082&idx=1&sn=2975c25fe12ac4ba2185cba827b9ff39&chksm=fc73b759cb043e4f36f6b5a14feab61e59868794b94ca642d7406f2b6993cd232fca55b2b068&mpshare=1&scene=1&srcid=0313PjgySl9OMmmdq9kTZ7Jy&sharer_shareinfo=3ccb2d9ff47d19e55d1f7d38fbfccdcf&sharer_shareinfo_first=3ccb2d9ff47d19e55d1f7d38fbfccdcf#rd
clipped_at: 2024-03-31 16:01:58
category: temp
tags: 
 - mp.weixin.qq.com
---


# 五分钟搞定YAML

YAML\[2\]是一种数据序列化语言，允许我们以紧凑、可读的格式存储复杂的数据。YAML如今是DevOps和虚拟化的重要工具，能够帮助我们高效处理数据管理系统和自动化。

虽然开发人员常常有意无意的忽视YAML，但其实这是一个强大而简单的工具，只需要几个小时的学习，就有可能大大提升职业发展前景。

**以下是本文主要内容:**

-   什么是YAML?
    
-   YAML的显著特征
    
-   YAML语法
    
-   今后需要进一步学习的进阶概念
    

- - -

#### 什么是YAML?

YAML是一种数据序列化语言，用于以人类可读的形式存储信息。其最初为"Yet Another Markup Language"的缩写，但后来被更改为"YAML Ain’t Markup Language"，以区别于真正的标记语言。

YAML类似于XML\[3\]和JSON\[4\]，但用更简单的语法实现类似的功能。YAML通常用于在基础设施即代码(IoC, infrastructure-as-code)应用中创建配置文件，或管理DevOps开发流水线中的容器。

最近，YAML被用于创建自动化协议，以执行YAML文件中列出的一系列命令。这意味着系统可以更加独立，具备更强的响应性，而无需开发人员的额外关注。

随着越来越多的公司接受DevOps和虚拟化，YAML正迅速成为现代开发人员的必备技能。通过PyYAML\[5\]库、Docker\[6\]或Ansible\[7\]等流行技术的支持，YAML也很容易与现有技术集成。

- - -

#### YAML vs. JSON vs. XML

##### YAML (.yml)

-   人类可读的代码
    
-   最简语法
    
-   专为数据设计
    
-   类似于JSON的内联样式(是JSON的超集)
    
-   允许添加评论/注释
    
-   不带引号的字符串
    
-   被认为是"更干净"的JSON
    
-   高级特性(可扩展数据类型、关系锚和保持键顺序的映射类型)
    

**用例:** YAML最适合使用DevOps流水线或虚拟机的数据密集型应用程序。如果团队中其他开发人员经常使用这些数据，因此需要使其更具可读性时，也会很有帮助。

##### JSON

-   很难阅读
    
-   明确、严格的语法要求
    
-   类似YAML的内联样式(某些YAML解析器可以读取JSON文件)
    
-   没有注释
    
-   字符串需要双引号
    

**用例:** 因为JSON最适合序列化以及通过HTTP传输数据，因此在web开发中最受欢迎。

##### XML

-   很难阅读
    
-   更冗余
    
-   主要用于标记语言，而YAML用于数据格式化
    
-   包含比YAML更多的特性，比如tag属性
    
-   更严格定义的文档格式
    

**用例:** XML最适合需要对验证、格式和命名空间进行精确控制的复杂项目。XML不是人类可读的，需要更多的带宽和存储容量，但提供了无与伦比的控制力。

- - -

#### YAML的显著特征

下面是YAML提供的部分最佳特性。

##### 多文档支持

可以在一个YAML文件中包含多个YAML文档，以简化文件组织或数据解析。

每个文档之间的分隔由三个破折号(`---`)标记。

```plain
---
player:
playerOne
action:
attack
(miss)
---
player:
playerTwo
action:
attack
(hit)
--------
```

##### 内建注释

YAML允许使用类似于Python注释的井号(`#`)向文件添加注释。

```plain
key:
#Here is a single-line comment
-
value
line
5
#Here is a
#multi-line comment
-
value
line
13
```

##### 可读语法

YAML使用类似于Python的缩进来显示程序结构，需要通过空格而不是tab来创建缩进，以避免混淆。

它还减少了JSON和XML文件中的许多"噪声"格式，如引号、方括号和大括号。

这些格式化规范共同为YAML文件提供了超越XML和JSON的可读性。

**YAML**

```plain
#YAML
Imaro:
author:
Charles
R.
Saunders
language:
English
publication-year:
1981
pages:
224
```

**JSON**

```plain
#JSON
{
  "Imaro": {
    "author": "Charles R. Saunders",
    "language": "English",
     "publication-year": "1981",
     "pages": 224,
  }
}
```

注意，上述示例传达了同样的信息，但是在YAML文件中删除双引号、逗号和括号使其更容易阅读。

##### 隐式和显式类型

YAML通过自动检测数据类型提供了类型上的多功能性，同时还支持显式类型选项。要将数据标记为特定类型，只需在值之前包含`!![typeName]`。

```plain
# The value should be an int:
is-an-int:
!!int
14.10
# Turn any value to a string:
is-a-str:
!!str
67.43
# The next value should be a boolean:
is-a-bool:
!!bool
yes
```

##### 没有可执行的命令

作为一种数据表示格式，YAML不包含可执行文件，因此与外部方交换YAML文件是非常安全的。

为了添加可执行文件，YAML必须与其他语言(如Perl\[8\]或Java)集成。

- - -

#### YAML语法

YAML有一些基本概念用于组成大部分数据。

##### 键-值对(Key-value pairs)

YAML文件中的大多数内容通常都是键-值对的形式出现，其中键表示名称，值表示与该名称链接的数据。键-值对是所有其他YAML结构的基础。

```plain
<key>:
<value>
```

##### 标量和映射(Scalars and mapping)

标量表示单个存储的值，映射将标量分配给键名。可以使用名称、冒号和空格定义一个映射，然后为它定义一个值。

YAML支持常见类型，如整型和浮点型，以及非数值类型的布尔和字符串。

可以用不同方式表示数据，比如十六进制、八进制或指数。还有一些特殊类型的数学概念，如无穷大、负无穷大和非数字(`NAN`)。

```plain
integer:
25
hex:
0x12d4
#evaluates to 4820
octal:
023332
#evaluates to 9946
float:
25.0
exponent:
12.3015e+05
#evaluates to 1230150.0
boolean:
Yes
string:
"25"
infinity:
.inf
# evaluates to infinity
neginf:
-.Inf
#evaluates to negative infinity
not:
.NAN
#Not a Number
```

##### 字符串(String)

字符串是表示句子或短语的字符集合，可以使用`|`将每个字符串作为新行打印，或者使用`>`将其作为段落打印。

YAML中的字符串不需要放在双引号中。

```plain
str:
Hello
World
data:
|   These   Newlines   Are broken up
data:
>   This text is   wrapped and is a   single paragraph
```

##### 序列(Sequence)

序列是一种类似于列表或数组的数据结构，在同一个键下保存多个值，使用块(block)或内联流(flow)格式定义。

块样式使用空格来组织文档结构，更容易阅读，但与流格式相比不那么紧凑。

```plain
--------
# Shopping List Sequence in Block Style
shopping:
-
milk
-
eggs
-
juice
```

流格式允许使用方括号内联编写序列，类似于Python或JavaScript等编程语言中的数组声明。

流格式更紧凑，但更难阅读。

```plain
--------
# Shopping List Sequence in Flow Style
shopping:
[milk,
eggs,
juice]
```

##### 字典(Dictionaries)

字典是嵌套在同一子组中的键-值对的集合，有助于将数据以逻辑类别划分后使用。

字典的定义与映射类似，输入字典名称、冒号和空格，后面跟着一个或多个缩进的键值对。

```plain
# An employee record
Employees:
-
dan:
name:
Dan
D.
Veloper
job:
Developer
team:
DevOps
-
dora:
name:
Dora
D.
Veloper
job:
Project
Manager
team:
Web
Subscriptions
```

字典也可以包含更复杂的结构，比如序列。嵌套序列是表示复杂关系数据的一个好技巧。

- - -

#### 需要进一步学习的进阶概念

祝贺你迈出了学习YAML的第一步。虽然YAML经常被忽视，但确实是一个简单而有效的工具。

**接下来需要了解的一些高级主题:**

-   锚点(Anchors)
    
-   模板(Templates)
    
-   YAML与外部工具的集成(Docker, Ansible等)
    
-   高级序列/映射类型
    
-   高级数据类型(时间戳、null等)
    

学习愉快!

### 参考资料

\[1\]

YAML Tutorial: Get Started With YAML in 5 Minutes: *https://betterprogramming.pub/yaml-tutorial-get-started-with-yaml-in-5-minutes-549d462972d8*

\[2\]

YAML: *https://yaml.org*

\[3\]

XML: *https://www.w3schools.com/xml/xml\_whatis.asp*

\[4\]

JSON: *https://www.json.org/json-en.html*

\[5\]

PyYAML: *https://pypi.org/project/PyYAML*

\[6\]

Docker: *https://docs.docker.com/get-started/overview*

\[7\]

Ansible: *https://www.ansible.com*

\[8\]

Perl: *https://www.perl.org*

\- END -