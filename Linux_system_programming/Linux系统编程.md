# 讯为电子课程笔记

# 什么是 Linux 系统编程

Linux 系统编程也叫 Linux 下的高级编程，是介于应用层和驱动层
之间的。

# Linux 系统编程基本程序框架

# 标准 IO 和文件 IO

## 什么是文件 IO?

文件 IO 就是 **直接调用** 内核提供的系统调用函数。

## 什么是标准 IO?

标准 IO 就是 **间接调用** 系统调用函数，是 C 库函数。

## 关系图

![image-20251217194306562](Linux系统编程.assets/image-20251217194306562.png)



## 文件 IO 和标准 IO 的区别?

文件 IO 是直接调用内核提供的系统调用函数，头文件是 unistd.h，标准 IO 是间接调用系统调用函数，头文件是 stdio.h，文件 IO 是依赖于 **Linux 操作系统** 的，标准 IO 是不依赖操作系统的，所以在任何的操作系统下使用标准 IO，也就是 C 库函数操作文件的方法都是相同的。

## 文件IO的0 1 2 3 4 分别是什么意思??

在 Linux / POSIX 的**文件 I/O 语境**中，`0 1 2 3 4 ...` 并不是“魔法数字”，而是**文件描述符（file descriptor，fd）\**的编号。它们的含义既有\**约定俗成的标准值**，也有**运行时动态分配的值**。

下面分层说明。

------

## 一、0 / 1 / 2：**标准文件描述符（固定语义）**

这三个是 **POSIX 明确定义**的，所有进程在启动时就已经存在。

| fd   | 宏名            | 含义     | 默认指向    |
| ---- | --------------- | -------- | ----------- |
| 0    | `STDIN_FILENO`  | 标准输入 | 键盘 / 终端 |
| 1    | `STDOUT_FILENO` | 标准输出 | 终端        |
| 2    | `STDERR_FILENO` | 标准错误 | 终端        |

定义位置：

```c
#include <unistd.h>
#define STDIN_FILENO   0
#define STDOUT_FILENO  1
#define STDERR_FILENO  2
```

------

### 举例理解

```c
read(0, buf, 100);   // 从标准输入读
write(1, buf, len);  // 输出到终端
write(2, "err", 3);  // 错误输出
```

Shell 重定向的本质：

```sh
ls > out.txt      # 重定向 fd 1
ls 2> err.txt     # 重定向 fd 2
ls > all.txt 2>&1
```

------

## 二、3 / 4 / 5 ...：**普通文件描述符（动态分配）**

### 规则（非常重要）

> **内核总是返回“当前未使用的最小 fd”**

------

### 示例

```c
int fd1 = open("a.txt", O_RDONLY);   // fd = 3
int fd2 = open("b.txt", O_RDONLY);   // fd = 4
```

前提是：

- 0 / 1 / 2 已经被占用

------

### 管道示例

```c
int fd[2];
pipe(fd);   // fd[0]=3, fd[1]=4
```

------

## 三、为什么一定从 3 开始？

- 0 / 1 / 2 **在进程启动时就已占用**
- 用于：
  - 输入
  - 输出
  - 错误

👉 应用程序可以**默认安全使用 3 以后的 fd**

------

## 四、fd 本质是什么？（理解关键）

### 1️⃣ fd 是整数索引

- 指向进程的 **文件描述符表**

```text
进程
 └── fd table
     ├── 0 ──> struct file (stdin)
     ├── 1 ──> struct file (stdout)
     ├── 2 ──> struct file (stderr)
     ├── 3 ──> struct file (a.txt)
     ├── 4 ──> struct file (pipe write)
```

------

### 2️⃣ fd ≠ 文件本身

- 多个 fd 可指向同一个 `struct file`
- 例如：
  - `dup()`
  - `fork()`

------

## 五、fd 与 fork / exec 的关系（常考）

### fork

- 子进程 **继承父进程的 fd 0,1,2,3...**

------

### exec

- fd 默认不关闭
- 除非设置：
  - `FD_CLOEXEC`

------

## 六、嵌入式 Linux 常见场景

### 1️⃣ 串口

```c
fd = open("/dev/ttyS1", O_RDWR);   // fd=3
```

------

### 2️⃣ sysfs

```c
open("/sys/class/gpio/export", O_WRONLY);  // fd=3
```

------

### 3️⃣ 多路 I/O

```c
int fd1 = open(...);  // 3
int fd2 = open(...);  // 4
select(fd2+1, ...);
```

------

## 七、常见误区（重点）

### ❌ 认为 fd=3 一定是某个固定文件

❌ 错
✔ fd 只是“编号”，**内容完全取决于运行时**

------

### ❌ 认为 fd 和文件名绑定

❌ 错
✔ fd 是运行时对象，与路径无直接关系

------

## 八、一句话总结（工程级）

> **0、1、2 是进程启动时就存在的标准输入 / 输出 / 错误文件描述符；3 及之后是内核按“最小可用原则”动态分配给 open/pipe/socket 等资源的文件描述符。**

------

如果你愿意，下一步我可以：

- 画一张 **fd → struct file → inode 的完整关系图**
- 结合 `dup2()` 讲 I/O 重定向
- 解释为什么 close(1) 后 open() 会返回 1

你可以直接选一个继续。

## 文件IO的0 1 2 3 4 分别是什么意思??

在 Linux / POSIX 的**文件 I/O 语境**中，`0 1 2 3 4 ...` 并不是“魔法数字”，而是**文件描述符（file descriptor，fd）\**的编号。它们的含义既有\**约定俗成的标准值**，也有**运行时动态分配的值**。

