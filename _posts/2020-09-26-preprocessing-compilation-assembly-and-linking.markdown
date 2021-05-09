---
layout: post
title:  "Preprocessing Compilation Assembly and Linking"
categories: c compiler link
date: 2020-09-26 00:12:00 +0000
---

<figure class="image">
  <img src="{{site.baseurl}}/images/compile.png" alt="preprocessing compilation assembly and linking">
  <figcaption>preprocessing compilation assembly and linking</figcaption>
</figure>

# I. 从一行命令说起

{% highlight c %}
$ gcc main.c add.c mul.c
$ ./a.out
max: 3.141593, add: 5.859874, sub: 0.423311, mul: 8.539734, div: 1.155727
{% endhighlight %}

这一行命令涉及到的流程包括：预处理、编译、汇编和链接，最后生成可执行的文件。

# II. Preprocessing

预处理(preprocessing)是c语言整个编译流程的第一个阶段，将源代码(Source Code)处理成翻译单元(Translation Unit)。C语言代码中以‘#’开头是预处理指令, 这里是一个完整的包含了一些简单的预处理指令的C项目工程(`arith.h`, `add.c`, `mul.c`, `main.c`):

* `arith.h`是算数库头文件，声明了`add`, `sub`, `mul`以及`div`四个函数。
{% highlight c %}
#pragma once

double add(double a, double b);
double mul(double a, double b);
double sub(double a, double b);
double div(double a, double b);
{% endhighlight %}

* `add.c`实现了算数库的`add`和`sub`函数
{% highlight c %}
#include "arith.h"

double add(double a, double b) {
    return a + b;
}

double sub(double a, double b) {
    return a - b;
}
{% endhighlight %}

* `mul.c`实现了算数库的`mul`和`div`函数
{% highlight c %}
#include "arith.h"

double mul(double a, double b) {
    return a * b;
}

double div(double a, double b) {
    return a / b;
}
{% endhighlight %}

* `main.c`作为工程入口主文件，定义了`main`函数以及一些常见的宏
{% highlight c %}
#include <stdio.h>

#define PI 3.14159265359

#define MAX(a, b) ((a) > (b)) ? (a) : (b)

#ifndef E
#define E 2.718281828
#endif

int main(int argc, char const *argv[])
{
    printf("max: %f, add: %f, sub: %f, mul: %f, div: %f\n", 
           MAX(PI, E), add(PI, E), sub(PI, E), mul(PI, E), div(PI, E));
    return 0;
}
{% endhighlight %}

为了手动模拟GCC编译器的第一步处理，使用`cpp`命令可以对工程的每个`.c`文件进行预处理：
{% highlight bash %}
$ cpp add.c -c add1.c
$ cpp mul.c -c mul1.c
$ cpp main.c -c main1.c
{% endhighlight %}
为了看得更加清楚中间发生了什么，这里列出每个文件的预处理后的输出，由于`#include<stdio.h>`太长，省略了`main.c`的预处理输出部分片段。
* `add1.c`

{% highlight c %}
# 1 "add.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 31 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 32 "<command-line>" 2
# 1 "add.c"
# 1 "arith.h" 1
       

double add(double a, double b);
double mul(double a, double b);
double sub(double a, double b);
double div(double a, double b);
# 2 "add.c" 2

double add(double a, double b) {
    return a + b;
}

double sub(double a, double b) {
    return a - b;
}
{% endhighlight %}

* `mul1.c`

{% highlight c %}
# 1 "mul.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 31 "<command-line>"
# 1 "/usr/include/stdc-predef.h" 1 3 4
# 32 "<command-line>" 2
# 1 "mul.c"
# 1 "arith.h" 1
       

double add(double a, double b);
double mul(double a, double b);
double sub(double a, double b);
double div(double a, double b);
# 2 "mul.c" 2

double mul(double a, double b) {
    return a * b;
}

double div(double a, double b) {
    return a / b;
}
{% endhighlight %}

* `main1.c`

{% highlight c %}
...
# 873 "/usr/include/stdio.h" 3 4

# 2 "main.c" 2

# 1 "arith.h" 1
       


