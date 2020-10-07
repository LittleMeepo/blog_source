---
title: '[CSAPP] Bomb Lab'
date: 2020-10-6 0:41
tags:
- CSAPP
- Labs
categories:
- 编译
- 汇编
---

## Bomb Lab

传说中的bomb lab，我开始以为这名字意思是更改了程序汇编代码，让你修复...直到看了writeup...

一些预备知识，主要为[CSAPP第三方](https://wfc.ink/2020/10/06/csapp-pre-bomblab/)

> 题目大致意思就是不给源码（但给了源码框架），然你通过各种工具（gdb，objdump）调试，获取6次正确的输入。输入错误字符串会BOOMMMM !

部分源码：

```c
/* Do all sorts of secret stuff that makes the bomb harder to defuse. */
initialize_bomb();

printf("Welcome to my fiendish little bomb. You have 6 phases with\n");
printf("which to blow yourself up. Have a nice day!\n");

/* Hmm...  Six phases must be more secure than one phase! */
input = read_line();             /* Get input                   */
phase_1(input);                  /* Run the phase               */
phase_defused();                 /* Drat!  They figured it out!
                                  * Let me know how they did it. */
printf("Phase 1 defused. How about the next one?\n");
```

### phase_1

```bash
$objdump -d bomb > log.txt
```

main函数中调用phase_1的汇编代码：
```assembly
400e32:       e8 67 06 00 00          callq  40149e <read_line>;读入一串字符，地址存在%rax
400e37:       48 89 c7                mov    %rax,%rdi;%mov到%rdi
400e3a:       e8 a1 00 00 00          callq  400ee0 <phase_1>
400e3f:       e8 80 07 00 00          callq  4015c4 <phase_defused>
```

phase_1的汇编代码：

```assembly
0000000000400ee0 <phase_1>:
  400ee0:       48 83 ec 08             sub    $0x8,%rsp
  400ee4:       be 00 24 40 00          mov    $0x402400,%esi;字符常量地址存入%esi
  400ee9:       e8 4a 04 00 00          callq  401338 <strings_not_equal>
  400eee:       85 c0                   test   %eax,%eax;判断返回值是否为0
  400ef0:       74 05                   je     400ef7 <phase_1+0x17>
  400ef2:       e8 43 05 00 00          callq  40143a <explode_bomb>
  400ef7:       48 83 c4 08             add    $0x8,%rsp
  400efb:       c3                      retq
```

strings_not_equal的汇编代码：

```assembly
0000000000401338 <strings_not_equal>:
  401338:       41 54                   push   %r12
  40133a:       55                      push   %rbp
  40133b:       53                      push   %rbx
  40133c:       48 89 fb                mov    %rdi,%rbx;读入字符串地址
  40133f:       48 89 f5                mov    %rsi,%rbp;字符常量地址
  401342:       e8 d4 ff ff ff          callq  40131b <string_length>
  401347:       41 89 c4                mov    %eax,%r12d;求长度结果
  40134a:       48 89 ef                mov    %rbp,%rdi
  40134d:       e8 c9 ff ff ff          callq  40131b <string_length>
  401352:       ba 01 00 00 00          mov    $0x1,%edx;求长度结果
  401357:       41 39 c4                cmp    %eax,%r12d;比较长度，如不等长直接返回
  40135a:       75 3f                   jne    40139b <strings_not_equal+0x63>
  40135c:       0f b6 03                movzbl (%rbx),%eax
  40135f:       84 c0                   test   %al,%al
  401361:       74 25                   je     401388 <strings_not_equal+0x50>
  401363:       3a 45 00                cmp    0x0(%rbp),%al
  401366:       74 0a                   je     401372 <strings_not_equal+0x3a>
  401368:       eb 25                   jmp    40138f <strings_not_equal+0x57>
  40136a:       3a 45 00                cmp    0x0(%rbp),%al
  40136d:       0f 1f 00                nopl   (%rax)
  401370:       75 24                   jne    401396 <strings_not_equal+0x5e>
  401372:       48 83 c3 01             add    $0x1,%rbx;双字符串指针都+1判断是否相等
  401376:       48 83 c5 01             add    $0x1,%rbp
  40137a:       0f b6 03                movzbl (%rbx),%eax
  40137d:       84 c0                   test   %al,%al
  40137f:       75 e9                   jne    40136a <strings_not_equal+0x32>
  401381:       ba 00 00 00 00          mov    $0x0,%edx
  401386:       eb 13                   jmp    40139b <strings_not_equal+0x63>
  401388:       ba 00 00 00 00          mov    $0x0,%edx
  40138d:       eb 0c                   jmp    40139b <strings_not_equal+0x63>
  40138f:       ba 01 00 00 00          mov    $0x1,%edx
  401394:       eb 05                   jmp    40139b <strings_not_equal+0x63>
  401396:       ba 01 00 00 00          mov    $0x1,%edx
  40139b:       89 d0                   mov    %edx,%eax
  40139d:       5b                      pop    %rbx
  40139e:       5d                      pop    %rbp
  40139f:       41 5c                   pop    %r12
  4013a1:       c3                      retq
```

string_length的汇编代码：

```assembly
000000000040131b <string_length>:
  40131b:       80 3f 00                cmpb   $0x0,(%rdi);指针判空，空指针直接返回0
  40131e:       74 12                   je     401332 <string_length+0x17>
  401320:       48 89 fa                mov    %rdi,%rdx
  401323:       48 83 c2 01             add    $0x1,%rdx;每次地址+1，循环检测'\0'
  401327:       89 d0                   mov    %edx,%eax
  401329:       29 f8                   sub    %edi,%eax;返回的长度
  40132b:       80 3a 00                cmpb   $0x0,(%rdx)
  40132e:       75 f3                   jne    401323 <string_length+0x8>
  401330:       f3 c3                   repz retq
  401332:       b8 00 00 00 00          mov    $0x0,%eax
  401337:       c3                      retq
```

phase_1挺简单，就判断输入的字符串和给定的字符串常量是否相等，不相同则boom。输入的字符串起始地址为%rdi，给定的字符串起始地址为%esi，调用strings_not_equal判断是否相同。strings_not_equal首先调用string_length求字符串长度，不相同则直接返回，string_length就使用每次指针+1，检测所指内存区域是否为'\0'的方式求长度。如长度相等则再依次对比每个字符，不相等则直接返回。

所以，需要输入的字符串就是给定的字符串常量，位于内存0x402400，使用gdb打印出内存信息

```bash
$ gdb ./bomb
(gdb) b phase_1
Breakpoint 1 at 0x400ee0
(gdb) r
[随意的错误输入]
Breakpoint 1, 0x0000000000400ee0 in phase_1 ()
(gdb) print (char*) 0x402400
$1 = 0x402400 "Border relations with Canada have never been better."
```

答案是 Border relations with Canada have never been better.

### phase_2

phase_2的汇编代码：

```assembly
0000000000400efc <phase_2>:
  400efc:       55                      push   %rbp
  400efd:       53                      push   %rbx
  400efe:       48 83 ec 28             sub    $0x28,%rsp
  400f02:       48 89 e6                mov    %rsp,%rsi
  400f05:       e8 52 05 00 00          callq  40145c <read_six_numbers>
  400f0a:       83 3c 24 01             cmpl   $0x1,(%rsp);和1比较
  400f0e:       74 20                   je     400f30 <phase_2+0x34>
  400f10:       e8 25 05 00 00          callq  40143a <explode_bomb>
  400f15:       eb 19                   jmp    400f30 <phase_2+0x34>
  400f17:       8b 43 fc                mov    -0x4(%rbx),%eax;前一个单元的值
  400f1a:       01 c0                   add    %eax,%eax;前一个单元的值*2
  400f1c:       39 03                   cmp    %eax,(%rbx);前一个单元的值*2和当前单元比较
  400f1e:       74 05                   je     400f25 <phase_2+0x29>
  400f20:       e8 15 05 00 00          callq  40143a <explode_bomb>
  400f25:       48 83 c3 04             add    $0x4,%rbx
  400f29:       48 39 eb                cmp    %rbp,%rbx
  400f2c:       75 e9                   jne    400f17 <phase_2+0x1b>
  400f2e:       eb 0c                   jmp    400f3c <phase_2+0x40>
  400f30:       48 8d 5c 24 04          lea    0x4(%rsp),%rbx;将%rsp所指单元的上一个单元地址传%rbx
  400f35:       48 8d 6c 24 18          lea    0x18(%rsp),%rbp;将%rsp+24所指单元的地址传%rbx，作为循环结束条件
  400f3a:       eb db                   jmp    400f17 <phase_2+0x1b>
  400f3c:       48 83 c4 28             add    $0x28,%rsp
  400f40:       5b                      pop    %rbx
  400f41:       5d                      pop    %rbp
  400f42:       c3                      retq
```

read_six_numbers的汇编代码：

```assembly
000000000040145c <read_six_numbers>:
  40145c:       48 83 ec 18             sub    $0x18,%rsp
  401460:       48 89 f2                mov    %rsi,%rdx;存的就是phase_2栈帧底部的地址
  401463:       48 8d 4e 04             lea    0x4(%rsi),%rcx;传地址参数到寄存器
  401467:       48 8d 46 14             lea    0x14(%rsi),%rax
  40146b:       48 89 44 24 08          mov    %rax,0x8(%rsp)
  401470:       48 8d 46 10             lea    0x10(%rsi),%rax
  401474:       48 89 04 24             mov    %rax,(%rsp)
  401478:       4c 8d 4e 0c             lea    0xc(%rsi),%r9
  40147c:       4c 8d 46 08             lea    0x8(%rsi),%r8
  401480:       be c3 25 40 00          mov    $0x4025c3,%esi
  401485:       b8 00 00 00 00          mov    $0x0,%eax
  40148a:       e8 61 f7 ff ff          callq  400bf0 <__isoc99_sscanf@plt>
  40148f:       83 f8 05                cmp    $0x5,%eax;返回值：读取到的数量
  401492:       7f 05                   jg     401499 <read_six_numbers+0x3d>
  401494:       e8 a1 ff ff ff          callq  40143a <explode_bomb>
  401499:       48 83 c4 18             add    $0x18,%rsp
  40149d:       c3                      retq
```

phase_2主要是个循环，read_six_number将phase_2栈帧地址较低处6个int型内存单元的地址传给__isoc99_sscanf@plt，sscanff读取6个数到phase_2栈帧底部的6个单元，并返回读取到的int数量，存到%eax，如果数量小于6，则boom。

在phase_2的汇编代码中，首先将phase_2栈帧的最低地址单元与常数1相比，如果不等则boom，如果相等，则进行循环：用%rbx记录当前所指单元，将前一个单元的值的两倍和当前单元比较（第一个单元值为1），不相等则boom。当%rbx和%rbp相等时，循环结束。

<div align='center'> <img src="https://cdn.jsdelivr.net/gh/LittleMeepo/blog_images/images/csapp/20201007112329.png" width="250px"> </div>

### phase_3

