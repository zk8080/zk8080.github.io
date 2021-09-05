---
title: 每天实现一个自定义Hook之useLockFn
date: 2021-09-05 22:46:36
tags: 
    - React
    - React Hook
    - JavaScript
---
今天来实现一个可以对异步函数增加竞态锁的自定义Hook，这个在平时开发中也有很多场景可以使用，例如表单提交，当然也可以使用防抖、节流、Loading等方案。直接上代码：
```tsx
function useLockFn<P extends any[] = any[], V extends any = any>(fn: (...args: P) => Promise<V>) {
  // 用于判断异步函数是否执行完成的标识
  const lockRef = useRef(false);

  // 使用useCallback来优化函数定义，防止不必要的渲染
  return useCallback(
    async (...args: P) => {
      // 如果异步函数在执行中，则直接return 相当于Promise.resolve(undefined)
      if (lockRef.current) return;
      // 加锁
      lockRef.current = true;
      try {
        // 执行异步函数
        const ret = await fn(...args);
        // 解锁
        lockRef.current = false;
        return ret;
      } catch (e) {
        // 出现异常解锁
        lockRef.current = false;
        throw e;
      }
    },
    [fn],
  );
}

```
