

# ctf 逆向 - 迷宫题型总结（思路巨清晰详细） - 先知社区

ctf 逆向 - 迷宫题型总结（思路巨清晰详细）

- - -

前言：该篇文章会介绍一个在 ctf 当中 reverse 方向中常见的一种迷宫题，我从攻防世界中选取了一道比较有代表性的题目，逐一讲解，并且每个思路都相当清晰透彻，借此分享，相信看完之后对于迷宫类型的题目会有更深的理解。

# 例题：攻防世界-reverse\_re3

## 第一步

首先看有无壳，直接丢到 exeinfope---发现是 elf 文件是 linux 的可执行程序

[![](assets/1705886516-86bf4e1675d0de0205894259679b1cb3.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190129-434b8548-b783-1.png)

## 第二步

反手丢进 ida 看到 main 直接 F5 反编译看一手

[![](assets/1705886516-c2ef91f1ce07c23d1812dd5764396158.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190142-4b332158-b783-1.png)

## 第三步

一个一个函数来，sub\_11B4 双击点开好像没啥东西，看看 sub\_940

```bash
__int64 sub_940()
{
  int v0; // eax
  int v2; // [rsp+8h] [rbp-218h]
  int v3; // [rsp+Ch] [rbp-214h]
  char v4[520]; // [rsp+10h] [rbp-210h] BYREF
  unsigned __int64 v5; // [rsp+218h] [rbp-8h]
​
  v5 = __readfsqword(0x28u);
  v3 = 0;
  memset(v4, 0, 0x200uLL);
  _isoc99_scanf(&unk_1278, v4, v4);
  while ( 1 )
  {
    do
    {
      v2 = 0;
      sub_86C();
      v0 = v4[v3];
      if ( v0 == 'd' )               //原题是 ascii 码可以对着 ascii 按"r"快捷键转成字符
      {
        v2 = sub_E23();
      }
      else if ( v0 > 'd' )          //wasd 很明显就是游戏里常见的移动键呗
      {
        if ( v0 == 's' )
        {
          v2 = sub_C5A();
        }
        else if ( v0 == 'w' )
        {
          v2 = sub_A92();
        }
      }
      else
      {
        if ( v0 == 27 )
          return 0xFFFFFFFFLL;
        if ( v0 == 'a' )
          v2 = sub_FEC();
      }
      ++v3;
    }
    while ( v2 != 1 );
    if ( dword_202AB0 == 2 )
      break;
    ++dword_202AB0;
  }
  puts("success! the flag is flag{md5(your input)}");
  return 1LL;
}
```

这里留个心眼，flag 是要输入的 md5 值

接下来分析 wasd 到底是怎么走的（像这种题大概率就是迷宫图，所以要明白它是怎么运动的）

先打开 sub\_E23 康康

[![](assets/1705886516-9cc0392ea36bf697247a8b9350ecddb7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190232-68b7670c-b783-1.png)

## 第四步

千万不要给这几个变量绕晕了

dword\_202020：其实就是题目给的地图，可以双击进去看看（shift+e 提取数据）

dword\_202AB0：代表哪个迷宫，此题有 3 个迷宫（至于为什么后文会提及）

dword\_202AB4：代表行（因为 15\*，说明可能是一行 15 个数字，大胆猜测！）

dword\_202AB8：代表列

225：地图尺寸 15\*15

这里我转换了一下不容易混淆

[![](assets/1705886516-7e94339351aba3ce889bf7a04fe5e5eb.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190251-7410e3bc-b783-1.png)

## 第五步

第一个 if！=14 是因为跟下标有关系，一行 15 个数字下标最大是 14，我们这个函数是”d“的操作也就是向右移，所以是如果下标等于 14 的话就不能再”d“了  
第二个 if 判断当前位置右移的数字是不是 1，如果是将该位置标志为 3，之前的位置标志为 1，其实就是暗示我们 1 是可以走的，而 3 其实是我们的起点，如果不理解后面看看迷宫就明白了，  
第三个 if 如果右移后是 4 就返回 1，说明 4 就是我们的终点

再看看上下移动的函数（下图是”s“的函数，也就是下移）------有一些题目上下移动不是单纯的往上或者往下 而是斜向下或者斜向上噢

[![](assets/1705886516-4ef21ea5e7e48c2ac7703b28b05366e7.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190338-9092fb24-b783-1.png)

## 第六步

每次加 15 说明每次加一行

注意：这里分析的是右移”d“和下移”s“的函数至于”w“ ”a“都是差不多的，这里就不过多赘述了

再回到 sub\_940 函数

[![](assets/1705886516-deebbf86c87de664bfafee9f7f85b9a5.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190354-99ad4610-b783-1.png)

## 第七步

这里的 dword\_202AB0==2 我们在前面提到它可能是标识哪一个迷宫的，这里说==2 的话就 break，否则就++，说明这里会循环 3 次，说明会有 3 个迷宫，接下来我们来看看这个稍微的迷宫长什么样。

双击点开前文提到的 dword\_202020

[![](assets/1705886516-c547e38779ae5b1c2ffcd23d233a4ebd.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190406-a0f4ee32-b783-1.png)

## 第八步

这密密麻麻的数字就是它的数据啦，现在按 shift+e 提取数据，然后导出来

[![](assets/1705886516-d2ffcb913698ede260c4c2969ab4d13a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190430-af581198-b783-1.png)  
这里有个大大大坑，就是 dword 类型的数据是 4 位一组，只取第一位作为数值，也就是说 100010000111 最终会变成 110，后 3 位是填充的，接下来就是怎么转换成一个迷宫了，网上一堆 wp 却几乎没有人提到如何去把这份原始数据转换成迷宫，下面我提供了一个 python 脚本针对这道题目，只要运行即可得到迷宫的样子，

[![](assets/1705886516-522f61834562c63765d2c06aec16df72.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190446-b8b81b20-b783-1.png)  
一开始数据是这样的，先利用记事本的功能替换掉，和空格 以及前面的括号等等

[![](assets/1705886516-9130df2199a1f72c9f40ccd8f7d8a881.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190506-c486eb3e-b783-1.png)  
之后利用一下 py 脚本即可完成转换

```bash
# 输入需要处理的字符串  
input_string = input("请输入需要处理的字符串：")  

# 将所有字符连起来，去掉换行符  
output_string = input_string.replace("\n", "")  
ZeK1D = output_string.replace(" ", "")    
​
def extract_chars(input_string):  
    # 将输入字符串分割为每四位  
    chunks = [input_string[i:i+4] for i in range(0, len(input_string), 4)]  

    # 提取每四位中的第一位字符  
    first_chars = [chunk[0] for chunk in chunks if chunk]  

    # 将字符连在一起并输出  
    output= ''.join(first_chars)  
    print(output)  


# 调用函数并输出结果  
extract_chars(ZeK1D)
```

之后可以将输出结果复制到 word 改格式会好看一点

[![](assets/1705886516-859579192eca4151ab140ae9cc84043a.png)](https://xzfile.aliyuncs.com/media/upload/picture/20240120190647-01081f2e-b784-1.png)

## 第九步

之后再按每 15 行为一个迷宫分成 3 个小迷宫，以 3 为起点 4 为终点进行运动，之后再 md5 加密即可得到 flag

第一个迷宫

\[1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0\]

\[1, 1, 1, 1, 1, 0, 3, 1, 1, 0, 0, 0, 0, 0, 0\]

\[1, 1, 1, 1, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0\]

\[1, 1, 1, 1, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0\]

\[1, 1, 1, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0\]

\[1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0\]

\[1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0\]

\[1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0\]

\[1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0\]

\[1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 4, 0\]

\[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1\]

\[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1\]

\[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1\]

\[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1\]

\[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1\]

第二个迷宫

\[1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0\]

\[1, 1, 0, 3, 1, 1, 1, 1, 1, 0, 0, 0, 0, 0, 0\]

\[1, 1, 0, 1, 1, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0\]

\[1, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0\]

\[1, 1, 0, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0\]

\[1, 1, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0\]

\[1, 1, 0, 1, 1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0\]

\[1, 1, 0, 1, 1, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0\]

\[1, 1, 0, 1, 1, 0, 0, 0, 0, 0, 1, 0, 0, 1, 0\]

\[1, 1, 0, 1, 1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0\]

\[1, 1, 0, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1, 1, 0\]

\[1, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 0\]

\[1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 4, 0\]

\[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1\]

\[1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1\]

第三个迷宫

\[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0\]

\[0, 3, 1, 1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0\]

\[0, 0, 0, 1, 0, 1, 1, 1, 0, 0, 0, 0, 0, 0, 0\]

\[0, 0, 0, 1, 1, 1, 0, 1, 0, 0, 0, 0, 0, 0, 0\]

\[0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0\]

\[0, 1, 1, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0\]

\[0, 0, 1, 1, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0\]

\[0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0\]

\[0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0, 0, 0, 0\]

\[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0\]

\[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0\]

\[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 0\]

\[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 1, 1, 1, 0\]

\[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1, 0\]

\[0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 4, 0\]

行动轨迹为：ddsssddddsssdssdddddsssddddsssaassssdddsddssddwddssssssdddssssdddss

经过 md5 编码后 flag 是 flag{aeea66fcac7fa80ed8f79f38ad5bb953}

## 总结：

逆向遇到迷宫题的思路大概分 2 点

1.弄清楚移动的方式

2.分析迷宫图的尺寸

3.手动走迷宫或写脚本走迷宫（建议 BFS 算法）
