---
title: 浅析React性能优化
date: 2020-05-09 17:29:04
tags:
    - React
    - React Hook
    - JavaScript
---
使用React开发也有两年了，今天来总结一下怎么去优化我们的React应用。

## 减少render的次数
因为React的渲染机制，也就是Reconciliation的过程，每次render函数的执行，React会对比本次render的虚拟Dom和上次render的虚拟Dom，并且对有差异的地方进行更新，最后渲染为真实Dom。虽然React有自己优化后的Diff算法加持，但是只要对比就会出现性能消耗。所以我们首要的优化就是减少render的次数。

### 1.使用纯组件/纯函数
React中提供了`PurComponent`和`memo`两个API，这两个API的功能类似，都是对props进行浅比较(`PurComponent`还会对比state)，如果props浅比较相等，则跳过渲染，复用上一次的渲染结果。所以这两个API可以达到我们减少render次数的目的。`memo`方法有第二个参数可以传入一个自定义对比函数，可以用来自己控制对比过程，具体使用方法可以参考官方文档。这两个API的使用方法如下：
PurComponent：
```js
class Children extends React.PureComponent {
    render() {
        console.log('子节点渲染！')
        return (
            <div>
                子节点
            </div>
        )
    }
}

class Parent  extends Component {
    state = {
        test: '123',
        testObj: {a: 123}
    }
    testClick = () => {
        this.setState({
            test: '456'
        })
    }
    render() {
        return (
            <>
                <p>{this.state.test}</p>
                {/* 点击按钮并不会触发Children组件的重新渲染 */}
                <button onClick={this.testClick}>点击</button>
                <Children testObj={this.state.testObj}></Children>
            </>
        );
    }
}
```
React.memo：
```js
const Children = React.memo(({testObj}) => {
    return (
        <div>子节点</div>
    )
}) 

function Parent() {
    const [ test, setTest ] = useState('123')
    const [ testObj, setTestObj ] = useState({});

    const testClick = () => {
        setTest('456')
    }
    return (
        <>
            <p>{test}</p>
            {/* 点击按钮并不会触发Children组件的重新渲染 */}
            <button onClick={testClick}>点击</button>
            <Children testObj={testObj}></Children>
        </>
    )
}
```

### 2.使用shouldComponentUpdate生命周期函数
该函数是用来决定是否重新组件的，也就是是否进行render。该函数接收nextProps和nextState为参数，可以用来对比this.props和this.state来确定是否需要重新更新组件。需要注意的是该函数的返回值和`React.memo`方法的第二个参数`areEqual`函数的返回值相反，如果`shouldComponentUpdate`返回true则重新渲染组件，否则，不渲染。而`areEqual`函数返回true则代表props对比相等，不进行渲染，否则，重新渲染。使用如下：
```js
shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
}
```

### 3.使用正确的事件处理器
我们经常会在父组件中传递事件给子组件，但是不同的传递方式会导致不同的渲染过程。因为函数也属于引用类型，所以如果使用不当会导致每次都是新的引用，就导致不必要的渲染diff过程。这里列出类组件和函数组件传递事件比较好的方式：
类组件：
```js
// 类组件中使用Public Class Fields 箭头函数直接作为公共类方法
class Parent  extends Component {
    handleClick = () => {
        /*...*/
    }
    render() {
        return (
            <Children onClick={this.handleClick}></Children>
        );
    }
}

// 类组件中在构造函数中手动bind一次，那么每次都是使用都是同一个函数
class Parent  extends Component {
    constructor(props){
        super(props)
        this.handleClick = this.handleClick.bind(this)
    }
    handleClick() {
        /*...*/
    }
    render() {
        return (
            <Children onClick={this.handleClick}></Children>
        );
    }
}
```  
函数式组件：
```js
// 利用hooks API useCallback缓存函数 只要deps没有改变 这样每次组件执行的都是同一个函数引用
function Parent() {
    const handleClick = useCallback(() => {
        /*...*/
    }, [deps])
    return (
        <Children onClick={handleClick}></Children>
    )
}
```

