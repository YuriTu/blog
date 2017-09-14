
title: 调教磨人的小妖精 —— canvas.context.save()；sa/res的最佳实践
date: 2016-12-11 
categories: 技术
tags: [canvas,js]
---

我们在日常借（kao）鉴（bei）别人的代码的时候往往会看到 context.save() || restore()。
然后我们看文档就知道了，这是保存当前画布的状态.....个P啊！！！

![此处输入图片的描述][1]


----------


保存了什么？
为什么要保存这些？
为什么要使用？
何如判断是否需要使用？
最佳实践是什么？

完全啥没有好么！！！！



定义大家都懂，但是还是过不好一生啊！！！

然而实际上， save（） restore（）这一对磨人的小妖精在实际开发中非常重要，所以我们就来，好好和她们进行一下深入的~~交流~~学习。

## **save（）是 将当前状态放入栈中，保存 canvas 全部状态的方法。**

### 一、那么首先 ：**canvas状态** 是什么？

他不是保存canvas画布上画了什么，而是 用一个栈 将  那些你需要在`接下来使用`(一、3解释)的`工具或属性`的状态的当前状态定义并保存。

### 一、1 保存了哪些工具

1.确立的`变换 矩阵` (**敲黑板** 比如 translate、rotate、scale)
2.建立的裁剪区域
3.创建的虚线列表
### 一、2 保存了哪些属性
常用的有  strokeStyle、 fillStyle、globalAlpha、lineWidth、shadowOffsetX、shadowOffsetY、 shadowBlur,、shadowColor、 font、 textAlign。 

### 一、3 为什么用 为什么保存
上述状态其实是在动画中经常改变的变量，如果每次都要重新赋值...那得多长的代码量啊。。。

所以
接下来
如果你需要在你之后的动画中 改变上述的 状态，那么你就需要进行save，除非你要丢弃这些变量曾经的值。


这里我们看个🌰
```javascript
class Clock {
    constructor(props) {
        this.c = props;
        this.x = 100;
        this.y = 100;
        this.width = 30;
        this.height = 20;
        this.degrees = 45;
    }
    init(){
        // c === canvas.context
        const c = this.c;
        c.fillStyle = "reb(80,80,120)";
        c.strokeStyle = "red";
        //两次画矩形参数完全一样
        c.fillRect(this.x,this.y,this.width,this.height);

        c.save();

        c.rotate(this.degrees * Math.PI / 180);
        c.scale(1,1);

        c.translate(1,1);
        // c.restore();
        c.fillStyle = "gold";
        //两次画矩形参数完全一样
        c.fillRect(this.x,this.y,this.width,this.height);
    }
}
```

在restore被注释的情况下

> 嗯...意识流画风，将就看

![此处输入图片的描述][2]

在restore生效的情况下 
![此处输入图片的描述][3]

因为之前save中已经保存了transformations相关信息（rotate、scale等），经过restore，相关数据被推出栈，`覆盖`原有数据，金色矩形按照正常坐标轴输出，直接甩在黑矩形脸上。

所以，你可以用任意多次save，但是restore次数要比save少。

## 二、最佳实践
前面我们提到，save最明显的作用是减少代码量，这么单调的API能活到现在，一定有其他的作用。

## 小妖精的技能：

### 1.方便的进行初始化 ~~时间回朔 The World！！~~ 

假定在栈中，你只有一次的状态改变，那么你就可以，在最开始定义初始态的时候进行一次save（），使用restore来重置状态，非常方便。

### 2.默认属性的多次使用 ~~咲夜の世界！！~~

比如有的时候我们会有一套 默认 状态需要被重复利用。

例如我们要画很多个星星什么的，它有很多细节，我们将一套默认状态保存，以免我们重复的设置 set/get function，角度、旋转、字体等等。因为也许你的每一帧这些属性都在改变。

> 不是像我们的例子那么简单，这点非常常用

在很多项目中，如果我们看的深一点，我们就可以发现，别人经常会设置默认值，我看过的不少代码都喜欢这么做

在每一次的 step（你可以理解成一次执行动画或一帧）中， 确定相对新的坐标系，将新到达的绝对坐标点（相对于屏幕），设定为相对初始坐标点，之后使用相对的移动，到达实际绝对定位的坐标。

因为每一次的transformat都是相对之前坐标的。

这...有什么用？ 往下看。
![此处输入图片的描述][4]
### 3.简化计算 ~~光速「Ｃ．リコシェ」~~

> 毕竟，在你画图的时候，你也许就不知道x、y值

说~
我们要加一张图在h屏幕中间，我们一般会这么写

```javascript
context.drawImage(myImg, x, y, myImg.width, myImg.height);
```
当然你可以这么搞，没人阻止你...

可事实上，往往需要计算相对于 画布的“相机”的真实的x、y值，好吧，那我们就在timer里面算吧。。

如果你画的是一个人呢？有帽子，有手的位置，裙子的摆动，等等，你要将这些东西与本体人物割裂开做绝对坐标计算？

当然第一反应是，人物坐标body.x 那么手的坐标就是 body.x - hand.x 或者 + sin（x）....等等等等

嗯...如果你能搞出来 挺厉害的...草稿写一本子了吧，当年的高数是满分吧？


----------

所以！！这种情况我们果断采用使用相对默认坐标，改变坐标轴，进行矢量计算就好了。

比如手相对人物坐标长6个px，与人物夹角20°，那就直接旋转20°坐标轴画6个px，不比计算一大堆三角函数好搞得多？绘制完成restore回默认坐标点，继续下一个。

----------

所以，save/restore是相当常用也好用的API，好好利用会给我们带来很多的便利，以上的一些最佳实践只是抛砖引玉，欢迎探讨~


参考：
1. http://stackoverflow.com/questions/16409105/whats-the-purpose-of-canvas-context-save-and-restore-in-this-example
2. https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/save


  [1]: http://odf9m3avc.bkt.clouddn.com/%E6%B5%B7%E6%9C%AA-%E8%BF%99%E4%B8%AA%E4%BA%BA%E6%98%AF%E4%B8%8D%E6%98%AF%E5%82%BB%E4%BA%86.jpg
  [2]: http://odf9m3avc.bkt.clouddn.com/test-re-title.png
  [3]: http://odf9m3avc.bkt.clouddn.com/test-no-re.png
  [4]: http://odf9m3avc.bkt.clouddn.com/%E5%8D%81%E5%85%AD%E5%A4%9C%E5%92%B2%E5%A4%9C.png