下面分层说明。

------

### 一、0 / 1 / 2：**标准文件描述符（固定语义）**

这三个是 **POSIX 明确定义**的，所有进程在启动时就已经存在。

| fd   | 宏名            | 含义     | 默认指向    |
| ---- | --------------- | -------- | ----------- |
| 0    | `STDIN_FILENO`  | 标准输入 | 键盘 / 终端 |
| 1    | `STDOUT_FILENO` | 标准输出 | 终端        |
| 2    | `STDERR_FILENO` | 标准错误 | 终端        |

定义位置：

```c
#include <unistd.h>
#define STDIN_FILENO   0
#define STDOUT_FILENO  1
#define STDERR_FILENO  2
```

------

### 举例理解

```c
read(0, buf, 100);   // 从标准输入读
write(1, buf, len);  // 输出到终端
write(2, "err", 3);  // 错误输出
```

Shell 重定向的本质：

```sh
ls > out.txt      # 重定向 fd 1
ls 2> err.txt     # 重定向 fd 2
ls > all.txt 2>&1
```

------

### 二、3 / 4 / 5 ...：**普通文件描述符（动态分配）**

### 规则（非常重要）

> **内核总是返回“当前未使用的最小 fd”**

------

### 示例

```c
int fd1 = open("a.txt", O_RDONLY);   // fd = 3
int fd2 = open("b.txt", O_RDONLY);   // fd = 4
```

前提是：

- 0 / 1 / 2 已经被占用

------

### 管道示例

```c
int fd[2];
pipe(fd);   // fd[0]=3, fd[1]=4
```

------

### 三、为什么一定从 3 开始？

- 0 / 1 / 2 **在进程启动时就已占用**
- 用于：
  - 输入
  - 输出
  - 错误

👉 应用程序可以**默认安全使用 3 以后的 fd**

------

### 四、fd 本质是什么？（理解关键）

### 1️⃣ fd 是整数索引

- 指向进程的 **文件描述符表**

```text
进程
 └── fd table
     ├── 0 ──> struct file (stdin)
     ├── 1 ──> struct file (stdout)
     ├── 2 ──> struct file (stderr)
     ├── 3 ──> struct file (a.txt)
     ├── 4 ──> struct file (pipe write)
```

------

### 2️⃣ fd ≠ 文件本身

- 多个 fd 可指向同一个 `struct file`
- 例如：
  - `dup()`
  - `fork()`

------

### 五、fd 与 fork / exec 的关系（常考）

### fork

- 子进程 **继承父进程的 fd 0,1,2,3...**

------

### exec

- fd 默认不关闭
- 除非设置：
  - `FD_CLOEXEC`

------

### 六、嵌入式 Linux 常见场景

### 1️⃣ 串口

```c
fd = open("/dev/ttyS1", O_RDWR);   // fd=3
```

------

### 2️⃣ sysfs

```c
open("/sys/class/gpio/export", O_WRONLY);  // fd=3
```

------

### 3️⃣ 多路 I/O

```c
int fd1 = open(...);  // 3
int fd2 = open(...);  // 4
select(fd2+1, ...);
```

------

### 七、常见误区（重点）

### ❌ 认为 fd=3 一定是某个固定文件

❌ 错
✔ fd 只是“编号”，**内容完全取决于运行时**

------

### ❌ 认为 fd 和文件名绑定

❌ 错
✔ fd 是运行时对象，与路径无直接关系

------

### 八、一句话总结（工程级）

> **0、1、2 是进程启动时就存在的标准输入 / 输出 / 错误文件描述符；3 及之后是内核按“最小可用原则”动态分配给 open/pipe/socket 等资源的文件描述符。**

------

如果你愿意，下一步我可以：

- 画一张 **fd → struct file → inode 的完整关系图**
- 结合 `dup2()` 讲 I/O 重定向
- 解释为什么 close(1) 后 open() 会返回 1

你可以直接选一个继续。

# 文件 IO 之 open 函数

## 什么是文件描述符

文件描述符:
	对于文件 IO 来说，一切都是围绕文件描述符来进行的。在 Linux 系统中，所有打开的文件都有一个对应的文件描述符。

​	文件描述符的本质是一个非负整数，当我们打开一个文件时，系统会给我们分配一个文件描述符。
​	当我们对一个文件做读写操作的时候，我们使用 open 函数返回的这个文件描述符会标识该文件，并将其作为参数传递给 read 或者 write 函数

​	在 posix.1 应用程序里面，文件描述符 0,1,2 分别对应着标准输入，标准输出，标准出错。

## open 函数

open 函数

#include <sys/types.h>

#include <sys/stat.h>

#include <fcntl.h>

int open(const char *pathname, int flags);

int open(const char *pathname, int flags, mode t mode);



文件的访问权限我们可以使用数字来表示:
可执行-> 1

可写-> 2

可读-> 4

例:

如果我们要表示可读可写，就是上述值的 **和**，可读可写-> 6

rw- 为 6

r-- 为 4

# 文件 IO 之 close 函数

## close 函数

#include <unistd.h>
int close(int fd),
作用: 关闭一个文件描述符

# 文件 IO 之 read 函数

## read 函数

#include <unistd.h>
ssize t read(int fd, void *buf, size t count);

# 文件 IO 之 write 函数

## write 函数

#include <unistd.h>
ssize t write(int fd, const void *buf, size t count),

# 文件 IO 之 lseek 函数

## lseek 函数

#include <sys/types.h>

#include <unistd.h>

off_t lseek(int fd, off_t offset, int whence);

## 函数参数及返回值

参数:
fd: 文件描述符
whence: 当前位置的基点，可以使用以下三组值

