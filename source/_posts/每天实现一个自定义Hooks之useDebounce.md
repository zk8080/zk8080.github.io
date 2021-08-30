---
title: 每天实现一个自定义Hooks之useDebounce
date: 2021-08-30 23:35:43
tags: 
    - React
    - React Hook
    - JavaScript
---

很久没有更新博客了，突发一个想法，每天实现一个简单实用的自定义Hooks，来加深自己对React Hooks的理解。今天实现一个简单的防抖Hooks。
直接上代码，本文只实现了最简单的防抖原理，如果想实现一个完整可用的，可以参考`ahooks`的[useDebounce](https://github.com/alibaba/hooks/blob/master/packages/hooks/src/useDebounce/index.ts)

```tsx
const useDebounce = (value: any, delay = 300) => {
  // 要防抖的值
  const [debounceVal, setDebounceVal] = useState<any>(value);

  useEffect(() => {
    // 创建一个setTimeout，在delay(300)毫秒后设置debounceVal
    const handler = setTimeout(() => {
      setDebounceVal(value);
    }, delay);

    // 在delay(300)毫秒内value发生更新，则清空定时器，然后重新创建一个setTimeout
    // 直到delay(300)毫秒内value未发生更新，则执行handler，设置debounceVal的值
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debounceVal;
};

```