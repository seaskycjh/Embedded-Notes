# 文件I/O

[TOC]

---

### 1 概述

#### 1.1 系统调用与用户编程接口（API）

**系统调用**是指操作系统提供给用户程序调用的一组“特殊”接口，用户程序可以通过这组接口来获得操作系统内核提供的服务。

在 Linux 中，为了更好地保护内核空间，将程序的运行空间分为内核空间和用户空间（即内核态和用户态），它们分别运行在不同的级别上，在逻辑上是相互隔离的。

**用户编程接口**（API）遵循了在 UNIX 中流行的应用编程界面标准——POSIX 标准，这些编程接口主要是通过C库（libc）实现的。 

#### 1.2 文件描述符

所有对设备和文件的操作都是使用文件描述符来进行的。文件描述符是一个非负的整数，指向在内核中每个进程打开文件的记录表。当打开一个现有文件或创建一个新文件时，内核向进程返回一个文件描述符。通常一个进程启动时，会打开 3 个文件：标准输入、标准输出和标准出错处理，对应的文件描述符分别为0（STDIN_FILENO），1（STDOUT_FILENO），2（STDERR_FILENO）。
### 2 底层文件I/O操作
文件 I/O 操作的系统调用主要用到 5 个函数：open、read、write、lseek和 close。这些函数的特点是**不带缓冲**，直接对文件（包括设备）进行读写操作。

#### 2.1 open函数

打开或创建一个文件，在打开或创建文件时可以指定文件的属性及用户的权限等各种参数。 

```c
#include <fcntl.h>
int open(const char *pathname, int oflag, mode_t mode);
//成功返回文件描述符，出错返回-1
```
**pathname**：要打开或创建文件的名字。

**oflag**：说明此函数的多个选项，下列常量用"|"运算构成oflag参数。
下表中前三个常量中必须指定且只能指定一个，之后的常量是可选的。

| 常量       | 含义                             |
| ---------- | -------------------------------- |
| O_RDONLY   | 只读打开                         |
| O_WRONLY   | 只写打开                         |
| O_RDWR     | 读、写打开                       |
| O_APPEND   | 每次写时追加到文件尾             |
| O_CREAT    | 文件不存在则创建，需要参数mode   |
| O_EXCL     | 测试文件是否存在                 |
| O_TRUNC    | 文件存在且可写，将其长度截短为0  |
| O_NOCTTY   | 不将该设备分配作为进程的控制终端 |
| O_NONBLOCK | 为文件I/O操作设置非阻塞模式      |
**mode**：新建文件时设置的用户权限，可用一组宏定义：S_I(R/W/X)(USR/GRP/OTH)，例如S_IRUSR（用户读）。或者用八进制表示法，例如0400（用户读）。

#### 2.2 read、write和close函数

**read**函数用于将从指定的文件描述符中读出的数据放到缓存区中，并返回实际读入的字节数。若读到要求的字节数之前已到达文件的尾部，则返回的字节数会小于希望读出的字节数。

**write**函数用于向打开的文件写数据。

**close**函数用于关闭一个被打开的文件，当一个进程终止时，所有被它打开的文件都由内核自动关闭。

```c
#include <unistd.h>
ssize_t read(int fd, void *buf, size_t nbytes);
//成功返回读到的字节数，已到达文件尾返回0，出错返回-1
ssize_t write(int fd, const void *buf, size_t nbytes);
//成功返回已写字节数，出错返回-1
int close(int fd);
//成功返回0，出错返回-1
```
**fd**：文件描述符。

**buf**：指定存储器读出/写入数据的缓冲区。

**nbytes**：指定读出/写入的字节数。

#### 2.3.lseek函数

为一个打开的文件设置其偏移量，只能用在可定位（可随机访问） 文件操作中，管道、套接字和大部分字符设备文件是不可定位的。
```c
#include <unistd.h>
off_t lseek(int fd, off_t offset, int whence);
//成功返回新的文件偏移量，出错返回-1
```
**文件偏移量**：非负整数，用于度量从文件开始处计算的字节数，读、写都从当前文件偏移量开始并使偏移量增加所读写的字节数。