​		SEEK SET: 相对于文件开头

​		SEEK CUR: 相对于当前的文件读写指针位置

​		SEEK END: 相对于文件末尾

offset: 偏移量，单位是字节的数量，可以正可负，如果是 **负值**，表示 **向前移动**，如果是 **正值**，表示 **向后移动**。

返回值: 成功返回当前位移，失败返回-1.

## 举例

例 1: 把文件位置指针设置为 **100**(相对于文件开头移动了 100 页)

​	Iseek(fd,100, SEEK SET);

例 2: 把文件位置设置成文件末尾(相对于文件末尾的位置为 0，即处于文件末尾)

​	Iseek(fd,0, SEEK END);

例 3: 确定当前的文件位置(相对于当前的文件读写指针位置为 0，即处在当前位置)

​	Iseek(fd,0, SEEK CUR),

# 目录 IO 之 mkdir 函数

## 文件IO和目录IO对比

目录IO
opendir 打开目录

mkdir 创建目录

readdir 读目录

closedir 关闭目录

文件IO

open 打开文件

读文件read

close 关闭文件

区别:

之前我们学习的文件IO和提到过的标准IO都是对文件操作，接下来学习的目录IO都是对**目录**的操作



## mkdir函数

#include <sys/stat.h>

#include <sys/types.h>

int mkdir(const char *pathname, mode t_mode);

# 目录 IO 之 opendir 和 closedir 函数

## opendir函数

#include <sys/types.h>

#include <dirent.h>
DIR *opendir(const char *name);

## closedir函数

#include <sys/types.h>

#include <dirent.h>

int closedir(DlR *dirp);

# 目录 IO 之 readdir 函数

## readdir函数

#include <dirent.h>

struct dirent *readdir(DIR *dirp);

# 库的基本概念

## 库的基本知识

1.什么是库?

​	库是一种可执行的二进制文件，是编译好的代码。

2.为什么要使用库?

​	提高开发效率。

3.Linux下库的种类

​	静态库:静态库在程序编译的时候会被链接到目标代码里面。所以我们的程序运行就不在需要该静态库了。因此编译出来的体积就比较大。静态库以**lib**开头，以**.a**结尾。

​	动态库:(动态库也叫共享库)动态库在程序编译的时候不会被链接到目标代码里面，而是在程序运行的时候被载入的，所以我们的程序运行就需要该动态库了。因此编译出来的体积就比较小。动态库以**lib**开头，以**.so**结尾。

# 静态库的制作与使用

## 静态库的制作

静态库制作步骤
1.编写或准备库的源代码

2.将源码.c文件编译生成.o文件

3.使用ar命令创建静态库

4.测试库文件

libmylib.a:库文件名

mylib:库名

## 静态库使用

​	如果我们的程序代码用到了库文件里面的函数，我们在编译的时候需要链接库。系统默认会在/ib或者/usr/ib去找库文件。或者在编译的时候我们制定库的路径。
举例:
​	gcc test.c -lmylib -L .

-l:指定静态库的库名

-L:指定静态库的查找位置

-L.表示在当前目录下去查找

# 动态库的制作与使用

## 动态库的制作

动态库制作步骤:
1.编写或准备库的源代码

2.将源码.c文件编译生成.o文件

3.使用gcc命令创建动态库

4.测试库文件

## 动态库使用

​	如果我们的程序代码用到了库文件里面的函数，我们在编译的时候需要链接库。系统默认会在/lib或

者/usr/lib去找库文件。或者在编译的时候我们制定库的路径。

举例:
	gcc test.c -lmylib -L .

-l:指定动态库的库名

-L:指定动态库的查找位置。

-L .表示在当前目录下去查找



## 解决使用动态库时，系统会在默认路径下查找的问题

​	在动态库使用时，系统会默认去/ib，/usr/lib目录下去查找动态函数库，如果我们使用的库不在里面，就会提示错误。解决这个问题有三种方法。
第一种方法:
​	将生成的动态库拷贝到/ib或者/usr/ib里面去，因为系统会默认去这俩个路径下寻找。

第二种方法:
	把我们的动态库所在的路径加到环境变量里面去，比如我们动态库所在的路径为/home/test，我们就可以这样添加。(但是只对当前的终端有效)

​	export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/test

第三种方法:
	修改ubuntu下的配置文件/etc/ld.so.conf，我们在这个配置文件里面加入动态库所在的位置，然后使用命令ldconfig更新目录。

# 进程基本知识

## 什么是进程?

进程指的是正在运行的程序，程序就是编译好的，放在磁盘上的二进制文件

## 进程ID

每个进程都有一个唯一的标识符，即进程ID，简称pid

## 进程间的通信的几种方法?

管道通信:有名管道，无名管道

信号通信:信号的发送，信号的接受，信号的处理

IPC通信:共享内存，消息队列，信号灯

Socket通信

# 进程控制

## 创建进程函数

## fork函数

头文件:include <unistd.h>

函数原型: pid_t fork(void);

返回值:fork函数有三种返回值，在**父进程**(让A进程创建B进程，A进程就是B进程的父进程)中，fork返回新创建的**子进程的PID**，在子进程中，fork返回**0**，如果出现错误，fork返回一个**负值**。

## 获得pid函数

函数:
	getpid():获得**当前进程**的PID

​	getppid():获得**当前进程的父进程**的PID

# exec 函数族

## exec函数族

​	在Linux中并没有exec函数，而是有6个以exec开头的函数族，下面列举了exec函数族的6个函数成员的函数原型。

int execl(const char *path, const char *arg, ...)

