---
title: Linux 命令行工具：jq 命令介绍与实战（查询数据篇）
url: https://mp.weixin.qq.com/s?__biz=MzUxNTYzMjA5Mg==&mid=2247551397&idx=1&sn=f157edd36d8baacfcb664e68d4cd8d2a&chksm=f9b1f716cec67e00a0f40b4fa5aa3d3e47f973c750d59be1d8d45c7f49bbaf8de9e1971f8c36&mpshare=1&scene=1&srcid=0120TQzrAxD89Qz5AbLjdrpD&sharer_shareinfo=95cb947f8da6d80c5a562e311fe141d6&sharer_shareinfo_first=95cb947f8da6d80c5a562e311fe141d6#rd
clipped_at: 2024-03-31 19:17:37
category: temp
tags: 
 - mp.weixin.qq.com
---


# Linux 命令行工具：jq 命令介绍与实战（查询数据篇）

一

  

**简介**

**1.1** jq 是一个针对 JSON 的命令行工具，提供了用于查询、操作和使用 JSON 文件的大量功能，而且作为一个命令行工具，可配合 UNIX 管道使用，使用单行脚本就可以处理 JSON

**1.2** jq 可以对 json 数据进行'分片'、'过滤'、'映射'、'转换';和 sed、awk、grep 等命令一样，可以让你轻松地把玩文本

**1.3** 它能'轻松地'把你拥有的数据转换成你期望的格式，而且需要写的程序通常也比你期望的更加简短

**1.4** jq 是用 C 编写，运行时没有依赖，所以几乎可以运行在任何系统上

**1.5** 预编译的二进制文件可以直接在 Linux、OS X 和 windows 系统上运行

注意：jq 不是曾经流行的 JS 库 Jquery 的缩写。

二

  

**用法**

**2.1 基本用法  
**

```plain
jq [options] [filter] [file]
```

-   options: 可选参数，用于指定 jq 的选项。
    
-   filter: 必需参数，用于指定 JSON 数据的查询和转换操作。
    
-   file: 可选参数，要处理的 JSON 数据文件。
    

**2.2 常用选项**

-   \-r: 输出原始格式，而不是 JSON 编码。
    
-   \-c: 输出时将结果按行分隔。
    
-   \-s: 将输入视为多个 JSON 对象，用于处理多个 JSON 对象的数组。
    

**2.3 查询和过滤**

-   .: 表示当前对象，用于访问字段或属性。
    
-   .fieldName: 选择指定字段的值。
    
-   \[\]: 用于遍历数组元素。
    
-   select(condition): 根据条件选择元素。
    
-   map(transform): 对数组中的每个元素应用转换操作。
    

三

  

**示例**

## **3.1 数据格式化**

```plain
echo '{"name": "John", "age": 30, "city": "New York"}' | jq .
```

输出：

```plain
{
  "name": "John",
  "age": 30,
  "city": "New York"
}
```

## **3.2 选择特定字段**

## 查询姓名。

```plain
echo '{"name": "John", "age": 30, "city": "New York"}' | jq .name
```

输出：

```plain
"John"
```

## **3.3 使用通配符过滤**

## 查询 people 列表里的所有姓名。

```plain
echo '{"people": [{"name": "John", "age": 30}, {"name": "Alice", "age": 25}]}' | jq .people[].name
```

输出：

```plain
"John"
"Alice"
```

## **3.4 条件筛选**

## 打印 31 岁以上人员的信息。

```plain
echo '[{"name": "John", "age": 30, "city": "New York"},
{"name": "Peter", "age": 32, "city": "Chicago"}]' | jq '.[] | select(.age > 31) '
```

输出：

```plain
{
  "name": "Peter",
  "age": 32,
  "city": "Chicago"
}
```

## **3.5 数组操作**

## 将数组中的每个元素乘以 2 并打印。

```plain
echo '[1, 2, 3, 4, 5]' | jq '.[] | . * 2'
```

输出：

```plain
2
4
6
8
10
```

