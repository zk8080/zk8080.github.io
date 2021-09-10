---
title: 每天实现一个自定义Hook之useUpdateEffect
date: 2021-09-10 23:40:04
tags: 
    - React
    - React Hook
    - JavaScript
    - TypeScript
---
今天来实现一个只在依赖更新时执行的`useEffect`，逻辑简单。直接上代码：
<!-- more -->
```tsx
const useUpdateEffect = (effect, deps) => {
  // 判断组件是否已经挂载的ref
  const isMounted = useRef(false);

  useEffect(() => {
    
    if (!isMounted.current) {
      // 初次执行时
      isMounted.current = true;
    } else {
      // 组件已经挂载过
      return effect();
    }
  }, deps);
};
```