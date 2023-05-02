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

### INSTALL

```cmake
install (FILES file1 file2... DESTINATION dir)#安装文件到dir目录
install (PROGRAM xx DESTINATION dir)#安装脚本到dir目录
install (DIRECTORY xx/ DESTINATION dir)#安装目录到dir目录
```

### 语法规则

- ﻿变量使用＄{}方式取值，在if控制语句中直接用变量名
- CMAKE_ INSTALL_ PREFIX 默认为/usr/local/

## 外部构建

1. 在项目文件夹`mkdir build`
2. `cd build`
3. `cmake ..`
4. `make`
5. `make install`  安装

## 静态库构建

> 一般以.a或.lib结尾

## 动态库构建

> 一般以.so或.dll结尾 
