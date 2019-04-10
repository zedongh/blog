---
layout: post
title:  "Java Class File"
categories: java jvm
---

# 1. Java虚拟机

&nbsp;&nbsp;&nbsp;&nbsp;学习Java语言那么久，一直都没有去学习Java虚拟机（Java Virtual Machine，JVM），一方面是觉得基础还是不够扎实，另一方面是觉得虚拟机方面的知识用不上。最近，编程语言方面学习了很多，终于开始涉及虚拟机方面的内容，结合Java虚拟机和Lua虚拟机，窥探虚拟机。

# 2. Java虚拟机规范

&nbsp;&nbsp;&nbsp;&nbsp;Java虚拟机规范<sup>[1]</sup>

## 2.1 字节码文件格式

&nbsp;&nbsp;&nbsp;&nbsp;`.java`文件编译成`.class`文件，本质上是将文本形式的Java代码变成更加紧凑的二进制格式字节码文件（bytecode file）。字节码文件格式遵从Java Virtual Machine Specification中的定义，如同网络协议一样。

&nbsp;&nbsp;&nbsp;&nbsp;例如：

{% highlight java %}
// HelloWorld.java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("hello world");
    }
}
{% endhighlight %}

&nbsp;&nbsp;&nbsp;&nbsp;编译成`HelloWorld.class`文件，可以利用查看其二进制：

TODO

Java Virtual Machine Specification 12th Edition中第4章给出Class文件的格式，规范中使用类`C`的`struct`格式定义，使用`u1`，`u2`，`u4`分别表示一个，两个以及四个字节大小的数据类型。

{% highlight C %}
ClassFile {
       u4             magic;
       u2             minor_version;
       u2             major_version;
       u2             constant_pool_count;
       cp_info        constant_pool[constant_pool_count-1];
       u2             access_flags;
       u2             this_class;
       u2             super_class;
       u2             interfaces_count;
       u2             interfaces[interfaces_count];
       u2             fields_count;
       field_info     fields[fields_count];
       u2             methods_count;
       method_info    methods[methods_count];
       u2             attributes_count;
       attribute_info attributes[attributes_count];
}
{% endhighlight %}

对应到`HelloWorld.class`字节码文件中，也就是：
TODO


## 2.2 Instruction Set

&nbsp;&nbsp;&nbsp;&nbsp;Java虚拟机采用的是堆栈机，非等长指令集，这里不得不提下指令集。

寄存器机与堆栈机器
等长指令集与非等长指令集

# 3. Reference 

[1] https://docs.oracle.com/javase/specs/jvms/se12/html/jvms-4.html#jvms-4.1