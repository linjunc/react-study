# 🌮 React 入门学习（十一）-- React 路由传参

![React路由](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-%E8%B7%AF%E7%94%B1.gif)

> 📢 大家好，我是小丞同学，一名<font color=#2e86de>**大二的前端爱好者**</font>
>
> 📢 这篇文章是学习 React 中 React 路由的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 <font color=#f368e0>**愿你忠于自己，热爱生活**</font>

## 引言

在上一篇中，我们学习了 React 中使用路由技术，以及如何使用 `MyNavLink` 去优化使用路由时的代码冗余的情况。

这一节我们继续上一篇 React 路由进行一些补充

## 🍈 1. Switch 解决相同路径问题



首先我们看一段这样的代码

```js
<Route path="/home" component={Home}></Route>
<Route path="/about" component={About}></Route>
<Route path="/about" component={About}></Route>
```

这是两个路由组件，在2，3行中，我们同时使用了相同的路径 `/about` 

![image-20210903075753268](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210903075753268.png)

我们发现它出现了两个 `about` 组件的内容，那这是为什么呢？

其实是因为，`Route` 的机制，当匹配上了第一个 `/about` 组件后，它还会继续向下匹配，因此会出现两个 About 组件，这时我们可以采用 `Switch` 组件进行包裹

```js
<Switch>
    <Route path="/home" component={Home}></Route>
    <Route path="/about" component={About}></Route>
    <Route path="/about" component={About}></Route>
</Switch>
```

在使用 `Switch` 时，我们需要先从 `react-router-dom` 中暴露出 `Switch` 组件

这样我们就能成功的解决掉这个问题了

## 🥟 2. 解决二级路由样式丢失的问题

当我们将路径改写成 `path="/ljc/about"` 这样的形式时，我们会发现当我们强制刷新页面的时候，页面的 CSS 样式消失了。这是因为，我们在引入样式文件时，采取的是相对路径，当我们使用二级路由的时候，会使得请求的路径发生改变，浏览器会向 `localhost:3000/ljc` 下请求 css 样式资源，这并不是我们想要的，因为我们的样式存放于公共文件下的 CSS 文件夹中。

![react-router-tworouter](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-router-tworouter.gif)

我们有几种方法，可以解决这个问题 

1. 将样式引入的路径改成绝对路径
2. 引入样式文件时不带 `.`
3. 使用 HashRouter

我们一般采用**第一种方式**去解决

## 🍑 3. 路由的精准匹配和模糊匹配

路由的匹配有两种形式，一种是精准匹配一种是模糊匹配，React 中默认开启的是模糊匹配

模糊匹配可以理解为，在匹配路由时，只要有匹配到的就好了

精准匹配就是，两者必须相同

我们展示一个模糊匹配的例子

```js
<MyNavLink to = "/home/a/b" >Home</MyNavLink>
```

这个标签匹配的路由，我们可以拆分成 home a b，将会根据这个先后顺序匹配路由

```js
<Route path="/home"component={Home}/>
```

就可以匹配到上面的这个路由，因为它匹配的是 home

当匹配的路由改成下面这样时，就会失败。它会按照第一个来匹配，如果第一个没有匹配上，那就会失败，这里的 a 和 home 没有匹配上，很显然会失败

```js
<Route path="/a" component={Home}/>
```

当我们开启了精准匹配后，就我们的第一种匹配就不会成功，因为精准匹配需要的是完全一样的值，开启精准匹配采用的是 `exact` 来实现

```js
<Route exact={true}  path="/home" component={Home}/>
```

## 🍋 4. 重定向路由

在我们写好了这些之后，我们会发现，我们需要点击任意一个按钮，才会去匹配一个组件，这并不是我们想要的，我们想要页面一加载上来，默认的就能匹配到一个组件。

这个时候我们就需要时候 Redirecrt 进行默认匹配了。

```js
<Redirect to="/home" />
```

当我们加上这条语句时，页面找不到指定路径时，就会重定向到 `/home` 页面下因此当我们请求3000端口时，就会重定向到 `/home` 这样就能够实现我们想要的效果了

![image-20210904013342960](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210904013342960.png)

## 🍓 5. 嵌套路由

嵌套路由也就是我们前面有提及的二级路由，但是嵌套路由包括了二级、三级...还有很多级路由，当我们需要在一个路由组件中添加两个组件，一个是头部，一个是内容区

我们将我们的嵌套内容写在相应的组件里面，这个是在 Home 组件的 return 内容

```js
<div>
    <h2>Home组件内容</h2>
    <div>
        <ul className="nav nav-tabs">
            <li>
                <MyNavLink className="list-group-item" to="/home/news">News</MyNavLink>
            </li>
            <li>
                <MyNavLink className="list-group-item " to="/home/message">Message</MyNavLink>
            </li>
        </ul>
        {/* 注册路由 */}
        <Switch>
            <Route path="/home/news" component={News} />
            <Route path="/home/message" component={Message} />
        </Switch>
    </div>
</div>
```

