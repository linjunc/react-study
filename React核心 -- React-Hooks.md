# React核心 -- React-Hooks

![hooks](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/hooks.gif)

## hooks 存在的意义

1. hooks 之间的状态是独立的，有自己独立的上下文，不会出现混淆状态的情况

2. 让函数有了状态管理
3. 解决了 组件树不直观、类组件难维护、逻辑不易复用的问题
4. 避免函数重复执行的副作用

## 应用场景

1. 利用 hooks 取代生命周期函数
2. 让组件有了状态
3. 组件辅助函数
4. 处理发送请求
5. 存取数据
6. 做好性能优化

## hooks API

从 `react` 中引入

### 1. useState

给函数组件添加状态

- 初始化以及更新组件状态

```js
const [count, setCount] = React.useState(0)
```

接收一个参数作为初始值，返回一个数组：第一个是状态变量，第二个是修改变量的函数

### 2. useEffect

副作用 hooks

- 给没有生命周期的组件，添加结束渲染的信号

注意：

- render 之后执行的 hooks

第一个参数接收一个函数，在组件更新的时候执行

第二个参数接收一个数组，用来表示需要追踪的变量，依赖列表，只有依赖更新的时候才会更新内容

第一个参数的返回值，返回一个函数，在 `useEffect` 执行之前，都会先执行里面返回的函数

一般用于添加销毁事件，这样就能保证只添加一个

```js
React.useEffect(() => {
    console.log('被调用了');
    return () => {
        console.log('我要被卸载了');
    }
}, [count])
```

打印

![image-20210914172936843](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210914172936843.png)

### 3. useLayoutEffect

和 `useEffect` 很类似

它的作用是：在 DOM 更新完成之后执行某个操作

注意：

- 有 DOM 操作的副作用 hooks
- 在 DOM 更新之后执行

> 执行时机在 `useEffect` 之前，其他都和 `useEffect` 都相同

`useEffect` 执行时机在 **render 之后**

`useLayoutEffect` 执行时机在 **DOM 更新之后**

### 4. useMemo

作用：让组件中的函数跟随状态更新

注意：优化函数组件中的功能函数

**为了避免由于其他状态更新导致的当前函数的被迫执行**

第一个参数接收一个函数，第二个参数为数组的依赖列表，返回一个值

```js
const getDoubleNum = useMemo(() => {
    console.log('ddd')
    return 2 * num
}, [num])
```

#### 5. useCallback

作用：跟随状态更新执行

注意：

- 只有依赖项改变时才执行
- `useMemo( () => fn, deps)` 相当于 `useCallback(fn, deps)`

不同点：

1. `useCallback` **返回的是一个函数，不再是值**
2. `useCallback` 缓存的是一个函数，`useMemo` 缓存的是一个值，**如果依赖不更新，返回的永远是缓存的那个函数**
3. 给子组件中传递  `props`  的时候，如果当前组件不更新，不会触发子组件的重新渲染

### 6. useRef

作用：长久保存数据

注意事项：

- 返回一个子元素索引，这个索引在整个生命周期中保持不变
- 对象发生改变时，不通知，属性变更不重新渲染

1. 保存一个值，在整个生命周期中维持不变
2. 重新赋值 `ref.current` 不会触发重新渲染
3. 相当于**创建一个额外的容器来存储数据**，我们可以在外部拿到这个值

当我们通过正常的方式去获取计时器的 `id` 是无法获取的，需要通过 `ref` 

```js
useEffect(() => {
    ref.current = setInterval(() => {
        setNum(num => num + 1)
    }, 400)
}, [])
useEffect(() => {
    if (num > 10) {
        console.log('到十了');
        clearInterval(ref.current)
    }
}, [num])
```

### 7. useContext

作用：带着子组件渲染

注意：

- 上层数据发生改变，肯定会触发重新渲染

1. 我们需要引入 `useContext` 和 `createContext` 两个内容
2. 通过 `createContext` 创建一个 `Context` 句柄
3. 通过 `Provider` 确定数据共享范围
4. 通过 `value` 来分发数据
5. 在子组件中，通过 `useContext` 来获取数据

```js
import React, { useContext, createContext } from 'react'
const Context = createContext(null)
export default function Hook() {
    const [num, setNum] = React.useState(1)

    return (
        <h1>
            这是一个函数组件 - {num}
        // 确定范围
            <Context.Provider value={num}>
                <Item1 num={num} />
                <Item2 num={num} />
            </Context.Provider>
        </h1>
    )
}
function Item1() {
    const num = useContext(Context)
    return <div>子组件1  {num}</div>
}
function Item2() {
    const num = useContext(Context)
    return <div>子组件2 {num}</div>
}

```

### 8. useReducer

作用：去其他地方借资源

注意：函数组件的 Redux 的操作

1. 创建数据仓库 `store` 和管理者 `reducer`
2. 通过 `useReducer(store,dispatch)` 来获取 `state` 和 `dispatch`

```js
const store = {
    num: 10
}
const reducer = (state, action) => {
    switch (action.type) {
        case "":
            return
        default:
            return
    }
}
    const [state, dispatch] = useReducer(reducer, store)
```

通过 `dispatch` 去派发 `action`

### 9. 自定义 hooks

放在 `utils` 文件夹中，以 `use` 开头命名

例如：模拟数据请求的 Hooks

```js
import React, { useState, useEffect } from "react";
function useLoadData() {
  const [num, setNum] = useState(1);
  useEffect(() => {
    setTimeout(() => {
      setNum(2);
    }, 1000);
  }, []);
  return [num, setNum];
}
export default useLoadData;
```

减少代码耦合

我们希望 reducer 能让每个组件来使用，我们自己写一个 hooks

自定义一个自己的 LocalReducer

```js
import React, { useReducer } from "react";
const store = { num: 1210 };
const reducer = (state, action) => {
  switch (action.type) {
    case "num":
      return { ...state, num: action.num };
    default:
      return { ...state };
  }
};
function useLocalReducer() {
  const [state, dispatch] = useReducer(reducer, store);
  return [state, dispatch];
}
export default useLocalReducer;
```

1. 引入 react 和自己需要的 hook
2. 创建自己的hook函数
3. 返回一个数组，数组中第一个内容是数据，第二个是修改数据的函数
4. 暴露自定义 hook 函数出去
5. 引入自己的业务组件

