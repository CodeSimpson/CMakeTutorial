## CMake Tutortial

#### 1. 添加interface库

在Step1中我们通过一下命令指定当前project需要的c++标准：

```cmake
# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)
```

这里我们通过添加 interface library 的方式指明需要的c++标准。在CMake中，interface library是一种特殊类型的库，它不包含任何源代码或编译对象，而是仅包含头文件和链接库的信息。它的作用是定义一个接口，用于描述一个库或模块的公共接口，以便其他项目可以使用该接口而不需要了解其实现细节。

* add_library()：创建一个库文件，这里会生成一个interface库

```cmake
add_library(tutorial_compiler_flags INTERFACE)
```

`tutorial_compiler_flags`不会生成对应的 .a 文件或 .so 文件。相反，它只是一组头文件和编译选项，这些选项将传递给依赖它的其他目标。当一个目标链接到一个 interface 库时，它会自动继承该库的编译选项和头文件路径，但不会链接到该库的二进制文件。

* target_compile_features()：cmake中的一个函数，用于指定目标需要的c++特性。

```cmake
target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
```

它的作用是告诉CMake编译器需要支持哪些c++特性，以便在编译时启用相应的编译选项。这里是告诉CMake编译器需要支持c++11标准，并在编译时启用相应的编译选项，如果编译器不支持 c++11，CMake 将会报错。这里使用 `INTERFACE`表明依赖`tutorial_compiler_flags`的目标需要支持c++11，但`tutorial_compiler_flags`本身不需要。

> 在CMake中，`INTERFACE`、`PUBLIC`和`PRIVATE`是用于指定库的属性的关键字。它们的含义如下：
>
> - `INTERFACE`：表示该属性仅适用于依赖该库的目标，而不适用于该库本身。 
> - `PUBLIC`：表示该属性既适用于该库本身，也适用于依赖该库的目标。 
> - `PRIVATE`：表示该属性仅适用于该库本身，而不适用于依赖该库的目标。

* target_link_libraries(): 指定目标可执行文件或库所需要link的一个或多个库。

```cmake
target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS} tutorial_compiler_flags)
```

这里通过interface库`tutorial_compiler_flags`指明目标可执行文件Tutorial需要支持c++11标准。

在 CMake 中，`target_link_libraries` 命令用于将一个或多个库链接到目标中。`PUBLIC`、`INTERFACE` 和 `PRIVATE` 是用于指定链接库的可见性的关键字。 	

​	- `PUBLIC` 表示链接库既会被目标本身使用，也会被目标的用户使用，也就是说，链接库的头文件和链接库本身都会被暴露给目标的用户。 

​	-  `INTERFACE` 表示链接库只会被目标的用户使用，而不会被目标本身使用。也就是说，只有链接库的头文件会被暴露给目标的用户，链接库本身不会被暴露给目标。

​	- `PRIVATE` 表示链接库只会被目标本身使用，而不会被目标的用户使用。也就是说，链接库的头文件和链接库本身都不会被暴露给目标的用户。 

在这个例子中，使用 `PUBLIC` 的原因是，链接库 `${EXTRA_LIBS}` 和 `tutorial_compiler_flags` 不仅会被目标 `Tutorial` 使用，也会被 `Tutorial` 的用户使用。因此，需要使用 `PUBLIC` 来将这些库暴露给 `Tutorial` 的用户。

> 在这里，"目标的用户"指的是使用该目标库的其他程序或者库。当一个库被标记为 `PUBLIC` 时，它不仅会被目标本身使用，也会被使用该目标库的其他程序或者库使用。 举个例子，假设你有一个名为 `mylib` 的库，其中包含一个名为 `foo` 的函数。你的程序 `myprogram` 使用了 `mylib` 中的 `foo` 函数。如果你将 `mylib` 标记为 `PUBLIC`，那么其他程序或者库也可以使用 `mylib` 中的 `foo` 函数，只需要链接 `mylib` 即可。 如果你将 `mylib` 标记为 `PRIVATE`，那么其他程序或者库就不能使用 `mylib` 中的 `foo` 函数，因为 `foo` 函数不会被包含在 `myprogram` 的可执行文件中。





#### 2. 为编译器设置编译警告flag

添加编译警告flag在编译阶段而非安装阶段。

