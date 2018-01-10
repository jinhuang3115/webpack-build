# 使用webpack搭建简单的前端开发环境

### 简介
webpack 是一个模块打包器。它的主要目标是将 JavaScript 文件打包在一起，打包后的文件用于在浏览器中使用，但它也能够胜任转换(transform)、打包(bundle)或包裹(package)任何资源(resource or asset)。(from https://www.webpackjs.com/)

##搭建环境
前端代码需要被打包到两个环境中：开发环境和生产环境。

首先创建一个webpack.base.conf.js文件。这个文件用来存放生产环境和开发环境都会用到的配置
```apple js
const path = require('path');
const webpack = require('webpack');
const HtmlWebpackPlugin = require('html-webpack-plugin'); //插件：用于生成和修改HTML文件
const CopyWebpackPlugin = require('copy-webpack-plugin');//插件 用于复制第三方文件到指定目录

module.exports = {
    devtool: "source-map",//创建source-map
    entry: {
        app: path.join(__dirname, './js/pages/index.js'), //app入口文件
        vendor: [
            path.join(__dirname, './js/lib/ueditor/ueditor.config.js'), 
            path.join(__dirname, './js/lib/ueditor/ueditor.all.js')
        ]//第三方文件
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /(node_modules|bower_components)/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['es2015', 'react'], //通过babel来编译es2015语法和react语法
                        plugins: ["transform-remove-strict-mode"]
                    }
                }
            },
            {
                test: /\.(html)$/,
                use: {
                    loader: 'html-loader',
                    options: {
                        attrs: [':data-src']
                    }
                }
            },
            {
                test: /\.(scss|css)$/,
                use: [{
                    loader: "style-loader" // creates style nodes from JS strings
                }, {
                    loader: "css-loader" // translates CSS into CommonJS
                }, {
                    loader: "sass-loader" // compiles Sass to CSS
                }]
            },
            {
                test: /\.js$/,
                use: ["source-map-loader"],
                enforce: "pre"
            },
            {
                test: /\.(png|jpg|gif)$/,
                use: [
                    {
                        loader: 'file-loader',
                        options: {}
                    }
                ]
            }

        ]
    },
    plugins: [new HtmlWebpackPlugin(
        {
            template: path.join(__dirname, '/template/index.html') //移动html文件
        }
    ),
        new webpack.optimize.CommonsChunkPlugin({
            names: ['vendor', 'manifest'],
        }),//文件分包
        new CopyWebpackPlugin([
            {
                from: path.join(__dirname, './js/lib/ueditor/lang'), to: path.join(__dirname, 'build/lang')
            },
            {
                from: path.join(__dirname, './js/lib/ueditor/themes'), to: path.join(__dirname, 'build/themes')
            },
            {
                from: path.join(__dirname, './js/lib/ueditor/third-party'), to: path.join(__dirname, 'build/third-party')
            },
            {
                from: path.join(__dirname, './js/lib/ueditor/dialogs'), to: path.join(__dirname, 'build/dialogs')
            },
            {
                from: path.join(__dirname, './styles/lib/bootstrap'), to: path.join(__dirname, 'build/bootstrap')
            },
            {
                from: path.join(__dirname, './styles/lib/fonts'), to: path.join(__dirname, 'build/fonts')
            },
            {
                from: path.join(__dirname, './biz_publish_info.json'), to: path.join(__dirname, 'build/')
            }
        ])
    ]
};
```
之后创建开发环境代码webpack.dev.conf.js
```apple js
const path = require('path');
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.base.conf');

module.exports = merge(baseWebpackConfig, {
    output: {
        path: path.resolve(__dirname, 'build'),//输出到指定目录
        filename: "[name].js",
        chunkFilename: '[name].js'
    },
    devServer: {//建立本地运行环境
        contentBase: path.join(__dirname, "build"), 
        compress: true,
        port: 3000
    }
});
```
这样一个开发环境就搭建好了。

然后是生产环境webpack.producation.conf.js
```apple js
const path = require('path');
const merge = require('webpack-merge');
const baseWebpackConfig = require('./webpack.base.conf');
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');

module.exports = merge(baseWebpackConfig, {
    output: {
        path: path.resolve(__dirname, 'build'),
        filename: "[hash].[name].js", //静态文件名增加hash
        chunkFilename: '[name].js'
    },
    plugins: [
        new UglifyJSPlugin({
            test: /\.js($|\?)/i
        })//压缩代码
    ]
});
```

### 最后是执行命令
在package.json中创建npm scripts
```apple js
"scripts": {
    "dev": "webpack-dev-server --config webpack.dev.conf.js --open 'Google Chrome'",
    "build": "webpack --config webpack.producation.conf.js"
  }
```
执行npm run dev 会执行如下命令

"webpack-dev-server --config webpack.dev.conf.js --open 'Google Chrome'": 启动webpack-dev-server使用webpack.dev.conf.js配置同时打开Chrome浏览器。

执行npm run build 会执行如下命令
"webpack --config webpack.producation.conf.js": 执行webpack并使用webpack.producation.conf.js配置


这样一个简单的webpack开发环境及生产环境的搭建就完成了。当然在实际的项目可能还需要其他配置例如：检查代码规范、图片压缩、矢量图压缩等性能优化相关配置。