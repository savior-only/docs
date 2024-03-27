---
title: Find 过滤搜索、目录层级限制 (-maxdepth、-mindepth) 以及常用搜索技巧小结
url: https://www.cnblogs.com/kevingrace/p/11907695.html
clipped_at: 2024-03-28 00:24:42
category: default
tags: 
 - www.cnblogs.com
---


# Find 过滤搜索、目录层级限制 (-maxdepth、-mindepth) 以及常用搜索技巧小结

**1）find 过滤目录**  
使用 find 命令在 linux 系统中查找文件时，有时需要忽略某些目录，可以使用"\-path 过滤的目录路径 -prune -o"参数来进行过滤。不过必须注意：要忽略的路径参数要紧跟着搜索的路径之后，否则该参数无法起作用。

```bash
首先拿一个例子来说明下：
比如查找/data/web/ssy/online路径下的的目录，并统计目录大小，以G位单位进行排序（默认为降序），并统计前10个大小的目录。命令如下：
# find /data/web/ssy/online/* -maxdepth 0 -type d -exec /usr/bin/du -sh {} \;|grep '[0-9]G'|sort -rh|head -10

查找/data/web/ssy/online路径下除tmp目录之外的目录，并统计目录大小，以G位单位进行排序（默认为降序），并统计前10个大小的目录。命令如下
# find /data/web/ssy/online/* -path /data/web/ssy/online/tmp -prune -o -maxdepth 0 -type d -exec /usr/bin/du -sh {} \;|grep '[0-9]G'|sort -rh|head -10

注意：
1）"-maxdepth 0" 表示只查找到/data/web/ssy/online下的目录。如果是"-maxdepth 1"则表示查找到/data/web/ssy/online/xxx下的目录
2）find命令中的过滤、忽略、排除使用"-path 过滤的文件或目录-prune -o "，其中-prune类似于if判断，如果-prune之前的语句为真，比如找到了
   前面-path指定的/data/web/ssy/online/tmp目录，就不再执行后面-o跟的语句了，如果没有找到则执行后面的语句。这样就做到了排除效果！
   其中的"-o" 是 "-or" 的意思！
3）-path要过滤掉的文件或目录路径参数一定要紧跟在要搜索的路径之后，否则过滤效果就不会实现！！也就是说上面的"-path /data/web/ssy/online/tmp"
   必须紧跟着放在"/data/web/ssy/online/*"后面，否则查找时就不会过来掉/data/web/ssy/online/tmp这个目录。

========================================================================================================================================================
示例一：
假设/opt/kevin目录下有三个目录：test1，test2，test3，三个目录下都有list文件
[root@localhost kevin]# pwd
/opt/kevin
[root@localhost kevin]# ls
test1  test2  test3

现在要查找/opt/kevin路径下的list文件，并忽略掉test2目录，操作如下：
[root@localhost kevin]# pwd
/opt/kevin
[root@localhost kevin]# find . -type f -name list
./test1/list
./test2/list
./test3/list
[root@localhost kevin]# find . -type f -name list -print
./test1/list
./test2/list
./test3/list

使用-path 和 -prune -o实现过滤效果
[root@localhost kevin]# find . -path test2 -prune -o -type f -name list -print
./test1/list
./test2/list
./test3/list

[root@localhost kevin]# find . -path ./test2/ -prune -o -type f -name list -print
find: warning: -path ./test2/ will not match anything because it ends with /.
./test1/list
./test2/list
./test3/list

当搜索路径不是全路径时，过滤目录路径必须是./test2 才能实现过滤效果！
[root@localhost kevin]# find . -path ./test2 -prune -o -type f -name list -print
./test1/list
./test3/list


要过滤的目录操作-path必须紧跟着搜索路径 才能实现过滤效果
[root@localhost kevin]# find . -type f -path ./test2 -prune -o -name list -print
./test1/list
./test2/list
./test3/list

当搜索路径时全路径时，过滤路径也要是全路径，才能实现过滤效果
[root@localhost kevin]# find . -path /opt/kevin/test2 -prune -o -type f -name list -print
./test1/list
./test2/list
./test3/list

[root@localhost kevin]# find /opt/kevin/ -path /opt/kevin/test2 -prune -o -type f -name list -print
/opt/kevin/test1/list
/opt/kevin/test3/list

[root@localhost kevin]# find /opt/kevin/* -path /opt/kevin/test2 -prune -o -type f -name list -print
/opt/kevin/test1/list
/opt/kevin/test3/list

[root@localhost kevin]# find /opt/kevin -path /opt/kevin/test2 -prune -o -type f -name list -print
/opt/kevin/test1/list
/opt/kevin/test3/list

[root@localhost kevin]# find /opt/kevin -path /opt/kevin/test2/ -prune -o -type f -name list -print
find: warning: -path /opt/kevin/test2/ will not match anything because it ends with /.
/opt/kevin/test1/list
/opt/kevin/test2/list
/opt/kevin/test3/list

由上面可知：
1）当要搜索的目录不是全路径时，要过滤掉的目录必须是"./test2"才能实现过滤效果。如果是"test2"或者"./test2/"都不能实现过滤效果。
2）当要搜索的目录是全路径时，要过滤掉的目录也必须是全路径才能实现过滤效果！要过滤掉的目录后面不能加"/"，否则也不能实现过滤效果。
3）过滤操作"-path /opt/kevin/test2/ -prune -o"必须紧跟在要搜索路径的后面才能实现过滤效果，否则也不能实现过滤效果。

如果要过滤两个目录，比如过滤掉test2和test3目录，则使用转义符\( -path ./test2 -o -path ./test3 -prune -o \)
注意：两个转义符前面都要有空格！！

[root@localhost kevin]# find . -path ./test2 -o -path ./test3 -prune -o -type f -name list -print
./test1/list
./test2/list

[root@localhost kevin]# find . \( -path ./test2 -o -path ./test3 \) -prune -o -type f -name list -print
./test1/list

[root@localhost kevin]# find /opt/kevin/ \( -path /opt/kevin/test2 -o -path /opt/kevin/test3 \) -prune -o -type f -name list -print
/opt/kevin/test1/list

除了上面的方法，还有一个方法如下：
[root@localhost kevin]# find . -type f -name list ! -path ./test2/* ! -path ./test3/*
./test1/list
```

