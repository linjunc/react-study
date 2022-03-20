# React 入门（三） -- 生命周期 LifeCycle

![组件的生命周期](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/%E7%BB%84%E4%BB%B6%E7%9A%84%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F.png)

> 📢 大家好，我是小丞同学，这一篇是关于 React 的学习笔记，关于组件的生命周期
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 愿你生活明朗，万物可爱

## 引言

在 React 中为我们提供了一些生命周期钩子函数，让我们能在 React 执行的重要阶段，在钩子函数中做一些事情。那么在 React 的生命周期中，有哪些钩子函数呢，我们来总结一下

## React 生命周期

React 生命周期主要包括三个阶段：初始化阶段，更新阶段，销毁阶段

### 初始化阶段

#### 1. constructor 执行

`constructor` 在组件初始化的时候只会执行一次

通常它用于做这两件事

1. 初始化函数内部 `state`
2. 绑定函数

```js
constructor(props) {
    console.log('进入构造器');
    super(props)
    this.state = { count: 0 }
}
```

现在我们通常不会使用 `constructor` 属性，而是改用类加箭头函数的方法，来替代 `constructor` 

例如，我们可以这样初始化 `state`

```js
state = {
	count: 0
};
```

#### 2. static getDerivedStateFromProps 执行 （新钩子）

这个是 React 新版本中新增的2个钩子之一，据说很少用。

`getDerivedStateFromProps` 在初始化和更新中都会被调用，并且在 `render` 方法之前调用，它返回一个对象用来更新 `state`

`getDerivedStateFromProps` 是类上直接绑定的静态（`static`）方法，它接收两个参数 `props` 和 `state`

`props` 是即将要替代 `state` 的值，而 `state` 是当前未替代前的值

> 注意：`state` 的值在任何时候都取决于传入的 `props` ，不会再改变

如下

```js
static getDerivedStateFromProps(props) {
    return props
}
ReactDOM.render(<Count count="109"/>,document.querySelector('.test'))
```

`count` 的值不会改变，一直是 109

#### 2. componentWillMount 执行（即将废弃）

> 如果存在 `getDerivedStateFromProps` 和 `getSnapshotBeforeUpdate` 就不会执行生命周期`componentWillMount`。

该方法只在挂载的时候调用一次，表示组件将要被挂载，并且在 `render` 方法之前调用。

这个方法在 React 18版本中将要被废弃，官方解释是在 React 异步机制下，如果滥用这个钩子可能会有 Bug

#### 3. render 执行

`render()` 方法是组件中必须实现的方法，用于渲染 DOM ，但是它不会真正的操作 DOM，它的作用是把需要的东西返回出去。

实现渲染 DOM 操作的是 `ReactDOM.render()`

> 注意：避免在 `render` 中使用 `setState` ，否则会死循环

#### 4. componentDidMount 执行

`componentDidMount` 的执行意味着初始化挂载操作已经基本完成，它主要用于组件挂载完成后做某些操作

这个挂载完成指的是：组件插入 DOM tree 

#### 初始化阶段总结

执行顺序 `constructor` -> `getDerivedStateFromProps` 或者 `componentWillMount` -> `render` -> `componentDidMount`

![image-20210821102153009](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210821102153009.png)

### 更新阶段

![image-20210821102622645](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210821102622645.png)

这里记录新生命周期的流程

#### 1. getDerivedStateFromProps 执行

执行生命周期`getDerivedStateFromProps`， 返回的值用于合并 `state`，生成新的`state`。

#### 2. shouldComponentUpdat 执行

`shouldComponentUpdate()` 在组件更新之前调用，可以通过返回值来控制组件是否更新，允许更新返回 `true` ，反之不更新

#### 3. render 执行

在控制是否更新的函数中，如果返回 `true` 才会执行 `render` ,得到最新的 `React element`

#### 4. getSnapshotBeforeUpdate 执行

在最近一次的渲染输出之前被提交之前调用，也就是即将挂载时调用

相当于淘宝购物的快照，会保留下单前的商品内容，在 React 中就相当于是 即将更新前的状态

> 它可以使组件在 DOM 真正更新之前捕获一些信息（例如滚动位置），此生命周期返回的任何值都会作为参数传递给 `componentDidUpdate()`。如不需要传递任何值，那么请返回 null

#### 5. componentDidUpdate 执行

组件在更新完毕后会立即被调用，首次渲染不会调用

---

到此更新阶段就结束了，在 React 旧版本中有两个与更新有关的钩子函数 `componentWillReceiveProps` 和 `componentWillUpdate` 都即将废弃

`componentWillReceiveProps` 我不太懂

`componentWillUpdate` 在 `render` 之前执行，表示组件将要更新

### 销毁阶段

#### componentWillUnmount  执行

在组件即将被卸载或销毁时进行调用。

## 总结

**初始化**

- constructor()
- static getDerivedStateFromProps()
- render()
- componentDidMount()

**更新**

- static getDerivedStateFromProps()
- shouldComponentUpdate()
- render()
- getSnapshotBeforeUpdate()
- componentDidUpdate()

**销毁**

- componentWillUnmount()

---

> 初学 React ，对生命周期还没有深入的理解，只能大概知道在什么时候触发哪个钩子，希望各位大佬多多指教，有什么建议可以提一提 🙏

