- vscode 插件不再支持python3.6， 因此一些语法提示插件和格式化插件需要额外的配置

  - Pylance: `2022.8.10`

  - Python:`2022.8.10`

  - 格式化插件无效,需要进行如下配置

    ```json
      "python.formatting.yapfArgs": [
        "--style={based_on_style: google, indent_width: 2, allow_split_before_dict_value: false,indent_dictionary_value: true,split_penalty_import_names=300}"
      ],
    ```

  - Python3.7

    - Pylance: 2023.8.10
    - Python: 2023.6.1