---
title: CSAPP-10 System Level IO
categories: [csapp]
tags: [csapp]
---

## 1 UNIX I/O

一个 Linux 文件就是一个 m 个字节的序列：B<sub>0</sub>, B<sub>1</sub>, …, B<sub>k</sub>, …, B<sub>m-1</sub>

所有的I/O设备（例如网络、磁盘和终端）都被模型化为文件，而所有的输人和输出都被当作对相应文件的读和写来执行。这种将设备优雅地映射为文件的方式，允许 Linux内核引出一个简单、低级的应用接口，称为UNIX I/O,这使得所有的输入和输出都能以一种统一且一致的方式来执行：

- 打开文件 ｜Linux shell 创建的每个进程开始都有3个打开的文件<unistd.h>，用来代替显式的描述符值
  - 标准输入（0）STDIN_FILENO
  - 标准输出（1）STDOUT_FILENO
  - 标准错误（2）STDERR_FILENO
- 改变当前文件位置
  - 对于每个打开的文件，内核保持着一个文件位置，初始为 0。这个文件位置是从文件开头起始的字节偏移量。
  - 应用程序能够通过执行 seek 操作，显式地设置文件的当前位置为。
- 读写文件
  - 一个读操作就是从文件复制 n > 0 个字节到内存，从当前文件位置 k 开始，然后将 k 增加到 k + n。给定一个大小为 m 字节的文件，当 k≥m 时执行读操作会触发一个称为 end-of-file (EOF) 的条件，应用程序能检测到这个条件。在文件结尾处并没有明确的“EOF 符号”。
  - 类似地，写操作就是从内存复制 n>0 个字节到一个文件，从当前文件位置 k 开始，然后更新 k。
- 关闭文件
  - 当应用完成了对文件的访问之后，它就通知内核关闭这个文件。作为响应，内核释放文件打开时创建的数据结构，并将这个描述符恢复到可用的描述符池中。无论一个进程因为何种原因终止时，内核都会关闭所有打开的文件并释放它们的内存资源。

## 2 文件

每个 Linux 文件都有一个类型（type）表明它在系统中的角色：

- **普通文件 - regular file**
- **目录 - directory**
- **套接字 - socket**
- 命名通道（named pipe）
- 符号链接（symbolic link）
- 字符和块设备（character and block device）
- ...

Linux 内核将所有文件都组织成一个目录层次结构（directory hierarchy），由名为/（斜杠）的根目录确定。系统中的每个文件都是根目录的直接或间接的后代。图 10-1 显示了 Linux 系统的目录层次结构的一部分。

## 3 打开/关闭文件

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>

// 返回：若成功则为新文件描述符，若出错为 -1。
int open(char *filename, int flags, mode_t mode);
```

open 函数将 filename 转换为一个文件描述符，并且返回描述符数字。返回的描述符总是在进程中当前没有打开的最小描述符。flags 参数指明了进程打算如何访问这个文件：

- ● O RDONLY：只读
- ● O WRONLY：只写
- ● O RDWR：可读可写

flags 参数也可以是一个或者更多位掩码的或，为写提供给一些额外的指示:

- ●O CREAT：如果文件不存在，就创建它的一个截断的（truncated)（空）文件。
- ●O TRUNC：如果文件已经存在，就截断它。
- ●O_APPEND：在每次写操作前，设置文件位置到文件的结尾处。

mode 参数指定了新文件的访问权限位。这些位的符号名字如图 10-2 所示。![10-2](../statics/csapp/10-2.png)

作为上下文的一部分，每个进程都有一个 umask，它是通过调用 umask 函数来设置的。当进程通过带某个 mode 参数的 open 函数调用来创建一个新文件时，文件的访问权限位被设置为 mode &~ umask。例如，假设我们给定下面的 mode 和 umask 默认值：

```c
#define DEF_MODE 	S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH
#define DEF_UMASK 	S_IWGRP|S_IWOTH
```

接下来，下面的代码片段创建一个新文件，文件的拥有者有读写权限，而所有其他的用户都有读权限：

```c
umask  (DEF_UMASK);
fd = open("foo.txt",O_CREAT|O_TRUNC|O_WRONLY, DEF_MODE);
```

最后，进程通过调用 close 函数关闭一个打开的文件。

```c
#include <unistd.h>
int close (int fd);
//返回：若成功则为 0，若出错则为 -1。
```

## 4 读和写文件

```c
#include <unistd.h>

