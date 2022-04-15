# Hooks + TS 搭建一个任务管理系统（终）--  项目总结

![hook-ts-jira-end](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/hook-ts-jira-end.png)

> 📢 大家好，我是小丞同学，一名**大二的前端爱好者**
>
> 📢 这个系列文章是实战 jira 任务管理系统的最后一篇文章
>
> 📢 用来**总结项目中遇到的问题，以及解决方法**
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 **愿你忠于自己，热爱生活**

## 💡 内容抢先看

- 技术栈
- Q&A 文档

整个项目已经学习完了，也做出来了，但是缺少后端服务器，还无法上线，稍做总结吧~

## 一、采用技术栈

本文采用了以下技术

- React 17
- React Hook
- TS4
- Hook + Content
- React Query 
- CSS in JS
- React Router 6

采用 `content` 来做全局状态管理

利用 `React Query` 进行 `url` 缓存，实现 `url` 状态管理

利用 `CSS in JS` 来替代传统组织式的 CSS 代码，将 HTML 与 CSS 选择器解耦，实现真正的组件化

利用 `TS4` 来规范 `JS` 进行类型检查，规范代码

## 二、Q&A 文档

### 1. 怎么实现页面刷新后仍然是上一次的状态？

通过 `token` 以及本地存储实现，我们在登录时，会将token 存储到本地中，这一步不需要我们手动操作，用的老师的库会自动实现。我们在初始化页面的时候，需要挂载一个 `useMount` 方法进行初始化，在这个函数里，主要进行的是 `token` 令牌的判断，如果存在 `token` 我们就，发送一个请求去获取用户数据 `data`

然后返回 `user` 数据 

### 2. 为什么使用 catch 中的 err 会报错呢？

在 `TS4.4` 版本中规定了 `catch` 中的 `err` 对象默认类型为 `unknown` ，因此我们不能用它向其他东西赋值，我们可以先进行类型设置

那为什么使用连写的方式就可以呢 `login(values).catch(onError)` 原因是，我们的 `login` 调用是异步的，但是一旦调用就会执行 `catch` ，因此获取不到值

一方面可以采用 `async` 来解决，也可以连写

### 3. 为什么控制台打印 error 总是 null

原因是 Hook 中的事件是异步的，例如 `useState` 是异步的，会先执行打印 `error`

严重问题，error 无法获取

解决！！！！

通过 `then` 的第二个参数，获取到返回错误的 `promise` 对象，然后，再通过 `throw` 抛出这个错误

被外层的 `catch` 接收，注意！！抛出错误中的 `then` 方法是一个异步事件，需要通过 `async` 来解决

```js
.then(data => {
    // 成功则处理stat
    console.log(data);
    setData(data)
    // throw new Error('222')
    return data
}, async(err) => {
    console.log('失败');
    // 卧槽，尼玛的，解决了catch 获取不到错误的问题
    throw Promise.reject(await err.then())
})
```

其他代码不变

同时注意，在 `fetch` 中返回错误，不能用 return 需要用 `throw` ，抛出 promise 错误

### 4. 页面的不同 title 是如何实现的？

采用自定义的 hook `useDocumentTitle` ，监听title 的变化

```ts
export const useDocumentTitle = (title: string) =>{
    useEffect(() => {
        document.title = title
    }, [title])
}
```

但是这不是最优的方案，直接这样使用会造成页面退出时获取标题丢失，我们想要的是，当我们退出登录时，标题会到 `jira 平台...` 字样

我们需要将页面中的最开始的那个 `title` 保存起来，也就是 `jira...` 然后，在当前页面被卸载时，改变这个 `title` 

我们可以利用 `hook` 天然的闭包特性来实现，但是这样会造成的问题是，不利于别人阅读我们的代码，闭包还是一个挺难发现的东西，在 `hook` 中

我们可以使用 `useRef` ，它能够帮我们保存变量的最初始状态，也就是 `jira...` ，因此这样也可以解决我们的问题，我们添加多一个 `useEffect` 来监听页面的卸载，当卸载时我们就设置会原先的 `title`

最终版 `useDocumentTitle` 自定义 `hook`

```ts
// 添加 title 的 hook
export const useDocumentTitle = (title: string, keepOnUnmount: boolean = true) => {
    // 利用 useRef 自定义 hook 它会一直帮我们保存好这个 title值，不会改变，
    const oldTitle = useRef(document.title).current
    // const oldTitle = document.title
    useEffect(() => {
        document.title = title
    }, [title])
    // 页面卸载时，重新设置为原来的 title
    useEffect(() => {
        // 利用闭包不指定依赖得到的永远是旧title ，是代码初次运行时的 oldTitle
        // 不利于别人阅读
        return () => {
            if (!keepOnUnmount) {
                document.title = oldTitle
            }
        }
    }, [keepOnUnmount, oldTitle])
}
```

### 5. 为什么采用 Navigate 会无法设置默认跳转呢？

盲猜版本迭代

艹，不要安装 `beta4` 版本，安装 `beta.0` ，第四版中的 `Navigate` 失效了

### 6. 在采用 antd 自定义组件的时候，如何开放更多的类型呢？

我们可以利用 `React` 自带的方法，获取到组件身上的全部类型

```ts
type SelectProps = React.ComponentProps<typeof Select>
```

然后，通过 `extends` 来继承 `SelectProps` 身上的方法

```ts
interface IdSelectProps extends SelectProps 
```

但是这样会有类型冲突的问题

![image-20210924105645394](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210924105645394.png)

因此我们需要排除掉我们在这里使用过的类型，采用 `Omit` 方法

