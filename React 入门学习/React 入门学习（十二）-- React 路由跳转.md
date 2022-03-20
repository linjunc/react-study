# 🌮 React 入门学习（十二）-- React 路由跳转

![React路由](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-%E8%B7%AF%E7%94%B1.gif)

> 📢 大家好，我是小丞同学，一名<font color=#2e86de>**大二的前端爱好者**</font>
>
> 📢 这篇文章是学习 React 中 React 路由跳转的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 <font color=#f368e0>**愿你忠于自己，热爱生活**</font>

## 1. push 与 replace 模式

默认情况下，开启的是 push 模式，也就是说，每次点击跳转，都会向栈中压入一个新的地址，在点击返回时，可以返回到上一个打开的地址，

![react-router-push](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-router-push.gif)

就像上图一样，我们每次返回都会返回到上一次点击的地址中

当我们在读消息的时候，有时候我们可能会不喜欢这种繁琐的跳转，我们可以开启 replace 模式，这种模式与 push 模式不同，它会将当前地址**替换**成点击的地址，也就是替换了新的栈顶

我们只需要在需要开启的链接上加上 `replace` 即可

```js
<Link replace to={{ pathname: '/home/message/detail', state: { id: msgObj.id, title: msgObj.title } }}>{msgObj.title}</Link>
```

![react-router-replace](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-router-replace.gif)

## 2. 编程式路由导航

我们可以采用绑定事件的方式实现路由的跳转，我们在按钮上绑定一个 `onClick` 事件，当事件触发时，我们执行一个回调 `replaceShow` 

这个函数接收两个参数，用来仿制默认的跳转方式，第一个是点击的 id 第二个是标题

我们在回调中，调用 `this.props.location` 对象下的 replace 方法

```js
replaceShow = (id, title) => {
  this.props.history.replace(`/home/message/detail/${id}/${title}`)
}
```

同时我们可以借助 `this.props.history` 身上的 API 实现路由的跳转，例如 `go`、`goBack` 、`goForward`

## 3. withRouter

当我们需要在页面内部添加回退前进等按钮时，由于这些组件我们一般通过一般组件的方式去编写，因此我们会遇到一个问题，**无法获得 history 对象**，这正是因为我们采用的是一般组件造成的。

![image-20210906231051190](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210906231051190.png)

只有路由组件才能获取到 history 对象

因此我们需要如何解决这个问题呢

我们可以利用 `react-router-dom` 对象下的 `withRouter` 函数来对我们导出的 `Header` 组件进行包装，这样我们就能获得一个拥有 `history` 对象的一般组件

我们需要对哪个组件包装就在哪个组件下引入

```js
// Header/index.jsx
import { withRouter } from 'react-router-dom'
// 在最后导出对象时，用 `withRouter` 函数对 index 进行包装
export default withRouter(index);
```

这样就能让一般组件获得路由组件所特有的 API

## 4. BrowserRouter 和 HashRouter 的区别

#### **它们的底层实现原理不一样**

对于 BrowserRouter 来说它使用的是 React 为它封装的 history API ，这里的 history 和浏览器中的 history 有所不同噢！通过操作这些 API 来实现路由的保存等操作，但是这些 API 是 H5 中提出的，因此不兼容 IE9 以下版本。

对于 HashRouter 而言，它实现的原理是通过 URL 的哈希值，但是这句话我不是很理解，用一个简单的解释就是

我们可以理解为是锚点跳转，因为锚点跳转会保存历史记录，从而让 HashRouter 有了相关的前进后退操作，HashRouter 不会将 `#` 符号后面的内容请求。兼容性更好！

#### 地址栏的表现形式不一样

- HashRouter 的路径中包含 `#` ，例如 `localhost:3000/#/demo/test`

#### 刷新后路由 state 参数改变

1. 在BrowserRouter 中，state 保存在history 对象中，刷新不会丢失
2. HashRouter 则刷新会丢失 state



