# Hooks + TS 搭建一个任务管理系统（四）-- 搜索功能实现

![hook-ts-jira-4](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/hook-ts-jira-4.png)

> 📢 大家好，我是小丞同学，一名**大二的前端爱好者**
>
> 📢 这个系列文章是实战 jira 任务管理系统的一个学习总结
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 **愿你忠于自己，热爱生活**

在上一篇文章中，我们已经写过了关于**项目列表展示**的部分，通过大量的 `custom hook` 实现了项目的增删改查，也写很多复用性很高的 `hook` ，这样我们可以在后面的代码中复用，优化和缩减我们的开发时间

## 💡 知识点抢先看

- 封装 userSelect 组件
- 将输入框内容映射到 url 上
- 利用防抖优化输入框请求

先献上效果图

![jira-project-search](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/jira-project-search.gif)

## 一、封装 UserSelect 组件

这次的项目采用的是 `Antd` 组件库，在这部分中我们采用 `Form` 表单以及 `Input` 来实现搜索框的样式，对于下拉框，将采用以 `Select` 组件为基础的 `UserSelect` 自定义组件

重新封装 `Select` 组件，在这里我们首先是封装了一个 `IdSelect` 组件，再在这个组件的基础上抽象一个 `UserSelect` 组件

这样做的目的是为了让 `IdSelect` 组件能够**实现复用**

下面我们先来写 `IdSelect` 组件吧，从名字上也可以看出，它是通过 `id` 来选择 `option` 的

在前面的文章中，我们也有提到过，利用 `antd` 组件来封装自定义组件，需要**继承它的原先的类型**，来保持它的 `props` 正常工作

我们先来看看 `IdSelect` 应当接收的**参数类型**

```tsx
// 继承 Select 身上的方法
type SelectProps = React.ComponentProps<typeof Select>
// 在 type 中定义公共类型
interface IdSelectProps extends Omit<SelectProps, 'value' | "onChange" | "options" | "defaultOptionName"> {
    value?: Raw | null | undefined,
    // onChange 只能传入number
    onChange?: (value?: number) => void,
    defaultOptionName?: string,
    options?: { name: string, id: number }[]
}
```

它的类型还是比较复杂的

首先是 `SelectProps` 定义的一个类型等于 `Select` 的类型，再在  `IdSelectProps` 的类型中继承部分的 `SelectProps` 类型

**为什么说是部分呢？**

由于我们原生的 `Select` 组件中对于 `onChange` 属性的类型是采用泛型来定义的，这会导致我们的 `number` 类型数据转化成 `string` ，总之就会导致最后的后端返回数据的类型和 `Select` 中的类型不一致，因此我们需要将 `onChange` 限制为 `number` 类型

这个是 `onChange` 的**类型声明**

```tsx
onChange?: (value: ValueType, option: OptionsType[number] | OptionsType) => void;
```

 同时对于一些类型我们有自己明确的类型，因此我们不需要采用它原生的类型，我们自己重新定义

因此我们采用 `Omit` 关键字来除去 `SelectProps` 中的部分类型声明，重新写一份

```tsx
Omit<SelectProps, 'value' | "onChange" | "options" | "defaultOptionName">
```

这样我们就完成了对 `Select` 数据类型的封装，接着我们需要将一些相关的配置**全部传递**给它们

例如，`value` 属性的默认值，`onChange` 的执行时机，以及 `defaultOptionName`

```tsx
export const IdSelect = (props: IdSelectProps) => {
    const { value, onChange, defaultOptionName, options, ...restProps } = props
    return <Select
    // 这里设置了value ：0 ，当我们数据还没有返回的时候，它会显示 负责人字样
        value={options?.length ? toNumber(value) : 0}
        onChange={value => onChange?.(toNumber(value) || undefined)}
        {...restProps}
    >
        {
            defaultOptionName ? <Select.Option value={0}>{defaultOptionName}</Select.Option> : null
        }
        {
            options?.map(option => <Select.Option key={option.id} value={option.id}>{option.name}</Select.Option>)
        }
    </Select>
}
```

代码的思路很简单，当没有 `options` 时，`value` 设置为 `0` ，显示**默认负责人**。同时我们需要对传入的 `value` 进行类型转化，保证它是 `number` 类型

这样我们的 `IdSelect` 就封装好了，它相对于 `Select` 有更加严格的类型要求，以确保我们传递的参数类型不会出错

接着我们将这个 `IdSelect` 特殊化到 `User` 中，再封装一个 `UserSelect` 给 `project` 中按照人员查找来使用

```tsx
export const UserSelect = (props:React.ComponentProps<typeof IdSelect>)=>{
    const {data:users} = useUsers()
    return <IdSelect options={users|| []} {...props}></IdSelect>
}
```

 写熟练了真是随便拿捏

同样的，我们的数据类型继承自 `IdSelect` ，然后，我们先直接传入我们的 `Users` 数据，实现了一个 `UserSelect` 

**为什么这样就可以了呢？**

我们将数据传递下去之后，得到的 `Select` 就是一个人员列表了，这样我们只需要做一些其他配置就可以了，**不需要考虑人员数据的问题**

![image-20211008211259106](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211008211259106.png)

**接着**，我们在搜索部分的 `Form` 表单中，使用这个组件

```tsx
// search-panel.tsx
<UserSelect
// 默认选项
    defaultOptionName={'负责人'}
    value={param.personId}
    onChange={value =>
        setParam({
            ...param,
            personId: value
        })} />
```

在这里我们配置了默认选型，以及通过 `props` 传递的用户 `id` （`param.personId`），同时在输入框被选择时触发的事件，用来操控我们的页面 `url` 变化

## 二、将输入框内容映射到 url 上

