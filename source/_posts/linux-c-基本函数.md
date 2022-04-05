---
title: linux c 基本函数
date: 2021-08-15 16:55:07
categories: 
  - linux学习
tags:
  - linux
  - c
---

## open

### 函数原型

```c
 #include <fcntl.h>

       int open(const char *pathname, int flags);
       int open(const char *pathname, int flags, mode_t mode);
```

<!--more-->

open 函数可以打开 pathname 指定的文件，如果这个指定的文件不存在，可以选择创建它（前提是 O_CREAT 在 flags 中指定了），而指定了 O_CREAT 后需要指定第三个参数 mode 。

其中 flags 定义的宏在 fcntl.h 头文件中。

---

举例：

```c
int fd = open("yuhan.txt", O_RDWR | O_CREAT, 0644);
```

该语句打开当前目录下的 yuhan.txt 文件，指定打开的权限是可读可写，如果该文件不存在则创建它，创建该文件的权限为 rw-r--r--。

前面加0，0644代表八进制数。

**如果不创建：**

```c
int fd = open("yuhan.txt", O_RDWR);
```

---

### 返回值

该函数返回一个 int 类型的文件描述符（非负整数），搭配 read、write 等函数使用。

成功调用的返回值是当前未被进程打开的最小的可用的文件描述符。

**如果失败：**

返回 -1 并且设置 errno , errno 是被系统创建的一个用来标识错误的变量，用 perror 输出错误。

### 其它描述

- 默认的该函数返回的文件描述符所打开的文件处于始终打开的状态，可以用 close 函数关闭。

- 参数 flags 必须包含 **O_RDONLY、O_WRONLY、O_RDWR** 中的一个。

- O_TRUNC 覆盖写

比如：

```c
int fd = open("yuhan.txt", O_RDWR | O_CREAT | O_TRUNC , 0644);
```

- O_APPEND 追加写
- 创建文件 creat 可以用 open 实现，使用 flags ： O_CREAT|O_WRONLY|O_TRUNC 

比较重要的大概就这一些。

## read

### 函数原型

```c
 #include <unistd.h>

       ssize_t read(int fd, void *buf, size_t count);
```

read 函数通过文件描述符 fd 读取 count 字节大小的内容到 buf 缓冲区中

---

举例：

```c
    char buf[1024];
    // yuhan.txt: <hello i'am yuhanOvo>
    int fd = open("yuhan.txt", O_RDWR);
    while(read(fd, buf, 1024)) {
        printf(" buf: %s", buf);
    }

// >> buf: hello i am yuhanOvo
```

### 返回值

调用成功时，返回读取的字节数（为0时，则表示读到了文件结尾）。并且，下次读取文件的位置被偏移了返回值的大小个字节。也就是第二次 read 会从第一次 read 的结尾处继续读取。

如果返回的值小于请求的参数 count ， 这不是一个错误，因为剩余可读取的字节数小于 count 的大小是完全有可能的。*或者我们可能在一个管道上读取。其它原因不展开。

调用失败时，返回 -1 ，并且系统会设置 errno 来指示错误。

比如：

```c
n = read(1000, buf, 1024);
    if(n == -1) {
        perror("read error");
    }

// >> read error: Bad file descriptor
```

### 其它描述

关于 count 的大小选取，如果比 SSIZE_MAX 大，结果未定义。

## write

### 函数原型

```c
 #include <unistd.h>

       ssize_t write(int fd, const void *buf, size_t count);
```

write 函数从缓冲区 buf 中读取 count 字节的内容写入文件描述符 fd 所指向的文件中。

---

举例：

```c
int fd = open("yuhan.txt", O_RDWR | O_TRUNC);
char *hello = "hello world";
write(fd, hello, strlen(hello));
```

### 返回值

调用成功时，返回写入的字节数。

调用失败时，返回 -1，并设置 errno 。

**注意：**

返回的字节数可能小于 count 的大小，可能的原因之一是写入的硬件磁盘的容量不够。

### 其它描述

暂无

## fcntl

### 函数原型

```c
#include <fcntl.h>

       int fcntl(int fd, int cmd, ... /* arg */ );
```

fcntl 函数用来控制文件描述符

cmd 可以是各种被设置好的宏，当 cmd 表示为不同的宏时 fcntl 具有不同的功能。

#### 复制描述符

F_DUPFD (int类型)

---