**offset**：与参数**whence**的值有关

- 若whence是SEEK_SET(0)，则将偏移量设置为距文件开始处offset个字节。
- 若whence是SEEK_CUR(1)，则将偏移量设置为其当前值加offset个字节，offset可正可负。
- 若whence是SEEK_END(2)，则将偏移量设置为文件长度加offset个字节，offset可正可负。

**注意**：lseek不引起I/O操作，只用于下一次读、写操作。文件偏移量可大于文件当前长度，此情况下下次写操作将加长该文件，并构成一个空洞。

### 3 文件锁

#### 3.1 概述

**文件锁**：在文件已经共享的情况下，Linux通常采用的方法是给文件上锁，来避免共享的资源产生竞争的状态。文件锁包括建议性锁和强制性锁。建议性锁要求每个上锁文件的进程都要检查是否有锁存在，并且尊重已有的锁。强制性锁是由内核执行的锁，当一个文件被上锁进行写入操作的时候，内核将阻止其他任何文件对其进行读写操作。

**lockf**：用于对文件施加建议性锁。

**fcntl**：不仅可以施加建议性锁，还可以施加强制锁。同时，fcntl还能对文件的某一记录上锁，即记录锁。 

**记录锁**：可分为读取锁和写入锁，其中读取锁又称为共享锁，它能够使多个进程都能在文件的同一部分建立读取锁。而写入锁又称为排斥锁，在任何时刻只能有一个进程在文件的某个部分上建立写入锁。在文件的同一部分不能同时建立读取锁和写入锁。 

**死锁**：若两个进程互相等待对方持有并且不释放（锁定）的资源时，则这两个进程处于死锁状态。若一个进程已经控制了文件中的一个加锁区域，然后又试图对另一个进程控制的区域加锁，则它会休眠，有发生死锁的可能性。

#### 3.2 fcntl函数

可以对已打开的文件描述符进行各种操作，包括管理文件锁、获得和设置文件描述符和文件描述符标志、文件描述符的复 制等功能。

```c
#include <fcntl.h>
struct flock {
    short l_type;     /*F_RDLCK, F_WRLCK, F_UNLCK*/
    short l_whence;   /*SEEK_SET, SEEK_CUR, SEEK_END*/
    off_t l_start;    /*偏移量*/
    off_t l_len;      /*区域的字节长度*/
    pid_t l_pid;      /*进程的ID*/
};
int fcntl(int fd, int cmd, .../*struct flock *flockptr*/);
//成功依赖于cmd，否则返回-1
```
**cmd**参数：

| cmd值    | 函数作用                                                     |
| -------- | ------------------------------------------------------------ |
| F_DUPFD  | 复制文件描述符                                               |
| F_GETFD  | 获得文件描述符标记                                           |
| F_SETFD  | 设置文件描述符标记                                           |
| F_GETFL  | 获得文件状态标记                                             |
| F_SETFL  | 设置文件状态标记                                             |
| F_GETOWN | 获得异步I/O所有权                                            |
| F_SETOWN | 设置异步I/O所有权                                            |
| F_GETLK  | 判断由flockptr所描述的锁是否会被另外一把锁所排斥（阻塞）。   |
| F_SETLK  | 设置由flockptr所描述的锁。                                   |
| F_SETLKW | 是F_SETLK的阻塞版本，在无法获取锁时，会进入睡眠状态；如果可以获取锁或者捕捉到信号则会返回 |

**flock结构**：
l_type：所希望的所类型：F_RDLCK（共享读锁）、F_WRLCK（独占性写锁）或F_UNLCK（解锁一个区域）。
l_start、l_whence：加锁或解锁区域的起始字节偏移量。
**注意**：（1）锁可以在文件尾端处或越过文件尾端处开始，但不能在文件起始位置之前开始。
（2）若l_len为0，则表示锁的范围可以扩展到最大可能偏移量，意味着不管向该文件追加了多少数据，都可以处于锁的范围。
（3）为了对整个文件加锁，可设置l_start=0，l_whence=SEEK_SET，l_len=0。

