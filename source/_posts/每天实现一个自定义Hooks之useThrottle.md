---
title: 每天实现一个自定义Hooks之useThrottle
date: 2021-08-31 23:53:00
tags: 
    - React
    - React Hook
    - JavaScript
---
昨天实现了防抖的自定义Hooks`useDebounce`，那么今天来实现一下节流的自定义Hooks`useThrottle`。关于节流，之前的文章中有提到过可以使用定时器和时间戳两种方式来实现，那么我们的hooks也可以使用两种方案来实现。

### 使用定时器实现
```tsx
type Nullable<T> = T | null;

interface CbRef {
  fn: (...args: any[]) => any;
  timer: Nullable<ReturnType<typeof setTimeout>>;
}

const useThrottle = <T extends (...args: any[]) => any>(fn: T, delay: number, dep = []) => {
  const ThrottleRef = useRef<CbRef>({ fn, timer: null });

  // 每次执行hooks 重新赋值函数，可以保证每次执行的函数都能拿到组件内最新的state
  ThrottleRef.current.fn = fn;

  return useCallback((...args: any[]) => {
    // 不存在定时器时才执行函数
    if (!ThrottleRef.current.timer) {
      ThrottleRef.current.timer = setTimeout(() => {
        // 每隔delay毫秒后 清空定时器
        ThrottleRef.current.timer = null;
      }, delay);
      // 执行节流函数
      ThrottleRef.current.fn.call(undefined, ...args);
    }
  }, dep);
}

```

### 使用时间戳
```tsx
const useThrottle = <T extends (...args: any[]) => any>(
  fn: T,
  wait: number,
  deps = []
) => {
  const throttleRef = useRef({ fn, prev: 0 });
  throttleRef.current.fn = fn;

  return useCallback(
    (...args: any[]) => {
      const now = +Date.now();
      // 判断当前时间与上次事件差是否大于延时，大于则执行函数
      if (now - throttleRef.current.prev >= wait) {
        throttleRef.current.fn.apply(undefined, args);
        // 缓存当前时间
        throttleRef.current.prev = now;
      }
    },
    deps
  ) as T;
};

```