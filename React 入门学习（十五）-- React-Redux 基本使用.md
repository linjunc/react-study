#  React 入门学习（十五）-- React-Redux 基本使用

![react-redux](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-redux.gif)

> 📢 大家好，我是小丞同学，一名<font color=#2e86de>**大二的前端爱好者**</font>
>
> 📢 这篇文章是学习 React-Redux 的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 <font color=#f368e0>**愿你忠于自己，热爱生活**</font>

## 引言

在前面我们学习了 Redux ，我们在写案例的时候，也发现了它存在着一些问题，例如组件无法状态无法公用，每一个状态组件都需要通过订阅来监视，状态更新会影响到全部组件更新，面对着这些问题，React 官方在 redux 基础上提出了 React-Redux 库

在前面的案例中，我们如果把 store 直接写在了 React 应用的顶层 props 中，各个子组件，就能访问到顶层 props

```js
<顶层组件 store={store}>
  <App />
</顶层组件/>
```

这就类似于 React-Redux

## 容器组件和 UI 组件

1. 所有的 UI 组件都需要有一个容器组件包裹
2. 容器组件来负责和 Redux 打交道，可以随意使用 Redux 的API
3. UI 组件无任何 Redux API
4. 容器组件用于处理逻辑，UI 组件只会负责渲染和交互，不处理逻辑

![image-20210910094426268](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210910094426268.png)

在我们的生产当中，我们可以直接将 UI 组件写在容器组件的代码文件当中，这样就无需多个文件

首先，我们在 src 目录下，创建一个 `containers` 文件夹，用于存放各种容器组件，在该文件夹内创建 `Count` 文件夹，即表示即将创建 Count 容器组件，再创建 `index.jsx` 编写代码

要实现容器组件和 UI 组件的连接，我们需要通过 `connect` 来实现

```js
// 引入UI组件
import CountUI from '../../components/Count'
// 引入 connect 连接UI组件
import {connect} from 'react-redux'
// 建立连接
export default connect()(CountUI)
```

后面还会详细讲到

## Provider

由于我们的状态可能会被很多组件使用，所以 React-Redux 给我们提供了一个 Provider 组件，可以全局注入 redux 中的 store ，只需要把 Provider 注册在根部组件即可

例如，当以下组件都需要使用 store 时，我们需要这么做，但是这样徒增了工作量，很不便利

```js
<Count store={store}/>
{/* 示例 */}
<Demo1 store={store}/>
<Demo1 store={store}/>
<Demo1 store={store}/>
<Demo1 store={store}/>
<Demo1 store={store}/>
```

我们可以这么做：在 src 目录下的 `index.js` 文件中，引入 `Provider` ，直接用 `Provider` 标签包裹 `App` 组件，将 `store` 写在 `Provider` 中即可

```js
ReactDOM.render(
  <Provider store={store}>
    <App />
  </Provider>,
  document.getElementById("root")
);
```

这样我们在 `App.jsx` 文件中，组件无需手写指定 `store` ，即可使用 `store` 

## connect

在前面我们看到的 react-redux 原理图时，我们会发现容器组件需要给 UI 组件传递状态和方法，并且是通过 `props` 来传递，看起来很简单。但是，我们会发现容器组件中似乎没有我们平常传递 `props` 的情形

这时候就需要继续研究一下容器组件中的唯一一个函数 `connect` 

connect 方法是一个连接器，用于连接容器组件和 UI 组件，它第一次执行时，接收4个参数，这些参数都是**可选的**，它执行的执行的结果还是一个函数，第二次执行接收一个 UI 组件

第一次执行时的四个参数：`mapStateToProps` 、`mapDispatchToProps` 、`mergeProps`、`options`

### mapStateToProps 

```js
const mapStateToProps = state => ({ count: state })
```

它接收  `state` 作为参数，并且返回一个对象，这个对象标识着 UI 组件的同名参数，

返回的对象中的 key 就作为传递给 UI 组件 props 的 key，value 就作为 props 的 value

如上面的代码，我们可以在 UI 组件中直接通过 props 来读取 `count` 值

```js
<h1>当前求和为：{this.props.count}</h1>
```

这样我们就打通了 UI 组件和容器组件间的状态传递，那如何传递方法呢？

### mapDispatchToProps

connect 接受的第二个参数是 `mapDispatchToProps` 它是用于建立 UI 组件的参数到 `store.dispacth` 方法的映射

我们可以把参数写成对象形式，在这里面定义 action 执行的方法，例如 `jia` 执行什么函数，`jian` 执行什么函数？

我们都可以在这个参数中定义，如下定义了几个方法对应的操作函数

```js
{
    jia: createIncrementAction,
    jian: createDecrementAction,
    jiaAsync: createIncrementAsyncAction
}
```

写到这里其实 `connect` 已经比较完善了，但是你可以仔细想想 `redux` 的工作流程

![image-20210909194900532](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210909194900532.png)

似乎少了点什么，我们在这里调用了函数，创建了 `action` 对象，但是好像 `store` 并没有执行 `dispatch` ，那是不是断了呢？执行不了呢？

其实这里 `react-redux` 已经帮我们做了优化，当调用 `actionCreator` 的时候，会立即发送 `action` 给 `store` 而不用手动的 `dispatch`

- 自动调用 dispatch

## 完整开发

首先我们在 `containers` 文件夹中，直接编写我们的容器组件，无需编写 UI 组件

先打 `rcc` 打出指定代码段，然后暴露出 `connect` 方法

```js
import { connect } from 'react-redux'
```

从 `action` 文件中暴露创建 `action` 的方法

```js
import {createIncrementAction} from '../../redux/count_action'
```

编写 UI 组件，简单写个 demo，绑定 props 和方法

```js
return (
    <div>
        <h2>当前求和为：{this.props.count}</h2>
        <button onClick={this.add}>点我加1</button>
    </div>
);
```

 调用 `connect` 包装暴露 UI 组件

```js
export default connect(
    state => ({ count: state }),// 状态
    { jia: createIncrementAction } // 方法
)(Count);
```

第一次执行的参数就直接传递 `state` 和一个指定 `action` 的对象

---

> 非常感谢您的阅读，欢迎提出你的意见，有什么问题欢迎指出，谢谢！🎈

