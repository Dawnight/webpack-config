# 1. 手写 webpack

+ 我们知道 webpack 有两个主要的库，一个是 webpack，另一个是 webpack-cli。

+ webpack 的功能是打包，webpack-cli 的功能是命令行工具

+ 所以我们也要有两个，一个是需要打包的 webpack.config.js 配置文件，一个是命令行打包工具

+ 我们需要再新建一个命令行工具项目，就叫做 carl-pack，这也是我们命令行的全局命令

# 2. 源代码

+ 我们的源代码比较简单，所以直接看代码就可以，主要就是解析 require 语法，在浏览器中也可以运行代码

+ src/base/b.js

```javascript
module.exports = 'b';
```

+ src/a.js

```javascript
let b = require('./base/b');
module.exports = 'a' + b;
```

+ src/index.js

```javascript
let str  = require('./a');
console.log(str);
```

# 3. webpack.config.js 的配置

+ 我们先对 webpack.config.js 做配置，这样我们在解析的时候，就知道需要使用到哪些字段内容

+ 我们只定义最简单的入口和出口，之后我们会定义 rules 和 plugins

```javascript
const path = require('path');

module.exports = {
  entry: path.resolve(__dirname, './src/index.js'),
  output: {
    filename: "bundle.js",
    path: path.resolve(__dirname, 'dist')
  }
};
```

# 4. carl-pack 项目介绍

+ 首先我们要知道 carl-pack 项目的目的，然后再梳理一下 carl-pack 项目的流程

+ 目的：实际上，carl-pack 项目可以理解为 webpack-cli + webpack，就是一个命令行工具，通过这个命令行工具，我们可以打包源代码，最终生成打包后的代码

+ 流程：
  + 创建全局的 carl-pack 命令，就像是全局 webpack 命令一样，我们也要创建一个 carl-pack 的全局命令
  + 通过 carl-pack 命令解析 webpack.config.js 文件。当然，我们可以不使用 webpack.config.js 这个文件名，可以使用 carl-pack.config.js 文件名，这个是可以自己定义的，都可以的。由于我们还是采用 webpack 的配置方式，所以我们还是叫 webpack.config.js 这个名字
  + 根据 webpack.config.js 的配置，我们通过解析源代码，并且加载我们自己写的 loader 和 plugin
  + 最终使用一个模板引擎，把渲染好的字符串输出到目标文件夹，生成打包后的代码

# 5. 配置全局 `carl-pack` 命令

+ 首先我们先创建一个 carl-pack 的文件夹，进入文件夹，使用 `npm init -y` 进行初始化

+ 然后我们新建一个 bin 文件夹，在 bin 下边新建一个 www 文件，在 www 文件里写下下边的内容

+ 我们说一下 bin 目录及 www 文件里的代码

+ bin 目录，实际上就是执行文件的目录，这里的 www 文件就是可执行的文件
  + 关于文件目录的设定，具体可以查看文件系统层次结构标准（Filesystem Hierarchy Standard，FHS），这个规范制定了一系列的文件系统层次结构的标准，由Linux基金会维护
  + 如果我们注意观察，我们会发现我们之前安装的 git，vscode 等软件的安装目录下， 都有一个 bin 目录，bin目录下的都是 exe 和 cmd 文件，实际上都是可执行文件

+ www 文件，简称 3w 文件，这个就是可执行文件
  + 我们使用 express-generator 生成的项目里就有一个 bin 目录，里边有一个 www 文件，所以我们就直接采用 www 命名
  + 代码里的 `#! /usr/bin/env node` 只有在 unix/linux 下会被识别，在 windows 下会被忽略，但是如果写错了，在 windows 下也是会报错的
  + `#!/usr/bin/env node` 这种用法是为了防止操作系统用户没有将 node 装在默认的 `/usr/bin` 路径里。当系统看到这一行的时候，首先会到 env 设置里查找 node 的安装路径，再调用对应路径下的解释器程序完成操作。
  + 简单来说，就是为了调用解释器，在 python 里也有这样的用法
  + 然后直接引入了 src/index.js 文件
  + 我们没有直接在 www 文件里写代码，是因为我们要对代码统一管理，把 www 文件作为一个执行的入口文件即可

```javascript
#! /usr/bin/env node

require('../src/index');
```

+ 新建 /src/index.js 文件
  + 在这个文件里，我们就写一句话，`console.log('hello, carl-pack');`，用来测试全局命令是否安装成功

+ 定义 carl-pack 命令
  + 定义命令需要在 package.json 里进行设置
  + 在 bin 配置项里添加 carl-pack 的 key，value 就是 www 文件的相对路径

```json
{
  "bin": {
    "carl-pack": "./bin/www"
  }
}
```

+ 我们在项目的根目录下的命令行执行一条命令 `npm link`

+ 这个时候，就在全局新建了一条命令，叫做 `carl-pack`，直接在命令行输入 `carl-pack` 或者 `npx carl-pack`，就会输出我们在 src/index.js 里 console 的内容 `hello, carl-pack`

+ 如果正确输出了 console.log 的内容，那么说明全局命令已经配置成功


# 6. 解析 webpack.config.js 文件

## 6.1 解析准备

+ 我们已经创建了全局命令，那么我们就需要通过全局命令去解析 webpack.config.js

+ 那么我们需要知道解析的原理与流程

+ 原理
  + 由于 webpack.config.js 导出的是一个对象，所以我们直接使用导出的对象
  + 这个对象里有配置信息，entry ，output 和 module 的 rules 信息，其他的我们暂时都不做解析
  + 解析对象完成后，我们开始打包，通过 fs 模块，生成打包后的文件代码

+ 流程
  + 第 1 步，我们在使用 carl-pack 的时候，一定是在项目的根目录，所以，我们要在根目录通过 path 模块找到 webpack.config.js 文件，获取到 entry ，output 和 module 的 rules 信息
  + 第 2 步，从入口文件里找到文件的依赖，执行并创建文件的依赖关系，这个意思就是说，比如 index.js 引用了 a.js 和 b.js，那么 index.js 与 a.js 和 b.js 就有了依赖关系，但是 a.js 和 b.js 没有依赖关系
  + 第 3 步，使用 AST 对源代码进行编译，编译成浏览器能够识别的代码
  + 第 4 步，使用模板生成代码
  + 第 5 步，直接使用代码
