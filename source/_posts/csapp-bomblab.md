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

**答案**是 Border relations with Canada have never been better.

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

**答案**是 1 2 4 8 16 32

### phase_3

phase_3的汇编代码：

```assembly
0000000000400f43 <phase_3>:
  400f43:       48 83 ec 18             sub    $0x18,%rsp
  400f47:       48 8d 4c 24 0c          lea    0xc(%rsp),%rcx
  400f4c:       48 8d 54 24 08          lea    0x8(%rsp),%rdx
  400f51:       be cf 25 40 00          mov    $0x4025cf,%esi
  400f56:       b8 00 00 00 00          mov    $0x0,%eax
  400f5b:       e8 90 fc ff ff          callq  400bf0 <__isoc99_sscanf@plt>
  400f60:       83 f8 01                cmp    $0x1,%eax
  400f63:       7f 05                   jg     400f6a <phase_3+0x27>
  400f65:       e8 d0 04 00 00          callq  40143a <explode_bomb>
  400f6a:       83 7c 24 08 07          cmpl   $0x7,0x8(%rsp)
  400f6f:       77 3c                   ja     400fad <phase_3+0x6a>
  400f71:       8b 44 24 08             mov    0x8(%rsp),%eax
  400f75:       ff 24 c5 70 24 40 00    jmpq   *0x402470(,%rax,8)
  400f7c:       b8 cf 00 00 00          mov    $0xcf,%eax
  400f81:       eb 3b                   jmp    400fbe <phase_3+0x7b>
  400f83:       b8 c3 02 00 00          mov    $0x2c3,%eax
  400f88:       eb 34                   jmp    400fbe <phase_3+0x7b>
  400f8a:       b8 00 01 00 00          mov    $0x100,%eax
  400f8f:       eb 2d                   jmp    400fbe <phase_3+0x7b>
  400f91:       b8 85 01 00 00          mov    $0x185,%eax
  400f96:       eb 26                   jmp    400fbe <phase_3+0x7b>
  400f98:       b8 ce 00 00 00          mov    $0xce,%eax
  400f9d:       eb 1f                   jmp    400fbe <phase_3+0x7b>
  400f9f:       b8 aa 02 00 00          mov    $0x2aa,%eax
  400fa4:       eb 18                   jmp    400fbe <phase_3+0x7b>
  400fa6:       b8 47 01 00 00          mov    $0x147,%eax
  400fab:       eb 11                   jmp    400fbe <phase_3+0x7b>
  400fad:       e8 88 04 00 00          callq  40143a <explode_bomb>
  400fb2:       b8 00 00 00 00          mov    $0x0,%eax
  400fb7:       eb 05                   jmp    400fbe <phase_3+0x7b>
  400fb9:       b8 37 01 00 00          mov    $0x137,%eax
  400fbe:       3b 44 24 0c             cmp    0xc(%rsp),%eax
  400fc2:       74 05                   je     400fc9 <phase_3+0x86>
  400fc4:       e8 71 04 00 00          callq  40143a <explode_bomb>
  400fc9:       48 83 c4 18             add    $0x18,%rsp
  400fcd:       c3                      retq
```

phase_3主要内容为switch和跳转表，sscanf读取至少两个数，否则就boom。第一个参数在%rsp+8处，第二个参数在%rsp+12处。第一个参数不能大于7，否则就boom

0x402470处是跳转表的首地址，可以用gdb打印出跳转表：

```assembly
(gdb) print /x *0x402470 @16
$1 = {0x400f7c, 0x0, 0x400fb9, 0x0, 0x400f83, 0x0, 0x400f8a, 0x0, 0x400f91, 0x0,
  0x400f98, 0x0, 0x400f9f, 0x0, 0x400fa6, 0x0}
```

可以看出来，%rsp+8中不同的值，也就是%rax中不同的值对应着跳转表中不同的跳转地址，尝试使%rax等于1，则跳转到0x400fb9处指令，则第二个参数%rsp+12需要等于0x137才能使下一条cmp指令相等。

其中一个**答案**为1 311

### phase_4

phase_4的汇编代码：

```assembly
000000000040100c <phase_4>:
  40100c:       48 83 ec 18             sub    $0x18,%rsp
  401010:       48 8d 4c 24 0c          lea    0xc(%rsp),%rcx
  401015:       48 8d 54 24 08          lea    0x8(%rsp),%rdx
  40101a:       be cf 25 40 00          mov    $0x4025cf,%esi
  40101f:       b8 00 00 00 00          mov    $0x0,%eax
  401024:       e8 c7 fb ff ff          callq  400bf0 <__isoc99_sscanf@plt>
  401029:       83 f8 02                cmp    $0x2,%eax
  40102c:       75 07                   jne    401035 <phase_4+0x29>
  40102e:       83 7c 24 08 0e          cmpl   $0xe,0x8(%rsp)
  401033:       76 05                   jbe    40103a <phase_4+0x2e>
  401035:       e8 00 04 00 00          callq  40143a <explode_bomb>
  40103a:       ba 0e 00 00 00          mov    $0xe,%edx
  40103f:       be 00 00 00 00          mov    $0x0,%esi
  401044:       8b 7c 24 08             mov    0x8(%rsp),%edi
  401048:       e8 81 ff ff ff          callq  400fce <func4>
  40104d:       85 c0                   test   %eax,%eax
  40104f:       75 07                   jne    401058 <phase_4+0x4c>
  401051:       83 7c 24 0c 00          cmpl   $0x0,0xc(%rsp)
  401056:       74 05                   je     40105d <phase_4+0x51>
  401058:       e8 dd 03 00 00          callq  40143a <explode_bomb>
  40105d:       48 83 c4 18             add    $0x18,%rsp
  401061:       c3                      retq
```

func4的汇编代码：

```assembly
0000000000400fce <func4>:
  400fce:       48 83 ec 08             sub    $0x8,%rsp
  400fd2:       89 d0                   mov    %edx,%eax
  400fd4:       29 f0                   sub    %esi,%eax
  400fd6:       89 c1                   mov    %eax,%ecx
  400fd8:       c1 e9 1f                shr    $0x1f,%ecx
  400fdb:       01 c8                   add    %ecx,%eax
  400fdd:       d1 f8                   sar    %eax
  400fdf:       8d 0c 30                lea    (%rax,%rsi,1),%ecx
  400fe2:       39 f9                   cmp    %edi,%ecx
  400fe4:       7e 0c                   jle    400ff2 <func4+0x24>
  400fe6:       8d 51 ff                lea    -0x1(%rcx),%edx
  i00fe9:       e8 e0 ff ff ff          callq  400fce <func4>
  400fee:       01 c0                   add    %eax,%eax
  400ff0:       eb 15                   jmp    401007 <func4+0x39>
  400ff2:       b8 00 00 00 00          mov    $0x0,%eax
  400ff7:       39 f9                   cmp    %edi,%ecx
  400ff9:       7d 0c                   jge    401007 <func4+0x39>
  400ffb:       8d 71 01                lea    0x1(%rcx),%esi
  400ffe:       e8 cb ff ff ff          callq  400fce <func4>
  401003:       8d 44 00 01             lea    0x1(%rax,%rax,1),%eax
  401007:       48 83 c4 08             add    $0x8,%rsp
  40100b:       c3                      retq
```

