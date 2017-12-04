---
title: webpack 3.0 上车指北
date: 2017-07-21 
categories: 技术
tags: [webpack, node]
---

想要解决的问题：
项目是否要 换 3.0
如何从webpack1.x 向 3.0过渡
相关兼容问题

> **TL；DR** 
> 对打包体积优化差强人意，但是速度优化让人惊喜 
> 有一些API的改变，总体上可以无痛升级

----------

webpack3前段时间推出了，我对新版本进行了一些探索。

# 需不需要升级
判断标准就是：想解决的问题webpack能帮你以**低成本**解决吗

我想解决的问题是：
1. 打包时间太长，每次fixbug进行项目打包需要5分多钟
2. 体积太大，简单页面100k+ ，复杂页面500k-700k+，

## 技术选型标准

关于技术选型我觉得不同项目需要不同对待，我的理解是

> 个人项目怎么激进怎么来
>  集体项目在保守的大方向下前进

如果是简单项目，只需要一个人来完成，比如说简单的活动页，后台管理页，这种需求往往一个人就做完了，不涉及配合的问题，试错成本小，那么激进无妨。

如果是集体项目，如果使用新技术涉及到团队的学习成本问题，成员间的沟通问题，而且非常容易牵一发而动全身，以前代码的兼容怎么办，用了太新的技术以后不好招人怎么办，都是需要考虑的问题，所以已经成熟的项目，前进要慎重。

## webpack能解决什么

### 速度
我在一个自己负责的小项目中尝试了webpack的3.0版本，最直观感觉就是打包速度快了,之后我开始在主要项目中进行测试,在开发时，我需要起一个本地server

我主项目entry是20个文件，大小各异，样本多点我们好看对比。

我的主项目使用的是1.13.1
![enter description here][1]
毕竟涉及把所有文件写到虚拟内存里，有点慢

如果使用3.0
![enter description here][2]

这里启开发环境不涉及压缩混淆一类的就是过一下loader，差距还是很明显的；

**接下来我们看上线打包**
在1.X版本慢就慢在：
![enter description here][3]


然后时间上是
![enter description here][4]


所以每次改完bug打包都能喝个茶了...

![][5]

然后如果使用3.0
当然也是有慢的地方的
![enter description here][6]


但是！
时间快了20%所右

![ ][7]



### 体积

webpack3.0的新特性之一是 Scope Hoisting 译为 作用域提升
特别是这张图传的神乎其神
![enter description here][8]

近50%的体积减小，惊不惊喜，意不意外！这还有什么好犹豫的快上车啊！

![][9]

![ ][10]

**嗯...好像哪里不太对**
![enter description here][11]
![ ][12]

(╯‵□′)╯︵┻━┻ 这不对啊，你就优化了这1%？

### webapck到底优化了什么
webpack3优化的原因是加入了Scope Hoisting 译为 作用域提升，我们来一起看看他到底做了什么

我们的实验如下,es6module对照组与webpack版本对照组，webpack3版本增加作用于提升插件
```javascript
//test1
var foo = require("./module1")

foo();
//module1
module.exports = function foo () {
    console.log("hello world")
}

//test2.js
import foo from "./module2"
foo();
//module2
export default function foo() {
    console.log("hello world")
}

//webpack.config for 1&2
module.exports = {
    entry:{
		"es5":"./webpacktest/test1",
        "es6":"./webpacktest/test2"
    },
    output:{
       filename:"[name].bundle.js"
    },
	plugins: [
    new webpack.optimize.ModuleConcatenationPlugin()//for webpack3
  ]
}

```
首先我们一般都知道webapck处理依赖是将其封在一个独立闭包函数里面， 来完成依赖的独立作用域

然后ES5 与 ES6 的实现还有区别我们之后看

我们先说ES6module
webpack2 的打包情况

这里是2

![enter description here][13]

这里是3

![enter description here][15]


