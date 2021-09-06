---
title: 每天实现一个自定义Hook之useClickAway
date: 2021-09-06 21:48:06
tags: 
    - React
    - React Hook
    - JavaScript
---
今天来实现一个判断是否在目标元素外点击的自定义Hook，逻辑比较简单。直接上代码：
<!-- more -->
```tsx
function useClickAway(ref: RefObject<HTMLElement>, handler: Function) {
  useEffect(() => {
    // 监听事件执行的回调
    const listener = (event: MouseEvent) => {
      // 如果不存在目标元素，或者当前点击的元素在目标元素内，则不处理传入的handler
      if (!ref.current || ref.current.contains(event.target as HTMLElement)) {
        return;
      }
      // 执行传入的handler
      handler(event);
    };
    // 注册监听事件
    document.addEventListener('click', listener);
    return () => {
      // 卸载监听事件
      document.removeEventListener('click', listener);
    };
  }, [ref, handler]);
}
```