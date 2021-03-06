---
title: hctf2016 giligili writeup
date: 2016-11-19 16:32:13
tags:
- Blogs
- ctf
- hctf
categories:
- Blogs
---

题目是我很早以前做的sctf q1中的web500,我做的时候踩了坑，所以花了很久，感觉题目很有意思，所以就修改了题目又放上来了。

正解wp
[https://github.com/sternze/CTF_writeups/blob/master/sCTF/2016_Q1/obfuscat/readme.md](https://github.com/sternze/CTF_writeups/blob/master/sCTF/2016_Q1/obfuscat/readme.md)

最终分数：124
完成队伍：64

解题人数意外的多，感觉还是有很多py的人，迷...

<!--more-->
分析代码
```
h = new MersenneTwister(parseInt(btoa(answer[_[$[6]]](0, 4)), 32));
```
首先我们根据第一句得到h，h为一个伪随机的数组，根据前四位构成，所以flag的头hctf很重要。

紧接着很多伪随机数都成了固定的数字，也就便于分析了

```
o = answer.split("_");
```
这里把输入根据下划线分割得到o

```
e =- (this[_[$[42]]](_[$[31]](o[1])) ^ s[0]); if (-e != $[21]) return false;
e ^= (this[_[$[42]]](_[$[31]](o[2])) ^ s[1]); if (-e != $[22]) return false; e -= 0x352c4a9b;
```
根据这部分，相互异或得到了中间两部分的ascii码，这里其实分析逻辑容易进入误区，这里要肯定的是，其实flag一定是可显示的字符，所以肯定不可以是3位或更多位分割，这么一来就很容易确认了


代码到了这里
```
a += _[$[31]](o[3].substring(o[3].length - 2)).split("x")[1];
//o[3]的最后两位
d = parseInt(a, 16) == (Math.pow(2, 16)+ -5+ "") + o[3].charCodeAt(o[3].length - 3).toString(16) + "53846" + (new Date().getFullYear()- +1+ "");
						
```

出现了很多可变的东西，首先是a
a由o[0]和o[3]共同决定
```
a = parseInt(_[$[23]]("1", Math.max(o[0].length, o[3].length)), 3) ^ eval(_[$[31]](o[0]));
```
其次a会在十进制的基础上，拼接上o[3]出大括号的后两位的十六进制，转十进制
`a += _[$[31]](o[3].substring(o[3].length - 2)).split("x")[1];`

和a相比较的后面的代码，其中有一个可变量为o[3]的倒数第三位，这里出现了隐藏条件
1、中间的o[3]倒数第三位转十六进制后不允许存在字母
2、整体转16进制之后，除了后四位不能存在字母
3、倒数2位必须可显

综合上面的条件，我们就需要脚本来解决问题了

```
for i in range(30,120):

	 if 'a' not in hex(int("65531" + repr(i) + "53846" + "2015"))[2:-5]:
	 	if 'b' not in hex(int("65531" + repr(i) + "53846" + "2015"))[2:-5]:
	 		if 'c' not in hex(int("65531" + repr(i) + "53846" + "2015"))[2:-5]:
	 			if 'd' not in hex(int("65531" + repr(i) + "53846" + "2015"))[2:-5]:
	 				if 'e' not in hex(int("65531" + repr(i) + "53846" + "2015"))[2:-5]:
	 					if 'f' not in hex(int("65531" + repr(i) + "53846" + "2015"))[2:-5]:
							print i
							print hex(int("65531" + repr(i) + "53846" + "2015"))
							print hex(int("65531" + repr(i) + "53846" + "2015"))[-5:-1]
							# print chr(int(hex(int("65531" + repr(i) + "53846" + "2015"))[-5:-3],16))
							# print chr(int(hex(int("65531" + repr(i) + "53846" + "2015"))[-3:-1],16))
```

下面我们得到了7条，紧接着，筛选可显示字符
只剩下3个了
```
64
0x17481184783f3fL
3f3f
```
只剩下这个了，那么最后二位是**??**,倒数第三位是**d**

那么我们现在还得到了o[3]和o[0]相关的关系，那么我们接下去看

```
i = 0xffff;
n = (f = _[$[23]](o[3].charAt(o[3].length - 4), 3)) == o[3].substring(1, 4);
// f 是o[3]的倒数第4位重复3遍和o[3]234位相等
g = 3;
t = _[$[23]](o[3].charAt(3), 3) == o[3].substring(5, 8) && o[3].charCodeAt(1) * o[0].charCodeAt(0) == 0x2ef3;
//o[3]的第四位重复三遍和o[3]的678位相同，o[3]第2位的阿斯克码-2×o[0]第1位的阿斯克码==0x2ef3
```

这里我们拆解0x32ab得到119和101
得到两个字符分别为e和w,那么可以确定2位，但是不确定顺序

下面接着分析
```
i = 0xffff;
g = 3;
h = ((31249*g) & i).toString(16);
i = _[$[31]](o[3].split(f).join("").substring(0, 2)).split("x")[1];
s = i == h;
```
这里的s判断给了我们新的信息，因为h已知，所以我们就能得到o[3]的第一位和第五位，这么一来，o[3]所有的位我们都知道了是
`neee3eeed??`

现在我们知道了长度，可以肯定的是，o[3]肯定比o[1]长，那么基本可以得到o[1]了，回到刚才的逻辑

```
a = parseInt(_[$[23]]("1", Math.max(o[0].length, o[3].length)), 3) ^ eval(_[$[31]](o[0]));
```
这里我们得到的o[0]为`h3r3`, 特别的是，我们刚才得到o[0]第一位为w，这里有个小坑，由于`parseInt(_[$[23]]("1", Math.max(o[0].length, o[3].length)), 3)`过小，所以异或没有影响到所有的位，根据意思，我们加上了w，那么getflag

```
hctf{wh3r3_iz_y0ur_neee3eeed??}
```