# 3 "arith.h"
double add(double a, double b);
double mul(double a, double b);
double sub(double a, double b);
double div(double a, double b);
# 4 "main.c" 2
# 13 "main.c"
int main(int argc, char const *argv[])
{
    printf("max: %f, add: %f, sub: %f, mul: %f, div: %f\n",
           ((3.14159265359) > (2.718281828)) ? (3.14159265359) : (2.718281828), add(3.14159265359, 2.718281828), sub(3.14159265359, 2.718281828), mul(3.14159265359, 2.718281828), div(3.14159265359, 2.718281828));
    return 0;
}
{% endhighlight %}

可以看出，使用`#include`包含的头文件，在预处理阶段会将其中的内容预处理后包含进入处理结果中，`#define`的常量和宏会替换源文件中对应出现的地方。

值得一提的是，条件编译`#ifndef`，`#ifdef`需要根据一些预定义判断条件选择处理分支，也可以通过命令行`-D`显示指定条件影响预处理流程：
{% highlight bash %}
$ cpp main.c -D E=2 -o main2.c
{% endhighlight %}
通过命令行显式定义`E`，影响`ifndef E`条件，进而影响预处理的结果：
{% highlight c %}
...
# 4 "main.c" 2
# 13 "main.c"
int main(int argc, char const *argv[])
{
    printf("max: %f, add: %f, sub: %f, mul: %f, div: %f\n",
           ((3.14159265359) > (2)) ? (3.14159265359) : (2), add(3.14159265359, 2), sub(3.14159265359, 2), mul(3.14159265359, 2), div(3.14159265359, 2));
    return 0;
}
{% endhighlight %}

处理后的结果中以‘#’开后的行，类似于注释的作用，不再是预处理指令。

> gcc -E pipeline1.c  # 同样也可以看到预处理后结果

# III. Compilation

经过预处理器得到的翻译单元(pipeline2.c)作为编译的输入，得到相应的汇编代码。通过gcc的`-S`选项，会生成对应的`.s`后缀(pipeline2.s)的汇编代码。
{% highlight bash %}
$ gcc -S add1.c
$ gcc -S mul1.c
$ gcc -S main1.c
{% endhighlight %}
编译器解析翻译单元，将其翻译成目标架构平台的汇编代码
* `add1.s`
{% highlight asm %}
	.file	"add1.c"
	.text
	.globl	add
	.type	add, @function
add:
.LFB0:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movsd	%xmm0, -8(%rbp)
	movsd	%xmm1, -16(%rbp)
	movsd	-8(%rbp), %xmm0
	addsd	-16(%rbp), %xmm0
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	add, .-add
	.globl	sub
	.type	sub, @function
sub:
.LFB1:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movsd	%xmm0, -8(%rbp)
	movsd	%xmm1, -16(%rbp)
	movsd	-8(%rbp), %xmm0
	subsd	-16(%rbp), %xmm0
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1:
	.size	sub, .-sub
	.ident	"GCC: (Ubuntu 9.3.0-10ubuntu2) 9.3.0"
	.section	.note.GNU-stack,"",@progbits
	.section	.note.gnu.property,"a"
	.align 8
	.long	 1f - 0f
	.long	 4f - 1f
	.long	 5
0:
	.string	 "GNU"
1:
	.align 8
	.long	 0xc0000002
	.long	 3f - 2f
2:
	.long	 0x3
3:
	.align 8
4:
{% endhighlight %}
* `mul1.s`
{% highlight asm %}
	.file	"mul1.c"
	.text
	.globl	mul
	.type	mul, @function
mul:
.LFB0:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movsd	%xmm0, -8(%rbp)
	movsd	%xmm1, -16(%rbp)
	movsd	-8(%rbp), %xmm0
	mulsd	-16(%rbp), %xmm0
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	mul, .-mul
	.globl	div
	.type	div, @function
div:
.LFB1:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	movsd	%xmm0, -8(%rbp)
	movsd	%xmm1, -16(%rbp)
	movsd	-8(%rbp), %xmm0
	divsd	-16(%rbp), %xmm0
	popq	%rbp
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE1:
	.size	div, .-div
	.ident	"GCC: (Ubuntu 9.3.0-10ubuntu2) 9.3.0"
	.section	.note.GNU-stack,"",@progbits
	.section	.note.gnu.property,"a"
	.align 8
	.long	 1f - 0f
	.long	 4f - 1f
	.long	 5
