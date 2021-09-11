# React 入门学习（五）-- 认识脚手架

![](https://ljcimg.oss-cn-beijing.aliyuncs.com/img/%E8%84%9A%E6%89%8B%E6%9E%B6.png)

> 📢 大家好，我是小丞同学，这篇文章是学习 React 脚手架的学习笔记
>
> 📢 非常感谢你的阅读，不对的地方欢迎指正 🙏
>
> 📢 愿你生活明朗，万物可爱

## 简介

这篇文章主要围绕 React 中的脚手架，来解决一下几个问题

**灵魂三问：是什么？为什么？怎么办？**

1. 什么是脚手架？
2. 为什么要用脚手架？
3. 怎么用脚手架？

## 🍕 1. 什么是 React 脚手架？

在我们的现实生活中，脚手架最常用的使用场景是在工地，它是为了保证施工顺利的、方便的进行而搭建的，在工地上搭建的脚手架可以帮助工人们高校的去完成工作，同时在大楼建设完成后，拆除脚手架并不会有任何的影响。

在我们的 React 项目中，脚手架的作用与之有异曲同工之妙

React 脚手架其实是一个工具帮我们快速的生成项目的工程化结构，每个项目的结构其实大致都是相同的，所以 React 给我提前的搭建好了，这也是脚手架强大之处之一，也是用 React 创建 SPA 应用的最佳方式

## 🍔 2. 为什么要用脚手架？

在前面的介绍中，我们也有了一定的认知，脚手架可以帮助我们快速的搭建一个项目结构

在我之前学习 `webpack` 的过程中，每次都需要配置 `webpack.config.js` 文件，用于配置我们项目的相关 `loader` 、`plugin`，这些操作比较复杂，但是它的重复性很高，而且在项目打包时又很有必要，那 React 脚手架就帮助我们做了这些，它不需要我们人为的去编写 `webpack` 配置文件，它将这些配置文件全部都已经提前的配置好了。

据我猜测是直接输入一行命令就能打包完成。

> 目前还没有学习到哪，本文主要讲**脚手架的项目目录结构以及安装**

## 🍟 3. 怎么用 React 脚手架？

这也是这篇文章的重点，如何去安装 React 脚手架，并且理解它其中的相关文件作用

首先介绍如何安装脚手架

### 1. 安装 React 脚手架

首先确保安装了 `npm` 和`Node`，版本不要太古老，具体是多少不大清楚，建议还是用  `npm update` 更新一下

然后打开 cmd 命令行工具，全局安装 `create-react-app`

```shell
npm i create-react-app -g
```

然后可以**新建**一个文件夹用于存放项目

在当前的文件夹下执行

```shell
create-react-app hello-react
```

**快速搭建项目**

再在生成好的 `hello-react` 文件夹中执行

```shell
npm start
```

**启动项目**

接下来我们看看这些文件都有什么作用

### 2. 脚手架项目结构

```
hello-react
├─ .gitignore               // 自动创建本地仓库
├─ package.json             // 相关配置文件
├─ public                   // 公共资源
│  ├─ favicon.ico           // 浏览器顶部的icon图标
│  ├─ index.html            // 应用的 index.html入口
│  ├─ logo192.png           // 在 manifest 中使用的logo图
│  ├─ logo512.png           // 同上
│  ├─ manifest.json         // 应用加壳的配置文件
│  └─ robots.txt            // 爬虫给协议文件
├─ src                      // 源码文件夹
│  ├─ App.css               // App组件的样式
│  ├─ App.js                // App组件
│  ├─ App.test.js           // 用于给APP做测试
│  ├─ index.css             // 样式
│  ├─ index.js              // 入口文件
│  ├─ logo.svg              // logo图
│  ├─ reportWebVitals.js    // 页面性能分析文件
│  └─ setupTests.js         // 组件单元测试文件
└─ yarn.lock
```

再介绍一下public目录下的 `index.html` 文件中的代码意思

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="theme-color" content="#000000" />
    <meta
      name="description"
      content="Web site created using create-react-app"
    />
    <link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
    <link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
    <title>React App</title>
  </head>
  <body>
    <noscript>You need to enable JavaScript to run this app.</noscript>
    <div id="root"></div>
  </body>
</html>
```

以上是删除代码注释后的全部代码

**第5行**

指定浏览器图标的路径，这里直接采用 `%PUBLIC_URL%` 原因是 `webpack` 配置好了，它代表的意思就是 `public` 文件夹

```html
<link rel="icon" href="%PUBLIC_URL%/favicon.ico" />
```

**第6行**

用于做移动端网页适配

```html
<meta name="viewport" content="width=device-width, initial-scale=1" />
```

**第七行**

用于配置安卓手机浏览器顶部颜色，兼容性不大好

```html
<meta name="theme-color" content="#000000" />
```

**8到11行**

用于描述网站信息

```html
<meta
	name="description"
    content="Web site created using create-react-app"
/>
```

**第12行**

苹果手机触摸版应用图标

```html
<link rel="apple-touch-icon" href="%PUBLIC_URL%/logo192.png" />
```

**第13行**

应用加壳时的配置文件

```html
<link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
```

> 以上就是关于 React 脚手架的全部内容了，非常感谢你的阅读💕