int execv(const char *path, char *const argv[])

int execle(const char *path, const char *arg, ..., char *const envp[])

int execve(const char *path, char *const argv[], char *const envp[])

int execlp(const char *file, const char *arg, ...)

int execvp(const char *file, char *const argv[])

## exec函数族的使用场景

在Linux中使用exec函数族主要有以下两种情况:

1.当进程认为自己不能再为系统和用户做出任何贡献时，就可以调用任何exec 函数族让自己重生。

2.如果一个进程想执行另一个程序，那么它就可以调用fork函数新建

个进程，然后调用任何一个exec函数使子进程重生。

# ps 和 kill 命令

ps和kill命令

​	ps命令:ps命令可以列出系统中当前运行的那些进程

命令格式:ps [参数]

命令功能:用来显示当前进程的状态

常用参数:aux

kill命令:

​	kill命令用来杀死进程

举例:kill -9(SIGKILL) PID

# 孤儿进程和僵尸进程

## 孤儿进程和僵尸进程

孤儿进程:
	**父进程结束以后，子进程还未结束**，这个子进程就叫做孤儿进程，孤儿进程会被系统的**Init进程**所领养，所以它的父进程会变。
僵尸进程:
	**子进程结束以后，父进程还在运行**，但是父进程不去释放进程控制块(PCB)，这个子进程就叫做僵尸进程。

## 僵尸进程如何变成孤儿进程?? 孤儿进程的条件不是父进程结束以后，子进程还未结束么，但是僵尸进程的条件是子进程结束以后，父进程还在运行。它们不是相互矛盾吗?

你的理解有一定道理，但僵尸进程和孤儿进程的概念和行为之间确实有些细微的差别。让我们一起理清这两个概念：

### 1. **僵尸进程**（Zombie Process）

- **定义**：僵尸进程是指已经终止（即退出）的子进程，但其父进程还没有调用 `wait()` 或 `waitpid()` 等函数来获取子进程的退出状态（即子进程的“退出信息”）。在这种情况下，子进程的进程表项不会被完全清除，进程的 **PID** 和其他信息仍然存在，虽然进程已经结束。
- **原因**：当子进程执行完毕时，它的退出状态（如退出码）需要由父进程收集（调用 `wait()`）。如果父进程没有及时收集该信息，子进程就会变成一个僵尸进程。

### 2. **孤儿进程**（Orphan Process）

- **定义**：孤儿进程是指父进程结束后，子进程仍然存在。由于父进程已经死亡，操作系统会将这些子进程的父进程（即父进程 PID）更改为 `init` 进程（PID 1）。`init` 进程会负责清理这些孤儿进程（包括收集它们的退出状态）。
- **原因**：如果父进程提前结束，而子进程仍在运行，子进程就成为孤儿进程。操作系统会自动将这些子进程的父进程设置为 `init` 进程，确保这些进程不会遗留在系统中。

### 为什么“僵尸进程”和“孤儿进程”不是相互矛盾的？

你提到的“父进程结束后子进程还没结束”是孤儿进程的典型条件。**但僵尸进程的条件是子进程已经结束且父进程尚未处理子进程的退出状态**。这两个概念虽然都涉及到父子进程，但它们描述的是不同的阶段：

- **僵尸进程的形成**：父进程还活着，子进程已经结束，但父进程没有收集子进程的退出状态（通过 `wait()` 系统调用）。在这种情况下，子进程的进程表条目不会完全释放，变成了“僵尸”状态。
- **孤儿进程的形成**：父进程提前终止，而子进程仍在运行。此时，子进程会成为孤儿进程，操作系统会把它的父进程设置为 `init` 进程，`init` 进程负责回收孤儿进程的退出状态。

### 僵尸进程如何变成孤儿进程？

这两者的关系也可以通过实际场景来理解。**如果一个父进程结束时，其子进程还未结束，那么这些子进程就会成为孤儿进程**，它们的父进程会被转交给 **`init` 进程**。**如果孤儿进程最终结束，而它的退出状态仍然没有被回收，那么它可能会成为僵尸进程**。所以，孤儿进程和僵尸进程可以在生命周期中**有所重叠**，但它们并不是同一概念。

### 举个例子：

1. 假设父进程A启动了子进程B。
2. 父进程A结束，而子进程B仍在运行。此时，子进程B成为**孤儿进程**，它的父进程变成了 **`init` 进程**。
3. 子进程B最终结束，但如果 `init` 进程没有及时收集子进程B的退出状态（通过 `wait()`），那么子进程B就变成了**僵尸进程**，直到 `init` 进程收集它的退出信息。

### 总结：

- **僵尸进程**：子进程已经终止，但父进程尚未收集子进程的退出信息。
- 孤儿进程：父进程结束时，子进程仍在运行，操作系统会将子进程的父进程设为 **`init` 进程**。

两者的核心区别在于**父进程的状态**（是否仍然存活）。孤儿进程是由于**父进程死亡**导致的，而僵尸进程是因为**父进程没有处理子进程退出状态**导致的。

# wait 函数

## wait函数

头文件:#include <sys/wait.h>

函数原型:pid_t wait(int *status)

返回值:成功返回回收的子进程的pid，失败返回-1

与wait函数的参数有关的俩个宏定义:

WIFEXITED(status):如果子进程正常退出，则该宏定义为真

WEXITSTATUS(status):如果子进程正常退出，则该宏定义的值为子进程的退出值。

# 守护进程

## 什么是守护进程?

守护进程运行在**后台**，不跟任何控制终端关联。

## 怎么创建一个守护进程?

有俩个基本要求:1.必须作为我们init进程的子进程 2.不跟控制终端交互。

