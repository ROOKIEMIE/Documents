# GDB笔记

## 安装

对于Debian系统，可以使用apt一件安装

## 编译配置

对于需要使用gdb进行调试的程序，需要在编译时增加编译参数，以下使用cmake文件的配置为例

```cmake
cmake_minimum_required(VERSION 3.10)

project(Test)

# -DUSE_DEBUG=ON
option(USE_DEBUG "Enable debug mode" OFF)

# ...

add_executable(xxx xxx_test.c ${SOURCE})

if(USE_DEBUG)
	# 开启调试信息，该配置会为可执行程序在编译时增加-g参数
    set(CMAKE_BUILD_TYPE Debug)
    # 启用完整的调试信息 -g3 保留宏和源文件中的所有符号信息 -gdwarf-2 或 -gdwarf-4
    ## set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} -g3 -gdwarf-4")
    # 添加宏定义 DEBUG，这个宏可以在代码种使用，以便输出一些调试信息
    target_compile_definitions(xxx PRIVATE DEBUG)
endif()

# ...
```

并不是只有增加了``` -g ```参数的可执行程序才能被gdb调试，而是增加了``` -g ```参数就可以通过gdb获得更多调试信息，方便调试

## 运行

### 执行

若可执行程序没有主函数传参，可以使用gdb中的``` run ```命令直接运行

```shell
(gdb) run
```

如存在主函数传参，则需要使用``` set args [param1] [param2] ... ```的方式进行传参

```shell
(gdb) set args param1 param2 ...
```

再进行启动

```shell
(gdb) run
```

### 运行控制

#### 执行到断点/结束

```shell
(gdb) continue
```

#### 步入（进入函数)

```shell
(gdb) step
```

#### 单步执行（不跳入函数）

```shell
(gdb) next
```

#### 查看当前上下文（代码）

```shell
(gdb) list
```

## 添加断点/条件断点

使用``` break ```命令在gdb控制台中为可执行程序添加断点

### 为函数添加断点

```shell
(gdb) break <func name>
# (gdb) break main
```

### 在特定行添加断点

```shell
(gdb) break <c file name>.c:<line count>
# (gdb) break xxx.c:10
```

### 条件断点

```shell
(gdb) break <c file name>.c:<line count> if <var name> == <var value>
# (gdb) break xxx.c:10 if i == 5
```

## 查看变量

### 查看函数中的临时变量

```shell
info locals
```

### 查看函数的传入参数

```shell
info args
```

### 打印变量的值

```shell
(gdb) print myVar
```

### 打印复杂数据结构

```shell
(gdb) print *someStruct
```

### 修改变量的值

```shell
(gdb) set var myVar=4
```

## 查看线程

### 列出所有线程

``` shell
(gdb) info threads
```

### 切换到特定线程

```shell
(gdb) thread <thread ID>
```

## 查看内存

### 查看当前函数调用栈

```shell
(gdb) bt
```

### 查看特定帧

```shell
(gdb) frame 1
```

### 查看内存内容

```shell
(gdb) x/16xb 0xADDRESS
```

这里`x`表示“检查内存”，`16`表示要显示16个单元，`x`表示按十六进制显示，`b`表示每个单元是一个字节。`0xADDRESS`需要替换为实际的内存地址。

## 退出

```shell
(gdb) quit
```