举例：

```c
int fd = open("yuhan.txt", O_RDWR | O_TRUNC);
int fd2 = fcntl(fd, F_DUPFD, 0);
```

此时 fd2 和 fd 所指向的文件相同。

注意第三个参数 0 ，代表返回一个大于等于 0 的可用的文件描述符。



这个函数还不太熟悉。

## lseek - 定位读/写文件偏移量

参考： https://man7.org/linux/man-pages/man2/lseek.2.html

### 函数原型

```c
 #include <unistd.h>

       off_t lseek(int fd, off_t offset, int whence);
```

定位读或者写文件的偏移量。

### 描述

lseek 函数根据偏移量 *offset* 和偏移模式 *whence* 来定位 *fd* 所关联文件的偏移位置。 

####　whence 参数

SEEK_SET：  从文件开始的位置偏移 *offset* 个偏移量

SEEK_CUR：  从文件指针当前挪到的位置偏移 *offset* 个偏移量

SEEK_END： 从文件结尾的位置偏移 *offset* 个偏移量

---

*offset* 可以是负数， 表示向左偏移。

lseek 函数允许偏移量设置在文件的结尾之后，但是这将不会改变文件的大小，只有在偏移之后再写入数据到文件之后，那些偏移了但是还没有填入数据的位置为一个空洞（'\0'）。

### 返回值

成功时，返回从文件开头到所偏移到的位置一共的偏移量（以字节为单位）。

失败时，返回 -1， 设置 errno。

###　一些用处

拓展文件大小：

```c
int fd = open("txt", O_RDWR | O_CREAT, 0644);
lseek(fd, 100, SEEK_SET);
write(fd, "\0", 1);
```

此文件大小为 101个字节。

其它方式还有：

ftruncate() 

```c
       #include <unistd.h>

       int truncate(const char *path, off_t length);
       int ftruncate(int fd, off_t length);
```

ftruncate(fd, 100);

## dup, dup2, dup3 - 复制一个文件描述符

参考： https://man7.org/linux/man-pages/man2/dup.2.html

### 函数原型

```c
       #include <unistd.h>

       int dup(int oldfd);
       int dup2(int oldfd, int newfd);

       #define _GNU_SOURCE             /* See feature_test_macros(7) */
       #include <fcntl.h>              /* Definition of O_* constants */
       #include <unistd.h>

       int dup3(int oldfd, int newfd, int flags);
```

---

举例：

dup() :

```c
int fd = open("txt", O_RDWR);
int new_fd = dup(fd);
write(new_fd, "x", 1);
```

dup2() :

```c
int fd = open("txt", O_RDWR);
dup2(fd, STDOUT_FILENO);
printf("HELLO WORLD");
```

![image-20210821195326502](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/linux-c-基本函数/image-20210821195326502.1h80zd76196o.png)

### 描述

暂无

### 返回值

成功时，这些系统调用将返回新的文件描述符。

失败时，返回 -1， 设置 errno。

## fork - 创建一个子进程

参考：https://man7.org/linux/man-pages/man2/fork.2.html

### 函数原型

```c
#include <unistd.h>

       pid_t fork(void);
```

fork函数用来创建一个子进程。

![image-20210817145349164](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/linux-c-基本函数/image-20210817145349164.3vfbf8e1tka0.png)

---

举例：

```c
    printf("start fork()\n");
    pid_t pid = fork();

    if(pid == -1) {
        perror("fork error\n");
        return 0;
    } else if(pid == 0) {
        printf("child process\n");
    } else if(pid > 0) {
        printf("parent process\n");
    }
    printf("end\n");

/*
>>  start fork()
    parent process
    end
    child process
    end
*/
```

### 描述

fork() 函数通过重复调用进程创建一个新的进程，这个新的进程被称为子进程，调用 fork() 的进程被称为父进程。子进程和父进程在独立的内存空间中运行。在 fork() 时，两块内存空间具有相同的内容。

子进程与父进程完全相同，除了一下几点：

- 子进程有它独特的进程 ID
- 子进程的父进程ID与父进程ID相同。

等等

### 返回值

成功时，在父进程中，返回子进程的 PID；在子进程中，返回0。

失败时，父进程中返回 -1，并且没有子进程被创建。并且，errno 被设置。

## getpid, getppid - 获取进程标识

参考：https://man7.org/linux/man-pages/man2/getpid.2.html

