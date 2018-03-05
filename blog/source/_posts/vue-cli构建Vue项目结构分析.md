---
title: vue-cli构建Vue项目结构分析
comments: true
date: 2018-02-10 14:15:15
updated: 2018-02-10 14:15:15
toc: true
description: 使用vue-cli脚手架可以快速构建vue项目，本文主要分析vue-cli构建的项目结构。
tags: 
 - vue

---
## 构建项目
使用vue-cli脚手架搭建vue项目的具体步骤如下：
```
npm install -g vue-cli
cd E:(跳转到项目目录)
vue init webpack vueproject (vueproject 为项目目录名称，可行更改)
cd vueproject
npm install
npm run dev
```

##  项目结构分析
![项目结构](img/project.png)

##  package.json

抽取package.json文件重要部分分析

>scripts字段

```
"scripts": {
    "dev": "node build/dev-server.js",
    "build": "node build/build.js"
}
```
项目开发周期主要执行的两个任务分别是开发环境``npm run dev``和打包任务``npm run build``,script字段是用来指定npm相关命令的缩写的,即相当于在node环境下执行build/dev-server.js和node build/build.js文件。

-------------------------------------------------------------

>dependencies和devDependencies字段

```
"dependencies": {
    "vue": "^2.3.3",
    "vue-router": "^2.6.0"
},
"devDependencies": {
    "autoprefixer": "^7.1.2",
    "babel-core": "^6.22.1"
}
```

dependencies字段指定了项目运行时所依赖的模块，devDependencies字段指定了项目开发时所依赖的模块。项目开发应使用命令管理package.json文件：
```
npm i --save vue              \\自动写入package.json文件的dependencies字段；
npm i --save-dev babel-core \\自动写入package.json文件的devDependencies字段；
```
注：i为install的缩写

------------------------------------------------------------

>engine和browserslist字段

