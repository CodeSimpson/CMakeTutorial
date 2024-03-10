## CMake Tutorial Step11: Adding Export Configuration

==不是很理解本节，需要进一步实验加深理解==

在前面的步骤中，我们介绍了通过CMake安装和测试，同时介绍了如何打一个安装包将项目发布给其他人使用，当前步骤中我们将添加必要的信息，以便其他使用 CMake 的项目可以使用我们的项目。换句话说就是将我们的项目配置成可重用的，以便其他项目可以方便地使用它。这通常包括将头文件和库文件安装到标准位置，以及提供 CMake 配置文件和导出目标等。

> 在这里是将`MathFunctions`库设置为可重用，使其他项目可以使用。

* 从项目构建树中导出目标的cmake文件

> export()命令的作用：Export targets from the build tree for use by outside projects.

```cmake
# MathFunctions/CMakeLists.txt
set(installable_libs MathFunctions tutorial_compiler_flags)
if(TARGET SqrtLibrary)
  list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs}
        EXPORT MathFunctionsTargets
        DESTINATION lib)
# install include headers
install(FILES MathFunctions.h DESTINATION include)
```

​	修改install()代码，在除了将`installable_libs`中的库安装到`lib`目录下之外，添加`EXPORT`关键字。

```cmake
install(TARGETS ${installable_libs}
        EXPORT MathFunctionsTargets
        DESTINATION lib)
```

​	这句代码只会将`${installable_libs}` 中列出的库安装到 `${CMAKE_INSTALL_PREFIX}/lib` 目录下，同时创建并导出导出`installable_libs`对一个的cmake文件 `MathFunctionsTargets.cmake`，但是它不会指定导出文件的路径。如果没有指定导出文件的路径，那么默认情况下，导出文件会被创建在 `${CMAKE_CURRENT_BINARY_DIR}` 目录下。

​	`${CMAKE_CURRENT_BINARY_DIR}` 是 CMake 在生成构建系统时自动创建的一个变量，它表示当前的构建目录。因此，如果没有指定导出文件的路径，那么 `MathFunctionsTargets.cmake` 文件会被创建在当前的构建目录中。 如果要将导出文件安装到指定的路径，需要使用 `install(EXPORT ...)` 命令，并指定导出文件的路径。

* 修改导出文件的安装路径

```cmake
# CMakeLists.txt
install(EXPORT MathFunctionsTargets
  FILE MathFunctionsTargets.cmake
  DESTINATION lib/cmake/MathFunctions
)
```

​	这样，`MathFunctionsTargets.cmake` 文件就会被安装到 `${CMAKE_INSTALL_PREFIX}/lib/cmake/MathFunctions` 目录下，`MathFunctionsTargets.cmake` 文件才能被其他项目使用 `find_package` 命令查找和链接 `MathFunctions` 库。

​	如果此时执行构建命令 `cmake ../Step11`，报错：

```shell
Target "MathFunctions" INTERFACE_INCLUDE_DIRECTORIES property contains
path:

  "/Users/robert/Documents/CMakeClass/Tutorial/Step11/MathFunctions"

which is prefixed in the source directory.
```

​	这里是因为Target `MathFunctions`的安装路径为固定路径，和当前机器强相关，需要update当前install路径。执行以下命令更新安装路径。

```cmake
# MathFunctions/CMakeLists.txt
target_include_directories(MathFunctions
                           INTERFACE
                            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                            $<INSTALL_INTERFACE:include>
                           )
```

​	这句代码的作用是将`MathFunctions`库的头文件目录添加到编译器的搜索路径中。其中，`$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>`表示在编译时添加当前源代码目录作为头文件搜索路径，`$<INSTALL_INTERFACE:include>`表示在安装时添加`include`目录作为头文件搜索路径。`MathFunctions`的安装路径取决于CMakeLists.txt文件中的`CMAKE_INSTALL_PREFIX`变量的值。如果该变量没有被设置，默认安装路径为`/usr/local`。在这种情况下，MathFunctions库的头文件将被安装到`/usr/local/include`目录下。

* 生成cmake配置文件`MathFunctionsConfig.cmake`

模板文件内容：

```cmake
# Config.cmake.in
@PACKAGE_INIT@

include ( "${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake" )
```

