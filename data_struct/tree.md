# 树

树是一种比较高级的基础数据结构，由多个有限节点组成的具有层次关系的集合

树的定义:

    1.有节点间的层次关系，分为父节点和子节点
    2.有唯一一个根节点，该节点没有父节点
    3.除了根节点，每个节点都有一个父节点
    4.每个节点本身以及它的子节点都是一颗树，是一个递归结构
    5.没有后代的节点称为叶子节点，没有节点的树称为空树

根据二子节点的多寡，有二叉树、三叉树、四叉树等。
故二叉树每个节点最多只有两个子节点的树，我们一般使用二叉树主要实现查找功能

## 二叉树

二叉树节点结构:

```go
type treeNode struct {
	Value       interface{}
	Times       int64
	Left, Right *treeNode
	Lock sync.Mutex
}
```

## 代码实现

```go
package main

import (
	"sync"
)

type Tree struct {
	Root *treeNode
}

type treeNode struct {
	Value       interface{}
	Times       int64
	Left, Right *treeNode
	Lock sync.Mutex
}

func (t *treeNode) Add(value interface{}) {
	t.Lock.Lock()
	defer t.Lock.Unlock()
	if t.Value.(int) < value.(int) {
		if t.Left != nil {
			t.Left.Add(value)
		} else {
			t.Left = &treeNode{Value: value}
		}
	} else if t.Value.(int) > value.(int) {
		if t.Right != nil {
			t.Right.Add(value)
		} else {
			t.Right = &treeNode{Value: value}
		}
	} else {
		t.Times++
	}
	return
}

func (t *Tree) Add(value interface{}) {
	if t.Root == nil {
		t.Root = &treeNode{Value: value}
		return
	} else {
		t.Root.Add(value)
	}
}
```

