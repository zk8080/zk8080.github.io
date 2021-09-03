---
title: 每天实现一个自定义Hook之useToggle
date: 2021-09-03 23:05:31
tags: 
    - React
    - React Hook
    - JavaScript
---
今天来实现一个切换布尔状态值的自定义Hook。直接上代码：
<!-- more -->
```tsx
const useToggle = (initialState: boolean = false): [boolean, any] => {
    // 接受默认布尔状态，默认为false
    const [state, setState] = useState<boolean>(initialState);
    // 创建切换函数，布尔值取反
    // 使用useCallback优化函数定义，防止函数重新定义 导致组件不必要的渲染
    const toggle = useCallback((): void => setState(state => !state), []);
    return [state, toggle];
}

```