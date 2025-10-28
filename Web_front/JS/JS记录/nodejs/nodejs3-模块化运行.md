- [link_1](https://juejin.cn/post/6844903744518389768?searchId=2023111222040290AC1C7AA83D4ED3211E)

  ```js
  当使用browserify打包时，如果出现了ParseError: ‘import’ and ‘export’ may appear only with 'sourceType: module’的错误信息，这是因为browserify默认使用的是commonjs规范，而import和export是ES6的模块化规范，所以会出现这个错误。解决方法如下：
  
  安装browserify的esmify插件：npm install esmify --save-dev
  在打包命令中加入–plugin esmify参数，例如：browserify main.js --plugin esmify -o bundle.js 这样就可以解决这个错误了。
  
  ```

  

- [link_2](https://juejin.cn/post/7210216031536431162?searchId=2023111222040290AC1C7AA83D4ED3211E#heading-22)

  - 前端模块化的发展历史和不同标准的特点
  - 官方模块化的方式



- npm install 的本地变量

  - 本地变量只在本地moduble有，运行 npm run build 命令执行
  - npm run 获取相关的本地环境

  ```pascal
  // 比如运行 npm run build
  
  
  {
    "name": "coord_tool2",
    "version": "1.0.0",
    "description": "",
    "type": "module",
    "main": "script.js",
    "scripts": {
      "transpile": "babel coord.js -d lib",
      "bundle": "browserify lib/coord.js --plugin esmify -o bundle.js",
      "build": "npm run transpile && npm run bundle"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
      "axios": "^1.6.1",
      "browser-resolve": "^2.0.0",
      "dependencies": "^0.0.1",
      "package.json": "^0.0.0"
    },
    "devDependencies": {
      "@babel/preset-env": "^7.28.3",
      "babel-cli": "^6.26.0",
      "babel-preset-es2015": "^6.24.1",
      "browserify": "^17.0.1",
      "esmify": "^0.1.2"
    }
  }
  ```

  - 使用npx 执行
    - `npx browserify lib/coord.js --plugin esmify -o bundle.js`