---
title: redis数据结构-1
date: 2020-4-30 22:56
tags: redis
categories: redis源码
---

## 简单动态字符串sds
sds 的源码主要在 sds.h 和 sds.c 中

``` c
/*
 * 保存字符串对象的结构
 */
struct sdshdr {
    
    // buf 中已占用空间的长度
    int len;

    // buf 中剩余可用空间的长度
    int free;

    // 数据空间
    char buf[];
};
```
其中 buf 数组是柔性数组，在分配的时候不占内存大小

sds.h 中还有两个 inline 的静态函数，用于返回实际保存的字符串长度和可用空间的字符串长度
``` c
/*
 * 返回 sds 实际保存的字符串的长度
 */
static inline size_t sdslen(const sds s) {
    struct sdshdr *sh = (void*)(s-(sizeof(struct sdshdr))); //buf地址往回sizeof(struct sdshdr))
    return sh->len;
}
```

``` c
/*
 * 返回 sds 可用空间的长度
 */
static inline size_t sdsavail(const sds s) {
    struct sdshdr *sh = (void*)(s-(sizeof(struct sdshdr)));
    return sh->free;
}
```
这边用了一个骚操作，用 buf 的数组指针减去8字节，就得到了结构体的初始位置，也就是结构体的指针：
<div align='center'>
  <img src="https://cdn.jsdelivr.net/gh/LittleMeepo/blog_images/images/redis/20200502162607.png" height="180px">
</div>
这个技巧在 redis 中多处用到

sds 主要接口函数声明：
``` c
// 根据给定的初始化字符串 init 和字符串长度 initlen 创建一个新的 sds T = O(N)
sds sdsnewlen(const void *init, size_t initlen);

// 根据给定字符串 init ，创建一个包含同样字符串的 sds T = O(N)
sds sdsnew(const char *init);

// 创建并返回一个只保存了空字符串 "" 的 sds T = O(1)
sds sdsempty(void);

// 返回 sds 实际保存的字符串的长度 T = O(1)
size_t sdslen(const sds s);

// 复制给定 sds 的副本 T = O(N)
sds sdsdup(const sds s);

// 释放给定的 sds T = O(N)
void sdsfree(sds s);

// 返回 sds 可用空间的长度 T = O(1)
size_t sdsavail(const sds s);

// 将 sds 扩充至指定长度，未使用的空间以 0 字节填充 T = O(N)
sds sdsgrowzero(sds s, size_t len);

// 将长度为 len 的字符串 t 追加到 sds 的字符串末尾 T = O(N)
sds sdscatlen(sds s, const void *t, size_t len);

// 将给定字符串 t 追加到 sds 的末尾 T = O(N)
sds sdscat(sds s, const char *t);

// 将另一个 sds 追加到一个 sds 的末尾 T = O(N)
sds sdscatsds(sds s, const sds t);

// 将字符串 t 的前 len 个字符复制到 sds s 当中，并在字符串的最后添加终结符 T = O(N)
sds sdscpylen(sds s, const char *t, size_t len);

// 将字符串复制到 sds 当中，覆盖原有的字符 T = O(N)
sds sdscpy(sds s, const char *t);
```

其中几个：
- **sdsnew**   
  根据给定字符串 init ，创建一个包含同样字符串的 sds
``` c
sds sdsnew(const char *init) {
    size_t initlen = (init == NULL) ? 0 : strlen(init);
    return sdsnewlen(init, initlen);
}
```
其中调用了：
- **sdsnewlen**   
  根据给定的初始化字符串 init 和字符串长度 initlen 创建一个新的
``` c
sds sdsnewlen(const void *init, size_t initlen) {

    struct sdshdr *sh;

    // 根据是否有初始化内容，选择适当的内存分配方式
    if (init) {
        // zmalloc 不初始化所分配的内存
        sh = zmalloc(sizeof(struct sdshdr)+initlen+1); // 结构体+buf数组+'\0'
    } else {
        // zcalloc 将分配的内存全部初始化为 0
        sh = zcalloc(sizeof(struct sdshdr)+initlen+1);
    }

    // 内存分配失败，返回
    if (sh == NULL) return NULL;

    // 设置初始化长度
    sh->len = initlen;
    // 新 sds 不预留任何空间
    sh->free = 0;
    // 如果有指定初始化内容，将它们复制到 sdshdr 的 buf 中
    // T = O(N)
    if (initlen && init)
        memcpy(sh->buf, init, initlen);
    // 以 \0 结尾
    sh->buf[initlen] = '\0';

    // 返回 buf 部分，而不是整个 sdshdr
    return (char*)sh->buf;
}
```
其中 zmalloc 和 zcalloc 为 redis 使用的内存管理工具，在 zmalloc.c 和 zmalloc.h 中定义

## 双端链表
双端链表 的源码主要在 adlist.h 和 adlist.c 中

双端链表节点：
``` c
/*
 * 双端链表节点
 */
typedef struct listNode {

    // 前置节点
    struct listNode *prev;

    // 后置节点
    struct listNode *next;

    // 节点的值
    void *value;

} listNode;
```

双端链表结构:
``` c
/*
 * 双端链表结构
 */
typedef struct list {

    // 表头节点
    listNode *head;

    // 表尾节点
    listNode *tail;

    // 节点值复制函数
    void *(*dup)(void *ptr);

    // 节点值释放函数
    void (*free)(void *ptr);

    // 节点值对比函数
    int (*match)(void *ptr, void *key);

    // 链表所包含的节点数量
    unsigned long len;

} list;
```

