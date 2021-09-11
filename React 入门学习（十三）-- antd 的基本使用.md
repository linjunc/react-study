#  React 入门学习（十三）-- antd 组件库的基本使用

![antd](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/antd.gif)

> 📢 大家好，我是小丞同学，一名**大二的前端爱好者**
>
> 📢 这篇文章是学习 React 中 React antd组件库的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 **愿你忠于自己，热爱生活**

## 引言

在我们学习` JavaScript` 的时候，我们学习了一个 `bootstrap` 的组件库。可以让我们快速开发，但是我们现在学习了 React ，一种组件化编程方式，很少说会去贴大量的 HTML 代码，再配一下 CSS，JS。我们也有一些现成的组件库可以使用，我们只需要写一个组件标签即可调用。这让我们 React 开发变得十分的快速，方便和整洁。

我们这里学习的是 `Ant-design` （应该是这样），它有很多的组件供我们使用

![image-20210907180731157](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210907180731157.png)

按钮，日历，这些都是非常常用的组件，我们一起看看如何使用吧

## 1. Antd 组件基本使用

使用 `Antd` 组件非常的简单

引包 ----- 暴露 ---- 使用

首先我们通过组件库来实现一个简单的按钮

#### 第一步

安装并引入 `antd` 包

使用命令下载这个组件库

```shell
yarn add antd
```

在我们需要使用的文件下引入，我这里是在 `App.jsx` 内引入

```js
import { Button } from 'antd'
```

在引入的同时，暴露出要使用的组件名 `Button`

推荐去[官方文档](https://ant.design/components/button-cn/)查看，都会有代码解释

![image-20210907181354552](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210907181354552.png)

现在我们可以在 App 中使用 `Button` 组件

```js
<div>
    App..
    <Button type="primary">Primary Button</Button>
    <Button>Default Button</Button>
    <Button type="dashed">Dashed Button</Button>
    <br />
    <Button type="text">Text Button</Button>
    <Button type="link">Link Button</Button>
</div>
```

我这里使用了几种按钮

但是就这样你会发现按钮少了样式

我们还需要引入 `antd` 的 CSS 文件

```js
@import '/node_modules/antd/dist/antd.less';
```

可以在 `node_modules` 文件中的 `antd` 目录下的 `dist` 文件夹中找到相应的样式文件，引入即可

![image-20210907181854774](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210907181854774.png)

即可成功引入 `antd` 组件

## 2. 自定义主题颜色

由于这些组件采用的颜色，都是支付宝蓝，有时候我们不想要这样的颜色，想要用其他的配色，这当然是可以实现的，我们需要引用一些库和更改一些配置文件来实现

在视频中，老师讲解的是 `3.几` 版本中的实现方法，这种方法需要去暴露 React 中的配置文件，这种操作是不可返回的，一旦暴露就不可回收。我觉得这不是一个好方法~

在 `antd` 最新版中，引入了 `craco` 库，我们可以使用 `craco` 来实现自定义的效果

首先我们需要安装 `craco` 

```shell
yarn add @craco/craco
```

同时我们需要更改 `package.json` 中的启动文件

```json
"scripts": {
  "start": "craco start",
  "build": "craco build",
  "test": "craco test",
  "eject": "react-scripts eject"
},
```

更改成 `craco` 执行

接下来我们需要在根目录下新建一个 `craco.config.js` 文件，用于配置自定义内容

```js
const CracoLessPlugin = require('craco-less');

module.exports = {
  plugins: [
    {
      plugin: CracoLessPlugin,
      options: {
        lessLoaderOptions: {
          lessOptions: {
            modifyVars: { '@primary-color': 'skyblue' },
            javascriptEnabled: true,
          },
        },
      },
    },
  ],
};
```

其实它就是用来操作 `less` 文件的全局颜色

简单的说，`antd` 组件是采用 `less` 编写的，我们需要通过重新配置的方式去更改它的值

同时我们需要将我们先前的 `App.css` 文件更改为 `App.less` 文件，在当中引入我们的 `less` 文件

```js
@import '/node_modules/antd/dist/antd.less';
```

注意一定要添加**分号结尾**，这是一个非常容易犯的错误

![image-20210907200116517](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/image-20210907200116517.png)

可见，我们成功的将主题色修改成了红色

> antd ui组件库就记这么多，还有样式的按需引入没有记录，不太喜好暴露 React 配置文件...

