---
title: 下一代native开发工具Flutter
date: 2018-06-06 16:36:53
tags: [Flutter,native]
categories: web前端,大前端,native app, web开发者眼中的Flutter
---
新一代native开发工具Flutter
=======


# 什么是Flutter？
[Flutter](https://flutter.io/)是谷歌新一代的开源移动端开发平台，同时支持 iOS, Android。

# Flutter有哪些特点
* 1.更快的开发速度--------支持热更新
* 2.原生带有艳丽的UI（Material Design）、流畅的动画、丰富的API、丰富的控件系统
* 3.数据驱动视图，响应式编程
  由于笔者是web开发者，所有以下的观点都是从一个web开发者的角度来说的。

    
# Flutter的核心内容

## Widget
Flutter的Widget是响应式的，跟react比较像，都是数据驱动视图。从结构上来说，整个Flutter app都是很多Widget组合起来的。
跟web不一样，Widget不仅仅包含结构布局，还包含了样式和事件。怎么理解呢， 在web开发里面  页面 = dom + css + event,而在Flutter里面， 页面 = Widget, 样式同样也是一个Widget。

 ```javaScript
import 'package:flutter/material.dart';

void main() {
  runApp( //  起始函数， 接收一个 Widget, 并让这个Widget成为根节点
    new Center(  // 这是一个 Widget
      child: new Text(
        'Hello, world!',
      ),
    ),
  );
}
 ```

如果说我要给中间的文字加个圆形的背景呢，如果是web开发，我们在 text包裹标签上加个类，写个background + border-radius 就行了。但是在Flutter里面，我们得额外包几个Widget
```javaScript
  class TextBackground extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
      return new Center(
        child: new ClipRRect(
          borderRadius: new BorderRadius.all(new Radius.circular(5.0)), // 设置圆角 = border-radius: 5px
          child: new Container(
            color: Colors.red, // 设置背景颜色 =  background-color: red
            child: new Text('hello, world!')
          )
        )
      )
    }
  }
```


Flutter提供了非常丰富的Widget控件，一个UI有好几种写法，下面是只包一个使用BoxDecoration的Container的写法，同时给 Text Widget控件增加点击事件
```javaScript
  class TextBackground extends StatelessWidget {
    @override
    Widget build(BuildContext context) {
      return new Center(
        child: new Container(
          decoration: new BoxDecoration(
            color: Colors.red, // 设置背景颜色 =  background-color: red
            borderRadius: new BorderRadius.all(new Radius.circular(5.0)) // 设置圆角 = border-radius: 5px
          ),
          color: Colors.red, // 设置背景颜色 =  background-color: red
          child: new GestureDetector(  // 点击文字事件
            child: new Text('hello, world'),
            onTap: () {
              print('hello, flutter') // 打印 hello, flutter
            }
          )
        )
      )
    }
  }

```
## Flutter常用控件
Flutter提供了一些常用的基础控件：
* Text, 文字组件，可以在制定文字样式
* Row, Column, Flutter的弹性布局控件，跟web的弹性布局一样。Row控件是水平排列， column控件是垂直排列。
* Stack, 层叠控件，在Stack控件内，子组件可以使用left、top、bottom、right等属性相对于父组件布局，相当于web里面的position:absolute
* Container, 矩形控件，相当于web里面的块级元素，可以使用background，border，shadow，padding，margin, width, height等属性
* PageView, 单个的页面控件，可以在里面设置子页面, 在需要在当前页面设置多个子页的时候用到，类似于单页web应用的子路由
* Image, 图片控件, 跟web只需要设置src属性不同， 这里Image加载图片分为加载本地图片(Image.asset(path))跟加载网络图片(Image.network(path))
* ListView, 列表渲染控件， 相当于react开发里面的 Array.map(renderComponent)
......... Flutter的控件实在是太多了，想要全部了解得花好多时间。总的来说，个人觉得Flutter写UI没有web那么灵活，但是更加高级（Fultter已经写好了大部分样式；web啥样式都得自己封装，基础样式惨不忍睹），同样的UI跟交互，Flutter写下来Widget tree 可能比 web dom tree复杂好几倍。写Flutter页面，可能一不小心一个文件就写了好几百行。

