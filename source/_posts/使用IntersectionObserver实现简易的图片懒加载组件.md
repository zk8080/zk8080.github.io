---
title: 使用IntersectionObserver实现简易的图片懒加载组件
date: 2022-01-06 21:28:53
categories: 前端
tags:
    - JavaScript
    - React
    - TypeScript
---
最近在项目中准备做一下首屏加载时间优化，发现大量的图片资源加载，这些静态资源阻塞了页面onload。打算对这些图片进行懒加载处理，图片懒加载成熟的库有很多，今天来自己实现一个简单的懒加载图片组件，加深自己的理解，以及对`IntersectionObserver`API的使用。
<!-- more -->

## 实现
传统方式通过监听容器元素的`scroll`事件，来获取目标元素的坐标判断是否可见，然后加载静态资源。这种实现方式不仅要处理目标元素坐标位置的换算，还要使用防抖或者节流优化`scroll`事件频发触发，防止造成性能问题。
由于`IntersectionObserver`API的出现，我们可以简单的实现一个懒加载的功能。具体API解释可以查看MDN的[IntersectionObserver介绍](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)。直接上代码：

```tsx
import React, { forwardRef, useEffect, useRef, useState } from 'react';

interface IProps extends React.DetailedHTMLProps<React.ImgHTMLAttributes<HTMLImageElement>, HTMLImageElement> {}

interface ComProps extends IProps {
  customizeRef?: React.ForwardedRef<HTMLImageElement>
}

const ImgComponent = (props: ComProps) => {
  const { customizeRef, src, ...rest } = props;
  // 是否可见
  const [loaded, setLoaded] = useState<boolean>(false);
  // 图片ref
  const imgRef = useRef<HTMLImageElement>(null);
  useEffect(() => {
    // 转发ref
    if(customizeRef && typeof customizeRef === "object") {
      customizeRef.current = imgRef.current;
    }
    if(customizeRef && typeof customizeRef === 'function') {
      customizeRef(imgRef.current);
    }

    if(!imgRef.current) return;
    
    // 图片懒加载
    let intersectionObserver: IntersectionObserver = new IntersectionObserver((entries) => {
      if (entries[0].isIntersecting) {
        // isIntersecting为true，则代表当前元素可见
        if(imgRef.current && intersectionObserver) {
          // 加载静态资源
          setLoaded(true);
           // 加载过后，禁止监听该元素
          intersectionObserver.unobserve(imgRef.current);
          intersectionObserver.disconnect();
        }
      }
    });

    // 监听target元素
    intersectionObserver.observe(imgRef.current);

    return () => {
      // 组件卸载时停止监听
      if (intersectionObserver && imgRef.current) {
        intersectionObserver.unobserve(imgRef.current);
        intersectionObserver.disconnect();
      }
    };
    // 图片src更新时，重新执行监听操作
  }, [src])

  return (
    <img {...rest} src={loaded ? src : undefined} ref={imgRef} />
  );
}


// 使用forwardRef转发ref
const LazyLoadImg = forwardRef<HTMLImageElement, IProps>(( props,ref )=><ImgComponent  {...props} customizeRef={ref}  />)

export default LazyLoadImg;
```
