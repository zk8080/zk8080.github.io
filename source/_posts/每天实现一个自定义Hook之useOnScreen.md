---
title: 每天实现一个自定义Hook之useOnScreen
date: 2021-09-12 23:22:50
tags: 
    - React
    - React Hook
    - JavaScript
    - TypeScript
---
今天来实现一个可以检测元素何时可见的自定义Hook，该Hook实现逻辑依赖了`IntersectionObserver`API，对其不了解的可以去[MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Intersection_Observer_API)上查看详细介绍，主要是提供了一种异步检测目标元素与祖先元素或 viewport相交情况变化的方法，后续打算整理一篇详细的使用情况。这里先实现我们的自定义Hook，直接上代码：
<!-- more -->
```tsx
function useOnScreen<T extends Element>(ref: MutableRefObject<T>, rootMargin: string = "0px"): boolean {
  // 目标元素是否可见的状态state
  const [isIntersecting, setIntersecting] = useState<boolean>(false);
  useEffect(() => {
    // 创建一个 IntersectionObserver 对象
    const observer = new IntersectionObserver(
      ([entry]) => {
        // entry代表当前元素IntersectionObserverEntry对象，isIntersecting属性代表其是否在当前窗口是否显示
        setIntersecting(entry.isIntersecting);
      },
      {
        // 配置root元素的外边距，即元素距离窗口多少距离时开始触发回调函数
        rootMargin,
      }
    );
    if (ref.current) {
      // 开始监听目标元素
      observer.observe(ref.current);
    }
    return () => {
      // 停止监听目标元素
      observer.unobserve(ref.current);
    };
  }, []); 

  // 返回目标元素可见状态
  return isIntersecting;
}
```