**2）find 过滤文件**  
先查看对应文件，然后使用"grep -v"进行过滤

```bash
比如只查找/opt/kevin目录下的文件（不查找/opt/kevin的二级目录下的文件），并过滤到haha2文件
[root@localhost kevin]# pwd
/opt/kevin
[root@localhost kevin]# ll
total 0
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha1
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha2
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha3
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha4
drwxr-xr-x 2 root root 18 Nov 21 18:24 test1
drwxr-xr-x 2 root root 18 Nov 21 18:24 test2
drwxr-xr-x 2 root root 18 Nov 21 18:24 test3

[root@localhost kevin]# find . -maxdepth 1 -type f
./haha
./haha1
./haha2
./haha3
./haha4

[root@localhost kevin]# find . -maxdepth 1 -type f |grep -v "haha2"
./haha
./haha1
./haha3
./haha4

过滤多个文件，就使用多个"grep -v"
[root@localhost kevin]# find . -maxdepth 1 -type f |grep -v "haha2"
./haha
./haha1
./haha3
./haha4

[root@localhost kevin]# find . -maxdepth 1 -type f |grep -v "haha2"|grep -v haha3
./haha
./haha1
./haha4

[root@localhost kevin]# find . -maxdepth 1 -type f |grep -v "haha2"|grep -v haha3|grep -v haha4
./haha
./haha1
```

**3）find 命令中的-maxdepth 和-mindepth：控制搜索深度的选项**  
\-maxdepth：指定遍历搜索的最大深度。最大目录层级  
\-mindepth：指定开始遍历搜索的最小深度。最小目录层级