1.使用fork函数创建一个新的进程，然后让父进程使用exit函数直接退出(必须要的)

2.调用setsid函数，摆脱控制终端。(必须要的)

3.调用chdir函数，将当前的工作目录改成根目录，增强程序的健壮性。(不是必须要的)

4.重设我们umask文件掩码，增强程序的健壮性和灵活性(不是必须要的)

5.关闭文件描述符，节省资源(不是必须要的)

6.执行我们需要执行的代码(必须要的)

## 会话 进程组 进程

在操作系统中，**会话**、**进程组** 和 **进程** 都是进程管理的重要概念。它们在进程的组织、控制以及信号传递等方面起着关键作用。下面我们逐一介绍这些概念。

### 1. **进程（Process）**

进程是操作系统进行**资源分配**和**调度**的基本单位。它是程序在某个数据集合上的一次执行活动。一个进程通常包含以下几部分：

- **程序代码**：执行的指令。
- **数据**：进程在运行过程中需要的数据。
- **状态信息**：包括进程的当前状态（运行、就绪、阻塞等）、程序计数器、堆栈、寄存器等。
- **资源**：文件描述符、内存等。

每个进程都有唯一的 **进程标识符（PID）**，用于在系统中标识进程。进程在操作系统中执行时，会在 **进程表** 中注册相关信息。

### 2. **进程组（Process Group）**

进程组是**多个进程的集合**，通常用于对一组进程进行统一管理和信号控制。进程组是为了方便管理和控制多个进程而引入的概念，尤其是在处理信号时非常有用。

- **进程组标识符（PGID）**：进程组有自己的标识符，即 **PGID**。默认情况下，进程的 **PGID** 和进程的 **PID** 相同，但一个进程可以通过调用 `setpgid()` 函数修改自己的进程组。
- **进程组的用途**：
  - 方便发送信号：通过进程组，父进程可以向一组子进程发送信号（例如，发送终止信号给整个进程组）。
  - 会话控制：进程组与会话密切相关，它们帮助操作系统对进程进行分组、管理。
- **如何形成进程组**：
  - 当一个进程通过 `fork()` 创建一个子进程时，父进程和子进程通常会属于同一个进程组。
  - 进程组通常是由一个进程（称为进程组的“领头进程”）作为“组长”，通过 `setpgid()` 可以显式地设置进程的组长。

### 3. **会话（Session）**

会话是由**一个或多个进程组**组成的集合。每个会话都有一个**会话领导进程**（通常是会话中的第一个进程）。会话的引入是为了方便管理与控制终端中的多个进程，尤其是在需要用户交互的情况下（例如，登录时启动的shell进程）。

- **会话标识符（SID）**：每个会话都有一个唯一的会话标识符。会话标识符通常是会话领导进程的PID。
- **会话的用途**：
  - **控制终端**：一个会话可以与控制终端相关联，这样用户可以与会话中的所有进程进行交互。
  - **进程组控制**：会话和进程组的关系帮助实现对进程的管理，特别是通过发送信号。
  - **后台进程**：会话可以包括**前台进程和后台进程**。通过会话的概念，可以区分终端上显示的前台进程和在后台运行的进程。
- **如何形成会话**：
  - 一个进程可以通过调用 `setsid()` 创建一个新的会话。`setsid()` 会使调用进程成为**会话领导进程**，并**将该进程从其当前进程组中分离出来**，成为一个**新的会话的会话领导进程**。
  - 会话中的所有进程可以访问**同一个控制终端**（如果它们有一个控制终端的话）。

### 会话、进程组和进程之间的关系：

- **进程** 是最基本的单位，每个进程有自己的PID。
- **进程组** 是多个进程的集合，可以方便地对进程组中的所有进程进行信号操作。进程组中通常有一个**“组长”进程**。
- **会话** 是由多个进程组组成的集合。一个会话中通常有一个会话领导进程（SID）。会话中的进程**共享一个控制终端**，并且能够接受终端的输入和输出。

### 总结：

- **进程**：操作系统中资源分配和调度的基本单位，每个进程有一个唯一的PID。
- **进程组**：多个进程的集合，进程组有一个PGID。进程组用于统一管理多个进程，特别是在信号传递时。
- **会话**：由多个进程组组成的集合。会话中包含一个会话领导进程（SID），可以管理多个进程组，通常与终端相关联。

这些概念通常用于操作系统的进程管理、信号处理、会话控制等方面，帮助系统更好地管理和控制进程的生命周期。

# 管道通信之无名管道

## pipe函数

头文件:#include <unistd.h>

函数原型:int pipe(int pipefd[2]):

作用:创建管道

# 管道通信之有名管道

## mkfifo函数

#include <sys/types.h>

#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode)

# 信号通信(一)

## 信号发送函数

kill函数:
#include <sys/types.h>

#include <signal.h>

int kill(pid_t pid, int sig),

raise函数:

#include <signal.h>

int raise(int sig);

raise函数等价于 kill(getpid(),sig);

getpid()是获得当前进程的PID号

alarm函数:

#include <unistd.h>

unsigned int alarm(unsigned int seconds);

# 信号通信(二)

## 信号的接收

​	接收信号:如果要让我们接收信号的进程可以接收到信号，那么这个进程就不能停止。

## pause函数

头文件:#include <unistd.h>

定义函数:int pause(void);

函数说明:pause()会令目前的进程暂停(进入睡眠状态),直到被信号(signal)所中断.

返回值:只返回-1.

# 信号通信(三)

## 信号的处理

信号处理的三种方式:

系统默认(即中止进程),