/** 返回：若成功则为读的字节数，若 EOF 则为 0，若出错为 -1。
 * 	函数从描述符为 fd 的当前文件位置复制最多 n 个字节到内存位置 buf。
 *	返回值 -1 表示一个错误，而返回值 0 表示 EOF。否则，返回值表示的是实际传送的字节数量。
 * */
ssize_t read(int fd, void *buf, size_t n);

/**	返回：若成功则为写的字节数，若出错则为 -1。
 * 	函数以内存位置 buf 复制至多 n 个字节到描述符 fd 的当前文件位置。
 * 	图 l0-3 展示了一个程序使用 read 和 write 调用一次一个字节地从标准输入复制到标准输出。
 * */
ssize_t write(int fd, const void *buf, size_t n);

// 10-3
#include "csapp.h"
int main (void){
	char c;

	while (read(STDIN_FILENO, &c,1)! =0)
		write(STDOUT_FILENO, &c,1); 
	exit(O);
}	// 通过调用 1seek 函数，应用程序能够显示地修改当前文件的位置，这部分内容不在我门的讲述范围之内。
```

在某些情况下，read 和 write 传送的字节比应用程序要求的要少。这些不足值（short count）不表示有错误。出现这样情况的原因有：

> 实际上，除了 EOF，当你在读写磁盘文件时，不会遇到不足值。然而，如果你想创建健壮的（可靠的）诸如 Web 服务器这样的网络应用，就必须通过反复调用 read 和 write 处理不足值，直到所有需要的字节都传送完华。

- ● 读时遇到 EO。假设我们准备读一个文件，该文件从当前文件位置开始只含有 20 多个字节，而我们以 50 个字节的片进行读取。这样一来，下一个 read 返回的不足值为 20，此后的 read 将通过返回不足值 0 来发出 EOF 信号。
- ● 从终端读文本行。如果打开文件是与终端相关联的（如键盘和显示器），那么每个 read 函数将一次传送一个文本行，返回的不足值等于文本行的大小。
- ● 读和写网络套接字（socket）。如果打开的文件对应于网络套接字（11.4 节），那么内部缓冲约束和较长的网络延迟会引起 read 和 write 返回不足值。对 Linux 管道（pipe）调用 read 和 write 时，也有可能出现不足值，这种进程间通信机制不在我们讨论的范围之内。

### ssize_t 和 size_t 有些什么区别

在 x86-64 系统中，size_t 被定义为 unsigned long，而 ssize_t (有符号的大小）被定义为 long

read 函数返回一个有符号的大小，而不是一个无符号大小，这是因为出错时它必须返回 -1。有趣的是，返回一个 -1 的可能性使得 read 的最大值减小了一半。

## 5 用 RIO 包健壮地读写

Robust I/O ：会自动为你处理上文中所述的不足值。在像网络程序这样容易出现不足值的应用中，RIO 包提供了方便、健壮和高效的I/O。RIO提供了两类不同的函数：

- ● 无缓冲的输入输出函数，这些函数直接在内存和文件之间传送数据，没有应用级缓冲。它们对将二进制数据读写到网络和从网络读写二进制数据尤其有用。

- ● 带缓冲的输入函数。这些函数允许你高效地从文件中读取文本行和二进制数据，这些文件的内容缓存在应用级缓冲区内，类似于为 printf 这样的标准 I/O 函数提供的缓冲区。与[110]中讲述的带缓冲的I/O例程不同，带缓冲的RIO输入函数是线程安全的（12.7.1 节），它在同一个描述符上可以被交错地调用。

  > 例如，你可以从一个描述符中读一些文本行，然后读取一些二进制数据，接着再多读取一些文本行。

我们讲述 RIO 例程有两个原因

- 第一，在接下来的两章中，我们开发的网络应用中使用 了它们；
- 第二，通过学习这些例程的代码，你将从总体上对 UNIX I/O 有更深入的了解。

### RIO 的无缓冲输入输出函数

通过调用 rio_readn 和 rio_writen 函数，应用程序可以在内存和文件之间直接传送数据

```c
#include "csapp.h"

