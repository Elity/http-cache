name:inverse
class: center, middle,inverse
layout: true

---

# Http Cache

Fighting

---

## 为什么需要缓存？

???

21 世纪，什么最贵？

---

![21世纪](https://s2.ax1x.com/2019/05/23/VPpkM4.png)

???

21 世纪，什么最贵？ 时间

21 世纪，什么东西便宜？ 存储

## 缓存的本质，用空间换时间

---

## 你所知的缓存有哪些？

???

### 思考：一次网络请求打交道的缓存有哪些？

---

- cookie、webstorage
- 虚拟 dom
- 内存
- dns 缓存
- 静态资源缓存
- redis/memcached
- ...

???

- 硬件缓存：cpu、硬盘都有分级缓存，内存就是用于硬盘与 cpu 之间的缓存
- 客户端缓存：https session 缓存、浏览器 dns 缓存、静态资源缓存
- 网络链路：路由表、cdn
- 接入层： ng
- 服务器应用层： redis、memcached、连接池、mysql 查询缓存、动态页面静态化

---

## 你了解哪些缓存算法？

---

- LFU (Least Frequently Used) 淘汰访问频率低的数据
- LRU (Least Recently Used ) 淘汰最长时间未被访问的数据
- MRU (Most Recently Used ) 淘汰最近最常使用的条目
- ...

???

FIFO、ARC

https://searchstorage.techtarget.com/definition/cache

---

## Http Cache

???

我们的互联网就建立在缓存之上，而 http cache 只是缓存的冰山一角

由 http 协议所定义的缓存策略，用于提高网络请求性能

我今天要分享的只是我现阶段对缓存的理解，还未涉及代理层(cdn)的缓存策略

若有错误、欢迎指正

---

## 你会怎么设计 http 缓存

???

设计一个可扩展的协议是多么不容易

---

![未命中本地缓存的情况](https://s2.ax1x.com/2019/05/26/VVmN1U.png)

---

![缓存](http://www.ccc5.cc/wp-content/uploads/2017/12/QQ%E6%88%AA%E5%9B%BE20180825150350.jpg)

???

这个图是我在 2017 年 12 月画的，你们觉得有没有问题

- 未考虑启发式缓存
- 未体现时序

---

## 强缓存

???

所谓强缓存，就是在缓存生效期间，不会向服务器发起任何网络请求

---

.left-column[

### Expires

]
.right-column[
.left[

- http1.0 规范引入
- 设置一个绝对日期格式
- 只会出现在 response header
- http1.0 时有一个与之对应的 request header，Pragma: no-cache

]
]

???

[Expires mdn](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Expires)

> 由于设置的是绝对时间，要求服务器与客户端的时间一致，否则无法精确控制

---

.left-column[

### Expires

### Cache-Control

]
.right-column[
.left[

取值

- no-cache
- no-store
- max-age
- public
- private

要点

- 既会在 request header 也会在 response header
- no-cache 与 no-store 区别

]
]

???

还有好几个配置，不需要死记，当有特别的缓存需求的时候，记得你还有备选方案，再去查手册

[Cache-Control mdn](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Cache-Control)

---

## 协商缓存

???

与服务器协商，是否使用本地缓存

---

.left-column[

### Last-Modified

]
.right-column[
.left[

- response header
- 值为文件最后被修改的日期
- 请求头里的 If-Modified-Since 被用来与之比较

]
]

???

[Last-Modified mdn](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Last-Modified)

---

.left-column[

### Last-Modified

### ETag

]
.right-column[
.left[

- http1.1 引入
- GET、HEAD 请求携带 If-None-Match 与之比较确定是否使用缓存
- PUT、POST 请求携带 If-Match 与之比较是否能更新服务端资源

]
]

???
[演示网址](http://fighting.com/)

- [ETag mdn](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/ETag)
- [If-Match mdn](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-Match)
- [If-None-Match mdn](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/If-None-Match)

---

## 这样就 OK 了？

???

演示 https://www.shiguangkey.com 首页缓存

反思之前写的文章： https://www.ccc5.cc/2351.html

---

## 启发式缓存

---

启发式缓存

- 未明确设置强缓存时生效
- 缓存时间=(Date-Last_Modified)\*0.1

???

不希望页面走启发式缓存，那就明确指定强缓存。

---

## 还漏了什么？

???

[fetch 请求已经被 img 加载的图片](http://fighting.com/vary.html)

演示要点：

- 默认无法请求
- 添加 query 参数可以正常请求

首先追问原因，http cache 是基于什么缓存的

---

- 基于什么缓存？
- 缓存了哪些东西？

???

基于请求方式 + URL 缓存

缓存 所有响应

引入解决方案

---

## Vary

???

一个跟缓存息息相关的 http 响应头

[Vary mdn](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Vary)

---

## Vary:Origin

???

为什么配置 `Vary:Origin` 响应头可以解决问题？

`Origin`头只会出现在`CORS请求`和`POST`请求中。

浏览器在哪些情况下会发起 CORS 请求，哪些情况下发起非 CORS 请求，是有严格规定的。比如在一般的 <img>标签下发起的就是个非 CORS 请求，而在 XHR/fetch 下默认发起的就是 CORS 请求；还比如在一般的<script>标签下发起的是非 CORS 请求（所以才能有 jsonp），而在新的 <script type="module"下发起的是 CORS 请求。

- img 标签默认加载图片是非 cors
- fetch 默认是 cors 请求 （设置 mode:'no-cors'无法取到 response）

另外的解决方案：img 标签加载图片设置 crossorigin

[Origin mdn](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Origin)

[那些年被我们忽视的 Vary:Origin](https://www.ccc5.cc/2375.html)

---

## 打开网址、F5、Ctrl+F5 什么鬼区别

???

首先通过打开网址、F5、Ctrl+F5 演示请求头的不一致性

- 打开网址： 无 cache-control 头
- F5： Cache-Control: max-age=0;
- Ctrl+F5: Cache-Control: no-cache;

1. 打开 https://crm.shiguangkey.com/#/login
2. 分别请求如下：(用 fiddler 抓包，chrome 显示的 http status 不正确)

```javascript
fetch('https://crm.shiguangkey.com/img/logn_p@2x.164071a7.png');

fetch('https://crm.shiguangkey.com/img/logn_p@2x.164071a7.png', {
  headers: { 'Cache-Control': 'max-age=0' },
});

fetch('https://crm.shiguangkey.com/img/logn_p@2x.164071a7.png', {
  headers: { 'Cache-Control': 'no-cache' },
});
```

---

## Service Worker

???

精细化控制

前端自定义缓存 pwa 演示淘宝网

[神奇的 Workbox 3.0](https://zoumiaojiang.com/article/amazing-workbox-3/)

[设计一个无懈可击的浏览器缓存方案：关于思路，细节，ServiceWorker，以及 HTTP/2](https://zhuanlan.zhihu.com/p/28113197)

---

```javascript
console.log('Thanks!');
```
