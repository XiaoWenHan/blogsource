---
layout: post
title: "Android GridView 点击效果（可能是最快捷的实现，另有福利）"
date: 2015-08-12 16:07:25
categories: Android开发

---

好久没有发过文章了，今天给朋友们分享的是：给GirdView添加点击效果， **点击时改变背景** 。  
这其实也是我在项目开发中遇到的问题，习惯性的百度了一圈，又Google了一圈。解决方案大致可归为两类：  
1. 代码实现，逻辑处理，监听点击时间，改变相应控件的BackGround；
2. Selector实现，用XML定义，Layout中赋予相应属性。

首先第1个解决办法，怎么说呢，虽然逻辑不是很复杂，但也无形中增加了Bug出现的可能。第二个我试了好久，都达不到理想效果。于是我做了一件自认为聪明的做法，没想到居然管用。  
很简单，首先还是 **写一个Selector** ，预先定义按下去是用什么素材，普通状态时什么素材；然后 **GridView本身没有进行任何selector的赋值** ，只是有一个Background和horizontalSpacing，用来显示分割线（哈，不小心又透露了一招）。然后在每个item的布局的根节点， **把Selector作为Background赋值** ，就可以了。