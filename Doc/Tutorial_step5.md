## CMake Tutortial Step5: Installing and Testing

对于一个项目来说，只编译一个可执行程序是不够的，还需要能够被安装，这里就是把当前项目的可执行程序、头文件、动态库等文件复制到对应的目录中使之能够继续执行。

#### 1. 安装程序

* set(): 用于设置变量的值。

```cmake
# MathFunctions/CMakeLists.txt
set(installable_libs MathFunctions tutorial_compiler_flags)
```

​	定义变量 `installable_libs`，值为 `MathFunctions` 和 `tutorial_compiler_flags`。在后续的cmake脚本中，可以使用`${installable_libs}`来引用该变量的值。

* install(): 用于安装构建好的文件到指定目录中。

```cmake
# MathFunctions/CMakeLists.txt
install(TARGETS ${installable_libs} DESTINATION lib)
```

​	将`MathFunctions` 和 `tutorial_compiler_flags`安装到 `lib` 目录下，这句CMake代码中的`lib`路径是相对于安装目标文件夹的路径，具体路径取决于在CMakeLists.txt文件中设置的`CMAKE_INSTALL_PREFIX`变量的值。默认情况下，`CMAKE_INSTALL_PREFIX`的值为`/usr/local`，因此在这种情况下，`MathFunctions`和 `tutorial_compiler_flags`文件将被安装到`/usr/local/lib`目录中。

```cmake
# MathFunctions/CMakeLists.txt
install(FILES MathFunctions.h DESTINATION include)
```

​	将头文件`MathFunctions.h`安装到 `include`目录中。`install()` 指令中的 `TARGETS` 和 `FILES` 参数用于指定要安装的目标类型。`TARGETS` 用于安装编译生成的目标文件，例如可执行文件、库文件等，而 `FILES` 用于安装其他类型的文件，例如头文件、配置文件、脚本文件等。需要注意的是，`TARGETS` 参数指定的目标必须是通过 `add_executable()`、`add_library()` 等命令定义的 CMake 目标，而不能是任意的文件。而 `FILES` 参数则可以指定任意类型的文件。

```cmake
# CMakeLists.txt
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
  DESTINATION include
  )
```

​	将可执行文件`Tutorial`安装到 `bin` 目录下，在 CMake 中，`PROJECT_BINARY_DIR` 是一个变量，它表示当前项目的构建目录，这里将自动生成的位于`build`目录下的`TutorialConfig.h`安装到`include`目录下。

* 执行安装程序

```cmake
# 在项目构建目录下生成构建文件
cmake ../Step5
# 开始构建项目
cmake --build .
# 执行安装程序，使用默认的安装目录
cmake --install .
# 修改变量CMAKE_INSTALL_PREFIX，使用自定义的安装目录
cmake --install . --prefix "/home/myuser/installdir"
```



#### 2. 自动化测试

这里利用CTest测试框架为项目添加单元测试。

* 使能单元测试

```cmake
# CMakeLists.txt
enable_testing()
```

* add_test(): 为项目添加测试项。

```cmake
# CMakeLists.txt
add_test(testname Exename arg1 arg2 ...)
```

​	其中，`testname`是测试的名称，`Exename`是要运行的测试程序的名称，`arg1`、`arg2`等是传递给测试程序的参数。 `add_test()`命令会在构建目标时自动运行测试程序，并将测试结果输出到控制台。如果测试程序返回非零退出代码，则测试失败。在使用`add_test()`命令之前，需要使用`enable_testing()`命令启用测试功能。

```cmake
add_test(NAME Runs COMMAND Tutorial 25)
```

​	添加测试项`Runs`，执行命令`Tutorial 25`。

```cmake
# CMakeLists.txt
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage
  PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
  )
```

​	添加测试项`Usage`，测试不输入参数的case，同时通过程序输出检查程序运行是否正常。

​	在测试程序执行后，CMake 会检查测试程序的输出是否与预期的输出匹配。这个检查过程可以使用 `PASS_REGULAR_EXPRESSION` 参数来指定一个正则表达式，用于匹配测试程序的输出。

```cmake
# CMakeLists.txt
add_test(NAME StandardUse COMMAND Tutorial 4)
set_tests_properties(StandardUse
  PROPERTIES PASS_REGULAR_EXPRESSION "4 is 2"
  )
```

​	添加测试case计算 4 的平方根，同时检查输出是否正常。

* function(): 用于定义一个函数。

```cmake
 function(<function_name> [arg1 [arg2 [arg3 ...]]])  
 # function body 
 endfunction() 
```

​	`function`函数可以接受参数，并且可以在函数体内执行一系列命令。函数可以在CMakeLists.txt文件中定义，然后在同一个文件中的其他地方调用。其中，`<function_name>`是函数的名称，`arg1`, `arg2`, `arg3`等是函数的参数。函数体内可以包含任意数量的CMake命令。

```cmake
# CMakeLists.txt
function(do_test target arg result)
  add_test(NAME Comp${arg} COMMAND ${target} ${arg})
  set_tests_properties(Comp${arg}
    PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()

# do a bunch of result based tests
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")
```

​	通过定义函数，快速添加测试case并添加正则表达式验证。

* 项目构建完成后，执行单元测试

```cmake
# ctest -N 命令用于列出所有可用的测试用例，而不运行它们。
ctest -N
# 运行所有测试用例并输出测试结果。
ctest
```



