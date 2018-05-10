---
title: Javascript 实现基础数据结构
---


![](http://upload-images.jianshu.io/upload_images/5236403-1cf3032199b90fa6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##Tip
计算机基础不管是什么语言、什么开发岗位都是必须要了解的知识，本文主要是通过 JS 来实现一些常用的数据结构，很简单。
> 本文插图引用：[啊哈！算法](https://book.douban.com/subject/25894685/) ，由于是总结性文章，所以会不定时地修改并更新，欢迎关注！

## Stack 栈
**堆栈**（stack），也可以直接称为**栈**，它是一种后进先出的数据结构，并且限定了只能在一段进行插入和删除操作。
![](http://upload-images.jianshu.io/upload_images/5236403-0dd8645b6e109ae7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图所示，假设我们有一个球桶，并且桶的直径只能允许一个球通过。如果按照 2 1 3 的顺序把球放入桶中，然后要将球从桶中取出，取出的顺序就是 3 1 2，先放入桶中的球被后取出，这就是 **栈** 插入和删除的原理。

我们接触过的算法书基本也都是用c语言实现这个数据结构，它的特殊之处在于只能允许链接串列或者阵列的一端（top）进行 **插入数据**（push）和 **输出数据**（pop）的运算。那么如果要用 JS 来实现它又该怎么操作呢？
```javascript
// 栈可以用一维数组或连结串列的形式来完成

class Stack {
    constructor() {
        this.data = []
        this.top = 0
    }
    // 入栈
    push() {
        const args = [...arguments]
        args.forEach(arg => this.data[this.top++] = arg)
        return this.top
    }
    // 出栈
    pop() {
        if(this.top === 0) {
            throw new Error('The stack is already empty!')
        }
        const popData = this.data[--this.top]
        this.data = this.data.slice(0, -1)
        return popData
    }
    // 获取栈内元素的个数
    length() {
        return this.top
    }
    //  判断栈是否为空
    isEmpty() {
        return this.top === 0
    }
    // 获取栈顶元素
    goTop() {
        return this.data[this.top-1]
    }
    // 清空栈
    clear() {
        this.top = 0
        return this.data = []
    }
}
```

通过 **面向对象** 的思想就可以抽象出一个 **栈** 的对象，实例化：
```javascript
const stack = new Stack()

stack.push(1, 2)
stack.push(3, 4, 5)
console.log(stack.data)
// [ 1, 2, 3, 4, 5 ]
console.log(stack.length())
// 5
console.log(stack.goTop())
// 5
console.log(stack.pop())
// 5
console.log(stack.pop())
// 4
console.log(stack.goTop())
// 3
console.log(stack.data)
// [ 1, 2, 3 ]

stack.clear()
console.log(stack.data)
// []
```

在 [啊哈！算法](https://book.douban.com/subject/25894685/) 描述 **栈** 的应用时，举了一个求回文数的例子，我们也用 JS 来把它实现一波：
```javascript
const isPalindrome = function(words) {
    const stack = new Stack()
    let copy = ''
    words = words.toString()

    words.split('').forEach(word => stack.push(word))

    while(stack.length() > 0) {
        copy += stack.pop()
    }
    return copy === words
}

console.log(isPalindrome('123ada321'))
// true
console.log(isPalindrome(1234321))
// true
console.log(isPalindrome('addaw'))
// false
```

## Linked List 链表
在存储一大波数的时候，我们通常使用的是数组，但有时候数组显得不够灵活，比如：有一串已经从小到大排好序的数 2 3 5 8 9 10 18 26 32。现需要往这串数中插入 6 使其得到的新序列仍符合从小到大排列。

![](http://upload-images.jianshu.io/upload_images/5236403-5da913a488feb1f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图所示，如果使用数组来实现的话，则需要将 8 和 8 后面的 数都依次往后挪一位，这样操作显然很耽误时间，如果使用 **链表** 则会快很多：

![](http://upload-images.jianshu.io/upload_images/5236403-f0ded633e01d7456.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**链表**（Linked List）是一种常见的数据结构。它是一种线性表，但是并不会按线性的顺序存储数据，而是在每一个节点里存到下一个节点的指针。和图中一样，如果需要在 8 前面插入一个 6，直接插入就行了，而无需再将 8 及后面的数都依次往后挪一位。

那么问题来了，如何使用 JS 实现 **链表**呢？还是运用 **面向对象** 的思想：

```javascript
// 抽象节点类
function Node(el) {
    this.el = el
    this.next = null
}

// 抽象链表类
class LinkedList {
    constructor() {
        this.head = new Node('head')
    }
    // 查找数据
    find(item) {
        let curr = this.head
        while(curr.el !== item) {
            curr = curr.next
        }
        return curr
    }
    // 查找前一个节点
    findPre(item) {
        if(item === 'head') {
            throw new Error('Now is head!')
        }
        let curr = this.head
        while(curr.next && curr.next.el !== item) {
            curr = curr.next
        }
        return curr
    }
    // 在 item 后插入节点
    insert(newEl, item) {
        let newNode = new Node(newEl)
        let curr = this.find(item)
        newNode.next = curr.next
        curr.next = newNode
    }
    // 删除节点
    remove(item) {
        let pre = this.findPre(item)
        if(pre.next !== null) {
            pre.next = pre.next.next
        }
    }
    // 显示链表中所有元素
    display() {
        let curr = this.head
        while(curr.next !== null) {
            console.log(curr.next.el)
            curr = curr.next
        }
    }
}
```
实例化：
```javascript
const list = new LinkedList()
list.insert('0', 'head')
list.insert('1', '0')
list.insert('2', '1')
list.insert('3', '2')
console.log(list.findPre('1'))
// Node { el: '0', next: Node { el: '1', next: Node { el: '2', next: [Object] } } }
list.remove('1')
console.log(list)
// LinkedList { head: Node { el: 'head', next: Node { el: '0', next: [Object] } } }
console.log(list.display())
// 0 2 3
```

## Binary tree 二叉树
**二叉树**（binary tree）是一种特殊的树。**二叉树** 的特点是每个结点最多有两个儿子，左边的叫做左儿子，右边的叫做右儿子，或者说每个结点最多有两棵子树。更加严格的递归定义是：**二叉树** 要么为空，要么由根结点、左子树和右子树组成，而左子树和右子树分别是一棵 **二叉树**。

![](http://upload-images.jianshu.io/upload_images/5236403-c8d2452517c8136e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

和前两个例子一样，通过类的抽象来实现 **二叉树**：
```javascript
// 抽象节点类
function Node(key) {
    this.key = key
    this.left = null
    this.right = null
}
// 插入节点的方法
const insertNode = function(node, newNode) {
    if (newNode.key < node.key) {
        if (node.left === null) {
            node.left = newNode
        } else {
            insertNode(node.left, newNode)
        }
    } else {
        if (node.right === null) {
            node.right = newNode
        } else {
            insertNode(node.right, newNode)
        }
    }
} 
// 抽象二叉树
class BinaryTree {
    constructor() {
        this.root = null
    }
    insert(key) {
        let newNode = new Node(key)
        if(this.root === null) {
            this.root = newNode
        } else {
            insertNode(this.root, newNode)
        }
    }
}
```
实例化测试一波：
```javascript
const data = [8, 3, 10, 1, 6, 14, 4, 7, 13]

// 实例化二叉树的数据结构
const binaryTree = new BinaryTree()
// 遍历数据并插入
data.forEach(item => binaryTree.insert(item))
// 打印结果
console.log(binaryTree.root)
/*
Node {
  key: 8,
  left:
   Node {
     key: 3,
     left: Node { key: 1, left: null, right: null },
     right: Node { key: 6, left: [Object], right: [Object] } },
  right:
   Node {
     key: 10,
     left: null,
     right: Node { key: 14, left: [Object], right: null } } }
*/
```

## 总结
算法和数据结构是非常重要的一环，之后也会慢慢总结更新，欢迎关注！