在上一小节我们最后谈到了 `url` 的变化，确实如此，当我们在输入框中输入内容时，或者时 `Select` 中选择内容时，都应该要**映射到 `url` 中**，这样我们将 `url` 复制在新页面打开，还会**保留同样的信息**，这种功能也是非常常见的，例如**掘金社区**的文章标题，`h1、h2` 标签

![image-20211008212554826](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211008212554826.png)

因此我们有理由，有必要实现这样的功能！

想到 `url` 操作，我们很容易想到我们的 `useProjectsQueryKey` 这一类 `hook`，当然这有一定的关系

在这里我们需要使用我们之前封装过的 **`useProjectsSearchParams`** 这个 `custom hook` ，

 我们先再看看这个 `hook` 的源码

```tsx
export const useProjectsSearchParams = () => {
    // 返回的是一个新的对象，造成地址不断改变，不断的渲染
    const [param, setParam] = useUrlQueryParam(['name', 'personId'])
    return [
        // 采用 useMemo 解决 重复调用的问题
        useMemo(() => ({ ...param, personId: Number(param.personId) || undefined }), [param]),
        setParam
    ] as const
}
```

简单梳理一下，就是通过 `useUrlQueryParam` 来设置和查询相关的`query` 数据，返回的是一个数组，形式类似于 `useState` ，一个是值，一个更改这个值

我们可以看到这个 `hook` 监听的 `url query` 是 `name、personId` 也就是**项目名和负责人**，正符合我们的查询需求

我们先在 `ProjectListScreen` 这个 `project` 的最外层组件中暴露 `hook` 中返回的两个方法

```tsx
const [param, setParam] = useProjectsSearchParams()
```

这样如果我们通过 `setParam` 导致了 `param` 的变化，就会触发 `useUrlQueryParam` **实现页面的 `url` 的更新**                                                                                                                                                                                                                                  

例如这里的**搜索模块**，我们通过 `props` 传递 `setParam` 方法给子组件

```tsx
<SearchPanel users={users || []} param={param} setParam={setParam} />
```

在子组件中使用这个方法来控制 `param` 的变化，从而引起 `url` 的变化

例如，我们在监听 `input` 框输入时

```tsx
<Input
    type="text"
    placeholder={'项目名'}
    value={param.name}
    onChange={e => setParam({
        ...param,
        name: e.target.value
    })} />
```

我们在 `onChange` 中调用了 `setParam` 设置了新的 `param` 值，在 `UserSelect` 中同样的采用这样的方式修改 `param` 值，触发 `url` 的更新，这样我们的功能就实现了一半了，接下来我们需要**利用当前用户查询的 `param` 去获取数据**

```tsx
const { isLoading, error, data: list} = useProjects(param, 200)
```

返回获取到的结果和状态即可，这里采用的 `useProjects` ，是一个封装的 `custom hook` ，它会在 `param` 变化时 ，**通过 `useQuery` 不断的请求数据**，这也是我们返回的数据中能够有 `isLoading、error` 这些的原因 

**在这里提一下 `useQuery`** ，它是 `reacy-query` 中的一个 `api` ，用来做缓存的，接收的第一个参数是用来起名字，第二个参数是异步请求，它会把请求的结果放到缓存中，但是**这个缓存不是浏览器缓存**

第一个参数可以是一个数组，类似于 `useEffect` ，当依赖项变化的时候就会触发 `useQuery` 重新执行

```tsx
export const useProjects = (param?: Partial<Project>) => {
    const client = useHttp()
    // 当 param 变化的时候触发 useQuery 重新渲染，我们需要在第一个参数中传入一个数组，数组的第二位传入依赖
    return useQuery<Project[]>(['projects', param], () => client('projects', { data: param }))
}
```

现在我们的功能也算是基本实现了，但是我们打开控制台会发现有很多很多的请求，这并不是我们想要的，因此我们可以采用防抖，**每隔多少秒，再请求一次**

![jira-project-search-keep](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/jira-project-search-keep.gif)

## 三、useDebounce 实现防抖

为了减少请求的次数，我们封装了一个 `useDebounce` 方法，用来对数据进行防抖操作

关于防抖不必多说了吧，这里我们采用的是 `useState` 来创建这个全局变量，通过 `set...` 来控制它值的变化，也就这一点不一样的地方

简单说一说这里的泛型吧，这里我们采用了一个泛型 `V` ，第一个 `<V>` 是用来做泛型声明的，它的类型由我们传入的 `value` 来指定，`value` 是什么就是什么 

```tsx
export const useDebounce = <V>(value: V, delay?: number): any => {
    // 设置一个 debouncedValue 值，用于暂存值，以及监控变化
    const [debouncedValue, setDebouncedValue] = useState(value)
    useEffect(() => {
        // 接收一个定时器，参数为一个函数和延时时间 
        // 每次value变化，设置一个定时器
        const timeout = setTimeout(() => setDebouncedValue(value), delay)
        // 每次上一个useEffect 的定时器被清除，相当于上一个定时器被卸载了
        return () => clearTimeout(timeout)
        // 监听value 和 delay 变化，当参数变化时，重新调用这个函数设置定时器
    }, [value, delay])
    // 返回值
    return debouncedValue
}
```

![jira-project-search1](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/jira-project-search1.gif)

## 📌 总结

在这篇文章中我们做完了项目列表的搜索模块，我们能学到这些东西

1. 已有组件封装新的组件参数类型问题
2. 如何 实现了输入框与 `url` 统一
3. 采用 `hook` 实现防抖

> 最后，可能在很多地方讲诉的不够清晰，请见谅
>
> 💌 如果文章有什么错误的地方，或者有什么疑问，欢迎留言，也欢迎私信交流