0:
	.string	 "GNU"
1:
	.align 8
	.long	 0xc0000002
	.long	 3f - 2f
2:
	.long	 0x3
3:
	.align 8
4:
{% endhighlight %}
* `main1.s`
{% highlight asm %}
	.file	"main1.c"
	.text
	.section	.rodata
	.align 8
.LC2:
	.string	"max: %f, add: %f, sub: %f, mul: %f, div: %f\n"
	.text
	.globl	main
	.type	main, @function
main:
.LFB0:
	.cfi_startproc
	endbr64
	pushq	%rbp
	.cfi_def_cfa_offset 16
	.cfi_offset 6, -16
	movq	%rsp, %rbp
	.cfi_def_cfa_register 6
	subq	$48, %rsp
	movl	%edi, -4(%rbp)
	movq	%rsi, -16(%rbp)
	movsd	.LC0(%rip), %xmm0
	movq	.LC1(%rip), %rax
	movapd	%xmm0, %xmm1
	movq	%rax, %xmm0
	call	div@PLT
	movsd	%xmm0, -24(%rbp)
	movsd	.LC0(%rip), %xmm0
	movq	.LC1(%rip), %rax
	movapd	%xmm0, %xmm1
	movq	%rax, %xmm0
	call	mul@PLT
	movsd	%xmm0, -32(%rbp)
	movsd	.LC0(%rip), %xmm0
	movq	.LC1(%rip), %rax
	movapd	%xmm0, %xmm1
	movq	%rax, %xmm0
	call	sub@PLT
	movsd	%xmm0, -40(%rbp)
	movsd	.LC0(%rip), %xmm0
	movq	.LC1(%rip), %rax
	movapd	%xmm0, %xmm1
	movq	%rax, %xmm0
	call	add@PLT
	movq	.LC1(%rip), %rax
	movsd	-24(%rbp), %xmm4
	movsd	-32(%rbp), %xmm3
	movsd	-40(%rbp), %xmm2
	movapd	%xmm0, %xmm1
	movq	%rax, %xmm0
	leaq	.LC2(%rip), %rdi
	movl	$5, %eax
	call	printf@PLT
	movl	$0, %eax
	leave
	.cfi_def_cfa 7, 8
	ret
	.cfi_endproc
.LFE0:
	.size	main, .-main
	.section	.rodata
	.align 8
.LC0:
	.long	2332332443
	.long	1074118410
	.align 8
.LC1:
	.long	1413754602
	.long	1074340347
	.ident	"GCC: (Ubuntu 9.3.0-10ubuntu2) 9.3.0"
	.section	.note.GNU-stack,"",@progbits
	.section	.note.gnu.property,"a"
	.align 8
	.long	 1f - 0f
	.long	 4f - 1f
	.long	 5
0:
	.string	 "GNU"
1:
	.align 8
	.long	 0xc0000002
	.long	 3f - 2f
2:
	.long	 0x3
3:
	.align 8
4:
{% endhighlight %}

# IV. Assembly

流程的下一个处理阶段是汇编器——根据上一步处理的汇编代码(pipeline2.s)生成机器指令（机器码），通常将包含这样的机器指令的文件称为目标文件(Object File)。

{% highlight bash %}
$ as add1.s -o add.o
$ as mul1.s -o mul.o
$ as main1.s -o main.o
{% endhighlight %}

可重定位目标文件是二进制文件，Linux下有很多方便的工具查看其中的内容信息。

* 使用`nm`查看目标文件中的符号表

{% highlight bash %}
$ nm add1.o
0000000000000000 T add
000000000000001e T sub
$ nm mul1.o
000000000000001e T div
0000000000000000 T mul
$ nm main1.o
                 U _GLOBAL_OFFSET_TABLE_
                 U add
                 U div
0000000000000000 T main
                 U mul
                 U printf
                 U sub
{% endhighlight %}

可以看到`nm`的输出中`main.o`，存在未定义的符号

* 除了可以使用`nm`也可以使用`readelf`命令查看 

{% highlight bash %}
$ readelf -s add1.o

Symbol table '.symtab' contains 11 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS add1.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    2 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
     8: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     9: 0000000000000000    30 FUNC    GLOBAL DEFAULT    1 add
    10: 000000000000001e    30 FUNC    GLOBAL DEFAULT    1 sub
