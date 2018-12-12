---
title: Progressive Web App
date: 2018-11-02
categories:
- 技术分享
tags:
- pwa
---

# Progressive Web App

## PWA 的主要特点

1. 可靠 - 即使在不稳定的网络环境下，也能瞬间加载并展现
2. 体验 - 快速响应，并且有平滑的动画响应用户的操作
3. 粘性 - 像设备上的原生应用，具有沉浸式的用户体验，用户可以添加到桌面

## PWA的能力

1. 持久缓存
2. 后台同步
3. 消息通知

## Service Worker生命周期

![Service Worker生命周期](/img/pwa/sw-lifecycle.png "Service Worker生命周期")

- 安装中( installing )

  这个状态发生在 Service Worker 注册之后，表示开始安装，触发 install 事件回调指定一些静态资源进行离线缓存

- 安装后( installed )

  Service Worker 已经完成了安装，并且等待其他的 Service Worker 线程被关闭。

- 激活( activating )

  在这个状态下没有被其他的 Service Worker 控制的客户端，允许当前的 worker 完成安装，并且清除了其他的 worker 以及关联缓存的旧缓存资源，等待新的 Service Worker 线程被激活。

- 激活后( activated )

  在这个状态会处理 activate 事件回调 (提供了更新缓存策略的机会)。并可以处理功能性的事件 fetch (请求)、sync (后台同步)、push (推送)。

- 废弃状态 ( redundant )

  这个状态表示一个 Service Worker 的生命周期结束。

## Service Worker支持事件

1. install安装成功后触发
2. activate 安装成功后触发激活状态
3. fetch 在注册的作用域内发起的请求会触发fetch事件
4. push 事件用于消息推送
5. sync 事件用于用户离线数据处理

## demo

Service Worker启用

```js
if('serviceWorker' in navigator) {
  window.addEventListener('load', function() {
    navigator.serviceWorker.register('./sw-demo.js', {
      scope: '/'
    }).then(function (registration) {
      // 注册成功
      console.log('注册成功: ', registration.scope);
    }).catch(function (err) {
      // 注册失败
      console.log('注册失败: ', err);
    });
  })
}
```

sw-demo.js

```js
// cacheStorage API
// https://developer.mozilla.org/zh-CN/docs/Web/API/CacheStorage

var cacheName = 'pwa-cache-v1';
var cacheList = [];
// install事件
self.addEventListener('install', function (e) {
  e.waitUntil(self.skipWaiting());
});
// 激活事件
self.addEventListener('activate', function (e) {
  e.waitUntil(self.clients.claim());
});
self.addEventListener('fetch', function (event) {
  const url = new URL(event.request.url);
  if (url.pathname !== '/pwa.html') {
    event.respondWith(
      caches.match(event.request, {
        ignoreSearch: true,
      }).then(function (response) {
        if (response) {
          return response;
        }
        var requestToCache = event.request.clone();
        return fetch(requestToCache).then((response) => {
          if (!response || response.status !== 200) {
            return response;
          }
          var responseToCache = response.clone();
          caches.open(cacheName).then(function (cache) {
            cache.put(requestToCache, responseToCache);
          });
          return response;
        })
      })
    )
  }
});
```

### webp应用场景

```js
// install事件
self.addEventListener('install', function (e) {
  e.waitUntil(self.skipWaiting())
})
// 激活事件
self.addEventListener('activate', function (e) {
  e.waitUntil(self.clients.claim());
});
self.addEventListener('fetch', function (event) {
  if (/\.jpg$|.png$/.test(event.request.url)) {
    var supportWebp = false;
    if (event.request.headers.has('accept')) {
      supportWebp = event.request.headers.get('accept').includes('webp');
    }
    if (supportWebp) {
      var req = event.request.clone();
      var returnUrl = req.url.substr(0, req.url.lastIndexOf('.')) + '.webp';
      event.respondWith(
        fetch(returnUrl, {
          mode: 'no-cors',
        })
      );
    }
  }
});
```

## Workbox

### Workbox默认的五种缓存策略

- staleWhileRevalidate 使用缓存同时更新缓存

![staleWhileRevalidate](/img/pwa/Stale-While-Revalidate.png "staleWhileRevalidate")

- networkFirst 优先网络

![Network First](/img/pwa/NetworkFirst.png "Network First")

- networkOnly 只用网络

![Network Only](/img/pwa/NetworkOnly.png "Network Only")

- cacheFirst 优先缓存

![Cache First](/img/pwa/CacheFirst.png "Cache First")

- cacheOnly 只用缓存

![Cache Only](/img/pwa/CacheOnly.png "Cache Only")

### Workbox demo

```js
importScripts('https://g.alicdn.com/kg/workbox/3.3.0/workbox-sw.js');
workbox.setConfig({
  modulePathPrefix: 'https://g.alicdn.com/kg/workbox/3.3.0/'
});
if (workbox) {
  console.log(`Yay! Workbox is loaded 🎉`);
} else {
  console.log(`Boo! Workbox didn't load 😬`);
}
workbox.routing.registerRoute(
  new RegExp('.*\.(jpg|png)$'),
  workbox.strategies.staleWhileRevalidate()
);
workbox.routing.registerRoute(
  new RegExp('.*\.html'),
  workbox.strategies.networkFirst()
);
workbox.routing.registerRoute(
  new RegExp(),
  workbox.strategies.networkOnly()
);
workbox.routing.registerRoute(
  new RegExp('.*\.(?:js|css)'),
  workbox.strategies.cacheFirst()
);
workbox.routing.registerRoute(
  new RegExp(),
  workbox.strategies.cacheOnly()
);
// 自定义策略
const handlerCb = ({url, event, params}) => {
  var supportWebp = false;
  if (event.request.headers.has('accept')) {
    supportWebp = event.request.headers.get('accept').includes('webp');
  }
  if (supportWebp) {
    var req = event.request.clone();
    var returnUrl = req.url.substr(0, req.url.lastIndexOf('.')) + '.webp';
    return fetch(returnUrl, {
      mode: 'no-cors',
    }).then((response) => {
      return response;
    });
  }
};
workbox.routing.registerRoute(
  new RegExp('.*\.png$'),
  handlerCb,
);
```

## PWA学习资料

[PWA官方文档](https://developers.google.cn/web/progressive-web-apps/ "Title")
[Workbox文档](https://developers.google.com/web/tools/workbox/ "Title")
[Lavas](https://lavas.baidu.com/pwa "Title")
[点击下载demo](/asset/pwa-demo.zip "Title")
