# Hooks + TS 搭建一个任务管理系统（七）-- 任务组页面实现

![hook-ts-jira-7](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/hook-ts-jira-7.png)

> 📢 大家好，我是小丞同学，一名**大二的前端爱好者**
>
> 📢 这个系列文章是实战 jira 任务管理系统的一个学习总结
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 **愿你忠于自己，热爱生活**

在上一篇文章中，我们处理了看板页面的布局，以及它的逻辑功能，基础功能已经基本实现，项目、任务的增删改查，搜索功能的实现，在这一篇我们就对任务组页面进行最后的布局，和功能实现，写到这里，大部分的功能 `hook` 已经实现了，对于增删改查我们也已经非常了解了。

## 💡 知识点抢先看

- 增删任务组功能
- 路由跳转

![jira-project-epic-show](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/jira-project-epic-show.gif)

## 一、页面布局

这部分已经写过好几次了，速战速决

### 1. 布局的简单介绍

这里我们采用的是 `antd` 中的 `List` 组件，顶部左右两侧采用的是自己封装的 `Row` 组件，让它们排列在两侧，链接跳转部分采用的 `Link` 组件，通过遍历数据的方式实现渲染

![image-20211012154536792](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211012154536792.png)

### 2. 数据的获取

在这里我们需要获取到我们的任务数据，在这里我们需要写一个获取数据的 `custom hook`： `useEpics` ，和其他获取数据的 `hook` 一样

我们接收一个 `param` 数据对象，通过 `useQuery` 发送请求

> 再复习一下，它的第二个参数是一个异步事件，第一个参数是**元组**，当依赖项 `param` 发生改变时，会重新发送请求，更新缓存中的 `epics` 数据内容

```tsx
export const useEpics = (param?: Partial<Epic>) => {
    const client = useHttp()
    return useQuery<Epic[]>(['epics', param], () => client('epics', { data: param }))
}
```

我们在 `epic/index.ts` 中使用 ，获取任务组数据 `epics` 以及用于跳转链接的 `tasks` 数据

```tsx
// 关于任务的信息
const { data: epics } = useEpics(useEpicSearchParams())
// 获取任务组中的任务列表
const { data: tasks } = useTasks({ projectId: currentProject?.id })
```

这样我们就实现了**数据的获取**

接下来我们来看看如何在组件中使用这两个数据的

对于 `epics` 它作为我们需要渲染的主内容，需要通过 `List.Item` 进行渲染

在 `List` 组件中，我们可以传入我们的数据源 `dataSource` ，通过 `renderItem` 属性，对 `epics` 数据进行遍历

```tsx
 <List dataSource={epics} renderItem={epic => <List.Item></List.Item> />
```

 这样我们的 `epic` 就是每一个任务数据通过**对象取值**方式就能获取需要的数据

在这里主要提一下对于时间的渲染 

![image-20211012160029127](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211012160029127.png)

**后端给我们返回的数据格式是时间戳，我们需要将她转变成这种格式便于阅读**

在这里我们采用了一个 `dayjs` 的库，通过 `format` 方法确定了她输出的时间格式 `YYYY-MM-DD` ，只需要传入它的时间即可

```tsx
<div>开始时间：{dayjs(epic.start).format("YYYY-MM-DD")}</div>
<div>结束时间：{dayjs(epic.end).format("YYYY-MM-DD")}</div>
```

## 二、增删任务组功能

首先我们先来实现删除任务组的功能

### 1. 删除任务组

**实现思路**如下

1. 点击删除按钮，弹出提示框
2. 确认删除
3. 调用接口删除缓存

**代码实现**

当我们点击删除时，我们调用 `confirmDeleteEpic` 函数，进行删除确认

这个函数封装的是一个 `Modal.config` 组件

```tsx
// 删除时的提示框
const confirmDeleteEpic = (epic: Epic) => {
    Modal.confirm({
        title: `你确定删除项目组${epic.name}吗？`,
        content: '点击确定删除',
        okText: '确定',
        onOk() {
            // 确认时调用删除
            deleteEpic({ id: epic.id })
        }
    })
}
```