$ readelf -s mul1.o

Symbol table '.symtab' contains 11 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS mul1.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    2 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
     8: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     9: 0000000000000000    30 FUNC    GLOBAL DEFAULT    1 mul
    10: 000000000000001e    30 FUNC    GLOBAL DEFAULT    1 div
$ readelf -s main1.o

Symbol table '.symtab' contains 17 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS main1.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    8 
     8: 0000000000000000     0 SECTION LOCAL  DEFAULT    9 
     9: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
    10: 0000000000000000   205 FUNC    GLOBAL DEFAULT    1 main
    11: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND _GLOBAL_OFFSET_TABLE_
    12: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND div
    13: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND mul
    14: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND sub
    15: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND add
    16: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND printf
{% endhighlight %}

* 使用`objdump`反汇编其中的机器指令

{% highlight bash %}
$ objdump -d add1.c

add1.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <add>:
   0:	f3 0f 1e fa          	endbr64 
   4:	55                   	push   %rbp
   5:	48 89 e5             	mov    %rsp,%rbp
   8:	f2 0f 11 45 f8       	movsd  %xmm0,-0x8(%rbp)
   d:	f2 0f 11 4d f0       	movsd  %xmm1,-0x10(%rbp)
  12:	f2 0f 10 45 f8       	movsd  -0x8(%rbp),%xmm0
  17:	f2 0f 58 45 f0       	addsd  -0x10(%rbp),%xmm0
  1c:	5d                   	pop    %rbp
  1d:	c3                   	retq   

000000000000001e <sub>:
  1e:	f3 0f 1e fa          	endbr64 
  22:	55                   	push   %rbp
  23:	48 89 e5             	mov    %rsp,%rbp
  26:	f2 0f 11 45 f8       	movsd  %xmm0,-0x8(%rbp)
  2b:	f2 0f 11 4d f0       	movsd  %xmm1,-0x10(%rbp)
  30:	f2 0f 10 45 f8       	movsd  -0x8(%rbp),%xmm0
  35:	f2 0f 5c 45 f0       	subsd  -0x10(%rbp),%xmm0
  3a:	5d                   	pop    %rbp
  3b:	c3                   	retq   

$ objdump -d mul1.c

mul1.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <mul>:
   0:	f3 0f 1e fa          	endbr64 
   4:	55                   	push   %rbp
   5:	48 89 e5             	mov    %rsp,%rbp
   8:	f2 0f 11 45 f8       	movsd  %xmm0,-0x8(%rbp)
   d:	f2 0f 11 4d f0       	movsd  %xmm1,-0x10(%rbp)
  12:	f2 0f 10 45 f8       	movsd  -0x8(%rbp),%xmm0
  17:	f2 0f 59 45 f0       	mulsd  -0x10(%rbp),%xmm0
  1c:	5d                   	pop    %rbp
  1d:	c3                   	retq   

000000000000001e <div>:
  1e:	f3 0f 1e fa          	endbr64 
  22:	55                   	push   %rbp
  23:	48 89 e5             	mov    %rsp,%rbp
  26:	f2 0f 11 45 f8       	movsd  %xmm0,-0x8(%rbp)
  2b:	f2 0f 11 4d f0       	movsd  %xmm1,-0x10(%rbp)
  30:	f2 0f 10 45 f8       	movsd  -0x8(%rbp),%xmm0
  35:	f2 0f 5e 45 f0       	divsd  -0x10(%rbp),%xmm0
  3a:	5d                   	pop    %rbp
  3b:	c3                   	retq   

$ objdump -d main1.c

main1.o:     file format elf64-x86-64


Disassembly of section .text:

