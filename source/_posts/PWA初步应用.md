---
title: PWA初步应用
date: 2018-02-27 16:36:53
tags: [PWA,service work]
categories: web前端
---
pwa应用
=======

# 1 什么是 Service Worker？

Service Worker 是 Chrome 团队提出和力推的一个 WEB API，用于给 web 应用提供高级的可持续的后台处理能力。该 WEB API 标准起草于 2013 年，于 2014 年纳入 W3C WEB 标准草案，当前还在草案阶段。

Service Worker 最主要的特点是：在页面中注册并安装成功后，运行于浏览器后台，不受页面刷新的影响，可以监听和截拦作用域范围内所有页面的 HTTP 请求。

基于 Service Worker API 的特性，结合 
	[Fetch API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API)、
	[Cache API](https://developer.mozilla.org/zh-CN/docs/Web/API/Cache)、
	[Push API](https://developer.mozilla.org/zh-CN/docs/Web/API/Push_API)、
	[postMessage API](https://developer.mozilla.org/zh-CN/docs/Web/API/Worker/postMessage)、
	[Notification API](https://developer.mozilla.org/zh-CN/docs/Web/API/notification)，
	可以在基于浏览器的 web 应用中实现如离线缓存、消息推送、静默更新等 native 应用常见的功能，以给 web 应用提供更好更丰富的使用体验。

## 1.1 Service Worker 特点
* 网站必须使用 HTTPS。除了使用本地开发环境调试时(如域名使用 localhost或者是使用[ngrok](https://ngrok.com/)做内网穿透)
* 运行于浏览器后台，可以控制打开的作用域范围下所有的页面请求
* 单独的作用域范围，单独的运行环境和执行线程
* 不能操作页面 DOM。但可以通过事件机制来处理


## 1.2 Service Worker 浏览器支持情况
![](https://gitlab-wenba.xueba100.com:2443/frontend-example/aixue-pwa-example/raw/master/READMEIMAGE/WX20171120-160604@2x.png)


## 1.3 什么是 PWA

谷歌给以 Service Worker API 为核心实现的 web 应用取了个高大上的名字：`Progressive Web Apps`（PWA，渐进式增强 WEB 应用），并且在其主要产品上进行了深入的实践。那么，符合 PWA 的应用特点是什么？以下为来自谷歌工程师的解答。

Progressive Web Apps 是:

* `渐进增强` – 能够让每一位用户使用，无论用户使用什么浏览器，因为它是始终以渐进增强为原则。  
* `响应式用户界面` – 适应任何环境：桌面电脑，智能手机，笔记本电脑，或者其他设备。  
* `不依赖网络连接` – 通过 Service Workers 可以在离线或者网速极差的环境下工作。  
* `类原生应用` – 有像原生应用般的交互和导航给用户原生应用般的体验，因为它是建立在 app shell model 上的。  
* `持续更新` – 受益于 Service Worker 的更新进程，应用能够始终保持更新。  
* `安全` – 通过 HTTPS 来提供服务来防止网络窥探，保证内容不被篡改。  
* `可发现` – 得益于 W3C manifests 元数据和 Service Worker 的登记，让搜索引擎能够找到 web 应用。  
* `再次访问` – 通过消息推送等特性让用户再次访问变得容易。  
* `可安装` – 允许用户保留对他们有用的应用在主屏幕上，不需要通过应用商店。  
* `可连接性` – 通过 URL 可以轻松分享应用，不用复杂的安装即可运行。  


# 2 Service Worker 的生命周期
Service Worker 的生命周期，使用 Service Worker 大概需要如下几个过程。
		install -> installed -> actvating -> Active -> Activated -> Redundant
![](https://gitlab-wenba.xueba100.com:2443/frontend-example/aixue-pwa-example/raw/master/READMEIMAGE/sw-lifecycle.png)

进入 Redundant (废弃)状态的原因可能为这几种：
* 安装(install)失败
* 激活(activating)失败
* 新版本的 Service Worker 替换了它并成为激活状态

# 3 使用 Service Worker

使用 `HTTPS` 访问，并且 SSL 证书要正确。  
在开发阶段，可以通过localhost(127.0.0.1)使用service worker，也可以使用ngrok生成https地址，但是一旦上线，就需要server支持HTTPS。

## 3.1 注册
在网站页面上注册实现 Service Worker 功能逻辑的脚本。例如注册 /sw/sw.js 文件，参考代码：
```javascript
if ('serviceWorker' in navigator) {
    navigator.serviceWorker
        .register('/sw.js')
        .then(registration => console.log('ServiceWorker 注册成功！作用域为: ', registration.scope))
        .catch(err => console.log('ServiceWorker 注册失败: ', err));
}
```
chrome 浏览器下，注册成功后，可以打开 `chrome://serviceworker-internals/` 查看浏览器的 Service Worker 信息。

###注意:  
Service Worker 的注册路径决定了其 `scope` 默认作用范围。如果 sw.js 是在 /sw/ 路径下，这使得该 Service Worker 默认只会收到 /sw/ 路径下的 fetch 事件。如果存放在网站的根路径下，则将会收到该网站的所有 fetch 事件。如果希望改变它的作用域，可在第二个参数设置 scope 范围。

## 3.2 安装
```javascript
// 用于标注创建的缓存，也可以根据它来建立版本规范
const cacheStorageKey = "minimal-pwa-2";
// 列举要默认缓存的静态资源，一般用于离线使用
const cacheList = [
    '/',
    "index.html",
    "lib/css/main.css",
    "lib/image/logo.png",
    "lib/js/index.js"
];

// self 为当前 scope 内的上下文
self.addEventListener('install', event => {
    // event.waitUtil 用于在安装成功之前执行一些预装逻辑
    // 但是建议只做一些轻量级和非常重要资源的缓存，减少安装失败的概率
    // 安装成功后 ServiceWorker 状态会从 installing 变为 installed
    event.waitUntil(
        // 使用 cache API 打开指定的 cache 文件
        caches.open(cacheStorageKey).then(cache => {
            console.log(cache);
            // 添加要缓存的资源列表
            return cache.addAll(cacheList);
        })
    );
});
```

可以看到，示例中在文件 sw.js 内监听了 install 事件。当 sw.js 被安装时会触发 install 事件，监听该事件可执行安装时要做的事情。示例中是缓存用于离线时使用的静态资源，这也是最常见的行为。  

需要注意的是，只有 `cacheList` 中的文件全部安装成功，`Service Worker` 才会认为安装完成。否则会认为安装失败，安装失败则进入 `redundant` (废弃)状态。所以这里应当尽量少地缓存资源(一般为离线时需要但联网时不会访问到的内容)，以提升成功率。  

安装成功后，即进入等待(`waiting`)或激活(`active`)状态。在激活状态可通过监听各种事件，实现更为复杂的逻辑需求。具体参见后文事件处理部分。  

## 3.3 Service Worker 的更新

如果 sw.js 文件的内容有改动，当访问网站页面时浏览器获取了新的文件，它会认为有更新，于是会安装新的文件并触发 install 事件。但是此时已经处于激活状态的旧的 Service Worker 还在运行，新的 Service Worker 完成安装后会进入 waiting 状态。直到所有已打开的页面都关闭，旧的 Service Worker 自动停止，新的 Service Worker 才会在接下来打开的页面里生效。  

如果希望在有了新版本时，所有的页面都得到及时更新怎么办呢？  

可以在 `install` 事件中执行 `skipWaiting `方法跳过 `waiting` 状态，然后会直接进入 `activate` 阶段。接着在 `activate` 事件发生时，通过执行 `self.clients.claim()` 方法，更新所有客户端上的 Service Worker。示例：  
```javascript
// 安装阶段跳过等待，直接进入 active
self.addEventListener('install', function(event) {
    event.waitUntil(self.skipWaiting());
});

self.addEventListener('activate', evnet => event.waitUntil(
    Promise.all([
        // 更新客户端
        clients.claim(),
        // 清理旧版本
        caches.keys().then(cacheList => Promise.all(
            cacheList.map(cacheName => {
                if (cacheName !== cacheStorageKey) {
                    caches.delete(cacheName);
                }
            })
        ))
    ])
));
```
关于 `clients.claim`： [https://developer.mozilla.org/zh-CN/docs/Web/API/Clients/claim](https://developer.mozilla.org/zh-CN/docs/Web/API/Clients/claim)

## 3.4 Service Worker 相关事件处理
在安装过程中我们实现了资源缓存，安装完成后则进入了空闲阶段，此时可以通过监听各种事件实现各种逻辑。  
### 3.4.1 install 事件

当前脚本被安装时，会触发 `install` 事件，具体参考前文的 `安装` 部分的示例。

### 3.4.2 fetch 事件

当浏览器发起请求时，会触发 `fetch` 事件。

Service Worker 安装成功并进入激活状态后即运行于浏览器后台，可以通过 `fetch` 事件可以拦截到当前作用域范围内的 `http/https` 请求，并且给出自己的响应。结合 `Fetch API` ，可以简单方便地处理请求响应，实现对网络请求的控制。

这个功能是十分强大的。

参考下面的示例，这里实现了一个缓存优先的策略逻辑：监控所有 http 请求，当请求资源已经在缓存里了，直接返回缓存里的内容；否则使用 `fetch API` 继续请求。
```javascript
self.addEventListener('fetch', e => {
    e.respondWith(
        caches.match(e.request).then(function (response) {
            if (response != null) {
                return response
            }
            return fetch(e.request.url)
        })
    )
})
```

### 3.4.3 activate 事件

当安装完成后并进入激活状态，会触发 `activate` 事件。通过监听 `activate` 事件你可以做一些预处理，如对于旧版本的更新、对于无用缓存的清理等。

```javascript
self.addEventListener('activate', e => e.waitUntil(
    Promise.all([
        // 更新客户端
        clients.claim(),
        // 清理旧版本
        caches.keys().then(cacheList => Promise.all(
            cacheList.map(cacheName => {
                if (cacheName !== cacheStorageKey) {
                    caches.delete(cacheName);
                }
            })
        ))
    ])
));
```
传给 `waitUntil()` 的 `Promise` 会阻塞其他的事件，直到它完成。这可以确保清理操作会在第一次 fetch 事件之前完成。
在激活时也可执行 `clients.claim` 方法，更新所有客户端上的 Service Worker。具体参见 `Service Worker 更新` 。

## 3.5 相关插件
* [offline-webpack](https://github.com/NekR/offline-plugin)
* [sw-precache-webpack-plugin](https://github.com/goldhand/sw-precache-webpack-plugin)

## 3.6 相关文章
* [Service Worker 生命周期](https://segmentfault.com/a/1190000006061528)
* [Service Worker API](https://developer.mozilla.org/zh-CN/docs/Web/API/Service_Worker_API)
* [改造你的网站，变身 PWA](https://segmentfault.com/a/1190000008880637)
