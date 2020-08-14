---
title: 移动端rem适配记录
date: 2020-08-14 15:34:19
tags: 
    - styled
    - rem
    - flexible
---
最近在项目中做了一个h5页面，发现适配方案有很多，最终选择了使用`flex`加`lib-flexible`和`rem`去进行适配，本文记录下在项目中是如何配置适配的。

## lib-flexible
说到移动端适配，手淘的`flexible`方案应该是使用的最多的。不过由于`viewport`单位兼容性越来越好，官方也建议大家使用`viewport`单位替代`flexible`方案。但是本文还是选择继续使用`flexible`，后面可能会使用`viewport`重构。

### 安装和使用

```
npm install lib-flexible --save
```
安装完成后，在项目的入口文件中引入该插件即可。
```js
import 'lib-flexible'
```
在html模板文件中，定义meta标签，该标签定义了用户通过手指放大缩小无效，页面比例始终为1:1。
```html
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no">
```

## px2rem-loader
引入了`lib-flexible`后，我们要做的就是把设计稿中的`px`转为`rem`了。当然，在现在这个前端工程化如火如荼的环境下，我们有很多自动化的转换工具。我在项目中使用了`px2rem-loader`这个插件去自动转换。

### 安装和使用

```
npm install px2rem-loader --save-dev
```
安装完成后，我们就需要在`webpack`打包配置中，配置一下`loader`了。我的项目中使用的是`less`文件处理器，所以增加以下配置：
```js
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [{
                    loader: 'style-loader'
                }, {
                    loader: 'css-loader'
                }, {
                    loader: 'px2rem-loader',
                    // options here
                    options: {
                        remUnit: 37.5,
                        remPrecision: 8
                    }
                }]
            },
            {
                test: /\.less$/,
                use: [{
                    loader: 'style-loader'
                }, {
                    loader: 'css-loader'
                }, {
                    loader: 'px2rem-loader',
                    // options here
                    options: {
                        remUnit: 37.5,
                        remPrecision: 8
                    }
                }, {
                    loader: 'less-loader',
                    // options here
                    options: {
                        javascriptEnabled: true
                    }
                }]
            }
        ]
    }
```
这里需要注意的是`options`中的`remUnit`这个配置，`lib-flexible`默认将页面分成100份，`1rem`等于10份，所以这个`remUnit`就是我们`1rem`的数据。我们可以根据设计稿的尺寸来设置这个配置。因为我的设计稿是`375px`的所以这里我配置的是`37.5`。

## styled-px2rem
可能大家很疑惑，上面的配置已经可以开始开发了，为什么还有一个`styled-px2rem`。这个主要是我们项目中没有统一化`css`预处理器的使用导致的，大家不要学习。但是为了恰饭，也要继续工作呀。所以要继续配置`styled`中`px`转`rem`的工作。
因为`styled`是`js`文件，所以`px2rem-loader`并不能解析我们的样式。但是我又不甘心自己去手动算`rem`，然后经过一顿搜索，找到了一个`styled-px2rem`的工具。具体使用方法，大家可以参考[github上的说明](https://github.com/win-winFE/styled-px2rem)。配置很简单，完成后就可以开开心心的开始开发了。

## 结语
关于移动端适配的方式有很多，本文只是对这次配置的一个记录，后续开发应该会选择`flex`加`vw`的方式，不需要这么多繁琐的配置。