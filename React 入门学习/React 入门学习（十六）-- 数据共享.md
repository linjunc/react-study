#  React 入门学习（十六）-- 数据共享

![数据共享](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/%E6%95%B0%E6%8D%AE%E5%85%B1%E4%BA%AB.gif)

> 📢 大家好，我是小丞同学，一名<font color=#2e86de>**大二的前端爱好者**</font>
>
> 📢 这篇文章是学习 React-Redux 数据共享 的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 <font color=#f368e0>**愿你忠于自己，热爱生活**</font>

## 引言

在写完了基本的 Redux 案例之后，我们可以尝试一些更实战性的操作，比如我们可以试试多组件间的状态传递，相互之间的交互

![react-redux-demo](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-redux-demo.gif)

如上动图所示，我们想要实现上面的案例，采用纯 React 来实现是比较困难的，我们需要**很多层的数据交换**才能实现，但是我们如果采用 Redux 来实现会变得非常简单

因为 Redux **打通了组件间的隔阂**，我们可以自由的进行数据交换，所有存放在 `store` 中的数据都可以实现共享，那我们接下来看看如何实现的吧~

##  1. 编写 Person 组件

> 上面的 Count 组件，已经在前面几篇写过了，但是我没有记录详细的实现过程，只是做了一些小小的总结（我摸鱼了）

不管如何，我们先来实现一个 Person 组件吧

首先我们需要在 `containers` 文件夹下编写 Person 组件的**容器组件**

如何编写一个容器组件呢？（上一篇也讲过了）

首先我们需要编写 `index.jsx` 文件，在这个文件里面编写 Person 组件的 **UI 组件**，并使用 `connect` 函数将它包装，**映射它的状态和方法**

#### **编写 UI 组件架构**

```js
<div>
    <h2>我是 Person 组件,上方组件求和为:{this.props.countAll}</h2>
    <input ref={c => this.nameNode = c} type="text" placeholder="输入名字" />
    <input ref={c => this.ageNode = c} type="text" placeholder="输入年龄" />
    <button onClick={this.addPerson}>添加</button>
    <ul>
        {
            this.props.persons.map((p) => {
                return <li key={p.id}> {p.name}--{p.age}</li>
            })
        }
    </ul>
</div>
```

我们可以看到这里采用了 `ref` 来获取到当前事件触发的节点，并通过 `this.addPerson` 的方式给按钮绑定了一个点击事件

#### **编写点击事件回调**

```js
addPerson = () => {
    const name = this.nameNode.value
    const age = this.ageNode.value
    const personObj = { id: nanoid(), name, age }
    this.props.add(personObj)
    this.nameNode.value = ''
    this.ageNode.value = ''
}
```

在这里我们需要处理输入框中的数据，并且将这些数据用于创建一个 `action` 对象，传递给 `store` 进行状态的更新

在这里我们需要回顾的是，这里我们使用了一个 `nanoid` 库，这个库我们之前也有使用过

##### **下载，引入，暴露**

```js
import { nanoid } from 'nanoid'
```

暴露的 `nanoid` 是一个函数，我们每一次调用时，都会返回一个不重复的数，用于确保 `id` 的唯一性，同时在后面的 `map` 遍历的过程中，我们将 `id` 作为了 `key` 值，这样也确保了 `key` 的唯一性，关于 `key` 的作用，可以看看 `diffing` 算法的文章

#### **状态管理**

在这里我们需要非常熟练的采用 `this.props.add` 的方式来更新状态

那么它是如何实现状态更新的呢？我们来看看

在我们调用 `connect` 函数时，我们第一次调用时传入的第二个参数，就是用于传递方法的，我们传递了一个 `add` 方法

```js
export default connect(
    state => ({ persons: state.person, countAll: state.count }),//映射状态
    { add: createAddPersonAction }
)(Person);
```

它的原词是：**mapDispatchToProps**

我的理解是，传入的东西会被映射映射成 `props` 对象下的方法，这也是我们能够在 `props` 下访问到 `add` 方法的原因

> 对于这一块 `connect` ，我们必须要能够形成自己的理解，这里非常的重要，它实现了数据的交互，不至于一个组件，而是全部组件

#### **我是如何理解的呢？**

> 想象一个 store 仓库，在我们这个案例当中，Count 组件需要存放 count 值在 store 中，Person 组件需要存放新增用户对象在 store 中，我们要把这两个数据存放在一个对象当中。当某个组件需要使用 store 中的值时，可以通过 connect 中的两个参数来获取，例如这里我们需要使用到 Count 组件的值，可以通过 `.count` 来从 store 中取值。

也就是说，所有的值都存放在 store 当中，通过点运算符来获取，所有的操作 store 的方法都需要通过 action 来实现。**当前组件需要使用的数据都需要在 `connect` 中暴露**

## 2. 编写 reducer

首先，我们需要明确 reducer 的作用，它是用来干什么的？

**根据操作类型来指定状态的更新**

也就是说当我们点击了**添加按钮**后，会将输入框中的数据整合成一个对象，作为当前 action 对象的 data 传递给 reducer

我们可以看看我们编写的 action 文件，和我们想的一样

```js
import { ADD_PERSON } from "../constant";
// 创建一个人的action 对象
export const createAddPersonAction = (personObj) => ({
  type: ADD_PERSON,
  data: personObj,
});
```

当 reducer 接收到 action 对象时，会对 type 进行判断

```js
export default function personReducer(preState = initState, action) {
  const { type, data } = action;
  switch (type) {
    case ADD_PERSON:
      return [data,...preState]
    default:
      return preState
  }
}
```

一般都采用 `switch` 来编写

**这里有个值得注意的地方是**，这个 `personReducer` 函数是一个纯函数，什么是纯函数呢？这个是高阶函数部分的知识了，纯**函数是一个不改变参数的函数，也就是说，传入的参数是不能被改变的。**

为什么要提这个呢？在我们 return 时，有时候会想通过**数组的 API** 来在数组前面塞一个值，不也可以吗？

但是我们要采用 `unshirt` 方法，这个方法是会改变原数组的，也就是我们传入的参数会被改变，因此这样的方法是不可行的！

## 3. 打通数据共享

写到这里，或许已经写完了，但是有些细节还是需要注意一下

采用 Redux 来进行组件的数据交互真的挺方便。

我们可以在 Count 组件中引入 Person 组件存在 store 中的状态。

```js
export default connect(state => ({ count: state.count, personNum: state.person.length }),
    {
       ...
    }
)(Count)
```

在这里我们将 store 中的 person 数组的长度暴露出来这样 Count 组件就可以直接通过 props 来使用了

同样的我们也可以在 Person 组件中使用 Count 组件的值

从而实现了我们的这个 Demo

## 4. 最终优化

1. 利用对象的简写方法，将键名和键值同名，从而只写一个名即可
2. 合并 reducer ，我们可以将多个 reducer文件 写在一个 index 文件当中，需要采用 `combineReducers` 来合并

## 5. 项目打包

执行 `npm run build` 命令，即可打包项目，打包完成后，会生成一个 `build` 文件，这个文件我们需要部署到服务器上才能运行

我们可以放在自己的服务器上即可

但是我遇到了一个问题

打包后的文件路径少了一个 `.` 导致文件无法找到，报错无法执行，我通过手动添加的方式解决了，不知道还有没有什么其他方法解决

![react-redux-demo](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-redux-demo.gif)

> 也可以采用 `npm i serve -g` 安装，如何通过 serve '指定文件夹' 来执行

---

> 非常感谢您的阅读，欢迎提出你的意见，有什么问题欢迎指出，谢谢！🎈

