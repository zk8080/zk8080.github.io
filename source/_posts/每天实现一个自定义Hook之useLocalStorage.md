---
title: 每天实现一个自定义Hook之useLocalStorage
date: 2021-09-02 23:16:25
tags: 
    - React
    - React Hook
    - JavaScript
    - TypeScript
---
今天来实现一个可以将`state`同步存储在`localStorage`中的自定义Hook。直接上代码：
<!-- more -->

```tsx
// 接受要设置key和初始值
function useLocalStorage<T>(key: string, initialValue: T) {
  // 将接受的初始值传入state
  const [storageValue, setStorageValue] = useState<T>(() => {
    try {
      // 先从localStorage中获取key，判断是否有值，有则用本地，否则使用初始值
      const localValue = window.localStorage.getItem(key);
      return localValue ? JSON.parse(localValue) : initialValue;
    } catch (error) {
      console.log(error);
      return initialValue;
    }
  });
  
  // 封装setStorageValue, 将传入的值存入localStorage
  const setValue = (value: T | ((val: T) => T)) => {
    try {
      // 支持传入回调函数，并将storageValue传入回调
      const newValue =
        value instanceof Function ? value(storageValue) : value;
      // 更新state
      setStorageValue(newValue);
      // 保存到localStorage
      window.localStorage.setItem(key, JSON.stringify(newValue));
    } catch (error) {
      console.log(error);
    }
  };
  return [storageValue, setValue] as const;
}
```