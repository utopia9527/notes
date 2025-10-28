以下是一个简单的示例，演示如何使用 Webpack 来构建一个基本的 Web 应用程序。在这个示例中，我们将使用 Webpack 来打包 JavaScript 文件，并将其输出到一个单独的文件中。

1. **创建项目结构**：
   首先，创建一个新的目录并在其中初始化一个新的 Node.js 项目。在命令行中执行以下命令：

   ```
   mkdir webpack-demo
   cd webpack-demo
   npm init -y
   ```

2. **安装 Webpack**：
   在命令行中执行以下命令来安装 Webpack 和 Webpack CLI：

   ```
   npm install webpack webpack-cli --save-dev
   ```

3. **创建 JavaScript 文件**：
   在项目目录下创建一个名为 `src` 的子目录，并在其中创建一个名为 `index.js` 的 JavaScript 文件。在 `index.js` 文件中编写一些简单的 JavaScript 代码，例如：

   ```
   javascript// src/index.js
   const message = "Hello, Webpack!";
   alert(message)
   ```

4. **创建 Webpack 配置文件**：
   在项目根目录下创建一个名为 `webpack.config.js` 的文件，用于配置 Webpack 的打包规则和行为。一个简单的配置文件可能如下所示：

   ```
   const path = require('path');
   
   module.exports = {
     entry: './src/index.js',
     output: {
       path: path.resolve(__dirname, 'dist'),
       filename: 'bundle.js'
     }
   };
   ```

5. **运行 Webpack**：
   在命令行中执行以下命令来运行 Webpack：

   ```
   npx webpack
   ```

   这将会根据配置文件对您的 JavaScript 模块进行打包，并生成一个名为 `bundle.js` 的静态资源文件。

6. **创建 HTML 文件**：
   在项目根目录下创建一个名为 `index.html` 的 HTML 文件，引入刚刚打包生成的 `bundle.js` 文件：

   ```
   html<!-- index.html -->
   <!DOCTYPE html>
   <html lang="en">
   <head>
     <meta charset="UTF-8">
     <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Webpack Demo</title>
   </head>
   <body>
     <script src="dist/bundle.js"></script>
   </body>
   </html>
   ```

7. **在浏览器中打开 HTML 文件**：
   最后，在浏览器中打开 `index.html` 文件，您将会看到控制台输出 "Hello, Webpack!"，这意味着您的 JavaScript 文件已经成功地通过 Webpack 进行了打包。

