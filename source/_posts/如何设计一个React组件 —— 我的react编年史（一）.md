title: 如何设计一个React组件 —— 我的react编年史（一）
date: 2016-09-08
categories: 技术
tags: [react,js,工程思想]
---

开始用react以来，最大的问题就是如何设计一个组件，使用react以来，对一个组件的设计方式也多次更迭，因为一步一步的更迭，也对react本身的机制有了一些感悟，反思自己曾经犯下的错误，总结经验。粗鄙之言，莫要见笑。

# 一、尧舜时代

空有react的壳子，这个时候的理解是 react无非是 Html in JS，把willMount当$()一样用，里面写了一大堆事件绑定。

这里的问题是很显然的，完全没有利用上react的特性，把react仅仅当做了一个mvc中的view模板用了而已。

# 二、春秋战国

一个特点就是乱

开始对state了有了一定的认识，事件驱动，简单的交互开始以来state。

这个时候还是一知半解的状态，在一个组件中，混杂了react的事件逻辑，也混杂着jq的事件绑定。

子组件里有自己的state，也有组件的props，对state和props的使用是混乱的
导致的问题是：1.完全没有组件化的思想 2.逻辑混乱debugger非常困难

# 三、秦汉时代

完全摒弃了jq，所有的state以及逻辑集中于祖先组件，父组件完成一切的mvc，子组件只有v层的作用。

活生生的君主集权专治，带来的问题也是相似的，上令下达非常困难，一个数据，一个逻辑处理函数，传3、4层，这还只是父-子、子-父 传递，同级通信更是要命，子-父-子，在传递过程中，深层次嵌套的问题会非常棘手。

>其实深层次嵌套的问题一直在react中存在，react全家桶也没有解决，redux的store说白了也就是一个大的祖先组件，在之后的无状态组件的开发模式中有一些改善

# 四、三国时代
开始尝试mvc中vc分离。

因为集权统治导致的中央过于臃肿的问题是不可避免的。

我在工作中需要写一个类似列表筛选的组件，满打满4个筛选项目，我的祖先组件足足1200行...WTF!!!



这显然不能忍，我尝试把事件处理逻辑和本身的V层分开。

### 一次不成熟的尝试
```javascript

const RenderFilter = require("../controller/common/renderFilterResultList");
const Render = new RenderFilter();
// 开放四项筛选逻辑
const OpenFilter = require("../controller/open/openFilter");
//......
/**
 * 清除所有
 * @param {object} e event
 * @returns {null} null
 */
handleClickClearAll(e){
    e.preventDefault();
    Clear.handleClickAny.call(this, 0);
    Clear.handleClickAny.call(this, 1);
//......
//in render function
<FundAchieve
    ref = "fundAchieve"
    config = {this.basicConfig.achieve}
    fundAchieveParam = {this.state.fundAchieveParam}
    handleCilck = {AF.handleClickAchieveList.bind(this)}
    handleClickAny={Clear.handleClickAny.bind(this)}
    warning = {this.state.warningAch}
></FundAchieve>
```

没错...我想出来的方法是传this

我把所有逻辑封装成一个类，每次事件处理调用的时候，把其本身的this传递进去，
为什么？为了能在 逻辑层里面修改state
>（这时候问题就很明显了，MC你没分开啊），

之后带来的问题是之前设计组件的数据结构没有设计好，光state的定义就300行....




于是...把state代表的M层又封装了。

### 这只是看起开很美好
是啊，祖先组件只有300行嘞，eslint也能过了， QA也回测了，要啥自行车啊。

这是运气好罢了，差点循环引用，debugger困难，巨无霸函数，运行效率低，都充斥着这一团翔一般的代码。
>最明显的问题
this的混乱，造成其可读性不好
本质上没有解决复用性差的问题，需要联代引用

以前的中央集权的state设计方式在react0.14之后已经不提倡了。

三国还是要归一啊。
# 五、盛唐时代

`事件处理函数是干嘛的？`
为了复用，事件处理函数应该是想一个管道一样，我给你数据，你输出数据就好，我怎么用，不是你管的。—— 数据处理者的身份

`组件嵌套 臃肿 `
使用stateless component 
虽然依然没有解决嵌套的问题，但是其结构更加明了props通过结构赋值也能够更好传递，解放生产力。

因为没有附加的state 事件处理也是不直接改变state的，module层我们自己控制，复用性好了很多

`数据结构是主心骨`

组件是数据驱动的，单向数据流就要求我们对module层的设计必须合理，否则VC层都要跟着复杂起来，这东西不好说，有点看理论知识积累和经验总结，我本身不过一个空油瓶，能力有限。


所以目前情况下，设计组件的最佳实践是

`stateless component` + `函数式编程思想`  + `良好的数据结构`