在这里我们需要使用嵌套路由的方式，才能完成匹配

首先我们得 React 中路由得注册是有顺序得，我们在匹配得时候，因为 Home 组件是先注册得，因此在匹配的时候先去找 home 路由，由于是模糊匹配，会成功的匹配

在 Home 组件里面去匹配相应的路由，从而找到 /home/news 进行匹配，因此找到 News 组件，进行匹配渲染

> 如果开启精确匹配的话，第一步的 `/home/news` 匹配 `/home` 就会卡住不动，这个时候就不会显示有用的东西了！

## 🍟 6. 传递 params 参数

![react-router-params](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-router-params.gif)

首先我们需要实现的效果是，点击消息列表，展示出消息的详细内容

这个案例实现的方法有三种，第一种就是传递 params 参数，由于我们所显示的数据都是从数据集中取出来的，因此我们需要有数据的传输给 Detail 组件

我们首先需要将详细内容的数据列表，保存在 DetailData 中，将消息列表保存在 Message 的 state 中。

我们可以通过将数据拼接在路由地址末尾来实现数据的传递

```js
 <Link to={`/home/message/detail/${msgObj.id}/${msgObj.title}`}>{msgObj.title}</Link>
```

如上，我们将消息列表的 id 和 title 写在了路由地址后面

> 这里我们需要注意的是：需要采用模板字符串以及 `$` 符的方式来进行数据的获取

在注册路由时，我们可以通过 `:数据名` 来接收数据

```js
<Route path="/home/message/detail/:id/:title" component={Detail} />
```

如上，使用了 `:id/:title` 成功的接收了由 Link 传递过来的 id 和 title 数据

这样我们既成功的实现了路由的跳转，又将需要获取的数据传递给了 Detail 组件

我们在 Detail 组件中打印 `this.props` 来查看当前接收的数据情况

![image-20210906153042353](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210906153042353.png)

我们可以发现，我们传递的数据被接收到了对象的 match 属性下的 params 中

因此我们可以在 Detail 组件中获取到又 Message 组件中传递来的 params 数据

并通过 params 数据中的 `id` 值，在详细内容的数据集中查找出指定 `id` 的详细内容

```js
const { id, title } = this.props.match.params
const findResult = DetailData.find((detailObj) => {
    return detailObj.id === id
})
```

最后渲染数据即可

## 🍀 7. 传递 search 参数

我们还可以采用传递 search 参数的方法来实现

首先我们先确定数据传输的方式

我们先在 Link 中采用 `?` 符号的方式来表示后面的为可用数据

```js
<Link to={`/home/message/detail/?id=${msgObj.id}&title=${msgObj.title}`}>{msgObj.title}</Link>
```

采用 `search` 传递的方式，无需在 Route 中再次声明，可以在 Detail 组件中直接获取到

![image-20210906155217647](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210906155217647.png)

我们可以发现，我们的数据保存在了 `location` 对象下的 `search` 中，是一种字符串的形式保存的，我们可以引用一个库来进行转化 `querystring`

```js
import qs from 'querystring'
```

这个库是 React 中自带有的，它有两个方法，一个是 `parse` 一个是 `stringify` 

我们可以采用 `parse` 方法，将字符串转化为键值对形式的对象

```js
const { search } = this.props.location
const { id, title } = qs.parse(search.slice(1))
```

这样我们就能成功的获取数据，并进行渲染

> tips：无需声明接收

## 🌷 8. 传递 state 参数

采用传递 state 参数的方法，是我觉得最完美的一种方法，因为它不会将数据携带到地址栏上，采用内部的状态来维护

```js
<Link to={{ pathname: '/home/message/detail', state: { id: msgObj.id, title: msgObj.title } }}>{msgObj.title}</Link>
```

首先，我们需要在 Link 中注册跳转时，传递一个路由对象，包括一个 跳转地址名，一个 state 数据，这样我们就可以在 Detail 组件中获取到这个传递的 state 数据

> 注意：采用这种方式传递，无需声明接收

我们可以在 Detail 组件中的 location 对象下的 state 中取出我们所传递的数据

```js
const { id, title } = this.props.location.state
```

![image-20210906160940033](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210906160940033.png)

直接使用即可~

解决清除缓存造成报错的问题，我们可以在获取不到数据的时候用空对象来替代，例如，

```js
const { id, title } = this.props.location.state || {}
```

当获取不到 `state` 时，则用空对象代替

> 这里的 state 和状态里的 state 有所不同

---

> 非常感谢您的阅读，欢迎提出你的意见，有什么问题欢迎指出，谢谢！🎈

