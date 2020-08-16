---
layout: post
title: webview拦截APP返回
date: 2020-08-16
Author: 轩
categories: cache
tags: [summary]
introduction: webview中拦截app的返回操作
coments: true
---
在APP的开发中，通常可以分为原生开发、web App、Hybird、flutter等方式。原生开发的app其可以访问手机的所有功能，运行速度快，能够提供给用户绝佳的体验，但其开发成本高、周期长的，需要同时进行Android和ios的开发。web App是利用H5的方式进行开发，可以大大缩短开发周期，但是不能利用手机的相关功能，功能有限，用户体验差。Hybird（混合开发），在原生开发中嵌入H5页面的方式开发，开发效率相比原生开发大大提高，可通过app提供的sdk使用手机的部分功能，用户体验尚可。

### 问题

在利用Hybird方式开发时，当想对H5的某些页面进行用户挽留时，需要在H5的页面中拦截App的返回操作，并进行相关操作。在App中没有返回相关sdk并且团队也没有对history进行进一步封装时，如何拦截物理返回呢？

### 解决方案

H5的开发过程中，是利用vue进行开发的，因此之后的内容以vue为例。
***
解决方案如下：
```
  mounted() {
    // 在进入页面时，通过popState添加一条history记录
    window.history.pushState({ page: 1}, null, 'http://geogle/home');
    // 监听popstate事件，popstate在返回和前进时触发
    window.addEventListener('popstate', this.preventBack, false);
  },
  
  destroyed() {
    // 在页面销毁时，移除监听事件，不影响之后的页面
    window.removeEventListener('popstate', this.preventBack, false);
  },

  methods: {
    preventBack() {
      // 监听到popstate的触发事件，
      // 在此进行拦截操作
    }
  }
```

#### pushState

将window.history打印出来，可以发现history包含三个属性：length、scrollRestoration、state。
* length: 当前窗口访问过的网址数量，包含当前网页
* scrollRestoration: 滚动恢复属性，设置在历史导航中的滚动恢复行为，默认值为auto
* state: history堆栈中最上层的状态值
* back、go等方法则位于__proto__中

![img](https://raw.githubusercontent.com/dyx2019/dyx2019.github.io/master/images/history.png)

> history.pushState(state, title[, url])

pushState用于向history中添加一条历史记录，但不会立即刷新当前页面，即只改变当前网页的url，页面不发生改变。
例如在跳转到http://geogle/about页面时，执行如下代码，则url会更改为http://geogle/home，页面不会进行刷新。
```
mounted() {
    // 在进入页面时，通过popState添加一条history记录
    window.history.pushState({ page: 1}, null, 'http://geogle/home');
  },
```
pushState接收三个参数state、title、url

state为一个自定义的对象，其中可以包含任何值如：{title: '', key: ''}
> state对象是一个JavaScript对象，它与pushState（）创建的新历史记录条目相关联。 每当用户导航到新状态时，都会触发{{event（“ popstate”）}}事件，并且该事件的状态属性包含历史记录条目的状态对象的副本。

title设置当前页面的title，并非被所以浏览器支持
> 当前大多数浏览器都忽略此参数，尽管将来可能会使用它。 在此处传递空字符串应该可以防止将来对方法的更改。 或者，您可以为要移动的州传递简短的标题。

url当前页面显示的url，history堆栈中最上层的url，即即将返回的url
> 新历史记录条目的URL由此参数指定。 请注意，浏览器不会在调用pushState（）之后尝试加载此URL，但可能会稍后尝试加载URL，例如在用户重新启动浏览器之后。 新的URL不必是绝对的。 如果是相对的，则相对于当前URL进行解析。 新网址必须与当前网址相同{{glossary（“ origin”）}}}； 否则，pushState（）将引发异常。 如果未指定此参数，则将其设置为文档的当前URL。

#### popstate

```
// 添加popstate事件监听事件
window.addEventListener('popstate', this.preventBack, false);
```

popstate事件在浏览器的历史记录发生改变时会被触发，但history.pushState并不会触发该事件，通常在点击返回按钮或显示调用history.back()、history.go(-1)事件时触发。
***  
**vue中，在未添加pushState时，点击返回事件时会立即执行销毁当前页面的操作，即会触发destroyed生命周期函数，将popstate事件的监听事件移除，因而并不能拦截到app的返回行为，这也是为什么需要pushState添加一条历史记录。popstate事件的触发需要完整的执行完history.back()事件时才会触发**

例如在跳转到http://geogle/about页面时，执行如下代码，则url会更改为http://geogle/home，页面不会进行刷新。
```
mounted() {
    // 在进入页面时，通过popState添加一条history记录
    window.history.pushState({ page: 1}, null, 'http://geogle/home');
    window.addEventListener('popstate', this.preventBack, false);
  },
```
**pushState添加一条历史记录http://geogle/home，点击返回按钮，当前url恢复为http://geogle/about，且当前页面不会重新渲染及销毁，因而触发popstate监听事件，执行拦截相关操作。**