忽略(你给我发个信号，我给忽略了),

捕获(你给我发个信号，我给捕获了，然后我去执行一些自己定义的事情)

## signal函数

头文件:

#include <signal.h>

函数原型:

typedef void (*sighandler_t)(int);

sighandler_t signal(int signum, sighandler_t handler);

可以简化成:

signal(参数1，参数2):

参数1:我们要进行处理的信号,系统的信号我们可以再终端键入 kill -l查看。

参数2:处理的方式(是系统默认还是忽略还是捕获)

## 示例

1)signal(SIGINT ,SIG_IGN, );

​	SIG_IGN,代表**忽略**，也就是忽略SIGINT信号

2)signal(SIGINT ,SIG_DFL );

​	SIG_DFL 代表**执行系统默认操作**，大多数信号的系统默认动作时**终止**该进程。

3)signal(SlGlNT ,myfun);

​	**捕捉**SIGINT这个信号，然后执行myfun函数里面的代码。myfun由我们自己定义。

# Linux 信号速查总表

下面我对 **Linux 常见信号进行系统性总结**，并**标注标准 Linux 下的信号编号**。内容按照你图中思路，结合**默认行为**进行整理，便于学习、记忆和查 man 手册。

> 说明：
>
> - 信号编号以 **Linux（x86 / ARM，glibc）常见取值**为准
> - 个别体系结构编号可能略有差异，但**语义一致**

------

## 一、Linux 信号速查总表（重点版）

| 编号   | 信号名    | 含义说明               | 默认行为    |
| ------ | --------- | ---------------------- | ----------- |
| **1**  | SIGHUP    | 终端挂断 / 会话结束    | 终止        |
| **2**  | SIGINT    | 键盘中断（Ctrl+C）     | 终止        |
| **3**  | SIGQUIT   | 键盘退出（Ctrl+\）     | 终止 + core |
| **4**  | SIGILL    | 非法指令               | 终止 + core |
| **5**  | SIGTRAP   | 调试陷阱               | 终止 + core |
| **6**  | SIGABRT   | abort() 触发           | 终止 + core |
| **7**  | SIGBUS    | 总线错误（对齐/硬件）  | 终止 + core |
| **8**  | SIGFPE    | 算术异常（除 0）       | 终止 + core |
| **9**  | SIGKILL   | 强制终止（不可捕获）   | 终止        |
| **10** | SIGUSR1   | 用户自定义信号 1       | 终止        |
| **11** | SIGSEGV   | 段错误（非法内存访问） | 终止 + core |
| **12** | SIGUSR2   | 用户自定义信号 2       | 终止        |
| **13** | SIGPIPE   | 管道破裂               | 终止        |
| **14** | SIGALRM   | alarm() 定时到期       | 终止        |
| **15** | SIGTERM   | 终止请求（推荐）       | 终止        |
| **16** | SIGSTKFLT | 协处理器栈错误         | 终止        |
| **17** | SIGCHLD   | 子进程状态变化         | 忽略        |
| **18** | SIGCONT   | 继续执行               | 忽略        |
| **19** | SIGSTOP   | 停止进程（不可捕获）   | 停止        |
| **20** | SIGTSTP   | 终端停止（Ctrl+Z）     | 停止        |
| **21** | SIGTTIN   | 后台进程读终端         | 停止        |
| **22** | SIGTTOU   | 后台进程写终端         | 停止        |
| **23** | SIGURG    | 紧急 I/O               | 忽略        |
| **24** | SIGXCPU   | CPU 时间超限           | 终止        |
| **25** | SIGXFSZ   | 文件大小超限           | 终止        |
| **26** | SIGVTALRM | 虚拟定时器到期         | 终止        |
| **27** | SIGPROF   | 性能统计定时器         | 终止        |
| **28** | SIGWINCH  | 窗口大小变化           | 忽略        |
| **29** | SIGIO     | I/O 就绪               | 终止        |
| **30** | SIGPWR    | 断电（Power fail）     | 终止        |
| **31** | SIGSYS    | 非法系统调用           | 终止 + core |

------

## 二、按“默认行为”分类（理解用）

### 1️⃣ 默认 **终止进程**

```text
SIGHUP  SIGINT  SIGKILL  SIGPIPE
SIGALRM SIGTERM SIGUSR1 SIGUSR2
SIGXCPU SIGXFSZ SIGPROF SIGPWR
SIGIO
```

------

### 2️⃣ 默认 **终止 + core dump**

```text
SIGQUIT SIGILL SIGTRAP SIGABRT
SIGBUS  SIGFPE SIGSEGV SIGSYS
```

> 常见段错误 / 崩溃类信号

------

### 3️⃣ 默认 **停止进程**

```text
SIGSTOP SIGTSTP SIGTTIN SIGTTOU
```

------

### 4️⃣ 默认 **忽略**

```text
SIGCHLD SIGCONT SIGURG SIGWINCH
```

------

## 三、最重要的“不可捕获信号”

| 信号    | 编号 | 特点               |
| ------- | ---- | ------------------ |
| SIGKILL | 9    | 不能捕获、不能忽略 |
| SIGSTOP | 19   | 不能捕获、不能忽略 |

👉 **内核级强制行为**

------

## 四、与你前面学过内容的对应关系

| 函数 / 场景   | 使用信号     |
| ------------- | ------------ |
| `Ctrl + C`    | SIGINT (2)   |
| `Ctrl + Z`    | SIGTSTP (20) |
| `kill pid`    | SIGTERM (15) |
| `kill -9 pid` | SIGKILL (9)  |
| `alarm()`     | SIGALRM (14) |
| `raise()`     | 任意信号     |
| 子进程退出    | SIGCHLD (17) |

