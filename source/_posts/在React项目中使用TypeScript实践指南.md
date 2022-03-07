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

#### useState
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

#### useRef
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

#### useCallback
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
useCallback会根据返回值自动推断出类型，如果传入的参数不指定类型，则会默认为`any`，所以为了严谨和可维护性，一定要指定入参的类型。也可以手动传入泛型指定函数类型。如下：
```ts
// 会自动推导出类型： (a: number, b: number) => number;
const add = useCallback((a: number, b: number) => a + b, [a, b])

// 传入泛型，则指定函数类型
const toggle = useCallback<(a: number) => number>((a: number) => a * 2, [a])
```
#### useMemo
声明定义：
```ts
function useMemo<T>(factory: () => T, deps: DependencyList | undefined): T;
   /**
    * `useDebugValue` can be used to display a label for custom hooks in React DevTools.
    *
    * NOTE: We don’t recommend adding debug values to every custom hook.
    * It’s most valuable for custom hooks that are part of shared libraries.
    *
    * @version 16.8.0
    * @see https://reactjs.org/docs/hooks-reference.html#usedebugvalue
    */

```
useMemo和useCallback类似，只是定义类型为具体返回值的类型，而不是函数的类型。如下：
```ts
// 会自动推导出类型： number;
const add = useCallback((a: number, b: number) => a + b, [a, b])

// 传入泛型，则指定函数类型
const toggle = useCallback<number>((a: number) => a * 2, [a])
```
#### useContext
声明定义：
```ts
function useContext<T>(context: Context<T>/*, (not public API) observedBits?: number|boolean */): T;
/**
  * Returns a stateful value, and a function to update it.
  *
  * @version 16.8.0
  * @see https://reactjs.org/docs/hooks-reference.html#usestate
  */
```
useContext会根据传入的上下文对象自动推导出context的类型，当然也可以使用泛型来设置context的类型，如下：
```ts
interface ITheme {
	color: string;
}
const ThemeContext = React.createContext<ITheme>({ color: "red" });

// 自动推导出类型为ITheme
const theme = useContext(ThemeContext); // 等同于const theme = useContext<ITheme>(ThemeContext);
```
#### useReducer
声明定义：
```ts
function useReducer<R extends Reducer<any, any>>(
    reducer: R,
    initialState: ReducerState<R>,
    initializer?: undefined
): [ReducerState<R>, Dispatch<ReducerAction<R>>];
/**
  * `useRef` returns a mutable ref object whose `.current` property is initialized to the passed argument
  * (`initialValue`). The returned object will persist for the full lifetime of the component.
  *
  * Note that `useRef()` is useful for more than the `ref` attribute. It’s handy for keeping any mutable
  * value around similar to how you’d use instance fields in classes.
  *
  * @version 16.8.0
  * @see https://reactjs.org/docs/hooks-reference.html#useref
  */
```
上面只列出了一种类型定义，我在项目中也是使用这种定义去指定`useReducer`的类型。普通的案例如下：
```ts
type StateType = {
  name: string;
  age: number;
}

type Actions = {
  type: 'Change_Name';
  payload: string;
} | {
  type: 'Change_Age';
  payload: number;
}

const initialState = {
  name: '小明',
  age: 18
}

const reducerAction: Reducer<StateType, Actions> = (
  state,
  action,
) => {
  switch (action.type) {
    case 'Change_Name':
      return { ...state, name: action.payload };
    case 'Change_Age':
      return { ...state, age: action.payload };
    default:
      return state;
  }
};

function Index() {
  const [state, dispatch] = useReducer(reducerAction, initialState);
  return (
    <div>
      <div>姓名：{state.name}</div>
      <div>年龄：{state.age}</div>
    </div>
  );
}

```
可以看到，这样能够得到正确的类型推断，但是略微繁琐。在这篇[文章](https://medium.com/hackernoon/finally-the-typescript-redux-hooks-events-blog-you-were-looking-for-c4663d823b01)中，
学到了一个泛型定义，可以稍微简化一下定义`Actions`的过程。案例如下：
```ts
// 定义一个生成Action类型的泛型
type ActionMap<M extends Record<string, any>> = {
  [Key in keyof M]: M[Key] extends undefined
    ? {
        type: Key
      }
    : {
        type: Key
        payload: M[Key]
      }
}

type StateType = {
  name: string;
  age: number;
}

// 定义具体的Action类型
type PayloadType = {
  Change_Name: string;
  Change_Age: number;
}

/** 
  ActionMap<PayloadType>会生成类型
  {
    Change_Name: {
        type: Types.Name;
        payload: string;
    };
    Change_Age: {
        type: Types.Age;
        payload: number;
    };
  }
  而keyof ActionMap<PayloadType>则会生成 'Change_Name' | 'Change_Age'的类型。
  所以Action最终的类型便为：
  type Actions = {
      type: Types.Name;
      payload: string;
  } | {
      type: Types.Age;
      payload: number;
  }
*/
type Actions = ActionMap<PayloadType>[keyof ActionMap<PayloadType>]

const initialState = {
  name: '小明',
  age: 18
}

const reducerAction: Reducer<StateType, Actions> = (
  state,
  action,
) => {
  switch (action.type) {
    case Types.Name:
      return { ...state, name: action.payload };
    case Types.Age:
      return { ...state, age: action.payload };
    default:
      return state;
  }
};
```
我们定义了一个`ActionMap`泛型，该泛型会将传入的类型`{key: value}`生成为新的`{key: {type: key, payload: value }`类型。然后我们利用`keyof`关键字获取到所有的key，就可以得到我们所需要的`{type: key1, payload: value1} | {type: key2, payload: value2}`的类型了。只要我们定义好`PayloadType`类型，则可以自动推导出我们需要的`Actions`类型。
如果你觉得这样写还是很繁琐，那么可以去看我的这篇文章[在TypeScript中使用useReducer](https://zkat.site/2021/11/03/%E5%9C%A8TypeScript%E4%B8%AD%E4%BD%BF%E7%94%A8useReducer/#%E7%AE%80%E5%8C%96%E7%9A%84useReducer%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F)，里面介绍了简化的useReducer使用方式。