**文件记录锁的实现**：

首先给 flock 结构体的对应位 赋予相应的值。接着使用两次 fcntl函数，设置cmd 值为 F_GETLK 用于判断文件是否可以上锁，若可以进行，则 flock 结构的 l_type 会被 设置为 F_UNLCK，其他域不变；若不可行，则 l_pid 被设置为拥有文件锁的进程号，其他域不变。设置cmd 值为F_SETLK（或 F_SETLKW）给相关文件上锁。

#### 3.3  锁的隐含继承和释放

- 锁与进程和文件相关联。当一个进程终止时，它所建立的锁全部释放；当一个描述符关闭，通过描述符引用的文件上的锁全部释放。
- 由fork产生的子进程不继承父进程所设置的锁。
- 执行exec后，新程序可以继承原执行程序的锁。

### 4 高级I/O

#### 4.1 I/O处理模型

**阻塞I/O模型**：在这种模型下，若所调用的 I/O 函数没有完成相关的功能，则会使进程挂起，直 到相关数据到达才会返回。对管道设备、终端设备和网络设备进行读写时经常会出现这种情况。

**非阻塞模型**：在这种模型下，当请求的 I/O 操作不能完成时，则不让进程睡眠，而且立即返回。

**I/O多路转接模型**：在这种模型下，如果请求的 I/O 操作阻塞，且它不是真正阻塞 I/O，而是让其中 的一个函数等待，在这期间，I/O还能进行其他操作。

**信号驱动I/O模型**：在这种模型下，通过安装一个信号处理程序，系统可以自动捕获特定信号的到来， 从而启动I/O。

**异步I/O模型**：在这种模型下，当一个描述符已准备好，可以启动 I/O 时，进程会通知内核。

**select**和**poll**的 I/O 多路转接模型是处理 I/O 复用的一个高效的方法。从 select()和 poll()函数返回时，内核会通知用户 已准备好的文件描述符的数量、已准备好的条件等。通过使用 select()和 poll()函数的返回结果，就可以调 用相应的 I/O 处理函数。

#### 4.2 select函数

```c
#include <sys/types.h>
#include <sys/time.h>
#include <unistd.h>
/* 文件描述符集处理函数 */
FD_ZERO(fd_set *set)		   //清除一个文件描述符集
FD_SET(int fd, fd_set *set)	   //将一个文件描述符加入文件描述符集中
FD_CLR(int fd, fd_set *set)	   //将一个文件描述符从文件描述符集中清除
FD_ISSET(int fd, fd_set *set)  //若fd为fd_set集中的一个元素，则返回非零值

struct timeval
{
    long tv_sec; /* 秒 */
    long tv_unsec; /* 微秒 */
}

int select(int maxfdp1, fd_set *readfds, fd_set *writefds,
		   fd_set *exeptfds, struct timeval *timeout);
//成功返回准备好的文件描述符数目，超时返回0，出错返回-1
```

**maxfdp1**：该参数值为需要监视的文件描述符的大值加 1 。

**readfds**：由 select监视的读文件描述符集合 。

**writefds**：由 select监视的写文件描述符集合 。

**exeptfds**：由 select监视的异常处理文件描述符集合 。

**timeout **：为NULL则永远等待，直到捕捉到信号或文件描述符已准备好为止 ；为0则从不等待，测试所有指定的描述符并立即返回。

**使用方法**：在使用 select()函数之前，首先使用 FD_ZERO和 FD_SET来初始化文件描述符集，在使用了select函数时，可循环使用 FD_ISSET来测试描述符集，在执行完对相关文件描述符的操作之后，使用 FD_CLR来清除描述符集。

#### 4.3 poll函数