* 根据编译器的类型不同生成不同的编译选项。

```cmake
set(gcc_like_cxx "$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
set(msvc_cxx "$<COMPILE_LANG_AND_ID:CXX,MSVC>")
```

这两句命令是CMake中的Generator Expressions，用于根据编译器类型生成不同的编译选项。 

第一句命令定义了一个变量`gcc_like_cxx`，它的值是一个Generator Expression，用于匹配GCC系列编译器和Clang编译器。具体来说，`$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>`表示如果当前编译器是C++编译器（`CXX`），并且编译器类型是ARMClang、AppleClang、Clang、GNU或LCC中的任意一种，则返回true，否则返回false。这个变量可以在后续的CMake命令中使用，比如设置编译选项。

第二句命令定义了一个变量`msvc_cxx`，它的值也是一个Generator Expression，用于匹配MSVC编译器。具体来说，`$<COMPILE_LANG_AND_ID:CXX,MSVC>`表示如果当前编译器是C++编译器（`CXX`），并且编译器类型是MSVC，则返回true，否则返回false。这个变量也可以在后续的CMake命令中使用，比如设置编译选项。

> message("Compiler ID: ${CMAKE_CXX_COMPILER_ID}")
>
> 使用CMAKE_CXX_COMPILER_ID可以打印字符串 GUN。
>
> message("$<COMPILE_LANG_AND_ID:CXX,ARMClang,AppleClang,Clang,GNU,LCC>")
>
> 没有任何打印，不知道为什么。

* 通过新的生成器表达式为CMake目标设置编译选项。

```cmake
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>"
  "$<${msvc_cxx}:-W3>"
)
```

这个目标`tutorial_compiler_flags`是一个INTERFACE库，它没有源文件，只包含头文件和编译选项。这个目标的作用是将编译选项传递给其他目标，以便它们也能使用这些选项。

这里使用了CMake的生成器表达式，根据`${gcc_like_cxx}`和`${msvc_cxx}`这两个变量的值（表示当前使用的编译器是否是类似于GCC的编译器和MSVC编译器），当值为true时，将相应的编译选项添加到当前project 编译器的编译选项中。

* GCC编译器的编译选项：
  * `-Wall`：开启所有警告。
  * `-Wextra`：开启一些额外的警告。
  * `-Wshadow`：开启有关变量遮蔽的警告。
  * `-Wformat=2`：开启有关格式化字符串的警告。
  * `-Wunused`：开启有关未使用变量和函数的警告。
* MSVC编译器的编译选项：
  * "-W3"选项表示启用所有警告，并将警告视为错误。这个选项可以帮助开发者在编译时发现潜在的问题，提高代码质量。

* 限制编译器的警告flag只在编译阶段生效，安装阶段不生效。

```cmake
target_compile_options(tutorial_compiler_flags INTERFACE
  "$<${gcc_like_cxx}:$<BUILD_INTERFACE:-Wall;-Wextra;-Wshadow;-Wformat=2;-Wunused>>"
  "$<${msvc_cxx}:$<BUILD_INTERFACE:-W3>>"
)
```

使用`BUILD_INTERFACE`条件。在CMake中，`BUILD_INTERFACE`是一个生成器表达式的关键字，用于指定编译选项的使用范围。具体来说，BUILD_INTERFACE表示这些编译选项只在构建目标时使用，而**不会传递给依赖目标或安装目标**。 在这个命令中，BUILD_INTERFACE用于指定"-Wall"、"-Wextra"、"-Wshadow"、"-Wformat=2"和"-Wunused"这些编译选项的使用范围。由于这些选项是为了提高代码质量而添加的，因此它们只需要在构建目标时使用，而不需要传递给依赖目标或安装目标。 

需要注意的是，BUILD_INTERFACE只在生成器表达式中使用，而不是在普通的CMake命令中使用。它的作用是告诉CMake如何处理生成器表达式中的编译选项，以便正确地传递给编译器。 总之，BUILD_INTERFACE是一个生成器表达式的关键字，用于指定编译选项的使用范围。在这个命令中，它用于指定编译选项只在构建目标时使用。





使用生成器表达式（Generator Expressions），其主要分为3类：

* 逻辑表达式
* 信息表达式
* 输出表达式

