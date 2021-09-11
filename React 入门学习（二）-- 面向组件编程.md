# React 从入门到入土（二）-- 面向组件编程

![组件2](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/%E7%BB%84%E4%BB%B62.png)

> 📢 大家好😪 ，我是小丞同学，最近在学习 React、小程序、阅读 JS 高程，以及整理 Node 的笔记，这是关于 React 的**第二篇**文章，也是我学习的第一个框架，内容如有错误，欢迎大家指正
>
> 📢 <font color=#e84393>愿你生活明朗，万物可爱</font>

## 一、组件的使用

当应用是以多组件的方式实现，这个应用就是一个组件化的应用

> **注意：** 
>
> 1. 组件名必须是首字母大写
>
> 2. 虚拟DOM元素只能有一个根元素
>
> 3. 虚拟DOM元素必须有结束标签 `< />`

**渲染类组件标签的基本流程**

1. React 内部会创建组件实例对象

2. 调用`render()`得到虚拟 DOM ,并解析为真实 DOM

3. 插入到指定的页面元素内部

### 1. 函数式组件

```js
//1.先创建函数，函数可以有参数，也可以没有，但是必须要有返回值 返回一个虚拟DOM
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
//2.进行渲染
ReactDOM.Render(<Welcom name = "ljc" />,document.getElementById("div"));
```

上面的代码经历了以下几步

1. 我们调用 `ReactDOM.render()` 函数，并传入 `<Welcome name="ljc" />` 作为参数。
2. React 调用 `Welcome` 组件，并将 `{name: 'ljc'}` 作为 props 传入。
3. `Welcome` 组件将 `Hello, ljc` 元素作为返回值。
4. React DOM 将 DOM 高效地更新为 `Hello,ljc`。

### 2. 类式组件

![weather](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/weather.gif)

```js
class MyComponent extends React.Component {
    state = {isHot:false}
    render() {
        const {isHot} = this.state
        return <h1 onClick={this.changeWeather}>今天天气很{isHot?'炎热':'凉爽'}</h1>
    }
    changeWeather = ()=>{
        const isHot = this.state.isHot
        this.setState({isHot:!isHot})
    }
}
ReactDOM.render(<MyComponent/>,document.querySelector('.test'))
```

这玩意底层不简单，`this`的指向真的需要好好学习

**在优化过程中遇到的问题**

1. 组件中的 render 方法中的 this 为组件实例对象
2. 组件自定义方法中由于开启了严格模式，this 指向 `undefined` 如何解决
   1. 通过 bind 改变 this 指向
   2. 推荐采用箭头函数，箭头函数的 `this` 指向
3. state 数据不能直接修改或者更新

### 3. 其他知识

包含表单元素的组件分为非受控租价与受控组件

- **受控组件**：表单组件的输入组件随着输入并将内容存储到状态中（随时更新）
- **非受控组件**：表单组件的输入组件的内容在有需求的时候才存储到状态中（即用即取）

## 二、组件实例三大属性

### 1. state

> React 把组件看成是一个状态机（State Machines）。通过与用户的交互，实现不同状态，然后渲染 UI，让用户界面和数据保持一致。
>
> React 里，只需更新组件的 state，然后根据新的 state 重新渲染用户界面（不要操作 DOM）。

简单的说就是组件的状态，也就是该组件所存储的数据

**类式组件中的使用**

![image-20210720203721926](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210720203721926.png)

使用的时候通过`this.state`调用`state`里的值

在类式组件中定义`state`

- 在构造器中初始化`state`
- 在类中添加属性`state`来初始化

**修改 state**

在**类式组件**的函数中，直接修改`state`值

```js
this.state.weather = '凉爽'
```

> 页面的渲染靠的是`render`函数

这时候会发现页面内容不会改变，原因是 React 中不建议 `state`不允许直接修改，而是通过类的原型对象上的方法 `setState()`

**setState()**

```js
this.setState(partialState, [callback]);
```

- `partialState`: 需要更新的状态的部分对象
- `callback`: 更新完状态后的回调函数

