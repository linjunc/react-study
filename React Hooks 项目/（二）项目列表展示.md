# Hooks + TS 搭建一个任务管理系统（二）-- 项目列表展示

![hook-ts-jira-2](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/hook-ts-jira-2.png)

> 📢 大家好，我是小丞同学，一名**大二的前端爱好者**
>
> 📢 这个系列文章是实战 jira 任务管理系统的一个学习总结
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 **愿你忠于自己，热爱生活**

在我们写好登录注册界面后，我们需要开始解决登录后的项目列表展示页，这也是我们在自动登录后显示的页面

## 💡 知识点抢先看

这篇文章将讲到以下几个知识点

- antd 组件库渲染项目列表
- `...` **更多**按钮的实现
- 通过 **URL 进行状态管理**
- 封装项目列表中的 `url` 操作

![image-20211006224843631](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211006224843631.png)

## 一、antd 组件库渲染项目列表

首先我们先来讲讲页面中最重要的列表，这里采用的是 Antd 组件库中的 `Table` 组件为基础架构，我们在它的基础上重新创建了一个 `List` 组件表示我们的项目列表

大概的结构如下

```tsx
export const List = ({ users, ...props }: ListProps) => {
    return <Table rowKey={"id"} pagination={false} columns={[
        // 此处省略6个 {} 结构 
    ]} {...props} />
}
```

我们需要向 `columns` 中注入数据，在这里我们的 `List` 组件接收了需要使用的数据，用户数据以及相关配置项

这里利用的是一个**类型的继承**

```tsx
interface ListProps extends TableProps<Project> {
    users: User[];
    refresh?: () => void;
}
```

我们通过这个接口继承了 `Table` 组件原先的所有 `props` 参数的类型的基础上，又添加了几个类型，这样我们的数据既能符合需求，也能顺利的**穿透**到 `Table` 组件中。同时我们需要给 `Table` 组件指定数据源 `dataSource` ，在这样处理后，我们直接可以使用 `{...props}` 即可

> 在这里我们使用的 `Project` 泛型，其实也指定了 `dataSource` 的类型，也是 `columns` 中的获取数据类型

根据我们  UI 图，这里一共需要有6个数据：收**藏情况、名称、部门、负责人、创建时间、更多按钮**

这里将从三个问题来讲解如何渲染数据

1. **如何分列渲染数据？**

我们通过 `Table` 组件的 `columns` 属性添加对象的方式来实现 `List` 中的每一列，简单的说就是组件自带的属性，直接配置就好，这里的 `title` 也就是用来设置列头的标题

```tsx
{
    title: '名称',
    //其他配置
},
// 其他5列
```

![image-20211006233238264](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211006233238264.png)

不用标题的话可以不设置 `title` 属性

2. **如何显示数据呢？**

我们可以使用 `dataIndex` 以及 `render` 来实现

**首先** `dataIndex` 这个是 `columns` 中的一个 `API` ，我们可以通过它来指定**列数据的来源**

> `dataIndex` :  列数据在数据项中对应的路径，支持通过数组查询嵌套路径

对于部门的数据展示

```tsx
{
    title: '部门',
    dataIndex: 'organization',
    sorter: (a, b) => a.name.localeCompare(b.name)
}
```

**它可以指定从哪里来获取这些数据，这里就是指定从 `project` 内直接获取数据**

我们这里采用的就是这种方法，这样就能直接的对数据进行**列渲染**

**同时**我们还可以采用 `render` 方法

> 生成复杂数据的渲染函数，参数分别为当前行的值，当前行数据，行索引

一般用来处理一些比较难的逻辑，比如 名称

我们采用的就是 `render` 来渲染

```tsx
{
    title: '名称',
    sorter: (a, b) => a.name.localeCompare(b.name),
    render(value, project) {
        return <Link to={String(project.id)}>{project.name}</Link>
    }
}
```

首先值得注意的是，这里的 `render` 和其他的 `render` 不同，这里的 `render` 更像是一个函数，我们通过传递参数，然后返回结构，就能渲染在页面上

> function(text, record, index) {}

它接收三个参数，都是可选的，分别是当前行的值，**当前行数据**，行索引

这里特别注意的是当前行数据，我们可以直接使用 `props` 中的数据，这里我们传入的是 `project` ，最后返回一个 `Link` 元素，这样渲染到页面上的就是一个 `Link` 标签

3. **如何实现列排序呢？**

在 `columns` 中有一个 `sorter` API，我们可以通过它来实现排序

