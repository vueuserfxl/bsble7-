
## bable7升级踩坑记录

* 使用 babel-upgrade 升级到babel7：
```
npx babel-upgrade --write

npm i @bable/rumtime  core-js --save-dev
```
* 将工程中.babelr或者babel.config.js改成babelConfig.js，原因：当前版本下的babel-loader无法指定.babelrc的文件路径。
由于部分插件尚未支持core-js@3，@babel/preset-env和core-js需要指定版本
```
//  babelConfig.js   将 useBuiltIns 修改为 usage, 表示按需加载polyfill
    module.exports = {
      presets: [
        [
          '@babel/preset-env',
          {
            useBuiltIns: 'usage',
            targets: ['&gt; 1%', 'last 2 versions', 'IE &gt;= 10'],
            corejs: '2',
          }
        ],
        '@babel/preset-react'
      ],
      plugins: [
        ['@babel/plugin-proposal-decorators', { legacy: true }],
        ['@babel/plugin-proposal-class-properties', { loose: true }],
        '@babel/plugin-proposal-optional-chaining',
        '@babel/plugin-proposal-nullish-coalescing-operator',
    };
 ```
 ## 配置导入
 * 在webpack.config.js中要引入babelConfig.js。
 ```
 // webpack.config.base.js
 const babelConfig = require('../babelConfig.js');
 ...
 ...
loaders: [
    {
      loader: 'babel-loader',
      options: babelConfig
    }
  ]
  ```
*  运行npm run build，出现以下报错情况的解决方式：
#### 情景一：（你会发现报错的都是在export default 上一行用了修饰器(@**)的文件,所以你首先要将babelConfig.js中plugins里的插件安装好）
```
Parsing error: Using the export keyword between a decorator and a class is not allowed. Please use `export @dec class` instead.
```
#### 解决方式：修改对应jsx文件为
```
@connect('aaa')
 class AppView extends React.Component{
    render(){
       //....
    }
  }
export default AppView 
```
#### 情景二：（在依赖的文件中没有找到 export default 对应报错的公共组件）
```
解决方式：建议使用npm uninstall & npm install 卸载重装。
```
#### 情景三、运行起项目之后，如果页面样式出现问题了，引用的框架(比如antd)失效了。
解决方式：在babelConfig.js中plugins里加入以下内容
```
 ['import', {
      libraryName: 'antd',
      libraryDirectory: 'es',
      style: true
    }]
```
####decorator报错
#### 虽然已经默认安装了装饰器编译的babel插件，然而新版的eslint却没有放过我们。在你的eslint配置文件中加入如下配置即可
```
"parserOptions": {
    "ecmaFeatures": {
      "legacyDecorators": true
    }
  },
```
#### 如果 还有 webpack 警告 比如这种
```
WARNING in asset size limit: The following asset(s) exceed the recommended size limit (244 KiB).解决
```
#### 则只需在webpack.config.base.js 中加上
```
 performance: {
    hints: false,
  },
```
#### 最后：此时项目应该已经能成功编译，并且你会发现构建速度得到了很大的提升。
