---
title: 每天实现一个自定义Hook之usePrevious
date: 2021-09-04 22:34:17
tags: 
    - React
    - React Hook
    - JavaScript
---
今天来实现一个可以获取组件更新前一轮的`props`和`state`的自定义Hook，这个Hook在React文档中有描述，我在之前的文章中也有单独解读，这里为了记录每天的自定义Hook，就重新实现一次。直接上代码：
<!-- more -->
```tsx
function usePrevious<T>(value: T): T {
  // 使用ref存储要获取的props或者state
  const ref: any = useRef<T>();
  // 在useEffect中重新设置ref的值
  useEffect(() => {
    // 因为useEffect是在Render后才执行的，所以会先返回ref.current 再执行useEffect把新的value更新到ref上，
    // 下次更新时获取的是上一轮更新的值
    ref.current = value;
  }, [value]); 
  // 返回上一轮的值
  return ref.current;
}
```