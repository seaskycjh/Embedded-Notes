## GDB

[TOC]

------

### 1 gdb使用流程

```shell
#编译生成包含调试信息的可执行文件
$ gcc -g -o test test.c
#进入gdb，以(gdb)开头的命令行界面
$ gdb test
#查看载入的文件
(gdb) l
#设置断点，6为行号
(gdb) b 6
#查看断点情况
(gdb) info b
#查看调用函数（堆栈）的情况
(gdb) bt
#运行代码，默认从首行开始
(gdb) r
#查看变量值，i为变量名
(gdb) p i
#单步运行，有函数调用时，n不会进入该函数，而s会
(gdb) n
(gdb) s
#恢复程序运行
(gdb) c
```

### 2 gdb基本命令

gdb命令行中可以通过help查找命令。

#### 2.1 工作环境相关命令

gdb 中不仅可以调试所运行的程序，而且还可以对程序相关的工作环境进行相应的设定，甚至还可以使用 shell 中的命令进行相关的操作。

| 命令格式                     | 含义                 |
| ---------------------------- | -------------------- |
| set args 参数                | 指定运行时的参数     |
| show args                    | 查看设置好的运行参数 |
| Path dir                     | 设定程序的运行路径   |
| show paths                   | 查看程序的运行路径   |
| set environment var [=value] | 设置环境变量         |
| show environment [var]       | 查看环境变量         |
| cd dir                       | 进入dir目录          |
| Pwd                          | 显示当前工作目录     |
| shell command                | 运行shell的命令      |

