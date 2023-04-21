---
title: MVC、MVP、MVVM架构模式的区别和联系
tags: ["架构"]
categories: 架构
date: 2021-08-17
updated: 2021-08-17
grammar_cjkRuby: true
copyright: ture
---

> 我们都知道，复杂的软件必须有清晰合理的架构，否则无法开发和维护。

MVC、MVP、MVVM这些模式是为了解决开发过程中的实际问题而提出来的，目前作为主流的几种架构模式而被广泛使用。

<!-- more -->

MVC（Model-View-Controller）是最常见的软件架构之一，业界有着广泛应用。它本身很容易理解，但是要讲清楚，它与衍生的 MVP 和 MVVM 架构的区别就不容易了。

接下来，我们就来详细解释一下这三种架构的区别。

### MVC（Model-View-Controller）

MVC即Model-VIew-Controller。他是1970年代被引入到软件设计大众的。MVC模式致力于关注点的切分，这意味着model和controller的逻辑是不与用户界面（View）挂钩的。因此，维护和测试程序变得更加简单容易。

MVC是比较直观的架构模式:
用户操作->View（负责接收用户的输入操作）->Controller（业务逻辑处理）->Model（数据持久化）->View（将结果反馈给View）。

MVC使用非常广泛。比如JavaEE中的SSH框架（Struts/Spring/Hibernate），Struts（View, STL）-Spring（Controller, Ioc、Spring MVC）-Hibernate（Model, ORM）以及ASP.NET中的ASP.NET MVC框架，xxx.cshtml-xxxcontroller-xxxmodel。（实际上后端开发过程中是v-c-m-c-v，v和m并没有关系，下图仅代表经典的mvc模型）。

MVC模式的意思是，软件可以分成三个部分。

![](/images/123123.png)

**视图（View）**：用户界面。
**控制器（Controller）**：业务逻辑
**模型（Model）**：数据保存

各部分之间的通信方式如下：

![](/images/123124.png)

1. View 传送指令到 Controller。
2. Controller 完成业务逻辑后，要求 Model 改变状态。
3. Model 将新的数据发送到 View，用户得到反馈。

**注意**: 所有通信都是单向的。

### MVP（Model-View-Presenter）

MVP是把MVC中的Controller换成了Presenter（呈现），目的就是为了完全切断View跟Model之间的联系，由Presenter充当桥梁，做到View-Model之间通信的完全隔离。

NET程序员熟知的ASP.NET webform、winform基于事件驱动的开发技术就是使用的MVP模式。控件组成的页面充当View，实体数据库操作充当Model，而View和Model之间的控件数据绑定操作则属于Presenter。控件事件的处理可以通过自定义的IView接口实现，而View和IView都将对Presenter负责。

MVP 模式将 Controller 改名为 Presenter，同时改变了通信方向。

![](/images/123125.png)

1. 各部分之间的通信，都是双向的。
2. View 与 Model 不发生联系，都通过 Presenter 传递。
3. View 非常薄，不部署任何业务逻辑，称为"被动视图"（Passive View），即没有任何主动性，而 Presenter非常厚，所有逻辑都部署在那里。

### MVVM（Model-View-ViewModel）

如果说MVP是对MVC的进一步改进，那么MVVM则是思想的完全变革。

它是将“数据模型数据双向绑定”的思想作为核心，因此在View和Model之间没有联系，通过ViewModel进行交互，而且Model和ViewModel之间的交互是双向的，因此视图的数据的变化会同时修改数据源，而数据源数据的变化也会立即反应到View上。

这方面典型的应用有.NET的WPF，js框架Knockout、AngularJS等。

MVVM 模式将 Presenter 改名为 ViewModel，基本上与 MVP 模式完全一致。

![](/images/123126.png)

唯一的区别是，它采用双向绑定（data-binding）：
View的变动，自动反映在 ViewModel，反之亦然。Angular 和 Ember 都采用这种模式。
