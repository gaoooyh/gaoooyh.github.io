---
layout: post
title: IoC控制反转和依赖注入
date: 2020-10-14 11:00:00 +0800
category: note
thumbnail: /style/image/note.png
icon: note
---

* content
{:toc}

# 控制反转是什么?
Ioc—Inversion of Control,即“控制反转”,是一种设计思想.

在Java开发中,IoC意味着将你设计好的对象交给容器控制,而不是传统的在你的对象内部直接控制.

* 谁控制谁,控制什么:传统Java SE程序设计,我们直接在对象内部通过new进行创建对象,是程序主动去创建依赖对象;
  
  而IoC是有专门一个容器来创建这些对象,即由IoC容器来控制对象的创建; 谁控制谁? 当然是IoC 容器控制了对象; 控制什么? 那就是主要控制了外部资源获取（不只是对象包括比如文件等）.

* 为何是反转,哪些方面反转了: 有反转就有正转,传统应用程序是由我们自己在对象中主动控制去直接获取依赖对象,也就是正转; 而反转则是由容器来帮忙创建及注入依赖对象; 为何是反转? 因为由容器帮我们查找及注入依赖对象,对象只是被动的接受依赖对象,所以是反转; 哪些方面反转了? 依赖对象的获取被反转了.



# 什么是依赖注入
DI—Dependency Injection，即“依赖注入”：组件之间依赖关系由容器在运行期决定，即由容器动态的将某个依赖关系注入到组件之中。

依赖注入的目的并非为软件系统带来更多功能，而是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。

* 谁依赖谁：应用程序依赖于IoC容器；

* 为什么需要依赖：应用程序需要IoC容器来提供对象需要的外部资源；

* 谁注入谁：很明显是IoC容器注入应用程序某个对象，应用程序依赖的对象；

* 注入了什么：就是注入某个对象所需要的外部资源（包括对象、资源、常量数据）。


IoC和DI其实是同一个概念的不同角度描述，由于控制反转概念比较含糊（可能只是理解为容器控制对象这一个层面，很难让人想到谁来维护对象关系），所以2004年大师级人物Martin Fowler又给出了一个新的名字：“依赖注入”，相对IoC 而言，“依赖注入”明确描述了“被注入对象依赖IoC容器配置依赖对象”。


Spring所倡导的开发方式就是如此，所有的类都会在spring容器中登记，告诉spring你是个什么东西，你需要什么东西，然后spring会在系统运行到适当的时候，把你要的东西主动给你，同时也把你交给其他需要你的东西。所有的类的创建、销毁都由 spring来控制，也就是说控制对象生存周期的不再是引用它的对象，而是spring。对于某个具体的对象而言，以前是它控制其他对象，现在是所有对象都被spring控制，所以这叫控制反转。

IoC的一个重点是在系统运行中，动态的向某个对象提供它所需要的其他对象。这一点是通过DI（Dependency Injection，依赖注入）来实现的

那么DI是如何实现的呢？ Java 1.3之后一个重要特征是反射（reflection），它允许程序在运行的时候动态的生成对象、执行对象的方法、改变对象的属性，spring就是通过反射来实现注入的。

　　理解了IoC和DI的概念后，一切都将变得简单明了，剩下的工作只是在spring的框架中堆积木而已。

控制反转IoC(Inversion of Control)是说创建对象的控制权进行转移，以前创建对象的主动权和创建时机是由自己把控的，而现在这种权力转移给IoC容器