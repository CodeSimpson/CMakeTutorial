## CMake Tutorial Step8: Ading a Custom Command and Generated File

​	假设我们不用`cmath`中自带的`log`和`exp`接口计算平方根，而是利用查表的方式。这里我们通过CMake新建文件头文件`Table.h`，并通过`MakeTable.cxx`生成可执行文件为`Table.h`填入`table`值。

* add_executable(): 生成可执行文件

```cmake
# MathFunctions/CMakeLists.txt
add_executable(MakeTable MakeTable.cxx)
```

​	在`MathFunctions/CMakeLists.txt`第一行写入以上代码，创建一个名为 `MakeTable` 的可执行文件，并将 `MakeTable.cxx` 文件作为源代码文件进行编译，可执行文件位于 `CMAKE_CURRENT_BINARY_DIR`中。

* add_custom_command(): 自定义构建规则。

```cmake
# MathFunctions/CMakeLists.txt
add_custom_command(
  OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
  DEPENDS MakeTable
  )
```

​	该规则的作用是生成一个名为 `Table.h` 的文件，并将其放置在 `${CMAKE_CURRENT_BINARY_DIR}` 目录下。

具体来说，该规则的行为如下： 

​	a. `OUTPUT` 参数指定了生成的文件名和路径，即 `${CMAKE_CURRENT_BINARY_DIR}/Table.h`。 

​	b. `COMMAND` 参数指定了生成文件的命令，即 `MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h`。这意味着在生成 `Table.h` 文件时，将会执行`MakeTable` 可执行文件，并将生成的文件放置在 `${CMAKE_CURRENT_BINARY_DIR}/Table.h` 路径下。 

​	c. `DEPENDS` 参数指定了生成文件所依赖的文件或目标，即 `MakeTable`。这意味着在生成 `Table.h` 文件之前，需要先生成 `MakeTable` 可执行文件。 

​	总之，这段 CMake 代码的作用是定义了一个自定义构建规则，用于生成 `Table.h` 文件，并将其放置在 `${CMAKE_CURRENT_BINARY_DIR}` 目录下。在生成 `Table.h` 文件之前，需要先生成 `MakeTable` 可执行文件。**值得注意的是，这里指只是自定义了一个构建规则，用于生成`Tabel.h`文件，但文件并不会实际生成。**

* add_library(): 定义一个库同时指定源文件。

```cmake
# MathFunctions/CMakeLists.txt
add_library(MathFunctions
            mysqrt.cxx
            ${CMAKE_CURRENT_BINARY_DIR}/Table.h
            )
```

​	这里将头文件`Table.h`编入静态库`MathFunctions`中。在CMakeLists.txt文件中，`${CMAKE_CURRENT_BINARY_DIR}`的路径是在编译时生成的，因此`Table.h`文件将在编译时生成，并将其路径添加到`MathFunctions`库的源文件列表中。

​	这里静态库`MathFunctions`实际并不需要编入头文件`Table.h`，只需要include生成的头文件即可。**但在实践中我发现，如果删除这里的`Table.h`，构建项目时就不会自动生成`Table.h`文件。**详细分析一下原因如下：

​	由于`${CMAKE_CURRENT_BINARY_DIR}/Table.h`是`add_library`命令的源文件之一，因此当执行`add_library`命令时，CMake会检查`${CMAKE_CURRENT_BINARY_DIR}/Table.h`文件是否存在。如果`${CMAKE_CURRENT_BINARY_DIR}/Table.h`文件不存在，则CMake会尝试执行`add_custom_command`命令来生成`${CMAKE_CURRENT_BINARY_DIR}/Table.h`文件。如果`${CMAKE_CURRENT_BINARY_DIR}/Table.h`文件存在，则CMake不会执行`add_custom_command`命令。

​	因此，如果删除`add_library`命令中的`${CMAKE_CURRENT_BINARY_DIR}/Table.h`，则CMake不会检查`${CMAKE_CURRENT_BINARY_DIR}/Table.h`文件是否存在，也就不会执行`add_custom_command`命令来生成`${CMAKE_CURRENT_BINARY_DIR}/Table.h`文件。

* target_include_directories(): 该命令用于为一个目标（target）指定头文件搜索路径。

```cmake
# MathFunctions/CMakeLists.txt
target_include_directories(MathFunctions
			INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
			PRIVATE   ${CMAKE_CURRENT_BINARY_DIR})
```

​	添加头文件`Table.h`所在的路径到库`MathFunctions`所对应的头文件搜索路径中。`PRIVATE`: 表示这个头文件搜索路径是私有的，也就是说，这个路径只会被添加到`MathFunctions`库的内部接口中，而不会被添加到其他目标的接口中。

* 修改 `MathFunctions/mysqrt.cxx`中的源代码，查表计算平方根。

```c++
// a hack square root calculation using simple operations
double mysqrt(double x)
{
  if (x <= 0) {
    return 0;
  }
  // use the table to help find an initial value
  double result = x;
  if (x >= 1 && x < 10) {
    std::cout << "Use the table to help find an initial value " << std::endl;
    result = sqrtTable[static_cast<int>(x)];
  } else {
    // do ten iterations
    for (int i = 0; i < 10; ++i) {
      if (result <= 0) {
        result = 0.1;
      }
      double delta = x - (result * result);
      result = result + 0.5 * delta / result;
      std::cout << "Computing sqrt of " << x << " to be " << result << std::endl;
    }
  }
  return result;
}
```

查表计算 1~9的平方根。