```tsx
sorter: (a, b) => a.name.localeCompare(b.name)
```

通过名字大小写来排序

> 其实这里讲的都是 `Table` 组件的用法而已，查看文档也能实现

在这里有一些列中渲染的是**一个组件**，在后面会讲到

## 二、更多按钮的实现

在 `Table` 列表的 `columns` 属性中我们的最后一列（更多），采用的是一个封装的组件，这样可以减少我们 `Table` 组件的代码，同时实现组件复用（这次没有用到）

更多按钮的实现也是利用了一个 Antd 库中的 `Dropdown` 和 `Menu` 组件，实现一个下拉框的效果

```tsx
<Dropdown overlay={<Menu>
    <Menu.Item onClick={editProject(project.id)} key={'edit'}>
        <ButtonNoPadding type={'link'}>编辑</ButtonNoPadding>
    </Menu.Item>
    <Menu.Item onClick={() => confirmDeleteProject(project.id)} key={'delete'}>
        <ButtonNoPadding type={'link'}>删除</ButtonNoPadding>
    </Menu.Item>
</Menu>}>
    <ButtonNoPadding type={"link"}>
        ...
    </ButtonNoPadding>
</Dropdown>
```

利用 `overlay` 配置一个 `Menu` 组件，在 `Menu` 中配置**下拉显示的内容** ,`Dropdown` 中直接配置 **当前显示的内容**

![image-20211007003107927](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211007003107927.png)

这个就是实现的效果，这里封装了一个 `ButtonNoPadding` 组件，是一个 Antd 中去除 `padding` 的 `Button` 组件

> 关于删改的实现后面会讲解

关于布局就涉及这么多，接下来才是重头戏

## 三、通过 URL 进行状态管理

这里有很多的问题！！！

在这里我们就讲几个 `custom hook` 吧

- `useUrlQueryParam` 
- `useSetUrlSearchParam`

这两个 `hook` 分别是用与返回页面 `url` 中的 `query` 和设置当前的 `URL` 地址的

知道了它们的作用，我们来一步步实现它

首先在这里有人可能会有疑惑我们为什么不将这两个 `hook` 写成一个呢？

> 这里一开始实现的时候是写的一个 `hook` ，但是到后面逻辑复杂了之后，就会出现无限循环的情况，同时造成 `url` 的重复跳转，难以实现我们的逻辑，因此我们将两个逻辑分离开来，让它的功能更加具体化

这里我们先来写 `useSetUrlSearchParam` ，因为在我们的查看逻辑中使用了这部分的代码

### 1. useSetUrlSearchParam

**首先**我们使用 `react-router-dom` 中的 `useSearchParams` 这个 `hook` ，它返回一个 `searchParams` 和 ` setSearchParams`，从用法上来看有点像 `useState` ，通过这个 `hook` 可以来处理我们的**查询字符串**

在这里我们接收一个参数 `params` ，也就是查询字符串，用来设置我们的 `url`，例如我们的编辑页面的 `url`

![image-20211007144245611](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211007144245611.png)

是通过拼接了一个 `editingProjectId=id` 实现的，**转化成代码**的话就是我们这里的 `params` ，在传递的时候是以对象**键值对**的方式来传递的，因此在这里我们对 `params` 的类型的定义应该符合这个规则

```tsx
params: { [key in string]: unknown }
```

对于初学 TS 的来说，**如何理解这样的类型定义**呢？

我们指定 `params` 的类型是一个对象 `{}` ，它的 `:` 左侧也就是 `key` 被指定为 `string` ，右侧 `unknown` 指定 `value` 的类型

在我们成功接收到这个 `params` 时，我们将这个数据解构出来，与原先 `url` 中存在的 `query` 一同经过清理之后，将得到的对象传递给 `setSearchParams` 来设置当前的 `url`

```tsx
// 通过这个单独得 hook 来 set search param
// 把输入框的内容映射到url地址上
export const useSetUrlSearchParam = () => {
    const [searchParams, setSearchParams] = useSearchParams()
    return (params: { [key in string]: unknown }) => {
        const o = cleanObject({
            ...Object.fromEntries(searchParams),
            ...params
        }) as URLSearchParamsInit
        return setSearchParams(o)
    }
}
```

**讲讲我自己对这里的理解吧**

由于我们这部分采用的是 SPA 

一方面我们需要实现打开网址时，显示对应的页面，另一方面我们需要实现我们的跳转          

