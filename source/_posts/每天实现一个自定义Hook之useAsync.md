---
title: 每天实现一个自定义Hook之useAsync
date: 2021-09-15 23:09:18
tags: 
    - React
    - React Hook
    - JavaScript
    - TypeScript
---
今天来实现一个简单的用来处理`async`函数或者`Promise`函数的自定义Hook。`useAsync`只会单纯的处理异步操作，返回请求的状态和数据，如果想要使用功能齐全和成熟的的异步请求自定义Hook，可以使用[useSWR](https://swr.vercel.app/zh-CN)或者[useRequest](https://ahooks.js.org/zh-CN/hooks/async)。接下来来实现我们的`useAsync`，直接上代码：
<!-- more -->
```tsx
interface AsyncData<T> {
	data: T | null;
	isLoading: boolean;
	error: any;
} 

const useAsync = <T>(
  fn: () => Promise<T>,
  deps: any[]
): AsyncData<T> => {
  // 异步操作需要返回的state
  const [data, setData] = useState<T | null>(null);
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState();
  
  // 利用useCallback缓存异步操作函数
  const callback = useCallback(() => {
    // loading状态
    setIsLoading(true);
    return fn().then((response: T) => {
        // success
        setIsLoading(false);
        setData(response);
      })
      .catch((error: any) => {
        // error
        setIsLoading(false);
        setError(error);
      });
  }, deps);

  useEffect(() => {
    // 依赖改变时重新执行异步操作
    callback();
  }, [callback]);

  return {
    data,
    isLoading,
    error,
  };
};
```