---
title: 实现一个获取上一轮props和state的自定义Hook
date: 2020-04-17 17:29:40
tags:
    - React
    - React Hooks
---
最近在项目中使用了React Hooks，刚上手感觉很好用，不用在写class组件中那么多的生命周期模板语法，尤其是一些需要在componentDidMount和componentDidUpdate中进行的副作用操作，可以直接使用useEffect去替代。写完感觉代码清爽很多，也更乐意去使用React Hooks了。当然，在不熟练的和不去深入了解的情况下会出现很多意想不到的问题。本文是我在将Class组件转换成Hooks的过程中，发现需要在组件中获取上一轮的props和state，但是内置的Hooks Api中并没有提供这个方法，在文档中找到一个可以自定义的Hooks，在此记录下。直接上代码：
```
const usePrevious = (preValue) => {
    const ref = useRef();
    useEffect(() => {
        ref.current = preValue;
    }, [preValue])
    
    return ref.current;
}
```
如果不太能理解这段代码为什么可以取到上一次的值，那么可以把这段代码放在我们的业务代码中去看一下：
```
function Counter() {
    const [count, setCount] = useState(0);

    const prevCountRef = useRef();
    useEffect(() => {
        prevCountRef.current = count;
    });
    const prevCount = prevCountRef.current;

    return <h1>Now: {count}, before: {prevCount}</h1>;
}
```
另外还要记住两点：
* useRef会保持引用不变，ref.current的值改变并不会引起组件重新渲染
- React Hooks中函数式组件的生命周期中，useEffect是在jsx渲染之后执行的

只要记住这两点，再结合上面这段代码，你就会知道为什么我们可以获取到上一次的props和state了。

## 参考文章
> [React官方文档](https://react.docschina.org/docs/hooks-faq.html#how-to-get-the-previous-props-or-state)
> [useRef为什么可以用来封装成usePrevious?(知乎)](https://www.zhihu.com/question/346140951)
> [一篇文章，带你学会useRef、useCallback、useMemo(知乎)](https://zhuanlan.zhihu.com/p/117577458)