```c
#include <sys/types.h>
#include <poll.h>
struct pollfd
{
    int fd;       	 /* 需要监听的文件描述符 */
    short events;    /* 需要监听的事件 */
    short revents;   /* 已发生的事件 */
}

int poll(struct pollfd *fds, int numfds, int timeout);
//成功返回大于0的值，表示事件发生的pollfd结构的个数，超时返回0，出错返回-1 
```

| events值 | 含义                                                      |
| -------- | --------------------------------------------------------- |
| POLLIN   | 文件中有数据可读                                          |
| POLLPRI  | 文件中有紧急数据可读                                      |
| POLLOUT  | 可以向文件写入数据                                        |
| POLLERR  | 文件中出现错误，只限于输出                                |
| POLLHUP  | 与文件的连接被断开了，只限于输出                          |
| POLLNVAL | 文件描述符是不合法的，即它并没有指向一个成功打开的文件 。 |

**fds**：pollfd结构数组的指针，用于描述需要对哪些文件的哪种类型的操作进行监控。

**numfds**：需要监听的文件个数，即第一个参数所指向的数组中的元素数目。

**timeout**：表示 poll 阻塞的超时时间（毫秒）。如果该值小于等于 0，则表示无限等待。

### 5 标准I/O

#### 5.1 概述

标准 I/O 操作都是基于流缓冲的，标准 I/O 提供流缓冲的目的是尽可能减少使用 read和 write等系统调用的数量，从而提高程序的效 率。标准 I/O 提供了 3 种类型 的缓冲存储。标准输入、标准输出和标准出错这三个标准I/O流通过预定义文件指针stdin、stdout、stderr引用。

**全缓冲**：在填满标准I/O缓冲区后才进行实际I/O操作，磁盘文件通常 是由标准 I/O 库实施全缓冲的。

**行缓冲**：当输入和输出中遇到换行符时标准I/O库执行I/O操作。当流为终端时通常使用行缓冲。

**不带缓冲**：标准I/O库不对字符进行缓冲存储。标准出错通常时不带缓冲的。

#### 5.2 打开文件

打开文件有三个标准函数，**fopen**可以指定打开文件的路径和模式，**fdopen**可以指定打开的文件描述符和模式， **freopen**除可指定打开的文件、模式外，还可指定特定的 I/O 流。它们都返回一个指向FILE的指针，该指针指向对应的 I/O 流。

```c
#include <stdio.h>
FILE *fopen(const char *restrict pathname, const char *restrict type);
FILE *freopen(const char *restrict pathname, const char *restrict type, FILE *restrict fp);
FILE *fdopen(int filedes, const char *type);
//成功返回文件指针，出错返回NULL
int fclose(FILE *fp);
//成功返回0，出错返回EOF
```

| type值  | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| r或rb   | 打开只读文件，该文件必须存在                                 |
| r+或rb+ | 打开可读写文件，规则同上                                     |
| w或wb   | 打开只写文件，若文件存在则清空文件内容，若文件不存在则建立该文件 |
| w+或wb+ | 打开可读写文件，规则同上                                     |
| a或ab   | 以追加方式打开只写文件，若文件存在则写入的数据追加到文件尾，若文件不存在则建立该文件 |
| a+或ab+ | 打开可读写文件，规则同上                                     |

**注意**：在每个选项中加入 b 字符用来告诉函数库打开的文件为二进制文件，而非纯文本文件。

#### 5.3 字符I/O

字符I/O函数一次仅读写一个字符。

```c
#include <stdio.h>
int getc(FILE *fp);
int fgetc(FILE *fp);
int getchar(void);
//成功返回下一个字符，已达文件尾或出错返回EOF
int putc(int c, FILE *fp);
int fputc(int c, FILE *fp);
int putchar(int c);
//成功返回c，出错返回EOF
#int ferror(FILE *fp);
int feof(FILE *fp);
//若条件为真返回非0，否则返回0
void clearerr(FILE *fp);
```

#### 5.4 行I/O

行I/O函数一次操作一行。

