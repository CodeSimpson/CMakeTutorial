## CMake Tutorial Step7: Adding System Introspection

添加系统自检，这里是在编译的时候检查系统是否支持`std::exp`和`std::log`接口运算。

* include(): 用于加载CMake的一个模块。

```cmake
# MathFuncions/CMakeLists.txt
include(CheckCXXSourceCompiles)
```

这里它会加载 CMake 模块 `CheckCXXSourceCompiles`，该模块提供了一个函数 `check_cxx_source_compiles()`，可以用于检查 C++ 代码是否可以编译通过。这个函数通常用于检查某些特性是否可用，例如某个 C++ 标准的支持情况。

* check_cxx_source_compiles(): CMake 中的一个函数，用于检查 C++ 源代码是否可以编译通过。它的作用是在 CMake 构建过程中编译一个 C++ 源文件，并检查编译是否成功。如果编译成功，则该函数返回 `TRUE`，否则返回 `FALSE`。

```cmake
# MathFuncions/CMakeLists.txt
check_cxx_source_compiles("
  #include <cmath>
  int main() {
    std::log(1.0);
    return 0;
  }
" HAVE_LOG)
check_cxx_source_compiles("
  #include <cmath>
  int main() {
    std::exp(1.0);
    return 0;
  }
" HAVE_EXP)
```

接下来我们需要将变量 `HAVE_LOG`和`HAVE_EXP`传入源文件中使用，在Step2中我们做过类似的操作，对变量`USE_MYMATH`，通过`TutorialConfig.h.in`文件获取`USE_MYMATH`的值，使这个宏的值可以在 C++ 代码中使用。这里介绍一个新方法：

* target_compile_definitions(): CMake 中的一个函数，用于为一个目标（例如库或可执行文件）添加编译器定义。编译器定义是一些预定义的宏，可以在编译时用于控制代码的行为。

```cmake
# MathFuncions/CMakeLists.txt
if(HAVE_LOG AND HAVE_EXP)
  target_compile_definitions(MathFunctions
                             PRIVATE "HAVE_LOG" "HAVE_EXP")
endif()
```

这里在`HAVE_LOG`和`HAVE_EXP`都定义都为true时，为c++代码添加宏定义`HAVE_LOG`和`HAVE_EXP`，**这里为c++代码添加的宏定义可以与check_cxx_source_compiles返回的变量结果名不相同**。

`target_compile_definitions` 函数的语法如下： 

```cmake
target_compile_definitions(target
                           PUBLIC | PRIVATE | INTERFACE
                           [item1 [item2 ...]])
```

其中，`target` 是要添加编译器定义的目标，`PUBLIC`、`PRIVATE` 和 `INTERFACE` 是可选的关键字，用于指定编译器定义的可见性。`item1`、`item2` 等参数是要添加的编译器定义，可以是一个或多个字符串。 

`PUBLIC`、`PRIVATE` 和 `INTERFACE` 关键字的作用如下： 

​	a.`PUBLIC`：编译器定义将应用于目标本身以及使用该目标的任何目标。 

​	b.`PRIVATE`：编译器定义将仅应用于目标本身。 

​	c.`INTERFACE`：编译器定义将仅应用于使用该目标的任何目标，而不会应用于目标本身。

* 添加cmath头文件

```c++
// MathFunctions/mysqrt.cxx
#include <cmath>
```

* 使用cmath中的接口计算平方根

```c++
// MathFunctions/mysqrt.cxx
#if defined(HAVE_LOG) && defined(HAVE_EXP)
  double result = std::exp(std::log(x) * 0.5);
  std::cout << "Computing sqrt of " << x << " to be " << result
            << " using log and exp" << std::endl;
#else
  double result = x;
```

