# Redis

Redis包含了六个主要的底层数据结构：`动态字符串` `链表` `字典` `跳跃表` `整数集合` `压缩列表`

通过底层数据结构实现了五个基本对象：`字符串` `列表` `哈希` `集合` `有序集合`

## 数据结构

底层数据结构：`动态字符串` `链表` `字典` `跳跃表` `整数集合` `压缩列表`

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

### 链表(list)
`链表(list)`提供高效的节点重排能力，以及顺序的节点访问，可以通过增删节点灵活调整链表长度

链表应用于`列表键`，在`列表键`元素数量比较多，或者元素成员是比较长的字符串时，Redis会使用`链表(list)`作为`列表键`的底层实现

链表由`listNode`和`list`机构在`src/adlist.h` 定义：

`listNode`结构如下:

```c
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;
```

`list`结构如下:

```c
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

### 字典(dict)

Redis中的字典使用哈希表作为底层实现，一个哈希表包含多个哈希节点，每个哈希表节点就保存了字典中的一个键值对。

`dictht`结构如下:
```c
//哈希表
typedef struct dictht {
    dictEntry **table;
    unsigned long size;
    unsigned long sizemask;
    unsigned long used;
} dictht;
```

`dictEntry`结构如下:
```c
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

`dict`结构如下:

```c
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

### 跳跃表(zskiplist)

`跳跃表(zskiplist)`是一个有序数据结构, 它通过维持多个指向其他节点的指针，从而达到快速访问节点的目的。
`跳跃表(zskiplist)`应用于`有序集合`，在有序集合元素数量比较多，或者元素成员是比较长的字符串时，Redis会使用`跳跃表(zskiplist)`作为`有序集合键`的底层实现

跳跃表`zskiplistNode`和`zskiplist`两个结构在`src/server.h`文件中定义：

`zskiplist`结构如下:

```c
typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
```

* `header` `tail`记录跳跃表表头节点和尾节点
* `length`记录跳跃表长度, 既跳跃表节点数量
* `level`记录跳跃表内, 层数最大的节点层数

`zskiplistNode`结构如下:

```c
typedef struct zskiplistNode {
    sds ele;
    double score;
    struct zskiplistNode *backward;
    struct zskiplistLevel {
        struct zskiplistNode *forward;//前进指针
        unsigned long span;//层的跨度
    } level[];
} zskiplistNode;
```

* `level[]`层
* `backward`后退指针
* `score`分值

### 整数集合
### 压缩列表

## 对象

Redis通过上述章节里介绍的数据结构，构建一个对象系统，包括`字符串` `列表` `哈希` `集合` `有序集合` 这五类对象，每一种对象都至少使用了一种数据结构来实现

在`src/server.h`文件可以看到`redisObject`结构:

```c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (least significant 8 bits frequency
                            * and most significant 16 bits access time). */
    int refcount;//引用计数
    void *ptr;//指向底层实现数据结构的指针
} robj;
```

### type 记录了对象的类型

type 记录了对象的类型: 

在`src/server.h`文件定义了`encoding`编码类型:

```c
/* The actual Redis Object */
#define OBJ_STRING 0    /* String object. */
#define OBJ_LIST 1      /* List object. */
#define OBJ_SET 2       /* Set object. */
#define OBJ_ZSET 3      /* Sorted set object. */
#define OBJ_HASH 4      /* Hash object. */
```

可以通过redis命令`TYPE`查看对象`key`的类型

```sh
127.0.0.1:6379[1]> TYPE A
string
```

#### encoding

encoding 记录了对象所使用的底层编码既上述章节介绍数据结构， 如 `embstr` `ziplist` `intset`

在`src/server.h`文件定义了`encoding`编码类型:

```c
/* Objects encoding. Some kind of objects like Strings and Hashes can be
 * internally represented in multiple ways. The 'encoding' field of the object
 * is set to one of this fields for this object. */
