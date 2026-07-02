# 一、find_package 和 find_path、find_library的区别

- find_packaget 和最后两个最大的区别就是 find_packaget 的使用需要提前有相关的配置文件来进行支撑

```cmake
1、find_package： cmake 去寻找对应的包路径，并且存储在"一些变量中"。直接使用即可
	# 补充相关用例
	find_package(leveldb REQUIRED)
	target_link_libraries(echo_server leveldb::leveldb)


find_path、find_library
# 1、寻找头文件位置，得到对应头文件所在的前缀，然后存储至GFLAGS_INCLUDE_PATH 变量之中
find_path(GFLAGS_INCLUDE_PATH gflags/gflags.h)
# 2、暴漏头文件位置，#include 包含的时候可以省去前缀, include_directories 会暴露给接下来的所有项目(target_include_directories 只会暴露给当前项目)
include_directories(${GFLAGS_INCLUDE_PATH})
# 3、寻找库文件
CMAKE_LIBRARY_PATH(GFLAGS_LIBRARY NAMES gflags libgflags)
# 几句哪些库的链接
target_link_libraries(echo_server ${GFLAGS_LIBRARY})

```

# 二、使用find_package时如何确定包名称

- **CMake Find Modules**：许多库提供了 CMake 的 Find Module 或 Config 文件，这些文件在库的安装过程中被放置在一个标准的位置，允许 CMake 自动发现并包括这些库。例如，如果你安装了一个库并且它包含了 `Find<Package>.cmake` 或 `<Package>Config.cmake` 文件，那么 `<Package>` 就是你应该在 `find_package` 中使用的名称。

- `find_package` 根据以下几个目录来查找库

  - **CMAKE_PREFIX_PATH** - 这是一个环境变量或 `CMake` 变量，指向可能包含库的根目录。在这个目录下，`CMake` 会查找 `lib/cmake/<package>` 或 `lib64/cmake/<package>`、`share/cmake/<package>` 等子目录
  - **CMAKE_MODULE_PATH** - 这是一个变量，包含了 `Find<Package>.cmake` 模块文件的路径。这是非官方或自定义的查找模块的存放位置
  - 默认的系统安装路径** - 这些路径通常包括 `/usr/local/`、`/usr/` 等，以及在 Windows 上的默认程序安装路径

- ### `find_package` 可以在两种模式下运行：Config 模式和 Module（Find） 模式。

  - **Config 模式**：在这种模式下，`find_package` 查找由库提供的配置文件（通常命名为 `<package>Config.cmake` 或 `<package>-config.cmake`）。这些文件通常由库的开发者提供，并在库的安装过程中放置在适当的目录中。
  - "Module"(Find) 模式**：如果 Config 模式失败，`find_package` 会退回到 Find 模式。在这个模式下，`CMake` 使用一个名为 `Find<Package>.cmake` 的模块，**这是一个由 `CMake` 或社区提供，用于寻找和设置库的脚本**(也就是说看cmake 社区是否支持)

- 查找过程

  - 当运行 `find_package(<PackageName>)` 时，`CMake` 遵循这些步骤：

    1. **查找配置文件**：首先在 `CMAKE_PREFIX_PATH` 指定的路径中查找 `<PackageName>Config.cmake` 或 `<PackageName>-config.cmake`。
    2. **使用 Find 模块**：如果在上述路径中没有找到配置文件，`CMake` 会查找 `CMAKE_MODULE_PATH` 和默认的模块目录，尝试加载 `Find<PackageName>.cmake` 模块。
    3. **系统路径和附加路径**：如果上述步骤都未成功，`CMake` 会在系统的标准安装路径中查找库。

    

# 三、以protobuf 为示例

- 按照官方文档进行protobuf-cpp 版本的安装(3.14)，安装在目录`opt/sdk/lib` 之下

- 编写Cmakelists 进行相关测试

  ```cmake
  set(CMAKE_PREFIX_PATH "/opt/sdk/")
  find_package(Protobuf REQUIRED)
  
  
  message("_______")
  message("${PROTOBUF_INCLUDE_DIRS}")  # 找到头文件的位置
  message("${PROTOBUF_LIBRARIES}")     # 找到库文件的位置
  message("_______")
  
  # 问题
  # 1、Protobuf 名字的由来，为什么不是
  
  
  # FindProtobuf.cmake 文件中的部分内容
  Example:
  
  .. code-block:: cmake
  
    find_package(Protobuf REQUIRED)
    include_directories(${Protobuf_INCLUDE_DIRS})
    include_directories(${CMAKE_CURRENT_BINARY_DIR})
    protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS foo.proto)
    protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS EXPORT_MACRO DLL_EXPORT foo.proto)
    protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS DESCRIPTORS PROTO_DESCS foo.proto)
    protobuf_generate_python(PROTO_PY foo.proto)
    add_executable(bar bar.cc ${PROTO_SRCS} ${PROTO_HDRS})
    target_link_libraries(bar ${Protobuf_LIBRARIES})
  ```

- 相关问题

  - Protobuf 名字的由来，为什么不是protobuf ？
    - **`Find<Package>.cmake` 或 `<Package>Config.cmake` 文件，那么 `<Package>` 就是你应该在 `find_package` 中使用的名称**
    - 安装在了 /usr/share/cmake/Modules/FindProtobuf.cmake 
  - PROTOBUF_INCLUDE_DIRS、PROTOBUF_LIBRARIES 都是大写的么
    - `FindProtobuf.cmake ` 之中有说明

