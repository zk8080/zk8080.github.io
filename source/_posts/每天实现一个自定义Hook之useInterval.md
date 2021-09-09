---
title: 每天实现一个自定义Hook之useInterval
date: 2021-09-09 23:34:41
tags: 
    - React
    - React Hook
    - JavaScript
    - TypeScript
---
今天来实现一个可以处理`setInterval`的自定义Hook。直接上代码：
<!-- more -->
```tsx
function useInterval(fn: () => void, delay: number | null | undefined, options?: { immediate?: boolean }): void {
  // 是否首次立即执行
  const immediate = options?.immediate;
  // 要执行的函数
  const fnRef = useRef<() => void>();
  // 每次更新 拿到最新的函数重新赋值
  fnRef.current = fn;

  useEffect(() => {
    // 如果没有传入间隔 则不执行定时器
    if (delay === undefined || delay === null) return;
    // 是否立即执行函数
    if (immediate) {
      fnRef.current?.();
    }
    // 创建定时器
    const timer = setInterval(() => {
      fnRef.current?.();
    }, delay);
    return () => {
      // 清除定时器
      clearInterval(timer);
    };
  }, [delay]);
}
```