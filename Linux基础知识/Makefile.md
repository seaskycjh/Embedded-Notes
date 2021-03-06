## Makefile

[TOC]

### 1 概述

#### 1.1 简介

Makefile 文件描述了整个工程的编译、连接等规则。其中包括：工程中的哪些源文件需要编译以及如何编译、需要创建那些库文件以及如何创建这些库文件、如何最后产生我们想要得可执行文件。

#### 1.2 基础知识

**编译**：把高级语言书写的代码转换为机器可识别的机器指令。一个高级语言的源文件可对应一个目标文件（Linux 中默认后缀为“.o”）。

**链接**：将多.o 文件，或者.o 文件和库文件链接成为可被操作系统执行的可执行程序。

**静态库**：又称为文档文件（Archive File）。它是多个.o 文件的集合。

**共享库**：也是多个.o 文件的集合，但是这些.o 文件时有编译器按照一种特殊的方式生成（共享库已经具备了可执行条件）。

### 2 GNU Make

make 通过比较对应文件（规则的目标和依赖，）的最后修改时间，来决定哪些文
件需要更新、那些文件不需要更新。

#### 2.1 Makefile规则介绍

简单Makefile描述规则：

```makefile
TARGET... : PREREQUISITES...
	COMMAND
	...
```

**target**：规则的目标。通常是最后需要生成的文件名或者为了实现这个目的而必需的中间过程文件名。可以是.o文件、可执行程序名或者一个make执行的动作的名称，如目标“clean”。

**prerequisites**：规则的依赖。生成规则目标所需要的文件名列表。

**command**：规则的命令行。一个规则可以有多个命令行，每一条命令占一行。**每一个命令行必须以[Tab]字符开始，[Tab]字符告诉 make 此行是一个命令行**。

#### 2.2 简单示例

```makefile
#sample Makefile
obj = main.o kbd.o command.o display.o \
	  insert.o search.o files.o utils.o
edit : $(obj)
	cc -o edit $(obj)
main.o : main.c defs.h
	cc -c main.c
kbd.o : kbd.c defs.h command.h
	cc -c kbd.c
command.o : command.c defs.h command.h
	cc -c command.c
display.o : display.c defs.h buffer.h
	cc -c display.c
insert.o : insert.c defs.h buffer.h
	cc -c insert.c
search.o : search.c defs.h buffer.h
	cc -c search.c
files.o : files.c defs.h buffer.h command.h
	cc -c files.c
utils.o : utils.c defs.h
	cc -c utils.c
clean :
	rm edit $(obj)
```

- 可以将一个较长行使用反斜线（\）来分解为多行。**反斜线之后不能有空格**。

- 目标“clean”不是一个文件，它仅仅代表执行一个动作的标识，用于清除当前目录中编译过程中产生的临时文件Makefile中把那些没有任何依赖只有执行动作的目标称为**伪目标**。

- 默认的情况下，make执行的是Makefile中的第一个规则，此规则的第一个目标称之为“最终目的”或者“终极目标”。

#### 2.3 指定变量

使用一个变量“objects”、“OBJECTS”、“objs”、“OBJS”、“obj”或者“OBJ”来作为所有的.o 文件的列表的替代。在定义了此变量后，在需要使用这些.o文件列表的地方使用“$(变量名)”来表示它。

#### 2.4 自动推导规则

**make的隐含规则**：在Makefile中我们只需要给出需要重建的目标文件名（一个.o文件），make会自动为这个.o文件寻找合适的依赖文件（对应的.c文件。对应是指：文件名除后缀外，其余都相同的两个文件），而且使用正确的命令来重建这个目标文件。此默认规则称为。

因此，**书写 Makefile 时可以省略掉描述.c 文件和.o 依赖关系的规则**，如上例中的所有`cc -c ...`语句。

### 3 Makefile

#### 3.1 Makefile的内容

在一个完整的 Makefile 中，包含了 5 个东西：

**显式规则**：它描述了在何种情况下如何更新一个或者多个被称为目标的文件（Makefile 的目标文件）。

**隐含规则**：它是make根据一类目标文件（典型的是根据文件名的后缀）而自动推导出来的规则。

**变量定义**：使用一个字符或字符串代表一段文本串，当定义了一个变量以后，Makefile后续在需要使用此文本串的地方，通过引用这个变量来实现对文本串的使用。

**指示符**：指示符指明在 make 程序读取 makefile 文件过程中所要执行的一个动作。

**注释**：Makefile 中“#”字符后的内容被作为是注释内容（和 shell 脚本一样）处理。

#### 3.2 makefile文件命名

默认的情况下，make 会在工作目录（执行 make 的目录）下按照文件名顺序寻找makefile文件读取并执行，查找的文件名顺序为：**GNUmakefile**、**makefile**、**Makefile**。 不推荐使用GNUmakefile。

当 makefile 文件的命名不是这三个任何一个时，需要通过make的 **-f** 或者 **--file** 选项来指定make读取的makefile文件，可指定多个文件。

#### 3.3 包含其它makefile文件

Makefile 中包含其它文件所需要使用的关键字是**include**。格式为：`include FILENAMES ...`。

**FILENAMES**：是 shell 所支持的文件名。include和文件名之间、多个文件之间使用空格或者[Tab]键隔开。

通常指示符include用在以下场合： 

- 有多个不同的程序，由不同目录下的几个独立的Makefile来描述其重建规则。

- 当根据源文件自动产生依赖文件时；我们可以将自动产生的依赖关系保存在另外一个文件中，主Makefile使用指示符“include”包含这些文件。