ssize_t rio_readn(int fd, void *usrbuf, size_t n);
ssize_t rio_writen(int fd, void *usrbuf, size_t n);
```

rio_readn 函数从描述符 fd 的当前文件位置最多传送 n 个字节到内存位置 usrbuf。类似地，rio_writen 函数从位置 usrbuf 传送 n 个字节到描述符 fd。rio_read 函数在遇到 EOF 时只能返回一个不足值。rio_wr1ten 函数决不会返回不足值。对同一个描述符，可以任意交错地调用 rio_readn 和 rio_writen.

图 10-4 显示了 rio_readn 和 rio_writen 的代码。注意，如果 rio_readn 和 rio_writen 函数被一个从应用信号处理程序的返回中断，那么每个函数都会手动地重启 read 或 write。为了尽可能有较好的可移植性，我们允许被中断的系统调用，且在必要时重启它们。

```c
ssize_t rio_readnb(int fd, void *usrbuf, size_t n)
{
	size_t nleft = n;
	ssize_t nread;
	char *bufp = usrbuf;

	while (nleft > 0) {
		if ((nread = read(fd, bufp, nleft)) < 0)
		{
			if (errno == EINTR)
				nread = 0;
			 else 
				return -1;
		} else if (nread == 0)
		{
			break;
		}
		nleft -= nread;
		bufp += nread;
	}
	return (n - nleft);
}

ssize_t rio_writen(int fd, void *usrbuf, size_t n)
{
	size_t nleft = n;
	ssize_t nwritten;
	char *bufp = usrbuf;

	while (nleft > 0) {
		if ((nwritten = write(fd, bufp, nleft)) <= 0)
		{
			if (errno == EINTR)		/* Interrupted by sig handler return */
				nwritten = 0;		/* and call write() again*/
			else
				return -1;			/* errno set by write() */
		}
		nleft -= nwritten;
		bufp += nwritten;
	}
	return n;
}
```



### RIO 的带缓冲的输入函数

假设我们要编写一个程序来计算文本文件中文本行的数量，该如何来实现呢？

一种方法就是用 read 函数来一次一个字节地从文件传送到用户内存，检查每个字节来查找换行符。这个方法的缺点是效率不是很高，每读取文件中的一个字节都要求陷人内核。

一种更好的方法是调用一个包装函数（rio_readlineb），它从一个内部读缓冲区复制一个文本行，当缓冲区变空时，会自动地调用 read 重新填满缓冲区。对于既包含文本行也包含二进制数据的文件（例如 11.5.3 节中描述的 HTTP 响应），我们也提供了一个 rio_readnb 带缓冲区的版本，叫做 rio_readnb，它从和 rio_readlineb 一样的读缓冲区中传送原始字节。

```c
#include "csapp.h"

void rio_readinitb(rio_t *rp, int fd);

ssize_t rio_readlineb(rio_t *rp, void *usrbuf, size_t maxlen);
ssize_t rio_readnb(rio_t *rp, void *usrbuf, size_t n);
```

每打开一个描述符，都会调用一次 rio_readinitb 函数。它将描述符 fd 和地址 rp 处的一个类型为 rio_t 的读缓冲区联系起来。

rio_readlineb 函数从文件 rp 读出下一个文本行（包括结尾的换行符），将它复制到内存位置 usrbuf，并且用 NULL（零）字符来结束这个文本行。rio_readlineb 函数最多读 maxlen-1 个字节，余下的一个字符留给结尾的 NULL 字符。超过 maxlen-1 字节的文本行被截断，并用一个 NULL 字符结束。

rio_readnb 函数从文件 rp 最多读 n 个字节到内存位置 usrbuf。对同一描述符，对 rio_readlineb 和 rio_readnb 的调用可以任意交叉进行。然而，对这些带缓冲的函数的调用却不应和无缓冲的 rio_readn 函数交又使用。

在本书剩下的部分中将给出大量的 RIO 函数的示例。

```c
#include "csapp.h"

