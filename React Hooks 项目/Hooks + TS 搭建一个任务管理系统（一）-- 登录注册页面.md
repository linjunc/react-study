# Hooks + TS 搭建一个任务管理系统（一）-- 登录注册页面

![hook-ts-jira-1](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/hook-ts-jira-1.png)

> 📢 大家好，我是小丞同学，一名**大二的前端爱好者**
>
> 📢 这个系列文章是实战 jira 任务管理系统的一个学习总结
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 **愿你忠于自己，热爱生活**

## 💌 前言

这篇文章是这个专栏中的第一篇文章，因此就写点前言吧~，简单的介绍一下吧

最近刚学完 React 的一些基本内容，教学视频已经看完了，然后也学习了一下 TS 这门强类型的语言，对前端开发简直就是利器。同时也了解了一下 Hooks 的一些内容，但是对这部分掌握的不是很好，因此跟着视频利用 `Hooks + TS4 + Router6 `做了一个任务管理系统练练手。在做这个 hooks 的项目之前，也有跟着做过一个基于 `React 16.4 版本 + Redux` 实现的简书博客平台，对 `Redux` 也有一定的了解。

扯这么多，来说说这个项目吧！

> 这个项目是跟着视频做的并不是完全由我创新的 😥，因此**如果文章有侵权行为的话，麻烦联系一下删除**（应该不会吧，毕竟文章是我自己写的）

这个项目采用的技术栈是 `React Hooks + TS4`

主要实现的功能有 ：用户登录注册，项目列表的展示，项目的 CRUD，项目详情展示，看板及任务组管理...

接下来的系列更文，将会围绕实现这些功能，以及在项目中遇到的难题，提出一些问题和解决方案。

强迫自己开启这个专栏是想要更加深入的理解，自己写的代码是什么意思，能够如何优化，了解更多代码上的细节，而不是跟着老师敲完代码就算了...，因此这个专栏会尽我所能将知识点囊括齐全！

高能预警：本项目采用了很多的 `custom hook` ，真的非常不错

下面开始今天的主题，实现登录注册页面

![image-20211006203414683](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20211006203414683.png)

## 一、用状态驱动页面更新

为什么第一个要讲“用状态驱动页面更新”呢？

我们需要通过当前的**登录状态**，来展示不一样的页面。通过状态来做很多的事情...

首先我们需要通过 `useState` ，来创建两个状态，一个是 `isRegister` 用来标识是展示登录界面还是注册界面，当 `isRegister` 为 `true` 时展示注册页面

第二个状态是**错误状态**，用来接收登录页面的错误信息，当有错误发生时，都会丢到这个变量当中

```tsx
// 标识当前是注册还是登录，false 表示当前是登录状态
const [isRegister, setIsRegister] = useState(false);
const [error, setError] = useState<Error | null>(null)
```

在上面的两行代码中，值得注意的是，通过 `useState` 创建的变量类型默认会是**初始化**时的类型

也就是说 `isRegister` 的类型会因为我们初始化时传的 `false` 变成 `boolean` 类型

而对于 `error` 而已，在不加泛型的情况下，它默认会是 `null` 类型，因此，在后面对它赋值 `Error` 对象类型时，会发生错误，因此在这里我们需要定义泛型 `Error | null` 这样 `error` 就能接收 `Error` 类型了~

现在我们的状态设置好了，接下来看看如何驱动页面更新呢，那一个例子讲讲

```tsx
<Button type={'link'} onClick={() => setIsRegister(!isRegister)}>{isRegister ? '已经有账号了？直接登录' : '没有账号？注册新账号'}</Button>
```

这个是登录和注册切换的按钮，当点击这个按钮时，会触发 `setIsRegister` 改变 `isRegister` 的值，我们通过这个值的 `true or false` 来判断展示的内容

```tsx
{/* 判断展示登录页面还是注册页面 */}
{
    isRegister ? <RegisterScreen onError={setError} /> : <LoginScreen onError={setError} />
}
```

