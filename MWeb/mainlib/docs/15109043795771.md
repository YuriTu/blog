---
title: Make React Great Again —— React v16 新特性
date: 2017-10-17 
categories: 技术
tags: [技术,react]
---
tl;dr  对于开发者而言最大的影响是MIT协议的改变，其次是传送门对开发方式的优化以及ssr方面的巨大提升。

---

React16 在国庆前发布了，这次版本迭代带来了许多令人激动的新特性。

# 协议的改变
前不久的React在业界引起了不小的波澜，更有国外WordPress与国内大厂百度的封杀React，以至于不少开发者认为Vue或成最大赢家。
终于FB将React16改为MIT协议，对于不升级的开发者也有MIT协议的15.6.2的旧版本可以选择。不过之前此举多少伤害了开发者对其的信任，FB是否再会出尔反尔，也影响这技术负责人对技术栈的选取。

# 传送门 （Portal）
长久以来全局的模态框、dialog、tooltips等等全局组件的DOM位置问题，终于有了优雅的解决方案。

当我们需要一个全局的提示框时，作为通用组件，它是脱离我们当前开发的业务组件的，因为从state的设计上，提示框的显隐是一个私有变量，而文案内容等一般由当前业务组件的props来决定，这就导致在业务组件中添加了一个无关组件，一来影响我们关注点分离的思想，二来难以复用。

另一方面由于DOM层级的影响css的规则与书写有会造成困难，你无法控制Module父元素的css状态，从而导致无法良好的抽象。

举个栗子：

理想的状态是这样

我们的模态框可以复用，等待数据流灌入即可。

```jsx
let = jsx = (
<body>
	<Main/>
	<Other/>
	<Module/>
</body>
)
```
可是实际上开发却是这样
```jsx
let jsx = (
<body>
	<Main>
		<components/>
		<Module/>
	</Main>
	<Other>
		<components/>
		<Module>
	</Other>
</body>
)
```
当然是用Redux可以完成第一种的设计，可是单单为了一个提示框就如此大弄干戈真的有必要吗？

React16提供了一种 将组件内部一些DOM元素绑定在外部组件的DOM上的方法，简单来说就是 在组件内部render一个组件，却能把这个render 的结果挂载在别的DOM上。从而完成我们所说的组件结构的优化与复用。

而如果我们使用portals 开发时组件结构就可以不变
```jsx
let jsx = (
<body>
	<Main>
		<components/>
		<Module/ ...props> // 接受数据与事件冒泡
	</Main>
	<Other>
		<components/>
		<Module ...props> // 接受数据与事件冒泡
	</Other>
	<Module>// 真实渲染 选择挂载位置为根元素
</body>
)

```
我们在业务组件中，控制公共组件的state数据，进而指定公共组件的渲染位置，最后达到良好的复用。

从代码上来说，我们的业务组件可以这么写
```javascript
//以前我们渲染会这么写
render() {
	reutrn (
		<div>{this.props.children}</div>
	)
}
//如果我们需要制定Module的挂载位置
render () {
	return ReactDOM.createPortal(
		<Module ...props.module>,//这里是我们要给传送门的东西
		this.props.node//需要挂载的地方
	)
}
```

而且传送门能够实现事件的冒泡，你不用担心开发时的父组件无法监听相关的事件，它的表现会像一个正常的组件一样，无论你把全局组件挂载到哪里。

# 异步渲染 （async rendering）
这东西厉害了，这是react提出的一个很先进的概念，由于React16使用了新的Fiber架构来进行项目构建(这个Fiber架构具体是什么，[博客][1]里说的很难我没看懂)，从而开发了异步渲染。

异步渲染是一种 与浏览器进行协作 以 定期执行对渲染任务的排序的策略（这块不太好理解 原文是 a strategy for cooperatively scheduling rendering work by periodically yielding execution to the browser. ）

通过异步渲染，以及避免react占用主线程，从而让页面动画更加流畅，可以看一下这个例子 [demo][2]

主要注意右上角那个黑方块，他一直在旋转占用着主线程，然后我们去进行请求，请求后会渲染页面，导致进程占用，页面重新渲染，方块旋转停止。

如果我们采用异步渲染就可以完成 在不占用主进程的情况进行渲染操作，从而不影响其他的元素。

这个功能并没有在React16实装，但是在博客中提到了未来几个月后可能会推出。

# 碎片DOM （fragments）
你再也不用给jsx带套了，`render`函数接受更加多样的数据结构。
一般我们写一个列表可能是这样

```javascript
const renderChildren = (arr) => (
	<ul>
		{arr.map( i => (<li>{i}</li>)  )}
	</ul>
)
```
我们干什么都要给加个父元素的contianer，这样一方面结构不明朗，另一面有可能会有多余的元素，而现在我们可以直接输出数组或者字符串了。
```javascript
render() {
  return [
    <li key="A">First item</li>,
    <li key="B">Second item</li>,
    <li key="C">Third item</li>,
  ];
}
// or
render() {
  return 'Look ma, no spans!';
}
```

# 文件体积优化

            
相对于上个版本有30%的减少


| 名称    |  新版本（kb）   |   压缩后（kb） |    旧版本（kb） |  压缩后 （kb）  |
| --- | --- | --- | --- | --- |
|   react  |5.3     | 2.2    |  20.7   | 6.9    |
| react-dom    | 103.7    | 32.6    | 141    |42.9     |
| react+react-dom    | 109    | 34.8    | 161.7    | 49.8    |


React文档上说由于对打包方式的改变，React 采用 Rollup打包，所产生的bundle不会受到工具的影响，比如webpack、Browserify、UMD或者其他的工具。这个Rollup是一个模块打包器，它能把小的代码打进比较大的或者更加复杂的库，是一个比较新的库。

另一个方面是 react自持自定义DOM属性了，由于不用再设置属性的白名单，一定程度上减少了文件大小。这个特性主要是为ssr服务的。

# 提升服务端渲染性能
这是新版本重点更新的特性之一，笔者SSR经验比较少，体会不是很深，感觉React16比较明显的提升是性能上的。

React16对ssr主要是性能提升了3倍以上，如果是最新版本node实验环境下提升更是达到了数量级的提升。

还有一个比较大的区别是在html到达客户端之后，react并不会去比较初次的渲染结果与服务端结果，react会尽量去重用相同的DOM元素，这也是为了优化ssr中的“注水”过程，从而提升ssr的性能。因为一般情况下我们的服务端与客户端也不会渲染不同的内容，除非是做时间戳的一些东西。


笔者认为对开发者影响比较大的特性就是以上这些了，总体来说新的特性能够解决一些日常开发的痛点，开发体验也越来越好了，不过React16因为用了ES6的一些集合数据结构例如`Map` 和`Set`所以，兼容性要求比较高，如果需要兼容老式浏览器需要上polyfill比如`core.js`和 `babel-polyfill`。

还有一些特性比如 错误捕捉的优化、api更新与打包的改变有兴趣的同学可以关注一下[React的博客][4]来了解。


  [1]: https://code.facebook.com/posts/1716776591680069/react-16-a-look-inside-an-api-compatible-rewrite-of-our-frontend-ui-library/
  [2]: https://build-mbfootjxoo.now.sh/
  [3]: ./images/react-size_1.png "react-size"
  [4]: https://reactjs.org/blog/2017/09/26/react-v16.0.html