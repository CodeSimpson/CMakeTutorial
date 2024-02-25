## CMake Tutorial Step6: Adding Support for a Testing Bashboard

利用CDash为项目添加测试面板

> ​	CDash是一个开源的、基于Web的持续集成系统，用于跟踪软件开发项目的构建、测试和分析结果。它可以与各种不同的构建系统和测试框架集成，例如CMake、CTest、Google Test等。CDash提供了一个可视化的仪表板，用于展示构建和测试结果，以及趋势分析和错误报告。

#### 1. 使能 CDash

* include(): 用于加载CMake的一个模块。

​	这里删除使能 CTest 的命令`enable_testing()`，在项目根目录下通过`include`命令包含 CTest 模块，这将同时使能 CTest 和提交测试面板到 CDash 中。

```cmake
# CMakeLists.txt
# enable_testing()
include(CTest)
```

* 利用公共的CDash服务器查看测试面板

​	这里需要添加一个配置文件`CTestCinfig.cmake`，当CTest执行的时候会自动读取这个配置文件。

```cmake
# 项目名
set(CTEST_PROJECT_NAME "CMakeTutorial")
# 项目开始运行时间
set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")

# 项目生成的面板文件发送的url地址
set(CTEST_DROP_METHOD "http")
set(CTEST_DROP_SITE "my.cdash.org")
set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
set(CTEST_DROP_SITE_CDASH TRUE)
```

值得注意的是：

​	CTestConfig.cmake 文件是用于配置 CTest 测试工具的文件，其中包含了CDash 服务器的地址和其他相关信息，以便 CTest 将测试结果发送到 CDash服务器进行分析和可视化。在实际应用中，这个文件应该从 CDash 项目的设置页面下载，以便用于托管测试结果的 CDash 实例。一旦从 CDash 下载了该文件，就不应在本地进行修改。

#### 2. 执行 CDash

新建构建目录build，首先构建当前项目但不执行编译，然后通过下面的ctest命令，这将自动编译并运行测试case，同时提交测试结果到CDash服务器进行分析可视化。

```cmake
# CMakeLists.txt
cmake ../Step6
ctest [-VV] -D Experimental
```

> CDash服务器地址：https://my.cdash.org/index.php?project=CMakeTutorial



`ctest [-VV] -D Experimental`:

这是一个 CMake 的测试命令，具体含义如下： 

- `ctest` 是 CMake 自带的测试工具，用于运行测试程序并输出测试结果。 
-  `[-VV]` 是一个可选参数，表示输出详细的测试结果信息。其中 `-V` 表示输出测试的详细信息，`-VV` 表示输出更加详细的测试信息。 -
-  `-D Experimental` 是一个 CMake 变量，表示运行实验性的测试。这个变量可以用来控制测试的行为，例如指定测试的超时时间、测试的并行度等等。在这个命令中，`-D Experimental` 表示运行实验性的测试。