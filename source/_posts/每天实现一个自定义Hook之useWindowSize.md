---
title: 每天实现一个自定义Hook之useWindowSize
date: 2021-09-13 23:26:01
tags: 
    - React
    - React Hook
    - JavaScript
    - TypeScript
---
今天来实现一个获取当前window窗口的宽高的自定义Hook。逻辑简单，直接上代码：
<!-- more -->
```tsx
interface Size {
  width: number;
  height: number;
}

function useWindowSize(): Size {

  const [windowSize, setWindowSize] = useState<Size>({
    width: window.clientWidth,
    height: window.clientHeight,
  });
  useEffect(() => {
    // resize的回调函数
    function handleResize() {
      // 设置窗口大小
      setWindowSize({
        width: window.clientWidth,
        height: window.clientHeight,
      });
    }
    // 监听resize
    window.addEventListener("resize", handleResize);
    // 取消监听
    return () => window.removeEventListener("resize", handleResize);
  }, []);
  return windowSize;
}
```