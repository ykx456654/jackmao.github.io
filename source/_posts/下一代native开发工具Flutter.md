---
title: 下一代native开发工具Flutter
date: 2018-06-06 16:36:53
tags: [Flutter,native]
categories: web前端,大前端,native app
---
新一代native开发工具Flutter
=======


# 什么是Flutter？
[Flutter](https://flutter.io/)是谷歌新一代的开源移动端开发平台，同时支持 iOS, Android。

# Flutter有哪些特点
* 1.更快的开发速度--------支持热更新
* 2.原生带有艳丽的UI（Material Design）、流畅的动画、丰富的API
* 3.数据驱动视图，响应式编程


    
# Flutter的核心内容

## Widget
 Flutter的Widget是响应式的，跟react比较像，灵感也是来自于react。从结构上来说，整个Flutter app都是很多Widget组合起来的。Flutter Widget又分为
 ```javaScript
import 'package:flutter/material.dart';

void main() {
  runApp( //  起始函数， 接收一个 Widget, 并让这个Widget成为根节点
    new Center(  // 这是一个 Widget
      child: new Text(
        'Hello, world!',
        textDirection: TextDirection.ltr,
      ),
    ),
  );
}
 ```
Flutter提供了一些常用的基础组件：
* Text, 文字组件，可以在制定文字样式
* Row, Column, Flutter的弹性布局控件，跟web的弹性布局一样。Row控件是水平排列， column控件是垂直排列。
* Stack，层叠控件，在Stack控件内，子组件可以使用left、top、bottom、right等属性相对于父组件布局，相当于web里面的position:absolute
* Container, 矩形控件，相当于web里面的块级元素，可以使用background，border，shadow，padding，margin, width, height等属性。


