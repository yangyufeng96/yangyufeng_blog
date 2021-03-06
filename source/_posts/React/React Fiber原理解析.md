---
type: blog
title: React Fiber原理解析
categories:
  - 前端
tags:
  - React
abbrlink: 8e322b93
date: 2021-03-08 00:00:00
---

 `React`团队重写了`React` 的核心算法---`reconciliation`,一般将之前的算法叫`stack reconciliation`，现在的叫`fiber reconciliation`。 

### React Fiber是什么

 React Fiber是个什么东西呢？官方的一句话解释是“**React Fiber是对核心算法的一次重新实现”**。 

 为什么Facebook要搞React Fiber呢？  在现有React中，更新过程是同步的，这可能会导致性能问题。 

**同步更新过程的局限**

当React决定要加载或者更新组件树时，会做很多事，比如调用各个组件的生命周期函数，计算和比对Virtual DOM，最后更新DOM树，这整个过程是同步进行的，也就是说只要一个加载或者更新过程开始，那React就以不破楼兰终不还的气概，一鼓作气运行到底，中途绝不停歇。

表面上看，这样的设计也是挺合理的，因为更新过程不会有任何I/O操作嘛，完全是CPU计算，所以无需异步操作，的确只要一路狂奔就行了，但是，当组件树比较庞大的时候，问题就来了。

假如更新一个组件需要1毫秒，如果有200个组件要更新，那就需要200毫秒，在这200毫秒的更新过程中，浏览器那个唯一的主线程都在专心运行更新操作，无暇去做任何其他的事情。想象一下，在这200毫秒内，用户往一个input元素中输入点什么，敲击键盘也不会获得响应，因为渲染输入按键结果也是浏览器主线程的工作，但是浏览器主线程被React占着呢，抽不出空，最后的结果就是用户敲了按键看不到反应，等React更新过程结束之后，咔咔咔那些按键一下子出现在input元素里了。

这就是所谓的界面卡顿，很不好的用户体验。

[![68SdWF.md.png](https://s3.ax1x.com/2021/03/09/68SdWF.md.png)](https://imgtu.com/i/68SdWF)

现有的React版本，当组件树很大的时候就会出现这种问题，因为更新过程是同步地一层组件套一层组件，逐渐深入的过程，在更新完所有组件之前不停止，函数的调用栈就像下图这样，调用得很深，而且很长时间不会返回。

<!-- more -->

**React Fiber的方式**

 破解JavaScript中同步操作时间过长的方法其实很简单——**分片**。 

把一个耗时长的任务分成很多小片，每一个小片的运行时间很短，虽然总时间依然很长，但是在每个小片执行完之后，都给其他任务一个执行的机会，这样唯一的线程就不会被独占，其他任务依然有运行的机会。

React Fiber把更新过程碎片化，执行过程如下面的图所示，每执行完一段更新过程，就把控制权交还给React负责任务协调的模块，看看有没有其他紧急任务要做，如果没有就继续去更新，如果有紧急任务，那就去做紧急任务。

维护每一个分片的数据结构，就是**Fiber**。

 有了分片之后，更新过程的调用栈如下图所示，中间每一个波谷代表深入某个分片的执行过程，每个波峰就是一个分片执行结束交还控制权的时机。 

[![68Sco6.md.png](https://s3.ax1x.com/2021/03/09/68Sco6.md.png)](https://imgtu.com/i/68Sco6)

### React Fiber 的特点

- 暂停工作，并在之后可以返回再次开始
- 可以为不同类型的工作设置优先级
- 复用之前已经完成的工作
- 终止已经不再需要的工作

原文地址：https://juejin.cn/post/6844904202267787277