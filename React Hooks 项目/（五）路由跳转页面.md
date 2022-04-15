# Hooks + TS 搭建一个任务管理系统（五）-- 路由跳转页面

![hook-ts-jira-5](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/hook-ts-jira-5.png)

> 📢 大家好，我是小丞同学，一名**大二的前端爱好者**
>
> 📢 这个系列文章是实战 jira 任务管理系统的一个学习总结
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 **愿你忠于自己，热爱生活**

在上一篇文章中我们已经写完了**首页项目列表的展示部分**，利用了大量的 `custom hook` 来处理对 `url` 进行操作，实现了将 `query` 映射到 `url` 的操作，同时利用 `react-query` 中的 `useMutation` 搭配实现了乐观更新的效果，同时利用 `useDebounce` 来减少请求，优化性能

接下来我们将处理一下其他的页面，在开发其他页面之前，我们先树立好骨架，先将页面的跳转以及 `title` 变化这些基本的独立于业务之外的东西写好

## 💡 知识点抢先看

- 利用 `router 6` 实现路由跳转
- 封装 `useDocumentTitle` 来**设置文档的标题**

实现效果

![jira-project-router-jump](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/jira-project-router-jump.gif)

## 一、利用 router 实现路由跳转

实现跳转我们先把视线放到点击的链接上，在这里我们给项目利用了 `Link` 组件进行包裹，同时采用 `to` 属性实现了 `url` 的转变

```tsx
{
    title: '名称',
    sorter: (a, b) => a.name.localeCompare(b.name),
    render(value, project) {
        return <Link to={String(project.id)}>{project.name}</Link>
    }
}
```

现在当我们点击第一个项目时，会将路由跳转到了 `projects/1`  地址下，这样显然是不能找到对应的页面的，它缺少了页面的标识

我们在 `project/index.tsx` 文件中，编写侧边栏的样式，以及设置路由的跳转，这里我们需要采用 `react-router` ，以及  `antd` 配合实现

```tsx
<Aside>
    <Menu mode={'inline'} selectedKeys={[routeType]}>
        <Menu.Item key={'kanban'}>
            <Link to={'kanban'}>看板</Link>
        </Menu.Item>
        <Menu.Item key={'epic'}>
            <Link to={'epic'}>任务组</Link>
        </Menu.Item>
    </Menu>
</Aside>
```

在这里我们采用了 `Menu` 菜单标签，利用了 `react-router-dom` 中的 `Link` 组件来实现地址的跳转，侧边栏对地址的操作，会导致右侧，看板和任务组的切换，因此我们需要给右侧配置相应的 `Route` 连接组件

```tsx
<Main>
    <Routes>
        <Route path={'/kanban'} element={<KanbanScreen />} />
        <Route path={'/epic'} element={<EpicScreen />} />
        {/* 默认路由是push，相当于又成为了栈顶，也就是当前页面被push了两次，第一次的值不匹配第二次才匹配 */}
        {/* 采用replace这样就能替换掉传入的栈顶元素，下面的路由成为了栈顶*/}
        <Navigate to={window.location.pathname + '/kanban'} replace={true} />
    </Routes>
</Main>
```

在这里我们需要设置一个 `Navigate` 用来做路由跳转的兜底方案，当上面**两个都没有匹配**上时，我们将它的地址**拼接**上 `/kanban` 强制的跳转到 `/kanban` 页面，这也是实现我们**从项目列表点击跳转后显示看板页面的原因**

在这里有很多值得注意的地方，我们在这里采用了 `replace` 来**替换路由，这是有原因的！**

浏览器的历史记录就像一个**栈的数据结构**，当我们采用 `to` 跳转时，实际上是向栈中 `push` 了一个路由地址，这里我们采用 `Navigate` 来进行**设置默认路由**，它的操作也是 `push`，也就是说，我们为了跳转到当前页面被 `push` 了**两次**

因此当我们点击返回上一页时，又会跳转到当前的 `kanban` 页面，又向栈中 `push` 了**两个地址**，这样我们的返回就永远在这里不断地循环，**永远返回不去上一页。**

因此在这里我们需要采用 `repalce` 去**替换栈顶那个匹配不上的路由地址！**

**Q&A**

**在实现这部分的时候，遇到了一些问题，稍微提及一下，给后人乘凉**

由于使用的是最新版的 `router` 在安装的时候，会让你选择版本，目前应该是更新到了 `react-router6 - beta4` 版本了，在这个版本中使用 `Navigate` 会有问题，这个 `Navigate` 的默认路由不会生效，具体原因不是很清楚，遇到这种情况可以降低一下版本到 `beta0`

这个版本中是没有问题的

## 二、封装 useDocumentTitle 来设置文档的标题

在上面我们已经顺利的实现了路由跳转，对 `Router` 有了一定的理解，接下来我们来做一个好玩的 `hook` ，它用来控制文档的标题

![jira-project-router-title](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/jira-project-router-title.gif)

大概的效果是这样，这个 `hook` 我们可以**迁移到其他的项目中使用，复用性很高**

我们先理一下思路

1. 首先需要获取到当前的 `title`
2. 在调用 `hook` 的时候需要接收一个 `title` ，并设置一个 `title`
3. 会不会有时候设置 `title` 一样 ，不需要重新设置呢

我们先来看看我们实现好的 `useDocumentTitle` 是如何使用的

```tsx
useDocumentTitle('项目列表', false)
```

第一个参数传递的是需要设置的 `title` ，第二个参数用来配置 `title` 在组件卸载时是不是需要变化

首先我们先来设置一下它接收的参数类型

```tsx
export const useDocumentTitle = (title: string, keepOnUnmount: boolean = true) => {
}
```

这里我们接收传来的 `title` 和 配置选项

首先我们先让 `title` 能够驱动页面 `title` 的更新

我们利用 `useEffect` 来实现在 `title` 变化时，修改文档标题

```tsx
useEffect(() => {
    document.title = title
}, [title])
```

接下来我们来处理，组件在卸载时不变化的情况，为什么需要添加这个逻辑呢？

如果我们不添加这个逻辑的话，需要每个页面都指定 `title` 如果未指定就会显示默认的 `title` ，因此我们增加了这个可选配置项

```tsx
// 利用 useRef 自定义 hook 它会一直帮我们保存好这个 title值，不会改变，
const oldTitle = useRef(document.title).current
```

首先我们采用 `useRef` 来保存当前的 `title`，也就是更改前的 `title`

接着我们采用 `useEffect` 来处理在组件卸载时的 `title` 变化

```tsx
useEffect(() => {
    // 利用闭包不指定依赖得到的永远是旧title ，是代码初次运行时的 oldTitle
    // 不利于别人阅读
    return () => {
        if (!keepOnUnmount) {
            document.title = oldTitle
        }
    }
}, [keepOnUnmount, oldTitle])
```

这里我们利用到了闭包的知识， `oldTitle`  一直是初次运行的 `title`

## 📌 总结

这篇文章没有太多的内容，写了一个简单的 `hook` ，稍稍总结一下

1. 利用 `Router` 实现路由跳转
2. 避免 `react-router` 版本问题，产生的错误
3. 封装高复用性的 `hook` `useDocumentTitle`

> 最后，可能在很多地方讲诉的不够清晰，请见谅
>
> 💌 如果文章有什么错误的地方，或者有什么疑问，欢迎留言，也欢迎私信交流

