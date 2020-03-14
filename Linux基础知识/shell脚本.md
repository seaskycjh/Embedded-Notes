## shell脚本

标签（空格分隔）： Linux基础

[TOC]

---

### 1 简介
shell是一个作为用户与Linux系统间接口的程序。若想切换到另一个shell，只需直接执行需要的shell程序就可以运行新的shell并改变命令提示符。
`$ /bin/bash --version  //查看bash的版本号`

### 2 管道和重定向
**重定向输出**：使用>操作符把标准输出重定向到一个文件，若该文件已存在，它的内容将被覆盖。用>>c操作符将输出内容附加到一个文件中。
利用>&操作符来结合两个输出，如
`$ ps >out.txt 2>&1   //将标准输出和标准错误定向到同一文件`
**重定向输入**：使用<操作符。
**管道**：使用管道操作符|来连接进程，管道连接的进程可同时运行，并且随数据流在它们间的传递自动进行协调。如查看系统中所有运行的进程，但不包含shell本身。
`ps -xo comm | sort | uniq | grep -v sh | more`

### 3 shell脚本程序
#### 3.1 交互式程序
直接在命令行上输入shell脚本可简单而快捷地测试短小代码段。当shell期待进一步输入时，正常地$提示符变为>提示符，可一直输下去，由shell判断何时输入完毕并执行脚本。
#### 3.2 创建脚本
以下是一个名为first的脚本：
```shell
#!/bin/sh

#first
#查找当前目录下所有保护POSIX字符串地文件

for file in *
do
  if grep -q POSIX $file
  then
    echo $file
  fi
done
exit 0
```
脚本中的注释以#符开始，#!是一种特殊地注释，用来告诉系统它的参数是执行脚本的程序。exit命令确保脚本返回一个有意义的退出码（在脚本中调用其他脚本并查看它是否执行成功需要退出码）。一般情况下，脚本不需要文件拓展名或后缀（可以使用.sh）。

#### 3.3 设置为可执行
运行脚本有两种方法：
(1)调用shell，把脚本文件名作为参数。`$ /bin/sh first`
(2)使用chmod命令改变文件模式使其可执行。`$ chmod +x first`
将脚本文件复制到bin目录，并防止其他用户修改脚本程序：

```
# cp first /usr/local/bin
# chown root /usr/local/bin/first
# chgrp root /usr/local/bin/first
# chmod 755 /usr/local/bin/first
```

### 4 shell语法
#### 4.1 变量
shell中使用变量前无需声明，通过使用它们来创建它们。通过在变量名前加 \$ 符号来访问它的内容，为变量赋值时只需使用变量名。
**引号**：若字符串包含空格，需用引号把它们括起来。此外等号两边不能有空格。
用echo输出变量值到终端，用read将输入赋值给变量。
**环境变量**：shell脚本中一些变量会根据环境设置中的值进行初始化。
​\$HOME：当前用户家目录
​\$PATH：用来搜索命令的目录列表
​\$0：shell脚本的名字
$#：传递给脚本的参数个数
$$：shell脚本的进程号
**参数变量**：
\$1，\$2，...脚本程序的参数 \$*：在一个变量中列出所有的参数 \$@：同上，以空格分隔变量

#### 4.2 条件
**test或[命令**：布尔判断命令。test 条件 等同于 [ 条件 ]（注意空格）。
条件类型：字符串比较、算术比较和与文件相关的条件测试。

| 比较                      | 结果                                          |
| ------------------------- | --------------------------------------------- |
| string1 = string2         | 两字符串相同结果为真                          |
| string1 != string2        |                                               |
| -n string                 | 字符串不为空                                  |
| -z string                 | 字符串为null                                  |
| -eq -ne -gt -ge -lt -le ! | = != > >= < <= !                              |
| -d -e -f -r -s -w -x      | 目录 存在 普通文件 可读 大小不为0 可写 可执行 |
**控制结构**：statements表示要执行的一系列命令。
(1)if、elif语句
```shell
if condition
then
  statements
elif condition; then
  statements
else
  statements
fi
```
(2)for语句
用for结构来循环处理一组值，这组值可以是任意字符串的集合。
```shell
for variable in values
do
  statements
done

/*打印当前目录中所有以字母f开头的脚本文件（都以.sh结尾）*/
#!/bin/sh
for file in $(ls f*.sh); do
  lpr $file
done
exit 0
```
(3)while语句：重复执行一个命令序列，但不知道该执行的次数。
```
while condition do
  statements
done
```
until语句与while循环相似，只是测试条件相反，即循环反复执行到条件为真。
(4)case语句
```shell
case variable in
  pattern [ | pattern] ...) statements;;
  pattern [ | pattern] ...) statements;;
  ...
esac

/*示例*/
case "$value" in
    yes | Y | Yes | YES )
        echo "..."
        echo "..."
        ;;
    [nN]*)
        echo "..."
        ;;
esac
```
(5)命令列表
AND列表：只有在前面所有命令都执行成功的情况下才执行后一条命令。
`statement1 && statement2 && statement3 && ...`
从左到右顺序执行，直到有一条命令返回false或列表中所有命令都执行完毕。&&的作用是检查前一条命令的返回值。
OR列表：持续执行一系列命令直到有一条命令成功为止，其后的命令将不再被执行。
`statement1 || statement2 || statement3 || ...`
从左到右顺序执行，直到有一条命令返回true或列表中所有命令都执行完毕。
可以将lia两种结构组合在一起：`condition && command for true || command for false`
(6)语句块
在某些只允许使用单语句的地方使用多条语句，将它们括在{}中构成一个语句块。
#### 4.3 函数
调用函数前要先定义，故把函数定义放在任何一个函数调用之前，可以通过return命令让函数返回数字，可以使用local关键字在函数中声明局部变量，此外，函数可以访问全局作用范围的其他变量。
```shell
/*定义一个shell函数*/
function_name () {
  statements
}
```
#### 4.4命令
shell内部可执行两类命令。一类是可以在命令行中执行的命令称为外部命令。一类是不能作为外部程序被调用的命令称为内部命令。
**break命令**：跳出循环。
**：命令**：空命令。
**continue命令**：使循环跳到下一次循环继续执行。
**.命令**：用于在当前shell中执行命令。**echo命令**：输出结尾带有换行符的字符串，去掉换行符用`echo -n ...`。**eval命令**：对参数求值。
**exit n命令**：使脚本以退出码n结束运行。
**export命令**：将参数变量导出到子shell中。**printf命令**：`printf "..."`，不支持浮点数。
**return命令**：使函数返回，若无数值参数，则返回最后一条命令的退出码。
**set命令**：为shell设置参数变量。
**shift命令**：把所有参数变量左移一个位置。
**trap命令**：用于指定在接收信号后将要采取的行动。`trap command signal`
**unset命令**：从环境中删除变量或函数。
两个有用的命令：
**find命令**：用于搜索文件。
**grep命令**：