当我们在点击确认时，正式调用删除接口 `deleteEpic` ，传入我们删除的任务组 `id` ，即可删除

我们来看看如何实现这个 `deleteEpic` 

首先我们还是需要封装一个 `useDeleteEpic` 的 `hook` 用来**处理删除请求**，这里采用 `useMutation` 来处理，传入当前的 `id` ，**配置删除的 `config` 对象**

> 写到这里自己也对 `useMutation` 有了进一步的认识，它可以接收两个参数，第一个参数我们**传入我们的异步请求**，第二个参数来配置 `config` **如何处理缓存中的数据**

```tsx
// 删除看板
export const useDeleteEpic = (queryKey: QueryKey) => {
    const client = useHttp()
    return useMutation(
        // 这里我没有出现问题，视频出现了问题
        // 直接（id:number)
        ({ id }: { id: number }) => client(`epics/${id}`, {
            method: "DELETE",
        }),
        useDeleteConfig(queryKey)
    )
}
```

这样我们的删除功能就实现了

### 2. 添加任务组功能

实现思路

1. 写一个 `create-epic` 页面
2. 写入新增任务组信息
3. 提交创建请求

**代码实现**

首先我们需要在 `epic` 文件夹目录下创建一个 `create-epic` 文件，用来编写创建任务页面

> 这样做的好处是能够将复杂部分分离出来，使得主文件中的代码量减少，阅读性更佳

新增任务组页面，我们同样采用的是 `Drawer` 组件来实现

值得注意的是我们必须要添加 `forceRender={true}` 组件，否则在页面第一次加载时会报错

在 `Drawer` 组件中同样的我们采用了 `Form` 组件，**当表单提交时自动调用 `onFinish` 方法，处理添加请求**

```tsx
const onFinish = async (values: any) => {
    // 仅仅传一个values 不够，需要传入 projectid
    await addEpic({ ...values, projectId })
    props.onClose()
}
```

在这里我们采用的时一个 `async`、`await` 的组合，等待接口返回结果后我们再关闭窗口，但是由于我们采用了乐观更新，这里其实只要写入缓存中就会关闭窗口了

同时为了让 `Form` 表单在窗口关闭时自动清空，这里我们采用了 `useEffect` 来实现，在依赖项中写入 `visible` 监听变化

```tsx
useEffect(() => {
    form.resetFields()
}, [form, props.visible])
```

这样我们的创建功能也实现了，最后我们再稍微讲讲任务组 `item` 中的路由跳转

## 三、路由跳转

当我们点击下面的任务时，需要跳转到看板页面对应任务的编辑窗口，我们来看看效果图

![jira-project-epic-router](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/jira-project-epic-router.gif)

其实这只要我们的路由地址配置好了就没有问题了

我们来看看如何配置这个跳转的**路由地址**

指定到对应的 `editingTaskId` 页面，这样窗口就会弹出来了，这样是我们采用 `url` 进行状态管理的好处

```tsx
to={`/projects/${currentProject?.id}/kanban?editingTaskId=${task.id}`}
```

**那么我们如何将对应的任务绑定到对应的任务组下呢？**

这里我们采用 `filter` 来实现，当 `task` 下的 `epicId` 和 `epic` 下的 `id` 一致时说明是这个任务组下的，我们遍历渲染即可

```tsx
{
    tasks?.filter(task => task.epicId === epic.id)
        .map(task => <Link
            style={{ marginRight: '20px' }}
            key={task.id}
            // 链接到看板页面
            to={`/projects/${currentProject?.id}/kanban?editingTaskId=${task.id}`}>
            {task.name}
        </Link>)
}
```

> 注意：采用 `map` 是一定要注意 `key` 唯一噢~

---

## 📌 总结

1. 能够熟练的实现了增删功能
2. 认识到了 `url` 状态管理的好处
3. 采用合适的数组的方法可以极好的帮助我们实现功能

> 最后，可能在很多地方讲诉的不够清晰，请见谅
>
> 💌 如果文章有什么错误的地方，或者有什么疑问，欢迎留言，也欢迎私信交流