```c
#include <stdio.h>
char *fgets(char *buf, int n, FILE *fp);
char *gets(char *buf);
//成功返回buf，已达文件尾或出错返回NULL
int fputs(const char *str, FILE *fp);
int puts(const *str);
//成功返回非负值，出错返回EOF
```

#### 5.5 格式化I/O

格式化I/O函数可以指定输入输出的具体格式。

**格式化输出**：**printf**将格式化数据写到标准输出，**fprintf**写至指定流，**sprintf**将格式化字符送入数组buf中。

**格式化输入**：**scanf**从标准输入读入格式化数据，**fscanf**读入指定流，**sscanf**从数组buf中读入格式化字符。

```c
#include <stdio.h>
/* 格式化输出 */
int printf(const char *format, ...);
int fprintf(FILE *fp, const char *format, ...);
//成功返回输出字符数，出错返回负值
int sprintf(char *buf, const char *format, ...);
//成功返回存入数组的字符数，出错返回负值
/* 格式化输入 */
int scanf(const char *format, ...);
int fscanf(FILE *fp, const char *format, ...);
int sscanf(const char *buf, const char *format, ...);
//返回输入项数，出错或已达文件尾返回EOF
```

#### 5.6 二进制I/O

二进制I/O函数对文件流进行读写操作。

```c
#include <stdio.h>
size_t fread(void *ptr, size_t size, size_t nobj, FILE *fp);
size_t fwrite(const void *ptr, size_t size, size_t nobj, FILE *fp);
//返回读写的对象数
/* 读或写一个二进制数组 */
float data[10];
if(fwrite(&data[2], sizeof(float), 4, fp) != 4) printf("...");
/* 读或写一个结构 */
struct Item { ... } item;
if(fwrite(&item, sizeof(item), 1, fp) != 1) printf("...");
```

### 6 ioctl函数
I/O操作的杂物箱，一般用于操作终端I/O。

```c
#include <unistd.h>
#include <sys/ioctl.h>
int ioctl(int fd, int request, ...);
//成功返回其他值，出错返回-1
```



###  7 文件与目录

#### 7.1 获取文件信息

**stat**函数返回与此命名文件有关的信息结构。

**fstat**函数获取在描述符上打开文件的有关信息。

**lstat**函数类似于stat，当文件是一个符号链接时返回该符合链接的有关信息。

```c
#include <sys/stat.h>

struct stat
{
    mode_t st_mode;
    ino_t st_ino;
    dev_t st_dev;
    time_t st_atime;	/*文件数据的最后访问时间*/
	time_t st_mtime;	/*文件数据的最后修改时间*/
	time_t st_ctime;	/*i节点状态的最后更改时间*/
}
int stat(const char *restrict pathname, struct stat *restrict buf);
int fstat(int fd, struct stat *buf);
int lstat(const char *restrict pathname, struct stat *restrict buf);
//成功返回0，出错返回-1
```

#### 7.2 umask、utime函数

**umask**为进程设置文件模式创建屏蔽字，并返回以前的值。**utime**更改一个文件的访问和修改时间。

```c
#include <sys/stat.h>
#include <utime.h>
mode_t umask(mode_t cmask);
//返回以前的文件模式创建屏蔽字，没有出错返回
int utime(const char *pathname, const struct utimbuf *times);
//成功返回0，出错返回-1
```

#### 7.3 更改文件权限

chmod和fchmod可以更改现有文件的访问权限。
```c
#include <sys/stat.h>
int chmod(const char *pathname, mode_t mode);
int fchmod(int filedes, mode_t mode);
//成功返回0，出错返回-1
```
**chmod**：在指定文件上进行操作。
**fchmod**：对已打开文件进行操作。

#### 7.4 创建目录

**mkdir**用于创建一个新的空目录，其中.和..目录项时自动创建的。**rmdir**用于删除一个空目录。

```c
#include <sys/stat.h>
int mkdir(const char *pathname, mode_t mode);
#include <unistd.h>
int rmdir(const char *pathname);
//成功返回0，出错返回-1
```
#### 7.5 读目录

