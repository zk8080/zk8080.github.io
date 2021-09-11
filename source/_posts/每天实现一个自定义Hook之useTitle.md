---
title: 每天实现一个自定义Hook之useTitle
date: 2021-09-12 00:15:03
tags: 
    - React
    - React Hook
    - JavaScript
    - TypeScript
---
今天来实现一个设置页面标题的自定义Hook，逻辑简单。直接上代码：
<!-- more -->
```tsx
function useTitle(title: string, restore?: boolean) {
  // 原始的也页面标题
  const titleRef = useRef(document.title);
  
  useEffect(() => {
    // 将页面标题更新为传入的标题
    document.title = title;
  }, [title]);

  useEffect(() => {
    return () => {
      if(restore) {
        // 如果需要在页面卸载时恢复原始标题
        document.title = titleRef.current;
      }
    }
  }, []);
}

```