------

## 五、学习与工程建议（嵌入式 Linux）

1. **只重点记住 10 个左右**
   - SIGINT / SIGTERM / SIGKILL
   - SIGSEGV / SIGABRT
   - SIGALRM / SIGCHLD
   - SIGSTOP / SIGCONT
2. 其余信号：
   - 会查 `man 7 signal`
   - 知道分类即可

------

## 六、一句话总结（面试级）

> **Linux 信号是进程间异步通知机制；大多数信号可捕获，SIGKILL 和 SIGSTOP 由内核强制；默认行为分为终止、终止并转储、停止和忽略四类。**

------

如果你愿意，下一步我可以：

- 帮你画一张 **“信号 → 默认行为 → 常见来源”思维导图**
- 用代码演示 **sigaction 捕获多个信号**
- 把这张表压缩成 **嵌入式 Linux 必背版**

你可以直接说继续哪一个。

# 共享内存

## shmget函数

头文件:

#include <sys/ipc.h>

#include <sys/shm.h>

函数原型:

int shmget(key_t key, size_t size, int shmflg),

参数:

key t key:IPC_PRIVATE或者是ftok函数的返回值

size_t size:共享内存的大小

int shmflg:权限

返回值:

成功返回共享内存的标识符，失败返回-1

## ftok函数

头文件:

#include <sys/types.h>

#include <sys/ipc.h>

函数原型:

key_t ftok(const char *pathname, int proj_id);参数:

const char *pathname 文件路径以及文件名

int proj_id:字符

返回值:成功返回key值，失败返回-1

## shmat函数

头文件:

#include <sys/types.h>

#include <sys/shm.h>

函数原型:

void *shmat(int shmid, const void *shmaddr, int shmflg)

参数:

int shmid:共享内存的标识符，也就是shmget函数的返回值

const void *shmaddr:映射到的地址，一般写NULL，NULL为系统自动

帮我完成映射

int shmflg:通常为0，表示共享内存可读可写，或者为SHM_RDONLY表示共享内存只读

返回值:成功返回共享内存映射到进程中的地址，失败返回-1

## shmdt函数

头文件:

#include <sys/types.h>

#include <sys/shm.h>

函数原型:

int shmdt(const void *shmaddr);

参数:const void *shmaddr:共享内存映射后的地址
返回值:成功返回0，失败返回-1注意:shmdt函数是将进程中的地址映射删除，也就是说当一个进程不需要共享内存的时候，就可以使用这个函数将他从进程地址空间中脱离，并不会删除内核里面的共享内存对象。

## shmctl函数

头文件:

#include <sys/ipc.h>

#include <sys/shm.h>

函数原型:

int shmctl(int shmid, int cmd, struct shmid_ds *buf);

参数:intshmid:要删除的共享内存的标识符

int cmd:IPC_STAT(获取对象属性) IPC_SET(设置对象属性)IPC_RMID(删除对象)

struct shmid_ds *buf:指定IPC STAT(获取对象属性)IPC_SET(设置对象属性) 时用来保存或者设置的属性。

# 消息队列

## msgget函数

头文件:
#include <sys/types.h>

#include <sys/ipc.h>

#include <sys/msg.h>

函数原型:int msgget(key_t key, int msgflg)

参数:key_tkey:和消息队列相关的key值

​	int msgflg:访问权限

返回值:成功返回消息队列的ID，失败-1

## msgctl函数

头文件:

#include <sys/types.h>

#include <sys/ipc.h>

#include <sys/msg.h>

函数原型:int msgctl(int msqid, int cmd, struct msqid_ds *buf);

参数:intmsqid:消息队列的ID
	int cmd:IPC_STAT:读取消息队列的属性，然后把它保存在buf指向的缓冲区。

IPC_SET:设置消息队列的属性，这个值取自buf参数

IPC_RMID:删除消息队列
	struct msqid_ds *buf:消息队列的缓冲区

返回值:成功返回0，失败返回-1

## msgsnd函数

头文件:

#include <sys/types.h>

#include <sys/ipc.h>

#include <sys/msg.h>

函数原型:int msgsnd(int msqid, const void msgp, size_t msgsz, int msgflg)

参数:int msqid:消息队列ID，const void*msgp:指向消息类型的指针size_t msgsz:发送的消息的字节数。

int msgfg:如果为0，直到发送完成函数才返回，即阻塞发送，IPC_NOWAIT:消息没有发送完成，函数也会返

回，即非阻塞发送

返回值:成功返回0，失败返回-1

## 消息结构体

struct msgbuf {

​	long mtype;	消息的类型

​	char mtext[1];	消息的内容

};

## msgrcv函数

头文件:

#include <sys/types.h>

#include <sys/ipc.h>

#include <sys/msg.h>

函数原型:ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp, int msgflg);

参数:int msqid:消息队列ID，void *msgp:接收消息的缓冲区

size_t msgsz:接收消息的字节数  long msgtyp:接收消息的标识

int msgflg:0表示阻塞读取，IPC NOWAIT表示非阻塞读取

返回值，成功返回接收到的消息的长度，失败返回-1

# 信号量

## semget函数

头文件:

#include <sys/types.h>

#include <sys/ipc.h>

#include <sys/sem.h>

函数原型:int semget(key_t key, int nsems, int semflg)

函数功能:获得信号量的ID

参数:key tkey:信号量的键值。int nsems:信号量的数量。

​	int semflg:标识

返回值:成功返回信号量的ID

​	失败返回-1



## semctl函数

头文件:

#include <sys/types.h>