对目录有访问权限的用户都可读目录，只有内核可以写目录。

**opendir**：返回DIR结构指针给另外5个函数使用。
**readdir**：读目录中的第一个目录项。

```c
#include <dirent.h>
DIR *opendir(const char *pathname);
//成功返回指针，出错返回NULL
struct dirent *readdir(DIR *dp);
//成功返回指针，出错返回NULL
void rewinddir(DIR *dp);
int closedir(DIR *dp);
//成功返回0，出错返回-1
```
#### 7.6 工作目录

每个进程都有一个当前工作目录，此目录是搜索所有相对路径名的起点。**chdir**和**fchdr**分别用文件路径和文件描述符来指定新的当前工作目录。**getcwd**获取当前工作目录绝对路径。
```c
#include <unistd.h>
int chdir(const char *pathname);
int fchdir(int fd);
//成功返回0，出错返回-1
char *getcwd(char *buf, size_t size);
//成功返回buf，出错返回NULL
```

### 8 终端I/O

#### 8.1 概述
**规范模式**：终端输入以行为单位进行处理。
**非规范模式**：输入字符并不组成行。
所有可检测和更改的终端设备特性都包含在termios结构中：
```c
#include <termios.h>
struct termios{
    tcflag_t c_iflag;
    tcflag_t c_oflag;
    tcflag_t c_cflag;
    tcflag_t c_lflag;
    cc_t c_cc[NCCS];
}
```
输入标志：控制字符的输入（如允许输入奇偶校验等）；输出标志：控制驱动程序输出（如执行输出处理等）；控制标志：影响RS-232串行线，本地标志影响驱动程序和用户之间的接口（如回显打开或关闭等）。c_cc数组包含所有可更改的特殊字符，元素数量在15~20

#### 8.2 特殊输入字符
POSIX定义了11个在输入时要特殊处理的字符，其中
| 字符  | 说明                                                       | c_cc下标 | 启用            | 典型值 |
| ----- | ---------------------------------------------------------- | -------- | --------------- | ------ |
| CR    | 回车，与NL作用相同，此字符返回给读进程                     | 不能更改 | c_lflag(ICANON) | \r     |
| EOF   | 文件结束，等待被读的所有字节（除此字符）被立即传送给读进程 | VEOF     |                 | ^D     |
| EOL   | 行结束，与NL作用相同                                       | VEOL     |                 |        |
| ERASE | 退格，向前擦除字符                                         | VERASE   |                 | ^H, ^? |
| INTR  | 中断，产生SIGINT信号，送至前台进程组的所有进程             | VINTR    | c_lflag(ISIG)   | ^C     |
| KILL  | 擦行，擦除一整行，此字符不返回给读进程                     | VKILL    |                 | ^U     |
| NL    | 换行，此字符返回给读进程                                   | 不能更改 |                 | \n     |
| QUIT  | 退出，产生SIGQUIT信号，送至前台进程组的所有进程            | VQUIT    | c_lflag(ISIG)   | ^\     |
| START | 启动，使停止的输出重新启动                                 | VSTRAT   | c_iflag(IXON)   | ^Q     |
| STOP  | 停止输出                                                   | VSTOP    |                 | ^S     |
| SUSP  | 挂起，产生SIGTSTP信号，送至前台进程组的所有进程            | VSUSP    | c_lflag(ISIG)   | ^Z     |

#### 8.3 获得和设置终端属性
调用tcgetattr和tcsetattr函数可检测和修改各种终端选项标志和特殊字符。
```c
#include <termios.h>
int tcgetattr(int fd, struct termios *termptr);
int tcsetattr(int fd, int opt, const struct termios *termptr);
//成功返回0，出错返回-1
```
指向termios结构的指针返回当前终端属性或设置该终端属性，若fd没有引用终端设备则出错返回-1。
参数opt指定在什么时候新的终端属性才起作用，可指定为TCSANOW（更改立即发生）、TCSADRAIN（发送所有输出后更改发送）、TCSAFLUSH（同上，更改发生未读输入数据被丢弃）。

