## CMake Tutorial Step9: Packging an Installer

在step5中，我们通过`install()`函数将当前项目包含的可执行文件、静态库、头文件等安装到对应的目录中，但这还不够，为了能使其他人也能使用当前项目，我们需要打包安装程序，同时发布源码。这里我们将使用`CPack`来创建基于特定平台的安装程序。

> CPack是CMake的一个组件，用于创建软件包。它可以将CMake项目打包成各种格式的软件包，例如RPM、DEB、NSIS、ZIP等。CPack可以自动检测项目的依赖关系，并将它们打包到软件包中。CPack还可以生成安装程序，使用户可以轻松地安装软件包。
>
> CPack的使用非常简单，只需要在CMakeLists.txt文件中添加以下代码即可：
>
> ```
> include(CPack)
> ```
>
> 然后在命令行中运行以下命令即可生成软件包：
>
> ```
> cmake .
> make
> cpack
> ```

* include(): 用于加载CMake的一个模块。

```cmake
include(InstallRequiredSystemLibraries)
# 用于设置项目的许可证文件路径，这个文件将会被打包到安装包中。
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/License.txt")
# 用于设置项目的主版本号和次版本号，这些信息将会被用于生成安装包的文件名。
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
# 用于设置生成源码包的格式，这里设置为 TGZ，表示生成 tar.gz 格式的源码包。
set(CPACK_SOURCE_GENERATOR "TGZ")
# 用于包含 CPack 模块，这个模块提供了打包和安装相关的命令和变量，可以用于生成各种类型的安装包和源码包。
include(CPack)
```

InstallRequiredSystemLibraries: 包含基于当前平台项目中的所需要的运行时库。需要注意的是，`InstallRequiredSystemLibraries` 命令只会安装项目所依赖的系统库，而不会安装其他的第三方库或者依赖项。

* 打包安装程序

```cmake
# 首先构建项目
cmake ../Step9
# 编译项目
cmake --build .
# 按默认打包
cpack
# 打包成ZIP格式，使用 Debug 模式。 
cpack -G ZIP -C Debug
# 打包源代码
cpack --config CPackSourceConfig.cmake
```

