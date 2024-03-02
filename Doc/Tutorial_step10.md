## CMake Tutorial Step10: Selecting Static or Shared Libraries

​	在前面的Step中，`MathFunctions`库被变为一个静态库编入可执行文件`Tutorial`中，若MathFunctions库代码发生变化，则需要全编整个项目，更新和维护不方便。这里我们引入动态库的形式，在执行`cmake`命令时使用`-D`选项，控制`BUILD_SHARED_LIBS`的值，隐式的将`MathFunctions`编为动态库的形式，提高项目的可维护性。采用动态库`MathFunctions`后，宏`USE_MYMATH`将不再控制`MathFunctions`是否被编译，而是控制动态库内部的逻辑，库本身始终被编译出来。

* 设置CMake项目的输出目录

```cmake
# CMakeLists.txt
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}")
```

具体来说，它将 CMake 项目的静态库、动态库和可执行文件的输出目录都设置为 `${PROJECT_BINARY_DIR}`，也就是项目的二进制目录。

​	a.`CMAKE_LIBRARY_OUTPUT_DIRECTORY` 用来设置动态库（共享库）的输出目录；

​	b. `CMAKE_RUNTIME_OUTPUT_DIRECTORY` 用来设置可执行文件的输出目录；

​	c.`CMAKE_ARCHIVE_OUTPUT_DIRECTORY` 是 CMake 中用来设置静态库输出目录的变量。

* 控制`MathFunctions`编译为动态库

```cmake
# CMakeLists.txt
option(BUILD_SHARED_LIBS "build using shared libraries" ON)
```

定义一个 CMake 变量 `BUILD_SHARED_LIBS`，用于控制是否构建共享库。如果 `BUILD_SHARED_LIBS` 的值为 `ON`，则构建共享库；如果 `BUILD_SHARED_LIBS` 的值为 `OFF`，则构建静态库。 

这个变量的默认值是 `OFF`，也就是默认情况下构建静态库【这里被默认改为`ON`】。如果需要构建共享库，可以在执行 `cmake` 命令时通过 `-DBUILD_SHARED_LIBS=ON` 参数来设置。

```cmake
# MathFunctions/CMakeLists.txt
add_library(MathFunctions MathFunctions.cxx)
```

这里的 `MathFunctions` 库的类型取决于 `BUILD_SHARED_LIBS` 变量的值。如果 `BUILD_SHARED_LIBS` 的值为 `ON`，则 `MathFunctions` 库是一个动态库；如果 `BUILD_SHARED_LIBS` 的值为 `OFF`，则 `MathFunctions` 库是一个静态库。在上一级的`CMakeLists.txt`文件中，`BUILD_SHARED_LIBS`被设为`ON`。

* 添加动态库`MathFunctions`的头文件路径

```cmake
# MathFunctions/CMakeLists.txt
target_include_directories(MathFunctions
						INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

这句CMake代码的作用是将 `MathFunctions` 库所对应的头文件搜索路径添加到 `MathFunctions` 库的接口中。这样，在使用 `MathFunctions` 库的其他项目中，只需要使用 `target_link_libraries` 命令链接 `MathFunctions` 库即可，而不需要再手动添加 `MathFunctions` 库的头文件搜索路径。

这里不添加自动生成的`table.h`所在的路径`CMAKE_CURRENT_BINARY_DIR`到动态库`MathFunctions`所对应的头文件搜索路径中是因为`MathFunctions`所对应的源码没有用到`table.h`，它将在静态库`SqrtLibrary`中被使用。

> `MathFunctions` 库的接口一般会包含以下内容：
>
> 1. 头文件搜索路径：通过 `target_include_directories` 命令将库的头文件搜索路径添加到库的接口中，使得其他项目在使用该库时可以直接包含该库的头文件。
>
> 2. 编译选项：通过 `target_compile_options` 命令将库的编译选项添加到库的接口中，使得其他项目在使用该库时可以继承该库的编译选项。
>
> 3. 链接选项：通过 `target_link_options` 命令将库的链接选项添加到库的接口中，使得其他项目在链接该库时可以继承该库的链接选项。
>
> 4. 链接库：通过 `target_link_libraries` 命令将库所依赖的其他库添加到库的接口中，使得其他项目在链接该库时可以自动链接该库所依赖的其他库。
>
> 5. 宏定义：通过 `target_compile_definitions` 命令将库的宏定义添加到库的接口中，使得其他项目在使用该库时可以继承该库的宏定义。
>
> 这些内容可以根据实际需要进行添加或者删除。

* 控制是否使用自定义的求平方根接口

```cmake
# MathFunctions/CMakeLists.txt
# should we use our own math functions
option(USE_MYMATH "Use tutorial provided math implementation" ON)
```

通过变量`USE_MYMATH`控制使用使用自定义的开平方根接口。

* 添加静态库`SqrtLibrary`

```cmake
# MathFunctions/CMakeLists.txt
if(USE_MYMATH)

  target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")

  # first we add the executable that generates the table
  add_executable(MakeTable MakeTable.cxx)
  target_link_libraries(MakeTable PRIVATE tutorial_compiler_flags)

  # add the command to generate the source code
  add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
    DEPENDS MakeTable
    )

  # library that just does sqrt
  add_library(SqrtLibrary STATIC
              mysqrt.cxx
              ${CMAKE_CURRENT_BINARY_DIR}/Table.h
              )

  # state that we depend on our binary dir to find Table.h
  target_include_directories(SqrtLibrary PRIVATE
                             ${CMAKE_CURRENT_BINARY_DIR}
                             )

  target_link_libraries(SqrtLibrary PUBLIC tutorial_compiler_flags)
  target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()
