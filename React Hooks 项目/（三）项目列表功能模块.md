# Hooks + TS 搭建一个任务管理系统（三）-- 项目列表功能模块

![hook-ts-jira-3](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/hook-ts-jira-3.png)

> 📢 大家好，我是小丞同学，一名**大二的前端爱好者**
>
> 📢 这个系列文章是实战 jira 任务管理系统的一个学习总结
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 **愿你忠于自己，热爱生活**

在上一篇中，我们封装好了一些 `custom hook` 例如，用于操作 `url` 的 `useUrlQueryParam` 以及 `useSetUrlSearchParam` 同时我们封装了专门在 `project` 列表中使用的 `hook` ，搭建好了基本的框架，这一篇我们来使用这些 `hook` 来实现我们的功能，同时我们也会引出几个 `custom hook`

## 💡 知识点抢先看

- 实现对项目的增删改查 
- 收藏功能的实现 
- 利用乐观更新来优化用户体验

## 一、实现对项目的增删改查 

### 1. 模态框的实现

首先我们先理顺现在的思路，我们现在的单页面都已经布局好了，还有几个功能没有实现，创建项目、编辑项目、删除项目、收藏项目、查找项目（这个在下一篇讲）

先来看看我们的效果图

![jira-project-create](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/jira-project-create.gif)

我们的创建项目和编辑项目**都是在一个弹出的模态框**内实现的，仔细观察，会发现我们的项目列表并没有消失，效果看起来是**叠加**的。这样，我们接下来就可以**先写创建项目和编辑项目的模态框**，我们只需要将被编辑的**项目数据**传递给模态框就可以了，对于创建项目，我们给一个**空白**的即可

