---
layout: post
title: 浏览器缓存相关知识
date: 2019-08-02
Author: 轩
categories: cache
tags: [summary]
introduction: 总结一下浏览器缓存
coments: true
---
浏览器缓存，也就是客户端缓存。分为强缓存和协商缓存，强缓存命中时，不会发送请求，直接从本地加载缓存资源；先发送请求，协商缓存命中时，不从服务端加载资源，直接从本地加载缓存资源。

### 强缓存

强缓存时效性的判断标志位Expires和Cache-Control，其中Expires为GMT格式的绝对时间，如：Mon, 12 Aug 2019 02:31:27 GMT；Cache-Control：s-maxage=3600（覆盖max-age或Expires，仅适用于共享缓存）, max-age=3600（请求时间后的过期期限，单位为秒）, public（允许被任何对象缓存），no-cache（强制提交缓存验证）.......

#### 强缓存过程

1. 客户端第一次请求资源时，如果设置了强缓存，返回的response Header内添加Expires或者Cache-Control，指定缓存资源的过期时间，**返回code为200**。
2. 客户端接收到资源时，将含有Expires或者Cache-Control的response Header缓存到本地，下一次再请求相同的资源时，通过Expires或者Cache-Control对缓存资源的有效性进行判断，若还在有效期内，则直接从本地加载缓存资源，**返回code为200(disk cache or memory cache)**，返回的response Header为之前缓存的。
3. 若request Header设置了Cache-Control为no-cache或者max-age=0，则会强制发送请求验证缓存的有效性，**返回code为304（资源未更新）**，返回新的response Header。
4. 由于Expires返回的为一个绝对时间，因此若客户端与服务端的时间存在差别，或修改客户端本地的时间，会造成对缓存时效性的判断不准确。
5. Cache-Control：max-age的时间为距离请求时间的秒数，相对于Expires，可靠一些。
6. Expires与Cache-Control可单独设置一个，也可同时设置，同时设置时Cache-Control优先级高于Expires

![img](https://raw.githubusercontent.com/dyx2019/dyx2019.github.io/master/images/cache/expires.png)

#### 强缓存的应用与更新

强缓存一般用于长期不改变的静态资源，给其设置一个较长的有效期，提高加载效率。在此期间需要更新静态资源，改变请求的路径如加个版本号?v=001。

### 协商缓存

协商缓存时效性标志[Last-Modified, if-Modified-Since]或[Etag, If-None-Match]。其中[Last-Modified, if-Modified-Since]分别为服务端和客户端存储的GMT格式的资源最后修改时间。Last-Modified由服务端的response Header传递给客户端，客户端下次请求时加上if-Modified-Since（上次接收的Last-Modified的值）；[Etag, If-None-Match]分别为服务端和客户端存储的资源修改标识符。Etag由服务端生成的一个特殊字符串，response Header传递给客户端保存，客户端下次请求时加上If-None-Match（上次接收的Etag的值）。

#### 协商缓存过程

1. 客户端第一次请求资源时，如果设置了协商缓存，返回的response Header内添加Last-Modified或Etag，分别为资源的最后修改时间和服务端生成的资源唯一的标识符，**返回code为200**。
2. 客户端接收到资源后将其缓存，下一次再请求资源时，在request Header内加上if-Modified-Since（上次接收到的Last-Modified）或If-None-Match（上次接收到的Etag）。如果if-Modified-Since或Etag与服务端相比没有发生改变，则不返回任何资源，从本地加载缓存资源。
3. 如果if-Modified-Since或Etag与服务端相比发生改变，则返回新的资源，并更新Last-Modified或Etag
4. 由于存在服务端的文件改变，但最后修改时间未更新的情况存在，因此[Last-Modified, if-Modified-Since]不是一定可靠，而[Etag, If-None-Match]为服务端生成的每个资源的唯一标识符，只要资源发生改变，便会进行更新，因此更加可靠。

![img](https://raw.githubusercontent.com/dyx2019/dyx2019.github.io/master/images/cache/lastModified.png)

### 用户行为对浏览器缓存的影响

1. 输入url或者点击链接时，依次检查本地强缓存（有效则加载，否则发生请求），服务端资源有效性（有效则加载本地资源，无效则返回资源）。
2. 刷新，跳过检查本地强缓存，发送请求检查服务端资源有效性。
3. 强制刷新，跳过本地强缓存和远端服务端资源有效性检查，加载服务端最新资源。

### 总结一下

浏览器缓存分为强缓存和协商缓存，两者区别在于是否发送请求。强缓存标识符为Expires和Cache-Control，区别在于Expires为具体时间，会被改变，不一定准确；Cache-Control为请求后的秒数，更加准确。协商缓存标识符为[Last-Modified, if-Modified-Since]和[Etag, If-None-Match]，[Last-Modified, if-Modified-Since]为资源最后改变时间，可能被欺骗，不一定准确；[Etag, If-None-Match]为资源的唯一标识符，能准确标定资源的变化。