当为 `true` 的时候展示注册页面，在这里我们将两个页面抽象出了两个组件，将逻辑分开来，我们通过 `props` 向这两个组件传递了 `onError` 方法，在组件中可以通过调用这个方法来设置 `error` 状态的值，再展示到页面上

在这里值得我们注意的是，和类式组件不同，函数式组件会默认的接收 `props` 参数，因此我们不需要显式的去使用 `props` 我们可以直接在参数列表中**解构**出来，这样我们整个项目开发完成都不会见到一个 `props` 

## 二、通过 Antd 布局页面

关于布局方面采用的是 `flex` 布局，主要是通过 Antd 组件来实现的

```tsx
<ShadowCard>
    <Title>
        {
            isRegister ? '请注册' : "请登录"
        }
    </Title>
    <ErrorBox error={error} />
    {/* 判断展示登录页面还是注册页面 */}
    {
        isRegister ? <RegisterScreen onError={setError} /> : <LoginScreen onError={setError} />
    }
    <Divider />
    {/* 点击切换状态 */}
    <Button type={'link'} onClick={() => setIsRegister(!isRegister)}>{isRegister ? '已经有账号了？直接登录' : '没有账号？注册新账号'}</Button>
</ShadowCard>
```

这里的 `ShadowCard` 其实是对 `Antd` 中的 `Card` 组件进行了**加工**，让它有了一些**阴影**，同时对它进行了一定的布局

```tsx
// 组件加样式，给Card组件更改样式
const ShadowCard = styled(Card)`
    width: 40rem;
    min-height: 56rem;
    padding: 3.2rem 4rem;
    box-shadow: rgba(0,0,0,0.1) 0 0 10px;
    text-align: center;
`
```

在 `emotion` 中，想要个 Antd 组件添加样式，我们只需要用 `styled(组件名)` 即可

对于登录和注册页面，采用的是 Antd 中的  `Form` 表单实现的，在控制好盒子大小后，基本不需要过多的布局

```tsx
<Form onFinish={handleSubmit}>
    <Form.Item name={'username'} rules={[{ required: true, message: '请输入用户名' }]}>
        <Input placeholder={"用户名"} type="text" id={"username"} />
    </Form.Item>
    <Form.Item name={'password'} rules={[{ required: true, message: '请输入密码' }]}>
        <Input placeholder={"密码"} type="password" id={"password"} />
    </Form.Item>
    <LongButton loading={isLoading} htmlType={"submit"} type={"primary"}>登录</LongButton>
</Form>
```

对于登录注册的关键就是，**通过前台认证之后，发送请求开启认证即可**，关键就在于这个认证如何实现，当然如果只是简单的发请求是非常简单的，但是往后想想，我们会有**很多个请求**，如果我们每次都写一遍那串代码，那代码的冗余程度可想而知。

因此我们想在这里抽象出两个 `custom hook` ，**一个用来获取数据，一个用来处理异步请求**，写这两个之前，我们先写一个专门用来发送请求的文件，我们将我们关于登录注册的请求全部写在这个文件当中，再暴露出去，这样代码看起来思路更加清晰

## 三、编写 auth-provider 文件

我们在这个文件中来处理我们需要发送的相关请求，首先，由于我们需要实现刷新后仍保持登录状态的效果，我们需要设置 `token` ，并且对于 `token` 数据我们是放在本地存储当中的

```tsx
// 保存本地存储中的 token 键
const localStorageKey = "__auth_provider_token__";
// 获取 token 的值
export const getToken = () => window.localStorage.getItem(localStorageKey);
```

通过封装一个函数用来获取我们本地的 `token` 值

```tsx
export const handleUserResponse = ({ user }: { user: User }) => {
  window.localStorage.setItem(localStorageKey, user.token || "");
  return user; 
};
```

通过这个函数来设置本地 `token` ，在登录注册后调用

**处理登录请求**

```tsx
export const login = (data: { username: string; password: string }) => {
  return fetch(`${apiUrl}/login`, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify(data),
  }).then(async (response) => {
    if (response.ok) {
      return handleUserResponse(await response.json());
    } else {
      throw Promise.reject(await response.json());
    }
  });
};
```

