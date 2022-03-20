# React 入门学习（六）-- TodoList 案例 

![React-todolist](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/React-todolist.png)

> 📢 大家好，我是小丞同学，一名<font color=#2e86de>**准大二的前端爱好者**</font>
>
> 📢 这篇文章是学习 React 练习中 TodoList 案例的操作笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 <font color=#f368e0>**愿你忠于自己，热爱生活**</font>

## 引言

TodoList 案例在前端学习中挺重要的，从原生 JavaScript 的增删查改，到现在 React 的组件通信，都是一个不错的案例，这篇文章主要记录，还原一下通过 React 实现 TodoList 的全过程

![image-20210826091013929](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210826091013929.png)

## 一、拆分组件

首先第一步需要做的是将这个页面拆分成几个组件

首先顶部的输入框，可以完成添加项目的功能，可以拆分成一个 **Header 组件**

中间部分可以实现一个渲染列表的功能，可以拆分成一个 **List 组件**

在这部分里面，每一个待办事项都可以拆分成一个 **Item 组件**

最后底部显示当前完成状态的部分，可以拆分成一个 **Footer 组件**

![image-20210826092737826](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210826092737826.png)

在拆分完组件后，我们下一步要做的就是去实现这些组件的静态效果

## 二、实现静态组件

首先，我们可以先写好这个页面的静态页面，然后再分离组件，所以这就要求我们

以后写静态页面的时候，一定要有明确的规范

1. 打好注释
2. 每个部分的 CSS 要写在一个地方，不要随意写
3. 命名一定要规范
4. CSS 选择器不要关联太多层级
5. 在写 HTML 时就要划分好布局

这样有利于我们分离组件

首先，我们在 `src` 目录下，新建一个 `Components` 文件夹，用于存放我们的组件，然后在文件夹下，新建 `Header` 、`Item`、`List` 、`Footer` 组件文件夹，再创建其下的 `index.jsx`，`index.css` 文件，用于创建对应组件及其样式文件

```markdown
todolist
├─ package.json
├─ public
│  ├─ favicon.ico
│  └─ index.html
├─ src
│  ├─ App.css
│  ├─ App.jsx
│  ├─ Components
│  │  ├─ Footer
│  │  │  ├─ index.css
│  │  │  └─ index.jsx
│  │  ├─ Header
│  │  │  ├─ index.css
│  │  │  └─ index.jsx
│  │  ├─ item
│  │  │  ├─ index.css
│  │  │  └─ index.jsx
│  │  └─ List
│  │     ├─ index.css
│  │     └─ index.jsx
│  └─ index.js
└─ yarn.lock
```

最终目录结构如上

然后我们将每个组件，对应的 HTML 结构 CV 到对应组件的 `index.jsx` 文件中 `return` 出来，再将 CSS 样式添加到 `index.css` 文件中

**记得**，在 `index.jsx` 中一定要引入 `index.css` 文件

实现了静态组件后，我们需要添加事件等，来实现动态组件

## 三、实现动态组件

### 🍎 1. 动态展示列表

我们目前实现的列表项是固定的，我们需要它通过**状态**来维护，而不是通过**组件标签**来维护

首先我们知道，父子之间传递参数，可以通过 `state` 和 `props` 实现

我们通过在父组件也就是 `App.jsx` 中设置状态

![image-20210826103418053](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210826103418053.png)

再将它传递给对应的渲染组件 `List` 

```jsx
const { todos } = this.state
<List todos={todos}/>
```

这样在 `List` 组件中就能通过 `props` 来获取到 `todos` 

我们通过解构取出 `todos`

```jsx
const { todos, updateTodo } = this.props
```

再通过 `map` 遍历渲染 `Item` 数量

```js
{
  todos.map(todo => {
    return <Item key={todo.id} {...todo}/>
  })
}
```

同时由于我们的数据渲染最终是在 `Item` 组件中完成的，所以我们需要将数据传递给 `Item` 组件

这里有两个注意点

1. 关于 `key` 的作用在 diff 算法的文章中已经有讲过了，需要满足**唯一性**
2. 这里采用了简写形式 `{...todo}` ，这使得代码更加简洁，它代表的意思是 

```js
id = {todo.id} name = {todo.name} done = {todo.done}
```

在 `Item` 组件中取出 `props` 即可使用

```js
const { id, name, done } = this.props
```

这样我们更改 `APP.jsx` 文件中的 `state` 就能驱动着 `Item` 组件的更新，如图

![react-todolist-1](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-todolist-1.gif)

同时这里需要注意的是

对于复选框的选中状态，这里采用的是 `defaultChecked = {done}`，相比于 `checked` 属性，这个设定的是默认值，能够更改

