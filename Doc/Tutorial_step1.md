## CMake Tutorial

> int main(int argc, char* argv[]) 
>
> `argc`表示命令行参数的个数，`argv`是一个指向字符指针数组的指针，每个指针指向一个命令行参数的字符串。通常情况下，`argv[0]`表示程序的名称，`argv[1]`、`argv[2]`等表示程序的命令行参数。也就是说，程序在不输入其他命令行参数的情况下，命令行参数个数为1。

这里介绍一下CMake中各命令的作用。

* cmake_minimum_required()：当前工程支持的cmake最小版本

  ```cmake
  cmake_minimum_required(VERSION 3.10)
  ```

* project()：设置当前工程名

  ```cmake
  project(Tutorial)
  ```

* add_executable()：添加可执行文件以及对应需要编译的源码文件。

  ```cmake
  add_executable(Tutorial tutorial.cxx)
  ```

* configure_file()：读取 **xx.h.in** 格式文件同时替换占位符，生成对应的 **xx.h** 头文件。

  ```cmake
  project(Tutorial VERSION 1.0)
  configure_file(TutorialConfig.h.in TutorialConfig.h)
  ```

  这里通过project命令确定了项目的主版本号 1 和次版本号 0 ，可以在` TotorialConfig.h.in` 中通过占位符调用各自的版本号值。

  ```c++
  // TotorialConfig.h.in 文件
  // the configured options and settings for Tutorial
  #define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
  #define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
  
  // 构建后自动生成的TotorialConfig.h内容为
  // the configured options and settings for Tutorial
  #define Tutorial_VERSION_MAJOR 1
  #define Tutorial_VERSION_MINOR 0
  ```
  
  `@Tutorial_VERSION_MAJOR@`**值为1**，`@Tutorial_VERSION_MINOR@`值为0，需要说明一下这里版本号占位符的命名规则
  
  ​		代表项目版本号的占位符的命名规则是： `@PROJECT_NAME_VERSION_MAJOR@`：代表项目的主版本号。 `@PROJECT_NAME_VERSION_MINOR@`：代表项目的次版本号。 `@PROJECT_NAME_VERSION_PATCH@`：代表项目的修订版本号。其中，`PROJECT_NAME`是在`project()`命令中指定的项目名称。如果你的项目名称是`MyProject`，那么在`TutorialConfig.h.in`文件中，代表项目版本号的占位符应该是
  
  `@MyProject_VERSION_MAJOR@`
  
  `@MyProject_VERSION_MINOR@`
  
  `@MyProject_VERSION_PATCH@`
  
  还有一点，当我们使用`configure_file`命令生成配置文件（这里是头文件 `TutorialConfig.h`）时，配置文件会被写入到二进制树中，这意味着，生成的配置文件不会被放置在源代码目录中，而是会被放置在构建目录中。因此，为了能够在源代码中包含这个配置文件，我们需要将构建目录添加到**包含文件搜索路径列表**中，有两种方法：
  
  ```cmake
  # 方法一
  include_directories(${CMAKE_BINARY_DIR})
  # 方法二，better
  target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
  ```
  
  * `CMAKE_BINARY_DIR`和`PROJECT_BINARY_DIR`内容相同，都是项目的构建目录，可以用`message`命令打印
  
    ```cmake
    message("The value of MY_VARIABLE is ${MY_VARIABLE}")
    ```
  
    `${MY_VARIABLE}` 将会被替换为变量 `MY_VARIABLE` 的值。在 CMake 中，变量名通常使用大写字母。
  
  * `target_include_directories`和`include_directories`都可以用来将目录添加到包含文件搜索路径列表中，但它们的作用范围不同。`include_directories`命令将目录添加到全局包含文件搜索路径列表中，这意味着所有的目标都可以访问这个目录中的头文件。而`target_include_directories`命令则是将目录添加到特定目标的包含文件搜索路径列表中，这意味着只有这个目标（这里指 Tutorial）可以访问这个目录中的头文件。
  
    在这个例子中，`target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")`命令将`${PROJECT_BINARY_DIR}`目录添加到`Tutorial`目标的包含文件搜索路径列表中，并且将其设置为`PUBLIC`属性。这意味着，不仅`Tutorial`目标可以访问`${PROJECT_BINARY_DIR}`目录中的头文件，其他依赖于`Tutorial`的目标也可以访问这个目录中的头文件。
  
    相比之下，`include_directories(${CMAKE_BINARY_DIR})`命令将`${CMAKE_BINARY_DIR}`目录添加到**全局包含文件搜索路径列表**中，这意味着所有的目标都可以访问`${CMAKE_BINARY_DIR}`目录中的头文件。但是，这种方法可能会导致命名冲突或者不必要的头文件包含，因为所有的目标都可以访问这个目录中的头文件。因此，最好使用`target_include_directories`命令将目录添加到特定目标的包含文件搜索路径列表中。
  
* CMAKE_CXX_STANDARD / CMAKE_CXX_STANDARD_REQUIRED

  ```cmake
  # specify the C++ standard
  set(CMAKE_CXX_STANDARD 11)
  set(CMAKE_CXX_STANDARD_REQUIRED True)
  ```

  `CMAKE_CXX_STANDARD`是一个CMake变量，用于指定C++标准的版本。它告诉CMake编译器应该使用哪个C++标准来编译代码。这个变量的值可以是C++11、C++14、C++17等。如果你将`CMAKE_CXX_STANDARD`设置为C++11，CMake会自动为你设置编译器选项“-std=c++11”，以确保编译器使用C++11标准来编译代码。

  `CMAKE_CXX_STANDARD_REQUIRED` 是一个CMake变量，用于指定是否需要编译器支持指定的C++标准。如果将其设置为ON，则表示编译器必须支持指定的C++标准，否则CMake会报错并停止构建。通常情况下，建议将`CMAKE_CXX_STANDARD_REQUIRED`设置为ON，以确保代码在编译时使用正确的C++标准。

  
