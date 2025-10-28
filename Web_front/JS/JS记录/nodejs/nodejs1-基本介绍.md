# 一、项目管理

- npm 是什么
  - npm 是 Node Package Manager 的缩写，是一个用于管理和共享 JavaScript 代码的包管理工具。<font color=yellow>它是 Node.js 平台的默认包管理器</font>，也是世界上最大的开源软件注册表之一。通过 npm，开发者可以安装、发布和共享代码包，方便地管理项目的依赖关系和版本控制
- [package.json 文件作用](https://blog.csdn.net/song_6666/article/details/123739622?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169436154216800182751940%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=169436154216800182751940&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-123739622-null-null.142^v93^chatgptT3_1&utm_term=package.json%20%E6%98%AF%E4%BB%80%E4%B9%88&spm=1018.2226.3001.4187)
  - package.json 文件其实就是对项目或者模块包的描述，里面包含许多元信息。比如项目名称，项目版本，项目执行入口文件，项目贡献者等等。**npm install** 命令会根据这个文件下载所有依赖模块
  - package.json 文件创建有两种方式，手动创建或者自动创建
    - 手动创建： 直接在项目根目录新建一个 package.json 文件，然后输入相关的内容
    - 自动创建： 也是在项目根目录下执行 npm init，然后根据提示一步步输入相应的内容完成后即可自动创建



# 二、项目使用

- nodejs 的代码无法在游览器之中直接运行，需要打包引入使用

# 三、选项

- `npm start`是npm的内置命令之一，用于启动项目。它会在项目的根目录下寻找`package.json`文件，并且在该文件中查找`scripts`字段中的`start`命令。如果找到了该命令，就会执行该命令来启动项目，否则会报错
- npm run也是npm的命令，用于运行package.json文件中的自定义脚本。它的语法是npm run script_name，其中script_name是在package.json文件中定义的脚本名称。与npm start不同的是，npm run可以运行除了start以外的其他自定义脚本，而且必须显式地指定脚本名称。 总之，npm start是一个特殊的、内置的命令，用于启动项目；而npm run则是一个通用的命令，用于运行自定义脚本



# 四、package.json

- [link](https://blog.csdn.net/qq_34703156/article/details/121401990?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522169436800116800192225258%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=169436800116800192225258&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-121401990-null-null.142^v93^chatgptT3_1&utm_term=package.json&spm=1018.2226.3001.4187)

- 在每个前端项目中，都有 package.json 文件，它是项目的配置文件，常见的配置有配置项目启动、打包命令，声明依赖包等

## 4.1 描述信息

- 无关重要，具体查看链接

- description
  - description 字段用来描述这个项目包，它是一个字符串，可以让其他开发者在 npm 的搜索中发现我们的项目包
- keywords
  - keywords 字段是一个字符串数组，表示这个项目包的关键词。和 description 一样，都是用来增加项目包的曝光率的。下面是 eslint 包的描述和关键词
- author
  - 作者
- contributors
  - 贡献者
- homepage
  - 项目主页

## 4.2 依赖信息

- 通常情况下，我们的项目会依赖一个或者多个外部的依赖包，根据依赖包的不同用途，可以将他们配置在下面的五个属性下：dependencies、devDependencies、peerDependencies、bundledDependencies、optionalDependencies 
- dependencies
  - dependencies 字段中声明的是项目的生产环境中所必须的依赖包。当使用 npm 或 yarn 安装 npm 包时，该 npm 包会被自动插入到此配置项中
  - 当在安装依赖时使用 --save 参数，也会将新安装的 npm 包写入 dependencies 属性
- devDependencies
  - devDependencies 中声明的是开发阶段需要的依赖包，如 Webpack、Eslint、Babel 等，用于辅助开发。它们不同于 dependencies，因为它们只需安装在开发设备上，而无需在生产环境中运行代码。当打包上线时并不需要这些包，所以可以把这些依赖添加到 devDependencies 中，这些依赖依然会在本地指定 npm install 时被安装和管理，但是不会被安装到生产环境中
- peerDependencies
  - 用来供插件指定其所需要的主工具的版本
  - 如果需要在找不到包或者安装包失败时，npm 仍然能够继续运行，则可以将该包放在 optionalDependencies 对象中，optionalDependencies 对象中的包会覆盖 dependencies 中同名的包，所以只需在一个地方进行设置即可
- bundledDependencies
  - 上面的几个依赖相关的配置项都是一个对象，而 bundledDependencies 配置项是一个数组，数组里可以指定一些模块，这些模块将在这个包发布时被一起打包
- optionalDependencies 
  - 如果需要在找不到包或者安装包失败时，npm 仍然能够继续运行，则可以将该包放在 optionalDependencies 对象中，optionalDependencies 对象中的包会覆盖 dependencies 中同名的包，所以只需在一个地方进行设置即可

## 4.3 脚本配置

- scripts 是 package.json 中内置的脚本入口，是 key-value 键值对配置，key 为可运行的命令，可以通过 npm run 来执行命令。除了运行基本的 scripts 命令，还可以结合 pre 和 post 完成前置和后续操作
- config 字段用来配置 scripts 运行时的配置参数

## 4.4 文件和目录

- main 字段用来指定加载的入口文件，在 browser 和 Node 环境中都可以使用。如果我们将项目发布为 npm 包，那么当使用 require 导入 npm 包时，返回的就是 main 字段所列出的文件的 module.exports 属性。如果不指定该字段，默认是项目根目录下的 index.js。如果没找到，就会报错
- browser 字段可以定义 npm 包在 browser 环境下的入口文件。如果 npm 包只在 web 端使用，并且严禁在 server 端使用，使用 browser 来定义入口文件
- module 字段可以定义 npm 包的 ESM 规范的入口文件，browser 环境和 node 环境均可使用。如果 npm 包导出的是 ESM 规范的包，使用 module 来定义入口文件
- bin 字段用来指定各个内部命令对应的可执行文件的位置
- files 配置是一个数组，用来描述当把 npm 包作为依赖包安装时需要说明的文件列表。当 npm 包发布时，files 指定的文件会被推送到 npm 服务器中，如果指定的是文件夹，那么该文件夹下面所有的文件都会被提交
- man 命令是 Linux 中的帮助指令，通过该指令可以查看 Linux 中的指令帮助、配置文件帮助和编程帮助等信息。如果 node.js 模块是一个全局的命令行工具，在 package.json 通过 man 属性可以指定 man 命令查找的文档地址
- directories 字段用来规范项目的目录。node.js 模块是基于 CommonJS 模块化规范实现的，需要严格遵循 CommonJS 规范。模块目录下除了必须包含包项目描述文件 package.json 以外，还需要包含以下目录

## 4.5 发布和配置

# 五、webpack

- webpack本身是, node的一个第三方模块包, 用于打包代码
- [link](https://www.w3cschool.cn/webpackguide/webpackguide-cue52777.html)