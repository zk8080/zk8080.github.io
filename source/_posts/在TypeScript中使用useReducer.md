---
title: 在TypeScript中使用useReducer
date: 2021-11-03 21:31:55
tags: 
    - React Hook
    - TypeScript
---
在项目中已经使用了很久的TypeScript和Hooks，记录一下如何在TypeScript中友好的使用`useReducer`。
<!-- more -->
## 简单的例子🌰
先实现一个简单的例子
```tsx
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
在这个例子中，我们的TypeScript类型已经全部加入，并且可以得到正确的类型推断。但是大家可以发现`Actions`类型在每增加一个状态时都需要再手动增加一个类型，我们可以优化一下该步骤，定义一个`ActionMap`泛型和一个`PayloadType`类型来自动生成这个联合类型。

## 自动生成Actions联合类型
优化上处代码如下：
```tsx
// 定义一个生成Action类型的泛型
type ActionMap<M extends { [index: string]: any }> = {
  [Key in keyof M]: M[Key] extends undefined
    ? {
        type: Key
      }
    : {
        type: Key
        payload: M[Key]
      }
}

enum Types {
  Name = 'Change_Name',
  Age = 'Change_Age',
}

type StateType = {
  name: string;
  age: number;
}

// 定义具体的Action类型
type PayloadType = {
  [Types.Name]: string;
  [Types.Age]: number;
}

type Action = ActionMap<PayloadType>[keyof ActionMap<PayloadType>]

const initialState = {
  name: '小明',
  age: 18
}

const reducerAction: Reducer<StateType, Action> = (
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
这样优化后，我们新增状态后，只需要增加`Types`和`PayloadType`的类型定义，便可自动生成相关的`Action`类型。

## 简化的useReducer使用方式
看完上面的代码，你可能会说使用`useReducer`过于繁琐。那我们可以简化使用成本，放弃`action.type`的判断。
```tsx
type StateType = {
  name: string;
  age: number;
}

const initialState = {
  name: '小明',
  age: 18
}

const simpleReducer = (prevState: StateType, updatedProperty: ): StateType => ({
  ...prevState,
  ...updatedProperty
})

function Index() {
  const [state, setState] = useReducer(simpleReducer, initialState);
  return (
    <div>
      <div>姓名：{state.name}</div>
      <div>年龄：{state.age}</div>
      <button
        onClick={() => {
          setState({
            name: '小李'
          })
        }}
      >
        修改姓名
      </button>
    </div>
  );
}
```
我们的reducer函数，接受一个`Partial<StateType>`类型的对象，每次调用时使用扩展运算符的方式返回一个新的State。这样可以略过`ActionType`的定义和匹配调用更新，减少模板代码的冗余。但是也会带来代码逻辑不清晰的缺点，可以根据自己的喜好来决定使用哪一种方式。