```bash
-maxdepth 0：最大目录层级为0，表示只针对当前目录本身(比如/opt/kevin)进行搜索操作或du -sh 统计操作。
-maxdepth 1：最大目录层级为1，表示针对/opt/kevin/ 路径进行搜索操作或du -sh 统计操作。
-maxdepth 2：最大目录层级为2，表示针对/opt/kevin/xxx/ 路径进行搜索操作或du -sh 统计操作。

[root@localhost kevin]# pwd
/opt/kevin
[root@localhost kevin]# ll
total 0
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha1
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha2
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha3
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha4
drwxr-xr-x 2 root root 18 Nov 21 18:24 test1
drwxr-xr-x 2 root root 18 Nov 21 18:24 test2
drwxr-xr-x 2 root root 18 Nov 21 18:24 test3

-maxdepth 0 表示最小目录层级是0，即搜索路径它本身
[root@localhost kevin]# find . -maxdepth 0 -type f

但是如果当前路径加入"*"使用"-maxdepth 0" 效果和 当前路径不加"*"使用"-maxdepth 1" 是一样的！
[root@localhost kevin]# find ./* -maxdepth 0 -type f
./haha
./haha1
./haha2
./haha3
./haha4

[root@localhost kevin]# find . -maxdepth 1 -type f
./haha
./haha1
./haha2
./haha3
./haha4

[root@localhost kevin]# find /opt/kevin -maxdepth 0 -type f
[root@localhost kevin]# find /opt/kevin/ -maxdepth 0 -type f
[root@localhost kevin]# find /opt/kevin/* -maxdepth 0 -type f
/opt/kevin/haha
/opt/kevin/haha1
/opt/kevin/haha2
/opt/kevin/haha3

[root@localhost kevin]# find /opt/kevin -maxdepth 1 -type f
/opt/kevin/haha
/opt/kevin/haha1
/opt/kevin/haha2
/opt/kevin/haha3
/opt/kevin/haha4
[root@localhost kevin]# find /opt/kevin/ -maxdepth 1 -type f
/opt/kevin/haha
/opt/kevin/haha1
/opt/kevin/haha2
/opt/kevin/haha3
/opt/kevin/haha4

[root@localhost kevin]# find /opt/kevin/* -maxdepth 1 -type f
/opt/kevin/haha
/opt/kevin/haha1
/opt/kevin/haha2
/opt/kevin/haha3
/opt/kevin/haha4
/opt/kevin/test1/list
/opt/kevin/test2/list
/opt/kevin/test3/list

[root@localhost kevin]# find . -maxdepth 2 -type f
./test1/list
./test2/list
./test3/list
./haha
./haha1
./haha2
./haha3
./haha4

结论：
如果搜索路径后面加了"*"，则使用"-maxdepth n"
和
不加"*"使用"-maxdepth n+1"
的效果是一样的！！

超过了实际目录级层，效果是一样的
[root@localhost kevin]# find . -maxdepth 3 -type f
./test1/list
./test2/list
./test3/list
./haha
./haha1
./haha2
./haha3
./haha4

[root@localhost kevin]# find . -maxdepth 4 -type f
./test1/list
./test2/list
./test3/list
./haha
./haha1
./haha2
./haha3
./haha4

如果仅仅只是在/opt/kevin/xxx下搜索，即这里的最小目录深度是2
[root@localhost kevin]# pwd
/opt/kevin
[root@localhost kevin]# ll
total 0
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha1
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha2
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha3
-rw-r--r-- 1 root root  0 Nov 21 18:51 haha4
drwxr-xr-x 2 root root 18 Nov 21 18:24 test1
drwxr-xr-x 2 root root 18 Nov 21 18:24 test2
drwxr-xr-x 2 root root 18 Nov 21 18:24 test3

[root@localhost kevin]# find . -mindepth 2 -type f
./test1/list
./test2/list
./test3/list

最小目录层级为0
[root@localhost kevin]# find . -mindepth 0 -type f
./test1/list
./test2/list
./test3/list
./haha
./haha1
./haha2
./haha3
./haha4

最小目录层级为1
[root@localhost kevin]# find . -mindepth 1 -type f
./test1/list
./test2/list
./test3/list
./haha
./haha1
./haha2
./haha3
./haha4

最小目录层级为3，即超过当前最大目录层级，则就搜索不到了！
[root@localhost kevin]# find . -mindepth 3 -type f

========================================================================
-mindepth和-maxdepth可以一起结合起来使用，用于搜索指定层级范围内的文件。
========================================================================
如果只想搜索/opt/kevin/xxx下的文件，可行的做法：
第一种做法：最大目录层级是1，即-maxdepth 1
[root@localhost kevin]# find ./* -maxdepth 0 -type f
./haha
./haha1
./haha2
./haha3
./haha4

[root@localhost kevin]# find . -maxdepth 1 -type f
./haha
./haha1
./haha2
./haha3
./haha4

第二种做法：最小目录层级是1，最大目录层级是1，即-mindepth 1 -maxdepth 1
[root@localhost kevin]# find . -mindepth 1 -maxdepth 1 -type f
./haha
./haha1
./haha2
./haha3
./haha4

再来看下面的示例
[root@localhost kevin]# echo "123456" > bo/bobo/list1
[root@localhost kevin]# echo "123456" > bo/bobo/list2
[root@localhost kevin]# echo "123456" > bo/bobo/ke/list3
[root@localhost kevin]# echo "123456" > bo/bobo/ke/list4
[root@localhost kevin]# ll bo/
total 0
drwxr-xr-x 3 root root 42 Nov 21 23:23 bobo
[root@localhost kevin]# ll bo/bobo/
total 8
drwxr-xr-x 2 root root 32 Nov 21 23:23 ke
-rw-r--r-- 1 root root  7 Nov 21 23:23 list1
-rw-r--r-- 1 root root  7 Nov 21 23:23 list2
[root@localhost kevin]# ll bo/bobo/ke/
total 8
-rw-r--r-- 1 root root 7 Nov 21 23:23 list3
-rw-r--r-- 1 root root 7 Nov 21 23:23 list4

如果想搜索/opt/kevin/xxx/xxx下的文件，即最小目录层级是3,最大目录层级是3
[root@localhost kevin]# find . -mindepth 3 -type f
./bo/bobo/ke/list3
./bo/bobo/ke/list4
./bo/bobo/list1
./bo/bobo/list2

[root@localhost kevin]# find . -maxdepth 3 -type f
./test1/list
./test2/list
./test3/list
./haha
./haha1
./haha2
./haha3
./haha4
./bo/bobo/list1
./bo/bobo/list2

[root@localhost kevin]# find . -mindepth 3 -maxdepth 3 -type f
./bo/bobo/list1
./bo/bobo/list2

如果想要搜索第二层级和第三层级之间的文件，如下：
[root@localhost kevin]# find . -mindepth 2 -type f
./test1/list
./test2/list
./test3/list
./bo/bobo/ke/list3
./bo/bobo/ke/list4
./bo/bobo/list1
./bo/bobo/list2

[root@localhost kevin]# find . -maxdepth 3 -type f
./test1/list
./test2/list
./test3/list
./haha
./haha1
./haha2
./haha3
./haha4
./bo/bobo/list1
./bo/bobo/list2

[root@localhost kevin]# find . -mindepth 2 -maxdepth 3 -type f
./test1/list
./test2/list
./test3/list
./bo/bobo/list1
./bo/bobo/list2
```

