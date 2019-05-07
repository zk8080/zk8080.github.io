---
title: setState同步异步问题
date: 2019-05-04 17:15:40
categories: 前端
tags:
    - React
    - JavaScript
---
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在React开发中，setState一般都会被认为是异步的。但是最近在掘金上看到一篇文章 [你真的理解setState吗？](https://juejin.im/post/5b45c57c51882519790c7441) ，看完后发现setState并不是想象中的异步那么简单。

# setState是异步的吗？
在React官方文档中，有这样一段话：
>setState()不总是立刻更新组件。其可能是成批处理或推迟更新。这使得在调用setState()后立刻读取this.state变成一个潜在陷阱。代替地，使用componentDidUpdate或一个setState回调（setState(updater, callback)），当中的每个方法都会保证在更新被应用之后触发。

读完这段话，我们本能的认为setState就是异步的，因为连官方都推荐我们在componentDidUpdate生命周期和setState的callback中去读取this.state。
接下来我们用代码去检验一下setState的同步异步：
## 生命周期中的setState
    ```
        class Index extends Component {
            constructor(props) {
                super(props);
                this.state={
                    count: 0
                }
            }

            componentDidMount() {
                this.setState({
                    count: 1
                })
                console.log( this.state.count )
            }

            componentDidUpdate(){
                console.log(this.state.count)
            }
            
            render() {
                return (
                    <div>
                        测试
                    </div>
                )
            }
        }
    ```
在上面这个例子中，我们在componentDidMount中使用setState去更新了count，然后立马打印了一下count；接着在componentDidUpdate中我们也打印了一下count,然后我们在chrome中查看一下控制台，输出为：0 1。这个结果我们并不意外，在开发中这种代码是我们经常遇到，这也说明在生命周期中setState是异步的。

## 合成事件中的setState
```
class Index extends Component {
    constructor(props) {
        super(props);
        this.state={
            count: 0
        }
    }

    onClick = () => {
        this.setState({
            count: 1
        })
        console.log(this.state.count)
    }

    render() {
        return (
            <div>
                <button
                    onClick={this.onClick}
                >
                    测试
                </button>
            </div>
        )
    }
}

```
看一下这个例子中，我们在onClick事件中使用setState去更新count，接着立马打印了count；我们到chrome中去验证一下，发现打印的还是0，这说明在合成事件中setState也是异步的。

## setTimeout中的setState
```
class Index extends Component {
    constructor(props) {
        super(props);
        this.state={
            count: 0
        }
    }

    componentDidMount() {
        setTimeout(() => {
            this.setState({
                count: 1
            })
            console.log(this.state.count)
        }, 0)
    }

    render() {
        return (
            <div>
                测试
            </div>
        )
    }
}
```
在这个例子中，我们在componentDidMount中定义了一个setTimeout，并且在回调函数中使用setState更新count，然后打印了count；我们在chrome中验证一下，发现控制台中输出的是：1。这个和前面的结果不同，count被实时读取到了，也说明在setTimeout中setState是同步的。

## 原生事件中的setState
```
class Index extends Component {
    constructor(props) {
        super(props);
        this.state={
            count: 0
        }
    }

    componentDidMount() {
        document.querySelector('button').addEventListener('click', this.onClick, false)
    }

    onClick = () => {
        this.setState({
            count: 1
        })
        console.log(this.state.count)
    }

    render() {
        return (
            <div>
                <button>
                    测试
                </button>
            </div>
        )
    }
}
```
在上面的例子中，我们给button按钮使用原生`addEventListener`的方式，绑定了click事件，并在click事件中使用setState去更新count，然后立马打印了count，接下来我们在chrome中验证一下，发现控制台中输出的是：1。这个结果和setTimeout中setState的结果一致，说明在原生事件中setState也是同步的。

## 结论
通过上面的例子，我们可以得到一下结论：
+ 在生命周期函数中，setState是异步的；
- 在合成事件中，setState是异步的；
* 在异步操作中，setState是同步的；
+ 在原生事件中，setState是同步的。

目前得到了这些结论，但是我们并不清楚其中的原理是什么，我查阅了一些资料，发现自己还是不能理解其中奥妙，所以就不在此误导大家，但是自己会努力学习理解这个问题。不过大家可以参考下面文章去发现更深层次的原理：
> [你真的理解setState吗？](https://juejin.im/post/5b45c57c51882519790c7441)