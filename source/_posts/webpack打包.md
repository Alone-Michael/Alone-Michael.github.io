---
title: webpack打包
tags: Eric的博客
date: 2023-06-02
author: Eric
comments: false
cover: /img/31.jpg
index_enable: true #是否显示文章封面
aside_enable: true #侧栏是否显示文章封面图s
archives_enable: true 
position: both #封面显示的位置# 三个值可配置left , right , both 
---

学习webpack

# 约定loader一览表

| 扩展名 |  语义              | loader举例      |
| :---  |    :----           |    :---         |
| .js   | returns module exports    | babel-loader
| .ts  | returns module exports |ts-loader
| .coffee |returns module exports |coffee-loader coffee-redux-loader
| .jsx |returns module exporcomponent) | jsx-loader react-hot-loader!jsx-loader
| .json .json5 |returns json value |json-loader json5-loader
|.txt |return string value |raw-loader
|.css |returns nothing, side effect oadding style to DOM | loader!css-lostyle-loader!css-loloaderader!autoprefixer-ader style-
|.less |returns nothing, side effect oadding style to DOM | style-loader!css-loloader ader!less-
|.scss |returns nothing, side effect oadding style to DOM |style-loader!css-loloaderader!scss-
|.styl |returns nothing, side effect oadding style to DOM |style-loader!css-loloaderader!stylus-
|.png .jpg .jpeg .gif .svg |returns url to image |file-loader url-loader
|.woff .ttf |returns url to font |file-loader url-loader
|.wav .mp3 |returns url to audio |file-loader url-loader
|.mpeg .mp4 .webm .ogv |returns url to video |file-loader
|.html |returns HTML as string |html-loader
|.md .markdown |returns HTML as string |html-loader!markdown-loader
|.jade |returns template function |jade-loader
|.hbs .handlebars |returns template function |handlebars-loader

# loader列表

## 基本

- json : 把文件加载为JSON

- hson : 把HanSON 文件（JSON for Humans）加载为JSON对象

- raw : 把文件加载为纯文本（utf-8）

- val : 把代码当作模块来执行，并将出口看作是javascript代码

- to-string : 把代码当作模块来执行，转换输出字符串并将其导出

- imports : 引入东西到模块

- exports : 从模块导出东西

- expose : 把模块出口暴露给全局上下文

- script : 在全局上下文执行一个javascript文件（像script标签一样）, requires不被解析

- apply : 执行导出的可带参数的JavaScript函数，输出它的返回值

- callback : 分析JS，调用指定的函数（你在webpack上下文中执行的）并用结果替换他们

- if-loader : 这是预处理程序为webpack模块打捆。它支持if指令,像C语言的 #ifdef

- source-map : 从模块中提取sourceMappingURL 注释,并把它提供给webpack

- checksum : 计算文件的校验

- null : 发出空模块

- cowsay : 发射一个带cowsay头的模块

- dsv : 加载csv、tsv文件

- glsl : 加载glsl文件，支持glsl-chunks

- render-placement : 添加react，为你渲染 组件（大多数情况下不实用）

- xml : 把一个xml文件加载为JSON

- svg-react : 把SVG文件加载为JSX文件。React.createClass声名类

- svg-url : 把SVG文件加载为utf-8编码数据：URI string

- svg-as-symbol : 把svg源文件根目录下的元素内容包裹在symbol元素里，返回结果标记

- base64 : 加载文件的内容为base64字符串

- ng-annotate : 给angular应用的依赖注入编号的Loader

- node : 加载用node-gyp生成的.node文件

- required : 加载整个目录树。加载js,css和它们里面的其它东西

- icons : 从svg文件生成字体

- block-loader : 基于内容start/end分隔符 ,只重写文件的一部分的通用Loader

- bundler-configuration : 把配置文件加到打包结果文件中的一个工具

- console : 在控制台打印解析webpack required的 resolved

- solc : 编译 Solidity 代码(.sol)，返回带有应用程序二进制结口和字节编码的js对象，为部署到

- Ethereum作准备

- web3 : 部署Ethereum虚拟机字节码，返回部署智能协议的ready-to-use 的js实例，还返回Web3初始化对象

## 打包用

- file 发出文件到输出文件夹。并返回相对路径

- url url-loader的工作原理跟 file-loader很像。但是如果文件小于一个阈值，它能返回一个Data url

- extract 为提取文件准备 HTML和css模块

- worker 这个loader给预备文件创建一个webworker

- shared-worker 跟 worker差不多，但是是为 shared worker准备的

- serviceworker 跟 worker差不多，但是是为 service worker准备的

### 配置参数

1. Entry：入口起点指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的

