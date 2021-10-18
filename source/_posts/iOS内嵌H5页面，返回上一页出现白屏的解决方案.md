---
title: iOS内嵌H5页面，返回上一页出现白屏的解决方案
date: 2021-10-18 14:49:43
tags: 
    - H5
---
## 问题场景描述
在iOS内嵌的H5页面中，从带滚动的A页面底部点击->跳转B页面后返回->A页面出现白屏，触摸或滚动后页面恢复。

## 解决方案

### 修改history.scrollRestoration属性
使用`history.back`返回上一页的时候，浏览器会记录页面的滚动位置，而在iOS上面，滚动后返回的时候页面渲染会出现问题，导致白屏。可以修改`history.scrollRestoration`属性，它默认是`auto`，也就是会记录滚动位置；将其设置为`manual`，则不会再记录滚动位置。
```js
  // History.scrollRestoration
  
  // 定义：允许web应用程序在历史导航上显式地设置默认滚动恢复行为
  
  // 参数值：
  // 1. auto 将恢复用户已滚动到的页面上的位置
  // 2. manual 未还原页上的位置。用户必须手动滚动到该位置
  
  if ('scrollRestoration' in window.history) {
    window.history.scrollRestoration = 'manual';
  }
```
### 触发滚动
在返回A页面并且加载数据完成后，使用js方法自动触发滚动。
```js
  // 加载数据代码...
  window.scrollTo(0, 1); 
  window.scrollTo(0, 0);
```