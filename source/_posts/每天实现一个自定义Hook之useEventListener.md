---
title: 每天实现一个自定义Hook之useEventListener
date: 2021-09-07 23:24:53
tags: 
    - React
    - React Hook
    - JavaScript
    - TypeScript
---
今天实现一个封装`addEventListener`绑定事件的自定义Hook，逻辑简单。直接上代码：
<!-- more -->

```tsx

type TargetElement = HTMLElement | Element | Document | Window;

type Options = {
  target?: TargetElement || MutableRefObject;
  capture?: boolean;
  once?: boolean;
  passive?: boolean;
};

function useEventListener(eventName: string, handler: Function, options: Options = {}) {
  // 存储事件回调的ref
  const handlerRef = useRef<Function>();
  // 每次渲染更新ref回调
  handlerRef.current = handler;

  useEffect(() => {
    // 判断是否存在目标元素
    const targetElement = options.target || window;
    if (!targetElement.addEventListener) {
      return;
    }
    // 事件回调
    const eventListener = (
      event: Event,
    ): EventListenerOrEventListenerObject | AddEventListenerOptions => {
      return handlerRef.current && handlerRef.current(event);
    };
    // 绑定事件
    targetElement.addEventListener(eventName, eventListener, {
      capture: options.capture,
      once: options.once,
      passive: options.passive,
    });

    return () => {
      // 销毁事件
      targetElement.removeEventListener(eventName, eventListener, {
        capture: options.capture,
      });
    };
    // 依赖
  }, [eventName, options.target, options.capture, options.once, options.passive]);
}
```