2. Output：output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件，默认值为 ./dist。基本上，整个应用程序结构，都会被编译到你指定的输出路径的文件夹中。你可以通过在配置中指定一个 output 字段，来配置这些处理过程
3. Loader：loader 让 webpack 能够去处理那些非 JavaScript 文件（webpack 自身只理解 JavaScript）loader 可以将所有类型的文件转换为 webpack 能够处理的有效模块 ，然后你就可以利用 webpack 的打包能力，对它们进行处理
4. Plugins 插件：loader 被用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。插件的范围包括，从打包优化和压缩，一直到重新定义环境中的变量。 插件接口功能极其强大，可以用来处理各种各样的任务。想要使用一个插件，你只需要 require() 它，然后把它添加到 plugins 数组中多数插件可以通过选项(opon)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 new 操作符来创建它的一个实例
5. NoParse ：noParse 是 webpack 的另一个很有用的配置项，如果你确定一个模块中没有其它新的依赖 就可以配置这项，webpack 将不再扫描这个文件中的依赖模块

```js
 // 使用正则表达式
 noParse: /jquery|chartjs/
 // 使用函数，从 Webpack 3.0.0 开始支持
 noParse: (content)=> {
 // content 代表一个模块的文件路径
 // 返回 true or false
    return /jquery|chartjs/.test(content);
  }
```

6. Resolve：Webpack 在启动后会从配置的入口模块出发找出所有依赖的模块，Resolve 配置 Webpack如何寻找模块所对应的文件。 Webpack 内置 JavaScript 模块化语法解析功能，默认会采用模块化标准里约定好的规则去寻找，但你也可以根据自己的需要修改默认的规则

7. parser：因为 Webpack 是以模块化的 JavaScript 文件为入口，所以内置了对模块化 JavaScript 的解析功能，支持 AMD、CommonJS、SystemJS、ES6。 parser 属性可以更细粒度的配置哪些模块语法要解析哪些不解析，和 noParse 配置项的区别在于 parser 可以精确到语法层面， 而 noParse 只能控制哪些文件不被解析。 parser 使用如下

```js
module: {
rules: [
  {
    est: /\.js$/,
    se: ['babel-loader'],
    arser: {
      md: false, // 禁用 AMD
      ommonjs: false, // 禁用 CommonJS
      ystem: false, // 禁用 SystemJS
      harmony: false, // 禁用 ES6 import/export
      requireInclude: false, // 禁用 require.include
      requireEnsure: false, // 禁用 require.ensure
      requireContext: false, // 禁用 require.context
      browserify: false, // 禁用 browserify
      requireJs: false, // 禁用 requirejs
      }
    },
  ]
}
```

### 配置 Loader

rules 配置模块的读取和解析规则，通常用来配置 Loader。其类型是一个数组，数组里每一项都描述了如何去处理部分文件。 配置一项 rules 时大致通过以下方式：

1. 条件匹配：通过 test 、 include 、 exclude 三个配置项来命中 Loader 要应用规则的文件。
2. 应用规则：对选中后的文件通过 use 配置项来应用 Loader，可以只应用一个 Loader 或者按照从后往前的顺序应用一组
Loader，同时还可以分别给 Loader 传入参数。
3. 重置顺序：一组 Loader 的执行顺序默认是从右到左执行，通过 enforce 选项可以让其中一个 Loader 的执行顺序放到最
前或者最后。
下面来通过一个例子来说明具体使用方法：

```js
module: {
rules: [
  {
    // 命中 JavaScript 文件
    est: /\.js$/,
      // 用 babel-loader 转换 JavaScript 文件
      // ?cacheDirectory 表示传给 babel-loader 的参数，用于缓存 babel 编译结果加快重新编译速度
      se: ['babel-loader?cacheDirectory'],
      // 只命中src目录里的js文件，加快 Webpack 搜索速度
      include: path.resolve(__dirname, 'src')
    },
    {
      // 命中 SCSS 文件
      test: /\.scss$/,
      // 使用一组 Loader 去处理 SCSS 文件。
      // 处理顺序为从后到前，即先交给 sass-loader 处理，再把结果交给 css-loader 最后再给 style
      use: ['style-loader', 'css-loader', 'sass-loader'],
      // 排除 node_modules 目录下的文件
      exclude: path.resolve(__dirname, 'node_modules'),
    },
    {
      // 对非文本文件采用 file-loader 加载
      test: /\.(gif|png|jpe?g|eot|woff|ttf|svg|pdf)$/,
      use: ['file-loader'],
    },
  ]
}
```

在 Loader 需要传入很多参数时，你还可以通过一个 Object 来描述，例如在上面的 babel-loader 配置中有如下代码：

```js
  use: [
    {
      loader:'babel-loader',
      options:{
        cacheDirectory:true,
      },
      // enforce:'post' 的含义是把该 Loader 的执行顺序放到最后
      // enforce 的值还可以是 pre，代表把 Loader 的执行顺序放到最前面
      enforce:'post'
    },
    // 省略其它 Loader
  ]
```

上面的例子中 test include exclude 这三个命中文件的配置项只传入了一个字符串或正则，其实它们还都支持数组类型，
使用如下：

```js
{
  test:[
    /\.jsx?$/,
    /\.tsx?$/
  ],
  include:[
    path.resolve(__dirname, 'src'),
    path.resolve(__dirname, 'tests'),
  ],
  exclude:[
    path.resolve(__dirname, 'node_modules'),
    path.resolve(__dirname, 'bower_modules'),
  ]
}
```
