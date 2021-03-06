﻿title: Jquery源码学习计划——ready揭秘
date: 2016-09-07 
categories: 技术
tags: [jq,js]
---
# jQuery（document）.ready(funtion($){})

进入JQ的入口即是 jQuery（document）.ready(funtion($){})
1.在Document上绑定了ready事件，与window.onload的区别是
ready ： DOM加载完成
load : 文档加载完成
2.需要改为load的情况 ： 获得图片的长度

----------
## .ready( funtion($){} )

1.7

在建立JQ对象的时候,rootjQuery 是作为一个参数存在的

```javascript
var jQuery = function( selector, context ) {

		// The jQuery object is actually just the init constructor 'enhanced'
		
		return new jQuery.fn.init( selector, context, rootjQuery );
	},
```
### 关于rootjQuery

rootjQuery默认是指向jq(document),
其他的作用还有在context没有被指定的情况下，进行的代替，对选择器进行一种范围的界定，达到$("#div",document)的效果

```javascript
} else if ( !context || context.jquery ) {
	return ( context || rootjQuery ).find( selector );
```
init中的方法会先对传入的参数进行判断，如果是字符串作用可能是判断某个元素图片等是否载入完成；如果在确定传入的是函数之后，作用是一系列的动作执行,进入到jQuery.prototype的方法ready中

```javascript

// All jQuery objects should point back to these
rootjQuery = jQuery(document);
jQuery.fn = jQuery.prototype = {
	constructor: jQuery,
	init: function( selector, context, rootjQuery ) {
			if ( typeof selector === "string" ) {
			...
			} else if ( jQuery.isFunction( selector ) ) {
			return rootjQuery.ready( selector );
		}
```	

> 为什么可以使用`$()` 作为逻辑入口？
`$(需要运行的逻辑)`与`$("div")`的区别就在于其重载的特性，在对参数进行判断后如果发现是function 就直接走`jQuery(document).ready()`

进入到ready方法后， 参数为 `function( $ )`

我们知道初始化JQ对象的时候就已经定义了`_$ = window.$`

```javascript
ready: function( fn ) {
	// Attach the listeners
	jQuery.bindReady();

	// Add the callback
	readyList.add( fn );

	return this;
},
```

## bindReady()
bindReady 定义在jQuery.extend中，作用是初始化readylist,如果readylist已经存在，那么就会直接退出函数,否则初始化事件监听的函数列表readylist 参数one 表示只能被初始化一次，memory表示：“记录上一次触发监听函数列表readylist时的参数，之后添加任何监听函数都将用记录的参数值立即调用”

```javascript
bindReady: function() {
	if ( readyList ) {
		return;
	}

	readyList = jQuery.Callbacks( "once memory" );
```

### readyList.add( fn );

readyList 经过 jQuery.Callbacks 之后，把参数加入到监听列表，返回this，这是一贯的链式操作，直到此处，准备工作就做完啦.

开始监听DOM结构是否加载完成,一旦完成就开始执行jQuery.ready(wait)函数

>为什么readyList有add方法：`jQuery.Callbacks`。。。`C.a.l.l`。。。大写C这是个构造函数，是个对象

```javascript
if ( document.readyState === "complete" ) {
	// Handle it asynchronously to allow scripts the opportunity to delay ready
	return setTimeout( jQuery.ready, 1 );
}
```

但是这里有一个问题 jQuery对象下有同名的ready函数
真实的ready(wait)在jQuery.extend = jQuery.prototype.extend 下


> 回答extend的存在意义： 我们有`$("div").click` 这样的方法，也有`$.ajax` 这样的方法，前者需要实例化一个对象，后者不用，那么如果全放在init里面那么不需要实例化对象的方法也必须要一个对象作为“牵引”才能触发，这显然是不合理的。



### jQuery.ready(wait)

作用是执行readylist里面的函数，只有在设定延迟的情况，参数wait为true

```javascript
ready: function( wait ) {
	// Either a released hold or an DOMready/load event and not yet ready
	if ( (wait === true && !--jQuery.readyWait) || (wait !== true && !jQuery.isReady) ) {
```

isReady 是初始化时的一个boolean变量，用于表明没有执行过之后的if块的代码

```javascript
// Make sure body exists, at least, in case IE gets a little overzealous (ticket #5443).
if ( !document.body ) {
	return setTimeout( jQuery.ready, 1 );
}

// Remember that the DOM is ready
jQuery.isReady = true;

// If a normal DOM Ready event fired, decrement, and wait if need be
if ( wait !== true && --jQuery.readyWait > 0 ) {
	return;
}
```

用于确认之前是否被延迟设定

```javascript
// If there are functions bound, to execute
readyList.fireWith( document, [ jQuery ] );
```

执行列表的函数 fireWirh(context,listener)

```javascript
// Trigger any bound ready events
if ( jQuery.fn.trigger ) {
	jQuery( document ).trigger( "ready" ).off( "ready" );
}
```

trigger 用于触发由on('ready',function) 设置的函数

## 一句话总结：
IE9- 是对DOMContentLoaded事件的封装
IE9+ 对onreadystatechange事件的封装
