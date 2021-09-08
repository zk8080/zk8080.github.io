---
title: 每天实现一个自定义Hook之usePersistFn
date: 2021-09-08 23:36:57
tags: 
    - React
    - React Hook
    - JavaScript
    - TypeScript
---
今天来实现一个可以持久化`function`，保证函数引用地址不会改变的自定义Hook。直接上代码：
<!-- more -->

```tsx
export type noop = (...args: any[]) => any;

function usePersistFn<T extends noop>(fn: T) {
  // 存储传入函数的ref变量
  const fnRef = useRef<T>(fn);
  // 每次渲染fn的最新值都会记录在fnRef中
  fnRef.current = fn;
  
  // 返回给组件使用的函数
  const persistFn = useRef<T>();
  // 首次渲染时给persistFn赋值，后续渲染persistFn不会更新
  if (!persistFn.current) {
    // persistFn的引用地址一直在这个匿名函数中
    persistFn.current = function (...args) {
      return fnRef.current!.apply(this, args);
    } as T;
  }

  // 返回persistFn
  return persistFn.current!;
}
```