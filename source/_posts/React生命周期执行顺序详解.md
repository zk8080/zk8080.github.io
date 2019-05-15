---
title: React生命周期执行顺序详解
date: 2019-05-15 14:34:37
categories: 前端
tags:
    - React
    - JavaScript
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不管是使用React还是Vue，生命周期都是我们必须掌握的。最近在面试的时候，很多人搞不清楚React的生命周期的执行顺序，所以写一篇文章来解析一下React的生命周期函数的执行顺序。本文只介绍常用的生命周期函数的执行顺序，所以有部分不常用的执行顺序可以去看React官方文档。
我们先看一张官方文档中的生命周期图谱（v16.4）：
![React生命周期图谱](https://raw.githubusercontent.com/zk8080/blog-picture/master/img/WechatIMG16.png)
根据这个图，我们可以分三个阶段来解析React的生命周期执行顺序：
## 组件初始化创建(Mounting)阶段
此阶段的顺序为：constructor -> componentWillMount -> render -> ComponentDidMount。
由于React新的版本中将一些生命周期函数标记为‘过时’，所以在图谱中我们没有看到componentWillMount，但是现在依然可以使用这个生命周期函数。
我们可以看下面这行代码来检验这个阶段的执行顺序：
```
class Index extends Component {
    constructor(props) {
        super(props);
        console.log('constructor');
    }
    componentWillMount() {
        console.log('componentWillMount');
    }
    componentDidMount() {
        console.log('componentDidMount');
    }
    render() {
        console.log('render');
        return (
            <div>
                render
            </div>
        )
    }
}
```
打开chrome浏览器检验一下结果：
![初始化执行结果](https://raw.githubusercontent.com/zk8080/blog-picture/master/img/WechatIMG17.png)
这个顺序没有什么疑问，但是如果是嵌套组件呢？父子组件之前的顺序是什么样的呢？
### 嵌套组件初始化创建阶段
我们再看一段代码：
```
class Parent extends Component {
    constructor(props) {
        super(props);
        console.log('constructor---Parent');
    }
    componentWillMount() {
        console.log('componentWillMount---Parent');
    }
    componentDidMount() {
        console.log('componentDidMount---Parent');
    }
    render() {
        console.log('render---Parent');
        return (
            <div>
                <Child />
            </div>
        )
    }
}

class Child extends Component {
    constructor(props) {
        super(props)
        console.log('constructor---Child')
    }
    componentWillMount() {
        console.log('componentWillMount---Child')
    }
    componentDidMount() {
        console.log('componentDidMount---Child')
    }
    render() {
        console.log('render---Child')
        return (
            <div>
                Child
            </div>
        )
    }
}
```
这是一个最普通的嵌套组件，我们在chrome中检验一下结果：
![嵌套组件初始化结果](https://raw.githubusercontent.com/zk8080/blog-picture/master/img/WechatIMG18.png)
我们可以看到嵌套组件的执行顺序为：
+ 父组件：constructor -> componentWillMount -> render；
- 在父组件render的时候开始初始化创建子组件；
* 子组件：constructor -> componentWillMount -> render -> componentDidMount；
+ 最后子组件执行完毕后，再回到父组件执行componentDidMount。

## 组件更新(Updating)阶段
如果要触发组件更新有两种情况（不考虑Redux、Mobx等数据流管理工具）：
+ setState更新state后触发组件自身更新
- 父组件更新后触发子组件的更新

第一种情况是当前组件内部的更新，第二种情况是由于父组件更新导致子组件的更新。这两种情况都会触发更新，但是有部分生命周期函数的执行不一样，我们先看第一种情况：
### setState触发的组件自身更新
先看下面一段代码：
```
class Index extends Component {
    constructor(props) {
        super(props);
        console.log('constructor');
        this.state={
            count: 0
        }
    }
    componentWillMount() {
        console.log('componentWillMount');
    }
    componentDidMount() {
        console.log('componentDidMount');
        this.setState({
            count: 1
        })
    }
    componentWillUpdate(){
        console.log('componentWillUpdate')
    }
    componentDidUpdate(){
        console.log('componentDidUpdate');
    }
    render() {
        console.log('render');
        return (
            <div>
                Index
            </div>
        )
    }
}
```
接着在chrome中验证一下结果：
![setState更新的执行顺序](https://raw.githubusercontent.com/zk8080/blog-picture/master/img/WechatIMG21.png)
通过这个结果我们可以得到以下的结论：
更新state后的组件自身更新生命周期执行顺序：componentWillUpdate -> render -> componentDidUpdate

### 父组件更新触发的子组件更新
这种情况是在嵌套组件中，直接上代码：
```
class Parent extends Component {
    constructor(props) {
        super(props);
        console.log('constructor---Parent');
        this.state={
            count: 0
        }
    }
    componentWillMount() {
        console.log('componentWillMount---Parent');
    }
    componentDidMount() {
        console.log('componentDidMount---Parent');
        this.setState({
            count: 1
        })
    }
    componentWillUpdate(){
        console.log('componentWillUpdate---Parent')
    }
    componentWillReceiveProps(nextProps){
        console.log('componentWillReceiveProps---Parent')
    }
    componentDidUpdate(){
        console.log('componentDidUpdate---Parent');
    }
    render() {
        console.log('render---Parent');
        return (
            <div>
                <Child />
            </div>
        )
    }
}

class Child extends Component {
    constructor(props) {
        super(props)
        console.log('constructor---Child')
        this.state = {}
    }

    componentWillMount() {
        console.log('componentWillMount---Child')
    }

    componentDidMount() {
        console.log('componentDidMount---Child')
    }

    componentWillReceiveProps(nextProps){
        console.log('componentWillReceiveProps---Child')
    }

    componentWillUpdate(nextProps, nextState) {
        console.log('componentWillUpdate---Child')
    }

    componentDidUpdate(prevProps, prevState) {
        console.log('componentDidUpdate---Child')
    }
    render() {
        console.log('render---Child')
        return (
            <div>
                Child
            </div>
        )
    }
}
```
在chrome中验证一下结果：
![嵌套组件更新执行顺序](https://raw.githubusercontent.com/zk8080/blog-picture/master/img/WechatIMG22.png)
从这个结果中我们可以看到，只有子组件的componentWillReceiveProps方法执行了，我们可以得到以下结论：
+ 父组件执行更新：componentWillUpdate -> render
- 父组件render后开始更新子组件
* 子组件执行更新：componentWillReceiveProps -> componentWillUpdate -> render -> componentDidUpdate
+ 子组件更新完毕后，父组件执行componentDidUpdate，结束更新。

## 组件卸载(Unmounting)阶段
在这个阶段，组件只会执行一个生命周期函数：componentWillUnmount，在这个函数中，我们可以做一些初始化数据的操作。防止组件下次渲染的时候保留了上次的数据。
同上面两个阶段，这个阶段，也会出现在嵌套组件中，但是却和之前的执行顺序不太一样：
+ 父组件执行componentWillUnmount
- 子组件执行componentWillUnmount

## 结语
通过上面的解析，我们可以清晰的知道组件的生命周期执行顺序了：
+ 初始化创建组件阶段：
    父组件：constructor -> componentWillMount -> render -> 
    子组件：constructor -> componentWillMount -> render -> componentDidMount ->
    父组件：componentDidMount
- 更新阶段：
    父组件：componentWillUpdate -> render -> 
    子组件：componentWillReceiveProps -> componentWillUpdate -> render -> componentDidUpdate ->
    父组件：componentDidUpdate
* 卸载阶段：
    父组件：componentWillUnmount
    子组件：componentWillUnmount

通过这些代码的检验，得到了以上的结论，当然这些生命周期函数只是常用的，还有一些不常用的大家可以参考一下官网文档。其中有一个shouldComponentUpdate函数是React控制组件更新进行优化的一个生命周期函数，大家可以了解一下。