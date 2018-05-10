---
title: Javascript 的最佳实现方法总结（二）
---

![Javascript 的数组遍历方法总结](http://upload-images.jianshu.io/upload_images/5236403-7796f9caf62ceb90.JPG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>这是第二篇关于 Javascript 实现方法的总结文章，主要是总结从 ES5 到 ES6 中的各种数组遍历方式，以及使用这些方法可能会遇到的坑。当然了，刚开始总结的一定不会很全面，也希望有更多想法和问题的小伙伴可以大家一起交流，文章也会一直完善下去，虽然不一定会有人看，哈哈，那开始吧！

Javascript 有很多数组遍历的方法，我们先从传统的遍历方法开始，然后再引入`ES6`的新方法：

### 1. while
传统方法`while`理解起来很简单，给个`index`然后每次让`index ++`就ok了。然后我们引入了一个新的概念，就是`ES6`中的新语法`...`，它可以用于但不仅限于数组和对象的的扩展和解构，从一定程度讲，把它用好了在某种程度上就基本可以抛弃`ES5`里面像`concat`、`slice`等等的数组拼接方法了（只是说基本，目前还不敢全扔）。
关于`ES6`的用法已经基本整理好了，在未来两三个星期会陆续发，希望到时候可以关注。这里扔一个权威的传送门，阮一峰老师的《ECMAScript 6入门》：http://es6.ruanyifeng.com/

```Javascript
function whileLooping(...args) {
    let index = 0
    while(index < args.length){
        console.log(args[index])
        index ++
    }
}

const flag = [1, 2, 3, 4]
whileLooping('java', 'python', 'c', 'c++', 'ruby') 
// java python c c++ ruby
whileLooping('java', 'python', 'c', 'c++', 'ruby', flag) 
// java python c c++ ruby [1, 2, 3, 4]
```
很简单，主要是为了做一个关于`...`语法的引子，让大家多使用更便捷的方法（虽然这里这么写好像有些鸡肋）。

### 2. for（传统方法里用的最多的）
这个我觉得不管是写什么的都比我熟，所以先象征性地扔一波看起来鸡肋又尴尬的代码：

```Javascript
function forLooping(array = [a = 10, b = 20, ...args], nothing = 20){
    for(let i = 0; i < array.length; i++) {
        console.log(array[i])
    }
    console.log(nothing)
}

forLooping([1, 2, 3, 4, 5])
// 1 2 3 4 5 20
forLooping([1, 2, 3, 4, 5], 10)
// 1 2 3 4 5 10
```
虽然这里涉及了`ES6`里的默认参数值以及`...`的另一种用法，但看起来都容易理解，就不多哔哔。接下来要分享的是一个我曾经在用`for`循环遇到的一个坑，关于作用域的。
大家可以尝试把`let`变成`var`，看看输出结果有什么变化吗？并没有。
但是在接下来这段代码里好像就不太一样了：
```Javascript
const array = []

for (var i = 0; i < 5; i++) {
	array.push(function() {
	console.log(i)          
	})  	
}
array[0]() // 5
array[1]() // 5
array[2]() // 5
array[3]() // 5
array[4]() // 5
console.log(array) // [ [Function], [Function], [Function], [Function], [Function] ]
```
明明`push`到数组里的看起来都是不一样的值，可是最后一跑就发现输出了没有一个值是对的。这是在`var`声明的时候，被引用的外部作用域中只有一个`i`，而不是为每次迭代的函数都有一个`i`被引用。如果你用`let`声明`i`，输出的值就会是`0 1 2 3 4`。这关系到`var`和`let`的区别，上面给的传送门里有介绍到，而我之后的`ES6`系列也会解释到，这里也不多哔哔了。
还有一件事就是，在某些博客上有人说`const`是用来声明常量的，可我们这里的`array`还能改变，这是因为用`const`声明值类型变量之后不能继续赋值，但是声明的引用类型变量还是可以被改变的。

### 3. forEach
这个是新方法里最基础的一个，唯一要注意的是`IE 9`以下浏览器不兼容。用的时候要向里面扔三个参数：
- currentValue（必需）：当前元素；
- index（可选）：当前元素索引值；
- arr（可选）：当前元素所属的数组对象。

```Javascript
function forEachLooping(...args) {
	args.forEach((current_value, index, arr) => {
		console.log(current_value, index, arr)
	})
}

forEachLooping('java', 'python', 'c', 'c++', 'ruby')
// java   0 [ 'java', 'python', 'c', 'c++', 'ruby' ]
// python 1 [ 'java', 'python', 'c', 'c++', 'ruby' ]
// c      2 [ 'java', 'python', 'c', 'c++', 'ruby' ]
// c++    3 [ 'java', 'python', 'c', 'c++', 'ruby' ]
// ruby   4 [ 'java', 'python', 'c', 'c++', 'ruby' ]
```
箭头函数`=>`用法值得关注，很好用，会在`ES6`系列里面具体介绍到。其它暂时没发现什么值得特别提的点，如果有小伙伴遇到这个东西的坑，希望可以跟我说一下，我们一起进步哈！

### 4. map
这里的`map`不是`地图`的意思，而是指`映射`，很好理解，就是原数组被`映射`成新的数组。在`forEach`方法中，并不会返回一个新的数组，而`map`则是对数组的每个元素使用一个函数，然后返回一个全新的数组。
```Javascript
function mapLooping(information) {
	const formatInfo = information.map(info => {
		return `姓名：${info.name}，年龄：${info.age}`
	})
	console.log(formatInfo)
}
const information = [
	{name: 'zixiang.xiao', age: 22},
	{name: 'tong.li', age: 22},
	{name: 'meimei.han', age: 30}
]
mapLooping(information)
// [ '姓名：zixiang.xiao，年龄：22', 
//   '姓名：tong.li，年龄：22', 
//   '姓名：meimei.han，年龄：30' ]
```
上面的代码也引入了`ES6`模板字符串的概念，书写起来相当方便，没什么特别的概念，有兴趣可以去了解一下。
关于`map`的兼容性，它不兼容`IE 9`以下版本的浏览器，如果非要在`IE 6-8`之间使用的话，也可以在`Array.prototype`里面进行扩展：
```Javascript
if (typeof Array.prototype.map != "function") {
    Array.prototype.map = function (fn, context) {
        var arr = []
    if (typeof fn === "function") {
        for (var k = 0, length = this.length; k < length; k++) {      
           arr.push(fn.call(context, this[k], k, this))
        }
    }
      return arr
    }
}
```
### 5. reduce
从某种程度上说`reduce`像是一个累加器，它会在数组中从左到右依次遍历，最终返回一个经过函数处理后的累积值。
```Javascript
function reduceLooping(...args) {
	const array_sum = args.reduce((x, y) => {
		return x + y
	}, 10)
	console.log(`初始值为10的array_sum累加之后结果为：${array_sum}`)
}

reduceLooping(1, 2, 3, 4, 5)
// 初始值为10的array_sum累加之后结果为：25
```
还有一个值得关注的点，如果数组处理之后需要返回一个累积值的时候，推荐使用`reduce`，从一个最直接的角度来说，据统计`reduce`的运算速度比`for`快几十倍。同样的，`reduce`只兼容`IE 9`及其以上的浏览器。

### 6. filter
跟名字一样，`filter`方法就是用来对一个数组进行过滤的。和之前的方法不一样的一点在于，`filter`的`callback`函数返回的是一个`Boolean`值。
```Javascript
function filterLooping(array) {
	const after = array.filter(name => name.indexOf('D') >= 0)
	console.log(after)
}

const name = ['Durant','James','Bryant','Duncan','Curry']
filterLooping(name)
// [ 'Durant', 'Duncan' ]
```
`filter`只兼容`IE 9`及其以上的浏览器。

### 7. every
关于`every`方法，用于检测数组所有元素是否都符合指定条件,使用指定函数检测数组中的所有元素：
- 如果数组中检测到有一个元素不满足，则整个表达式返回`false`，且剩余的元素不会再进行检测；
- 如果所有元素都满足条件，则返回`true`。
```Javascript
function everyLooping(array) {
	let truth = array.every(user => user.indexOf('cool') >= 0)
	if(truth) {
		console.log('都很帅！'); 
	}else {
		console.log('并不是所有人都很帅！')
	}
}

const user = ['zixiang.xiao, cool','lei.li, 9','meimei.han, 9','mou.liu, pdd','benwei.lu, ugly']
everyLooping(user)
// 并不是所有人都很帅！
```
可解释的不多，`every`只兼容`IE 9`及其以上的浏览器。希望看完代码你不会觉得我厚颜无耻，就算你真这样觉得也没有什么用！

### 8. some
`some`的用法和`every`其实差不多，只不过`some`的含义是判断一个数组上的元素是否至少有一个符合某种条件， 方法会依次执行数组的每个元素：
- 如果有一个元素满足条件，则表达式返回`true`, 剩余的元素不会再执行检测；
- 如果没有满足条件的元素，则返回`false`。

```Javascript
function someLooping(array) {
	let truth = array.some(user => user.indexOf('benwei.lu') >= 0)
	if(truth) {
		console.log('有人长得丑！'); 
	}else {
		console.log('并没有特别丑的人！')
	}
}

const user = ['zixiang.xiao, cool','lei.li, 9','meimei.han, 9','mou.liu, pdd','benwei.lu, ugly']
someLooping(user)
// 有人长得丑！
```
这个代码看完我相信你会觉得我说的真对！

## 总结
其实每一种方法都有它特定的场景可以作为一个优秀的方案使用，作为一个前端小菜鸡，我也会多去尝试这些东西的使用，在一定的累积之后会继续写它们的应用。关于低版本浏览器不兼容的问题，之后我会在原文章上边添加关于低版本浏览器的兼容方法，不过目前我只能做到兼容到`IE6`，我相信也没多少人用更低的版本了，毕竟长得帅的人用浏览器都是与时俱进的！
**最后最后，如果你觉得我写的还行的话，就快给我点赞并且关注我吧！我会继续努力的！谢谢各位大佬！**