**############  更多小示例说明  ############**

```bash
1. 在当前目录下查找所有txt后缀文件
# find ./ -name "*.txt"

2.在当前目录下的dir0目录及子目录下查找txt后缀文件
# find ./ -path "./dir0*" -name "*.txt"

3.在当前目录下的dir0目录下的子目录dir00及其子目录下查找txt后缀文件
# find ./ -path "*dir00*" -name "*.txt"

4.在除dir0及子目录以外的目录下查找txt后缀文件
# find ./ -path "./dir0*" -a -prune -o -name "*.txt" -print
这里注意：
-a 是and的缩写, 意思是逻辑运算符'与'(&&);
-o 是or的缩写, 意思是逻辑运算符'或'(||), - not 表示非.
上条命令的意思是：
如果目录dir0存在（即-a左边为真），则求-prune的值，-prune 返回真，'与'逻辑表达式为真（即-path './dir0*' -a -prune 为真），
find命令将在除这个目录以外的目录下查找txt后缀文件并打印出来；
如果目录dir0不存在（即-a左边为假），则不求值-prune ，'与'逻辑表达式为假，则在当前目录下查找所有txt后缀文件。

5.在dir0、dir1及子目录下查找txt后缀文件
# find ./ \( -path "./dir0*" -o -path "./dir1*" \)  -a -name "*.txt" -print

6.在除dir0、dir1及子目录以外的目录下查找txt后缀文件
# find ./ \( -path "./dir0*" -o -path "./dir1*" \) -a -prune -o -name "*.txt" -print
这里注意：
圆括号()表示表达式的结合。即指示 shell 不对后面的字符作特殊解释，而留给 find 命令去解释其意义。由于命令行不能直接使用圆括号，
所以需要用反斜杠'\'进行转意(即'\'转意字符使命令行认识圆括号)。同时注意'\('，'\)'两边都需空格。

7. 在所有以名为dir_general的目录下查找txt后缀文件
# find ./ -path "*/dir_general/*" -name "*.txt" -print

8. 在当前目录下，过滤.git目录内容（.git目录本身也不要列出来），命令如下：
# find . -path ./.git -prune -o -print -a \( -type f -o -type l -o -type d \) | grep '.git'
```