**敲黑板！！**
这里我们看出来webpack2为每一个闭包都加了一个 
```javascript
(function(module, __webpack_exports__, __webpack_require__) {
	...
})
```
而webpack3把所有的依赖都打在了一个闭包里，那么你想，你一个组件要是个单页面应用那怎么不得十几个依赖，那就是十几个（function (module）...）
然后依赖再依赖别的这得多少啊...
Webpack 3 + ModuleConcatenationPlugin() 帮你全干掉，这就是为什么打包体积小了。

我们可以从下图看出来，es6.js 从3.17k 减少到了2.75k

![enter description here][16]


![enter description here][17]


----------
那有同学就问了，你说的这是2和3对比，1呢，我们来看实验


![enter description here][18]

![enter description here][19]


这不对啊，怎么1反而是最小的，这还升什么级，(╯‵□′)╯︵┻━┻。
我们来看1里面打出来是什么


**ES6module**

![enter description here][20]

**ES5module**

![enter description here][21]

这就要说到ES6module 的问题了，我们可以看到webpack1对es6module根本没有处理，那么这样的ES6代码，很多浏览器是不支持的，而2、3版本都对ES6module进行了处理

> 原因是webpack1对es6module支持的很差，在2.1.0版本中的，才开始支持，其bugfixes写到
> **It's now possible to combine webpack ES2015 modules with babel ES2015 modules
**

所以，在webpack1中传统的es5的方式，webpack对他进行了处理，和我们之前的结果一样，使用了闭包来保证独立作用域。

**那为什么webpack1实验结果体积最小呢**

是因为webpack在包里新加了一些getter setter方法用于优化依赖的加载

![enter description here][22]

所以虽然在我们的实验中webpack1更小，但是在实际项目的结果中，webapck3的体积要比1要小。

而在我的项目中因为兼容原因没有使用ES6module所以优化效果比较差，所以如果项目中完全使用es6的话，那么就可以完全发挥ES6的威力。

### webpack3的单一闭包模式的问题
我们在这张图中可以看到

![enter description here][23]

在webpack3中依赖存储于一个闭包，那么问题来了，在没有闭包的情况下，他们在同一个块级作用域，如果我在一个依赖里声明了一个变量，其他模块岂不是也能拿到？

**我们来搞事情**
```javascript
//test2.js
import foo from "./module2"
foo();
console.log(other)
//module2
var other = "The World"
export default function foo() {
    console.log("hello world")
}
```

![enter description here][24]

我们可以看出来，虽然没有了闭包，但是使用了另外的名字来保证了作用域。





# 相关兼容性
如果你决定升级了那么首先就要对配置文件进行一些更改，主要是 loader 与 plugins 的一些改变，语法上有一些不兼容。
另外webpack3的文档写的比1好多了，读起来清晰多了，所以基本的	api在这里不再赘述，这里列出来我遇到的一些问题
## 语法
在打包或者建立服务器的时候可能会报找不到一些node的模块比如fs，这个时候需要在webpack文件里面这么设置
```javascript
	node:{
        console: true,
        fs: 'empty',
        net: 'empty',
        tls: 'empty',
    },
```
## 插件
babel-loader 官方推荐更新到7.1+
loader得到的souce从String改为了buffer对象，


# 总结
所以说，总体上来说升级有一定收益，webpack官方也说webpack2可以直接升级webpack3，而webapck1升级也仅仅是一些api名字的变化。

如果用了es6那么升级就可以发挥他的威力。

而且webpack github上的issues 关于升级后的兼容性问题非常少，如果有时间的同学不妨一试。




参考：
[Releases · webpack_webpack][25]
[🍾🚀 webpack 3_ Official Release!! 🚀🍾 – webpack – Medium][26]
[https://github.com/lishengzxc/bblog/issues/34][27]


  [1]: http://odf9m3avc.bkt.clouddn.com/webpack1.jpeg
  [2]: http://odf9m3avc.bkt.clouddn.com/webpack2.jpeg
  [3]: http://odf9m3avc.bkt.clouddn.com/WechatIMG5.jpeg
  [4]: http://odf9m3avc.bkt.clouddn.com/WechatIMG4.jpeg
  [5]: http://odf9m3avc.bkt.clouddn.com/c15d6d87ba59982405c8195ef4e0020e.jpeg
  [6]: http://odf9m3avc.bkt.clouddn.com/WechatIMG6.jpeg
  [7]: http://odf9m3avc.bkt.clouddn.com/webapck3-time.jpg
  [8]: http://odf9m3avc.bkt.clouddn.com/webpack3.0%E6%95%88%E6%9E%9C%E5%9B%BE.png
  [9]: http://odf9m3avc.bkt.clouddn.com/webapck3.jpeg
  [10]: http://odf9m3avc.bkt.clouddn.com/webpack1.13-220s.png
  [11]: http://odf9m3avc.bkt.clouddn.com/hold2.png
  [12]: http://odf9m3avc.bkt.clouddn.com/hold1.png
  [13]: http://odf9m3avc.bkt.clouddn.com/note/20es6-webpack2.png "es6-webpack2"
  [14]: http://odf9m3avc.bkt.clouddn.com/note/20es6-webpack2.png "es6-webpack2"
  [15]: http://odf9m3avc.bkt.clouddn.com/note/20es6-webpack3.png "es6-webpack3"
  [16]: http://odf9m3avc.bkt.clouddn.com/note/20esmodule-webpack2.png "esmodule-webpack2"
  [17]: http://odf9m3avc.bkt.clouddn.com/note/20esmodule-webpack3.png "esmodule-webpack3"
  [18]: http://odf9m3avc.bkt.clouddn.com/note/20esmodule-webapck1.png "esmodule-webapck1"
  [19]: http://odf9m3avc.bkt.clouddn.com/note/20%E6%83%8A%E4%B8%8D%E6%83%8A%E5%96%9C.jpg "惊不惊喜"
  [20]: http://odf9m3avc.bkt.clouddn.com/note/20es6-webpack1.png "es6-webpack1"
  [21]: http://odf9m3avc.bkt.clouddn.com/note/20es5-webpack1.png "es5-webpack1"
  [22]: http://odf9m3avc.bkt.clouddn.com/note/20webpack2-getter.png "webpack2-getter"
  [23]: http://odf9m3avc.bkt.clouddn.com/note/20es6-webpack3.png "es6-webpack3"
  [24]: http://odf9m3avc.bkt.clouddn.com/note/20webpack3-sco.png "webpack3-sco"
  [25]: https://github.com/webpack/webpack/releases?after=v2.2.0-rc.0
  [26]: https://medium.com/webpack/webpack-3-official-release-15fd2dd8f07b
  [27]: https://github.com/lishengzxc/bblog/issues/34