title: 简析动画执行函数requestAnimationFrame
date: 2016-10-25 
categories: 技术
tags: [canvas,js]

---

我们今天讨论一下：

1.为什么用requestAnimationFrame
2.requestAnimationFrame的 作用机制——为什么会更流畅

我们先说 为什么用requestAnimationFrame
为什么？主要是为了解决timer导致以下的两个问题 1.跳帧 2.性能

# 一、跳帧问题

帧数在一定程度上能决定画面的流畅，一般动画剪辑等24帧25帧30帧就足够，电视30帧，人眼因为视觉暂留对动画的判定最低帧数也就是12帧。而我们平时玩的游戏，一般会固定帧速率（framerate）为60帧。


## 1.setTimeout/setInterval 

`timer有两方面的漏洞1.API本身的缺陷 2.浏览器执行的缺陷`

### 1.API本身达不到毫秒级的精确

如果使用 setTimeout或者setInterval 那么需要我们制定时间 假设给予 （1000/60）理论上就可以完成60帧速率的动画。实际上我们做个小demo也能跑。

但是这是很一厢情愿的想法，在HTML5规范中，浏览器为了节省电力消耗可以拉长执行代码的间隔时间[该API不保证计时器将在时间表上精确运行][1].
所以事实是浏览器可以“强制规定时间间隔的下限（clamping th timeout interval）”,一般浏览器所允许的时间再5-10毫秒，也就是说即使你给了某个小于10的数，可能也要等待10毫秒。

### 2.浏览器不能完美执行

我们的显示屏一般是16.7ms（即60FPS）的显示频率，我们看图片的第一行，每次刷新间隔就是黑色箭头的中间。
![突然插入的帧][2]

如果我们突然给了一个10ms的setTimeout，等于在每次刷新间隔中强制插入了一次渲染，超出本身的正常的绘制流程，这种过度绘制的情况会导致动画断续显示，因为会导致部分的第三帧会丢失。（当然显卡超频是另一码事，我们不谈）
打个比方，收银一分钟结两个人，你突然要插队进入，别人就要把你扔出来。
虽然浏览器厂家对timer做了很多优化，使其丢帧不会太严重，但是还是存在。简单的动画丢帧倒是也无妨，但是如果是复杂动画或者游戏丢帧就是很严重的事了。


## requestAnimationFrame

我们都知道个人的发展不仅要看自身的努力也要看历史的进程，而request就是按照浏览器的渲染进行走的。

request不像timer这么粗暴的插入，而是根据浏览器的绘制进行，更多的像是一种同化。
request 会把每一帧中的所有DOM操作集中起来，在一次重绘或回流中就完成（这点很像虚拟DOM不是~），并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率，它该是10ms咱们也10ms绘制，该是16.7咱们也16.7。跟随浏览器的绘制进行。

这样以来，就保证了不会出现过度渲染的问题，保证了流畅的需求以及浏览器的完美渲染。



二性能
性能上的影响有两点，一方面是对当前浏览器的影响，一个是对整个计算机内存的影响。

## timer
timer的渲染机制[导致了动画过度绘制，浪费 CPU 周期以及消耗额外的电能等问题。][3]，
而且，即使看不到网站，特别是当网站使用背景选项卡中的页面或浏览器已最小化时，动画都会频繁出现。
也就是说，你哪怕最小化了页面，切换到了别的页面，耗费着高资源的动画依然在执行。

这种计时器分辨率的降低也会对电池使用寿命造成负面影响，并会降低其他应用的性能。
## request
而request完全没有以上所述方面的性能支出[《setInterval与requestAnimationFrame的时间间隔测试》][4]这篇文章很好的佐证了request的高性能。而其本身的避免了过度渲染也规避了cpu的支出。


----------


参考：
[Animating with javascript: from setInterval to requestAnimationFrame][5]
[CSS3动画那么强，requestAnimationFrame还有毛线用？][6]
[setInterval与requestAnimationFrame的时间间隔测试][7]
[requestAnimationFrame The secret to silky smooth JavaScript animation!][8]
[requestAnimationFrame 性能更好][9]
[基于脚本的动画的计时控制（“requestAnimationFrame”）][10]


  [1]: https://www.w3.org/html/ig/zh/wiki/HTML5/timers
  [2]: https://i-msdn.sec.s-msft.com/dynimg/IC550966.png
  [3]: https://msdn.microsoft.com/library/hh920765%28v=vs.85%29.aspx
  [4]: https://segmentfault.com/a/1190000000386368
  [5]: https://hacks.mozilla.org/2011/08/animating-with-javascript-from-setinterval-to-requestanimationframe/
  [6]: http://www.zhangxinxu.com/wordpress/2013/09/css3-animation-requestanimationframe-tween-%E5%8A%A8%E7%94%BB%E7%AE%97%E6%B3%95/
  [7]: https://segmentfault.com/a/1190000000386368
  [8]: http://creativejs.com/resources/requestanimationframe/
  [9]: http://jinlong.github.io/2013/06/24/better-performance-with-requestanimationframe/
  [10]: https://msdn.microsoft.com/library/hh920765%28v=vs.85%29.aspx