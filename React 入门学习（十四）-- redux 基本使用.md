#  React 入门学习（十四）-- redux 基本使用

![redux基础](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/redux%E5%9F%BA%E7%A1%80.gif)

> 📢 大家好，我是小丞同学，一名<font color=#2e86de>**大二的前端爱好者**</font>
>
> 📢 这篇文章是学习 Redux 的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 <font color=#f368e0>**愿你忠于自己，热爱生活**</font>

## 引言

在了解了 Antd 组件库之后，我们现在开始学习了 Redux ，在我们之前写的案例当中，例如：todolist 案例，GitHub 搜索案例当中，我们对于状态的管理，都是通过 state 来实现的，比如，我们在给兄弟组件传递数据时，需要先将数据传递给父组件，再由父组件转发 给它的子组件。这个过程十分的复杂，后来我们又学习了**消息的发布订阅**，我们通过 **pubsub** 库，实现了消息的转发，直接将数据发布，由兄弟组件订阅，实现了兄弟组件间的数据传递。但是，随着我们的需求不断地提升，我们需要进行更加复杂的数据传递，更多层次的数据交换。**因此我们为何不可以将所有的数据交给一个中转站，这个中转站独立于所有的组件之外，由这个中转站来进行数据的分发，这样不管哪个组件需要数据，我们都可以很轻易的给他派发。**

而有这么一个库就可以帮助我们来实现，那就是 Redux ，它可以帮助我们实现集中式状态管理

## 1. 什么情况使用 Redux ？

首先，我们先明晰 `Redux` 的作用 ，实现集中式状态管理。

 `Redux`  适用于多交互、多数据源的场景。简单理解就是**复杂**

从组件角度去考虑的话，当我们有以下的应用场景时，我们可以尝试采用 `Redux` 来实现

1. 某个组件的状态需要共享时
2. 一个组件需要改变其他组件的状态时
3. 一个组件需要改变全局的状态时

除此之外，还有很多情况都需要使用 Redux 来实现（还没有学  hook，或许还有更好的方法）

![image-20210909194446988](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210909194446988.png)



（从掘友的文章里截的图）

这张图，非常形象的将纯 React 和 采用 Redux 的区别体现了出来

## 2. Redux 的工作流程

![image-20210909194900532](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210909194900532.png)

 首先组件会在 Redux 中派发一个 `action` 方法，通过调用 `store.dispatch` 方法，将 `action` 对象派发给 `store` ，当 `store` 接收到 `action` 对象时，会将先前的 `state` 与传来的 `action` 一同发送给 `reducer` ，`reducer`  在接收到数据后，进行数据的更改，返回一个新的状态给 `store` ，最后由 `store` 更改 `state` 

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/11/12/16e5fd1597faec4d~tplv-t2oaga2asx-watermark.awebp)

（图来自掘金社区，侵删）

## 3. Redux 三个核心概念

#### 1. store

`store` 是 Redux 的核心，可以理解为是 Redux 的数据中台，我们可以将任何我们想要存放的数据放在 `store` 中，在我们需要使用这些数据时，我们可以从中取出相应的数据。因此我们需要先创建一个 `store` ，在 Redux 中可以使用 `createStore` API 来创建一个 `store` 

在生产中，我们需要在 `src` 目录下的 `redux` 文件夹中新增一个 `store.js` 文件，在这个文件中，创建一个 `store` 对象，并暴露它

因此我们需要从 `redux` 中暴露两个方法 

```js
import {
    createStore,
    applyMiddleware
} from 'redux'
```

并引入为 count 组件服务的 reducer

```js
import countReducer from './count_reducer'
```

最后调用 `createStore` 方法来暴露 `store` 

```js
export default createStore(countReducer, applyMiddleware(thunk))
```

这里采用了中间件，本文应该不会写到~

在 `store` 对象下有一些常用的内置方法

获取当前时刻的 `store` ，我们可以采用 `getStore` 方法

```js
const state = store.getState();
```

在前面我们的流程图中，我们需要通过 `store` 中的 `dispatch` 方法来派生一个 `action` 对象给 `store`

```js
store.dispatch(`action对象`)
```

最后还有一个 `subscribe` 方法，这个方法可以帮助我们订阅 `store` 的改变，只要 `store` 发生改变，这个方法的回调就会执行

为了监听数据的更新，我们可以将 `subscribe` 方法绑定在组件挂载完毕生命周期函数上，但是这样，当我们的组件数量很多时，会比较的麻烦，因此我们可以直接将 `subscribe` 函数用来监听整个 `App`组件的变化

