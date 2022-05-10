---
layout: post
title: 关于EOF的那点事
---

## 介绍

初学C++，C语言的程序员在CSDN、SegmentFault上提的最多的问题便是

> 如何在读取用户输入或读取文件时处理EOF（End-of-file，文件结尾）条件？

然而，几乎所有的此类问题都体现了提问者对EOF这个概念的完全误解。本文章试着解释这个令人困惑的问题。

## EOF字符

初学者对EOF的第一个误解就是“EOF字符” — 简单地说，总有人觉得EOF是一个字符，实际上“EOF字符”这玩意儿压根就**不存在**。

无论是Windows还是Linux都没有用来“标记文件结尾”的这么一个字符。用Windows记事本、Linux的Vim甚至任何操作系统的任何编辑器创建一个文本文件，你在其中不会找到任何的“EOF特殊标识符”。Windows和Linux的文件系统知道文件内容具体占了几个字节，所以根本不需要用字符标记出文件结尾。

![80年代一本杂志上CP/M系统的广告](https://img-blog.csdnimg.cn/20200418012108350.jpg)

那“EOF字符”这个概念又是从哪来的呢？原来，~~在很久很久以前，一个遥远的国度~~有一个Zilog Z80和英特尔8080等8位元处理器使用的操作系统，叫做CP/M。CP/M的文件系统并不知道一个文件具体有几字节，只知道这个文件占用了几个磁盘块。

比如这个文件：

```
hello world
```

系统只知道它占用了1个磁盘块（128字节）并不知道它的实际大小是11字节。由于许多程序必须知道文件的准确大小，它们采用了文件结尾标识符，追加在文件的结尾。他们重用了ASCII编码中的 `Ctrl-Z` 字符（26~10~，0x1A，原本用途不详）当作EOF字符 — 当CP/M程序读到 `Ctrl-Z` 字符时便会停止继续读取后面的内容。注意这只是程序开发者间的一个惯例，CP/M系统本身并不会对 `Ctrl-Z` 做特殊处理。且EOF字符不适用于二进制文件：读取二进制文件的程序有其他识别EOF的办法。

![DOSSHELL，文件资源管理器的雏形](https://img-blog.csdnimg.cn/20200421011806141.png)

后来MS-DOS出现，微软为了市场决定要把系统搞得尽量和CP/M兼容，于是他们把许多CP/M程序不重写源码，直接扔进了8080-8086机器码转换器。结果就是这些旧程序依然把 `Ctrl-Z` 当作EOF标识符，其中有不少存活到了今天。微软的C语言运行库就留存有这种对**文本文件**里 `Ctrl-Z` 的处理方式。不过还要注意，Windows作为一个操作系统不知道，也不关心 `Ctrl-Z` 的特殊意义：这个锅得找**几乎每个Windows程序都调用**的运行库来背。对了，Linux（和其他Unix系统）从来都没有以任何形式用过任何“EOF字符”。

### 范例代码

下面的C++代码可以展现出上述运行库的特性：

```cpp
#include <iostream>
#include <fstream>
using namespace std;

int main() {
    ofstream ofs( "myfile.txt" );
    ofs << "line 1\n";
    ofs << char(26);
    ofs << "line 2\n";
}
```

在Windows和Linux上运行这段代码，会创建一个文本文件，两行字中间夹着一个 `Ctrl-Z` 字符。按理说这两个系统都不会按照这个字符的特殊意义进行输出。

可在Windows命令行<sup>注1</sup> 里输出该文件内容时，只有第一行被打印了出来：

```
C:\Users\fengshuo\Desktop>type myfile.txt
line 1
```

而在Linux上，两行（加上中间的问号<sup>注2</sup> ）都被打印出来了：

```
fengshuo@fengshuo:/mnt/c/Users/fengshuo/Desktop$ cat myfile.txt
line 1
�line 2
```

您要是以为这个区别是Windows系统造成的，那您就错了。把文件用Windows记事本打开试试：

![notepad](https://img-blog.csdnimg.cn/20200412120157666.png)

搞什么鬼？怎么又不转义 `Ctrl-Z` 啦？

### 文本模式VS二进制模式

那么type命令和记事本又有什么本质上的区别呢？事实是：我也不知道。也许type命令会在文件里检查特殊字符。但可以肯定的是，Windows程序员在使用C++的iostream库和C语言的stream库时可以选择以文本模式或者二进制模式打开文件，这个选择会影响读到的内容。

这是C++读取文本文件的典型方法：

```cpp
#include <iostream>
#include <fstream>
#include <string>
using namespace std;

int main() {
    ifstream ifs( "myfile.txt" );
    string line;
    while( getline( ifs, line ) ) {
        cout << line << '\n';
    }
}
```

在Windows上运行这段代码，得到输出:

```
line 1
```

证明了 `Ctrl-Z` 的确会被当做EOF标识对待。不过改改之前代码，把文件用二进制模式打开：

```cpp
ifstream ifs( "myfile.txt", ios::binary );
```

结果输出：

```
line 1
�line 2
```

这说明 `Ctrl-Z` 只会在默认的文本模式中受到特殊对待 — 在二进制模式下会被像其他ASCII字符一样对待。此话仅适用于Windows，在Linux中以上两个程序的行为完全一致。

得到的总结有两点：

- 如果你想让你的可移植文件<sup>注3</sup> 以文本模式正确读取，不要往里嵌入 `Ctrl-Z` 字符
- 如果你的可移植文件里必须有 `Ctrl-Z` ，请确保程序以二进制模式打开它

### Linux的Ctrl-D

许多Linux用户表示“诶？那我用来结束shell输入的 `Ctrl-D` 呢？它不是EOF字符吗？” 抱歉还真不是。

你可以试试往文本文件里写入一个 `Ctrl-D` 再cat出来，Linux才不管这个字符呢。这个 `Ctrl-D` 只是个“快捷键”，是你给**shell终端**的一个信号，告诉它要关闭stdin流 — 也就是说，这个字符不会被送进程序的输入流 — 不仅如此，这个“快捷键”还能改！以下命令就把当前会话用来“结束输入”的快捷键从默认的 `Ctrl-D` 改为 `Ctrl-Y` ：

```bash
stty eof ^Y
```

## C++和C语言里的EOF

### EOF常量

再增加点奇怪的知识：C++和C语言里有个叫EOF的常量。EOF常量是在C语言的 \<stdio.h\> 和C++的 \<cstdio\> 里定义的：

```c
#define	EOF	(-1)
```

可以发现这个EOF跟 `Ctrl-Z` 一点关系都没有，它的值不是26而是-1，类型也不是char而是int。它用作getchar函数的返回值：

```cpp
int getchar(void);
```

这个getchar()函数用来一个字符一个字符地读取stdin，当遇到文件结尾时便会返回EOF常量。这个函数返回的是int：带符号int和char比大小，结果不一定像你想象的那样。getchar()的典型用法是像下面这样读stdin：

```cpp
#include <stdio.h>

int main() {
    int c;
    while( (c = getchar()) != EOF ) {
        putchar( c );
    }
}
```

### eof() 和 feof() 函数

最后一层程序员迷惑行为：C++和C提供了检查输入流状态的这两个布尔函数。所以我也顺便讲讲他们的正确用途：

> eof() 和 feof() 用来检查一个输入流是否遇到了End-of-File条件。这个条件只会在一次读取<sup>注4</sup> 以后出现。如果在任何情况下，这两个函数会于读操作发生之前被调用，**你写的代码有问题！** 永远不要在循环里使用EOF函数<sup>注5</sup> 。

为了说明这一点，咱们写一个“读取文本文件，并标上行号打印出来”的程序。许多新手会这样写：

```cpp
#include <iostream>
#include <fstream>
#include <string>
using namespace std;

int main() {
    ifstream ifs( "afile.txt" );
    int n = 0;
    while( ! ifs.eof() ) {
        string line;
        getline( ifs, line );
        cout << ++n << " " << line << '\n';
    }
}
```

这吒看着没啥毛病，但是请听我一句劝 — *如果eof()函数在读操作发生之前被调用，**你写的代码有问题！*** 想象一下如果afile.txt是空的会怎样：第一次调用eof()会因为还没有进行读操作而返回false，然后循环体被执行，输出一个行标为1的空行，最后再次调用eof()返回true，while条件为false则跳出循环。输出的那个空行并不存在于文件中。这个程序的问题就在于它**总在结尾输出一个多余的空行**。

正确的打开方式是，在读操作以后调用eof()，或者根本不调用。如果你的程序不需要检测除EOF以外的特殊情况，你可以直接这么写：

```cpp
int main() {
    ifstream ifs( "afile.txt" );
    int n = 0;
    string line;
    while( getline( ifs, line ) ) {
        cout << ++n << " " << line << '\n';
    }
}
```

这里用到了转换操作符，把getline()的的返回值（一个文件流对象）转换成布尔值 — 循环会重复执行直到文件流遇到End-of-File（或者其他错误）。

同理，下面这段C语言代码也是有问题的：

```c
#include <stdio.h>

int main() {
    FILE * f = fopen( "afile.txt", "r" );
    char line[100];
    int n = 0;
    while( ! feof( f ) ) {
        fgets( line, 100, f );
        printf( "%d %s", ++n, line );
    }
    fclose( f );
}
```

如果打开的文件是空的，肯定会导致程序打印出一大堆内存垃圾甚至出现其他不确定行为。应当写成：

```c
#include <stdio.h>

int main() {
    FILE * f = fopen( "afile.txt", "r" );
    char line[100];
    int n = 0;
    while( fgets( line, 100, f ) != NULL ) {
        printf( "%d %s", ++n, line );
    }
    fclose( f );
}
```

此时，eof() 和 feof() 函数表示自己很鸡肋，被包括在C++和C标准库里却没有什么用处。不对！如果你的程序要处理其他读取错误，这两个函数可以帮助把其他错误和EOF区分出来：

```cpp
#include <iostream>
#include <fstream>
#include <string>
using namespace std;

int main() {
    ifstream ifs( "afile.txt" );
    int n = 0;
    string line;
    while( getline( ifs, line ) ) {
        cout << ++n << " " << line << '\n';
    }
    if ( ifs.eof() ) {
        // 没有出现读取错误，只触发了EOF条件
    }
    else {
        // 出现了EOF以外的某种读取错误
    }
}
```

## 总结

整篇文章也许把EOF问题描述的过于复杂了，但你只需要记住三点：

- 没有所谓的EOF字符，除非你用文本模式在Windows上打开文件，或者在程序里自己实现一个
- C++和C语言里的EOF常量与ASCII字符无关，它被一些库函数引用为特殊返回值。 
- 不要把 eof() 或 feof() 放在循环里

希望这篇文章解答了您对EOF概念的困惑，也帮助您避免写出相关的bug！

---

注1：指的是cmd不是powershell，powershell的type命令和linux的cat一样

注2：之所以在这里打印出问号，是因为cat把 `Ctrl-Z` 像别的字符一样读取并送进了stdout流，但 `Ctrl-Z` 对应的ASCII码26又不是可打印字符，所以被你的终端替换成了“�”。

注3：能用任何系统上的任何程序打开的文件

注4：无论读取操作成功或失败

注5：实际上只要保证函数在循环的末尾执行就没问题，但为保险起见我建议您避免使用 eof() 和 feof()
