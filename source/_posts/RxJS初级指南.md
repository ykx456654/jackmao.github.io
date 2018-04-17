# 1. RxJS是什么？
顾名思义，RxJS是 Reactive programming 的javaScript版本，它起源于  [Reactive Extensions](http://reactivex.io/), 是一个解决异步问题的JS开发库, 带来了观察者模式和函数式编程的相结合的最佳实践，在观察者模式之上，实现了丰富的操作符，适用于各种情况下的序列数据处理。
# 2. RxJS的基本概念
* Observable (可观察对象): 表示一个概念，这个概念是一个可调用的未来值或事件的集合。
* Observer (观察者): 一个回调函数的集合，它知道如何去监听由 Observable 提供的值。
* Subscription (订阅): 表示 Observable 的执行，主要用于取消 Observable 的执行。
* Operators (操作符): 采用函数式编程风格的纯函数 (pure function)，使用像 map、filter、concat、flatMap 等这样的操作符来处理集合。
* Subject (主体): 相当于 EventEmitter，并且是将值或事件多路推送给多个 Observer 的唯一方式。
* Schedulers (调度器): 用来控制并发并且是中央集权的调度员，允许我们在发生计算时进行协调，例如 setTimeout 或 requestAnimationFrame 或其他。

# 2.1 RxJS · 流 Stream
RxJS是一种面向数据流的编程模式，它将所有的操作、事件、数据变化等等，都通过流的方式来  

流，就好像现实生活中的水流一样。既然是水流，就有源头、会一条水流分成多条水流、会多条水流合成一条水流、水流还会变大变小、会干枯、最终会流向目的地……  

而RxJS就是将所有的操作、事件、数据变化等等，当作一条条类似自然界的水流，然后通过一系列的操作符(Operators)来进行管理和输出。  

创建流--->操作流---->输出
```javascript
 import { Observable } from 'rxjs';
// 举个最简单的例子，创建一个promise流(Observable), 返回1  
 const source = Observable.fromPromise(new Promise(resolve => resolve(1)));
// 对这个流进行乘2操作(map Operators)
 const example = source.map(val => val*2);
 // 订阅这个流获取值(Subscription)
 const subscribe = example.subscribe(val => console.log(val));
 //  output: 2
```




# 2.2  举个例子，实现一个简单的canvas的画笔。

业务逻辑： 
* 1.确认鼠标按下，开始笔画开始
* 2.获取鼠标移动的时候划过的点坐标， 绘制
* 3.鼠标放松，停止画笔

首先获取canvas:
```javascript
const canvas  = getById('canvas');
const context = canvas.getContext("2d");
const cRect   = canvas.getBoundingClientRect();
const offsetX = cRect.left;
const offsetY = cRect.top;
```

然后编写业务逻辑，常规写法：
```javascript
// 创建共享变量，随着业务的复杂，共享变量可能越增越多，应用状态管理越来越混乱
let drawing = false;
let prevX   = 0;
let prevY   = 0;

canvas.addEventListener('mousedown', e => {
  prevX = (e.clientX - offsetX);
  prevY = (e.clientY - offsetY);
  drawing= true;
});

canvas.addEventListener('mousemove', e => {
  if (!drawing) { return; }
  let curtX = (e.clientX - offsetX);
  let curtY = (e.clientY - offsetY);
  context.beginPath();
  context.moveTo(prevX, prevY);
  context.lineTo(curtX, curtY);
  context.lineWidth = 5;
  context.stroke();
  context.closePath();
  prevX = curtX;
  prevY = curtY;
});

canvas.addEventListener('mouseup', e => {
  drawing = false;
});

```
RxJS写法：
```javascript
const mousedown$ = Rx.Observable.fromEvent(canvas, 'mousedown');  
const mousemove$ = Rx.Observable.fromEvent(canvas, 'mousemove'); 
const mouseup$ = Rx.Observable.fromEvent(canvas, 'mouseup');  
const moving = mousedown$.flatMap(() => {    
    const drag = mousemove$.takeUntil(mouseup);
    return drag
        .scan((acc, v) => ({ prev: acc.curt, curt: v}),{prev: null, curt: null}).skip(1);
});

moving.subscribe((prevAndCurt) => {
  const prevX = (prevAndCurt.prev.clientX - offsetX);
  const prevY = (prevAndCurt.prev.clientY - offsetY);
  const curtX = (prevAndCurt.curt.clientX - offsetX);
  const curtY = (prevAndCurt.curt.clientY - offsetY);
  context.beginPath();
  context.moveTo(prevX, prevY);
  context.lineTo(curtX, curtY);
  context.lineWidth = 5;
  context.stroke();
  context.closePath();
});

可以看出rxjs只是将建好的各个流，通过一系列的函数式操作符组合起来，来分步执行逻辑，可读性更强、耦合性更低，更方便测试和修改。

```
    
## 2.3 RxJS操作符与交互图
交互图中每条连表示一个数据流，每个球表示每次流的变更，最后一条线表示最合并的流。  

combineLastest表示被组合的每个stream，一旦发射数据变更，必须拿到其余的stream的最新值（当异步时则等待，直到都拿到最新值），组合为新的数据，作为新stream发射的数据变更。  
```
stream1: ————————①——————————②——————————③————————————④—————————⑤——————————|——>
stream2: ———————————ⓐ————————ⓑ————————————ⓒ—————————————————————ⓓ—————————|——>
                combineLastest(stream1, stream2, (x, y) => x + y)
stream:  ———————(①ⓐ)—(②ⓐ)—(②ⓑ)—————(③ⓑ)—(③ⓒ)———(④ⓒ)————(⑤ⓒ)—(⑤ⓓ)——|——>
```
[RxJS弹珠图](http://rxmarbles.com/?spm=a2c4e.11153940.blogcont65027.13.4cac5054droDx6#every)  



# 3. RxJS的特点

### 1 统一了数据来源
RxJs 最大的特点就是可以把所有的事件封装成一个 Observable（可观察对象）。只要订阅这个可观察对象，就可以获取到事件源所产生的所有事件。想象一下，所有的 DOM 事件、ajax 请求、WebSocket、数组等等数据，统统可以封装成同一种数据类型。这就意味着，对于有多个来源的数据，我们可以每个数据来源都包装成 Observable，统一给视图层去订阅，这样就抹平了数据源的差异，统一了数据来源。

点击一次发一个请求，可看做一个DOM事件流转化为AJAX请求流
```javascript
function sendRequest () {
  return fetch('xxx').then(res => res.json())
}
Rx.Observable.fromEvent(document.querySelector('input[name=send]'), 'click')
  .flatMap(e => Rx.Observable.fromPromise(sendRequest()))
  .subscribe(value => {
    console.log(value)
  })
```



### 2 强大的异步同步处理能力
RxJs 提供了功能非常强大且复杂的操作符（ Operator） 用来处理、组合 Observable，因此 RxJs 拥有十分强大的异步处理能力，几乎可以满足任何异步逻辑的需求，同步逻辑更不在话下。它也抹平了同步和异步之间的鸿。  
假设我们要实现一个方法：当有某个值的时候，就返回这个值，否则去服务端获取这个值。  

```javascript
  getData() {
    if(data) {
      return Observable.of(data)
    } else {
      return Observable.fromPromise(PromiseA)
    }
  }

  getData().subscribe(data => console.log(data));
```
  
  





### 3 数据推送的机制把拉取的操作变成了推送的操作
RxJs 传递数据的方式和传统的方式有很大不同，那就是改“拉取”为“推送”。  
原本一个组件如果需要请求数据，那它必须主动去发送请求才能获得数据，这称为“拉取”。  
如果像 WebSocket 那样被动地接受数据，这称为“推送”。如果这个数据只要请求一次，那么采用“拉取”的形式获取数据就没什么问题。但是如果这个数据之后需要更新，那么“拉取”就无能为力了，开发者不得不在代码里再写一段代码来处理更新。但是 RxJs 则不同。RxJs 的精髓在于推送数据。组件不需要写请求数据和更新数据的两套逻辑，只要订阅一次，就能得到现在和将来的数据。这一点改变了我们写代码的思路。我们在拿数据的时候，不是拿到了数据就万事大吉了，还需要考虑未来的数据何时获取、如何获取。如果不考虑这一点，就很难开发出具备实时性的应用。如此一来，就能更好地解耦视图层和数据层的逻辑。视图层从此不用再操心任何有关获取数据和更新数据的逻辑，只要从数据层订阅一次就可以获取到所有数据，从而可以只专注于视图层本身的逻辑。  
假设要实现一个聊天室：socket获取实时消息，fetch获取历史消息，自己发送消息send，不管这个消息从哪里来，我们最终目的都是渲染到聊天界面。  

```javascript
const socket$ = Observable.webSocket('ws://localhost:3000');  // 获取实时消息的流
const fetch$ = Observable.fromPromise(fetch); // 获取历史消息的流
const send$ = Observable.fromEvent(buttom, 'click'); // 自己发送消息的流


const source$ = new Subject();
      .merge(socket$,fetch$,send$)

      source$.subscribe(data => render(data))
```



### 4 缓存数据
RxJS有一种特殊的 Observable，[BehaviorSubject](http://cn.rx.js.org/manual/overview.html#h26)、[ReplaySubject](http://cn.rx.js.org/manual/overview.html#h27)。如果 BehaviorSubject 已经产生过一次数据，那么当它再一次被订阅的时候，就可以直接产生上次所缓存的数据。避免了使用一个全局变量或属性来缓存数据。  

拿BehaviorSubject来举例
  ```javascript
  const subject = new Rx.BehaviorSubject(0); // 0是初始值

  subject.subscribe({
    next: (v) => console.log('observerA: ' + v)
  });
  subject.next(1);
  subject.next(2);
  subject.subscribe({
    next: (v) => console.log('observerB: ' + v)
  });

  subject.next(3);
  // 输出:
  // observerA: 0
  // observerA: 1
  // observerA: 2
  // observerB: 2
  // observerA: 3
  // observerB: 3

  ```






# 4. 参考文章

## 4.1 关于RxJS的介绍

[RxJS 入门指引和初步应用-----买房是为了创业，创业是为了买房](https://zhuanlan.zhihu.com/p/25383159)  
[流动的数据——使用 RxJS 构造复杂单页应用的数据逻辑](https://zhuanlan.zhihu.com/p/23305264)  
[RxJS中文文档](http://cn.rx.js.org/manual/overview.html#h26)  
[RxJS操作符文档](https://rxjs-cn.github.io/learn-rxjs-operators/operators/transformation/switchmap.html)  
[RxJS](https://rxjs-cn.github.io)  


## 4.2 具体业务例子
[响应式编程入门：实现电梯调度模拟器](http://ewind.us/2017/rx-elevator-demo/)  
[用 RxJS 实现一个协同编辑的表格应用](https://juejin.im/post/5a7d01265188257a7262a802)  
[RxJS 实战篇（一）拖拽](http://jerryzou.com/posts/rxjs-practice-01/)  
[RxJS 游戏之贪吃蛇](https://zhuanlan.zhihu.com/p/35457418)