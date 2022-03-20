# 🌮 React 入门学习（十）-- React 路由

![React路由](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-%E8%B7%AF%E7%94%B1.gif)

> 📢 大家好，我是小丞同学，一名<font color=#2e86de>**大二的前端爱好者**</font>
>
> 📢 这篇文章是学习 React 中 React 路由的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 <font color=#f368e0>**愿你忠于自己，热爱生活**</font>

## 引言

在我们之前写的页面当中，用我们的惯用思维去思考的话，可能会需要写很多的页面，例如做一个 tab 栏，我们可能会想每个选项都要对应一个 HTML 文件，这样会很麻烦，甚至不友好，我们把这种称为 MPA 也叫多页面应用。

## 🍕 1. SPA

而为了减少这样的情况，我们还有另一种应用，叫做 SPA ，单页应用程序

它比传统的 Web 应用程序更快，因为它们在 Web 浏览器本身而不是在服务器上执行逻辑。在初始页面加载后，**只有数据来回发送**，而不是整个 HTML，这会降低带宽。它们可以独立请求标记和数据，并直接在浏览器中呈现页面

## 🍔 2. 什么是路由？

路由是根据不同的 URL 地址展示不同的内容或页面

在 SPA 应用中，大部分页面结果不改变，只改变部分内容的使用

**前端路由的优缺点**

**优点**

用户体验好，不需要每次都从服务器全部获取整个 HTML，快速展现给用户

**缺点**

1. SPA 无法记住之前页面滚动的位置，再次回到页面时无法记住滚动的位置
2. 使用浏览器的前进和后退键会重新请求，没有合理利用缓存

## 🍟 3. 路由的原理

前端路由的主要依靠的时 history ，也就是浏览器的历史记录

> history 是 BOM 对象下的一个属性，在 H5 中新增了一些操作 history 的 API

浏览器的历史记录就类似于一个栈的数据结构，前进就相当于入栈，后退就相当于出栈

并且历史记录上可以采用 `listen` 来监听请求路由的改变，从而判断是否改变路径

在 H5 中新增了 `createBrowserHistory` 的 API ，用于创建一个 history 栈，允许我们手动操作浏览器的历史记录

新增 API：`pushState` ，`replaceState`，原理类似于 Hash 实现。 用 H5 实现，单页路由的 URL 不会多出一个 `#` 号，这样会更加的美观

## 🌭 4. 路由的基本使用

### react-router-dom 的理解和使用

> 专门给 web 人员使用的库

1. 一个 react 的仓库
2. 很常用，基本是每个应用都会使用的这个库
3. 专门来实现 SPA 应用

首先我们要明确好页面的布局 ，分好导航区、展示区

要引入 `react-router-dom` 库，暴露一些属性 `Link、BrowserRouter...`

```js
import { Link, BrowserRouter, Route } from 'react-router-dom'
```

导航区的 a 标签改为 Link 标签

```js
<Link className="list-group-item" to="/about">About</Link>
```

同时我们需要用 `Route` 标签，来进行路径的匹配，从而实现不同路径的组件切换

```js
<Route path="/about" component={About}></Route>
<Route path="/home" component={Home}></Route>
```

这样之后我们还需要一步，加个路由器，在上面我们写了两组路由，同时还会报错指示我们需要添加 `Router` 来解决错误，这就是需要我们添加路由器来管理路由，如果我们在 Link 和 Route 中分别用路由器管理，那这样是实现不了的，只有在一个路由器的管理下才能进行页面的跳转工作。

因此我们也可以在 Link 和 Route 标签的外层标签采用 BrowserRouter 包裹，但是这样当我们的路由过多时，我们要不停的更改标签包裹的位置，因此我们可以这么做

我们回到 App.jsx 目录下的 index.js 文件，将整个 App 组件标签采用 `BrowserRouter` 标签去包裹，这样整个 App 组件都在**一个路由器**的管理下

```js
// index.js
<BrowserRouter>
< App />
</BrowserRouter>
```

![react-router](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-router.gif)

## 🍿 5. 路由组件和一般组件

在我们前面的内容中，我们是把组件 Home 和组件 About 当成是一般组件来使用，我们将它们写在了 src 目录下的 components 文件夹下，但是我们又会发现它和普通的组件又有点不同，对于普通组件而言，我们在引入它们的时候我们是通过标签的形式来引用的。但是在上面我们可以看到，我们把它当作路由来引用时，我们是通过 `{Home}` 来引用的。

从这一点我们就可以认定一般组件和路由组件存在着差异

首先它们的写法不同

**一般组件**：`<Demo/>`，**路由组件**：`<Route path="/demo" component={Demo}/>`

同时为了规范我们的书写，一般将路由组件放在 `pages` 文件夹中，路由组件放在 `components` 

而最重要的一点就是它们接收到的 `props` 不同，在一般组件中，如果我们不进行传递，就不会收到值。而对于路由组件而言，它会接收到 3 个固定属性 `history` 、`location` 以及 `match` 

![image-20210901142121047](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210901142121047.png)

## 🍛 6. NavLink 标签

NavLink 标签是和 Link 标签作用相同的，但是它又比 Link 更加强大。

在前面的 demo 展示中，你可能会发现点击的按钮并没有出现高亮的效果，正常情况下我们给标签多添加一个 `active`  的类就可以实现高亮的效果

而 NavLink 标签正可以帮助我们实现这一步

当我们选中某个 NavLink 标签时，就会自动的在类上添加一个 `active` 属性

```js
<NavLink className="list-group-item" to="/about">About</NavLink>
```

![react-router-navlink](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-router-navlink.gif)

我们可以看到左侧的元素类名在不断的切换，当然 NavLink 标签是默认的添加上 `active` 类，我们也可以改变它，在标签上添加一个属性 `activeClassName` 

例如 `activeClassName="aaa"` 在触发这个 NavLink 时，会自动添加一个 `aaa` 类

## 🥩 7. NavLink 封装

在上面的 NavLink 标签种，我们可以发现我们每次都需要重复的去写这些样式名称或者是 `activeClassName` ，这并不是一个很好的情况，代码过于冗余。那我们是不是可以想想办法封装一下它们呢？

我们可以采用 `MyNavLink` 组件，对 NavLink 进行封装

首先我们需要新建一个 MyNavLink 组件

`return` 一个结构

```js
<NavLink className="list-group-item" {...this.props} />
```

首先，有一点非常重要的是，我们在标签体内写的内容都会成为一个 `children` 属性，因此我们在调用 `MyNavLink` 时，在标签体中写的内容，都会成为 `props` 中的一部分，从而能够实现

接下来我们在调用时，直接写

```js
<MyNavLink to="/home">home</MyNavLink>
```

即可实现相同的效果

---

以上就是本节关于 React 路由的相关知识！

> 非常感谢您的阅读，欢迎提出你的意见，有什么问题欢迎指出，谢谢！🎈