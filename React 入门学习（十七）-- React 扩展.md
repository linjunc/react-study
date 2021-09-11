#  React 入门学习（十七）-- React 扩展

![react-扩展](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-%E6%89%A9%E5%B1%95.gif)

> 📢 大家好，我是小丞同学，一名<font color=#2e86de>**大二的前端爱好者**</font>
>
> 📢 这篇文章是学习 React扩展部分的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 <font color=#f368e0>**愿你忠于自己，热爱生活**</font>

## 引言

学到这里 React 已经学的差不多了，接下来就学习一些 React 扩展内容，可以帮助我们更好的开发和理解，这部分的知识还有很多的东西可以探寻，比如：网红 React-Hook，就是我们需要注意的地方，打了 100 多集的类式组件，出来一个 hooks ，现在用函数式组件偏多了.............

所以 Hooks 就需要我们深入的学习一下了，下面我们就一起来看看扩展部分有哪些内容吧

## 1. setState

### 对象式 setState

首先在我们以前的认知中，`setState` 是用来更新状态的，我们一般给它传递一个对象，就像这样

```js
this.setState({
    count: count + 1
})
```

这样每次更新都会让 `count` 的值加 1。这也是我们最常做的东西

这里我们做一个案例，点我加 1，一个按钮一个值，我要在控制台输出每次的 `count` 的值

![react-extension-demo1](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-extension-demo1.gif)

那我们需要在控制台输出，要如何实现呢？

我们会考虑在 `setState` 更新之后 `log` 一下

```js
add = () => {
    const { count } = this.state
    this.setState({
        count: count + 1
    })
    console.log(this.state.count);
}
```

因此可能会写出这样的代码，看起来很合理，在调用完 `setState` 之后，输出 `count` 

![react-extension-demo1-2](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-extension-demo1-2.gif)

我们发现显示的 `count` 和我们控制台输出的 `count` 值是不一样的

这是因为，我们调用的 `setState` 是同步事件，但是它的作用是让 React 去更新数据，而 React 不会立即的去更新数据，这是一个异步的任务，因此我们输出的 `count` 值会是状态更新之前的数据。“React **状态更新是异步的**”

那我们要如何实现同步呢？

其实在 `setState` 调用的第二个参数，我们可以接收一个函数，这个函数会在状态更新完毕并且界面更新之后调用，我们可以试试

```js
add = () => {
    const { count } = this.state
    this.setState({
        count: count + 1
    }, () => {
        console.log(this.state.count)
    })
}
```

我们将 `setState` 填上第二个参数，输出更新后的 `count` 值

![react-extension-demo1-3](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-extension-demo1-3.gif)

这样我们就能成功的获取到最新的数据了，如果有这个需求我们可以在第二个参数输出噢~

### 函数式 setState

这种用法我也是第一次见，函数式的 `setState` 也是接收两个参数

第一个参数是 `updater`  ，它是一个能够返回 `stateChange` 对象的函数

第二个参数是一个回调函数，用于在状态更新完毕，界面也更新之后调用

与对象式 `setState` 不同的是，我们传递的第一个参数 `updater` 可以接收到2个参数 `state` 和 `props` 

我们尝试一下

```js
add = () => {
    this.setState((state) => ({ count: state.count + 1 }))
}
```

![react-extension-demo2-1](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-extension-demo2-1.gif)

我们也成功的实现了

我们在第一个参数中传入了一个函数，这个函数可以接收到 `state` ，我们通过更新 `state` 中的 `count` 值，来驱动页面的更新

利用函数式 `setState` 的优势还是很不错的，可以直接获得 `state` 和 `props` 

> 可以理解为对象式的 `setState` 是函数式 `setState` 的语法糖

## 2. LazyLoad

懒加载在 React 中用的最多的就是路由组件了，页面刷新时，所有的页面都会重新加载，这并不是我们想要的，我们想要实现点击哪个路由链接再加载即可，这样避免了不必要的加载