## **3.6 字符串拼接**

## 将名字和城市拼接在一起。

```plain
echo '{"name": "John", "age": 30, "city":
"New York"}' | jq '"Name: \(.name), City: \(.city)"'
```

输出：

```plain
"Name: John, City: New York"
```

## **3.7 计算数组长度**

```plain
echo '[1, 2, 3, 4, 5]' | jq 'length'
```

输出：

```plain
5
```

## **3.8 返回所有键值**

```plain
echo '{"name": "John", "age": 30, "city": "New York"}' | jq 'keys'
```

输出：

```plain
["name","age","city"]
```

## **3.9 比较并返回布尔值**

## 比较 John 的年龄是否比 Peter 大。

```plain
echo '[{"name": "John", "age": 30, "city": "New York"},
{"name": "Peter", "age": 32, "city": "Chicago"}]
' | jq '(.[] | select(.name == "John").age) > (.[] | select(.name == "Peter").age)'
```

输出：

```plain
false
```

## **3.10 返回数组中的最大值和最小值**

## 返回人员里边的最大值和最小值。

```plain
echo '[{"name": "John", "age": 30, "city": "New York"},{"name": "Peter", 
"age": 32, "city": "Chicago"},{"name": "Alice", "age": 28, "city": "Los 
Angeles"},{"name": "Bob", "age": 35, "city": "Miami"}]' | jq 'max_by(.age) as 
$max | min_by(.age) as $min | [$max.name, $max.age, $min.name, $min.age]'
```

输出：

```plain
[
  "Bob",
  35,
  "Alice",
  28
]
```

## **3.11 将数组元素进行排序**

## 按照由年龄由大到小顺序返回人员及年龄列表。

```plain
echo '[{"name": "John", "age": 30, "city": "New York"},
{"name": "Peter", "age": 32, "city": "Chicago"},
{"name": "Alice", "age": 28, "city": "Los Angeles"},
{"name": "Bob", "age": 35, "city": "Miami"}]' | jq 'sort_by(.age) | reverse | .[] | [.name, .age]'
```

输出：  

```plain
[
  "Bob",
  35
]
[
  "Peter",
  32
]
[
  "John",
  30
]
[
  "Alice",
  28
]
```

## **3.12 将数组转为逗号分隔的字符串**

```plain
echo '[1, 2, 3, 4, 5]' | jq -r 'join(",")'
```

输出：

```plain
1,2,3,4,5
```

## **3.13 提取数组中的某个范围的元素**

```plain
echo '[1, 2, 3, 4, 5]' | jq '.[1:4]'
```

输出：

```plain
[2,3,4]
```

## **3.14 合并两个对象**

```plain
echo '{"name": "John", "age": 30}' | jq '. + {"city": "New York"}'
```

输出：

```plain
{
  "name": "John",
  "age": 30,
  "city": "New York"
}
```

## **3.15 提取人员列表中年龄在特定范围内的人员信息**

```plain
echo '[{"name": "John", "age": 30, "city": "New York"},
{"name": "Peter", "age": 32, "city": "Chicago"},
{"name": "Alice", "age": 28, "city": "Los Angeles"},
{"name": "Bob", "age": 35, "city": "Miami"}]' | jq 'map(select(.age >= 30 and .age <= 35))'
```

输出：  

```plain
[
  {
    "name": "John",
    "age": 30,
    "city": "New York"
  },
  {
    "name": "Peter",
    "age": 32,
    "city": "Chicago"
  },
  {
    "name": "Bob",
    "age": 35,
    "city": "Miami"
  }
]
```

## **3.16 提取人员列表中城市为特定值的人员姓名**

```plain
echo '[{"name": "John", "age": 30, "city": "New York"},
{"name": "Peter", "age": 32, "city": "Chicago"},
{"name": "Alice", "age": 28, "city": "Los Angeles"},
{"name": "Bob", "age": 35, "city": "Miami"}]' | jq 'map(select(.city == "Chicago")) | .[].name'
```

输出：

```plain
"Peter"
```