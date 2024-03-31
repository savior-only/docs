---
title: Linux 运用 grep、sed、awk 三剑客
url: https://mp.weixin.qq.com/s?__biz=Mzg5OTY2NjUxMw==&mid=2247511047&idx=2&sn=6583baa81e9be16dd4272b1679b6e365&chksm=c04d2b39f73aa22f57e68c763eea920c675c19ef43c533d77ed73476972b26c60bc415859f0b&mpshare=1&scene=1&srcid=0216mii64JBOGXJToK35kAXb&sharer_shareinfo=55924997c1638eb9b741c24a42d43a6a&sharer_shareinfo_first=55924997c1638eb9b741c24a42d43a6a#rd
clipped_at: 2024-03-31 19:54:07
category: temp
tags: 
 - mp.weixin.qq.com
---


# Linux 运用 grep、sed、awk 三剑客

# **文本分析之AWK**

**awk是一种编程语言，用于在linux/unix下对文本和数据进行处理 ；****数据可以来自标准输入(stdin)、一个或多个文件，或其它命令的输出；****它支持用户自定义函数和动态正则表达式；****awk有很多内建的功能，比如数组、函数等，这是它和C语言的相同之处，灵活性是awk最大的优势**

  

```plain
awk [options] 'Pattern{Action}' filename
command [选项 参数] '模式{动作}'   文件
```

  

#### **常用命令选项**

```plain
-F fs  #fs指定命令分隔符，如 -F:  如果没有指定分割符，默认使用空格作为分隔符（或使用-v FS,FS是内置变量）
-v var=value  #赋值一个用户定义变量，将外部变量传递给awk，比如使用-v OFS="+",指定输出分隔符
-f script    #从脚本文件中读取awk命令
```

  

#### **最简单的用法：只用action**

```plain
$ echo "hello world" > test
$ awk '{print}' test
hello world
$ awk '{print $0}' test    #$0表示所有域(默认用空格当作分隔符)
hello world
$ awk '{print $1}' test    #$1表示第一个域,两个域用逗号隔开$1,$2，去掉逗号或换为空格输出时会没有分隔符显示在一列
hello
##############################
df -h | awk '{print $5}'  #df显示磁盘使用情况，这里使用awk只打印每行的第五列，$NF可以只输出每行的最后一列
```

  

#### **特殊模式的用法**

**BEGIN 模式指定了处理文本之前需要执行的操作：（BEGIN开始）；不指定文件也能输出（但是后边指定操作也会卡住）**

**END 模式指定了处理完所有行之后所需要执行的操作：（END结束）；不指定文件会卡住**

```plain
$ awk 'BEGIN{print "wintrysec"} {print $0} END{print "1080"}' test
wintrysec
hello world
1080
```

  

#### **AWK变量---****常用内置变量**

```plain
FS    #输入字段分隔符， 默认为空白字符
OFS    #输出字段分隔符， 默认为空白字符
RS    #输入记录分隔符(默认输入是换行符)， 指定输入时的换行符
ORS    #输出记录分隔符（输出换行符），输出时用指定符号代替换行符
NF    #number of Field，当前行的字段的个数(即当前行被分割成了几列)，字段数量
NR    #行号，当前处理的文本行的行号。
FNR    #各文件分别计数的行号，另一个文件从1开始计行数
ARGC  #命令行参数的个数
ARGV  #数组，保存的是命令行所给定的各参数，AGRV[0]是awk
FILENAME#当前文件名
```

  

**自定义变量的两种用法**

```plain
$ awk -v name="wintrysec",age=19 'BEGIN{print name}'  #第一种方法用-v选项，多个参数用逗号分隔   不加BENGIN会卡住
wintrysec
$ awk 'BEGIN{name="wintrysec";age=18;print name,age}'  #第二种方法，直接在程序中定义，多个参数用分号分隔
wintrysec 18
```

# **文本批量处理sed** 

**主要用来自动编辑一个或多个文件；简化对文件的反复操作；**

**命令格式**

```plain
sed [options] 'command' file(s)
sed [options] -f scriptfile file(s)
```

  

#### **常用选项**

```plain
-e<script>      #以选项中的指定的script来处理输入的文本文件；
-f<script文件>   #以选项中指定的script文件来处理输入的文本文件；
-n          #仅显示script处理后的结果；
```

#### **替换标记**

```plain
g  #表示行内全面替换；即替换每一行中的所有匹配
p  #表示打印行。
w  #表示把行写入一个文件。
x  #表示互换模板块中的文本和缓冲区中的文本。
y  #表示把一个字符翻译为另外的字符（但是不用于正则表达式）
\1  #子串匹配标记
&  #已匹配字符串标记
```

  

#### **常用命令**

```plain
d 删除，删除选择的行。s 替换指定字符 p 打印模板块的行。P (大写) 打印模板块的第一行。
```

**常见用法** 

**替换文本中的字符串 (如linux日志处理，伪造IP)**

```plain
sed -i 's/192.168.1.3/192.168.1.4/g' xxx.log  #-i选项表示直接编辑文件
```

  

**定界符**

**以上命令中字符 / 在sed中作为定界符使用，也可以使用任意的定界符**

```plain
sed 's|test|TEXT|g'
```

  

 **定界符出现在样式内部时，需要进行转义**

```plain
sed 's/\/bin/\/usr\/local\/bin/g'
```

**删除操作(d命令)**

```plain
sed '/^$/d' file  #删除空白行
sed '2d' file    #删除文件的第二行
sed '2,$d' file    #删除文件的第2行到末尾所有行
sed '$d' file     #删除文件最后一行
sed '/^test/'d file  #删除文件中所有开头是test的行
```

**grep文本正则匹配**

**常见用法：**

```plain
grep "text" file_name    #返回一个包含text的文本行
grep "text" file1 file2    #在多个文件中查找
grep -v "text" file_name  #-v选项，输出排除text之外的所有行
grep -E "[1-9]+"      #-e选项，使用正则表达式
grep -c "text" file_name  #统计文件中匹配字符串的行数
```