我们在这里采用的这样的方式：**在我们点击创建或者编辑时，我们将当前的项目列表组件切换成编辑组件**，同时我们通过我们封装的 `custom hook` 来**手动的更改**当前的 `url`，从而实现了 `url` **与数据与页面相匹配**    

###  2. useUrlQueryParam             

首先再次明确我们这个 hook 的功能：返回页面 `url` 中的 `query` ，同时利用 `useSetUrlSearchParam` 返回的方法来设置 `url` 

我们先来明确以下这个 `hook` **接收的参数和返回的值**

> 接收一个 keys 的数组，也就是 `query` 中的键名的数组，返回一个数组，第一个元素是一个对象保存着 `key-value` ，第二个元素是一个方法，也就是修改 `url` 的方法

接下来我们再来确定以下**接收参数的类型**

> 这里我们接收一个泛型 `K` 的数组，同时由于这是 `key` ，这个 `K` 应当继承 `string`

```tsx
<K extends string>(keys: K[])
```

接下来我们来引入一些我们需要用到的方法，查询和设置

```tsx
// 定义了一些实用的方法来处理 URL 的查询字符串
const [searchParams] = useSearchParams()
const setSearchParams = useSetUrlSearchParam() // 引入这个自定义的方法，不使用原生自带的
```

我们再来研究以下如何返回当前 `url` 的 `query` 对象

```tsx
useMemo(
    () => keys.reduce((prev, key) => {
        // 解决当get 的值是null 时的默认值
        return { ...prev, [key]: searchParams.get(key) || '' }
        // 传入的是一个 key 类型在 K 中值为 string 的对象
    }, {} as { [key in K]: string }),
    [keys, searchParams]
)
```

首先我们通过 `reduce` **遍历**传入的 `keys` 数组，每一次遍历都将使用 `searchParams` 方法去**查找对应的** `value` 值，遍历完成后会**返回整个对象**，利用 `reduce` 将每次的 `key-value` 添加到 `{}` 中，最后全部返回

这里我们给 `reduce` 传入了第二个参数，**指定了我们传入的函数的初始值**

同时在这里我们采用了 `useMemo` 这个 `hook` 来优化我们的代码，只有在依赖项改变的时候才会重新计算，这样可以**解决无限循环**的问题（**todo**: 关于无限循环的问题之后出一篇文）

接下来我们来研究返回数组的第二个值

```tsx
// 键值限定在我们设置的范围之内
(params: Partial<{ [key in K]: unknown }>) => {
    // 把 fromEntries 转化为一个对象
    return setSearchParams(params)
}
```

这个很简单，直接将传入的 `params` 传递给 `setSearchParams` 中添加就可以了~

在这里我们采用了一个 `Partial` 方法，它是 `TS ` 联合类型中的一个点，它可以把指定的泛型中的类型都**变成可选**的

**底层实现**

```tsx
type Partial<T> = {
    [P in keyof T]?: T[P];
};
```

最后一个非常重要的点是 `as const` ，这个也是 `TS` 中比较高级的用法，也叫做 **const 断言**，否则会错乱

关于 const 断言，做个简单的解释，如果没有使用 `as const` 的话，会默认的进行类型推断，`return` 返回的是一个函数类型的数组，但是它完全忘记了有两个元素，因此会丢失返回数组中元素的类型，采用 `const` 断言，就能指示使表达式的字面类型不被扩展

未采用 `const` 断言

![image-20211007172612517](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211007172612517.png)

采用 `const` 断言

![image-20211007172656899](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211007172656899.png)

能明显感受出来它们的不同

以下是 `return` 的完整代码

```tsx
return [
    useMemo(
        () => keys.reduce((prev, key) => {
            return { ...prev, [key]: searchParams.get(key) || '' }
        }, {} as { [key in K]: string }),
        [keys, searchParams]
    ),
    (params: Partial<{ [key in K]: unknown }>) => {
        return setSearchParams(params)
    }
] as const
```

为了给下一篇文章搭建好梯子，接下来我们写一下我们这两个 `custom hook` 在 `project` 列表中的应用

## 四、封装项目列表中的 url 操作                                                 

由于我们在 `project` 列表中会大量使用到 `url` 操作，为了能将我们的代码更加简洁，我们利用 `useUrlQueryParam` 这个轮子来造车，在这个基础上将 `project` 的特定 `keys` 传入即可，这样我们在 `project` 中使用时，就可以直接调用对应的 `searchParams` 方法

这里我们讲 3 个 `custom hook` 