```

​	a. 为目标`MathFunction`添加编译器定义`USE_MYMATH`，在`MathFunction`对应的源码【MathFunctions.cxx / MathFunctions.h】中使用。

​	b. 自动生成`Table.h`文件，与`Step8`相同。

​	c. 显性的添加静态库`SqrtLibrary`，将`table.h`所在的路径添加到库`SqrtLibrary`搜索路径中。

```cmake
  # library that just does sqrt
  add_library(SqrtLibrary STATIC
              mysqrt.cxx
              ${CMAKE_CURRENT_BINARY_DIR}/Table.h
              )
   # state that we depend on our binary dir to find Table.h
  target_include_directories(SqrtLibrary PRIVATE
                             ${CMAKE_CURRENT_BINARY_DIR}
                             )
```

​	d. 链接`tutorial_compiler_flags`添加编译选项，采用PUBLIC的方式，link的`SqrtLibrary`的库也会继承对应的编译选项。

​	e. `MathFunctions`以私有的形式链接静态库`SqrtLibrary`，`SqrtLibrary`中的内容将会复制到`MathFunctions`中。因此若安装的过程中不包含`SqrtLibrary`，程序也能照常运行。

* 兼容win32平台

```cmake
# MathFunctions/CMakeLists.txt
# define the symbol stating we are using the declspec(dllexport) when
# building on windows
target_compile_definitions(MathFunctions PRIVATE "EXPORTING_MYMATH")
```

* 为`MathFunctions/mysqrt.cxx`中的`mysqrt()`接口添加命名空间。

* 修改动态库`MathFunctions`的头文件

```c++
#if defined(_WIN32)
#  if defined(EXPORTING_MYMATH)
#    define DECLSPEC __declspec(dllexport)
#  else
#    define DECLSPEC __declspec(dllimport)
#  endif
#else // non windows
#  define DECLSPEC
#endif

namespace mathfunctions {
double DECLSPEC sqrt(double x);
}
```

这样可执行文件`tutorial.cxx`对应的源码中就不需要通过宏控制不同的接口调用，直接调用sqrt()接口即可。

```c++
const double outputValue = mathfunctions::sqrt(inputValue);
```

*  设置静态库`libSqrtLibrary`对应的目标文件为位置无关代码**PIC**。

通过以上修改，直接构建并编译项目，会出现报错。

```cmake
cmake ../Step10
cmake --build .
```

报错:

```cmake
/usr/bin/ld: ../libSqrtLibrary.a(mysqrt.cxx.o): relocation R_X86_64_PC32 against symbol `_ZSt4cout@@GLIBCXX_3.4' can not be used when making a shared object; recompile with -fPIC
```

这个错误是由于在链接时使用了静态库 `libSqrtLibrary.a`，但是在生成共享库时，**链接器要求所有的目标文件都必须是位置无关的代码（PIC）**，而静态库中的目标文件通常不是位置无关的代码。因此，链接器无法将静态库中的目标文件与共享库链接。 要解决这个问题，可以重新编译 `libSqrtLibrary.a`，使用 `-fPIC` 选项生成位置无关的代码。

这里我通过设置`SqrtLibrary`的属性来控制`libSqrtLibrary.a`生成位置无关代码。

```cmake
# MathFunctions/CMakeLists.txt
  set_target_properties(SqrtLibrary PROPERTIES
                        POSITION_INDEPENDeNT_CODE
                        ${BUILD_SHARED_LIBS})
```

> 位置无关代码（PIC）是指在编译时不依赖于代码的加载地址，而是在运行时通过重定位表来确定代码的实际地址。这种代码可以被加载到任何地址，而不需要修改代码本身。 
>
> 在生成共享库时，链接器要求所有的目标文件都必须是位置无关的代码，这是因为共享库可以被加载到任何进程的地址空间中，而不同的进程的地址空间可能是不同的。如果目标文件不是位置无关的代码，那么在加载共享库时就需要进行重定位，这会增加运行时的开销。 因此，为了避免这种开销，链接器要求所有的目标文件都必须是位置无关的代码，这样在加载共享库时就可以直接将代码映射到进程的地址空间中，而不需要进行重定位。