---
title: 每天实现一个自定义Hooks之useDebounce
date: 2021-08-30 23:35:43
tags: 
    - React
    - React Hook
    - JavaScript
---

很久没有更新博客了，突发一个想法，每天实现一个简单实用的自定义Hooks，来加深自己对React Hooks的理解。今天实现一个简单的防抖Hooks。本文只实现了最简单的防抖原理，如果想实现一个完整可用的，可以参考`ahooks`的[useDebounce](https://github.com/alibaba/hooks/blob/master/packages/hooks/src/useDebounce/index.ts)。直接上代码：
<!-- more -->

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

上面代码只适用于基础的防抖情景，例如input输入框`onChange`事件等，下面实现一个支持传入函数的防抖Hooks。
```tsx
interface CbRef {
  fn: (...args: any[]) => any;
  timer: ReturnType<typeof setTimeout> | null;
}

const useDebounce = <T extends (...args: any[]) => any>(fn: T, wait: number, deps = []) => {
  const DebounceRef = useRef<CbRef>({ fn, timer: null });

  // 每次执行hooks 重新赋值函数，可以保证每次执行的函数都能拿到组件内最新的state
  DebounceRef.current.fn = fn;

  return useCallback(
    (...args: any[]) => {
      // 每次执行先判断是否存在上次的定时器，如果有则先清除定时器
      if (DebounceRef.current.timer) {
        clearTimeout(DebounceRef.current.timer);
      }
      // 超过wait秒后，执行setTimeout
      DebounceRef.current.timer = setTimeout(() => {
        DebounceRef.current.fn.apply(undefined, args);
        DebounceRef.current.timer = null;
      }, wait);
    },
    deps
  ) as T;
}
```