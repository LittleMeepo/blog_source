---
title: '[CSAPP] 机器表示'
date: 2020-10-6 16:09
tags: 
- CSAPP
categories: 
- 编译
- 汇编
---

> 记录一些容易忘的...x86-64下...

- 寄存器：程序计数器PC(%rsp) + 整数寄存器(16*64bit) + 条件码寄存器 + 一组向量寄存器

<div align='center'> <img src="https://cdn.jsdelivr.net/gh/LittleMeepo/blog_images/images/csapp/20201006162747.png" width="500px"> </div>

生成1字节和2字节的指令会保持剩下的字节不变；生成4字节数字的指令会把高位4个字节置为0

- 寻址方式

<div align='center'> <img src="https://cdn.jsdelivr.net/gh/LittleMeepo/blog_images/images/csapp/20201006164718.png" width="500px"> </div>

- 压栈和出栈指令

<div align='center'> <img src="https://cdn.jsdelivr.net/gh/LittleMeepo/blog_images/images/csapp/20201006164337.png" width="500px"> </div>

```assembly
pushq %rbp
```

等同于

```assembly
subq $8,%rsp;栈指针减
movq %rbp,(%rsp);store
```

- lea 加载有效地址

  目的：将有效地址写入到目的操作数，**根本就没有引用内存**

  leaq 7(%rdx,%rdx,4),%rax  如%rdx为x，则将%rax设置为5x+7

- 过程
  - 传递控制：进入Q时，PC被设置为Q代码的起始地址，返回时，PC设置为P中调用Q后的指令
  - 传递数据：P必须向Q提供一个或多个参数，Q必须能够向P返回一个值
  - 分配和释放内存：开始时Q会分配局部空间，返回前必须释放这些空间

- 运行时栈
  
  - 通过寄存器传参最多只能6个，超过6个需要P在自己的栈帧里存储好这些参数

<div align='center'> <img src="https://cdn.jsdelivr.net/gh/LittleMeepo/blog_images/images/csapp/20201006170142.png" width="500px"> </div>

- 转移控制

  - call Q

    把返回地址压入栈，并把PC设为Q的起始地址

  - ret

    会从栈中弹出地址A，并把PC设置为返回地址（call指令的下一条）

- 数据传送

x86-64中，可以通过寄存器最多传递6个整型参（例如整数和指针）参数。寄存器的使用是有特殊顺序的，名字取决于数据类型大小

<div align='center'> <img src="https://cdn.jsdelivr.net/gh/LittleMeepo/blog_images/images/csapp/20201006171042.png" width="500px"> </div>

如果参数大于6，则把参数7~n放到栈上，**参数7位于栈顶**。通过栈传递参数时，所有的数据大小都向8对齐

- 栈上的局部存储

  有时候，局部数据必须存放在内存里：

  - 寄存器不足够存放所有的本地数据
  - 对一个局部变量使用地址运算符'&'，因此必须能够为它产生一个地址
  - 某些局部变量是数组或结构，因此必须能够通过数组或结构引用被访问到

- 寄存器中的局部存储空间
  - 被调用者保存寄存器：%rbx,%rbp和%r12~%r15 过程Q保存一个寄存器的值不变，要么就是根本不去改变它，要么就是把原始值压入栈中，改了值，然后在返回之前从栈中弹出旧值
  - 调用者保存寄存器：其他除了%rsp 过程Q可以随意修改这个寄存器，因为在调用之前保存是P的责任

- 数据对齐

<div align='center'> <img src="https://cdn.jsdelivr.net/gh/LittleMeepo/blog_images/images/csapp/20201006214508.png" width="500px"> </div>