当我们在其他文件中调用这个 `login` 时就会返回这个 `fetch` 能够发送登录的请求，当成功返回结果时，就会调用前面的函数来设置一个本地的 `token` 值，用来保存用户的登录状态

这里有个比较重要的点：由于我们的请求都是异步的因此我们在 `then` 中需要采用 `async await` 的方式，优雅的解决这个由于异步造成的 undefined 的问题，对于其他注册和登出的请求也是如此

在编写好几个请求函数之后，我们需要编写一个 `useAsync` 函数用来专门处理异步请求

## 四、编写 useAsync 发送异步请求

我们已经能够发送请求获取**登录信息**了，为什么我们还需要再编写一个这样的 `custom hook` 呢？

首先，我们在上面确实是能够满足我们最基本的业务需求了，我们编写这个 `custom hook` 能够帮我们将这个异步函数给具体化，什么是具体化呢？

我们先来看看这个 `custom hook` 返回的结果

```tsx
return {
    // 请求状态
    isIdle: state.stat === 'idle', 
    isLoading: state.stat === 'loading',
    isError: state.stat === 'error',
    isSuccess: state.stat === 'success',
    // run 接收一个promise 对象，返回执行结果
    run,
    setData,
    setError,
    // retry 被调用重新执行 run，让state 更新
    retry,
    ...state
}
```

看到这些返回的结果，相信已经有了一定的想法，我们可以通过这个 `hook` 来直视到**异步函数的执行过程**，而且又能将过程**抽象**在这个 hook 当中，在外部，我们只需要 `run` 一下，就能得到结果，这不正是我们想要的吗？

我们**不想关注异步的细节**，什么 `then` 啊，`async` 啊，这些我们都不想关心，我们想要的是，**执行后的结果**，因此这个 hook 需要帮我们解决这些问题！这在优化我们代码中起着非常重要的作用

对于这个 hook 的实现，比较复杂，类型复杂，

```tsx
interface State<D> {
    error: Error | null;
    // 返回的数据
    data: D | null;
    // 请求过程状态
    stat: 'idle' | 'loading' | 'error' | 'success'
}
```

首先我们定义**初始化状态的接口**

初始化我们的初始状态

```tsx
// 初始状态
const defaultInitialState: State<null> = {
    stat: 'idle',
    data: null,
    error: null
}
```

我们先写一个 hook 来帮我们判断组件**是否卸载**

```tsx
// 用这个dispatch 会帮我们判断 mountedRef 组件是否被卸载
const useSafeDispatch = <T>(dispatch: (...args: T[]) => void) => {
    const mountedRef = useMountedRef()
    return useCallback((...args: T[]) => (mountedRef.current ? dispatch(...args) : void 0), [dispatch, mountedRef])
}
```

当我们使用这个 hook 时，将会接收到当前组件的状态，当组件被卸载后，我们就不需要再将数据返回了，如果返回的话，就会造成数据无法渲染的情况从而报错，因此，我们编写这个 hook 也是出于这样的考虑

我们通过监听 `safeDispatch` 的变化来该判断当前的状态，同时我们可以通过 `setData` 来传递返回的数据，再通过 `safeDispatch` 来发送 `dispatch` 设置响应

```tsx
const safeDispatch = useSafeDispatch(dispatch)
// 正常响应时的数据处理
const setData = useCallback((data: D) => safeDispatch({
    data,
    stat: 'success',
    error: null
}), [safeDispatch])
// 发生错误时的错误处理
const setError = useCallback((error: Error) => safeDispatch({
    error,
    stat: 'error',
    data: null
}), [safeDispatch])
```

当然还有一些其他的状态也需要这样编写，基本一致

在这里我们开始编写我们的 `run` 函数，这个函数是主入口，**用于触发异步请求**，首先从我们的调用上来看 `run(login(values))` 我们只想传递一个 `promise` 对象就能获得所有的结果，

首先我们需要先判断一下，传入的对象是不是 `promise` 对象，如果不是则直接抛出错误