​	根据模板文件 `Config.cmake.in` 生成配置文件 `MathFunctionsConfig.cmake`，`MathFunctionsConfig.cmake` 文件的作用是为其他项目提供 `MathFunctions` 库的配置信息。当其他项目使用 `find_package(MathFunctions)` 命令时，CMake 会在系统中查找 `MathFunctionsConfig.cmake` 文件，并读取其中的配置信息。 

​	a. `@PACKAGE_INIT@` 是 CMake 自动生成的占位符，用于初始化 CMake 的内部变量和函数。

​	b. `include("${CMAKE_CURRENT_LIST_DIR}/MathFunctionsTargets.cmake")` 命令会**将 `MathFunctionsTargets.cmake` 文件包含进来，从而使其他项目能够链接 `MathFunctions` 库**。 `MathFunctionsTargets.cmake` 文件中定义了 `MathFunctions` 库的编译选项、链接选项、头文件路径、库文件路径等信息，`MathFunctionsConfig.cmake` 文件则将这些信息导出，以便其他项目使用。



引入cmake库`CMakePackageConfigHelpers`模块，类似于Step1中的`configure_file`接口自动生成`TotorialConfig.h`文件。

```cmake
# CMakeLists.txt
include(CMakePackageConfigHelpers)
# generate the config file that includes the exports
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake"
  INSTALL_DESTINATION "lib/cmake/example"
  NO_SET_AND_CHECK_MACRO
  NO_CHECK_REQUIRED_COMPONENTS_MACRO
  )
```

这句代码的作用是根据模板文件生成配置文件 `MathFunctionsConfig.cmake`，并将其安装到指定目录 `lib/cmake/example` 中。配置文件通常用于在 CMake 项目中导出库的信息，以便其他项目可以使用该库。在配置文件中，可以定义库的头文件路径、库文件路径、链接库等信息，以便其他项目可以使用该库。

>  CMakePackageConfigHelpers模块是CMake的一个模块，它提供了一些宏，用于生成用于配置安装包的CMake模块文件。这些模块文件包含了一些信息，例如头文件路径、库文件路径、链接器标志等，其他项目可以使用这些信息来查找和使用对应的库或可执行文件。 具体来说，CMakePackageConfigHelpers模块提供了以下两个宏： 
>
> ​	a. `configure_package_config_file`：用于生成一个配置文件，该文件包含相关库或可执行文件的安装路径、头文件路径、库文件路径等信息。
>
> ​	b. `write_basic_package_version_file`：用于生成一个版本文件，该文件包含相关的库或可执行文件的版本信息。
>
> 这些信息将被其他项目用于查找和使用库或可执行文件。 使用`CMakePackageConfigHelpers`模块可以使库或可执行文件更容易地被其他项目使用，因为其他项目可以使用CMake的`find_package`命令来查找和使用库或可执行文件。

* 生成版本相关的cmake文件

```cmake
# CMakeLists.txt
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake"
  VERSION "${Tutorial_VERSION_MAJOR}.${Tutorial_VERSION_MINOR}"
  COMPATIBILITY AnyNewerVersion
)
```

​	这句代码的作用是在构建过程中生成一个名为 `MathFunctionsConfigVersion.cmake` 的文件，其中包含了当前项目的版本号信息。具体来说，它使用 CMake 的 `write_basic_package_version_file` 命令来生成这个文件，该命令会自动将版本号信息写入到文件中。其中，`${CMAKE_CURRENT_BINARY_DIR}` 是一个 CMake 变量，表示当前构建目录的路径。`${Tutorial_VERSION_MAJOR}` 和 `${Tutorial_VERSION_MINOR}` 是项目的主版本号和次版本号，这些变量应该在项目的 CMakeLists.txt 文件中定义。`COMPATIBILITY AnyNewerVersion` 表示该文件与任何新版本兼容。这个文件通常会被其他 CMake 项目用来检查当前项目的版本信息，以便在构建时进行相应的处理。

* 确认cmake配置问题的安装路径

```cmake
# CMakeLists.txt
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsConfigVersion.cmake
  DESTINATION lib/cmake/MathFunctions
  )
```



* 不理解这里的作用

```cmake
export(EXPORT MathFunctionsTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/MathFunctionsTargets.cmake"
)
```