#### 8.4 终端选项标志
屏蔽字标志定义多个位，有一个定义名，每个值也有一个名字。例如为了设置字符长度，首先用字符长度屏蔽字标志CSIZE将表示字符长度的位清0，然后设置下列值之一：CS5、CS6、CS7、CS8。
```c
int main(void)
{
    struct termios term;
    if(tcgetattr(STDIN_FILENO, &term) < 0) printf("error\n");
    switch(term.c_cflag & CSIZE) {
    case CS5 :
        printf("5 bits/byte\n");
        break;
    ...
    }
    term.c_cflag &= ~CSIZE;  //将字符长度位清零
    term.c_cflag |= CS8;     //设置为8位每字节
    if(tcsetattr(STDIN_FILENO, &term) < 0) printf("error\n");
    exit(0);
}
```
各选项标志的说明：
CSIZE：指定发送和接收的每个字节的位数，不包含奇偶检验位。
CSTOPB：若设置则使用两个停止位，否则使用一个停止位。
ECHO：若设置则将输入字符回显到终端设备。
PARENB：若设置则使能奇偶校验。
PARODD：若设置则使用奇校验，否则使用偶校验。

#### 8.5 波特率函数
波特率（baud rate）指的是“位/秒”。
```c
#include <termios.h>
speed_t cfgetispeed(const struct termios *termptr);
speed_t cfsetispeed(const struct termios *termptr);
//返回波特率值
int cfsetospeed(struct termios *termptr, speed_t speed);
int cfsetospeed(struct termios *termptr, speed_t speed);
//成功返回0，出错返回-1
```

#### 7 行控制函数
下列函数提供了终端设备的行控制能力，参数fd需引用一个终端设备。
```c
#include <termios.h>
int tcdrain(int fd);
int tcflow(int fd, int action);
int tcflush(int fd, int queue);
int tcsendbreak(int fd, int duration);
//成功返回0，出错返回-1
```
tcdrain函数等待所有输出都被传递。tcflow函数对输入和输出流进行控制。tcflush函数冲洗输入缓冲区或输出缓冲区。tcsendbreak函数在一指定时间区内发送连续的0值位流。

#### 8 终端标识
大多数UNIX系统版本中，控制终端的名字是/dev/tty。ctermid函数用来确定控制终端的名字。isatty判断文件描述符引用的是否为终端设备，ttyname返回在该文件描述符上打开的终端设备的路径名。
```c
#include <stdio.h>
char *ctermid(char *ptr);
//成功返回指向控制终端名的指针，出错返回指向空字符串的指针
int isatty(int fd);
//若为终端设备返回1，否则返回0
char *ttyname(int fd);
//成功返回指向终端路径名的指针，出错返回NULL
```

#### 9 规范模式
发送一个读请求，当一行输入后，终端驱动程序即返回。当所请求字节数已读到、读到一个行定界符或捕捉到信号且该函数不再自动启动时都将读返回。

#### 10 非规范模式
通过关闭termios结构中c_lflag字段的ICANON标志来指定非规范模式。在非规范模式中，输入数据不装配成行。非规范模式下，当已读了指定量数据或超过给定量的时间后，即通知系统返回。这种技术使用了termios结构中c_cc数组的两个变量：MIN和TIME。
MIN指定read返回前的最小字节数，TIME指定等待数据到达的分秒数。

#### 11 终端窗口大小
使用winsize结构。
```c
struct winsize {
    unsignded short ws_row;
    unsignded short ws_col;
    unsigned short ws_xpixel;
    unsigned short ws_ypixel;
};
```
- 使用ioctl的TIOCGWINSZ命令可获取此结构当前值。

- 使用ioctl的TIOCSWINSZ命令可将此结构的值存储到内核，若此值与内核中的当前值不同，前台进程组收到SIGWINCH信号。