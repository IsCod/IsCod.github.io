# Redis

## 数据结构与对象

Redis包含了六个主要的底层数据结构：`动态字符串` `链表` `字典` `跳跃表` `整数集合` `压缩列表`

通过底层数据结构实现了五个基本对象：`字符串` `列表` `哈希` `集合` `有序集合`

### 动态字符串

动态字符串在`src/sds.h` 定义：
```c
struct __attribute__ ((__packed__)) sdshdr8 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```
* `len`记录sds已使用的字节数量
* `buf`用于保存字符串的字节数组 

### 链表

链表结构在`src/adlist.h` 定义：

```c
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct listIter {
    listNode *next;
    int direction;
} listIter;

typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);//节点释放函数
    int (*match)(void *ptr, void *key);
    unsigned long len;//链表包含的节点数量
} list;
```

* `listNode`定义了链表节点
* `list`定义了链表结构

### 字典

Redis中的字典使用哈希表作为底层实现，一个哈希表包含多个哈希节点，每个哈希表节点就保存了字典中的一个键值对。

哈希表和字典都在`src/dict.h` 定义：
```c
//哈希表
typedef struct dictht {
    dictEntry **table;
    unsigned long size;
    unsigned long sizemask;
    unsigned long used;
} dictht;

//哈希节点
typedef struct dictEntry {
    void *key;
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;
} dictEntry;
```

```c
//字典
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```




### 跳跃表
### 整数集合
### 压缩列表

### 对象
- 字符串对象
- 列表对象
- 哈希对象
- 集合对象
- 有序集合对象
### 内存回收