### 函数原型

```c
#include <unistd.h>

       pid_t getpid(void);
       pid_t getppid(void);
```

### 描述

getpid() 返回该进程的 ID（PID）。

getppid() 返回该进程的父进程的 ID。

## exec系列 - 执行一个文件

参考：https://man7.org/linux/man-pages/man3/exec.3.html

### 函数原型

```c
#include <unistd.h>

       extern char **environ;

       int execl(const char *pathname, const char *arg, ...
                       /*, (char *) NULL */);
       int execlp(const char *file, const char *arg, ...
                       /*, (char *) NULL */);
       int execle(const char *pathname, const char *arg, ...
                       /*, (char *) NULL, char *const envp[] */);
       int execv(const char *pathname, char *const argv[]);
       int execvp(const char *file, char *const argv[]);
       int execvpe(const char *file, char *const argv[], char *const envp[]);
```

可以按照函数名的后缀对其进行分类：

#### l - execl(), execlp(), execle()

这三个函数有一个共同的参数 const char *arg，这个const char *arg和后面的省略号可以看成 arg0，arg1，arg2 ... ... argn。这些参数构成了执行程序的参数列表。第一个参数 const char *file/pathname，约定俗成，应该指向正在执行的文件的文件名。

后面的参数列表必须以空指针 NULL 结尾，用来告诉这个可变参数的函数这已经是最后一个参数了。

除此之外，参数列表的第一个参数 arg0 也必须是执行的文件名。拿execlp举例来说，execlp根据环境变量中的路径找到pathname所指向的文件名，然后执行该文件，执行该文件的 argv[0]，argv[1]，... ，argv[n] 与参数列表中 arg0，arg1，arg2 ... ... argn 相对应，而argv[0] 正是当前执行的文件名，也就是说，除了第一个参数 const char *file/pathname 要指定文件名，第二个参数 arg0 也需要指定文件名。

---

举例：

```c
execlp("ls", "ls", "-l", NULL);
execl("../forkProcess/out/fork", NULL);
// ----------------------------------------
execlp("date", "date", NULL);
execl("/usr/bin/date", "date", NULL);
execle("/usr/bin/date", "date", NULL, NULL);
```

#### v - execv(), execvp(), execvpe()

#### e - execle(), execvpe()

#### p - execlp(), execvp(), execvpe()

### 描述

exec() 系列函数用一个新的进程映像替换当前进程映像。也就是说当前进程的 PID 和其父进程都没有改变，而该进程所执行的东西已经变为了 exec 所执行的文件。

该系列的函数是在 execve 函数的基础上是实现的。execve 函数参考：https://man7.org/linux/man-pages/man2/execve.2.html

### 返回值

exec() 系列函数仅在发生错误时返回 -1，并设置 errno。

## wait, waitpid, waitid - 等待进程改变状态

参考：https://man7.org/linux/man-pages/man2/waitpid.2.html

### 函数原型

```c
#include <sys/wait.h>

       pid_t wait(int *wstatus);
       pid_t waitpid(pid_t pid, int *wstatus, int options);

       int waitid(idtype_t idtype, id_t id, siginfo_t *infop, int options);
                       /* 这是glibc和POSIX接口;有关原始系统调用的信息，请参阅NOTES。 */
```

### 描述

所有这些 **系统调用** 都是用来等待正在进行的子进程的状态发生变化的，并且获取到子进程状态发生变化的具体信息。

**状态改变** 可以被认为是：

- 子进程终止
- 子进程被一个信号中断了
- 子进程被一个信号恢复了

在子进程结束的情况下，父进程执行等待可以有效释放子进程的系统资源，如果没有执行等待，那么结束的子进程将以一个“僵尸”状态继续被保留。

如果一个子进程改变了状态，这些调用将立即返回，否则，它们会阻塞，直到子进程改变状态或信号处理程序中断调用。

#### wait 和 waitpid

wait() **系统调用** 将暂停调用线程的执行，直到它的一个子线程终止。

函数调用  *wait(int \*wstatus)*  等同于：

```c
waitpid(-1, &wstatus, 0);
```

waitpid **系统调用** 挂起当前调用线程，直到由 *pid* 参数指定的子进程发生变化。默认情况下，waitpid 只等待终止的子进程，但此行为可以通过修改参数 *options* 改变。

##### pid 参数

*pid* 的值可以是：