这里我们的抽拉效果，采用的是 `antd` 中的 `Drawer` 组件实现的，对这个组件不熟悉的可以看看：[Drawer](https://ant.design/components/drawer-cn/#header)

![image-20211008140324136](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211008140324136.png)

从描述上来看，它会**覆盖住父窗体的内容**，正符合我们的想法，我们只需要将 `Form` 表单丢进这个 `Drawer` 组件中即可，

```tsx
<Drawer
    forceRender={true}
    onClose={closeModel}
    visible={projectModelOpen}
    width={'100%'} >
    {
        isLoading ? <Spin size={'large'} /> : <Container>
            <h1>{title}</h1>
            <ErrorBox error={error} />
            <Form form={form} layout={"vertical"} style={{ width: '40rem' }} onFinish={onFinish} >
                {/* 节省篇幅，此处省略了名称、部门和负责人的 form.Item */}
                <Form.Item style={{ textAlign: "right" }} >
                    {/* 点击提交触发onFinish方法 */}
                    <Button loading={mutateLoading} type={"primary"} htmlType={"submit"} >提交</Button>
                </Form.Item>
            </Form>
        </Container>
    }
</Drawer>
```

在这个组件中我们设置了 `forceRender` 属性，这个属性可以**控制是否强制渲染**，这也是为了解决，我们在刚打开时，组件未渲染导致**报错**的问题

同时我们也可以发现，我们在当中设置了**三元判断**，这样是为了优化我们的用户体验，前面也提过了，我们整个项目采用的是 `react-query` 进行 `url` 管理，在它的 `API` 中有能够返回 `isLoading` 状态的 `hook` 也就是我们的**数据请求的完成状态**，这也让我们可以利用这个 `isLoading` 去实现这个 `Spin` 的**加载效果**

```tsx
isLoading ? <Spin size={'large'} /> : <Container>
```

这样其实我们的 `modal` 就已经做好了，接下来我们来完善一下这个 `modal` 的**周边措施**，当我们创建完成或者编辑完成时，我们需要关闭 `modal` ，在我们的 `useProjectModel`  中已经暴露了 `close` 方法，我们只需要在 `onFinish` 中调用即可

> 当 `form` 表单成功提交时，会自动调用 `onFinish` 方法，同时会将 `form` 表单中的数据作为参数，因此我们采用 `useMutateProject` 这个 `hook` 来将数据维护到 `url` 中

```tsx
const useMutateProject = editingProject ? useEditProject : useAddProject
const { mutateAsync, error, isLoading: mutateLoading } = useMutateProject(useProjectsQueryKey())
const onFinish = (values: any) => {
    mutateAsync({ ...editingProject, ...values }).then(() => {
        form.resetFields()
        close()
    })
}
```

在这里我们采用了 2 个 `custom hook` ，`useEditProject ` 和 `useAddProject`，接下来我们就讲讲这两个 hook

> **tips**：`form.resetFields` 方法可以重置表单，也就是一个清空表单的效果

### 2. 封装增删改查 hook引出

在上一小节中，我们也看到了这些 `hook` 的使用，我们在使用的时候只需要传递一个 `queryKey` ，就能够返回一个 `mutate` 以及一些相关的配置，这些我们并没有手动的去写，那它是怎么实现的呢？

这其实利用的是 `useMutation` 这个 `react-query` 中的原生 `hook` 

```tsx
// 示例
return useMutation(
    (params: Partial<Project>) => client(`projects`, {
        method: "POST",
        data: params
    }),
    useAddConfig(queryKey)
)
```

在这里我们传递了两个参数，**第一个参数是异步请求，第二个参数是相关的配置项**

这样它就能返回一个用来实现乐观更新的 `mutate` 和 `mutateAsync` 我们可以自己选用，一个是同步的一个是异步的

在我们使用的时候，只需要要像发送请求一样，传递我们的数据即可 

```tsx
// 示例
mutateAsync({ ...editingProject, ...values }).then(() => {
    form.resetFields()
    close()
})
```

例如我们的编辑效果就采用了异步的方法

下面我们来编写这些 `hook` 

### 3. useEditProject

这是在编辑项目时需要调用的 `hook` 当我们编辑完之后，我们就可以调用这个 `hook` 暴露 `mutate` ，接着调用 `mutate` 来发送数据请求

首先我们还是逃不开我们的 `http` 这个 `hook` 所有的**异步请求都是通过这里来发送的** 

我们先返回我们的 `fetch` 方法封装的 `client` 函数 ，最后返回一个 `useMutation` 函数**调用的返回值**，这个函数接收 2 个参数，**一个是我们需要发的请求，一个是配置项**

我们通过 `client` 封装我们需要发送的请求，在编辑情况下，我们需要**传递**  `id` 来获取需要编辑的项目，`data` 则是整个传递过来的 `params` 这里面将包括了我们需要的数据，**为什么可以看出来呢？**

我们在给 `params` 限定类型时，采用了 `Partial<Project>` 这表明了 `params` 的变量和变量类型，必须**来自**于 `Project` 这个封装好的项目信息**接口** 

```tsx
// type/project.ts
export interface Project {
    id: number;
    name: string;
    personId: number;
    pin: boolean;
    organization: string;
    created: number
}

// utils/project.ts
export const useEditProject = (queryKey: QueryKey) => {
    const client = useHttp()
    // 实现乐观更新
    return useMutation(
        (params: Partial<Project>) => client(`projects/${params.id}`, {
            method: "PATCH",
            data: params
        }),
        useEditConfig(queryKey)
    )
}
```

关于 `config` 这些 `hook` 的配置，在乐观更新中会讲到

接下来我们再来处理添加请求

###  4. useAddProject

这几个 `hook` 的相似度非常高，都是一个套路，写习惯了 `custom hook` 真的可以轻松拿捏的

```tsx
export const useAddProject = (queryKey: QueryKey) => {
    const client = useHttp()
    return useMutation(
        (params: Partial<Project>) => client(`projects`, {
            method: "POST",
            data: params
        }),
        useAddConfig(queryKey)
    )
}
```

在这里我们同样的方式传递我们的 `params` 参数，使用 `useMutation` 来处理我们的请求

### 5. useDeleteProject

处理删除的请求，对于删除项目只需要传递 `id` 就可以了，删除指定 `id` 的项目

```tsx
export const useDeleteProject = (queryKey: QueryKey) => {
    const client = useHttp()
    return useMutation(
        ({ id }: { id: number }) => client(`projects/${id}`, {
            method: "DELETE",
        }),
        useDeleteConfig(queryKey)
    )
}
```

> 对于 `useMutation` 的一点理解
>
> 从上面的代码中我们可以可以发现，它都是用来处理我们的请求，我们传递一个异步请求，它也能返回一个请求的函数 (`mutate`)，因此可以理解为，使用这个 `hook`  包装我们的异步请求，让它具有能够乐观更新的功能，其他的功能和我们自己封装的 `client` 方法一致

### 6. 实现编辑，创建功能

我们在点击编辑按钮时，首先需要弹出 `modal` 编辑信息点击保存后，才需要调用发送请求

**上代码**

首先先处理 `modal` 的显示和关闭

（截取下拉框的关键代码）我们在点击编辑按钮时，会触发 `editProject` 方法，这个方法会触发 `startEdit` ，它是 `useProjectModel` 这个 `custom hook` 暴露出来的 用来开启 `modal`

```tsx
const editProject = (id: number) => () => startEdit(id)
<Menu.Item onClick={editProject(project.id)} key={'edit'}>
```

`modal` 开启来，现在我们需要将视线聚焦在 `project-modal` 这个文件当中，来处理在这个页面上的请求了！

在我们调用 `startEdit` 时，会将页面的 `url` 设置成 `editingProjectId` ，因此我们需要在 `modal` 中先**判断一下这个页面开启的请求是来自于编辑还是创建，**

```tsx
const useMutateProject = editingProject ? useEditProject : useAddProject
```

这样我们的 `useMutateProject` 就是这两个中的一个，在后面就没有什么担忧了

然后我们直接传递当前的 `queryKey` 给这个 `hook` 暴露出我们需要的 `mutate` 这个请求函数，以及错误状态和请求状态

```tsx
const { mutateAsync, error, isLoading: mutateLoading } = useMutateProject(useProjectsQueryKey())
```

当我们的 `form` 表单被提交时，我们调用这个方法传递我们 `params` 发送请求

```tsx
const onFinish = (values: any) => {
    mutateAsync({ ...editingProject, ...values }).then(() => {
        form.resetFields()
        close()
    })
}
```

这样我们的创建编辑功能就实现了

### 7. 删除功能

这里有一个比较好玩的东西，当我们点击删除时，不能立即执行，我们需要用户确认后才能发送请求，因此我们需要再多封装一层函数 `confirmDeleteProject` ，用来提示用户是否确定删除

```tsx
<Menu.Item onClick={() => confirmDeleteProject(project.id)} key={'delete'}>
```

再这里我们采用了 `antd` 组件中的 `Modal` 组件下的 `confirm` 框

```tsx
const confirmDeleteProject = (id: number) => {
    Modal.confirm({
        title: '你确定删除项目吗？',
        content: '点击确定删除',
        okText: '确定',
        onOk() {
            deleteProject({id})
        }
    })
}
```

在这里配置我们的提示框的相关信息，以及确定后执行的操作，这里当 `onOk` 时会调用 `deleteProject` 来发送请求，从这个命名也知道它调用的是 `useDeleteProject` ，这也是命名规范的好处之一

![image-20211008170502500](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211008170502500.png)

这样，我们的删除功能就也实现了，关于增删改就写到这里，在这里我们又写了大量的 `custom hook`，自己提升还是很大的

## 二、收藏功能的实现 

对于这个小星星的样式，我们采用的是 `Antd` 中而定 `Rate` 组件

![image-20211008172232127](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211008172232127.png)

它大概长这个样子它可以通过 `count` 来控制星星的个数，因此我们重新封装一个 `Pin` 组件

```tsx
interface PinProps extends React.ComponentProps<typeof Rate> {
    checked: boolean;
    onCheckedChange?: (checked: boolean) => void
}
export const Pin = ({ checked, onCheckedChange, ...restProps }: PinProps) => {
    return <Rate
        // 一颗星
        count={1}
        value={checked ? 1 : 0}
        onChange={num => onCheckedChange?.(!!num)}
        {...restProps}
    />
}
```

由于我们新封装的 `Pin` 组件也需要拥有 `Rate` 组件的属性，因此我们采用了一个**继承**的操作 ，我们可以通过 `React.ComponentProps<typeof Rate> ` 来获取 `Rate` 中的**所有 `props` 类型**，也就是**接收参数**的类型，我们将我们的 `Pin` 组件的 `props` 参数用上这个类型就可以了

> 这里采用了一个 `!!num` 的高端操作，其实就是一个转化成 `boolean` 类型的方法

接着我们就可以在 `columns` 中使用这个 `Pin` 组件了，在星星状态改变时调用编辑方法，改变数据中的 `pin` 状态

```tsx
{
    title: <Pin checked={true} disabled={true} />,
    render(value, project) {
        return <Pin
            checked={project.pin}
            // 利用柯里化技术，首先传入 id ,在传入pin ，最后执行 mutate
            onCheckedChange={
                pinProject(project.id)
            }
        />
    }
},
```

在这里我们采用柯里化的方式优化了这段代码，我们在编写 `pinProject` 时，采用了柯里化的方式，一次接收一个参数，返回一个函数，最后执行 `mutate` 

```tsx
const { mutate } = useEditProject(useProjectsQueryKey())
// 指定修改的pin id 即可
const pinProject = (id: number) => (pin: boolean) => mutate({ id, pin })
```

这样我们的收藏功能就成功的实现了

## 三、实现乐观更新

接下来我们来谈谈这个乐观更新，可能很多人都不太知道乐观更新是什么东西，我们先来科普一下

> 采用乐观更新，用户界面的行为就像在从服务器收到实际确认之前成功完成更改一样 ，它乐观地认为它最终会得到确认而不是错误。这可以提供更快速的用户体验。
>
> 简单的说，我们的页面信息会在服务器请求结果返回之前去更新，例如收藏按钮，如果我们的请求时间为 5s，那么不采用乐观更新，收藏的按钮就会在 `5s` 之后采用亮起，而采用乐观更新，则会默认的认为服务器返回的结果必然成功，我们先做去预判，先在用户点击的时候直接亮起按钮，请求让它慢慢请求去吧

现在我们就来编写一下乐观更新的代码吧~，在前面的 `hook` 中我们的第二个参数 `config` 没有讲，它就是实现乐观更新的关键

首先我们需要编写一个 `useConfig` ，这个在几个 `hook` 中都必须使用到，因为利用 `useMutation` 这个 API 来实现乐观更新，会牵扯到 `useMutation` 生命周期的问题，我们封装一个 `useConfig` 来编写这些生命周期函数

在这个 hook 中我们使用了大量的 `any` ，无关大雅

我们在成功、提交、失败中设置了相应的回调，来处理不同的请求情况

```tsx
// 乐观更新,用来生产代码的 hook
// 这里的类型非常的复杂采用了很多的any ，代价是可以接受的
export const useConfig = (queryKey: QueryKey, callback: (target: any, old?: any[]) => any[]) => {
    const queryClient = useQueryClient()
    return {
        // 生命周期函数
        onSuccess: () => queryClient.invalidateQueries(queryKey),
        // 提交请求
        async onMutate(target: any) {
            // 数据列表
            const previousItems = queryClient.getQueryData(queryKey)
            queryClient.setQueryData(queryKey, (old?: any[]) => {
                return callback(target, old)
            })
            return { previousItems }
        },
        onError(error: any, newItem: any, context: any) {
            // 发生错误继续缓存旧的值
            queryClient.setQueryData(queryKey, context.previousItems)
        }
    }
}
```

我们来简单讲讲这些 API 吧

1.  `queryClient.invalidateQueries`： 在提交成功/失败之后都进行重新查询更新状态
2. `queryClient.getQueryData`：获取缓存的旧值
3. `queryClient.setQueryData`：设置值

接下来我们来编写相应的 `config` ，那 `delete` 来讲

```tsx
export const useDeleteConfig = (queryKey: QueryKey) => useConfig(queryKey, (target, old) => old?.filter(item => item.id !== target.id) || [])
```

这段代码它其实就只是传入了我们删除项目的数据，然后通过 `filter` 整理了一下数据传递给了 `useConfig` ，因此，这几个都是类似的只是传递的参数不一样

`useConfig` 接收 2 个参数，一个是 `queryKey` ，一个是新值旧值的函数

因此我们通过 `filter` 从旧数据中过滤掉被删除的项目，这样返回的数据就是我们所要的新数据了

```tsx
export const useEditConfig = (queryKey: QueryKey) => useConfig(queryKey, (target, old) => old?.map(item => item.id === target.id ? { ...item, ...target } : item) || [])
export const useAddConfig = (queryKey: QueryKey) => useConfig(queryKey, (target, old) => old ? [...old, target] : [])
```

同理这两个 `hook` 也这么写，通过数组的方法筛选出新的数据即可

这样我们的乐观更新的逻辑就完成了！

> 对于底层的实现原理，还不是很熟悉，所以表诉的可能不大清楚

---

那么这部分的内容就到这里了，下一篇将会讲关于搜索部分的实现~

## 📌 总结

通过这篇文章我们可以学会以下这些内容

1. 在 antd 组件的基础上封装新的组件
2. 采用乐观更新优化体验
3. 项目的增删查功能
4. 采用 `react-query` 进行状态管理
5. 柯里化解决实际问题

> 最后，可能在很多地方讲诉的不够清晰，请见谅
>
> 💌 如果文章有什么错误的地方，或者有什么疑问，欢迎留言，也欢迎私信交流

