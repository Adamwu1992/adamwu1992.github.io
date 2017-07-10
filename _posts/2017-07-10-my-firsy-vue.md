---
layout: post
title:  "我的第一个vue"
date:   2017-07-10 10:00:56 +0800
categories: jekyll update
---

> 原本票无忧是一个miniui+jQuery项目，现在用vue2.0改造成一个SPA，作为我的vue入门练手工程。本工程采用vue-cli脚手架构建，所有配置都采用默认设置，只关注程序实现部分。

## 构建步骤

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run e2e tests
npm run e2e

# run all tests
npm test
```

## 目录结构

&emsp;&emsp;所有改动都在src目录下，其余和模版保持一致
src
&nbsp;|——asserts //图片等资源文件目录
&nbsp;|——components //全部组件
&nbsp;|——pages //业务相关页面
&nbsp;|——router //路由
&nbsp;——App.vue //根组件
&nbsp;——main.js //入口文件

## 实现细节

> 刚开始做的时候没有想到用UI框架，等到自己写到弹出框和下拉框的时候菜感到有些吃力。但是一想自己写组件正好可以起到锻炼臂力的功效就坚持没有引入第三方的UI框架。第一代的组件都是为本项目量身定制的，耦合度较高，比如一些宽高都是直接在组件里写死的，以后将逐步抽象出来。

### 组件

![hello world](../screenshots/overview.png)

&emsp;&emsp;首页主要是写在App.vue里，分为header和main两个部分，main里面分为左侧的menu和右侧的router-view。
&emsp;&emsp;header里包含一个logo，一个选择企业的下拉列表（ListBox），一个用户信息展示（PopupWindow）和按钮（ButtonGroup），除了logo其他都是用组件的方式实现。

#### ListBox

&emsp;&emsp;接受参数list作为显示的内容，同时可以自定义textField和valueField的属性名，默认是text和value，用chosen属性表示默认被选中的对象，否则默认选中第一个。
&emsp;&emsp;在组件的mouted钩子里实现对弹出层的定位，所以弹出层智能用v-show而不能用v-if，因为v-if是懒加载，如果默认是false时在mouted中获取不到弹出层对象。

#### ButtonGroup

&emsp;&emsp;比较简单，主要是接受一些参数控制按钮是否显示，在按钮被点击是发射相应的事件，在父组件接收并处理。

#### PopupWindow

&emsp;&emsp;使用了两个插槽trigger和popup，一个放置触发区域，一个放置弹出区域，在本处trigger中方的是当前的用户名，popup中放的是用户菜单，弹出层popup的定位用的和ListBox是同样的方式

#### Navigator

&emsp;&emsp;接受一个数组，每个元素都渲染成一个router-link，这个插件比较简单，没什么好抽象的。

#### OpenedWindow

&emsp;&emsp;这个是弹出窗口组件，由一个header和一个插槽组成，header里可以用参数控制显示title和最大化关闭等按钮，此处按钮引用的是ButtonGroup组件，也可以用参数控制隐藏header。插槽里放弹出窗口的内容，在本例中的放置的是一个router-view

#### MessageBox

&emsp;&emsp;提示信息组件，接受三个参数：提示等级（level）、提示内容（message）、显示时间（delay），在created钩子里会创建一个定时器，在delay到时会向父组件发送事件隐藏本组件。