当进入 `run` 函数后，我们需要将 `stat` 状态置为 `loading` 状态，这样我们可以通过这个值来实现请求 loading 的效果，

最后我们返回一个 `promise` 对象的执行结果，在这个返回当中有很多值得探讨的地方

为了获取到传入的 `promise` 对象**抛出的错误**，我需要使用 `then` 中的第二个参数来接收这 **错误对象**，再返回这个错误，才能使用 `catch` 获取，正常情况下，`catch` 获取不到这个错误

```tsx
// run是主入口，触发异步请求
// 采用useCallback,只有依赖中的数据发生变化的时候，run才会被重新定义
const run = useCallback((promise: Promise<D>, runConfig?: { retry: () => Promise<D> }) => {
    // 如果传入的不是 promise，直接 throw
    if (!promise || !promise.then) {
        throw new Error('请传入 Promise 类型数据')
    }
    // 定义重新刷新一次，返回一个有上一次 run 执行时的函数
    setRetry(() => () => {
        if (runConfig?.retry) {
            run(runConfig?.retry(), runConfig)
        }
    })
    // 如果是 promise 则设置状态，开始 loading
    safeDispatch({ stat: 'loading' })
    // 返回一个promise对象处理数据        
    return promise
        .then(data => {
            // 成功则处理stat
            // 判断组件状态
            setData(data)
            return data
        }, async (err) => {
            // 接收到扔来的错误，再扔一下
            return Promise.reject(await err)
        })
        .catch(error => {
            // 错误抛出了，但是接不住
            setError(error)
            if (config.throwOnError) {
                return Promise.reject(error)
            }
            return Promise.reject(error)
        })
}, [config.throwOnError, safeDispatch, setData, setError])
```

在这个 `hook` 中有太多值得我们学习的地方

首先当我们的 `custom hook` 返回的值是一个函数时，我们最好用 `useCallback` 来包一下，这样能解决无限循环的问题

在我们的请求当中需要对异步情况做出特别的处理，利用 `async` 来解决这些问题

对于数据的类型，需要我们对泛型有很清晰的认识

## 五、编写 useAuth 获取用户信息

在编写好 `useAsync` hook 后，我们需要 通过 `useAuth` 来**获取用户的信息**，主要是依赖于 `useAsync` ，这也能体现出 `useAsync` 的巨大威力

在这个 `custom hook` 当中，我们会采用 `useAsync` 暴露的方法，同时也会采用到 `react-query` 处理缓存，利用 `context` 来实现**数据共享**

```tsx
export const useAuth = () => {
    // 由于在使用 context 时，需要在子节点中声明一下这个 context
    const context = React.useContext(AuthContext)
    // 如果这个 context 不存在
    if (!context) {
        throw new Error('useAuth必须在 context 中使用')
    }
    // 返回这个 context 数据中心
    return context
}
```

当我们调用这个 hook 的时候，就会**返回这个 context 对象** ，`AuthContext` ，当然不会这么简单，关键在于我们如何将这些数据存储在 `context` 当中

我们编写一个 `AuthProvider` 方法

```tsx
export const AuthProvider = ({ children }: { children: ReactNode }) => {
    // 设置一个user变量 ，由于user 的类型由初始化的类型而定，但不能是 null ，我们需要进行类型断言
    // const [user, setUser] = useState<User | null>(null)
    const { data: user, error, isLoading, isIdle, isError, run, setData: setUser } = useAsync<User | null>()
    const queryClient = useQueryClient()
    // 设置三个函数 登录 注册 登出
    // setUser 是一个简写的方式 原先是：user => setUser(user)
    const login = (form: AuthForm) => auth.login(form).then(setUser)
    const register = (form: AuthForm) => auth.register(form).then(setUser)
    const logout = () => auth.logout().then(() => {
        setUser(null)
        // 清除数据缓存
        queryClient.clear()
    })
    // 当组件挂载时，初始化 user
    useMount(() => {
        run(bootstrapUser())
    })
        // 当初始化和加载中的时候显示loading
        if (isIdle || isLoading) {
            return <FullPageLoading />
        }
        if (isError) {
            return <FullPageErrorFallback error={error} />
        }
    // 返回一个 context 容器
    return <AuthContext.Provider children={children} value={{ user, login, logout, register }} />
}
```