为了规范化的宏定义函数：
``` c
// 返回给定链表所包含的节点数量
// T = O(1)
#define listLength(l) ((l)->len)
// 返回给定链表的表头节点
// T = O(1)
#define listFirst(l) ((l)->head)
// 返回给定链表的表尾节点
// T = O(1)
#define listLast(l) ((l)->tail)
// 返回给定节点的前置节点
// T = O(1)
#define listPrevNode(n) ((n)->prev)
// 返回给定节点的后置节点
// T = O(1)
#define listNextNode(n) ((n)->next)
// 返回给定节点的值
// T = O(1)
#define listNodeValue(n) ((n)->value)

// 将链表 l 的值复制函数设置为 m
// T = O(1)
#define listSetDupMethod(l,m) ((l)->dup = (m))
// 将链表 l 的值释放函数设置为 m
// T = O(1)
#define listSetFreeMethod(l,m) ((l)->free = (m))
// 将链表的对比函数设置为 m
// T = O(1)
#define listSetMatchMethod(l,m) ((l)->match = (m))

// 返回给定链表的值复制函数
// T = O(1)
#define listGetDupMethod(l) ((l)->dup)
// 返回给定链表的值释放函数
// T = O(1)
#define listGetFree(l) ((l)->free)
// 返回给定链表的值对比函数
// T = O(1)
#define listGetMatchMethod(l) ((l)->match)
```

几个主要接口函数：
- **listCreate**  
  创建一个新的链表 T = O(1)
``` c
list *listCreate(void)
{
    struct list *list;

    // 分配内存
    if ((list = zmalloc(sizeof(*list))) == NULL)
        return NULL;

    // 初始化属性
    list->head = list->tail = NULL;
    list->len = 0;
    list->dup = NULL;
    list->free = NULL;
    list->match = NULL;

    return list;
}
```
- **listRelease**  
  释放整个链表，以及链表中所有节点
``` c
void listRelease(list *list)
{
    unsigned long len;
    listNode *current, *next;

    // 指向头指针
    current = list->head;
    // 遍历整个链表
    len = list->len;
    while(len--) {
        next = current->next;

        // 如果有设置值释放函数，那么调用它
        if (list->free) list->free(current->value);

        // 释放节点结构
        zfree(current);

        current = next;
    }

    // 释放链表结构
    zfree(list);
}
```

- **listAddNodeHead**  
  将一个包含有给定值指针 value 的新节点添加到链表的表头
``` c
list *listAddNodeHead(list *list, void *value)
{
    listNode *node;

    // 为节点分配内存
    if ((node = zmalloc(sizeof(*node))) == NULL)
        return NULL;

    // 保存值指针
    node->value = value;

    // 添加节点到空链表
    if (list->len == 0) {
        list->head = list->tail = node;
        node->prev = node->next = NULL;
    // 添加节点到非空链表
    } else {
        node->prev = NULL;
        node->next = list->head;
        list->head->prev = node;
        list->head = node;
    }

    // 更新链表节点数
    list->len++;

    return list;
}
```

在 redis 双端队列中使用了迭代器这一技巧

双端链表迭代器：
``` c
/*
 * 双端链表迭代器
 */
typedef struct listIter {

    // 当前迭代到的节点
    listNode *next;

    // 迭代的方向
    int direction;

} listIter;
```

- **listGetIterator**  
  为给定链表创建一个迭代器，之后每次对这个迭代器调用 listNext 都返回被迭代到的链表节点
``` c
listIter *listGetIterator(list *list, int direction)
{
    // 为迭代器分配内存
    listIter *iter;
    if ((iter = zmalloc(sizeof(*iter))) == NULL) return NULL;

    // 根据迭代方向，设置迭代器的起始节点
    if (direction == AL_START_HEAD)
        iter->next = list->head;
    else
        iter->next = list->tail;

    // 记录迭代方向
    iter->direction = direction;

    return iter;
}
```

- **listNext**  
  返回迭代器当前所指向的节点  
  这个函数其实有两个作用：返回当前迭代器指向的节点 + 使迭代器指向下一个节点（我刚开始还没明白只return了一个current，怎么把iter传出去，后来看了这个函数的使用才知道，因为函数传入的是指针，只要定义一个listIter类型的指针，一直使用该函数就能迭代了
``` c
listNode *listNext(listIter *iter)
{
    listNode *current = iter->next;

    if (current != NULL) {
        // 根据方向选择下一个节点
        if (iter->direction == AL_START_HEAD)
            // 保存下一个节点，防止当前节点被删除而造成指针丢失
            iter->next = current->next;
        else
            // 保存下一个节点，防止当前节点被删除而造成指针丢失
            iter->next = current->prev;
    }

    return current;
}
```

- **listDup**  
  复制整个链表  
  其中就使用了上述的listNext来进行迭代
``` c
list *listDup(list *orig)
{
    list *copy;
    listIter *iter;
    listNode *node;

    // 创建新链表
    if ((copy = listCreate()) == NULL)
        return NULL;

    // 设置节点值处理函数
    copy->dup = orig->dup;
    copy->free = orig->free;
    copy->match = orig->match;

    // 迭代整个输入链表
    iter = listGetIterator(orig, AL_START_HEAD);
    while((node = listNext(iter)) != NULL) {
        void *value;

        // 复制节点值到新节点
        if (copy->dup) {
            value = copy->dup(node->value);
            if (value == NULL) {
                listRelease(copy);
                listReleaseIterator(iter);
                return NULL;
            }
        } else
            value = node->value;

        // 将节点添加到链表
        if (listAddNodeTail(copy, value) == NULL) {
            listRelease(copy);
            listReleaseIterator(iter);
            return NULL;
        }
    }

    // 释放迭代器
    listReleaseIterator(iter);

    // 返回副本
    return copy;
}
```
  