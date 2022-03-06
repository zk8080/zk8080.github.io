---
title: 在React项目中使用TypeScript实践指南
date: 2022-03-06 15:29:42
tags:
    - React Hook
    - TypeScript
    - React
---
本文主要记录我如何在React项目中优雅的使用TypeScript，来提高开发效率及项目的健壮性。
<!-- more -->
## 项目目录及ts文件划分
由于实际项目中大部分是使用`umi`来进行开发项目，所以使用`umi`生成的目录来做案例。

```
.
├── README.md
├── global.d.ts
├── mock
├── package.json
├── src
│   ├── assets
│   ├── components
│   │   └── PublicComA
│   │       ├── index.d.ts
│   │       ├── index.less
│   │       └── index.tsx
│   ├── layouts
│   ├── models
│   ├── pages
│   │   ├── PageA
│   │   │   ├── index.d.ts
│   │   │   ├── index.less
│   │   │   └── index.tsx
│   │   ├── index.less
│   │   └── index.tsx
│   └── utils
├── tsconfig.json
├── typings.d.ts
└── yarn.lock
```
在项目根目录下有typings.d.ts和global.d.ts这两个文件, 前者我们可以放置一些全局的导出模块,比如css,less, 图片的导出声明；后者可以放一些全局声明的变量, 接口等, 比如说window下全局变量的声明等。如下：
```ts
// typings.d.ts
declare module '*.css';
declare module '*.less';
declare module "*.png";
declare module "*.jpeg";
declare module '*.svg' {
  export function ReactComponent(props: React.SVGProps<SVGSVGElement>): React.ReactElement
  const url: string
  export default url
}
```
```ts
// global.d.ts
interface Window {
  appStateChange: () => void;
}
```
接下来介绍一下src目录：
+ assets 存放静态资源如图片/视频/音频等, 参与webpack的打包过程
+ layouts 存放公共布局
+ components  存放全局共同组件
+ models  dva的models文件夹, 处理redux流
+ pages  存放页面的目录, 内部可以有页面组件components, 结构类似于全局的components
+ utils  存放js工具库, 请求库等公共js文件

在pages和components中有存放当前组件/页面所需要的类型和接口声明的index.d.ts。另外如models中的文件由于是每个model私有类型和接口声明，所以可以直接在文件内部去声明。
具体的目录规划如上，可以根据实际项目来做更合理的划分。

## 在项目中使用TypeScript具体实践

### 组件声明
1. 函数组件
推荐使用`React.FC<P={}>`来表示函数类型，当使用该类型定义组件时，props中会默认带有children属性。
```ts
interface IProps {
  count: number
}

const App: React.FC<IProps> = (props) => {
  const {count} = props;
  return (
    <div className="App">
      <span>count: {count}</span>
    </div>
  );
}
```
2. 类组件
类组件接受两个参数，第一个是props的定义，第二个是state的定义，如果使用`React.PureComponent<P, S={} SS={}>`定义组件，则还有第三个参数，表示`getSnapshotBeforeUpdate`的返回值。
```ts
interface IProps {
  name: string;
}

interface IState {
  count: number;
}

class App extends React.Component<IProps, IState> {
  state = {
    count: 0
  };
  render() {
    return (
      <div>
        {this.state.count}
        {this.props.name}
      </div>
    );
  }
}
```

### React Hooks使用
1. useState
声明定义：
```ts
function useState<S>(initialState: S | (() => S)): [S, Dispatch<SetStateAction<S>>];
// convenience overload when first argument is omitted
	/**
	 * Returns a stateful value, and a function to update it.
   *
   * @version 16.8.0
   * @see https://reactjs.org/docs/hooks-reference.html#usestate
   */
    
function useState<S = undefined>(): [S | undefined, Dispatch<SetStateAction<S | undefined>>];
  /**
   * An alternative to `useState`.
   *
   * `useReducer` is usually preferable to `useState` when you have complex state logic that involves
   * multiple sub-values. It also lets you optimize performance for components that trigger deep
   * updates because you can pass `dispatch` down instead of callbacks.
   *
   * @version 16.8.0
   * @see https://reactjs.org/docs/hooks-reference.html#usereducer
   */

```
如果初始值能够体现出类型，那么可以不用手动声明类型，TS会自动推断出类型。如果初始值为null或者undefined则需要通过泛型显示声明类型。如下：
```ts
const [count, setCount] = useState(1);

const [user, setUser] = useState<IUser | null>(null);
```

2. useRef
声明定义：
```ts
 function useRef<T>(initialValue: T): MutableRefObject<T>;
 // convenience overload for refs given as a ref prop as they typically start with a null value
 /**
   * `useRef` returns a mutable ref object whose `.current` property is initialized to the passed argument
   * (`initialValue`). The returned object will persist for the full lifetime of the component.
   *
   * Note that `useRef()` is useful for more than the `ref` attribute. It’s handy for keeping any mutable
   * value around similar to how you’d use instance fields in classes.
   *
   * Usage note: if you need the result of useRef to be directly mutable, include `| null` in the type
   * of the generic argument.
   *
   * @version 16.8.0
   * @see https://reactjs.org/docs/hooks-reference.html#useref
   */
```
使用该Hook时，要根据使用场景来判断传入泛型类型，如果是获取DOM节点，则传入对应DOM类型即可；如果需要的是一个可变对象，则需要在泛型参数中包含'| null'。如下：
```ts
// 不可变DOM节点，只读
const inputRef = useRef<HTMLInputElement>(null);

// 可变，可重新复制
const idRef = useRef<string | null>(null);
idRef.current = "abc";
```

3. useCallback
声明定义：
```ts
 function useCallback<T extends (...args: any[]) => any>(callback: T, deps: DependencyList): T;
 /**
  * `useMemo` will only recompute the memoized value when one of the `deps` has changed.
  *
  * Usage note: if calling `useMemo` with a referentially stable function, also give it as the input in
  * the second argument.
  *
  * ```ts
  * function expensive () { ... }
  *
  * function Component () {
  *   const expensiveResult = useMemo(expensive, [expensive])
  *   return ...
  * }
  * ```
  *
  * @version 16.8.0
  * @see https://reactjs.org/docs/hooks-reference.html#usememo
  */
```