![react-extension-demo2-2](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-extension-demo2-2.gif)

我们可以发现，我们页面一加载时，所有的路由组件都会被加载

如果我们有 100 个路由组件，但是用户只点击了几个，这就会有很大的消耗，因此我们需要做懒加载处理，**我们点击哪个时，才去加载哪一个**

首先我们需要从 `react` 库中暴露一个 `lazy` 函数

```js
import React, { Component ,lazy} from 'react';
```

然后我们需要更改引入组件的方式

```js
const Home = lazy(() => import('./Home'))
const About = lazy(() => import('./About'))
```

采用 `lazy` 函数包裹

![image-20210911114307684](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210911114307684.png)

我们会遇到这样的错误，提示我们用一个标签包裹

这里是因为，当我们网速慢的时候，路由组件就会有可能加载不出来，页面就会白屏，它需要我们来指定一个路由组件加载的东西，相对于 loading

```js
<Suspense fallback={<h1>loading</h1>}>
    <Route path="/home" component={Home}></Route>
    <Route path="/about" component={About}></Route>
</Suspense>
```

> 在做这个案例的时候，一定不要设置重定向的东西，所有的路由我们要点击再加载

初次登录页面的时候

![image-20210911115542647](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210911115542647.png)

注意噢，这些文件都不是路由组件，当我们点击了对应组件之后才会加载

![react-extension-lazyload-2](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-extension-lazyload-2.gif)

从上图我们可以看出，每次点击时，才会去请求 `chunk` 文件

那我们更改写的 `fallback` 有什么用呢？它会在页面还没有加载出来的时候显示

![react-extension-lazyload-3](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-extension-lazyload-3.gif)

> 注意：因为 loading 是作为一个兜底的存在，因此 loading 是 必须提前引入的，不能懒加载

## 3. Hooks

### useState

`hooks` 解决了函数式组件和类式组件的差异，让函数式组件拥有了类式组件所拥有的 `state` ，同时新增了一些 API ，让函数式组件，变得更加的灵活

首先我们需要明确一点，函数式组件**没有**自己的 `this`

```js
function Demo() {
    const [count, setCount] = React.useState(0)
    console.log(count, setCount);
    function add() {
        setCount(count + 1)
    }
    return (
        <div>
            <h2>当前求和为：{count}</h2>
            <button onClick={add}>点我加1</button>
        </div>
    )
}
export default Demo
```

利用函数式组件完成的 **点我加1** 案例

这里利用了一个 Hook ：`useState` 

它让函数式组件能够维护自己的 `state` ，它**接收一个参数**，作为**初始化** `state` 的值，赋值给 `count`，因此 `useState` 的初始值只有**第一次有效**，它所映射出的两个变量 `count` 和 `setCount` 我们可以理解为 `setState` 来使用

> **useState 能够返回一个数组，第一个元素是 state ，第二个是更新 state 的函数**

我们先看看控制台输出的什么

![image-20210911123011304](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210911123011304.png)

`count` 是初始化的值，而 `setCount` 就像是一个 `action` 对象驱动状态更新

我们可以通过 `setCount` 来更新 `count` 的值

```js
setCount(count + 1)
```

### useEffect 

 在类式组件中，提供了一些声明周期钩子给我们使用，我们可以在组件的特殊时期执行特定的事情，例如 `componentDidMount` ，能够在组件挂载完成后执行一些东西

在函数式组件中也可以实现，它采用的是 `effectHook` ，它的语法更加的简单，同时融合了 `componentDidUpdata` 生命周期，极大的方便了我们的开发

```js
React.useEffect(() => {
    console.log('被调用了');
})
```

由于函数的特性，我们可以在函数中随意的编写函数，这里我们调用了 `useEffect` 函数，这个函数有多个功能

当我们像上面代码那样使用时，它相当于 `componentDidUpdata` 和 `componentDidMount` 一同使用，也就是在**组件挂载和组件更新**的时候都会调用这个函数

