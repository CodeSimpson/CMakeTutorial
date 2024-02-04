## CMake Tutorial

这里介绍一种新的将`MathFunctions`库的根目录添加到头文件搜索路径中的方法。

在Step2中，我们在project根目录下通过top level的CMakeLists.txt文件添加`MathFunctions`库的根目录到头文件搜索路径中，相关命令如下：

```cmake
list(APPEND EXTRA_LIBS MathFunctions)
target_include_directories(Tutorial PUBLIC ${EXTRA_LIBS})
```

新方法：

* target_include_directories(): 将目录添加到**头文件搜索路径列表**中。

```cmake
target_include_directories(MathFunctions INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

在这里，目标是MathFunctions库，`INTERFACE`关键字表示这个头文件搜索路径是用于这个库的接口，即这个库的用户也需要使用这个头文件搜索路径。`${CMAKE_CURRENT_SOURCE_DIR}`表示当前CMakeLists.txt文件所在的目录，也就是MathFunctions库的根目录。因此，这行代码的作用是将MathFunctions库的根目录添加到头文件搜索路径中，以便在使用MathFunctions库的时候可以方便地包含其头文件。

> 头文件搜索路径和库对应，在这里指MathFunctions库的根目录

* `INTERFACE`和`PUBLIC`的区别

`INTERFACE`和`PUBLIC`都是CMake中用于设置库的头文件搜索路径的关键字，它们的区别在于： - `INTERFACE`关键字表示这个头文件搜索路径是用于这个库的接口，即这个库的用户也需要使用这个头文件搜索路径。 - `PUBLIC`关键字表示这个头文件搜索路径是用于这个库的接口和依赖库的接口，即其他依赖于这个库的库也可以使用这个头文件搜索路径。 因此，如果一个库的头文件搜索路径只需要用于这个库的接口，那么可以使用`INTERFACE`关键字；如果这个头文件搜索路径还需要用于其他依赖于这个库的库的接口，那么应该使用`PUBLIC`关键字。

这里我们修改子目录`MathFunctions`中的CMakeLists.txt文件，top level的CMakeLists.txt文件中的相关修改可以直接删除，`tutorial.cxx`中还是可以直接引用头文件`TutorialConfig.h`。



需要注意的是，虽然top level的CMakeLists.txt中的`MathFunctions`库的根目录无需再设置添加到头文件搜索路径中，但`MathFunctions`还是需要link到目标文件中，换句话说就是下面命令需要保留。

```cmake
list(APPEND EXTRA_LIBS MathFunctions)
target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})
```