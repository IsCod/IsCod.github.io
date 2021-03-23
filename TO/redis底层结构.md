# redis

底层有五大主要数据结构类型：动态字符串（SDS）、双端链表(list)、字典（hashtable）、 压缩列表（ziplist）、整数集合（intset）

### 链表

链表节点:

```c
typedef struct listNode {
    struct listNode *prev;//前一个节点
    struct listNode *next;//下一个接单
    void *value;//节点值
} listNode;
```

链表迭代器:

```c
typedef struct listIter {
    listNode *next;
    int direction;
} listIter;
```

链表list:

```c
typedef struct list {
    listNode *head; //表头
    listNode *tail; //表未
    void *(*dup)(void *ptr); //节点复制函数
    void (*free)(void *ptr); //节点释放函数
    int (*match)(void *ptr, void *key); //节点对比函数
    unsigned long len; //节点长度
} list;
```

### 字典

字典节点:

```c

typedef struct dictEntry {
    void *key;//键
    union { //值
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;//指向下一个节点
} dictEntry;
```

字典哈希表：

```c
typedef struct dictht {
    dictEntry **table;
    unsigned long size;
    unsigned long sizemask;
    unsigned long used;
} dictht;
```



字典类型结构：

```c
typedef struct dictType {
    uint64_t (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
} dictType;

```



字典：

```c
typedef struct dict {
    dictType *type;
    void *privdata;
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

### 跳跃表(skiplist)

跳跃表(skiplist)是一种有序数据结构，它通过在每个几点中维持多个指向其他节点的指针，从而达到快速访问节点的目的


举个例子，goods-price 是一个有序集合，在这个有序集合中以商品goods_no为key，对应价格为value，保存了100款商品的价格：

```bash
redis-cli > ZRANGE goods_price 0 1 WITHSCORES
1) "20201021AKG"
2) "10.1"
3) "20201021AKGA"
4) "10.199999999999999"
```

在同一个跳跃表，各个节点保存的成员对象必须是唯一的，但是多个节点保存的分值却可以是相同的。分值相同的节点将按照对象成员在字典序列中的大小来进行排序，成员对象较小的节点排在前头


```bash
1) "2020102120S"
2) "1"
3) "202010212AS"
4) "1"
5) "202010212bS"
6) "1"
```