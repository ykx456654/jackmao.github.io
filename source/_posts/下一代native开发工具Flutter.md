---
title: 下一代跨平台开发工具Flutter
date: 2018-06-06 16:36:53
tags: [Flutter,native,跨平台]
categories: web前端,大前端,native app, web开发者眼中的Flutter
---
下一代跨平台开发工具Flutter
=======

# 跨平台移动应用的发展历程
* 1.Webview + Cordova（Phonegap）。第一代的跨平台应用基本上都是一个webview里面跑HTML，优点是简单，跟前端开发并无太大的差异，直接写HTML代码就行了，缺点就是webview性能一般，应用不够流畅。
* 2.ReactNative\Weex。 第二代跨平台应用是以Facebook的ReactNative为代表的  **原生 + JS桥** 应用，也是目前市场上主要得跨平台解决方案，原理是把JS翻译成原生代码，总体来说比单纯的webview跑HTML性能高出不少。但是还是受限制于 **JS桥** 的性能瓶颈。
* 3.Flutter。Flutter是以dart语言为基础的新一代移动应用开发框架。可以说是真正做到了一套代码，三端运行，同时也解决了上面JS桥性能瓶颈的问题。Flutter是一个Hybird但又不像Hybird的框架，先不管将来Flutter会发展的怎样，就它带来了技术的变更以及对目前行业中一些优秀解决方案的整合，都是值得我们去学习了解的。


# 什么是Flutter？
[Flutter](https://flutter.io/)是谷歌新一代的开源移动端开发平台，同时支持 iOS, Android。Flutter采用dart语言开发，[dart](https://baike.baidu.com/item/DART/22500518?fr=aladdin)语法跟JS比较像，对于web开发者来说学习dart难度不是很大，看个两天文档就可以开撸了。
[dart文档](https://www.dartlang.org/)

# Flutter有哪些特点
* 1.更快的开发速度--------支持热更新
* 2.原生带有艳丽的UI（Material Design）（自带 tree shaking）、流畅的动画、丰富的API、丰富的控件系统
* 3.数据驱动视图，响应式编程
* 4.对比之前的跨平台方案，性能更高，兼容性更好
  由于是web开发者，所有以下的观点都是从一个web开发者的角度来说的。

    
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
......... Flutter的控件实在是太多了，想要全部了解得花好多时间。总的来说，个人觉得Flutter写UI没有web那么灵活，但是更加高级（Fultter已经提供了大部分样式，兼容性好；web啥样式都得自己封装，基础样式惨不忍睹，各个浏览器表现有差异），同样的UI跟交互，Flutter写下来Widget tree 可能比 web dom tree复杂好几倍。写Flutter页面，可能一不小心一个文件就写了好几百行。


# Flutter开发时的不足

## 1.Widget Tree层级过深
  就如之前所说的, Flutter的一切都是Widget, 从布局到样式再到手势, 都是 Widget, 在页面足够复杂的情况下，没那么好检查Widget Tree。

## 2.开发时状态的跟踪
  在写demo的过程中，个人感觉最不爽的除了过深的Widget tree之外，就是Flutter开发模式对数据的追踪不够直观，无法从IDE上直接看出State的状态。虽然Flutter提供了[Flutter inspector](https://flutter.io/inspector/)这样的UI调试器，但是作为一个数据驱动视图的开发工具，其界面的核心应该是数据，应该提供类似于React-devTools，Vue-devTools那样可以直接看到数据变化的。

## 3.打包后的包体积
  写完demo后，打了一个ios的包，发现包的大小居然有30M，可是我只写了300行不到的代码，相比对ReactNative（只有几M） ,Flutter打包的体积要大上不少，希望以后能够优化一下。

## 4.生态圈
  Flutter生态圈目前不太繁荣，好用的依赖包较少。不过这个好像也不算什么大问题，毕竟Flutter1.0才刚刚发布，从关注度上来说Flutter已经足够火热，在跨平台的整体趋势下，Flutter肯定会走得更远。


# 开发Flutter的编辑器推荐
  不推荐使用vscode（缩进有点问题）、sublime这样的轻型编辑器，一是一旦嵌套层数过深，修改起来太麻烦，二是经常要增加包裹控件跟减少控件，手动操作太容易出问题。intelliJ跟Andriod Studio开发Flutter应用要更加方便，可以使用Flutter Inspector(UI inspector)、Flutter outline(可以在选中的控件上增加、删减控件，大大减少了层层嵌套的控件修改时的难度)、保存的时候自动热加载（不用再命令行里手动reload）、代码提示（可以显示Widget所有属性）。


# Flutter展望
  不仅仅可以开发ios跟Andriod，Flutter还是谷歌新系统Fuchsia的用户界面基础，万一以后Fuchsia取代Andriod了，Flutter的发展将更加迅猛。因此，大家没事的时候可以多看看Flutter文档，放下技术之争，多写写demo，技多不压身嘛。
