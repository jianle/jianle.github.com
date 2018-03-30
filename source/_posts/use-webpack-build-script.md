---
title: 使用 Webpack 管理前端 JS
date: 2017-06-29 14:01:46
tags:
 - webpack
 - ejs
 - html
 - js
categories: Frontend
lede: "简单描述使用 webpack 管理前端 js 依赖打包及版本控制，使用 wepack 的一种开发模式"
thumbnail: /img/webpack.png
---

Webpack是啥？官网的解释是“webpack takes modules with dependencies and generates static assets representing those modules.”

说简单点就是一款前端的模块加载兼打包工具。  

首先看看项目的目录结构：  

```javascript  
- ../resources
  - src
    - imgs               //图片
    - scripts            //JS脚本，按照page、components进行组织
      - page
      - components
    - styles             //CSS脚本，按照page、components进行组织
      - page
      - components
    - templates          //页面ejs
  - static               //webpack编译打包输出目录的静态文件
  - templates            //webpack编译输出的模板静态HTML文件
  - server.js            //用于启动dev模式
  - webpack.config.js    //webpack配置文件
```

通过 `webpack.config.js` 来完成各种类型文件的 loader ，让我们一起看看该文件内容：

```javascript  
var path = require('path'),
    glob = require('glob'),
    webpack = require('webpack'),
    WebpackNotifierPlugin = require('webpack-notifier'),
    ExtractTextPlugin = require('extract-text-webpack-plugin'),
    HtmlWebpackPlugin = require('html-webpack-plugin');

var CommonsChunkPlugin = webpack.optimize.CommonsChunkPlugin;
var UglifyJsPlugin = webpack.optimize.UglifyJsPlugin;

const debug = process.env.NODE_ENV !== 'production'
//const contextPath = process.env.npm_package_config_contextPath;

var entries = getEntry('src/scripts/page/**/*.js', 'src/scripts/page/');
var chunks = Object.keys(entries);
console.log(entries);

var config = {

  entry: entries,
  output: {
    path: path.join(__dirname, 'static'),
    publicPath: 'http://0.0.0.0:9001/static/',
    filename: 'scripts/page/[name]-[hash].js',
    chunkFilename: 'scripts/[id].chunk.js?[hash]'
  },
  module: {
    loaders: [
      {
        test: /\.css$/,
        loader: ExtractTextPlugin.extract('style', 'css')
      }, {
        test: /\.less$/,
        loader: ExtractTextPlugin.extract('css!less')
      }, {
        test: /\.ejs$/,
        loader: 'ejs-loader'
      }, {
        test: /\.html$/,
        loader: 'html?-minimize' // 避免压缩html,https://github.com/webpack/html-loader/issues/50
      }, {
        test: /\.(woff|woff2|ttf|eot|svg)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
        loader: 'file-loader?name=fonts/[name].[ext]'
      }, {
        test: /\.(png|jpe?g|gif)$/,
        loader: 'url-loader?limit=8192&name=imgs/[name]-[hash].[ext]'
      }
    ]
  },
  plugins: [
    new webpack.ProvidePlugin({ // 加载jquery
      $: 'jquery',
      JQuery: 'jquery',
      _: 'underscore'
    }),
    new CommonsChunkPlugin({
      name: 'vendors', // 将公共模块提取，生成名为`vendors`的chunk
      chunks: chunks,
      minChunks: chunks.length // 提取所有entry共同依赖的模块
    }),
    new WebpackNotifierPlugin({alwaysNotify: true}),
    new ExtractTextPlugin('styles/[name]-[chunkhash].css'), // 单独使用link标签加载css并设置路径，相对于output配置中的publickPath
    debug ? function () {} : new UglifyJsPlugin({ // 压缩代码
      compress: {
        warnings: false
      },
      except: ['$super', '$', 'exports', 'require'] // 排除关键字
    })
  ]

}

var pages = Object.keys(getEntry('src/templates/**/*.ejs', 'src/templates/'));
pages.forEach(function(pathname) {
  var conf = {
    filename: '../templates/' + pathname + '.html', // 生成的html存放路径，相对于path
    template: 'src/templates/' + pathname + '.ejs', // html模板路径
    cache: false,//true | false if true (default) try to emit the file only if it was changed.
    inject: false // js插入的位置，true/'head'/'body'/false
  };
  if (pathname in config.entry) {
    //conf.favicon = path.resolve(__dirname, 'src/imgs/favicon.ico');
    //conf.inject = 'body';
    if (pathname == 'layout') {
      conf.chunks = ['vendors', pathname];
    } else {
      conf.chunks = [pathname];
    }
    conf.hash = false;
  }
  config.plugins.push(new HtmlWebpackPlugin(conf));
});

module.exports = config

function getEntry(globPath, pathDir) {

  var files = glob.sync(globPath);
  var entries = {}, entry, dirname, basename, pathname, extname;

  for (var i = 0; i < files.length; i++) {
    entry = files[i];
    dirname = path.dirname(entry);
    extname = path.extname(entry);
    basename = path.basename(entry, extname);
    pathname = path.normalize(path.join(dirname, basename));

    pathDir = path.normalize(pathDir);
    if (pathname.startsWith(pathDir)) {
      pathname = pathname.substring(pathDir.length);
    }
    entries[pathname] = ['./' + entry];
  }

  return entries;

}
```

当我们开发的时候，只需要在 `src/scripts/page` 下编写 js 文件，`src/templates` 下编辑对于文件名的`.ejs` 文件。  

##### 举例说明  


* JS 开发案例 `index.js`   


```javascript
require('../../styles/page/index.css')

var message = 'Hello World';
console.log(message);
```

* HTML 案例 `index.ejs`  


```html
<!DOCTYPE HTML>
<html layout:decorate="~{layout.html}">

  <head>
    <title>首页</title>
    <!-- 引入css文件 -->
    <% for (key in htmlWebpackPlugin.files.css) { %>
    <link href="<%= htmlWebpackPlugin.files.css[key] %>" rel="stylesheet">
    <% } %>
  </head>

  <body>
  <div layout:fragment="content">
    <h2>Hello World.</h2>
    <div class="img">
      <!-- 引入图片 -->
      <img src="<%= require('../imgs/test.png') %>" alt="">
    </div>
  </div><!-- content -->
  </body>

  <th:block layout:fragment="scripts">
    <!-- 引入js文件 -->
    <% for (key in htmlWebpackPlugin.files.chunks) { %>
    <script src="<%= htmlWebpackPlugin.files.chunks[key].entry %>" type="text/javascript"></script>
    <% } %>
    <script type="text/javascript" th:inline="javascript">
    </script>
  </th:block>
</html>
```


##### 参考文献   

* [What is the webpack](http://webpack.github.io/docs/what-is-webpack.html)
* [基于webpack的前端工程化方案（自动入口配置及后端模板）](https://github.com/vhtml/webpack-MultiplePage)