#define OBJ_ENCODING_RAW 0     /* Raw representation */
#define OBJ_ENCODING_INT 1     /* Encoded as integer */
#define OBJ_ENCODING_HT 2      /* Encoded as hash table */
#define OBJ_ENCODING_ZIPMAP 3  /* Encoded as zipmap */
#define OBJ_ENCODING_LINKEDLIST 4 /* No longer used: old list encoding. */
#define OBJ_ENCODING_ZIPLIST 5 /* Encoded as ziplist */
#define OBJ_ENCODING_INTSET 6  /* Encoded as intset */
#define OBJ_ENCODING_SKIPLIST 7  /* Encoded as skiplist */
#define OBJ_ENCODING_EMBSTR 8  /* Embedded sds string encoding */
#define OBJ_ENCODING_QUICKLIST 9 /* Encoded as linked list of ziplists */
#define OBJ_ENCODING_STREAM 10 /* Encoded as a radix tree of listpacks */
```

#### lru

lru 记录了对象最后一次被程序访问的时间

可以通过redis命令`OBJECT IDLETIME`查看对象`key`的空转时长，既当前的时间减去`lru`的时间

```sh
127.0.0.1:6379[1]> OBJECT IDLETIME A
(integer) 604
```




同时可通过`OBJECT ENCODING`命令查看对象`key`的底层数据结构

```sh
127.0.0.1:6379[1]> OBJECT ENCODING user_score_rank
"ziplist"
127.0.0.1:6379[1]> set A string
OK
127.0.0.1:6379[1]> OBJECT ENCODING A
"embstr"
```



### 字符串对象
### 列表对象
### 哈希对象
### 集合对象
### 有序集合对象

## 内存回收

由于C语言不具备自动内存回收功能，所以`redis`在自身的对象系统中构建了`应用计数(refcount)`来实现内存的回收机制，通过`refcount`记录跟踪对象的引用计数，实现在适当时机的自动释放对象和内存回收。

在`src/server.h`文件可以看到`redisObject`结构:

```c
typedef struct redisObject {
    unsigned type:4;
    unsigned encoding:4;
    unsigned lru:LRU_BITS; /* LRU time (relative to global lru_clock) or
                            * LFU data (least significant 8 bits frequency
                            * and most significant 16 bits access time). */
    int refcount;
    void *ptr;
} robj;
```

`应用计数(refcount)`会随着对象的使用状态而不断变化

    创建对象时，引用计数会被初始化为 1
    对象没一个程序使用时，引用计数会被增加 1
    对象不被一个程序使用时，引用计数会被减 1
    对象的引用计数变为 0 时，对象所占用内存会被释放

通过redis命令`OBJECT REFCOUNT`可以查看对象的引用计数

```sh
127.0.0.1:6379> set A iscod
OK
127.0.0.1:6379> OBJECT REFCOUNT A
(integer) 1
```

### 对象共享

Redis在内存处理上除了`refcount`引用计数之外，还设计了对象共存

Redis在初始化服务器时，创建了一万个字符串对象，这些对象包含了从 0 到 9999 的所有整数值，当有其它程序或对象需要使用到这些字符串对象时，服务器就会共享这些对象，而不是新创建对象。

共享字符串的对象数量由`server.h`中`OBJ_SHARED_INTEGERS`指定

```c
#define OBJ_SHARED_INTEGERS 10000
```

如以下例子：

```sh
127.0.0.1:6379[1]> set A 1
OK
127.0.0.1:6379[1]> OBJECT REFCOUNT A
(integer) 2147483647
127.0.0.1:6379[1]> set A 100001
OK
127.0.0.1:6379[1]> OBJECT REFCOUNT A
(integer) 1
```

对象`A` 的键值为 `1`时, `A`的对象引用计数是 `1`。当A的键值设置 `100001` 时，引用计数为 `1`

## 集群

Redis集群是由多个redis节点组成的数据共享的程序集（最少三个节点）

### 数据分片

Redis集群没有使用一致性hash，而是使用哈希槽的该概念。

Redis 集群有16384个哈希槽,每个key通过CRC16校验后对16384取模来决定放置哪个槽.集群的每个节点负责一部分hash槽,举个例子,比如当前集群有3个节点,那么:

节点 A 包含 0 到 5500号哈希槽.
节点 B 包含5501 到 11000 号哈希槽.
节点 C 包含11001 到 16384号哈希槽.

* 参考
    * [Redis集群](http://www.redis.cn/topics/cluster-tutorial)
    * [Redis设计与实现](https://www.bookstack.cn/read/redisbook/2d294542c86f1acf.md)

