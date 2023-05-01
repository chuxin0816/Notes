### 1.makefile

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

