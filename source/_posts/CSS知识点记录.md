---
title: CSS知识点记录
date: 2021-08-06 13:51:07
tags: 
    - CSS
---

最近在开发中，发现一些css样式自己处理的不是很好，但是在网上搜索后都找到了解决方案，在此文章里记录下。

## 父元素设置min-height，子元素设置高度百分比继承无效
**表现**：对父元素使用`min-height`为100px，在子元素不脱离文档流的情况下，对子元素设置为100%，但实际显示的时候，子元素的`height`是0。
**原因**：子元素设置高度百分比时，是相对于父元素的高度进行继承计算的。当父元素只有`min-height`，但是没有`height`属性时，则无法获取活继承父元素高度，所以子元素的`height`为0。这是`webkit`的bug，有最小高度的父盒子里面的子元素不能继承高度。详见[该链接](https://bugs.webkit.org/show_bug.cgi?id=26559)。

**解决方案**
1. 在当前父元素外层再包一层div，并且设置为flex弹性布局。
```html
<div class='wrapper'>
  <div class='parent'>
    <div class='child'></div>
  </div>
</div>
```
```css
.wrapper {
    display: flex;
}
.parent {
    width: 100px;
    min-height: 100px;
}
.child {
    width: 100%;
    height: 100%;
}
```
该方案利用的是弹性盒子会默认拉伸高度。

2. 给父元素加绝对定位，然后子元素加相对定位，这样子元素的高度就会根据父元素的高度进行计算。
```html
<div class='parent'>
  <div class='child'></div>
</div>
```
```css
.parent {
    width: 100px;
    min-height: 100px;
    position: relative;
}
.child {
    width: 100%;
    height: 100%;
    position: absolute;
}
```

## flex布局和margin的结合使用
**表现**：在页面布局中，经常会出现3列子项，前两项左对齐，最后一项右对齐。往常我都会将前两项放入一个元素中，来转换成两列布局，然后使用`justify-content: space-between`来完成布局，但是这样并不合理。
**解决方案**
```
<div class='parent'>
  <div class='box1'></div>
  <div class='box2'></div>
  <div class='box3'></div>
</div>
```
```css
.parent {
    display: flex;
}
.box3 {
    margin-left: auto;
}
```
只要将最后一列的`margin-left`设置为`auto`，则该子项会被挤到右对齐的位置，前两项依然为左对齐。

### flex布局中flex-wrap换行逻辑
**表现**：当父元素设置为`flex`时，且使用了`flex-wrap: wrap`，如果子元素某一子项文本内容过长，则会将该子项整体自动换行。
**解决方案**
该问题记录的原因是因为在开发中有使用`contenteditable`属性做评论功能，动态渲染其子元素，出现文本换行，查询后发现自己在外层父元素中使用了`flex-wrap: wrap`，去除该属性后便无问题。查询了`flex`文档后才发现，自己之前对`flex`的了解不够深入，特此记录。
