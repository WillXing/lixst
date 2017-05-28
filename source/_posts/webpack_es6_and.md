---
title: 启动一个 Webpack + ES6 + React + CSSModule Web应用拢共分几步
date: 2017-05-28
thumbnail: /img/mine/flight_cloud_gaoxin.jpeg
tags:
  - frontend
  - javascript
  - react
  - webpack
  - css
---

16年写这篇文章的时候，在社区里，React实在是有太多太多的starter，各种各样，这也是React被诟病的一个点，这篇文章就是讲了一下如何自己手动搭建一个React的Starter。

## 开始 
**首先需要有[NodeJS](nodejs.org)环境**

### 全局安装Webpack 

```bash
npm i webpack -g 
```

### 创建并初始化项目 

```bash
mkdir project
cd project
npm init 
#init之后熟悉的话可以自己填填内容，不熟悉的话一路回车 
```

### 安装项目依赖

```bash
# 在Project根目录下
npm i webpack babel-core babel-loader babel-preset-es2015 babel-preset-react extract-text-webpack-plugin html-webpack-plugin --save-dev
npm i style-loader css-loader --save-dev
npm i react react-dom --save
```

> 依赖简介: 
>  webpack **打包工具**
>  babel-core, babel-loader, babel-preset-es2015, babel-preset-react **用来Compile ES6, React**
>  extract-text-webpack-plugin, html-webpack-plugin **Webpack打包所需插件，这里用于处理html和css**
>  style-loader css-loader **CSS laoder**
>  react, react-dom **React 依赖**
	
### Babel配置`.babelrc`

```bash
# 在Project根目录下
touch .babelrc
```

在`.babelrc`中加入如下内容

```json	
{
  "presets": ["es2015", "react"]
}
```
	
### Webpack配置`webpack.config.babel.js`	

```bash
mkdir build
touch build/webpack.config.babel.js
```

在`webpack.config.babel.js`中加入如下内容

```js
import webpack from 'webpack'
import HtmlWebpackPlugin from 'html-webpack-plugin'
import ExtractTextWebpackPlugin from 'extract-text-webpack-plugin'

export default {
  context: __dirname + '/../src',
  entry: {
  //JS的入口
    app: "./index.jsx"
  },
  output: { 
  //JS输出位置
    filename: '[name].js',
    path: __dirname + '/../dist'
  },
  module: {
    loaders: [
    //Loader 的配置
      {
      //Babel loader 用于编译ES2015的语法
          test: /\.jsx$/,
          loader: 'babel',
          exclude: /node_modules/
      },
      {
      //CSS Module需要如下配置
        test: /\.css/,
        loader: ExtractTextWebpackPlugin.extract(
          'style',
          'css?modules&importLoaders=1&localIdentName=[path]___[name]__[local]___[hash:base64:5]'
        )
      }
    ]
  },
  plugins: [
    new ExtractTextWebpackPlugin('[name].css', { allChunks: true }), //CSS相关
    new HtmlWebpackPlugin({
    //HTML的相关配置
      filename: 'index.html',
      template: 'index.html',
      inject: true,
      hash: true,
      chunks: ['app']
    })
  ]
}
```

### 创建应用代码

我们的实例Example目录结构大概如下

    src
    ├── home         # 为了演示简单，我们简化掉了这块，只是给大家看一下
    │   ├── Home.css # 如果使用CSSModule，这样的目录结构比较爽
    │   └── Home.jsx #
    ├── index.css
    ├── index.html
    └── index.jsx

* `index.html` 内容

```html	
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Demo</title>
</head>
<body>
<div id="app"></div>
</body>
</html>
```

* `index.jsx` 内容

```jsx
import React from 'react'
import styles from './index.css'
import { render } from 'react-dom'

class SearchBox extends React.Component {
  render() {
    return <form className={styles.search_container}>
      <input className={styles.search_box} type="text"/>
      <input className={styles.search_button} type="submit"/>
    </form>
  }
}

class Home extends React.Component {
  render() {
    return <SearchBox/>
  }
}

render(<Home/>, document.getElementById('app'))
```

* `index.css` 内容

```css
.search_container {
  text-align: center;
}		
.search_box {
  background-color: #d8d8d8;
  border: 0;
  border-radius: 10px;
  height: 50px;
  padding: 0 20px;
}
.search_button {
  height: 35px;
  width: 100px;
  border: 0;
  margin-left: 20px;
  border-radius: 10px;
  background-color: #ccffcc;
}
```

### 使用webpack进行编译

```bash
webpack --config build/webpack.config.babel.js
```
如果不出意外，那你应该可以看见在你项目下的`dist`目录，有编译之后的代码了。
	
## 	最后
我们的目录结构最后大概长这样

    project
    ├── build
    │   └── webpack.config.babel.js
    ├── dist
    │   ├── app.css
    │   ├── app.js
    │   └── index.html
    ├── node_modules
    ├── package.json
    └── src
        ├── index.css
        ├── index.html
        └── index.jsx

Ok了，一个 Webpack + ES6 + React + CSSModule 的项目就启动好了


## 后记
React当时没有个像样的cli tool（至少我没找到），这一点，[Vue](https://github.com/vuejs/vue-cli) 和 [Angular2](https://cli.angular.io/) 就做得非常好了。
但是现在，React也有了[Create React App](https://github.com/facebookincubator/create-react-app)。减轻了很多创建React项目所需的工作。