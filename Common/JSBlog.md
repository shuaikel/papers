---
path: "/blog/my-first-post"
date: "2017-11-07"
title: "My first blog post"
---

###javaScript 移动端
#####JS移动端解决方案：
>- React: [React Native](https://reactnative.cn/)
>
>- Vue：[Weex](https://weex.apache.org/) 和 [Quasar](http://quasar-framework.org/)
>
>- Angular：[Ionic](https://ionicframework.com/) 和 [NativeScript](https://www.nativescript.org/)
>

#####编译工具：Compilers：
![](https://user-gold-cdn.xitu.io/2018/1/20/16113a2fe2cf2e30?imageslim)

这里我们将讨论那些编译到标准 JavaScript 代码的语言。
通常情况下，在搭建自己的构建工作流时需要编译器可能有 2 个原因：
>- 想立即使用最新的 JavaScript 语言特性，并把它应用到尽可能多的浏览器中，这类需求让 [Babel](https://babeljs.io/) 获得了成功，很多项目都依赖于它；
>- 想为语言添加新的特性，比如“类型检查”；

对 Javascript 程序员进行分类有个热门的问题是：你是用类型的，还是不用类型的？

JavaScript 本身带有基本的动态类型，但缺乏静态类型。而很多开发者倾向于在代码中使用类型，尤其在大型项目中，因为这样可以让代码变得更为健壮且易于阅读和理解。

如果你需要类型，有两个主流可选项：微软的 [TypeScript](http://www.typescriptlang.org/) 和 Facebook 的 [Flow](https://flow.org/)（Facebook 在自己的主要项目 React，React Native，Jest 中都有使用);

你可以从 [James Kyle](https://github.com/thejameskyle) 的文章来感受两者的区别: A Comparison Between Adopting Flow or TypeScript。


#####构建工具：Build Tools
![](https://user-gold-cdn.xitu.io/2018/1/20/16113a32c157f9f4?imageslim)

构建工具分类中的排行冠军是 [Parcel](https://parceljs.org/)，这或许是今年最大的惊喜，作为一个 8 月份才在 GitHub 上发布的新项目却已达到 14K 个 star 的关注度。

Parcel 不仅提供现代前端开发所需的各种功能，还有个碾压性的优势：零配置！这是它与依靠大量 "loaders" 的 [Webpack](https://webpack.js.org/) 最大区别。

别误解数字，Webpack 依然是最流行的构建应用，它在 GitHub 上 有 35K 的 star 和超 500 人的贡献者。目前有许多项目使用了它，包括今年最流行的两个项目：Create React App 和 [Gatsby](https://www.gatsbyjs.org/)。

Webpack 不断在迭代更新，2.0 版本可以让开发者通过动态加载的方式轻松实现“代码分割”的功能。

Webpack 与 Parcel 同时定位于构建 WEB 应用，而 [Rollup](https://rollupjs.org/guide/en) 则定位于库的构建，它专注于 ES6 模块的性能提升上。

#####测试框架
![](https://user-gold-cdn.xitu.io/2018/1/20/16113a352f914ec6?imageslim)

Jest 最初是 Facebook 因为 React 组件测试目的而开发的，但最近几个月革命性的版本变更（发布了 22 个大版本）使得它现在能同时用于测试前端、后端代码。

Jest 有几个大的闪光点：
>- 无需配置，默认地设定已经满足通常需求；
>
>- 强大的开发者体验 (智能观察模式，直观的错误报告)；
>
>- 语法上与 Mocha 很接近，许多程序员熟悉 describe 和 it 这样的关键词；
>
>- 不需要额外的库来创建 assertions，已全部内置；
>
>- 独特的"快照"模式可以作为重新运行测试时的对比基准；

AVA，去年的第一名，仍然有许多吸引人的特点。
这个项目由 [Sindre Sorhus](https://github.com/sindresorhus) 创建并在他所有的项目中使用，熟悉他的同学肯定知道这意味着什么！
相较于 Jest，AVA 更侧重于并行测试上的速度，更轻量，也更接近测试标准，语法上与测试框架 Tape 接近。

#####IDE 和编辑器
![](https://user-gold-cdn.xitu.io/2018/1/20/16113a3801d8c17e?imageslim)

VS Code 都会发布新版本，带来更多实用 IDE 功能同时性能上却没有太大的损耗。即使不安装任何插件，你也有一大堆开箱即用的功能：
>- Git 集成功能；
>
>- 自动补全：JavaScript 语法，文件路径进行补全，npm 包名字等等；
>
>- React 语法集成等；
>
此外，你可以在编辑器中添加 Prettier 插件，这样每次保存时它都会自动格式化文件，在这样的编程环境下编码真是一种享受。


#####CSS in JavaScript

![](https://user-gold-cdn.xitu.io/2018/1/20/16113a3ad1d3cd34?imageslim)

目前 React 社区仍然没有就如何有效管理组件样式这个问题达成共识，即没有标准的解决方案。

如果无需太多自定义的标准样式，可以用 [Material UI](http://www.material-ui.com/) 或 [Ant Design](https://ant.design/index-cn) 这样现成的组件工具包。如果需要更高度灵活的自定义，你仍然能使用传统方式：用一个像 Bootstrap 或 Bulma 这样的全局 CSS 样式，通过修改组件的 className 属性来达到目的。这样做缺点是组件无法进行自我样式管理，因为样式分布在单独的文件中。

CSS in JavaScript 概念的出现即是为了解决上述问题。

概念本身很简单：既然我们在 React 中己能通过 JavaScript 来同时控制逻辑和模板部分，何不再进一步，连样式也一并管理了呢？

[Styled Components](https://www.styled-components.com/) 是今年本类别的冠军，它利用 JavasScript 最近新加入的模板字符串特性，让开发者在 React 组件中直接使用标准的 CSS 语法编写样式。

[CSS Modules](https://github.com/css-modules/css-modules)，作为本类别的亚军，则采用了混合的解决方案。它让开发者自己挑选诸如标准 CSS, SASS, NO slug Less, NO slug Stylus等方式编写样式，再以文件的方式导入到组件中。

#####静态网站生成器
![](https://user-gold-cdn.xitu.io/2018/1/20/16113a3d9a8a21f2?imageslim)

静态网站生成器（SSG，Static Site Generator）是指能够生成一坨 HTML、CSS、JS 文件，方便你快速部署到 WEB 服务器上而不需要安装和配置数据库的工具。
静态网站具有速度快，稳定且易于维护的特点。作为 2016 年的亚军，[Gatsby](https://www.gatsbyjs.org/) 今年成功拨得头筹。它新增了许多新功能来助你优化静态网站：
>- 快速浏览和导出速度；
>
>- 主动预加载机能；
>
>- 智能代码分解 (模板 + 网页数据)；
>

Gatsby 使用 React 来做视图层(View Layer)，构建时候则用 GraphQL 来查询内容。它有一个强大的社区并且 React 官网也是用 Gatsby 的来搭建的.

[React Static](https://github.com/nozzle/react-static) 是本类别的新面孔。它从 Create React App 项目中获得灵感，定位于做一个轻量的 Gatsby 替代方案，专注于性能和简洁。

此外，值得一提的是 [Next.js](https://zeit.co/blog/next4) 也能当静态网站生成器来用。

#####GraphQL

![](https://user-gold-cdn.xitu.io/2018/1/20/16113a406527287f?imageslim)


