在 Makefile 中可使用“-include”来代替“include”，来忽略由于包含文件不存在或者无法创建时的错误提示（“-”的意思是告诉 make，忽略此操作的错误，继续执行）。

#### 3.4 解析makefile文件

GUN make 的执行过程分为两个阶段。 

**第一阶段**：读取所有的 makefile 文件（包括“MAKIFILES”变量指定的、指示符
“include”指定的、以及命令行选项“-f(--file)”指定的 makefile 文件），内建所有的
变量、明确规则和隐含规则，并建立所有目标和依赖之间的依赖关系结构链表。 

**第二阶段**：根据第一阶段已经建立的依赖关系结构链表决定哪些目标需要更新，
并使用对应的规则来重建这些目标。 

### 4 Makefile规则

#### 4.1 规则语法

```makefile
TARGETS : PREREQUISITES
	COMMAND
	...
#或者
TARGETS : PREREQUISITES ; COMMAND
	COMMAND
	... 
```

**注意**：

- Makefile 中符号“$”有特殊的含义（表示变量或者函数的引用），在规则中需要使用符号“$”的地方，需要书写两个连续的（“$$”）。

- 目标文件的内容由依赖文件文件决定，依赖文件的任何一处改动，将导致已经存在的目标文件的内容过期。

- Mekfile 中表示文件名时可使用通配符。可使用的通配符有： “*”、“?”和“[…]”。

#### 4.2 目录搜寻

**一般搜索**：GNU make 可以识别一个特殊变量“VPATH”。通过变量“VPATH”可以指定依赖文件的搜索路径，当规则的依赖文件在当前目录不存在时，make 会在此变量所指定的目录下去寻找这些依赖文件。

定义变量“VPATH”时，使用空格或者冒号（:）将多个需要搜索的目录分开。

**选择性搜索**：“vpath”关键字（全小写的）是一个 make 的关键字，。它可以为不同类型的文件（由文件名区分）指定不同的搜索目录。它的使用方法有三种：

`vpath PATTERN DIRECTORIES`：为所有符合模式“PATTERN”的文件指定搜索目录“DIRECTORIES”。

PATTERN需要包含模式字符%，意思是匹配一个或者多个字符。

隐含规则、静态库和共享库也可以通过搜索目录得到。

#### 4.3 Makefile伪目标

伪目标不代表一个真正的文件名，在执行 make 时可以指定这个目标来执行其所在规则定义的命令，有时也可以将一个伪目标称为标签。

声明一个伪目标的方法：`.PHONY : clean`。

当一个伪目标没有作为任何目标（此目标是一个可被创建或者已存在的文件）的依赖时，我们只能通
过make的命令行来明确指定它为make的终极目标，来执行它所在规则所定义的命令。例如“make clean”。

#### 4.4 Makefile的特殊目标

**.PHONY**： 目标“.PHONY”的所有的依赖被作为伪目标。伪目标时这样一个目标：当使用make命令行指定此目标时，这个目标所在规则定义的命令、无论目标文件是否存在都会被无条件执行。

**.SUFFIXES**：特殊目标“SUFFIXES”的所有依赖指出了一系列在后缀规则中需要检查的后缀名。

**.DEFAULT**：Makefile 中，目标“.DEFAULT”所在规则定义的命令，被用在重建那些没有具体规则的目标（明确规则和隐含规则）。

**.PRECIOUS**：目标“.PRECIOUS”的所有依赖文件在make过程中会被特殊处理：当命令在执行过程中被中断时，make不会删除它们。

**.IGNORE**：如果给目标“.IGNORE”指定依赖文件，则忽略创建这个文件所执行命令的错误。

### 5 Makefile的变量

在 Makefile 中，变量是一个名字（像是 C 语言中的宏），代表一个文本字符串（变量的值）。

#### 5.1 变量的引用

变量的引用方式是：$ (VARIABLE_NAME)或者${ VARIABLE_NAME }”。在对一些简单变量（单字符）的引用，可以省略“（）”和“{}”。

#### 5.2 变量定义（赋值）

**递归展开式变量**：通过“=”或者使用指示符“define”定义的。

**直接展开式变量**：使用“:=”定义。变量被定义后就是一个实际需要的文本串，其中不再包含任何变量的引用。

**?=操作符**：条件赋值，只有此变量在之前没有赋值的情况下才会对这个变量进行赋值。

#### 5.3 变量的高级用法

**变量的替换引用**：对于一个已经定义的变量，可以使用“替换引用”将其值中的后缀字符（串）使用指定的字符（字符串）替换。格式为“$(VAR:A=B)”（或“${VAR:A=B}”），即替换变量“VAR”中所有“A”字符结尾的字为“B”结尾的字。

#### 5.4 变量取值

在运行make时通过命令行选项来取代一个已定义的变量值。

在makefile文件中通过赋值的方式或者使用“define”来为一个变量赋值

将变量设置为系统环境变量。

自动化变量，在不同的规则中自动化变量会被赋予不同的值。

一些变量具有固定的值。

#### 5.5 设置变量

Makefile 中变量的设置（也可以称之为定义）是通过“=”（递归方式）或者“:=”（静态方式）来实现的。左边的是变量名，右边是变量的值。

#### 5.6 追加变量值

一个通用变量在定义之后的其他一个地方，可以对其值进行追加。在 Makefile 中使用“+=”（追加方式）来实现对一个变量值的追加操作。

### 6 Makefile的条件执行

#### 6.1 基本语法

条件判断的基本语法：

```makefile
CONDITIONAL-DIRECTIVE
TEXT-IF-TRUE
endif
```

**ifeq**：此关键字用来判断参数是否相等，`ifegq (ARG1, ARG2)`。









