# React 入门学习（九）-- 消息订阅发布

![react-消息订阅](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-%E6%B6%88%E6%81%AF%E8%AE%A2%E9%98%85.gif)

> 📢 大家好，我是小丞同学，一名<font color=#2e86de>**大二的前端爱好者**</font>
>
> 📢 这篇文章是学习 React 中 GitHub 搜索案例的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 <font color=#f368e0>**愿你忠于自己，热爱生活**</font>

## 引言

在昨天写的 `Github` 案例中，我们采用的是 `axios` 发送请求来获取数据，同时我们需要将数据从 `Search` 中传入给 `App`，再由 `App` 组件再将数据传递给 `List` 组件，这个过程会显得多此一举。同时我们要将 `state` 状态存放在 `App` 组件当中，但是这些 `state` 状态都是在 `List` 组件中使用的，在 `Search` 组件中做的，只是更新这些数据，那这样也会显得很没有必要，我们完全可以将 `state` 状态存放在 `List` 组件中，但是这样我们又会遇到技术难题，兄弟组件间的数据通信。那这里我们就学习一下如何利用消息订阅发布来解决**兄弟组件间的通信**

## 消息发布订阅

要解决上面的问题，我们可以借助发布订阅的机制，我们可以将 App 文件中的所有状态和方法全部去除，因为本来就不是在 App 组件中直接使用这些方法的，App 组件只是一个中间媒介而已

我们先简单的说一下**消息订阅和发布的机制**

就拿我们平常订杂志来说，我们和出版社说我们要订一年的足球周刊，那每次有新的足球周刊，它都会寄来给你。

换到代码层面上，我们订阅了一个消息假设为 A，当另一个人发布了 A 消息时，因为我们订阅了消息 A ，那么我们就可以拿到 A 消息，并获取数据

那我们要怎么实现呢？

首先引入 `pubsub-js`

我们需要先安装这个库

```js
yarn add pubsub-js
```

引入这个库 

```js
import PubSub from 'pubsub-js'
```

订阅消息

我们通过 `subscribe` 来订阅消息，它接收两个参数，第一个参数是消息的名称，第二个是消息成功的回调，回调中也接受两个参数，一个是消息名称，一个是返回的数据

```js
PubSub.subscribe('search',(msg,data)=>{
  console.log(msg,data);
})
```

发布消息

我们采用 `publish` 来发布消息，用法如下

```js
PubSub.publish('search',{name:'tom',age:18})
```

有了这些基础，我们可以完善我们昨天写的 GitHub 案例

将数据的更新通过 `publish` 来传递，例如在发送请求之前，我们需要出现 loading 字样

```js
// 之前的写法
this.props.updateAppState({ isFirst: false, isLoading: true })
// 改为发布订阅方式
PubSub.publish('search',{ isFirst: false, isLoading: true })
```

这样我们就能成功的在请求之前发送消息，我们只需要在 List 组件中订阅一下这个消息即可，并将返回的数据用于更新状态即可

```js
PubSub.subscribe('search',(msg,stateObj)=>{
  this.setState(stateObj)
})
```

同时上面的代码会返回一个 `token` ，这个就类似于定时器的编号的存在，我们可以通过这个 `token` 值，来取消对应的订阅

通过 `unsubscribe` 来取消指定的订阅

```js
PubSub.unsubscribe(this.token)
```

## 扩展 -- Fetch

首先 fetch 也是一种发送请求的方式，它是在 xhr 之外的一种，我们平常用的 Jquery 和 axios 都是封装了 xhr 的第三方库，而 fetch 是官方自带的库，同时它也采用的是 Promise 的方式，大大简化了写法

如何使用呢？

```js
fetch('http://xxx')
  .then(response => response.json())
  .then(json => console.log(json))
  .catch(err => console.log('Request Failed', err)); 
```

它的使用方法和 axios 非常的类似，都是返回 Promise 对象，但是不同的是， fetch 关注分离，它在第一次请求时，不会直接返回数据，会先返回联系服务器的状态，在第二步中才能够获取到数据

我们需要在第一次 `then` 中返回 `response.json()` 因为这个是包含数据的 promise 对象，再调用一次 `then` 方法即可实现

但是这么多次的调用 `then` 并不是我们所期望的，相信看过之前生成器的文章的伙伴，已经有了想法。

我们可以利用 `async` 和 `await` 配合使用，来简化代码

可以将 `await` 理解成一个自动执行的 `then` 方法，这样清晰多了

```js

async function getJSON() {
  let url = 'https://xxx';
  try {
    let response = await fetch(url);
    return await reasponse.json();
  } catch (error) {
    console.log('Request Failed', error);
  }
}
```

最后关于错误对象的获取可以采用 `try...catch` 来实现

关于 fetch 的更多内容

强烈推荐阮一峰老师的博文：[fetch](http://www.ruanyifeng.com/blog/2020/12/fetch-tutorial.html)

---

>  非常感谢您的阅读，欢迎提出你的意见，有什么问题欢迎指出，谢谢！🎈