- useProjectsSearchParams
- useProjectsQueryKey
- useProjectModel

### 1. useProjectsSearchParams

这一个 `hook` 就是 `useUrlQueryParam` 的作用，只是将它具体到了 `project` 中使用

返回的是一个数组，第一个元素是查找的数据，第二个是修改的方法

```tsx
export const useProjectsSearchParams = () => {
    // 要搜索的数据
    // 返回的是一个新的对象，造成地址不断改变，不断的渲染
    // 用这个方法来设置路由地址跟随输入框变化
    // 服务器返回的都是 string 类型
    const [param, setParam] = useUrlQueryParam(['name', 'personId'])
    return [
        // 采用 useMemo 解决 重复调用的问题
        useMemo(() => ({ ...param, personId: Number(param.personId) || undefined }), [param]),
        setParam
    ] as const
}
```

我们在使用这个 `hook` 的时候，**直接调用**即可，因为我们已经指定了它的 `keys` 数组为 `['name', 'personId']`，这个是在 **搜索模块** 中使用的 `hook`

### 2. useProjectsQueryKey

这个 `hook` 用来返回 `query` 的键值对，返回的是 `{name: '', personId: undefined}` 样式

```tsx
export const useProjectsQueryKey = () => {
    const [params] = useProjectsSearchParams()
    // {name: '', personId: undefined}
    return ['projects', params]
}
```

我们在使用的时候也是直接调用即可返回数据

### 3. useProjectModel

我们通过这个 `hook` 来判断当前的状态**是不是在创建、编辑**，如果是的话我们就显示出我们对应的页面

首先我们先从利用 `useUrlQueryParams` 来获取到页面的 `query` 对象

再**通过对象解构的方式，解构出对应的数据**，例如这里我们解构出 `query` 中的 `projectCreate` 字段

那第一个来说就是利用 `useUrlQueryParam` 传入 `projectCreate` 来在 `url` 中查找有没有这个字段，**返回查找的结果**，同时返回一个可以修改它的函数 `setProjectCreate` ，这就是我们的 `url custom hook` 发挥的作用了

```tsx
const [{ projectCreate }, setProjectCreate] = useUrlQueryParam([
    'projectCreate'
])
// 判断当前是不是在编辑，解构出当前编辑项目的 id
const [{ editingProjectId }, setEditingProjectId] = useUrlQueryParam([
    'editingProjectId'
])
```

在接下来的代码中就是**封装一些更改它们的方法**，暴露出去给外部直接调用，例如**控制 modal 页面的开关**，`open` 和 `close` 方法，**控制编辑页面**开启的 `startEdit` 方法

代码逻辑非常简单，我们只需要调用对应的 `set...` 方法来改变 `url` 中的对应键值对的值就可以了

```tsx
const open = () => setProjectCreate({ projectCreate: true })
const startEdit = (id: number) => setEditingProjectId({ editingProjectId: id })
const close = () => setUrlParams({
    editingProjectId: undefined, projectCreate: undefined
})
```

例如 `open` 我们通过 `setProjectCreate({ projectCreate: true })` 将 `projectCreate` 改成 `true` 表示当前正在创建的页面

关于这个 `editingProjectId` 我们可以通过 `useProject` 这个 `custom hook` 来获取（或许在下一篇会讲到，这里不展开），采用的是 `react-query` ， 它返回的是一个 `data` 数 据

最后我们暴露这些方法

```tsx
return {
    // 采用 id才是最佳选择，这样不用等待数据返回就能打开编辑框
    projectModelOpen: projectCreate === 'true' || Boolean(editingProjectId),
    open,
    close,
    startEdit,
    editingProject,
    isLoading
}
```

这样我们的 `project` 列表下的 `url` 控制操作 `hook` 就全部完成了

那么这篇文章就到这里结束了，在接下来的文章中，会利用这些封装好的 `hook` **去实现项目列表的增删改查以及乐观更新等功能**

## 📌 总结

1. 在这篇文章中我们写了大量的 `custom hook` ，也更加的熟练了它的写法和好处
2. 对 `const` 断言有了一定的了解
3. 学会了如何使用 `Table` 、`Dropdown` 等组件
4. 大致的认识了 `useMemo` 的用法
5. 对 `useSearchParams` 有了一定的了解
6. `TS` 中的联合类型有了更深的理解

> 最后，可能在很多地方讲诉的不够清晰，请见谅
>
> 💌 如果文章有什么错误的地方，或者有什么疑问，欢迎留言，也欢迎私信交流

