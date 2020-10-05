---
title: CSAPP Data Lab
date: 2020-10-5 16:52
tags: 
- CSAPP
- Labs
categories: 
- 编译
---

## Data Lab

【踩的坑】在centos7下进行的实验，缺32位gcc库

```bash
[wangfangcao@wfcserver datalab-handout]$ make
gcc -O -Wall -m32 -lm -o btest bits.c btest.c decl.c tests.c
In file included from /usr/include/features.h:399:0,
                 from /usr/include/stdio.h:27,
                 from btest.c:16:
/usr/include/gnu/stubs.h:7:27: fatal error: gnu/stubs-32.h: No such file or directory
 # include <gnu/stubs-32.h>
                           ^
compilation terminated.
```

解决办法

```bash
yum -y install glibc-devel.i686
```

```bash
[wangfangcao@wfcserver datalab-handout]$ make
gcc -O -Wall -m32 -lm -o btest bits.c btest.c decl.c tests.c
/usr/bin/ld: skipping incompatible /usr/lib/gcc/x86_64-redhat-linux/4.8.5/libgcc_s.so when searching for -lgcc_s
/usr/bin/ld: cannot find -lgcc_s
collect2: error: ld returned 1 exit status
make: *** [btest] Error 1
```

解决办法

```bash
yum install -y libgcc.i686
```