```tsx
interface IdSelectProps extends Omit<SelectProps, 'value' | "onChange" | "options" | "defaultOptionName">
```

这样我们定义的类型就能够接收所有的 `props` 了，最后还要解构一下其他的 `props` 噢

### 7. 什么时候命名 ts，tsx 文件呢？

当包含模板文件的时候采用 `tsx` 文件，不包含模板代码的时候使用 `ts` 文件，不然会引起误会

### 8. 在代码中出现的 !! 是什么意思呢

```tsx
onCheckedChange?.(!!num)
```

例如这里的 `!!num` 

它代表的意思是 `Boolean(num)` 将 `num` 转化成 `boolean` 类型 `true or false`

### 9. 在组件中我们不能使用 hook，那我们如何更改组件状态呢？

我们可以在我们的自定义 hook 中，暴露一个函数，我们通过调用这个函数来实现状态的更新

### 10. 在请求数据返回之前如果页面被卸载了，造成报错如何解决

这个问题的来源是，我们在请求数据的时候，**我们登出了页面**，当前的 `setData` **还没有结束**，当完成时，需要渲染的页面已经不存在了，因此我们**需要判断一下**，页面是否被卸载再来渲染组件

为此我们写了一个自定义的 `hook` 用来判断组件是否被卸载

```tsx
export const useMountedRef = () => {
    const mountedRef = useRef(false)
    // 通过 useEffect hook 来监听组件状态 
    useEffect(() => {
        mountedRef.current = true
        return () => {
            mountedRef.current = false
        }
    })
    return mountedRef
}
```

主要利用了 `useEffect` 的特性，当组件卸载时执行 `return` ，当我们写自定义 hook 的话，如果返回一个函数，非常大概率是需要使用 `useMemo` 或 `useCallback` 

非常重要

### 11. 怎么理解 component composition 这种透传数据的模式

引用官网的一句话

> Context 主要应用场景在于*很多*不同层级的组件需要访问同样一些的数据。请谨慎使用，因为这会使得组件的复用性变差。
>
> **如果你只是想避免层层传递一些属性，[组件组合（component composition）](https://zh-hans.reactjs.org/docs/composition-vs-inheritance.html)有时候是一个比 context 更好的解决方案。**

**我们把我们需要用到数据的那个组件直接丢到数据来源的 props 身上** ，然后消费数据，把消费完的组件，也就是要被渲染到页面的内容，通过 `props` 传回来。这就是 `component compositon` ，简单粗暴，我们在原来的地方，直接渲染这个组件即可

例如：我们在 `Page` 组件中需要传递个 `Auth` 组件 `user` 信息，它们之间有很多的深层嵌套

我们可以这么做 （官网例子）

```tsx
function Page(props) {
  const user = props.user;
  const userLink = (
    <Link href={user.permalink}>
      <Avatar user={user} size={props.avatarSize} />
    </Link>
  );
  return <PageLayout userLink={userLink} />;
}

// 现在，我们有这样的组件：
<Page user={user} avatarSize={avatarSize} />
// ... 渲染出 ...Page的子组件
<PageLayout userLink={...} />
// ... 渲染出 ...PageLayout的子组件
<NavigationBar userLink={...} />
// ... 渲染出 ...
{props.userLink}
```

这样我们只用传递 `userLink` 即可，

### 12. 为什么创建和编辑中的关闭按钮，只有一个起作用？

造成这个问题主要原因在于这段代码

```tsx
const close = () => {
    setEditingProjectId({ editingProjectId: undefined });
    setProjectCreate({ projectCreate: undefined });
}
```

测试发现哪条语句在前面，哪个就生效，在前面的那个不会生效，初步判断造成问题的原因是异步操作，但是还没有找到解决的方法

更正问题来源：由于后面的那一条会把前面的数据重新设置上去造成的

最终将这里的两次调用抽成了一次，将 `seturl...` 函数抽象成两个，一个读取，一个设置

### 13. 搜索框的功能是如何实现的？ 

在 `useTask` 中触发，发送请求

```tsx
export const useTasks = (param?: Partial<Task>) => {
    const client = useHttp()
    // 搜索框请求在这里触发
    return useQuery<Task[]>(['tasks', param], () => client('tasks', { data: param }))
}
```

### 14. 如何部署到 github 上？

1. 部署github，创建一个 名字.github.io

2. `yarn add gh-pages -D` 安装包

3. ```shell
   // 在 package.json 中配置一下
   "predeploy": "npm run build",
   "deploy": "gh-pages -d build -r git@github.com:linjunc/linjunc.github.io.git -b main"
   ```

4. 执行 `npm run deploy` 命令

### 15. useMemo 和 useCallback 有什么区别？

- `useCallback` ：就是返回一个函数，只有在依赖项发生变化的时候才会更新。**一般在函数返回函数时，需要使用 `useCallback` 来包裹**。更多的时**防止子组件重新渲染**

> `useCallback` 返回一个**函数**，当把它返回的这个函数作为子组件使用时，可以避免每次父组件更新时都重新渲染这个子组件,子组件一般配合 `memo` 使用

- `useMemo`：传递一个创建函数和依赖项，创建函数会需要返回一个值，只有在**依赖项发生改变的时候，才会重新调用此函数**，返回一个新的值。**这里的改变，不表示地址的改变，只有值得改变**。主要**能够优化当前组件也可以优化子组件**

> `useMemo` 返回的的是一个**值**，用于避免在每次渲染时都进行高开销的计算

---

## 📌 总结

- 持续更新

> 最后，可能在很多地方讲诉的不够清晰，请见谅
>
> 💌 如果文章有什么错误的地方，或者有什么疑问，欢迎留言，也欢迎私信交流