### 4.使用不可变数据
使用第三方工具库[immutable.js](https://github.com/immutable-js/immutable-js)、[immer.js](https://github.com/immerjs/immer)，使状态变得可预测，而且配合`shouldComponentUpdate`、`PurComponent`、`React.memo`使用，会让浅比对变得更精确和高效。具体使用方法，大家感兴趣的话可以参考这两个库的文档。

## 减少渲染的计算量
上面减少render的次数，是为了diff的过程，但是如果组件达到的渲染的条件，那么优化的点就是减少diff的性能消耗了。

### 1.列表组件增加唯一标识key
我们知道React对比虚拟Dom的子节点时，使用递归同时遍历两个子节点列表，当产生差异时就会生成一个新的节点。如果我们对原有子节点列表尾部增加元素，那么之前的节点就会被复用。但是如果在头部增加节点，那么diff时React发现每个节点都不相同就会重新创建所有节点，导致不必要的性能消耗。
官方推荐我们给组件增加一个`key`属性，React会使用`key`的值来比对，判断节点是否改变，如果发现有已存在的`key`值，就会复用该节点。例如我们在头部新增节点，通过`key`值可以发现原有节点都可以复用，只是移动位置，那就比重新创建节点的性能好很多。
需要注意的是`key`值的选取原则：***不需要全局唯一，但必须列表中保持唯一***。另外使用数组元素的下标作为`key`值时，如果元素顺序不改变的情况下是没有问题的，但是如果元素顺序改变的话，会导致当前元素`key`值改变，也就会使diff效率下降，达不到我们优化的目的。
这个小结的代码大家可以参考官方文档[协调](https://react.docschina.org/docs/reconciliation.html)，在这就不贴代码了。

### 2.减少嵌套层级
使用`React.Fragment`代替不必要的父节点标签。在React组件中，我们render中必须具有一个公共父标签，而有时我们并不需要这个父标签，但是又不得不为了满足这个条件增加一个`div`标签。例如：
```js
function Parent() {
    const [ test, setTest ] = useState('123')
    const testClick = () => {
        setTest('456')
    }
    return (
        <div>
            <p>{test}</p>
            <button onClick={testClick}>点击</button>
        </div>
    )
}
```
可以看到这个外层`div`标签其实并没有任何意义，但是我们必须要额外提供一个父级标签，因为多了一个层级也会导致额外渲染元素的性能消耗。我们可以把代码改成如下方式：
```js
function Parent() {
    const [ test, setTest ] = useState('123')
    const testClick = () => {
        setTest('456')
    }
    return (
        <>
            <p>{test}</p>
            <button onClick={testClick}>点击</button>
        </>
    )
}
```
这样就节省了渲染不必要的元素，`<></>`是`<React.Fragment>`的短语法，但是该语法不支持`key`属性，所以如果有需要还是使用`<React.Fragment>`语法。

### 3.使用虚拟列表
虚拟列表可以对我们的长列表组件或者复杂组件树进行优化，减少渲染的节点，只渲染显示在当前视图的节点。可以应用在以下组件场景中：
* 无限滚动列表，表格，下拉列表，spreadsheets
- 无限切换的日历或轮播图
+ 大数据量或无限嵌套的树
* 聊天窗，数据流(feed)， 时间轴等

不过由于该方式我并没有使用过，大家可以参考对应组件方案的文档：
* [react-virtualized](https://github.com/bvaughn/react-virtualized)
- [react-window](https://github.com/bvaughn/react-window)

### 4.组件懒加载
使用`React.lazy`和`Suspense`组件，可以让我们实现组件懒加载的效果，从而减少渲染的节点。例如我们的`Tab`组件，并不需要一次性把所有的子组件都渲染出来，而是当前子节点被激活时再渲染，这样也会提升我们的渲染性能。懒加载的使用例子：
```js
import React, { Suspense } from 'react';

const OtherComponent = React.lazy(() => import('./OtherComponent'));

function MyComponent() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```
当然`React.lazy`配合`import()`使用，然后利用`webpack`打包的时候，也可以达到代码分割的效果。
