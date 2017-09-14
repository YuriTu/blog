title: MATRIX项目中的React组件通信小结
date: 2016-09-07 
categories: 技术
tags: [react,js]

---
2017/03注：这是初期的探索，一些方式已经不适用
在我们的开发过程中，会出现复杂的组件交互，而主要的组件交互关系为

>1.组件内部的通信
2.父子关系的通信
3.同级关系的通信

----------
# 组件内的通信
最简单的一种形式，使用事件函数进行监听即可。

----------
# 父子关系的通信
父子关系的思想核心是`props的传递`
>父子关系一般有三种情况：
1.父——子
2.子——父
3.多层嵌套


## 一 父-子 通信
父子间的通信一般多由props属性携带数据完成功能。
结合我们的项目,`父-子通信`最典型的是Lever写的搜索框。这里import了react-typeahead组件

```javascript
<div className="input-box">
    <Typeahead 
    data={ this.state.fundList } 
    highlight="true" 
    placeholder="请输入基金名称、代码或拼音"
    queryChange={this.getQueryList.bind(this)}
    onFocus={this.getList.bind(this)}>
    </Typeahead>
</div>
```
在这里加载了 react-typeahead 在header组件中的state.fundList 作为了 Typeahead 的props.data属性。

## 二 子-父 关系
同样是依靠props进行通信，但是子父关系则需要通过在父级组件中设置方法，在子组件中调用this.props.somefunction来实现。

结合我们的项目。
我在Footer组件中制造了假数据，在App中建立了用于存储的state以及改变state的方法
```javascript
// In App Components
constructor(){
    super();
    this.state = {
        scoreNotes : []//这个数组里面是一堆的对象
    }
}
addScore(newItem){
    this.setState({
        scoreNotes : newItem
    });
}
```
给于 Footer.props.addScore方法，拿到数据后会存储在state里面
```javascript
//In app render
<Footer scoreNotes = {this.addScore.bind(this)}></Footer>
```
在组件Footer的DOM中进行了一次触发
```javascript
//In Footer Compoents
var _score = [
    {name: 'nico',code : '000000'},
    {name: 'honoka',code : '000001'},
    {name: 'eli',code : '000002'},
    {name: 'nozomi',code : '000003'},
    {name: 'maki',code : '000004'},
];

...

//并在组件的DOM中进行了一次触发
<button onClick = {this.props.scoreNotes(this.state.data)}></button>
```

## 小tips
在开发过程中我发现，很多的bug都是this的指向判断错误了，而总部能在每次写this之前都顺一遍this是什么，太费劲了.不开心-.-....
所以我决定放弃this，在 `构造函数` 中显式指定this...

```javascript
getInitialState: function () {
        _StudentScoreTable = this;
        // 把StudentScoreTable组件赋值给一个变量，以便在其它组件中可以使用此组件的方法
        return {
            //something...
        }
    },
```
呼...这个世界安静了...妈妈再也不怕我找不到this是啥了O(∩_∩)O~

而且如果在一个文件中有多个类，通过全局变量也很容易访问其他类的方法与字段，非常方便。可以实现`单个文件`中的多层嵌套的类的通信，直接去除中间部分通信。

----------
# 同级关系()

同级关系有以下的的解决思路

##1.设置父级state——通过对父级state的监听来解决
这种方法实际上就是 父子关系的多次利用，这样就把同级关系转化为了`子1 —— 父 —— 子2`，两次的父子关系。

结合上面的例子
```javascript
//In App component render
<RightNav scoreNotes = {this.state.scoreNotes}></RightNav>
```
在从Footer拿到数据了之后，存储在了App.state.scoreNotes中，之后在传给了RightNav，这样Footer与RightNav就完成了一次通信

好处：简单、好写、学习成本低。
弊端：各组件间有耦合，内聚差，违反了react组件化的思想。总是要构建一个父级组件来通信，如果嵌套关系很复杂就没有办法使用。

## 2.观察者模式
>对于跨文件的复杂通信情况，中心思想是`去掉中间环节`，只留下handle与target的通信，所以我打算把 `数据 绑定在原型链` 上，之后传递对象来达到通信的效果。因为组件间的通信并不是向ajax请求接口一样，存在项目内部的发送与接收(TAT请求接口反而更简单)


### Event Emitter/Target/Dispatcher 模式，类似以前的事件绑定机制
我们需要一个handle来绑定与触发

结合项目，我在Footer中创建了一个用于触发的 testbtn，并在Footer渲染完成后绑定了一个click事件：

```javascript
// In Footer Component  |this is Footer.js
//_score是测试数据，前面有
componentDidMount(){
    $('.testbtn').on('click',function () {
        $('.testbtn').__proto__.datalist = _score
        console.log($('.testbtn').__proto__.datalist)
        return $('.testbtn')
    })
}
//In Footer render 
<button className="testbtn">testbtn</button>
```
把数据绑定在原型链上，接收数据的组件只要访问原型链就可以拿到数据完成通信。

```javascript
//In RightNav component |this is RightNav.js
componentDidUpdate(){
    console.log($('.testbtn'))//
    var data = $('.testbtn').__proto__.datalist
    console.log(data)
}
```



### PubSub模式/Signals——全局事件系统
个人发现其实就是在最大的父级组件中设置了多个全局的事件，根据事件的名字，所有子组件进行调用，还是要和其他组件耦合...还不如我们直接用JQ的on来的更简单实用。。
最关键的是....它还得引入其他库文件和依靠flux架构...项目不涉及，所以这里不多说。仅贴出来实例，研究flux架构的同学可以试试

>(2) Publish / Subscribe
特点：触发事件的时候，你不需要指定一个特定的源，因为它是使用一个全局对象来处理事件（其实就是一个全局
广播的方式来处理事件）
```
// to subscribe
globalBroadcaster.subscribe(‘click’, function() { alert(‘click!’); });
// to dispatch
globalBroadcaster.publish(‘click’);
```
(3) Signals

特点：与Event Emitter/Target/Dispatcher相似，但是你不要使用随机的字符串作为事件触发的引用。触发事件的每一个对象都需要一个确切的名字（就是类似硬编码类的去写事件名字），并且在触发的时候，也必须要指定确切的事件。

```
// to subscribe
otherObject.clicked.add(function() { alert(‘click’); });
// to dispatch
this.clicked.dispatch();
```
###没关系、不知道什么关系.. 组件的通信
复杂的情况往往都可以转化为简单情况，归根结底所有的关系都能转化成`父子或同级`的关系，再结合我们上述的方法，进行套用即可~ 

# 坑
1.ReactDom.findDOMNodes（this.refs.element）可以得到目标的元素节点。但是它的使用需要`const ReactDom = require("react-dom");`,多个节点只能拿到第一个。
2.一般情况下，想要render的更新，需要触发
setState\setProps
forceUpdate 
如果在render中触发以上情况...可能该显示的该拿到的数据都没问题，但是一会就报堆栈溢出了...因为在无限循环的调用render


----------
引用文献
[1] How to communicate between React components
http://ctheu.com/2015/02/12/how-to-communicate-between-react-components/
[2]React 组件之间如何交流
https://segmentfault.com/a/1190000004044592#articleHeader5
[3]react组件间通信
http://www.alloyteam.com/2015/07/react-zu-jian-jian-tong-xin/#prettyPhoto


  [1]: http://i1.buimg.com/e35cb968770dde90.gif