![react-extension-hook-1](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-extension-hook-1.gif)

它还可以接收第二个参数，这个参数表示它要**监测的数据**，也就是他要监视哪个数据的变化

当我们不需要监听任何状态变化的时候，我们可以就**传递一个空数组**，这样它就能当作 `componentMidMount` 来使用

```js
React.useEffect(() => {
    console.log('被调用了');
}, [])
```

这样我们只有在组件第一次挂载的时候触发

当然当页面中有多个数据源时，我们也可以选择个别的数据进行监测以达到我们想要的效果

```js
React.useEffect(() => {
    console.log('被调用了');
}, [count])
```

这样，我们就只**监视 count 数据的变化**

当我们想要在卸载一个组件之前进行一些**清除定时器**的操作，在类式组件中，我们会调用生命周期钩子 `componentDidUnmount` 来实现，在函数式组件中，我们的写法更为简单，我们直接在 `useEffect` 的第一个参数的返回值中实现即可
也就是说，第一个参数的函数体相当于 `componentDidMount` 返回体相当于 `componentDidUnmount` ，这样我们就能实现在组件即将被卸载时输出一些东西了

**实现卸载**

```js
function unmount() {
    ReactDOM.unmountComponentAtNode(document.getElementById("root"))
}
```

**卸载前输出**

```js
React.useEffect(() => {
    console.log('被调用了');
    return () => {
        console.log('我要被卸载了');
    }
}, [count])
```

![react-extension-hook-2](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-extension-hook-2.gif)

实现了在组件即将被卸载的时候输出

因此 `useEffect` 相当于三个生命周期钩子，`componentDidMount` 、`componentDidUpdata` 、`componentDidUnmount`

###  useRef

当我们想要获取组件内的信息时，在类式组件中，我们会采用 `ref` 的方式来获取。在函数式组件中，我们可以采用也可以采用 `ref` 但是，我们需要采用 `useRef` 函数来创建一个 ref 容器，这和 `createRef` 很类似。

```js
<input type="text" ref={myRef} />
```

获取 ref 值

```js
function show() {
    alert(myRef.current.value)
}
```

即可成功的获取到 input 框中的值

## 4. Fragment

我们编写组件的时候每次都需要采用一个 `div` 标签包裹，才能让它正常的编译，但是这样会引发什么问题呢？我们打开控制台看看它的层级

![image-20210911151643934](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210911151643934.png)

它包裹了几层无意义的 div 标签，我们可以采用 `Fragment` 来解决这个问题

首先，我们需要从 react 中暴露出 `Fragment` ，将我们所写的内容采用 `Fragment` 标签进行包裹，当它解析到 `Fragment` 标签的时候，就会把它去掉

![image-20210911152037120](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210911152037120.png)

这样我们的内容就直接挂在了 `root` 标签下

> 同时采用空标签，也能实现，但是它不能接收任何值，而 `Fragment` 能够接收 1 个值`key` 

## 5. Context

#### 仅适用于类式组件

当我们想要给子类的子类传递数据时，前面我们讲过了 redux 的做法，这里介绍的 Context 我觉得也类似于 Redux

首先我们需要引入一个 `MyContext` 组件，我们需要引用`MyContext` 下的 `Provider` 

```js
const MyContext = React.createContext();
const { Provider } = MyContext;
```

用 `Provider` 标签包裹 A组件内的 B 组件，并通过 `value` 值，将数据传递给子组件，这样以 A 组件为父代组件的所有子组件都能够接受到数据

```js
<Provider value={{ username, age }}>
    <B />
</Provider>
```

但是我们需要在使用数据的组件中引入 `MyContext` 

```js
static contextType = MyContext;
```

在使用时，直接从 `this.context` 上取值即可

```js
const {username,age} = this.context
```

#### 适用于函数和类式组件

由于函数式组件没有自己 `this` ，所以我们不能通过 `this.context` 来获取数据

这里我们需要从 `Context` 身上暴露出一个 `Consumer`

