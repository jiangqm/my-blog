---
title: Webapck
date: 2022-03-27 19:22:08
tags: 
---

# Webapck  

> ver 5

### 基本概念

##### 入口（entry）

​	**入口起点(entry point)** 指示 webpack 应该使用哪个模块，来作为构建其内部 [依赖图(dependency graph)](https://webpack.docschina.org/concepts/dependency-graph/) 的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。

默认值是 `./src/index.js`，但你可以通过在 [webpack configuration](https://webpack.docschina.org/configuration) 中配置 `entry` 属性，来指定一个（或多个）不同的入口起点。例如：

```typescript
module.exports = {
  entry: './path/to/my/entry/file.js',
};
```

##### 出口(output)

**output** 属性告诉 webpack 在哪里输出它所创建的 *bundle*，以及如何命名这些文件。主要输出文件的默认值是 `./dist/main.js`，其他生成文件默认放置在 `./dist` 文件夹中。

你可以通过在配置中指定一个 `output` 字段，来配置这些处理过程：

```typescript
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'), // 打包文件生成路径
    filename: 'my-first-webpack.bundle.js', // 文件名
  },
};
```

##### 转换器(loader)

webpack 只能理解 JavaScript 和 JSON 文件，这是 webpack 开箱可用的自带能力。**loader** 让 webpack 能够去处理其他类型的文件，并将它们转换为有效 [模块](https://webpack.docschina.org/concepts/modules)，以供应用程序使用，以及被添加到依赖图中

在更高层面，在 webpack 的配置中，**loader** 有两个属性：

1. `test` 属性，识别出哪些文件会被转换。
2. `use` 属性，定义出在进行转换时，应该使用哪个 loader。

```typescript
const path = require('path');
// 是
module.exports = {
  output: {
    filename: 'my-first-webpack.bundle.js',
  },
  module: {
      // 以.txt结尾的文件都用raw-loader转换一下
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
};
```

##### 插件(plugin)

loader 用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量。



想要使用一个插件，你只需要 `require()` 它，然后把它添加到 `plugins` 数组中。多数插件可以通过选项(option)自定义。你也可以在一个配置文件中因为不同目的而多次使用同一个插件，这时需要通过使用 `new` 操作符来创建一个插件实例。

```typescript
const HtmlWebpackPlugin = require('html-webpack-plugin');

const webpack = require('webpack'); // 用于访问内置插件

module.exports = {
  module: {
    rules: [{ test: /\.txt$/, use: 'raw-loader' }],
  },
  plugins: [new HtmlWebpackPlugin({ template: './src/index.html' })],
};
```

在上面的示例中，`html-webpack-plugin` 为应用程序生成一个 HTML 文件，并自动将生成的所有 bundle 注入到此文件中。

##### 模式(mode)

通过选择 `development`, `production` 或 `none` 之中的一个，来设置 `mode` 参数，你可以启用 webpack 内置在相应环境下的优化。其默认值为 `production`。

```javascript
module.exports = {
  mode: 'production',
};
```

### 入口

##### 单个入口

用法：`entry: string | [string]`

```javascript
module.exports = {
  entry: './path/to/my/entry/file.js',
};
```

上面的写法相当于

```javascript
module.exports = {
  entry: {
    main: './path/to/my/entry/file.js',
  },
};
```

多入口

我们也可以将一个文件路径数组传递给 `entry` 属性，这将创建一个所谓的 **"multi-main entry"**。在你想要一次注入多个依赖文件，并且将它们的依赖关系绘制在一个 "chunk" 中时，这种方式就很有用。

```javascript
module.exports = {
  entry: ['./src/file_1.js', './src/file_2.js'],
  output: {
    filename: 'bundle.js',
  },
};
```

##### 对象语法

用法：`entry: { <entryChunkName> string | [string] } | {}`

```javascript
module.exports = {
  entry: {
    app: './src/app.js',
    adminApp: './src/adminApp.js',
  },
};
```

对象语法会比较繁琐。然而，这是应用程序中定义入口的最可扩展的方式。

###### 描述入口的对象

用于描述入口的对象。你可以使用如下属性：

- `dependOn`: 当前入口所依赖的入口。它们必须在该入口被加载前被加载。
- `filename`: 指定要输出的文件名称。
- `import`: 启动时需加载的模块。
- `library`: 指定 library 选项，为当前 entry 构建一个 library。
- `runtime`: 运行时 chunk 的名字。如果设置了，就会创建一个新的运行时 chunk。在 webpack 5.43.0 之后可将其设为 `false` 以避免一个新的运行时 chunk。
- `publicPath`: 当该入口的输出文件在浏览器中被引用时，为它们指定一个公共 URL 地址。请查看 [output.publicPath](https://webpack.docschina.org/configuration/output/#outputpublicpath)。

```javascript
module.exports = {
  entry: {
    a2: 'dependingfile.js',
    b2: {
      dependOn: 'a2',
      import: './src/app.js',
    },
  },
};
```

tips

`runtime` 和 `dependOn` 不应在同一个入口上同时使用，所以如下配置无效，并且会抛出错误：

```javascript
module.exports = {
  entry: {
    a2: './a',
    b2: {
      runtime: 'x2',
      dependOn: 'a2',
      import: './b',
    },
  },
};
```

确保 `runtime` 不能指向已存在的入口名称，例如下面配置会抛出一个错误：

```javascript
module.exports = {
  entry: {
    a1: './a',
    b1: {
      runtime: 'a1',
      import: './b',
    },
  },
};
```

另外 `dependOn` 不能是循环引用的，下面的例子也会出现错误：

```javascript
module.exports = {
  entry: {
    a3: {
      import: './a',
      dependOn: 'b3',
    },
    b3: {
      import: './b',
      dependOn: 'a3',
    },
  },
};
```

### 出口(output)

可以通过配置 `output` 选项，告知 webpack 如何向硬盘写入编译文件。注意，即使可以存在多个 `entry` 起点，但只能指定一个 `output` 配置

```javascript
module.exports = {
  output: {
    filename: 'bundle.js',
  },
};
```

此配置将一个单独的 `bundle.js` 文件输出到 `dist` 目录中。

###### 多个入口起点

如果配置中创建出多于一个 "chunk"（例如，使用多个入口起点或使用像 CommonsChunkPlugin 这样的插件），则应该使用 [占位符(substitutions)](https://webpack.docschina.org/configuration/output#output-filename) 来确保每个文件具有唯一的名称。

```javascript
module.exports = {
  entry: {
    app: './src/app.js',
    search: './src/search.js',
  },
  output: {
    filename: '[name].js',
    path: __dirname + '/dist',
  },
};

// 写入到硬盘：./dist/app.js, ./dist/search.js
```

### Loader

在你的应用程序中，有两种使用 loader 的方式：

- [配置方式](https://webpack.docschina.org/concepts/loaders/#configuration)（推荐）：在 **webpack.config.js** 文件中指定 loader。
- [内联方式](https://webpack.docschina.org/concepts/loaders/#inline)：在每个 `import` 语句中显式指定 loader。

注意在 webpack v4 版本可以通过 CLI 使用 loader，但是在 webpack v5 中被弃用

#### 配置方式

[`module.rules`](https://webpack.docschina.org/configuration/module/#modulerules) 允许你在 webpack 配置中指定多个 loader。 这种方式是展示 loader 的一种简明方式，并且有助于使代码变得简洁和易于维护。同时让你对各个 loader 有个全局概览：

loader **从右到左（或从下到上）**地取值(evaluate)/执行(execute)。在下面的示例中，从 sass-loader 开始执行，然后继续执行 css-loader，最后以 style-loader 为结束。查看 [loader 功能](https://webpack.docschina.org/concepts/loaders/#loader-features) 章节，了解有关 loader 顺序的更多信息。

```js-with-links
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: [
          // [style-loader](/loaders/style-loader)
          { loader: 'style-loader' },
          // [css-loader](/loaders/css-loader)
          {
            loader: 'css-loader',
            options: {
              modules: true
            }
          },
          // [sass-loader](/loaders/sass-loader)
          { loader: 'sass-loader' }
        ]
      }
    ]
  }
};
```

#### 内联方式

可以在 `import` 语句或任何 [与 "import" 方法同等的引用方式](https://webpack.docschina.org/api/module-methods) 中指定 loader。使用 `!` 将资源中的 loader 分开。每个部分都会相对于当前目录解析。

```js
import Styles from 'style-loader!css-loader?modules!./styles.css';
```

#### loader特性

- loader 支持链式调用。链中的每个 loader 会将转换应用在已处理过的资源上。一组链式的 loader 将按照相反的顺序执行。链中的第一个 loader 将其结果（也就是应用过转换后的资源）传递给下一个 loader，依此类推。最后，链中的最后一个 loader，返回 webpack 所期望的 JavaScript。
- loader 可以是同步的，也可以是异步的。
- loader 运行在 Node.js 中，并且能够执行任何操作。
- loader 可以通过 `options` 对象配置（仍然支持使用 `query` 参数来设置选项，但是这种方式已被废弃）。
- 除了常见的通过 `package.json` 的 `main` 来将一个 npm 模块导出为 loader，还可以在 module.rules 中使用 `loader` 字段直接引用一个模块。
- 插件(plugin)可以为 loader 带来更多特性。
- loader 能够产生额外的任意文件。

可以通过 loader 的预处理函数，为 JavaScript 生态系统提供更多能力。用户现在可以更加灵活地引入细粒度逻辑，例如：压缩、打包、语言转译（或编译）和 [更多其他特性](https://webpack.docschina.org/loaders)。

#### 解析loader

loader 遵循标准 [模块解析](https://webpack.docschina.org/concepts/module-resolution/) 规则。多数情况下，loader 将从 [模块路径](https://webpack.docschina.org/concepts/module-resolution/#module-paths) 加载（通常是从 `npm install`, `node_modules` 进行加载）。

我们预期 loader 模块导出为一个函数，并且编写为 Node.js 兼容的 JavaScript。通常使用 npm 进行管理 loader，但是也可以将应用程序中的文件作为自定义 loader。按照约定，loader 通常被命名为 `xxx-loader`（例如 `json-loader`）。更多详细信息，请查看 [编写一个 loader](https://webpack.docschina.org/contribute/writing-a-loader/)。

#### 编写自己的loader

一个loader 的本质就是一个nodejs模块，这个模块导出一个函数，这个导出函数的工作就是**获取处理前的内容(source参数的值)**，返回处理后的内容

一个最简单的loader源码如下：

```js
module.exports = function(source){
    // 这边不做处理直接返回
    return source
}
```

由于loader是运行在nodeJs环境中，所以可以调用nodejs的API或者安装第三方插件

```js
const sass = require('node-sass')
module.exports = function(source){
    //  这边调用sass插件对资源进行转换，返回转换后的资源
    return sass(source)
}
```

webpack.config.js引入自定义loader需要`path.reslove转换下，不能直接require引入`

```javascript
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
module.exports = {
  mode: "development",
  entry: path.resolve(__dirname, "./src/main.js"), // 入口文件
  output: {
    filename: "[name].[hash:8].js", // 打包后的文件名称
    path: path.resolve(__dirname, "dist"), // 打包后的目录
  },
  module: {
      // .txt后缀的文件用自定义loader解析
    rules: [
      {
        test: /\.txt$/,
        use: [
          {loader:path.resolve("./src/loader/myloader")}
        ],
      },
    ],
  },

  plugins: [
    // 打包前删除原先的打包目录
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      //配置静态模板
      template: path.resolve(__dirname, "./public/index.html"),
    }),
  ],
};

```



### Plugin

#### 剖析

webpack **插件**是一个具有 [`apply`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/apply) 方法的 JavaScript 对象。`apply` 方法会被 webpack compiler 调用，并且在 **整个** 编译生命周期都可以访问 compiler 对象。

###### 自定义Plugin 

**ConsoleLogOnBuildWebpackPlugin.js**

```javascript
const pluginName = 'ConsoleLogOnBuildWebpackPlugin';

class ConsoleLogOnBuildWebpackPlugin {
  apply(compiler) {
    compiler.hooks.run.tap(pluginName, (compilation) => {
      console.log('webpack 构建正在启动！');
    });
  }
}

module.exports = ConsoleLogOnBuildWebpackPlugin;
```

compiler hook 的 tap 方法的第一个参数，应该是驼峰式命名的插件名称。建议为此使用一个常量，以便它可以在所有 hook 中重复使用。



