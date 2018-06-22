## 一. 导论

​	vue.js是一个简单而强大的 MVVM 库。它能够帮助我们构建网站的用户界面。

​	编写这篇文章的时候，vue.js 在 github 上有 36,312 个star, 在 npm 上每月有 230250 次下载。vue.js 2.0 让虚拟DOM的操作变得更轻和更快在渲染页面时。这让服务端渲染和原生组件渲染有了更多的可能。

​	vue.js 是一个渐进式的 JavaScript 框架。 vue.js 的核心库部分非常小，但是它仍然拥有很多的辅助工具和支持库。所以你可以使用 vue.js 的生态系统来构建大型应用。

### vue.js 的内部组件

​	让我们先熟悉一下 vue.js 的内部核心组件。vue 内部包含了以下几个部分：

![Vue internal](https://camo.githubusercontent.com/20196fafa4416c0bd29faa9dcb355f7aa6152525/68747470733a2f2f6f636363336576336c2e716e73736c2e636f6d2f567565253230736f757263652532306f766572766965772e706e67) 

##### 实例生命周期（Instance lifecycel）

​	一个新的 vue 实例生成会经过几个阶段。像观察数据，初始化事件，编译模板和渲染。你可以在这些具体的阶段中注册一个钩子函数来执行。

##### 响应式系统（Reactive System）

​	响应式系统是 vue 实现视图与数据绑定的原理。当你设置 vue 实例的数据时，它的视图也会进行改变，反之亦然。

​	Vue 使用 `Object.defineProperty` 去让数据对象的属性变为响应式。通过有名的观察者模式将数据改变与视图渲染绑定在一起。

##### 虚拟DOM（Virtual DOM）

​	虚拟DOM树是使用JavaScript对象在内存中表示真实的DOM树。

​	当数据改变时，vue 会渲染出一个全新的虚拟DOM树，当然也会保存老的。通过diff算法比较两棵树的不同，然后将改变更新到真实的DOM树上。

​	Vue 以 [snabbdom](https://github.com/snabbdom/snabbdom)  作为实现虚拟DOM的基础 ，并对它进行了一些修改，从而使得它能够与 Vue 的其他组件一起工作。

##### 编译器（Compiler）

​	编译器的工作是将编译的模板转换为渲染函数（抽象语法树）。它将 HTML 和 Vue 的指令解析成为一个包含普通的HTML属性与其他实体的树。它也会检测静态子树的最大数量（没有动态绑定的子树），并在渲染的时候提取出来。

> 在这个部分我们不会详细的解释编译器具体的执行细节。自从我们可以在构建时使用构建工具去将 vue 模板编译成渲染函数，编译器就不是 vue 运行时的一部分了。我们甚至可以直接写一个渲染函数，所以编译器不是一个理解 vue 实例的必要部分。

### 搭建开发环境

​	当我们构建我们自己的 vue.js 之前，我们需要进行一些设置。包括模块包和测试工具，因为我们将使用一个 测试驱动（test-driven）的工作流程(workflow)。

​	因为这是一个 JavaScript 的项目，并且我们将要使用到一些非常棒的工具，第一件事情就是执行`npm init`命令，然后设置一些项目的信息。

	##### 安装 Rollup 作为模块打包工具

​	我们将使用 Rollup 作为模块打包工具。 Rollup 是一个 JavaScript 模块打包工具。它允许你通过 ES2015 的 import/export 语法将你的应用或者库编写成一系列的模块。Vue.js 也使用Rollup作为它的打包工具。

​	为了Rollup能够正常工作，我们需要编写一个 Rollup的配置文件。在根目录下创建一个 `rollup.conf.js`文件：

```javascript
export default {
    input: 'src/instance/index.js',
    output: {
        name: 'Vue',
        file: 'dist/vue.js',
        format: 'life'
    },
};
```

别忘了之前需要执行`npm install rollup rollup-watch --save-dev`

##### 安装 Karma 和 Jasmine 进行测试

测试需要一些新的包，执行：

```shell
npm install karma jasmine karma-jasmine karma-chrome-launcher
 karma-rollup-plugin karma-rollup-preprocessor buble  rollup-plugin-buble --save-dev
```

