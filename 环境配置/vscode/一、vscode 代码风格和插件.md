# 一、vscode 代码风格

## 1.1 Yapf

- 使用yapf格式化插件，然后自定义风格

- `yapf  --style-help   `导出配置文件内容至`.style.yapf`。然后进行settings.json 之中进行如下配置即可

  ```python
  {
    "python.defaultInterpreterPath": "/home/didi/anaconda3/envs/my_python/bin/python",
    "python.formatting.provider": "yapf",
    "python.formatting.yapfArgs": [
      "--style",
      "/home/didi/test/.style.yapf" # 设置代码风格
    ],
    "json.schemas": []
  }
  ```

- vscode 不再支持python3.8以下的版本，因此使用配置的方式(直接在vscode里面进行修改)

  ![image-20231110115804407](https://yrecord.oss-cn-hangzhou.aliyuncs.com/picture/202311101158008.png)

## 1.2 Clang-format

- [link](http://www.5ityx.com/cate100/295771.html)
- [ubunutu 安装最新版本](https://stackoverflow.com/questions/20756924/how-can-i-install-clang-format-in-ubuntu)

# 二、vscode安装的插件

- [找不到定义插件](https://github.com/microsoft/vscode/issues/96754)
  - vscode-msr




# 三、配置C++环境

- [参考链接](https://blog.csdn.net/reachzh1/article/details/120413575?ops_request_misc=&request_id=&biz_id=102&utm_term=mac%20vscode%E9%85%8D%E7%BD%AEc++%E7%8E%AF%E5%A2%83&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-1-120413575.142^v41^pc_rank_34_1,185^v2^control&spm=1018.2226.3001.4187)



# 四、vscode 配置文件的作用

- `lanuch.json` 运行任务设置(调试配置)
  - 主要是配置可运行文件的位置所在，对于脚本语言，比如python就可以直接使用该配置文件即可
- `tasks.json`文件夹,任务配置
  - 比如在里面配置执行shell 脚本、编译c++代码等等
- settings.sjon



# 五、Vscode 编译器预设变量

- [官网](https://code.visualstudio.com/docs/editor/variables-reference)

```python
${workspaceFolder} - VS Code 中打开的文件夹目录(也就是项目的根目录)
${workspaceFolderBasename} - 没有任何斜杠 (/)的 VS Code 中打开的文件夹目录
${file} - 目前打开文件的绝对位置
${relativeFile} - 目前打开文件相对于 workspaceFolder 的相对位置
${fileBasename} -  目前打开文件的文件名（有拓展名，如： main.cpp）
${fileBasenameNoExtension} - 目前打开文件的出去拓展名的文件名（无拓展名， 如： main.cpp）
${cwd} - task runner的工作目录
${fileDirname} - 目前打开文件的目录位置
${fileExtname} - 目前打开文件的拓展名
${lineNumber} - 文件中目前被选择的行数
${selectedText} - 文件中目前被选择的内容
```



# 六、插件

- [插件](https://hackr.io/blog/best-vscode-extensions)
  - 正则表达式插件



# 七、句柄用完了

- [gd](https://code.visualstudio.com/docs/setup/linux#_visual-studio-code-is-unable-to-watch-for-file-changes-in-this-large-workspace-error-enospc)
- [好链接](https://blog.csdn.net/lalafans/article/details/128117079?spm=1001.2101.3001.6650.6&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-128117079-blog-119081283.235%5Ev38%5Epc_relevant_default_base3&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-128117079-blog-119081283.235%5Ev38%5Epc_relevant_default_base3&utm_relevant_index=7)

# 八、额外路径设置

```python
  // 4、额外路径设置
  "python.autoComplete.extraPaths": [
    "/home/didi/land/build/devel/lib/python3/dist-packages/",
  ],
  "python.analysis.extraPaths": [
    "/home/didi/land/build/devel/lib/python3/dist-packages/",
  ]
```



# 九、设置左边侧栏缩进

- [link](https://blog.csdn.net/kunkliu/article/details/123904333)

# 十、Clangd

- 是否开启内嵌提示
  - `editor > inlay hints: enabled`


![image-20240321185637775](/Users/didi/Library/Application Support/typora-user-images/image-20240321185637775.png)

- 不要对clangd 进行资源限制，否则大项目构建很难完成

  ```json
  // 如下所示，不要进行一些兼职
  "clangd.arguments": [
      // "--background-index", // 后台索引
      // "-j=50" // Limit clangd to few cores only
    ],
  ```

  

# 十一 杂记

- editor.inlineSuggest.enabled 控制是否灰度显示建议(比如coplit 代码补全)