- < -1 　　　　意思是等待任何*进程组 ID 等于 pid 绝对值的子进程
- -1 　　　　　意思是任何子进程
- 0 　　　　　意思是等待任何进程组 ID 等于该调用 waitpid 的进程的进程组 ID 的子进程
- \> 0 　　　　意思是等待进程 ID 等于 pid 的子进程

##### options 参数

*options* 是一个组合值，包含 0 个或多个一下值：

WNOHANG： 如果没有孩子死亡则立即返回

WUNTRACED：

。。。

### 返回值

##　pipe, pipe2 - 创建管道（进程间通信）

参考： https://man7.org/linux/man-pages/man2/pipe.2.html

### 函数原型

```c
 #include <unistd.h>

       int pipe(int pipefd[2]);

#define _GNU_SOURCE             /* See feature_test_macros(7) */
#include <fcntl.h>              /* Definition of O_* constants */
#include <unistd.h>

       int pipe2(int pipefd[2], int flags);
```

---

举例：

```c
int fd[2];
int x = pipe(fd);    // 先
int pid = fork();    // 后

char buf[1024];
char * str = "hello world";
if(x == -1) {
    perror("error");
    exit(1);
}

if(pid > 0) {
    printf("i'm parent\n");
    write(fd[1], str, strlen(str));
} else if(pid == 0) {
    printf("i am child\n");
    int ret = read(fd[0], buf, 1024);
    write(STDOUT_FILENO, buf, ret);
}
```

### 描述

pipe() 创建一个 **单向** 的可以用于进程间通信的数据通道。 *pipefd[]* 用来返回两个分别指向管道入端和出端的文件描述符。  *pipefd[0]* 指的是管道的读端， *pipefd[1]* 为写端。写入写端的数据会被内核缓冲，直到从读端读取数据，写入的数据只能被读取一次，不能重复读取。  

### 返回值

成功时，返回 0

失败时，返回 -1，设置 errno，并且 *pipefd* 不会被改变。

## mmap, munmap - 映射或取消映射文件或设备到内存中（进程间通信）

参考： https://man7.org/linux/man-pages/man2/mmap.2.html

### 函数原型

```c
   #include <sys/mman.h>

   void *mmap(void *addr, size_t length, int prot, int flags,
              int fd, off_t offset);
   int munmap(void *addr, size_t length);
```

---

举例：





### 描述

mmap() 在调用进程的虚拟地址空间中创建一个新的映射。新映射的起始地址子在 *addr* 中指定。 *length* 指定映射的大小（必须大于0）。如果 *addr* 为NULL，那么内核选择(页面对齐的)地址来创建映射。这种方式创建的 mmap() 具有最好的可移植性。如果 *addr* 不为NULL，那么内核将其作为放置映射的位置的提示。 *offset* 必须是内存页（page size = 4096）的整数倍，可以通过 sysconf(_SC_PAGE_SIZE) 的返回值查看。

文件映射的内容，初始化时使用从文件描述符 *fd* 引用的文件(或其他对象)的偏移量 *offset* 处开始的 *length* 字节大小的映射空间。

![image-20210822022207629](https://cdn.jsdelivr.net/gh/yuhanOvo/image-hosting@master/文章/linux-c-基本函数/image-20210822022207629.41lugd2750u0.png)

#### prot 参数

*prot* 指定映射的内存保护权限。有下面几个宏：

- PROT_EXEC：映射的内存页可以执行
- PROT_READ：映射的内存页可以读
- PROT_WRITE：映射的内存页可以写
- PROT_NONE：映射的内存页不可访问

可以组合，比如 PROT_READ | PROT_WRITE 。

#### flag 参数

*flags* 参数确定对映射的更新是否对映射同一区域的其他进程可见，以及是否对底层文件进行更新。

此行为由包含以下值之一确定：

- MAP_SHARED： 分享这个映射。映射的更新对映射同一区域的其它进程是可见的。*并且(在文件支持映射的情况下)将被传递到底层文件。
- MAP_PRIVATE： 创建私有的 **写时拷贝** 映射。

还能由以下值的零个或多个确定：

- MAP_32BIT
- MAP_ANON

。。。

### 返回值

mmap() :

成功时，返回一个指向映射区域的指针。

失败时，值 MAP_FAILED （类型为 （void *）-1 ），并设置 errno。

munmap()：