有两种写法：写法1

```js
this.setState({
    weather: "凉爽"
})
```

写法2：

```js
// 传入一个函数，返回x需要修改成的对象，参数为当前的 state
this.setState(state => ({count: state.count+1});
```

`setState`是一种合并操作，不是替换操作

---

- 在执行 `setState`操作后，React 会自动调用一次 `render()`
- `render()` 的执行次数是 1+n (1 为初始化时的自动调用，n 为状态更新的次数)

### 2. props

与`state`不同，`state`是组件自身的状态，而`props`则是外部传入的数据

**类式组件中使用**

![image-20210720211554914](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210720211554914.png)

在使用的时候可以通过 `this.props`来获取值 类式组件的 `props`:

1. 通过在组件标签上传递值，在组件中就可以获取到所传递的值
2. 在构造器里的`props`参数里可以获取到 `props`
3. 可以分别设置 `propTypes` 和 `defaultProps` 两个属性来分别操作 `props`的规范和默认值，两者都是直接添加在类式组件的**原型对象**上的（所以需要添加 `static`）
4. 同时可以通过`...`运算符来简化

![image-20210720212505232](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210720212505232.png)

**函数式组件中的使用**

> 函数在使用props的时候，是作为参数进行使用的(props)

![image-20210720213304037](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210720213304037.png)

函数组件的 `props`定义:

1. 在组件标签中传递 `props`的值
2. 组件函数的参数为 `props`
3. 对 `props`的限制和默认值同样设置在原型对象上

### 3. refs

Refs 提供了一种方式，允许我们访问 DOM 节点或在 `render` 方法中创建的 React 元素。

> 在我们正常的操作节点时，需要采用DOM API 来查找元素，但是这样违背了 React 的理念，因此有了`refs`

有三种操作`refs`的方法，分别为：

- 字符串形式
- 回调形式
- `createRef`形式

**字符串形式**`refs`

![image-20210720215332387](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210720215332387.png)

虽然这个方法废弃了，但是还能用，还很好用hhh~

**回调形式的**`refs`

组件实例的`ref`属性传递一个回调函数`c => this.input1 = c `（箭头函数简写），这样会在实例的属性中存储对DOM节点的引用，使用时可通过`this.input1`来使用

**使用方法**

```js
<input ref={c => this.input1 = c } type="text" placeholder="点击按钮提示数据"/>
```

**我的理解**

`c`会接收到当前节点作为参数，`ref`的值为函数的返回值，也就是`this.input1 = c`，因此是给实例下的`input1`赋值

**createRef 形式**（推荐写法）

React 给我们提供了一个相应的API，它会自动的将该 DOM 元素放入实例对象中

我们先给DOM元素添加ref属性

```html
<input ref={this.MyRef} type="text" placeholder="点击弹出" />
<input ref={this.MyRef1} type="text" placeholder="点击弹出" />
```

通过API，创建React的容器，会将DOM元素赋值给实例对象的名称为容器的属性的`current`，好烦..

```js
MyRef = React.createRef();
MyRef1 = React.createRef();
```

注意：专人专用，好烦，一个节点创建一个容器

```js
//调用
btnOnClick = () =>{
    //创建之后，将自身节点，传入current中
    console.log(this.MyRef.current.value);
}
```

注意：我们不要过度的使用 ref，如果发生时间的元素刚好是需要操作的元素，就可以使用事件对象去替代。过度使用有什么问题我也不清楚，可能有 bug 吧

### 4. 事件处理

1. React 使用的是自定义事件，而不是原生的 DOM 事件

2. React 的事件是通过事件委托方式处理的（为了更加的高效）

3. 可以通过事件的 `event.target`获取发生的 DOM 元素对象，可以尽量减少 `refs`的使用

![image-20210720222147149](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210720222147149.png)

## 三、高阶函数

关于这部分的知识，之前的笔记有记过了，我真是太棒了

链接[高阶函数](https://linjc.blog.csdn.net/article/details/116765732)，关于AOP，偏函数，柯里化都有不错的记录，感觉还是不错的