```
  "engines": {
    "node": ">= 4.0.0",
    "npm": ">= 3.0.0"
  },
  "browserslist": [
    "> 1%",
    "last 2 versions",
    "not ie <= 8"
  ]
```
engines字段表示项目运行所依赖的node以及npm版本号，browserslist表示项目的浏览器支持情况，具体详情可以查看[https://www.npmjs.com/package/browserslist](https://www.npmjs.com/package/browserslist)。

##  build文件夹

###  dev-server.js
```
// 检查 Node 和 npm 版本
require('./check-versions')()
//使用了 config/index.js
var config = require('../config')
// 如果 Node 的环境无法判断当前是 dev / product 环境
if (!process.env.NODE_ENV) {
// 使用 config.dev.env.NODE_ENV 作为当前的环境
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}
// 一个可以强制打开浏览器并跳转到指定 url 的插件
//(可以调用默认软件打开网址、图片、文件等内容的插件,
//这里用它来调用默认浏览器打开dev-server监听的端口，例如：localhost:8080)
var opn = require('opn')
// 使用 NodeJS 自带的文件路径工具
var path = require('path')
// 使用 express
var express = require('express')
// 使用 webpack
var webpack = require('webpack')
// http-proxy可以实现转发所有请求代理到后端真实API地址，以实现前后端开发完全分离
// 在config/index.js中可以对proxyTable想进行配置
var proxyMiddleware = require('http-proxy-middleware')
// 根据 Node 环境来引入相应的 webpack 配置
var webpackConfig = process.env.NODE_ENV === 'testing'
  ? require('./webpack.prod.conf')
  : require('./webpack.dev.conf')

// default port where dev server listens for incoming traffic
// 如果没有指定运行端口，使用 config.dev.port 作为运行端口
var port = process.env.PORT || config.dev.port
// automatically open browser, if not set will be false
// 用于判断是否要自动打开浏览器的布尔变量，当配置文件中没有设置自动打开浏览器的时候其值为 false
var autoOpenBrowser = !!config.dev.autoOpenBrowser

// Define HTTP proxies to your custom API backend
// https://github.com/chimurai/http-proxy-middleware
// 定义 HTTP 代理表，代理到 API 服务器
// 使用 config.dev.proxyTable 的配置作为 proxyTable 的代理配置
var proxyTable = config.dev.proxyTable
// 使用 express 启动一个服务
var app = express()
// 启动 webpack 进行编译
var compiler = webpack(webpackConfig)
// 启动 webpack-dev-middleware，将 编译后的文件暂存到内存中
//(webpack-dev-middleware使用compiler对象来对相应的文件进行编译和绑定
// 编译绑定后将得到的产物存放在内存中而没有写进磁盘
// 将这个中间件交给express使用之后即可访问这些编译后的产品文件)
var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})
// 启动 webpack-hot-middleware，也就是我们常说的 Hot-reload用于实现热重载功能的中间件
var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: () => {},
  heartbeat: 2000
})
// force page reload when html-webpack-plugin template changes
// 当html-webpack-plugin提交之后通过热重载中间件发布重载动作使得页面重载
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})

// proxy api requests
// 将 proxyTable 中的请求配置挂在到启动的 express 服务上
//Object.keys()返回对象的键名数组
Object.keys(proxyTable).forEach(function (context) {
  var options = proxyTable[context]
  if (typeof options === 'string') {
    options = { target: options }
  }
  app.use(proxyMiddleware(options.filter || context, options))
})

// handle fallback for HTML5 history API
// 使用 connect-history-api-fallback 匹配资源，如果不匹配就可以重定向到指定地址，常用于SPA
app.use(require('connect-history-api-fallback')())

// serve webpack bundle output
// 将暂存到内存中的 webpack 编译后的文件挂在到 express 服务上
app.use(devMiddleware)

// enable hot-reload and state-preserving
// compilation error display
// 将热重载中间件(Hot-reload)挂在到express服务器上
app.use(hotMiddleware)

// serve pure static assets
// 拼接 static 文件夹的静态资源路径
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)
// 为静态资源提供响应服务
app.use(staticPath, express.static('./static'))
// 应用的地址信息，例如：http://localhost:8080
var uri = 'http://localhost:' + port

var _resolve
var readyPromise = new Promise(resolve => {
  _resolve = resolve
})

console.log('> Starting dev server...')

// webpack开发中间件合法（valid）之后输出提示语到控制台，表明服务器已启动
devMiddleware.waitUntilValid(() => {
  console.log('> Listening at ' + uri + '\n')
  // when env is testing, don't need open it
  // 如果不是测试环境，自动打开浏览器并跳到我们的开发地址
  if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    opn(uri)
  }
  _resolve()
})
//监听服务器端口
var server = app.listen(port)

module.exports = {
  ready: readyPromise,
  close: () => {
    server.close()
  }
}
```

该文件主要完成以下事情：

1.检查node和npm的版本。

2.引入相关插件和配置。

3.创建express服务器和webpack编译器。

4.配置开发中间件（webpack-dev-middleware）和热重载中间件（webpack-hot-middleware）。

5.挂载代理服务和中间件。

6.配置静态资源。

7.启动服务器监听特定端口（8080）。

8.自动打开浏览器并打开特定网址（localhost:8080）。

注：express服务器提供静态文件服务，不过它可以使用了http-proxy-middleware,一个http请求代理的中间件。前端开发过程中需要使用到后台的API的话，可以通过配置proxyTable来将相应的后台请求代理到专用的API服务器。

------------------------------------------------------------

###   build.js

```
// 检查 Node 和 npm 版本
require('./check-versions')()
//生产环境
process.env.NODE_ENV = 'production'
// 一个很好看的 loading 插件
var ora = require('ora')
var rm = require('rimraf')
// 使用 NodeJS 自带的文件路径插件
var path = require('path')
// 用于在控制台输出带颜色字体的插件
var chalk = require('chalk')
//加载webpack
var webpack = require('webpack')
//加载config中index.js
var config = require('../config')
//加载webpack.prod.conf
var webpackConfig = require('./webpack.prod.conf')

var spinner = ora('building for production...')
spinner.start()// 开启loading动画
// 拼接编译输出文件路径
var assetsPath = path.join(config.build.assetsRoot, config.build.assetsSubDirectory);
// 删除这个文件夹 （递归删除）
rm(assetsPath, err => {
  if (err) throw err
    //  开始 webpack 的编译
  webpack(webpackConfig, function (err, stats) {
    // 编译成功的回调函数
    spinner.stop()
    if (err) throw err
    process.stdout.write(stats.toString({
      colors: true,
      modules: false,
      children: false,
      chunks: false,
      chunkModules: false
    }) + '\n\n')

    console.log(chalk.cyan('  Build complete.\n'))
    console.log(chalk.yellow(
      '  Tip: built files are meant to be served over an HTTP server.\n' +
      '  Opening index.html over file:// won\'t work.\n'
    ))
  })
})
```

build.js主要作用为：

1.显示打包loading动画。

2.删除并创建目标文件夹。

3.webpack编译源文件。

4.输出打包后的文件。

注：webpack编译之后会输出到配置里面指定的目标文件夹；删除目标文件夹之后再创建是为了去除旧的内容，以免产生不可预测的影响。

------------------------------------------------------------

###   check-versions.js

```
// 用于在控制台输出带颜色字体的插件
var chalk = require('chalk')
// 语义化版本检查插件
var semver = require('semver')
// 引入package.json
var packageConfig = require('../package.json')
var shell = require('shelljs')
// 开辟子进程执行指令cmd并返回结果
function exec (cmd) {
  return require('child_process').execSync(cmd).toString().trim()
}
// node和npm版本需求
var versionRequirements = [
  {
    name: 'node',
    currentVersion: semver.clean(process.version),
    versionRequirement: packageConfig.engines.node
  },
]

if (shell.which('npm')) {
  versionRequirements.push({
    name: 'npm',
    currentVersion: exec('npm --version'),
    versionRequirement: packageConfig.engines.npm
  })
}

module.exports = function () {
  var warnings = []
  // 依次判断版本是否符合要求
  for (var i = 0; i < versionRequirements.length; i++) {
    var mod = versionRequirements[i]
    if (!semver.satisfies(mod.currentVersion, mod.versionRequirement)) {
      warnings.push(mod.name + ': ' +
        chalk.red(mod.currentVersion) + ' should be ' +
        chalk.green(mod.versionRequirement)
      )
    }
  }
  // 如果有警告则将其输出到控制台
  if (warnings.length) {
    console.log(chalk.yellow('To use this template, you must update following to modules:'))
    for (var i = 0; i < warnings.length; i++) {
      var warning = warnings[i]
      console.log('  ' + warning)
    }
    process.exit(1)
  }
}
```

该文件主要是用来检测当前环境中的node和npm版本和我们需要的是否一致的。

------------------------------------------------------------

###   webpack.base.conf.js

```
// 使用 NodeJS 自带的文件路径插件
var path = require('path')
// 引入一些小工具
var utils = require('./utils')
// 引入 config/index.js
var config = require('../config')
var vueLoaderConfig = require('./vue-loader.conf')

// 拼接我们的工作区路径为一个绝对路径
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}

module.exports = {
  // 配置webpack编译入口
  entry: {
    app: './src/main.js'
  },
  // 配置webpack输出路径和命名规则
  output: {
    // webpack输出的目标文件夹路径（例如：/dist）
    path: config.build.assetsRoot,
    // webpack输出bundle文件命名格式
    filename: '[name].js',
    // 正式发布环境下webpack编译输出的发布路径
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath
      : config.dev.assetsPublicPath
  },
  resolve: {
    // 自动补全的扩展名
    extensions: ['.js', '.vue', '.json'],
    // 路径代理,创建路径别名，有了别名之后引用模块更方便
    // 例如:import Vue from 'vue/dist/vue.common.js'可以写成 import Vue from 'vue'
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src')
    }
  },
  // 配置不同类型模块的处理规则
  module: {
    rules: [
      {
        // 对src和test文件夹下的.js和.vue文件使用eslint-loader
        test: /\.(js|vue)$/,
        loader: 'eslint-loader',
        enforce: 'pre',
        include: [resolve('src'), resolve('test')],
        options: {
          formatter: require('eslint-friendly-formatter')
        }
      },
      {
        // 对所有.vue文件使用vue-loader
        test: /\.vue$/,
        loader: 'vue-loader',
        options: vueLoaderConfig
      },
      {
        // 对src和test文件夹下的.js文件使用babel-loader
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      },
      {
        // 对图片资源文件使用url-loader，query.name指明了输出的命名规则
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        // 对字体资源文件使用url-loader，query.name指明了输出的命名规则
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  }
}

```

webpack.base.conf.js主要完成了下面这些事情：

1.配置webpack编译入口

2.配置webpack输出路径和命名规则

3.配置模块resolve规则

4.配置不同类型模块的处理规则

注：这个配置里面只配置了.js、.vue、图片、字体等文件的处理规则，如果需要处理其他文件可以在module.rules里面配置。

------------------------------------------------------------

###   webpack.dev.conf.js

```
// 使用一些小工具
var utils = require('./utils')
// 使用 webpack
var webpack = require('webpack')
// 同样的使用了 config/index.js
var config = require('../config')
// 使用 webpack 配置合并插件,可以合并数组和对象
var merge = require('webpack-merge')
// 加载 webpack.base.conf，webpack基础配置
var baseWebpackConfig = require('./webpack.base.conf')
// 使用 html-webpack-plugin 插件，这个插件可以帮我们自动生成 html 并且注入到 .html 文件中
//(自动注入依赖文件（link/script）的webpack插件)
var HtmlWebpackPlugin = require('html-webpack-plugin')
// 用于更友好地输出webpack的警告、错误等信息
var FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')

// add hot-reload related code to entry chunks
// 将 Hol-reload 相对路径添加到 webpack.base.conf 的 对应 entry 前，指定入口js文件
Object.keys(baseWebpackConfig.entry).forEach(function (name) {
  baseWebpackConfig.entry[name] = ['./build/dev-client'].concat(baseWebpackConfig.entry[name])
})
// 将我们 webpack.dev.conf.js 的配置和 webpack.base.conf.js 的配置合并
module.exports = merge(baseWebpackConfig, {
  // 配置样式文件的处理规则，使用styleLoaders
  module: {
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap })
  },
  // cheap-module-eval-source-map is faster for development
   // 配置Source Maps。使用 #cheap-module-eval-source-map 模式作为开发工具，在开发中使用cheap-module-eval-source-map更快
  devtool: '#cheap-module-eval-source-map',

  // 配置webpack插件
  plugins: [
    // definePlugin 接收字符串插入到代码当中, 所以你需要的话可以写上 JS 的字符串
    new webpack.DefinePlugin({
      'process.env': config.dev.env
    }),
    // https://github.com/glenjamin/webpack-hot-middleware#installation--usage
    // HotModule 插件在页面进行变更的时候只会重回对应的页面模块，不会重绘整个 html 文件
    new webpack.HotModuleReplacementPlugin(),
     // 使用了 NoErrorsPlugin 后页面中的报错不会阻塞，但是会在编译结束后报错
    new webpack.NoEmitOnErrorsPlugin(),
    // https://github.com/ampedandwired/html-webpack-plugin
    // 将 index.html 作为入口，注入 html 代码后生成 index.html文件
    new HtmlWebpackPlugin({
      filename: 'index.html',
      template: 'index.html',
      inject: true
    }),
    new FriendlyErrorsPlugin()
  ]
})

```

该文件主要完成以下事情：

1.将hot-reload相关的代码添加到entry chunks。

2.合并基础的webpack配置。

3.使用styleLoaders。

4.配置Source Maps。

5.配置webpack插件。

------------------------------------------------------------

###   webpack.prod.conf.js

```
// 使用 NodeJS 自带的文件路径插件
var path = require('path')
// 使用一些小工具
var utils = require('./utils')
// 加载 webpack
var webpack = require('webpack')
// 加载 confi.index.js
var config = require('../config')
// 加载 webpack 配置合并工具
var merge = require('webpack-merge')
// 加载 webpack.base.conf.js
var baseWebpackConfig = require('./webpack.base.conf')
//使用copy-webpack-plugin插件
var CopyWebpackPlugin = require('copy-webpack-plugin')
// 使用 html-webpack-plugin 插件，这个插件可以帮我们自动生成 html 并且注入到 .html 文件中
var HtmlWebpackPlugin = require('html-webpack-plugin')

// 用于从webpack生成的bundle中提取文本到特定文件中的插件
// 可以抽取出css，js文件将其与webpack输出的bundle分离
var ExtractTextPlugin = require('extract-text-webpack-plugin')

//使用js,css压缩插件
var OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')

//判断当前环境是否为测试环境，如是加载测试环境配置文件，否则使用config.build.env
var env = process.env.NODE_ENV === 'testing'
  ? require('../config/test.env')
  : config.build.env
// 合并 webpack.base.conf.js
var webpackConfig = merge(baseWebpackConfig, {
  module: {
    // 使用的 loader
    rules: utils.styleLoaders({
      sourceMap: config.build.productionSourceMap,
      extract: true
    })
  },
  // 是否使用 #source-map 开发工具
  devtool: config.build.productionSourceMap ? '#source-map' : false,
  output: {
    // 编译输出目录
    path: config.build.assetsRoot,
    // 编译输出文件名
    // 我们可以在 hash 后加 :6 决定使用几位 hash 值
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    // 没有指定输出名的文件输出的文件名
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  // 使用的插件
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
     // definePlugin 接收字符串插入到代码当中, 所以你需要的话可以写上 JS 的字符串
    new webpack.DefinePlugin({
      'process.env': env
    }),
    // 压缩 js (同样可以压缩 css)
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      },
      sourceMap: true
    }),
    // extract css into its own file
    // 将 css 文件分离出来
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css')
    }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    new OptimizeCSSPlugin({
      cssProcessorOptions: {
        safe: true
      }
    }),
    // generate dist index.html with correct asset hash for caching.
    // you can customize output by editing /index.html
    // see https://github.com/ampedandwired/html-webpack-plugin
    // 输入输出的 .html 文件
    new HtmlWebpackPlugin({
      filename: process.env.NODE_ENV === 'testing'
        ? 'index.html'
        : config.build.index,
      template: 'index.html',
      // 是否注入 html
      inject: true,
      // 压缩的方式
      minify: {
        removeComments: true,
        collapseWhitespace: true,
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      chunksSortMode: 'dependency'
    }),
    // split vendor js into its own file
    // 没有指定输出文件名的文件输出的静态文件名
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks: function (module, count) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    // 没有指定输出文件名的文件输出的静态文件名
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      chunks: ['vendor']
    }),
    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})
// 开启 gzip 的情况下使用下方的配置,引入compression插件进行压缩
if (config.build.productionGzip) {
  var CompressionWebpackPlugin = require('compression-webpack-plugin')
 // 加载 compression-webpack-plugin 插件
 var reProductionGzipExtensions = '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$';
  // 使用 compression-webpack-plugin 插件进行压缩
  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      asset: '[path].gz[query]',
      algorithm: 'gzip',
      test: new RegExp(reProductionGzipExtensions),
      threshold: 10240,
      minRatio: 0.8
    })
  )
}

if (config.build.bundleAnalyzerReport) {
  var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig

```

该文件主要用处：

1.合并基础的webpack.base.conf.js文件配置。

2.使用styleLoaders。

3.配置webpack的输出路径。

4.配置webpack插件。

5.gzip模式下的webpack插件配置。

6.webpack-bundle分析。

注：webpack插件里面使用了压缩代码以及抽离css文件等插件。

------------------------------------------------------------

##  config文件夹

### index.js
```
// 使用 NodeJS 自带的文件路径插件
var path = require('path')

module.exports = {
  // production 环境
  build: {
    // 使用 config/prod.env.js 中定义的编译环境
    env: require('./prod.env'),
    // 编译输入的 index.html 文件
    index: path.resolve(__dirname, '../dist/index.html'),
    // 编译输出的静态资源根路径
    assetsRoot: path.resolve(__dirname, '../dist'),
    // 编译输出的二级目录
    assetsSubDirectory: 'static',
    // 编译发布上线路径的根目录，可配置为资源服务器域名或 CDN 域名
    assetsPublicPath: './',
    // 是否开启 cssSourceMap
    productionSourceMap: true,
    // Gzip off by default as many popular static hosts such as
    // Surge or Netlify already gzip all static assets for you.
    // Before setting to `true`, make sure to:
    // npm install --save-dev compression-webpack-plugin
    // 是否开启 gzip,默认不开启
    productionGzip: false,
    // gzip模式下需要压缩的文件的扩展名
    productionGzipExtensions: ['js', 'css'],
    // Run the build command with an extra argument to
    // View the bundle analyzer report after build finishes:
    // `npm run build --report`
    // Set to `true` or `false` to always turn it on or off
    bundleAnalyzerReport: process.env.npm_config_report
  },
  // dev 环境
  dev: {
    // 使用 config/dev.env.js 中定义的编译环境
    env: require('./dev.env'),
    // 运行测试页面的端口
    port: 8087,
    // 启动dev-server之后自动打开浏览器
    autoOpenBrowser: true,
    // 编译输出的二级目录
    assetsSubDirectory: 'static',
    // 编译发布上线路径的根目录，可配置为资源服务器域名或 CDN 域名
    assetsPublicPath: '/',
    // 需要 proxyTable 代理的接口（可跨域）
    proxyTable: {},
    //静态网址
    /*proxyTable: {

      // 下面的示例将代理请求/api/posts/1到http://jsonplaceholder.typicode.com/posts/1。
      '/api': {
        target: 'http://jsonplaceholder.typicode.com',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    }*/
    //changeOrigin:true,那么本地会虚拟一个服务端接收你的请求并代你发送该请求，这样就不会有跨域问题了，当然这只适用于开发环境。
    // 除了静态网址之外，您还可以使用glob模式来匹配URL，例如/api/**。有关详细信息，请参阅上下文匹配。
    // 此外，您可以提供一个filter可以是自定义函数的选项，以确定请求是否应被代理：
    /*proxyTable: {
      '*': {
            target: 'http://jsonplaceholder.typicode.com',
            filter: function (pathname, req) {
              return pathname.match('^/api') && req.method === 'GET'
            }
        }
    }*/

    // CSS Sourcemaps off by default because relative paths are "buggy"
    // with this option, according to the CSS-Loader README
    // (https://github.com/webpack/css-loader#sourcemaps)
    // In our experience, they generally work as expected,
    // just be aware of this issue when enabling this option.
    // 是否开启 cssSourceMap
    cssSourceMap: false
  }
}

```

该文件配置了开发和生产两种环境下的配置。

------------------------------------------------------------

## 总结

webpack的使用博大精深，仅仅了解到这里也只是入门。代码可以直接到我的github直接拉取，仓库地址[https://github.com/HeyingYe/vue-structural-analysis](https://github.com/HeyingYe/vue-structural-analysis)。