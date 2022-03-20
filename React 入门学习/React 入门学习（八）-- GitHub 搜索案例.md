# React 入门学习（八）-- GitHub 搜索案例

![github搜索](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/github%E6%90%9C%E7%B4%A2.gif)

> 📢 大家好，我是小丞同学，一名<font color=#2e86de>**准大二的前端爱好者**</font>
>
> 📢 这篇文章是学习 React 中 GitHub 搜索案例的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 <font color=#f368e0>**愿你忠于自己，热爱生活**</font>

## 引言

本文主要介绍 React 学习中 Github 搜索案例，这个案例主要涉及到了 Axios 发送请求，数据渲染以及一些中间交替效果的实现

个人感觉在做完 TodoList 案例之后，这个案例会很轻松，只是多加了一个 Loading 效果的实现思路，以及一些小细节的完善，感觉练练手还是很不错的

## 一、实现静态组件

和之前的 TodoList 案例一样，我们需要先实现静态组件，在实现静态组件之前，我们还需要拆分组件，这个页面的组件，我们可以将它拆成以下两个组件，第一个组件是 `Search`，第二个是 `List` 

![image-20210828065604542](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210828065604542.png)

接下来我们需要将提前写好的静态页面，对应拆分到组件当中

注意：

1. class 需要改成 className
2. style 的值需要使用双花括号的形式

最重要的一点就是，`img` 标签，一定要**添加** `alt` 属性表示图片加载失败时的提示。

同时，`a` 标签要添加 `rel="noreferrer"`属性，不然会有大量的警告出现

![image-20210828070148865](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210828070148865.png)

## 二、axios 发送请求

在实现静态组件之后，我们需要通过向 `github` 发送请求，来获取相应的用户信息

但是由于短时间内多次请求，可能会导致请求不返回结果等情况发生，因此我们采用了一个事先搭建好的本地服务器

我们启动服务器，向这个地址发送请求即可

![image-20210828071053508](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210828071053508.png)

这个请求类型是 GET 请求，我们需要传递一个搜索的关键字，去请求数据

我们首先要获取到用户点击搜索按钮后**输入框中的值**

在需要触发事件的 `input` 标签中，添加 `ref` 属性

```js
 <input ref={c => this.keyWordElement = c} type="text" placeholder="输入关键词点击搜索" />
```

我们可以通过 `this.keyWordElement` 属性来获取到这个当前节点，也就是这个 `input` 框

我们再通过 `value` 值，即可获取到当前 `input` 框中的值

```js
// search 回调
const { keyWordElement: { value: keyWord } } = this
```

这里采用的是连续的解构赋值，最后将 `value` 改为 `keyWord` ，这样好辨别

获取到了 `keyWord` 值，接下来我们就需要发送请求了

```js
axios.get(`http://localhost:3000/api1/search/users?q=${keyWord}`).then(
    response => {
        this.props.updateAppState({ isLoading: false, users: response.data.items })
    },
    error => {
        this.props.updateAppState({ isLoading: false, err: error.message })
    }
)
```

 我们将 `keyWord` 接在请求地址的后面，来传递参数，以获得相关数据

这里会存在跨域的问题，因我我们是站在 3000 端口向 5000 端口发送请求的

因此我们需要配置代理来解决跨域的问题，我们需要在请求地址前，加上启用代理的标志 `/api1`

```js
// setupProxy.js
const proxy = require('http-proxy-middleware')
module.exports = function (app) {
    app.use(
        proxy('/api1', {
            target: 'http://localhost:5000',
            changeOrigin: true,
            pathRewrite: {
                '^/api1': ''
            }
        })
    )
}
```

这样我们就能成功的获取到了数据

![image-20210828072705747](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210828072705747.png)

## 三、渲染数据

在获取到了数据之后，我们需要对数据进行分析，并将这些数据渲染到页面上

比较重要的一点是，我们获取到的用户个数是动态的，因此我们需要通过遍历的方式去实现

同时我们的数据当前存在于 `Search` 组件当中，我们需要在 `List` 组件中使用，所以我们需要个 `Search` 组件传递一个函数，来实现子向父传递数据，再通过 `App` 组件，向`List` 组件传递数据即可得到 `data`

```js
users.map((userObj) => {
  return (
    <div key={userObj.id} className="card">
      <a rel="noreferrer" href={userObj.html_url} target="_blank">
        <img alt="avatar" src={userObj.avatar_url} style={{ width: '100px' }} />
      </a>
      <p className="card-text">{userObj.login}</p>
    </div>
  )
})
```

这里我们通过 `map` 遍历整个返回的数据，来循环的添加 `card` 的个数

同时将一些用户信息添加到其中

## 四、增加交互

做到这里其实已经完成了一大半了，但是似乎少了点交互

- 加载时的 loading 效果
- 第一次进入页面时 List 组件中的**欢迎使用字样**
- 在报错时应该提示错误信息

这一些都预示着我们不能单纯的将用户数据直接渲染，我们需要添加一些判断，什么时候该渲染数据，什么时候渲染 loading，什么时候渲染 err 

首先我们需要增加一些状态，来指示我们该渲染什么，比如

- 采用 `isFrist` 来判断页面是否第一次启动，初始值给 `true`，点击搜索后改为 `false`
- 采用 `isLoading` 来判断是否应该显示 Loading 动画，初始值给 `false`，在点击搜索后改为 `true`，在拿到数据后改为 `false`
- 采用 `err` 来判断是否渲染错误信息，当报错时填入报错信息，初始值**给空**

```js
state = { users: [], isFirst: true, isLoading: false, err: '' }
```

这样我们就需要改变我先前采用的数据传递方式，采用更新状态的方式，接收一个状态对象来**更新数据**，这样就不用去指定什么时候更新什么，就可以减少很多**不必要**的函数声明

同时在 App 组件给 List 组件传递数据时，我们可以采用解构赋值的方式，这样可以减少代码量

```js
// App.jsx
// 接收一个状态对象
updateAppState = (stateObj) => {
    this.setState(stateObj)
}
<Search updateAppState={this.updateAppState} />
<List {...this.state} />
```

这样我们只需要在 List 组件中，判断这些状态的值，来显示即可

```js
// List/index.jsx
// 对象解构
const { users, isFirst, isLoading, err } = this.props
// 判断
{
  isFirst ? <h2>欢迎使用，输入关键字，点击搜索</h2> :
    isLoading ? <h2>Loading...</h2> :
      err ? <h2 style={{ color: 'red' }}>{err}</h2> :
        users.map((userObj) => {
          return (
           // 渲染数据块
           //为了减少代码量，就不贴了
          )
        })
}
```

我们需要先判断是否第一次，再判断是不是正在加载，再判断有没有报错，最后再渲染数据

我们的状态更新是在 Search 组件中实现的，在点击搜索之后数据返回之前，我们需要将 `isFirst` 改为 `false` ，`isLoading` 改为 `true` 

接收到数据后我们再将 `isLoading` 改为 `false` 即可

以上就是 Github 搜索案例的实现过程

![react-github](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-github.gif)

最终效果图

---

> 前端路还有很长，今天我就大二啦！加油吧！！！

> 非常感谢您的阅读，欢迎提出你的意见，有什么问题欢迎指出，谢谢！🎈

