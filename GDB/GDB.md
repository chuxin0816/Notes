# GDB

[TOC]

## 生成可调式文件

```shell
g++ -g xx.cpp -o 程序名(生成的可执行/调试文件)
```

## 开始调试

```shell
gdb 程序名
```

## 调试命令

> 回车重复上一次命令

### 设置命令行参数

**set args xx xxx...**

> int main(int argc, char *argv[])
>
> argc表示命令行参数个数（默认为1，即程序本身）
>
> argv为命令行参数，argv[0]为程序本身，以此类推

### 运行

**r/run**

### 查看源码并显示行号

**l/list**

### 设置断点

**b/break** 行号/函数名

### 查看断点

**info b**

### 运行到下一个断点

**c/continue**

### 单步执行

**n/next**

### 打印信息

**p/print 变量名**

### 设置变量值

**set var 变量名=xx**

### 监测变量

**watch 变量名**

> 当变量值改变时进行提示

### 查看监测点

**info watchpoints** 

### 进入函数

**s/step**

### 退出

**q/quit**

## 小技巧

### gdb中使用终端命令

**shell 终端命令** 	ps：shell ls

### 日志功能

**set logging on** 

> 将命令输出保存到gdb.txt

### 调试正在运行的文件

**./xxx &** 	后台运行（自动输出进程号）

**gdb -p 进程号**

### 调试core文件

> 运行异常会生成core文件
>
> 如未生成用ulimit -a查看系统参数
>
> 用ulimit -c unlimit解除core file size限制

**gdb 程序名 core文件名**

### 调试多进程文件

**调试父进程**：**set follow-fork-mode parent**（默认）

**调试子进程**：**set follow-fork-mode child**

**设置调试模式**：**set detach-on-fork on/off**（默认是on）

> on表示调试当前进程的时候，其它的进程继续运行
>
> off表示调试当前进程的时候，其它的进程被挂起

**查看调试的进程**：**info inferiors**

**切换当前调试的进程**：**inferior进程id**（id为info inferiors的Num）