```js
const { Provider ,Consumer} = MyContext;
```

然后通过 `value` 取值即可

```js
function C() {
  return (
    <div>
      <h3>我是C组件，我从A接收到的数据 </h3>
      <Consumer>
        {(value) => {
          return `${value.username},年龄是${value.age}`;
        }}
      </Consumer>
    </div>
  );
}
```

![image-20210911161103300](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210911161103300.png)

> 因此想要在函数式组件中使用，需要引入 `Consumer`

## 6. PureComponent

在我们之前一直写的代码中，我们一直使用的`Component` 是有问题存在的

1. 只要执行 `setState` ，即使不改变状态数据，组件也会调用 `render`
2. 当前组件状态更新，也会引起子组件 `render`

而我们想要的是只有组件的 `state` 或者 `props` 数据发生改变的时候，再调用 `render`

我们可以采用重写 `shouldComponentUpdate` 的方法，但是这个方法不能根治这个问题，当状态很多时，我们没有办法增加判断

我们可以采用 `PureComponent` 

我们可以从 `react` 身上暴露出 `PureComponent` 而不使用 `Component` 

```js
import React, { PureComponent } from 'react'
```

就这~听了半天结果就只一个 `PureComponent` 

`PureComponent` 会对比当前对象和下一个状态的 `prop` 和 `state` ，而这个比较属于浅比较，比较基本数据类型是否相同，而对于引用数据类型，**比较的是它的引用地址是否相同，这个比较与内容无关**

## 7. render props

采用 render props 技术，我们可以像组件内部动态传入带有内容的结构

> 当我们在一个组件标签中填写内容时，这个内容会被定义为 children props，我们可以通过 `this.props.children` 来获取

例如：

```js
<A>hello</A>
```

这个 hello 我们就可以通过 children 来获取

而我们所说的 render props 就是在组件标签中传入一个 render 方法，又因为属于 props ，因而被叫做了 render props

```js
<A render={(name) => <C name={name} />} />
```

你可以把 `render` 看作是 `props`，只是它有特殊作用，当然它也可以用其他名字来命名

在上面的代码中，我们需要在 A 组件中预留出 C 组件渲染的位置 在需要的位置上加上`{this.props.render(name)}`

那我们在 C 组件中，如何接收 A 组件传递的 `name` 值呢？通过 `this.props.name` 的方式

## 8. ErrorBoundary

当不可控因素导致数据不正常时，我们不能直接将报错页面呈现在用户的面前，由于我们没有办法给每一个组件、每一个文件添加判断，来确保正常运行，这样很不现实，因此我们要用到**错误边界**技术

**错误边界就是让这块组件报错的影响降到最小，不要影响到其他组件或者全局的正常运行**

> 例如 A 组件报错了，我们可以在 A 组件内添加一小段的提示，并把错误控制在 A 组件内，不影响其他组件

- 我们要对容易出错的组件的父组件做手脚，而不是组件本身

我们在父组件中通过 ` getDerivedStateFromError` 来配置**子组件**出错时的处理函数

```js
static getDerivedStateFromError(error) {
    console.log(error);
    return { hasError: error }
}
```

我们可以将 `hasError` 配置到状态当中，当 `hasError` 状态改变成 `error` 时，表明有错误发生，我们需要在组件中通过判断 `hasError` 值，来指定是否显示子组件

```js
{this.state.hasError ? <h2>出错啦</h2> : <Child />}
```

> 在服务器中启动，才能正常看到效果

可以在 `componentDidCatch` 中统计错误次数，通知编码人员进行 bug 解决

## 9. 组件通信方式总结

1. props
   - children props
   - render props
2. 消息发布订阅
   - 利用 pubsub 库来实现
3. 集中式状态管理
   - redux
4. conText
   - 生成者-消费者

**选择方式**

父子组件采用：`props`

兄弟组件采用：消息的发布订阅、redux

祖孙组件：消息发布订阅、redux、context