0000000000000000 <main>:
   0:	f3 0f 1e fa          	endbr64 
   4:	55                   	push   %rbp
   5:	48 89 e5             	mov    %rsp,%rbp
   8:	48 83 ec 30          	sub    $0x30,%rsp
   c:	89 7d fc             	mov    %edi,-0x4(%rbp)
   f:	48 89 75 f0          	mov    %rsi,-0x10(%rbp)
  13:	f2 0f 10 05 00 00 00 	movsd  0x0(%rip),%xmm0        # 1b <main+0x1b>
  1a:	00 
  1b:	48 8b 05 00 00 00 00 	mov    0x0(%rip),%rax        # 22 <main+0x22>
  22:	66 0f 28 c8          	movapd %xmm0,%xmm1
  26:	66 48 0f 6e c0       	movq   %rax,%xmm0
  2b:	e8 00 00 00 00       	callq  30 <main+0x30>
  30:	f2 0f 11 45 e8       	movsd  %xmm0,-0x18(%rbp)
  35:	f2 0f 10 05 00 00 00 	movsd  0x0(%rip),%xmm0        # 3d <main+0x3d>
  3c:	00 
  3d:	48 8b 05 00 00 00 00 	mov    0x0(%rip),%rax        # 44 <main+0x44>
  44:	66 0f 28 c8          	movapd %xmm0,%xmm1
  48:	66 48 0f 6e c0       	movq   %rax,%xmm0
  4d:	e8 00 00 00 00       	callq  52 <main+0x52>
  52:	f2 0f 11 45 e0       	movsd  %xmm0,-0x20(%rbp)
  57:	f2 0f 10 05 00 00 00 	movsd  0x0(%rip),%xmm0        # 5f <main+0x5f>
  5e:	00 
  5f:	48 8b 05 00 00 00 00 	mov    0x0(%rip),%rax        # 66 <main+0x66>
  66:	66 0f 28 c8          	movapd %xmm0,%xmm1
  6a:	66 48 0f 6e c0       	movq   %rax,%xmm0
  6f:	e8 00 00 00 00       	callq  74 <main+0x74>
  74:	f2 0f 11 45 d8       	movsd  %xmm0,-0x28(%rbp)
  79:	f2 0f 10 05 00 00 00 	movsd  0x0(%rip),%xmm0        # 81 <main+0x81>
  80:	00 
  81:	48 8b 05 00 00 00 00 	mov    0x0(%rip),%rax        # 88 <main+0x88>
  88:	66 0f 28 c8          	movapd %xmm0,%xmm1
  8c:	66 48 0f 6e c0       	movq   %rax,%xmm0
  91:	e8 00 00 00 00       	callq  96 <main+0x96>
  96:	48 8b 05 00 00 00 00 	mov    0x0(%rip),%rax        # 9d <main+0x9d>
  9d:	f2 0f 10 65 e8       	movsd  -0x18(%rbp),%xmm4
  a2:	f2 0f 10 5d e0       	movsd  -0x20(%rbp),%xmm3
  a7:	f2 0f 10 55 d8       	movsd  -0x28(%rbp),%xmm2
  ac:	66 0f 28 c8          	movapd %xmm0,%xmm1
  b0:	66 48 0f 6e c0       	movq   %rax,%xmm0
  b5:	48 8d 3d 00 00 00 00 	lea    0x0(%rip),%rdi        # bc <main+0xbc>
  bc:	b8 05 00 00 00       	mov    $0x5,%eax
  c1:	e8 00 00 00 00       	callq  c6 <main+0xc6>
  c6:	b8 00 00 00 00       	mov    $0x0,%eax
  cb:	c9                   	leaveq 
  cc:	c3                   	retq   

{% endhighlight %}

# V. Linking

上一个阶段的输出目标文件中，存在一些符号没有解决，这时候就是链接任务了，链接是通过ld命令，将多个文件

{% highlight bash %}
$ ld main1.o
ld: warning: cannot find entry symbol _start; defaulting to 0000000000401000
ld: main1.o: in function `main':
main1.c:(.text+0x2c): undefined reference to `div'
ld: main1.c:(.text+0x4e): undefined reference to `mul'
ld: main1.c:(.text+0x70): undefined reference to `sub'
ld: main1.c:(.text+0x92): undefined reference to `add'
ld: main1.c:(.text+0xc2): undefined reference to `printf'
{% endhighlight %}

既然提示未定义的引用`div`, `mul`说明链接缺少这一部分，而我们知道这些符号在`mul.o`中，尝试吧`mul.o`添加到`ld`命令中：

