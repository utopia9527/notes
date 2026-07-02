## 问题描述

在一个 CMake 项目中，使用 `ExternalProject_Add` 管理大量第三方依赖库（如 googletest、protobuf、gflags、glog、brpc 等）。通过 CMake 选项（如 `-DINSTALL_GOOGLETEST=ON`）控制是否构建特定依赖。
**现象**：当单独设置 `INSTALL_GOOGLETEST=ON` 时，googletest 编译正常且**启用了 RTTI**；但当同时开启多个依赖（如因依赖联动导致 `INSTALL_GFLAGS`、`INSTALL_GLOG` 也被强制打开）时，googletest 的编译过程丢失了 RTTI（运行时类型信息），导致链接或运行阶段出现与 RTTI 相关的错误。

## 关键代码上下文

cmake

```
# 部分依赖联动逻辑
if(INSTALL_GFLAGS OR INSTALL_GOOGLETEST OR INSTALL_GLOG)
  set(INSTALL_GLOG ON)
  set(INSTALL_GFLAGS ON)
  set(INSTALL_GOOGLETEST ON)
endif()

# googletest 的 ExternalProject 块
ExternalProject_Add(
  googletest
  URL file://${RESOURCE_PATH}googletest-1.14.0.tar.gz
  INSTALL_DIR "${SDK_PATH}"
  SOURCE_DIR "${DEP_SOURCE_DIR}/googletest"
  BINARY_DIR "${DEP_BINARY_DIR}/googletest_build"
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${SDK_PATH} -DCMAKE_BUILD_TYPE=Release
  BUILD_COMMAND make -j
  INSTALL_COMMAND make install
)
```



## 初步尝试及失败原因

### 尝试 1：显式传递 `-DCMAKE_CXX_FLAGS=-frtti`

cmake

```
CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${SDK_PATH} -DCMAKE_BUILD_TYPE=Release -DCMAKE_CXX_FLAGS=-frtti
```



**结果**：无效，RTTI 依然被禁用。

**分析**：

- CMake 命令行参数 `-DCMAKE_CXX_FLAGS=-frtti` 会设置缓存变量，但若构建过程中环境变量 `CXXFLAGS` 已被污染（例如包含 `-fno-rtti`），则：
  - CMake 配置阶段可能将环境变量值合并到缓存变量中。
  - 最终生成的 Makefile 可能直接引用环境变量 `$(CXXFLAGS)`，导致编译命令末尾追加 `-fno-rtti`，覆盖之前的 `-frtti`。
- 因此单纯修改 CMake 参数无法根治环境变量污染问题。

## 根本原因定位

### 1. 环境变量污染来源

当多个 `ExternalProject` 顺序执行时，某些依赖库的构建脚本会修改**父 CMake 进程的环境变量**，并被子进程继承。
典型的污染源是 `protobuf`（或其他使用 Autotools 的库），其 `configure` 脚本可能执行类似操作：

bash

```
export CXXFLAGS="-fno-rtti ..."
```



即使脚本未显式设置，若系统环境中已存在 `CXXFLAGS`，`configure` 也会保留并可能进一步导出。
由于 CMake 的 `ExternalProject_Add` 在运行命令时默认**继承当前进程的全部环境变量**，后续构建的 googletest 就会在一个带有 `CXXFLAGS=-fno-rtti` 的环境中编译。

### 2. 编译命令的生成顺序

googletest 构建时，实际编译行由 `make` 调用，其形式通常为：

text

```
g++ ... $(CXXFLAGS) ...
```



如果环境变量 `CXXFLAGS` 存在，Makefile 会将其展开并追加到命令行，**无论 CMake 传递了什么参数**。

### 3. 为什么单独编译 googletest 正常？

单独编译时，没有其他依赖库提前污染环境变量，`CXXFLAGS` 为空或未定义，googletest 使用编译器默认行为（通常默认启用 RTTI）。

## 最终解决方案

### 方案核心：在 ExternalProject 的构建命令中显式清除环境变量

利用 CMake 内置的 `${CMAKE_COMMAND} -E env` 命令在调用 `make` 前取消 `CXXFLAGS` 和 `CFLAGS` 环境变量。

#### 修改后的 googletest 配置

cmake

```
ExternalProject_Add(
  googletest
  URL file://${RESOURCE_PATH}googletest-1.14.0.tar.gz
  URL_HASH SHA256=...
  INSTALL_DIR "${SDK_PATH}"
  SOURCE_DIR "${DEP_SOURCE_DIR}/googletest"
  BINARY_DIR "${DEP_BINARY_DIR}/googletest_build"
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${SDK_PATH}
             -DCMAKE_BUILD_TYPE=Release
             -DCMAKE_CXX_FLAGS=-frtti    # 可选，作为双重保险
  # 清除环境变量，确保构建和安装步骤不受污染
  BUILD_COMMAND ${CMAKE_COMMAND} -E env --unset=CXXFLAGS --unset=CFLAGS make -j
  INSTALL_COMMAND ${CMAKE_COMMAND} -E env --unset=CXXFLAGS --unset=CFLAGS make install
)
```



### 补充说明

- **`CMAKE_COMMAND`**：CMake 内置变量，指向当前运行的 `cmake` 可执行文件路径。例如 `/usr/bin/cmake`。
- **`cmake -E env`**：CMake 提供的跨平台命令行工具，用于在修改环境变量后执行后续命令，语法为 `cmake -E env [--unset=NAME]... [NAME=VALUE]... COMMAND [ARGS...]`。

### 对其他依赖库的预防措施

若怀疑其他库（如 protobuf）是污染源，可对其构建步骤进行类似清理，或在父 CMake 文件开头强制清空环境变量：

cmake

```
set(ENV{CXXFLAGS} "")
set(ENV{CFLAGS} "")
```



但更推荐**仅对受影响的库使用环境隔离**，以避免意外破坏其他依赖库的构建环境。

## 验证方法

1. **检查编译命令行**
   在 googletest 构建日志中查看实际的 `g++` 命令，确认是否含有 `-fno-rtti`。正常情况应无此标志，或显式包含 `-frtti`。

2. **检查生成的库符号**
   使用 `nm -C libgtest.a | grep typeinfo` 查看是否包含 RTTI 相关符号（如 `typeinfo for testing::Test`）。若为空，则 RTTI 被禁用。

3. **添加临时诊断步骤**

   cmake

   ```
   ExternalProject_Add_Step(googletest check_env
     COMMAND ${CMAKE_COMMAND} -E echo "CXXFLAGS is: $ENV{CXXFLAGS}"
     DEPENDEES configure
   )
   ```

   

## 总结

| 问题                               | 原因                                                         | 解决方法                                                     |
| :--------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| googletest 在多依赖编译时丢失 RTTI | 其他依赖库的构建脚本污染了环境变量 `CXXFLAGS`（添加 `-fno-rtti`），导致 googletest 构建时继承该标志 | 在 googletest 的 `BUILD_COMMAND` 和 `INSTALL_COMMAND` 中使用 `cmake -E env --unset=CXXFLAGS` 清除环境变量 |

此问题揭示了 CMake `ExternalProject_Add` 在管理复杂依赖链时的环境隔离不足，建议对所有关键的构建步骤显式控制环境变量，以保证构建的可重复性。







# 快速聚焦的方案

- 单独编译看看是否可行



# 验证rtii 的命令

```
nm -C /opt/sdk/lib64/libgtest.a 2>/dev/null | grep -E "typeinfo for testing::Test"
```