# 四、演示

### Config 模式

- 生成版本文件

- 生成配置文件

  - testConfig.cmake  : 根据手动写的testConfig.in 模板文件来的。在调用configure_package_config_file 时会进行变量替换

    ```cmake
    # 假设文件名称是： testConfig.cmake.in
    @PACKAGE_INIT@
    
    include(CMakeFindDependencyMacro)
    
    set(TEST_INCLUDE_DIRS "@INSTALL_INCLUDE_DIR@") # 会进行变量替换(CMake当中写的)
    set(TEST_LIBRARIES "@TEST_LIB_PATH@") # 会进行变量替换(CMake当中写的)
    
    include("${CMAKE_CURRENT_LIST_DIR}/testTargets.cmake")
    
    # (在使用时，target_linK的名称就可以是 TEST_INCLUDE_DIRS、TEST_LIBRARIES
    # find_package 的名称是test(根据testConfig.camke.in截断而来)# )
    ```

    

  - testConfigVersion.cmake  : write_basic_package_version_file 写入版本信息

  - testTargets.cmake ： install EXPORT 导出安装的

- 库文件、头文件安装

- find_package 需要配置文件

#### 1. 生成 `test_libConfig.cmake` 和 `test_libTargets.cmake`

在 `CMakeLists.txt` 文件中添加配置文件的生成和安装指令：

```cmake
cmake_minimum_required(VERSION 3.17)
project(metric)

# 生成 test_lib库
add_library(test_lib test.cpp)

# 安装库文件
# EXPORT 是 find_package 需要用到的,因此下面的都需要进行相关填充，EXPORT 根据填充生成配置文件
# LIBRARY 动态库的位置、ARCHIVE 静态库的位置、RUNTIME 可执行文件
install(TARGETS test_lib
  EXPORT test_libTargets
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
  RUNTIME DESTINATION bin
  INCLUDES DESTINATION include
)

# 安装头文件
install(FILES test.h DESTINATION include)

# 生成并安装配置文件
# CMAKE_CURRENT_BINARY_DIR: 当前build 目录
# 在build 目录下生成 test_libConfigVersion.cmake文件, 将版本信息写入其中
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/test_libConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(EXPORT test_libTargets
  FILE test_libTargets.cmake
  NAMESPACE test_lib::
  DESTINATION lib/cmake/test_lib
)

# 根据模板文件进行生成
configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/test_libConfig.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/test_libConfig.cmake"
  INSTALL_DESTINATION lib/cmake/test_lib
  PATH_VARS INSTALL_INCLUDE_DIR
)

install(FILES
  "${CMAKE_CURRENT_BINARY_DIR}/test_libConfig.cmake"
  "${CMAKE_CURRENT_BINARY_DIR}/test_libConfigVersion.cmake"
  DESTINATION lib/cmake/test_lib
)
```

在 `cmake` 目录下创建 `test_libConfig.cmake.in` 文件：

```cmake
cmake复制代码@PACKAGE_INIT@

include(CMakeFindDependencyMacro)

# test_lib_INCLUDE_DIRS 就是目录的名称

set(test_lib_INCLUDE_DIRS "@PACKAGE_INCLUDE_INSTALL_DIR@") # PACKAGE_INCLUDE_INSTALL_DIR 会替换成INSTALL_INCLUDE_DIR

set(test_lib_LIBRARIES test_lib)

include("${CMAKE_CURRENT_LIST_DIR}/test_libTargets.cmake")
```

### Module 模式

#### 1. 创建 `Findtest_lib.cmake`

在 `cmake` 目录下创建 `Findtest_lib.cmake` 文件：

```
cmake复制代码# Findtest_lib.cmake

find_path(test_lib_INCLUDE_DIR test.h PATH_SUFFIXES include)
find_library(test_lib_LIBRARY NAMES test_lib PATH_SUFFIXES lib)

if (test_lib_INCLUDE_DIR AND test_lib_LIBRARY)
  set(test_lib_FOUND TRUE)
  set(test_lib_LIBRARIES test_lib)
  set(test_lib_INCLUDE_DIRS test_lib_INCLUDE_DIR)
else()
  set(test_lib_FOUND FALSE)
endif()

mark_as_advanced(test_lib_INCLUDE_DIR test_lib_LIBRARY)
```

将 `Findtest_lib.cmake` 放置在 CMake 可以找到的路径中，例如 `/usr/share/cmake/Modules`，或者用户手动指定模块路径。

### 示例项目结构

假设项目结构如下：

```
复制代码metric/
├── CMakeLists.txt
├── cmake/
│   ├── test_libConfig.cmake.in
│   └── Findtest_lib.cmake
├── include/
│   └── test.h
└── src/
    └── test.cpp
```

### 总结

通过提供 `test_libConfig.cmake`、`test_libTargets.cmake` 文件和 `Findtest_lib.cmake` 文件，你可以让用户通过 `find_package(test_lib)` 查找并使用你的库。Config 模式适合发布库，而 Module 模式为用户提供了查找库的标准化方式。