{% highlight bash %}
$ ld main1.o mul1.o 
ld: warning: cannot find entry symbol _start; defaulting to 0000000000401000
ld: main1.o: in function `main':
main1.c:(.text+0x70): undefined reference to `sub'
ld: main1.c:(.text+0x92): undefined reference to `add'
ld: main1.c:(.text+0xc2): undefined reference to `printf'
{% endhighlight %}

未定义的符号引用减少了，说明思路对了，尝试向命令中继续添加`add1.o`，减少未定义的引用`sub`, `add`的报错。

{% highlight bash %}
$ ld main1.o mul1.o add1.o
ld: warning: cannot find entry symbol _start; defaulting to 0000000000401000
ld: main1.o: in function `main':
main1.c:(.text+0xc2): undefined reference to `printf'
{% endhighlight %}

`printf`是C标准库的函数，为了解决相关符号引用，主函数入口等错误，可以使用`gcc`完成最后链接工作

{% highlight bash %}
$ gcc main1.o mul1.o add1.o
$ ./a.out
max: 3.141593, add: 5.859874, sub: 0.423311, mul: 8.539734, div: 1.155727
{% endhighlight %}

## 1. 静态链接库(Static Linker Library)

静态链接库不过是一堆目标文件的归档打包。这里可以将`mul.o`, `add.o`组成基本四则算数库打包成静态链接库：

* 创建基本算数静态链接库`libarithmetic.a`

{% highlight bash %}
$ ar crs libarithmetic.a mul1.o add1.o
$ mkdir /opt/arithmetic
$ mv libarithmetic.a /opt/arithmetic
{% endhighlight %}

* 可以使用`ar -t`查看静态链接库中包含的目标文件
{% highlight bash %}
$ ar -t /opt/arithmetric/libarithmetic.a
add1.o
mul1.o
{% endhighlight %}

* 链接的时候使用`-L`指定查找链接库的目录和`-l`指定需要链接的库
{% highlight bash %}
$ gcc -o main main1.o -L/opt/arithmetric -larithmetic
$ ./main
max: 3.141593, add: 5.859874, sub: 0.423311, mul: 8.539734, div: 1.155727
{% endhighlight %}

## 2. 动态链接库(Dynamic Linker Library)

当有多个程序依赖于同一个库的时候，使用静态链接库的方式要求每个程序都需要链接该静态库，最后的可执行文件都包含了相同的库部分，在空间上造成浪费的同时给程序维护和升级造成不便。此时使用动态链接库可以帮助解决这样的问题。动态链接库又称共享库，`gcc`提供`-shared`选项创建动态链接库，需要先将依赖库的代码编译成位置无关机器码(Position-independent code, PIC)：

{% highlight bash %}
$ gcc -c add1.c -fPIC -o add2.o                        # -fPIC
$ gcc -c mul1.c -fPIC -o mul2.o
$ gcc -shared add1.o mul1.o -o libarithmetic.so        # so 表示为 shared objects
$ rm -rv /opt/arithmetic/ libarithmetic.a              # 删除之前的静态链接库
$ mv libarithmetic.so /opt/arithmetic
{% endhighlight %}

链接的时候与链接静态链接库的命令一样：

{% highlight bash %}
$ gcc main1.o -larithmetic -L/opt/arithmetic
$ ./a.out              
./a.out: error while loading shared libraries: libarithmetic.so: cannot open shared object file: No such file or directory
{% endhighlight %}

链接虽然成功了，但是加载运行的时候报`libarithmetic.so`没有找到的错误，这里不同于静态链接，静态链接后的产出的可执行文件是可以独立加载执行，不需要指定任何依赖；而动态链接后产出的可执行文件加载运行依赖于动态链接库，需要指定链接库的位置——可以通过环境变量`LD_LIBRARY_PATH`显式指定：
{% highlight bash %}
$ export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/opt/arithmetic
$ ./a.out
max: 3.141593, add: 5.859874, sub: 0.423311, mul: 8.539734, div: 1.155727
{% endhighlight%}

# VI. 参考

[1] [GCC Options Controlling the Preprocessor](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html#Preprocessor-Options)

[2] [GCC Options for Linking](https://gcc.gnu.org/onlinedocs/gcc/Link-Options.html#Link-Options)

[3] [GCC Options for Code Generation Conventions](https://gcc.gnu.org/onlinedocs/gcc/Code-Gen-Options.html#Code-Gen-Options)
