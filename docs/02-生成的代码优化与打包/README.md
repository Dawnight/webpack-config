# 1. css 提取

+ 我们现在打包，是把 css 样式直接打包进 bundle.js 文件中，但是我们想要把 css 提取出来，与 js 代码分离

+ 使用 mini-css-extract-plugin 这个插件， `npm i mini-css-extract-plugin -D`，这个插件就是用来帮助我们抽离所有的 css 代码

+ 在 plugins 项里配置使用，filename 的打包生成的 css 文件的文件名

```javascript
const MiniCSSExtraPlugin = require('mini-css-extract-plugin');

module.exports = {
  plugins: [
    new MiniCSSExtraPlugin({
      filename: 'main.css'
    })
  ]
};
```

+ 这样仅仅是定义了打包输出的 css 文件名，但是具体我们要把这个插件用在那个 css 或预处理器上还不能确定，所以我们要把 MiniCSSExtraPlugin 用在 rules 的 css 配置里

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        loader: [
          MiniCSSExtraPlugin.loader,
          'css-loader',
          'postcss-loader'
        ],
        exclude: /node_modules/
      },
      {
        test: /\.less$/,
        loader: [
          MiniCSSExtraPlugin.loader,
          'css-loader',
          'postcss-loader',
          'less-loader'
        ],
        exclude: /node_modules/
      },
      {
        test: /\.styl$/,
        loader: [
          MiniCSSExtraPlugin.loader,
          'css-loader',
          'postcss-loader',
          'stylus-loader'
        ],
        exclude: /node_modules/
      },
      {
        test: /\.scss$/,
        loader: [
          MiniCSSExtraPlugin.loader,
          'css-loader',
          'postcss-loader',
          'sass-loader'
        ],
        exclude: /node_modules/
      }
    ]
  }
};
```

+ 这里是把 css 及各个预处理器里的代码，统一使用 MiniCSSExtraPlugin 来处理，最终生成一个 main.css 文件

+ 注意使用的顺序，一定要在卸载 css-loader 的前边，因为需要各个 loader 处理结束后，才能生成文件

+ 但是，如果我们想要针对不同的 css 及各个预处理器生成各自的 css 代码，那么我们可以针对各个预处理器，进行单独的配置。方法就是定义多个 MiniCSSExtraPlugin，在 plugins 里使用的时候，也要使用多个，而对应的预处理器里的 MiniCSSExtraPlugin 也必须是对应的

+ 下边的配置就实现了配置多个 MiniCSSExtraPlugin，这个根据需要，如果 css 样式不多，可以直接生成到一个文件里

```javascript
const MiniCSSExtraPlugin = require('mini-css-extract-plugin');
const MiniCSSExtraPluginLess = require('mini-css-extract-plugin');

module.exports = {
  plugins: [
    new MiniCSSExtraPlugin({
      filename: 'main.css'
    }),
    new MiniCSSExtraPluginLess({
      filename: 'main.less.css'
    })
  ],
  module: {
    rules: [
      {
        test: /\.css$/,
        loader: [
          MiniCSSExtraPlugin.loader,
          'css-loader',
          'postcss-loader'
        ],
        exclude: /node_modules/
      },
      {
        test: /\.less$/,
        loader: [
          MiniCSSExtraPluginLess.loader,
          'css-loader',
          'postcss-loader',
          'less-loader'
        ],
        exclude: /node_modules/
      }
    ]
  }
};
```

# 2. css 与 js 的压缩

+ webpack 有一个专门的 optimization 的配置项，专门用来优化代码，但是使用的时候需要注意，只有模式是在 development 的情况下才会生效

+ css 压缩，需要使用一个依赖包，`npm i optimize-css-assets-webpack-plugin -D`

+ js 压缩，需要使用一个依赖包，`npm i uglifyjs-webpack-plugin -D`

+ 使用的时候比较简单，直接在 plugins 里使用就可以

```javascript
// 优化打包的 css 代码
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
// 优化打包后的 JS 代码
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');

module.exports = {
  optimization: {
    minimizer: [
      new OptimizeCSSAssetsPlugin(),
      new UglifyJSPlugin({
        // 使用缓存
        cache: true,
        // 并发
        parallel: true,
        // 生成 sourceMap 文件
        sourceMap: true
      })
    ]
  }
}
```

# 3. babel 的配置

+ babel 配置有多种，[文档](https://babeljs.io/docs/en/configuration)

+ 一般常用的有 .babrlrc， babel.config.js， babelrc.js， package.json

+ 还有一种使用方式是 webpack 的使用方式，直接在 rule 里处理 js 或 jsx 的 option 里配置

+ 我们采用 .babelrc 的方式配置，根目录下新建一个 .babelrc 的文件，写入下边的内容

```json
{
  "presets": ["@babel/preset-env", "@babel/preset-react"],
  "plugins": [
    [
      "@babel/plugin-proposal-decorators",
      {
        "legacy": true
      }
    ],
    [
      "@babel/plugin-proposal-class-properties",
      {
        "loose": true
      }
    ],
    [
      "@babel/plugin-transform-runtime",
      {
        "builtins": "usage",
        "corejs": {
          "version": 3
        }
      }
    ],
    "@babel/plugin-syntax-dynamic-import"
  ]
}
```

+ presets 的意思是预设，意思就是官方提供的必须要使用的包

+ plugins 的意思是插件，比如类装饰器，类的属性等高级语法，@babel/core 和 @babel/preset-env 没有提供编译，需要采用第三方插件，这里的两个插件，一个是装饰器，一个是类属性

+ 关于 @babel/polyfill 和 @babel/plugin-transform-runtime
	+ @babel/polyfill 是一个垫片，Babel 不转化 JS 新的 API，比如 Iterator、Generator、Set、Maps 等全局对象，如果我们要使用这些新的对象和方法，就需要使用 @babel/polyfill
	+ @babel/plugin-transform-runtime 是一个辅助函数库，因为 babel 编译后的代码，要实现和源代码一样的功能，需要一些函数库，所以需要使用 @babel/plugin-transform-runtime，而使用这个库还需要 @babel/runtime 库，目前来说，polyfill 已经被废弃掉了，现在直接使用 @babel/plugin-transform-runtime 即可，还需要 @babel/runtime 和 @babel/runtime-corejs3 这两个库。

# 4. eslint 的配置

+ eslint 的目的是为了提高代码的书写规范，提高代码质量

+ eslint 需要使用到 eslint 和 eslint-loader 两个包，`npm i eslint eslint-loader -D`

+ 在 webpack 里配置 loader，这里的 enforce 的意思是，要在其他的 loader 解析之前先校验 JS 代码

```javascript
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        use: {
          loader: 'eslint-loader',
          options: {
            enforce: 'pre'
          }
        }
      }
    ]
  }
};
```

+ 根目录下新建一个 .eslintrc 的文件，具体每一条规范是什么意思，如何使用，可以直接查看[官网](https://eslint.org)

```json
{
  "parserOptions": {
    "ecmaVersion": 5,
    "sourceType": "script",
    "ecmaFeatures": {}
  },
  "rules": {
    "constructor-super": 2,
    "for-direction": 2,
    "getter-return": 2,
    "no-case-declarations": 2,
    "no-class-assign": 2,
    "no-compare-neg-zero": 2,
    "no-cond-assign": 2,
    "no-console": 2
  },
  "env": {
    "browser": true,
    "commonjs": true,
    "es6": true
  }
}
```

+ 完整的代码可以查看 02-webpack.config.js
