### `ExternalProject_Add` 在 CMakeLists.txt 中的含义

`ExternalProject_Add` 是 CMake 中的一个模块，提供了一种下载、配置、构建和安装外部项目的方法。这对于需要依赖其他库或项目的构建过程特别有用。

#### 主要功能

`ExternalProject_Add` 允许您在 CMake 项目中定义外部项目，并控制其下载、配置、构建和安装过程。它可以处理不同来源（如 Git 仓库、压缩文件或 URL）的项目。

#### 使用示例

以下是一个简单的 `CMakeLists.txt` 示例，演示如何使用 `ExternalProject_Add` 来添加一个外部项目：

```
cmake复制代码cmake_minimum_required(VERSION 3.10)
project(MyProject)

include(ExternalProject)

# 添加外部项目
ExternalProject_Add(
    my_external_project
    GIT_REPOSITORY https://github.com/some/repo.git
    GIT_TAG v1.0
    PREFIX ${CMAKE_BINARY_DIR}/external
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=<INSTALL_DIR>
)

# 添加外部项目生成的库到主项目
add_library(my_external_lib STATIC IMPORTED)
set_target_properties(my_external_lib PROPERTIES
    IMPORTED_LOCATION ${CMAKE_BINARY_DIR}/external/lib/libmy_external_lib.a
)

# 包含外部项目的头文件路径
include_directories(${CMAKE_BINARY_DIR}/external/include)

# 目标可执行文件
add_executable(MyExecutable main.cpp)
target_link_libraries(MyExecutable my_external_lib)
```

#### 参数解释

- `my_external_project`：外部项目的名称。
- `GIT_REPOSITORY`：外部项目的 Git 仓库 URL。
- `GIT_TAG`：指定 Git 仓库的特定标签或分支。
- `PREFIX`：外部项目构建和安装的前缀目录。
- `CMAKE_ARGS`：传递给外部项目的 CMake 配置参数。

#### 详细解释

1. **引入模块**：使用 `include(ExternalProject)` 引入 `ExternalProject` 模块。

2. 定义外部项目

   ：

   - 使用 `ExternalProject_Add` 定义外部项目的来源、配置、构建和安装步骤。
   - `GIT_REPOSITORY` 指定了外部项目的 Git 仓库 URL，`GIT_TAG` 指定要检出的标签或分支。
   - `PREFIX` 设置外部项目的构建目录。
   - `CMAKE_ARGS` 传递给外部项目的 CMake 配置选项。

3. 导入外部库

   ：

   - 使用 `add_library` 和 `set_target_properties` 将外部项目生成的库导入到主项目。

4. 包含头文件路径

   ：

   - 使用 `include_directories` 包含外部项目的头文件路径。

5. 目标可执行文件

   ：

   - 定义主项目的可执行文件并链接外部库。

### 总结

`ExternalProject_Add` 是 CMake 的强大功能，用于处理外部项目的依赖管理。它使得在构建过程中自动下载、配置、构建和安装外部项目变得简单和高效。通过使用 `ExternalProject_Add`，可以确保所有依赖项都在构建过程中自动管理，减少手动处理的复杂性。如果遇到任何问题，请随时提出。







# 三种命令

- INSTALL_COMMAND 会去执行make install 命令
  - **执行顺序和依赖关系**： `ExternalProject_Add` 定义的 `CONFIGURE_COMMAND`、`BUILD_COMMAND` 和 `INSTALL_COMMAND` 是按顺序执行的。如果 `b2` 命令被放在 `INSTALL_COMMAND` 中，而 `CONFIGURE_COMMAND` 或 `BUILD_COMMAND` 没有生成正确的 Makefile 或其他构建文件，`INSTALL_COMMAND` 中的 `b2` 就无法执行
- **分离配置和构建**： `CONFIGURE_COMMAND` 负责生成初始构建配置（例如 `bootstrap.sh` 脚本），而 `BUILD_COMMAND` 实际执行构建步骤





# 执行脚本的命令所放位置

- 放在 BUILD_COMMAND 之中，放在INSTALL_COMMAND 下 会去调用make 导致错误



# 为什么一些脚本命令放在其中行不通

```cmake
ExternalProject_Add(
  boost

  URL file:///boost_1_80_0.zip
  URL_HASH SHA256=e34756f63abe8ac34b35352743f17d061fcc825969a2dd8458264edb38781782
  INSTALL_DIR "${SDK_PATH}"
  SOURCE_DIR "${DEP_SOURCE_DIR}/boost"
  BINARY_DIR "${DEP_BINARY_DIR}/boost_build"
  BUILD_COMMAND /bin/bash /install_boost.sh  
  
  # CONFIGURE_COMMAND <SOURCE_DIR>/bootstrap.sh --prefix=<INSTALL_DIR> --with-bjam=/dep/binary/boost_build/
  # BUILD_COMMAND /dep/binary/boost_build/b2 install
)

# ------ install_boost.sh-------
#!/bin/bash

DEP_SOURCE_DIR=/dep/src
SDK_PATH=/opt/sdk

cd ${DEP_SOURCE_DIR}
CONFIGURE_COMMAND ./bootstrap.sh --prefix=${SDK_PATH}
./b2 install


# 如上所示： 为什么 CONFIGURE_COMMAND不行， BUILD_COMMAND 

```



# 三个命令

- 注意三个命令最好是给`""`, 如果不指定很有可能去执行cmake 命令