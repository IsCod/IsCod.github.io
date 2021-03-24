# 链表

链表由一个个数据节点组成，链表将数据和数据之间关联起来，从一个数据节点指向另一个数据节点。

### 链表和链表节点实现

常用链表节点结构包含指向上一个(prev)和下一个节点(next)的指针，和该节点值的value

链表结构一般包含头部节点(head)，末尾节点(tail)的指针和链表长度(len)

我们先看Redis中的链表结构(src/adlist.h)：

```c
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```

GoLand实现链表：

```golnd
package main

import "fmt"

type linkNode struct {
	Prev, Next *linkNode
	Value      interface{}
}

type link struct {
	Head, Tail *linkNode
	Len        int
}

func (l *link) Push(node *linkNode) {
	if l.Len == 0 {
		l.Head = &linkNode{Value: node.Value}
		l.Tail = l.Head
		l.Head.Next = l.Tail
		l.Tail.Prev = l.Head
	} else {
		tail := l.Tail
		tail.Next = &linkNode{Prev: tail, Value: node.Value}
		l.Tail = tail.Next
	}
	l.Len++
}

func (l *link) Pop() *linkNode {
	if l.Len < 1 {
		return nil
	}
	head := l.Head
	if head.Next != nil {
		head.Next.Prev = nil
	}
	l.Head = head.Next
	l.Len--
	return head
}

func main() {
	var l = &link{}
	l.Push(&linkNode{Value: 1})
	l.Push(&linkNode{Value: 10})
	l.Push(&linkNode{Value: 3})
	l.Push(&linkNode{Value: 2})
	head := l.Head
	for head != nil {
		fmt.Printf("%d\n", head.Value)
		head = head.Next
	}
}
```
* 参考
    * [redis设计与实现](https://www.bookstack.cn/read/redisbook/f8fc3415f3c66c78.md)
    * [数据结构(goland)](https://www.bookstack.cn/read/hunterhug-goa.c/algorithm-link.md)