```js
store.subscribe(() => {
    ReactDOM.render( < App /> , document.getElementById('root'))
})
```

#### 2. action

`action` 是 `store` 中唯一的数据来源，一般来说，我们会通过调用 `store.dispatch` 将 action 传到 store 

我们需要传递的 `action` 是一个对象，它必须要有一个 `type` 值

例如，这里我们暴露了一个用于返回一个 `action` 对象的方法

```js
export const createIncrementAction = data => ({
    type: INCREMENT,
    data
})
```

我们调用它时，会返回一个 `action` 对象

#### 3. reducer

在 Reducer 中，我们需要指定状态的操作类型，要做怎样的数据更新，因此这个类型是必要的。

reducer 会根据 action 的指示，对 state 进行对应的操作，然后返回操作后的 state 

如下，我们对接收的 action 中传来的 type 进行判断

```js
export default function countReducer(preState = initState, action) {
    const {
        type,
        data
    } = action;
    switch (type) {
        case INCREMENT:
            return preState + data
        case DECREMENT:
            return preState - data
        default:
            return preState
    }
}
```

更改数据，返回新的状态

## 4. 创建 constant 文件

在我们正常的编码中，有可能会出现拼写错误的情况，但是我们会发现，拼写错误了不一定会报错，因此就会比较难搞。

我们可以在 `redux` 目录下，创建一个 `constant` 文件，这个文件用于定义我们代码中常用的一些变量，例如

```js
export const INCREMENT = 'increment'
export const DECREMENT = 'decrement'
```

将这两个单词写在 `constant` 文件中，并对外暴露，当我们需要使用时，我们可以引入这个文件，并直接使用它的名称即可

直接使用 `INCREMENT` 即可

## 5. 实现异步 action

一开始，我们直接调用一个异步函数，这虽然没有什么问题，但是难道 redux 就不可以实现了吗？

```js
incrementAsync = () => {
    const { value } = this.selectNumber
    const { count } = this.state;
    setTimeout(() => {
        this.setState({ count: count + value * 1 })
    }, 500);
}
```

我们可以先尝试将它封装到 `action` 对象中调用

```js
export const createIncrementAsyncAction = (data, time) => {
    // 无需引入 store ，在调用的时候是由 store 调用的
    return (dispatch) => {
        setTimeout(() => {
            dispatch(createIncrementAction(data))
        }, time)
    }
}
```

当我们点击异步加操作时，我们会调用这个函数，在这个函数里接收一个延时加的时间，还有action所需的数据，和原先的区别只在于返回的时一个定时器函数

但是如果仅仅这样，很显然是会报错的，它默认需要接收一个对象

如果我们需要实现传入函数，那我们就需要告诉：你只需要默默的帮我执行以下这个函数就好！

这时我们就需要引入中间件，在原生的 `redux` 中暴露出 `applyMiddleware` 中间件执行函数，并引入 `redux-thunk` 中间件（需要手动下载）

```js
import thunk from 'redux-thunk'
```

通过第二个参数传递下去就可以了

```js
export default createStore(countReducer, applyMiddleware(thunk))
```

注意：异步 action 不是必须要写的，完全可以自己等待异步任务的结果后再去分发同步action

> 采用 `react-thunk` 能让异步代码像同步代码一样执行，在 `redux` 中我们也是可以实现异步的，但是这样我们的代码中会有很多异步的细节，这不是我们想看到的，利用 `react-thunk` 之类的库，就能让我们只关心我们的业务

## 6. Redux 三大原则

理解好 Redux 有助于我们更好的理解接下来的 React -Redux

### 第一个原则

**单向数据流**：整个 Redux 中，数据流向是单向的

UI 组件 --->   action  --->  store  --->  reducer --->  store

### 第二个原则

**state 只读**：在 Redux 中不能通过直接改变 state ，来控制状态的改变，如果想要改变 state ，则需要触发一次 action。通过 action 执行 reducer

### 第三个原则

**纯函数执行**：每一个reducer 都是一个纯函数，不会有任何副作用，返回是一个新的 state，state 改变会触发 store 中的 subscribe

## 参考资料

[Redux + React-router 的入门📖和配置👩🏾‍💻教程](https://juejin.cn/post/6844903998139400200)

小册：[React 进阶实践指南](https://juejin.cn/book/6945998773818490884)

---

> 非常感谢您的阅读，欢迎提出你的意见，有什么问题欢迎指出，谢谢！🎈





