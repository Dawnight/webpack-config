# 1. webpack 的小插件

## 1.1 删除某个目录

+ 我们每次打包之前，就想要删掉原来打包的目录里的文件，我们可以使用一个插件 `clean-webpack-plugin`，注意版本，不同的版本使用方法是不一样的

+ 使用比较简单，直接在 plugins 里添加即可
  + 1.x 版本使用，`new CleanWebpackPlugin([path.resolve(__dirname, 'dist')])`
  + 2.x 版本使用， `new CleanWebpackPlugin()`，不添加参数，默认是删除打包生成的目录

## 1.2 拷贝某个目录

+ 把某个目录拷贝到特定的目录，比如从(from) src/public 文件夹复制到(to) dist/public 文件夹下

+ 参数是一个数据，表示可以接收多个对象元素，拷贝多个目录

```javascript
module.exports = {
  plugins: [
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, './src/public'),
        to: path.resolve(__dirname, './dist/public')
      }
    ])
  ]
};
```

## 1.3 banner

+ 这个插件的目的是在打包生成的 js 文件顶部，添加一行注释，可以是版本，作者，时间信息等

```javascript
module.exports = {
  plugins: [
    new Webpack.BannerPlugin('made by carl')
  ]
};
```


## 1.4 更新

+ 热更新插件，new Webpack.HotModuleReplacementPlugin()

+ 更新的模块路径，new Webpack.NamedModulesPlugin()

```javascript
module.exports = {
  plugins: [
    // 热更新插件
    new Webpack.HotModuleReplacementPlugin(),
    // 打印更新的模块路径
    new Webpack.NamedModulesPlugin(),
  ]
};
```


# 2. 多页面配置

+ 多个页面，就要有多个入口，多个 HTML 模板，同样要有多个出口

+ 我们看一下 webpack 如何配置多个页面

```javascript
module.exports = {
  entry: {
    home: path.resolve(__dirname, 'src/index.js'),
    other: path.resolve(__dirname, 'src/other.js'),
  },
  output: {
    filename: "[name].js",
    path: path.resolve(__dirname, 'dist')
  },
  plugins: [
    new HTMLPlugin({
      template: './src/index.html',
      filename: 'home.html',
      chunks: ['home']
    }),
    new HTMLPlugin({
      template: './src/index.html',
      filename: 'other.html',
      chunks: ['other']
    })
  ]
}
```

+ 解释一下
  + 多个 js 入口，entry 需要为一个对象，不同的 key 对应不同的代码 chunk，注意不是页面
  + 多个 js 出口，出口就不能写死，需要使用变量的形式，使用 `[name].js`，这里的 name 对应入口里的 key
  + 因为有多个页面，每个页面可能有不同的模板内容，所以我们使用多个 HTMLPlugin，同时打包出的 HTML 文件名也不能一样，每个页面需要使用到的代码块也不一样，所以需要给每一个页面配置一个 chunks，因为页面可能会需要多个代码块，所以 chunks 是一个数组，每一个元素是入口里的 key，这里需要一一对应，不然是不生效的

+ 区分 chunk，module
  + chunk 是一个代码块，有一个 chunkId，这个 chunk 可能包含多个模块，比如 pageA 页面需要的是 A，B，C 模块，那么此时的 chunk 就包含了 A，B，C
  + module 是一个模块，比如 `a.js`，webpack 里每个文件都是一个模块，所以 a.js 是一个 module
  + 以下边的代码为例，文件最开始的 0，1，2，3 等，就是 chunkId，这个 chunk 可能包括了很多的模块，最终生成了一个 chunk
  + chunk 有利于分割代码，pageA 页面只需要 chunk1，那么只需要加载 chunk1 的代码就可以，pageB，pageC 等页面，可以预加载，或者用到这个页面的时候再加载对应的 chunk

```
static/js/0.3a91ecddb7d88c37bcac.js     1.6 MB       0  [emitted]
static/js/1.58bf968713a72f675411.js    75.5 kB       1  [emitted]
static/js/2.58ef09f2165baed4e87f.js    56.2 kB       2  [emitted]
static/js/3.27af2d97f5db70fda453.js    65.7 kB       3  [emitted]
static/js/4.2577c7354803078e6c95.js      19 kB       4  [emitted]
static/js/5.9cfa2c0cf963135a760b.js    41.6 kB       5  [emitted]
static/js/6.a1b37170f66141e48b3e.js    56.6 kB       6  [emitted]
static/js/7.bb90be261d7079a65894.js    13.2 kB       7  [emitted]
```

# 3. source-map

+ source-map 的作用是为了映射源码，因为 webpack 会把源码进行编译打包，最后生成的代码看起来比较难懂，如果代码报错了，那么我们可以使用 source-map 进行映射到源码， 查看具体是哪里报错

```javascript
module.exports = {
  devtools: 'source-map'
}
```

+ devtools 的值可以有多个
  + source-map，源码映射，单独生成一份soucemap 文件，出错后，会标识出错误的列和行，特点是大而全
  + eval-soure-map，不会产生单独的文件，但是会显示行和列
  + cheap-module-source-map，不会产生列，是一个单独的映射文件，可以保留用来调试
  + cheap-module-eval-source-map 不会产生文件，会集成在打包后的文件里，不会产生行和列

# 4. watch 打包监控

+ 我们想要每次修改完代码后都会进行一次打包，所以我们可以进行监控配置

```javascript
module.exports = {
  watch: true,
  // 监控的属性
  watchOptions: {
    // 每秒监控 1000 次，是否需要打包
    poll: 1000,
    // 防抖，500ms 内输入的内容只打包一次，500ms 后没有再输入内容，开始打包
    aggregateTimeout: 500,
    // 不需要进行监控的目录
    ignored:  /node_modules/
  }
}
```

+ 完整的代码可以查看 04-webpack.config.js
