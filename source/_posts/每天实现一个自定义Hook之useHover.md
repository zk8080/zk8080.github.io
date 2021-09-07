---
title: 每天实现一个自定义Hook之useHover
date: 2021-09-01 23:05:30
tags: 
    - React
    - React Hook
    - JavaScript
    - TypeScript
---
今天来实现一个获取Dom元素Hover状态的自定义Hooks，功能简单，逻辑也很简单。直接上代码：
<!-- more -->
```tsx
function useHover<T>(): [MutableRefObject<T>, boolean] {
  // hover状态
  const [value, setValue] = useState<boolean>(false); 
  // 要hover的dom元素Ref
  const ref: any = useRef<T | null>(null);
  // hover时
  const handleMouseOver = (): void => setValue(true);
  // 取消hover
  const handleMouseOut = (): void => setValue(false);

  useEffect(
    () => {
      const node: any = ref.current;
      if (node) {
        // 绑定事件
        node.addEventListener("mouseover", handleMouseOver);
        node.addEventListener("mouseout", handleMouseOut);
        return () => {
          // 取消绑定
          node.removeEventListener("mouseover", handleMouseOver);
          node.removeEventListener("mouseout", handleMouseOut);
        };
      }
    },
    // Dom改变时重新执行绑定逻辑
    [ref.current] 
  );
  return [ref, value];
}
```