当我们这个方法返回了一个 `provider` 容器，这需要我们对 `context` 有一定的了解，我们需要使用 `provider` 来包裹数据共享的范围，只有在这个范围内的元素才能使用这些数据

这里的意思是，所有的子元素都能够使用这个 `context` 容器 ，我们在使用的时候

```tsx
<AuthProvider>
    {children}
</AuthProvider>
```

这样**所有的子元素都能共享**它的 `context` 容器

接下来我们看看这个函数都写了什么，首先我们调用 `useAsync` 解构出了它的部分返回结果，这些都是我们后面可能会用到的

在这里我们对**当前的状态进行了判断**

```tsx
// 当初始化和加载中的时候显示loading
    if (isIdle || isLoading) {
        return <FullPageLoading />
    }
    if (isError) {
        return <FullPageErrorFallback error={error} />
    }
```

当状态为 `loading` 时我们展示一个加载框，当 `error` 时，展示一个**错误提示框**

```tsx
// 当组件挂载时，初始化 user
    useMount(() => {
        run(bootstrapUser())
    })
```

在组件刚挂载时，我们先检查是否存在 `token` 如果有，我们就对他进行**自动登录**

```tsx
// 保持用户登录状态，在组件挂载的时候就调用
const bootstrapUser = async () => {
    let user = null
    // 从本地取出 token
    const token = auth.getToken()
    if (token) {
        // 如果有值，就去发送请求获得 user 信息
        const data = await http('me', { token })
        user = data.user
    }
    // 返回 user
    return user
}
```

同时我们还将 `auth-provider` 中编写的三个方法，一同存放到了 `context` 容器当中，这样我们可以在外部调用

```tsx
<AuthContext.Provider children={children} value={{ user, login, logout, register }} />
```

这里的 value 设置的就是它的 context 容器中的值

通过编写这个 custom hook 我们对 `useAsync`  有了更好的理解，同时也学会了如何使用 `context` 来进行数据的共享

## 六、按钮触发函数执行

在编写完了前面的几个 `custom hook` 之后，我们已经将数据接口转到了 `context` 当中，因此我们在调用里面的内容时，只需要调用 `useAuth` 来解构出对应的数据即可

```tsx
// login.tsx
const { login } = useAuth()
// 采用 useAsync 来封装异步请求，添加loading
const { run, isLoading } = useAsync(undefined, { throwOnError: true })
```

我们得到了 login 函数，同时也得到了 `isLoading` 的状态

当表单提交时，会触发 `Form` 组件中的 `onFinish` 事件，我们给他绑定了一个 `handleSubmit` 方法，用于发送请求

```tsx
const handleSubmit = async (values: { username: string, password: string }) => {
    // 采用 antd 组件库后代码优化
    // 这里的catch 会捕获错误，调用 onError 这个函数相当于是 error => onError(error) 
    // 由于在index中传入的props是，onError={setError} 因此就相当于 setError(error)
    run(login(values)).catch(onError)
}
```

就这样我们就能够成功的发送请求，并且返回结果，当有错误发生时，会触发 `catch` 中的 `onError` 设置 `index` 中的 `error` 状态，显示在页面当中

## 📌 总结

在这个登录注册页面当中，我们可以**学到以下几点**

1. context 状态管理
2. custom hook 在 react 中的强大威力
3. 当 custom hook 返回函数时，需要使用 useCallback 包裹
4. 多利用解构赋值，来优化代码
5. useState 设置的变量，类型会跟随初始值的类型
6. 对于不同的事务，我们最好能分离出来写，这样我们的主文件思路会非常清晰
7. 利用 CSS in JS 解决样式混乱的问题

> 最后，可能在很多地方讲诉的不够清晰，请见谅
>
> 💌 如果文章有什么错误的地方，或者有什么疑问，欢迎留言，也欢迎私信交流

