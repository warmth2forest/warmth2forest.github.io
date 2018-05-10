---
title: Javascript 的最佳实现方法总结（一）
---

![Javascript 的最佳实现方法总结](http://upload-images.jianshu.io/upload_images/5236403-10d1806c18c1351b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> 本文将主要针对 Javascript 来总结一些常用的实现方法，有的是根据自身实践总结出来的经验，有的是在逛别人的博客和技术分享时候得到的启发，全部都会慢慢搬到这里，也欢迎有更多的人来提供你们的实现方式，大家互相交流。作为一个刚刚毕业的前端小白，也希望的各位老司机能为我指出不足。本文也将不定期更新！

### 1. 生成随机字符串的实现
还记得校招的时候，以为自己面的是前端岗，应该主要就是考察布局基础、Javascript基础、常用框架理解、Node.js、前端性能优化等等知识，但是没想到坐下来面试官就让我写一个随机字符串生成器，当时也写出来了，不过显得十分笨重，下面是我在阅读别人代码里面看到的比较不错的实现方式，希望可以对工作和面试的人有用。
```Javascript
const random_str = function(length) {
    const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890'
    let str = ''
    for(let i = 0; i <length; i++) {
        str += chars.charAt(Math.floor(Math.random() * chars.length))
    }
    return str
}
```
直接通过对输出字符串增加字符，要比使用数组方法好一些，而且这样生成一个指定位数的随机字符串，重复几率也会很低，并且随着`length`增大，重复几率降低。

### 2. 前端获取服务端时间
这个是在逛博客的时候看到的黑科技，简单粗暴！
```Javascript
const request = new XMLHttpRequest()
request.get('GET', location, false)
request.send(null)
console.log(req.getResponseHeader('Date'))
```

### 3. 检测类型
还记得以前去面试的时候，面试官问我怎么判断一个变量的类型，我张口就是`typeof`，现在想想真想给自己一个大耳刮子，实际上，typeof只能用来检测基本类型，我们举个例子：
```Javascript
typeof 20 //'number'
typeof 'hey man' //'string'
typeof true //'boolean'
typeof undefined //'undefined'
typeof Symbol() //'symbol'
typeof {} //'object'
typeof [] //'object'
typeof Function() //'function'
```
我们可以看到，基本类型使用`typeof`就可以检测出变量类型，可是在引用类型里，对象和数组输出的值都是`object`，函数则是输出`function`。对于自定义类型和原生的引用类型，一般使用`instanceof`进行判断。比如：
```Javascript
let obj = {}
let arr = []
obj instanceof Object //true
obj instanceof Array //false
arr instanceof Array // true
```
但是有时候情况往往不是这么简单。比如如果你想检测 iframe 里面的属性值的话，基本上是不可能的。因为检测的前提要求是在同一个全局作用于下。所以为了能够正常检测一些值的类型(排除在 iframe 的情况)。那有没有什么万能的方法呢？确实有，你可以使用调用`toString()`的方法，得到相关的类型。就像这样：
```Javascript
let value = new FormData()
console.log(Object.prototype.toString.call(value))
//'[object FormData]'
```
由此，也就可以得出一个通用的检测类型方法，基本上可以返回所有的类型。代码如下：
```Javascript
const getType = function(value) {
    return Object.prototype.toString.call(value).match(/\s{1}(\w+)/)[1]
}
```

### 4. 前端压缩图片
一年前找实习的时候被问到前端性能优化，就有提到前端压缩图片，那么要怎么压缩？为什么要压缩？对于压缩图片的需求一般会出现在上传图片（尤其是移动端）功能里，比如微信发朋友圈、QQ空间发说说（现在多了一个发送原图的按钮）的时候，就对上传的图片进行过压缩。

毕竟现在的手机硬件越来越流弊，随便一拍就是好几M，无论是用3G/4G还是Wifi上传起来都比较费劲，而且也比较浪费流量，一般只需要100+M就已经差不多了。因此，在用户选择好需要上传的图片之后，在客户端需要进行处理，将图片压缩好再上传到服务器进行下一步处理。

图片压缩的原理大同小异，自己也试着写过一些，都不理想，后来看到一篇博客分享了关于前端压缩图片的一个插件`localResizeIMG`，觉得很好用，就拿来分享了。引用`localResizeIMG`官方文档的原话：

> 基本原理是通过canvas渲染图片，再通过 toDataURL 方法压缩保存为 base64 字符串（能够编译为jpg格式的图片）。

那么怎么用呢？也很简单，传入照片（可以是`file`对象也可以直接传入图片路径），然后设置好自己想要的分辨率（就是最大`width`和`height`是多少`px`），接着设置个图片质量，再来一个`Promise`风格的`callback`差不多就行了，最后把压缩完的`base64`作为参数传进`callback`。作者还很贴心的把照片`base64`编码的长度也传参进来了，方便后端校验图片是否上传完整。示例如下：
```Javascript
<input onchange="upload().bind(this)" type="file" accept="image/*" />
function upload() {
    lrz(this.files[0])
        .then(rst => {
            // 处理成功会执行
        })
        .catch(err => {
            // 处理失败会执行
        })
        .always(() => {
            // 不管是成功失败，都会执行
        })
})
```
再给个传送门：https://github.com/think2011/localResizeIMG

-------

暂时先总结这四个，有需要的同学可以先看着，等过两天还会给大家带来更多的分享，有的是来自我自己，有的是我在逛技术圈的时候看到比较不错的实现方式，有什么不对的地方欢迎指出来，有什么更好的实现方式也可以提出来，大家一起交流一起进步！当然**点个赞和关注**是最好的啦！