**########## find 文件搜索条件 ############**  
\-name、-iname、通配符\*？、-size、-user、-group、-amin、-cmin、-mmin、-a、-o、-exec/-ok、-inum

```bash
#find [搜索范围] [匹配条件]
-------------------------------------------------
1. 根据文件名称进行find检索
find 命令中的 -name 选项可以根据文件名称进行检索（区分大小写）。如需要忽略文件名中的大小写，可以使用 -iname 选项。
-name   区分大小写
-iname  不分区大小写
?    可以表示任意一个单一的符号
*    可以表示任意数量（包括 0）的未知符号

#find /usr -name '*.txt'   #查找/usr目录下所有文件名以.txt结尾的文件
#find /usr -name '????'    #查找/usr目录下所有文件名刚好为4个字符的文件

#find /etc -name init
#find /etc -name *init*
#find /etc -name init???

#touch /tmp/inIt
#mkdir /tmp/Init
#find /tmp -name init
#find /tmp -iname init    #不区分大小写
#find /tmp -iname ini*

有些时候，需要在搜索时匹配某个文件或目录的完整路径，而不仅仅是匹配文件名。可以使用 -path 或 -ipath 选项。
如查找/usr下所有文件名以.txt结尾的文件或目录，且该文件的父目录必须是src。可以使用以下命令：
#find /usr -path '*/src/*.txt'
-------------------------------------------------
2. 根据文件类型进行find检索
如果只想搜索得到文件或目录，即不想它们同时出现在结果中。可以使用 -type 选项指定文件类型。
-type 选项最常用的参数如下：
f:   文件
d:   目录
l:   符号链接

# find /usr -type d -name 'python*'   #检索/usr下所有文件名以python开头的目录
#find /etc -name init* -type f
#find /etc -name init* -a -type d

-a 两个条件同时满足
-o 两个条件满足一个即可
-------------------------------------------------
3. 根据文件大小进行find检索
-size 选项允许用户通过文件大小进行搜索（只适用于文件，目录没有大小）。

表示文件大小的单位由以下字符组成：
c：字节
k：Kb
M：Mb
G：Gb

另外，还可以使用 + 或 - 符号表示大于或小于当前条件。
#find / -size +204800       #查找大于100M的文件
#find / -size -204800       #查找小于100M的文件
#find / -size 204800        #查找等于100M的文件
#find / -size +1G           #检索文件大小高于1GB的文件
#find / -size -10M          #检索文件大小小于10M的文件

204800单位是数据块
1数据块=512字节=0.5K
100MB = 102400KB
100MB = 2048数据块

#find /etc -size +163840 -a -size -204800   #查找大于80M小于100M的文件
# find / -size 50M
# find / -size +50M -size -100M
# find / -size +100M -exec rm -rf {} ;
# find / -type f -name *.mp3 -size +10M -exec rm {} ;
-------------------------------------------------
4. 检索空文件
find 命令支持 -empty 选项用来检索为空的文件或目录。空文件即文件里没有任何内容，空目录即目录中没有任何文件或子目录。
# find ~ -type d -empty    #检索用户主目录下所有的空目录
-------------------------------------------------
5. 反义匹配
find 命令也允许用户对当前的匹配条件进行"反义"（类似于逻辑非操作）。
如需要检索 /usr 下所有文件名不以 .txt 为后缀的文件。可以使用以下命令：
#find /usr -type f ! -name '*.txt'

也可以"翻转"任何其他的筛选条件，如：
#find /usr -type f ! -empty 检索 /usr 下所有内容不为空的文件
-------------------------------------------------
6. 根据文件的所属权进行find检索
#find / -type f -user starky  #检索根目录下所有属主为starky的文件
#find / -user root 在根目录下查找所有者为root的文件
#find / -group root 在根目录下查找所属组为root的文件

# find / -user root -name tecmint.txt
# find /home -user tecmint
# find /home -group developer
# find /home -user tecmint -iname "*.txt"
-------------------------------------------------
7. 根据时间日期进行find检索

修改时间（Modification time）：最后一次文件内容有过更改的时间点
访问时间（Access time）：最后一次文件有被读取过的时间点
变更时间（Change time）：最后一次文件有被变更过的时间点（如内容被修改，或权限等 metadata 被修改）
与此对应的是 find 命令中的 -mtime，-atime 和 -ctime 三个选项。

这三个选项的使用遵循以下示例中的规则：
-mtime 2： 该文件 2 天前被修改过
-mtime -2：该文件 2 天以内被修改过
-mtime +2：该文件距离上次修改已经超过 2 天时间

#find /usr -type f -mtime 2   #检索/usr下两天前被修改过的文件

如果觉得 -mtime 等选项以天为单位时间有点长，还可以使用 -mmin，-amin，-cmin 三个选项：
#find /usr -type f -mtime +50 -mtime -100    #检索/usr下50到100天之前修改过的文件
#find /usr -type f -mtime 2 -amin 5          #检索/usr下两天前被修改过且5分钟前又读取过的文件

#find /etc -amin 5    #查找5分钟之前被访问过的文件和目录
#find /etc -amin -5   #查找5分钟之内被访问过的文件和目录
#find /etc -amin +5   #查找距离上次被访问时间超过5分钟过的文件和目录

#find /etc -cmin 5    #查找5分钟之前被修改过属性的文件和目录
#find /etc -cmin -5   #查找5分钟之内被修改过属性的文件和目录
#find /etc -cmin +5   #查找距离上次被修改过属性超过5分钟的文件和目录

#find /etc -mmin 5    #查找5分钟之前被修改过内容的文件和目录
#find /etc -mmin -5   #查找5分钟之内被修改过内容的文件和目录
#find /etc -mmin +5   #查找距离上次被修改过内容超过5分钟的文件和目录
-------------------------------------------------
8. 根据文件权限进行find检索
find 命令可以使用 -perm 选项以文件权限为依据进行搜索。

使用符号形式
如需要检索 /usr 目录下权限为 rwxr-xr-x 的文件，可以使用以下命令：
# find /usr -perm u=rwx,g=rx,o=rx

搜索/usr目录下所有权限为 r-xr-xr-x（即系统中的所有用户都只有读写权限）的文件和目录，可以使用以下命令：
#find /usr -perm a=rx

很多时候，只想匹配文件权限的一个子集。比如，检索可以直接被任何用户执行的文件，即只关心文件的执行权限，而不用管其读写权限是什么。
上述的需求可以通过以下命令实现：
#find / -type f -perm /a=x
其中 a=x 前面的 / 符号即用来表示只匹配权限的某个子集（执行权限），而不用关心其他权限的具体设置。

使用数字形式
-perm 选项也支持数字形式的文件权限标记。
# find /usr -perm 644     #搜索/usr目录下权限为 644（即 rwxr-xr-x）的文件

# find . -type f -perm 0777 -print
# find / -type f ! -perm 777
# find / -perm 2644
# find / -perm 1551
# find / -perm /u=s
# find / -perm /g+s
# find / -perm /u=r
# find / -perm /a=x
# find / -type f -perm 0777 -print -exec chmod 644 {}\;
# find / -type d -perm 777 -print -exec chmod 755 {}\;
# find . -type f -name "tecmint.txt" -exec rm -f {} \;
# find . -type f -name "*.txt" -exec rm -f {} \;
# find . -type f -name "*.mp3" -exec rm -f {} \;
# find /tmp -type f -empty
# find /tmp -type d -empty
# find /tmp -type f -name ".*"
-------------------------------------------------
9. 限制遍历的层数（这个在上面文章已经详细介绍）
find 命令默认是以递归的方式检索项目的，这有时候会导致得到的结果数量非常巨大。可以使用 -maxdepth 限制 find 命令递归的层数。
# find / -maxdepth 3 搜索时向下递归的层数最大为 3
-------------------------------------------------
10. 逻辑组合
在之前的例子中有出现多个搜索条件的组合以及对某个搜索条件的反转。
实际上 find 命令支持 "and" 和 "or" 两种逻辑运算，对应的命令选项分别是 -a 和 -o。通过这两个选项可以对搜索条件进行更复杂的组合。

此外还可以使用小括号对搜索条件进行分组。注意 find 命令中的小括号常需要用单引号包裹起来。因小括号在 Shell 中有特殊的含义。

如检索 /usr 下文件名以 python 开头且类型为目录的文件
#find /usr -type d -name 'python*'
该命令等同于：
#find /usr -type d -a -name 'python*'
更复杂的组合形式如：
#find / '(' -mmin -5 -o -mtime +50 ')' -a -type f
-------------------------------------------------
11. -exec command {} \;  以及 -ok command {} \; 用法
#find /etc -name inittab -exec ls -l {} \;     #查找inittab文件并显示其详细信息
在{}和\之间要有一个空格
-exec跟{}\; 之间执行的是对搜索结果进行的操作动作

#find /etc -name inittab -a -type f -exec ls -l {} \;
#touch /tmp/testfile.rm
#find /tmp -name testfile.* -exec rm {} \;

-ok等同于-exec
-ok跟{}\; 之间执行的是对搜索结果进行的操作动作
和-exec不同的地方在于有一个询问，需要输入y或n确认
#find /etc -name init* -ok rm -l {} \;
-------------------------------------------------
12. 根据inode节点进行find检索
#find /etc -inum xxx     #根据I节点查找

#touch "test 000"
#ls -i
#find . -inum 396401 -exec rm {} \;

#touch test999
#ln test999 test9999
#ls -i test999
#find . -inum 396401 -exec ls -l {} \;
-------------------------------------------------
13. 对搜索结果执行命令
1）删除文件
-delete 选项可以用来删除搜索到的文件和目录。

如删除 home 目录下所有的空目录：
# find ~ -type d -empty -delete

2）执行自定义命令
-exec 选项可以对搜索到的结果执行特定的命令。

如需要将home目录下所有的MP3音频文件复制到移动存储设备（假设路径是/media/MyDrive），可使用下面的命令：
#find ~ -type f -name '*.mp3' -exec cp {} /media/MyDrive ';'

上面命令中的大括号{}作为检索到的文件的占位符 ，而分号;作为命令结束的标志。因为分号是Shell中有特殊含义的符号，所以需要使用单引号括起来。
每当find命令检索到一个符合条件的文件，会使用其完整路径取代命令中的 {}，然后执行 -exec 后面的命令一次。

另一个很重要的用法是，在多个文件中检索某个指定的字符串。
如在用户主目录下的所有文件中检索字符串 hello ，可以使用如下命令：
# find ~ -type f -exec grep -l hello {} ';'

-exec 选项中的 + 符号
现在假设需要将用户主目录下所有的MP3文件添加到压缩包 music.tar.gz 中，直观的感觉是，其命令应为如下形式：
#find ~ -type f -name '*.mp3' -exec tar -czvf music.tar.gz {} ';'

实际情况是，上面命令得到的music.tar.gz 其实只包含一个MP3文件。
原因是find命令每次发现一个音频文件，都会再执行一次-exec选项后面的压缩命令。导致先前生成的压缩包被覆盖。

可以先让find命令检索出所有符合条件的音频文件，再将得到的文件列表传递给后面的压缩命令。所以正确完整的命令如下：
#find ~ -type f -name '*.mp3' -exec tar -czvf music.tar.gz {} +

3）显示文件信息
如果想浏览搜索到的文件（目录）的详细信息（如权限和大小等），可以直接使用-ls选项。
#find / -type file -size +1G -ls    #浏览所有 1G 以上大小的文件的详细信息
```