### 🍍 2. 添加事项功能

首先我们需要在 Header 组件中，绑定键盘事件，判断按下的是否为回车，如果为回车，则将当前输入框中的内容传递给 APP 组件

> 因为，在目前的学习知识中，Header 组件和渲染组件 List 属于兄弟组件，没有办法进行直接的数据传递，因此可以将数据传递给 APP 再由 APP 转发给 List。

```jsx
// Header/index.jsx
handleKeyUp = (event) => {
  // 结构赋值获取 keyCode,target
  const { keyCode, target } = event
  // 判断是不是回车
  if (keyCode !== 13) return
  if(target.value.trim() === '') {
    alert('输入不能为空')
  }
  // 准备一个todo对象
  const todoObj = { id: nanoid(), name: target.value, done: false }
  // 传递给app
  this.props.addTodo(todoObj)
  // 清空
  target.value = ''
}
```

我们在 `App.jsx` 中添加了事件 `addTodo` ，这样可以将 Header 组件传递的参数，维护到 `App` 的状态中

```jsx
// App.jsx
addTodo = (todoObj) => {
  const { todos } = this.state
  // 追加一个 todo
  const newTodos = [todoObj, ...todos]
  this.setState({ todos: newTodos })
}
```

在这小部分中，需要我们注意的是，我们新建的 `todo`  对象，一定要保证它的 `id` 的唯一性

这里采用的 `nanoid` 库，这个库的每一次调用都会返回一个唯一的值

```shell
npm i nanoid
```

安装这个库，然后引入

通过 `nanoid()` 即可生成唯一值

![react-todolist-addtodo](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-todolist-addtodo.gif)

### 🍋 3. 实现鼠标悬浮效果

接下来我们需要实现每个 `Item` 中的小功能

首先是鼠标移入时的变色效果

我的逻辑是，通过一个状态来维护是否鼠标移入，比如用一个 `mouse` 变量，值给 `false` 当鼠标移入时，重新设定状态为 `true` 当鼠标移出时设为 `false` ，然后我们只需要在 `style` 中用`mouse` 去设定样式即可

下面我们来代码实现

在 `Item` 组件中，先设定状态

```jsx
state = { mouse: false } // 标识鼠标移入，移出
```

给元素绑定上鼠标移入，移出事件

```js
<li onMouseEnter={this.handleMouse(true)} onMouseLeave={this.handleMouse(false)} ><li/>
```

当鼠标移入时，会触发 `onMouseEnter` 事件，调用 `handleMouse` 事件传入参数 `true` 表示鼠标进入，更新组件状态

```js
handleMouse = flag => {
    return () => {
        this.setState({ mouse: flag })
    }
}
```

再在 `li` 身上添加由 `mouse` 控制的背景颜色

```js
style={{ backgroundColor: this.state.mouse ? '#ddd' : 'white' }}
```

同时通过 `mouse` 来控制删除按钮的显示和隐藏，做法和上面一样

![react-todolist-mouse](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-todolist-mouse.gif)

观察 mouse 的变化

### 🍉 4. 复选框状态维护

我们需要将当前复选框的状态，维护到 `state`  当中

我们的思路是

在复选框中添加一个 `onChange`  事件来进行数据的传递，当事件触发时我们执行 `handleCheck` 函数，这个函数可以向 App 组件中传递参数，这样再在 App 中改变状态即可

首先绑定事件

```js
// Item/index.jsx
<input type="checkbox" defaultChecked={done} onChange={this.handleCheck(id)} />
```

事件回调

```jsx
handleCheck = (id) => {
    return (event) => {
        this.props.updateTodo(id, event.target.checked)
    }
}
```

由于我们需要传递 `id` 来记录状态更新的对象，因此我们需要采用高阶函数的写法，不然函数会直接执行而报错，复选框的状态我们可以通过 `event.target.checked` 来获取

这样我们将我们需要改变状态的 `Item` 的 `id` 和改变后的状态，传递给了 App

内定义的`updateTodo` 事件，这样我们可以在 App 组件中操作改变状态

我们传递了两个参数 `id` 和 `done`

通过遍历找出该 `id` 对应的 `todo` 对象，更改它的 `done` 即可

```js
// App.jsx
updateTodo = (id, done) => {
  const { todos } = this.state
  // 处理
  const newTodos = todos.map(todoObj => {
    if (todoObj.id === id) {
      return { ...todoObj, done }
    } else {
      return todoObj
    }
  })
  this.setState({ todos: newTodos })
}
```