// 展示了如何使用 RIO 函数来一次一行地从标准输人复制一个文本文件到标准输出。
int main(int argc, char const *argv[])
{
	int n;
	rio_t rio;
	char buf[MAXLINE];

	rio_readinitb(&rio, STDIN_FILENO);
	while ((n = rio_readinitb(&rio, buf, MAXLINE)) != 0)
		rio_written(STDOUT_FILENO, buf, n);
	return 0;
}

// 展示了一个读缓冲区的格式，以及初始化它的 rio_readinitb 函数的代码。rio_readinitb 函数创建了一个空的读缓冲区，并且将一个打开的文件描述符和这个缓冲区联系起来。
typedef struct {
	int rio_fd;
	int rio_cnt;
	char *rio_bufptr;
	char rio_buf[RIO_BUFSIZE];
} rio_t;

void rio_readinitb(rio_t *rp, int fd) {
	rp->rio_fd = fd;
	rp->rio_cnt = 0;
	rp->rio_bufptr = rp->rio_buf;
}
```

RIO 读程序的核心是图 10-7 所示的 rio_read 函数。rio_read 函数是 Linux read 函数的带缓冲的版本。当调用 rio_read 要求读 n 个字节时，读缓冲区内有 rp-> rio_cnt 个未读字节。如果缓冲区为空，那么会通过调用 read 再填满它。这个 read 调用收到一个不足值并不是错误，只不过读缓冲区是填充了一部分。一旦缓冲区非空，rio read 就从读缓冲区复制 n 和 rP-> rio cnt 中较小值个字节到用户缓冲区，并返回复制的字节数。

```c
static ssize_t rio_read(rio_t *rp, char *usrbuf, size_t n)
{
	int cnt;

	while (rp->rio_cnt <= 0) {
		rp->rio_cnt = read(rp->rio_fd, rp->rio_buf,sizeof(ro->rio_buf));
		if (rp->rio_cnt < 0) {
			if (errno != EINTR) {
				return -1;
			}
		} else if (rp->rio_cnt == 0) {
			return
		} else {
			rp->rio_bufptr = rp->rio_buf;
		}
	}

	cnt = n;
	if (rp->rio_cnt < n)
		cnt = rp->rio_cnt;
	
	memcpy(usrbuf, rp->rio_bufptr, cnt);
	rp->rio_bufptr += cnt;
	rp->rio_cnt -= cnt;
	return cnt
}
```

对于一个应用程序，rio_read 函数和 Linux read 函数有同样的语义。在出错时，它返回值 -1，并且适当地设置 errno。在 EOF 时，它返回值 0。如果要求的字节数超过了读缓冲区内未读的字节的数量，它会返回一个不足值。两个函数的相似性使得很容易通过用 rio read 代替 read 来创建不同类型的带缓冲的读函数。例如，用 rio read 代替 read，图 l0-8 中的 rio readnb 函数和 rio readn 有相同的结构。相似地，图 l0-8 中的 rio readlineb 程序最多调用 maxlen-l 次 rio read。每次调用都从读缓冲区返回一个字节，然后检查这个字节是否是结尾的换行符。



### RIO 包的来源

RIO 函数的灵感来自于 W. Richard Stevens 在他的经典网络编程作品中描述的 readline、readn 和 writen 函数。rio readn 和 rio writen 函数与 Stevens 的 readn 和 writen 函数是一样的。然而，Stevens 的 readline 函数有一些局限性在 RIO 中得到了纠正。第一）因为 readline 是带缓冲的，而 readn 不带，所以这两个函数不能在同一描述符上一起使用。（第二因为它使用一个 static 缓冲区，Stevens 的 readline 函数不是线程安全的，这就要求 Stevens 引入一个不同的线程安全的版本，称为 read- line_r。我们已经在 rio readlineb 和 rio readnb 函数中修改了这两个缺陷，使得这两个函数是相互兼容和线程安全的。

```c

