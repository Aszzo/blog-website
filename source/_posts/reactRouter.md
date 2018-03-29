---
layout: '[layout]'
title: react-router 4.x的异步组件加载（Code Splitting）
date: 2017-11-15 14:50:38
comments: true
toc: true
tags:
	- 前端
	- react
---
在`react-router2.0`时代，实现组件异步加载主要依靠`getComponent`,按照下面的方法便可实现对组件的异步加载:
```js
<Route path={"home"}  getComponent={function (nextState,callback) {
            require.ensure([], function (require) {
                callback(null,require('../Component/Home'));
            });
}}/>
```
在`react-router4.x`中移除了`getComponent`方法，下面将介绍如何在基于`Create-React-App`搭建的项目中使用高阶组件`AsyncComponent`结合实现组件的异步加载。

在react应用中，基本的路由配置可能像下面这样：
<!-- more -->
```js
/*引入组件 */
import Home from './containers/Home';
import Posts from './containers/Posts';
import NotFound from './containers/NotFound';


/* 配置路由 */
export default () => (
  <Switch>
    <Route path="/" exact component={Home} />
    <Route path="/posts/:id" exact component={Posts} />
    <Route component={NotFound} />
  </Switch>
);
```
将我们需要的组件引入进来，然后定义好路由，页面会根据我们定义的路由规则去匹配相应的组件。但是这么做有一个问题，我们在一开始就把所有用到的组件引入进来了，这就意味着不管匹配哪个路由，所有的组件都会被加载。对于较小的项目这个问题可能不大明显，但是对于比较复杂的项目，拥有少则十几二十几个页面，多则好几十个，上百个页面，一开始就要把所有的资源加载完，如果服务器配置又不是那么高，on my god...不敢想象用户访问的时候会不会急出尿来#。#

下面我们一步步实现组件的按需加载：

<b>创建异步组件</b>

首先要创建一个异步加载的组件`AsyncComponent.js`：
```js
import React, { Component } from "react";

export default function asyncComponent(importComponent) {
  class AsyncComponent extends Component {
    constructor(props) {
      super(props);

      this.state = {
        component: null
      };
    }

    async componentDidMount() {
      const { default: component } = await importComponent();

      this.setState({
        component: component
      });
    }

    render() {
      const C = this.state.component;

      return C ? <C {...this.props} /> : null;
    }
  }

  return AsyncComponent;
}
```
说明：

1.这个`asyncComponent` 函数接受一个`importComponent` 的参数，`importComponent` 调用时候将动态引入给定的组件。

2.在`componentDidMount` 我们只是简单地调用`importComponent` 函数，并将动态加载的组件保存在状态中。

3.最后，如果完成渲染，我们有条件地提供组件。在这里我们如果不写`null`的话，也可提供一个菊花图，代表着组件正在渲染。

<b>使用异步组件</b>

对于需要加载的组件，我们应该通过异步组件来引入，而不是像一开始一样在开始就静态引入
```js
import Home from '../Component/Home'
```
使用`asyncComponent`来引入我们的组件

首先把我们的异步加载的功能组件引入进来：
```js
import asyncComponent from './AsyncComponent
```
```js
//引入我们需要加载的组件
const AsyncHome = asyncComponent(() => import('../Component/Home'));
```
```js
//在路由中使用异步加载的组件
<Route path="/" exact component={AsyncHome} />
```
大功告成，这样子组件就可以通过异步加载的方式来进行按需加载了。
贴一个`Routes.js`的配置来具体的体会一下
```js
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import asyncComponent from './Bundle';//动态加载组件，用作代码分割
import { Route,Switch } from 'react-router-dom';

const Home = asyncComponent(() => import("./containers/home/Home"));
const Service = asyncComponent(() => import("./containers/service/Service"));
const System = asyncComponent(() => import("./containers/system/Syetem"));
const Advisory = asyncComponent(() => import("./containers/advisory/Advisory"));
const Product = asyncComponent(() => import("./containers/product/Product"));
const Enterprise = asyncComponent(() => import("./containers/solution/enterprise/Enterprise"));
const Insurance = asyncComponent(() => import("./containers/solution/insurance/Insurance"));
const Hospital = asyncComponent(() => import("./containers/solution/hospital/Hospital"));
const Contact = asyncComponent(() => import("./containers/contact/Contact"));
const Best = asyncComponent(() => import("./containers/best/Best"));
const Details = asyncComponent(() => import("./containers/advisory/details/Details"));
const Example = asyncComponent(() => import('./containers/advisory/source/example/Example'));
ReactDOM.render(
    <Router>
        <Switch>
            <Route exact path="/" component={Home}/>
            <Route path="/service" component={Service}/>
            <Route path="/system" component={System}/>
            <Route path="/advisory" component={Advisory}/>
            <Route path="/product" component={Product}/>
            <Route path="/enterprise" component={Enterprise}/>
            <Route path="/insurance" component={Insurance}/>
            <Route path="/hospital" component={Hospital}/>
            <Route path="/contact" component={Contact}/>
            <Route path="/best" component={Best}/>
            <Route path="/details" component={Details}/>
            <Route path="/example" component={Example}/>
        </Switch>
    </Router>
    , document.getElementById('root'));
```

![](/images/reactRouter/code.png)

下面是部署好的在网站的真实截图

![](/images/reactRouter/load.jpg)
眼尖的同志可能会发现，虽然资源是进行异步加载了，但是怎么加载的资源有点大呢？这是因为没有对资源文件进行`gzip`压缩。有关`gzip`压缩的介绍可以参考本站文章：<a href='/2017/10/20/nginx-gzip/'>Nginx开启和配置gzip压缩</a>

该项目部署在`nginx`搭建的服务器中，在`nginx`开启`gzip`功能修改配置文件`nginx.conf`,增加如下代码:
```js    gzip  on;
       gzip_comp_level 4;
       gzip_buffers 4 16k;
       gzip_min_length 1k;
       gzip_vary on;
       gzip_types *;
```
下面贴一张经过gzip压缩后的网站请求资源的截图来作为结束吧。

![](/images/reactRouter/loadGzip.jpg)