这里更改的方式是 `{ ...todoObj, done }`，首先会展开 `todoObj` 的每一项，再对 `done` 属性做覆盖

![react-todolist-update](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-todolist-update.gif)



### 🍏 5. 限制参数类型

在我们前面写的东西中，我们并没有对参数的**类型以及必要性**进行限制

在前面我们也学过这个，我们需要借助 `propTypes` 这个库

首先我们需要引入这个库，然后对 `props` 进行限制

```js
// Header
static propTypes = {
  addTodo: PropTypes.func.isRequired
}
```

在Header 组件中需要接收一个 `addTodo` 函数，所以我们进行一下限制

同时在 List 组件中也需要进行对 `todos` 以及 `updateTodo` 的限制

如果传入的参数不符合限制，则会报 **warning**

### 🍒 6. 删除按钮

现在我们需要实现删除按钮的效果

这个和前面的挺像的，首先我们分析一下，我们需要在 `Item` 组件上的按钮绑定点击事件，然后传入被点击事项的 `id` 值，通过 `props` 将它传递给父元素 `List` ，再通过在 `List` 中绑定一个 `App` 组件中的删除回调，将 `id` 传递给 `App` 来改变 `state`

首先我们先编写 点击事件

```js
// Item/index.jsx
handleDelete = (id) => {
    this.props.deleteTodo(id)
}
```

绑定在点击事件的回调上

子组件想影响父组件的状态，需要父组件传递一个函数，因此我们在 `App` 中添加一个 `deleteTodo` 函数

```js
// app.jsx
deleteTodo = (id) => {
  const { todos } = this.state
  const newTodos = todos.filter(todoObj => {
    return todoObj.id !== id
  })
  this.setState({ todos: newTodos })
}
```

然后将这个函数传递给 List 组件，再传递给 Item

增加一个判断

```js
if(window.confirm('确认删除')) {
    this.props.deleteTodo(id)
}
```

![react-todolist-detele](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-todolist-detele.gif)

### 🍓 7. 获取完成数量

我们在 App 中向 `Footer` 组件传递 `todos` 数据，再去统计数据

统计 `done `为 `true` 的个数

```js
const doneCount = todos.reduce((pre, todo) => {
    return pre + (todo.done ? 1 : 0)
}, 0)
```

再渲染数据即可

![image-20210826160505813](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210826160505813.png)

### 🍊 8. 全选按钮

首先我们需要在按钮上绑定事件，由于子组件需要改变父组件的状态，所以我们的操作和之前的一样，先绑定事件，再在 App 中传一个函数个 Footer ，再在 Footer 中调用这个函数并传入参数即可

这里需要特别注意的是

`defaulChecked` 只有第一次会起作用，所以我们需要将前面写的改成 `checked` 添加 `onChange` 事件即可

首先我们先在 App 中给 Footer 传入一个函数 `checkAllTodo` 

```js
// App.jsx
checkAllTodo = (done) => {
  const { todos } = this.state
  const newTodos = todos.map((todoObj => {
    return { ...todoObj, done: done }
  }))
  this.setState({ todos: newTodos })
}
// render
 <Footer todos={todos} checkAllTodo={this.checkAllTodo}/>

```

然后我们需要在 Footer 中调用一下

```js
handleCheckAll = (event) => {
    this.props.checkAllTodo(event.target.checked)
}
```

这里我们传入了一个参数：当前按钮的状态，用于全选和取消全选

同时我们需要排除总数为0 时的干扰

```js
<input type="checkbox" checked={doneCount === total && total !== 0? true : false} onChange={this.handleCheckAll} />
```



![react-todolist-all](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-todolist-all.gif)

### 🥭 9. 删除已完成

给删除按钮添加一个点击事件，回调中调用 App 中添加的删除已完成的函数，全都一个套路

**强烈建议这个自己打**

首先在 Footer 组件中调用传来的函数，在 App 中定义函数，过滤掉  `done` 为 `true` 的，再更新状态即可

```js
// App.jsx
clearAllDone = () => {
  const { todos } = this.state
  const newTodos = todos.filter((todoObj) => {
    return todoObj.done !== true
  })
  this.setState({ todos: newTodos })
}
```

![react-todolist-clear](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/react-todolist-clear.gif)



## 总结

1. 注意：className、style 写法
2. 父组件给子组件传递数据，采用 `props`
3. 子组件给父组件传递数据，通过 `props`，同时提前给子组件传递一个函数
4. 注意 `defaultChecked` 和 `checked` 的区别
5. 一定要自己敲一下，好好理解数据传递

> 非常感谢您的阅读，欢迎提出你的意见，有什么问题欢迎指出，谢谢！🎈
