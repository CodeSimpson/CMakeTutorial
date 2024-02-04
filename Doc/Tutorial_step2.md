## CMake Tutorial

这里向`Tutorial`项目中添加单独的库`MathFunctions`（这里还是静态库），使项目能调用该库。

#### 1.编译新库 MathFunctions

首先在当前项目文件下新建目录 MathFunctions，添加源文件`mysprt.cxx`（计算平方根）和头文件`MathFunctions.h`，在当目录下添加新的mk文件`CMakeLists.txt`，编写以下指令。

* add_library(): 创建一个库文件，这里会生成一个静态库文件 `libMathFunctions.a`。

```cmake
add_library(MathFunctions mysqrt.cxx)
```

为了能使MathFunctions中的cmake文件能被查找编译到，这里我们需要在最上层的项目目录的CMakeLists.txt文件中添加当前cmake文件的路径。

* add_subdirectory(): 向CMake构建系统添加一个子目录。

```cmake
# add the MathFunctions library
add_subdirectory(MathFunctions)
```

这个命令告诉CMake在当前项目中包含一个子目录 MathFunctions，并在该子目录中查找CMakeLists.txt文件，以便**在该子目录中构建一个独立的项目**。

#### 2. 链接新库 MathFuntions

* target_link_libraries(): 将一个或多个库链接到目标可执行文件或库中。

```cmake
target_link_libraries(Tutorial PUBLIC MathFunctions)
```

这个命令的作用是告诉 CMake 在链接目标时需要使用哪些库。在编译过程中，CMake 会自动查找这些库的位置，并将它们链接到目标中。这样，当目标被执行时，它就可以使用这些库中的函数和变量了。

#### 3. 添加新库接口对应的头文件

* target_include_directories(): 将目录添加到*头文件搜索路径列表**中。

```cmake
target_include_directories(Tutorial PUBLIC "${PROJECT_SOURCE_DIR}/MathFunctions")
```

#### 4. 添加CMake命令行选项

这里我们添加一个命令行选项，在编译的时候决定项目`Tutorial`是使用自定义的求平方根方法还是使用c++接口。

* option(): 定义一个布尔类型的选项，它可以用来控制 CMake 构建过程中的一些行为。

```cmake
option(USE_MYMATH "Use tutorial provided math implementation" ON)
```

这里`USE_MYMATH`是选项名称，"Use...implementation"是选项的帮助文本，`ON`为选项的初始值，默认为ON。当使用了 option() 定义一个选项后，CMake会自动生成一个对应的变量，变量名就是选项名称 `USE_MYMATH`，如果在**构建过程**（注意不是编译过程）中使用了 `-DUSE_MYMATH=OFF`的命令行选项，那么这个变量值就会被设置为OFF。

```shell
cmake ../Step2 -DUSE_MYMATH=OFF
```

我们直接在CMakeLists.txt中利用变量`USE_MYMATH`来写条件构建命令。

```cmake
if(USE_MYMATH)
  add_subdirectory(MathFunctions)
  list(APPEND EXTRA_LIBS MathFunctions)
  list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()

...

target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

target_include_directories(Tutorial PUBLIC
                           ${EXTRA_INCLUDES}
                           )
```

这里在构建时，变量`USE_MYMATH`的值在不通过命令行选项修改的情况下就会被当做ON，使 if 条件成立。

需要注意的是，**CMakeLists.txt中通过`option()`命令定义的变量不能被直接用到源文件或头文件中使用**，需要通过`configure_file`命令将其传递给源代码或头文件中。如在Step1中使用的一样，通过`TutorialConfig.h.in`文件获取`USE_MYMATH`的值，其中的`@USE_MYMATH@`将被替换为CMakeLists.txt中定义的`USE_MYMATH`变量的值。

```cmake
configure_file(TutorialConfig.h.in TutorialConfig.h)
```

```c++
// TutorialConfig.h.in 文件
#cmakedefine USE_MYMATH
```

这里就可以在头文件 / 源文件中使用`USE_MYMATH`

```c++
// tutorial.cxx 文件
#ifdef USE_MYMATH
#include "MathFunctions.h"
#endif

...
    
#ifdef USE_MYMATH
  const double outputValue = mysqrt(inputValue);
#else
  const double outputValue = sqrt(inputValue);
#endif
```

* #cmakedefine: CMake 中的一个预处理指令，用于在 CMakeLists.txt 文件中定义一个 C++ 宏。它的作用类似于 C++ 中的 `#define`，但是它是在 CMake 生成 Makefile 或者其他构建系统的过程中使用的。

```cmake
#cmakedefine USE_MYMATH
```

具体来说，`#cmakedefine` 用于在 CMakeLists.txt 文件中定义一个 C++ 宏，这个宏的值可以在 C++ 代码中使用。在 CMake 生成 Makefile 或者其他构建系统的过程中，CMake 会根据这个宏的定义生成相应的代码。

这里如果 `USE_MYMATH`的值为 ON，则`TutorialConfig.h.in`文件中使用 `#cmakedefine USE_MYMATH` 时，会生成 `#define USE_MYMATH`，如果 `USE_MYMATH`的值为 OFF，则会生成`/* #undef USE_MYMATH */`



最后补充一下 `list()`命令的用法:

在CMake中，`list()`函数用于操作列表（list）类型的变量。它可以用来创建、修改、查询和删除列表中的元素。

以下是`list()`函数的一些常见用法：

- `list(APPEND <list> <element1> <element2> ...)`：将一个或多个元素添加到列表的末尾。
- `list(INSERT <list> <index> <element>)`：在指定索引处插入一个元素。
- `list(REMOVE_ITEM <list> <item1> <item2> ...)`：从列表中删除指定的元素。
- `list(REMOVE_AT <list> <index>)`：从列表中删除指定索引处的元素。
- `list(REVERSE <list>)`：反转列表中的元素顺序。
- `list(SORT <list>)`：按字典顺序对列表中的元素进行排序。

这些操作可以用于处理CMake中的各种列表类型，例如`CMAKE_MODULE_PATH`、`CMAKE_PREFIX_PATH`、`CMAKE_INCLUDE_PATH`等。