#include <sys/ipc.h>

#include <sys/sem.h>

函数原型:int semctl(int semid, int semnum, int cmd, union semun arg)

参数:int semid:信号量ID，intsemnum:信号量编号

​	cmd: IPC_STAT(获取信号量的属性) IPC_SET(设置信号量的属性)

​	IPC_RMID(删除信号量)SETVAL(设置信号量的值)

arg: union semun{

​		int val;

​		struct semid_ds *buf;

​		unsigned short *array;

​		struct seminfo **__*buf;

};

## semop函数

头文件:

#include <sys/types.h>

#include <sys/ipc.h>

#include <sys/sem.h>

函数原型:int semop(int semid, struct sembuf *sops, size_t nsops)

参数:int semid信号量ID，struct sembuf *sops信号量结构体数组

​	size_t nsops 要操作信号量的数量

struct sembuf{
	unsigned short sem_num; //要操作的信号量的编号

​	short sem_op; //P/V操作，1为V操作，释放资源。-1为P操作，分配资源。0为等待，直到信号量的值变成0

​	short sem_flg; //0表示阻塞，IPC_NOWAIT表示非阻塞

}

# man 章节对照表

在 Linux 中，`man 1`、`man 3` 并不是两个不同的命令，而是**同一个 `man` 命令，查看“不同章节（section）”的手册页**。这是一个非常典型、也非常容易混淆的问题。

下面我从**man 手册的结构 → 各章节含义 → 为什么会有 man 1 / man 3 → 实际使用建议**来系统说明。

------

## 一、`man` 手册的整体结构

Linux 的 man 手册被分成多个**章节（Section）**，每一章对应一类内容。

> 章节号写在 `man` 后面，用来**指定你要看的“是哪一类说明”**。

------

## 二、常见 man 章节对照表（重点）

| 章节  | 含义           | 典型内容                    |
| ----- | -------------- | --------------------------- |
| **1** | 用户命令       | `ls`, `cp`, `grep`          |
| **2** | 系统调用       | `open`, `read`, `fork`      |
| **3** | 库函数         | `printf`, `malloc`, `sleep` |
| 4     | 设备与特殊文件 | `/dev/tty`                  |
| 5     | 配置文件格式   | `/etc/passwd`               |
| 6     | 游戏           | `sl`                        |
| 7     | 杂项 / 规范    | `man-pages`, `signal`       |
| 8     | 管理员命令     | `mount`, `ifconfig`         |

> **你现在嵌入式 Linux 学习中最常用的：1 / 2 / 3**

------

## 三、`man 1` 和 `man 3` 的本质区别

### 1️⃣ `man 1 xxx`

- 查的是：**“终端里输入的命令”**
- 面向对象：**用户 / Shell 使用者**

示例：

```bash
man 1 ls
```

你看到的是：

- 命令行选项
- 用法
- 示例

------

### 2️⃣ `man 3 xxx`

- 查的是：**C 语言库函数**
- 面向对象：**程序员**

示例：

```bash
man 3 printf
```

你看到的是：

- 函数原型
- 头文件
- 参数说明
- 返回值
- 错误码

------

## 四、为什么同一个名字会同时出现在 man1 和 man3？

这是**最容易困惑的地方**。

### 示例：`printf`

```bash
man 1 printf   # shell 命令
man 3 printf   # C 库函数
```

### 本质原因

| 名称              | 实际身份       |
| ----------------- | -------------- |
| `printf`（man 1） | shell 内建命令 |
| `printf`（man 3） | libc 函数      |

👉 **名字相同，但完全不是一个东西**

------

## 五、如果不写章节号会发生什么？

```bash
man printf
```

行为规则：

> **man 会按照“默认优先级顺序”查找章节**

通常顺序是：

```text
1 → 2 → 3 → ...
```

所以你很可能看到的是：

```text
printf(1)
```

而不是你想要的：

```text
printf(3)
```

------

## 六、嵌入式 Linux 学习中的实用建议（经验）

### 1️⃣ 查命令

```bash
man 1 ls
man 1 ps
```

------

### 2️⃣ 查系统调用

```bash
man 2 open
man 2 read
man 2 fork
```

------

### 3️⃣ 查库函数（你现在最常用）

```bash
man 3 sleep
man 3 malloc
man 3 strcpy
```

------

### 4️⃣ 不确定在哪一章？

```bash
man -k open
```

或：

```bash
apropos open
```

输出示例：

```text
open (2)   - open and possibly create a file
open (3)   - open a stream
```

------

## 七、与你当前学习内容的直接关联

你最近学的这些函数：

| 函数       | 正确章节 |
| ---------- | -------- |
| `open()`   | man 2    |
| `read()`   | man 2    |
| `sleep()`  | man 3    |
| `mkfifo()` | man 3    |
| `printf()` | man 3    |
| `ls`       | man 1    |

------

## 八、常见误区总结

### ❌ 误区 1

`man 1` 和 `man 3` 是**两个命令**

✔ 实际：**同一个 man，不同章节**

------

### ❌ 误区 2

不写章节也能查到正确内容

✔ 实际：**可能查错对象**

------

## 九、一句话总结（面试级）

> **`man 1` 查看用户命令，`man 3` 查看 C 语言库函数；章节号用于区分同名但语义完全不同的对象，是 Linux man 手册的分类体系。**

------

如果你愿意，下一步我可以：

- 专门帮你区分 **man 2（系统调用）vs man 3（库函数）**
- 给你一张 **嵌入式 Linux 必查 man 页速查表**
- 用 `strace` 演示“man2 的函数在内核里怎么跑”

你可以直接说继续哪一项。