ssize_t rio_readlineb(rio_t *rp, void *usrbuf, size_t maxlen)
{
	int n, rc;
	char c, *buf = usrbuf;

	for (n = 1; n < maxlen; n++) {
		if ((rc = rio_read(rp, &c, 1)) == 1) {
			*bufp++ = c;
			if (c == '\n')
			{
				n++;
				break;
			}
		} else if (rc == 0) {
			if (n == 1)
				return 0;
			else
				break;
		} else {
			return -1
		}
	}
	*bufp = 0;
	return n-1;
}

ssize_t rio_readnb(rio_t *rp, void *usrbuf, size_t n) {
	size_t nleft = n;
	ssize_t nread;
	char *bufp = usrbuf;

	while (nleft > 0) {
		if ((nread = rio_read(rp, bufp, nleft)) < 0)
			return -1;
		 else if (nread == 0)
			break;
		nleft -= nread;
		bufp += nread;
	}
	return (n - nleft);
}
```



## 6 读取文件元数据

应用程序能够通过调用 stat 和 fstat 函数，检索到关于文件的信息（有时也称为文件的元数据（metadata））。

```c
#include <unistd.h>
#include <sys/stat.h>

int stat(const char *filename, struct stat *buf);
int fstat(int fd, struct stat *buf);
//返回：若成功则为 0，若出错则为 -1
```

stat 函数以一个文件名作为输人，并填写如图 I0-9 所示的一个 stat 数据结构中的各个成员。fstat 函数是相似的，只不过是以文件描述符而不是文件名作为输入。当我们

在 11.5 节中讨论 Web 服务器时，会需要 stat 数据结构中的 st_mode 和 st_size 成员，其他成员则不在我们的讨论之列。

```c
/* Metadata returned by the stat and fstat functions */
struct stat {
    dev_t           st_dev;     // Device
    ino_t           st_ino;     // inode
    mode_t          st_mode;    // Protection and file type
    nlink_t         st_nlink;   // Number of hard links
    uid_t           st_uid;     // User ID of owner
    gid_t           st_gid;     // Group ID of owner
    dev_t           st_rdev;    // Device Type (if inode device)
    off_t           st_size;    // Total size, in bytes
    unsigned long   st_blksize; // Block size for filesystem I/O
    unsigned long   st_blocks;  // Number of blocks allocated
    time_t          st_atime;   // Time of last access
    time_t          st_mtime;   // Time of last modification
    time_t          st_ctime;   // Time of last change
};
```

st_size 成员包含了文件的字节数大小。st_mode 成员则编码了文件访问许可位 (图 10-2) 和文件类型 (10.2节)。Linux 在sys/stat.h 中定义了宏谓词来确定 st_mode 成员的文件类型：

- S_ISREG(m)。这是一个普通文件吗？

- S_ISDIR(m)。这是一个目录文件吗？

- S_ISSOCK(m)。这是一个网络套接字吗？


图 10-l0 展示了我们会如何使用这些宏和 stat 函数来读取和解释一个文件的 st_mode 位。

```c
#include "csapp.h"
// 展示了我们会如何使用这些宏和 stat 函数来读取和解释一个文件的 st_mode 位。
int main(int argc, char const *argv[])
{
    struct stat stat;
    char *type, *readok;
    
    stat(argv[1], &stat);
    if (S_ISREG(stat.st_mode))
        type = "regular";
    else if (S_ISDIR(stat.st_mode))
        type = "directory";
    else
        type = "other";
    if ((stat.st_mode & S_IRUSR))
        readok = "yes";
    else
        readok = "no";
    
    printf("type: %s, read: %s\n", type, readok);
    
    exit(0);
}
```



## 7 读取目录和内容

## 8 共享文件

## 9 I/O 重定向

## 10 标准 I/O

## 11 综合：我该使用哪些 I/O 函数
