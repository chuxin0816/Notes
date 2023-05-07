# CMake

[TOC]



## makefile

```makefile
CXX = g++
TARGET = hello	#生成的可执行文件名
SRC = $(wildcard *.cpp)	#wildcard 通配符
OBJ = $(patsubst %.cpp, %.o, $(SRC))	#路径替换

CXXFLAGS = -c -Wall

$(TARGET): $(OBJ)
	$(CXX) -o $@ $^	#$@ 冒号前面的，$^ 冒号后面的

%.o: %.cpp
	$(CXX) $(CXXFLAGS) $< -o $@	#$< 冒号后面的第一个

.PHONY: clean	#屏蔽名为clean的文件
clean:
	rm -f *.o $(TARGET)	#清除

```

## CMakeLists.txt

### PROJECT

```cmake
project (xx CXX)#xx为项目名，支持语言为C++
```

### SET

```cmake
set(xx a.cpp b.cpp...)#变量xx代表了a.cpp,b.cpp...
```

### MESSAGE

```cmake
message([<mode>] "message to display" ...)
	<mode> 用于指定输出信息的模式(STATUS,WARNING,FATAL_ERROR),默认STATUS
```

### ADD_EXECUTABLE

```cmake
add_executable(MyExecutable main.cpp)#指定可执行文件的名称和源代码文件
```

### ADD_SUBDIRECTORY 

```cmake
add_subdirectory(source_dir [binary_dir])
#添加一个子目录source_dir，该目录包含另一个CMakeLists.txt文件，可以通过该文件构建一个独立的库或可执行文件
#binary_dir 指定生成的构建文件存放的路径，如果不指定则默认和 source_dir 相同
```

### INCLUDE_DIRECTORIES

```cmake
include_directories(dir) 
#指定头文件搜索路径
```

### LINK_DIRECTORIES

```cmake
link_directories(dir) 
#指定非标准动态库的搜索路径
```

### TARGET_LINK_LIBRARIES

```cmake
target_link_libraries(project lib)
#project为生成的可执行文件名，lib为动态/静态库名
```

### ADD_LIBRARY

```cmake
add_library(<name> [STATIC | SHARED | MODULE] source1 [source2 ...])
#<name> 是库文件的名称，可以包含路径，例如 subdir/libname
#STATIC、SHARED 和 MODULE 分别表示编译静态库、动态库和可加载模块，如果不指定，则默认为 STATIC
#source 为源文件
```

### SET_TARGET_PROPERTIES

> 设置一个目标（例如可执行文件或库）的属性，包括编译器标志、链接器标志、目标文件名、输出路径等

```cmake
set_target_properties(target1 target2 ... PROPERTIES prop1 value1 prop2 value2 ...)
target1、target2 等为目标名称，prop1、prop2 等为属性名称，value1、value2 等为属性值。
```

### INSTALL

```cmake
install (FILES file1 file2... DESTINATION dir)#安装文件到dir目录
install (PROGRAM xx DESTINATION dir)#安装脚本到dir目录
install (DIRECTORY xx/ DESTINATION dir)#安装目录到dir目录
```

### 语法规则

- ﻿变量使用＄{}方式取值，在if控制语句中直接用变量名
- CMAKE_ INSTALL_ PREFIX 默认为/usr/local/

## 命令

```shell
#生成构建文件
cmake /path/to/source -B /path/to/build
/path/to/source是源码目录，/path/to/build是构建目录
#编译项目
cmake --build /path/to/build
#安装项目
cmake --install /path/to/build
```

## 示例

```shell
.
├── CMakeLists.txt
├── include
│   └── Hello.hpp
├── main.cpp
└── src
    └── Hello.cpp
```

`hello.hpp`内容如下：

```cpp
#pragma once
#include <iostream>
void hello();
```

`hello.cpp`内容如下：

```cpp
#include "hello.hpp"
void hello() {
  std::cout << "Hello, World!" << std::endl;
}
```

主函数`main.cpp`如下：

```cpp
#include "hello.hpp"
int main() {
  hello();
  return 0;
}
```

`CMakeLists.txt`如下：

```cmake
cmake_minimum_required(VERSION 3.15)
project(hello)
add_executable(hello main.cpp src/hello.cpp)
include_directories(include)
set(EXECUTABLE_OUTPUT_PATH bin)#指定生成可执行文件到build/bin
```

## 外部构建

## 构建示例

```shell
.
├── build
├── CMakeLists.txt
├── COPYRIGHT
├── doc
│   └── hello.txt
├── README
├── runhello.sh
└── src
    ├── CMakeLists.txt
    └── main.cpp
```

1. 在项目文件夹`mkdir build`
2. `cd build`
3. `cmake ..`
4. `make`
5. `make install`  安装

## 静态库和动态库

> 静态库一般以.a或.lib结尾，动态库一般以.so或.dll或.dylib结尾 

### 构建示例

```shell
.
├── build
├── CMakeLists.txt
└── lib
    ├── CMakeLists.txt
    ├── hello.cpp
    └── hello.h
```

### 构建静态库或动态库

lib中CMakeLists.txt中的内容

```cmake
add_library(MyLibrary STATIC|SHARED mylibrary.cpp)
```

1. `cd build`
2. `cmake ..`
3. `make`

### 构建静态库和动态库

lib中CMakeLists.txt中的内容

```cmake
# 定义静态库
add_library(MyLibrary STATIC mylibrary.cpp)

# 定义动态库
add_library(MyLibrary_shared SHARED mylibrary.cpp)

# 设置静态库和动态库的输出名称为 MyLibrary
set_target_properties(MyLibrary PROPERTIES OUTPUT_NAME MyLibrary)
set_target_properties(MyLibrary_shared PROPERTIES OUTPUT_NAME MyLibrary)

#可选项
[
# 设置静态库和动态库的版本信息
set_target_properties(MyLibrary PROPERTIES VERSION 1.0.0)
set_target_properties(MyLibrary_shared PROPERTIES VERSION 1.0.0 SOVERSION 1)
VERSION 和 SOVERSION 属性，用于指定库的版本号和 API

# 设置静态库和动态库的输出路径
set_target_properties(MyLibrary PROPERTIES ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/lib")
set_target_properties(MyLibrary_shared PROPERTIES LIBRARY_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/lib")
]
```

### 安装共享库和头文件

lib中CMakeLists.txt中补充：

```cmake
# 安装静态库和动态库
install(TARGETS MyLibrary MyLibrary_shared 
		ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)
ARCHIVE DESTINATION 选项用于指定静态库文件的安装路径
LIBRARY DESTINATION 选项用于指定动态库文件的安装路径
# 安装头文件
install(FILES mylibrary.h DESTINATION include)
```

安装的时候，指定一下路径，放到系统下

1. `cd build`
2. `cmake -D CMAKE_INSTALL_PREFIX=/usr ..`
3. `make`
4. `make install `安装

### 使用外部共享库和头文件

**结构：**

```shell
.
├── build
├── CMakeLists.txt
└── src
    ├── CMakeLists.txt
    └── main.cpp
```

main.cpp

```cpp
#include <hello.h>

int main(){
	HelloFunc();
}
```

在CMakeLists.txt中

**加入头文件搜索路径：**

INCLUDE_DIRECTORIES(/usr/include/hello)

> 以下二选一

**添加非标准的共享库搜索路径**

LINK_DIRECTORIES(/home/myproject/libs)

**添加需要链接的共享库**

TARGET_LINK_LIBRARIES(hello libhello.so)

在CMakeLists.txt中插入链接共享库，主要要插在executable的后面

