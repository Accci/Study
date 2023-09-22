### MIT lab

#### 系统调用

- `int fork()` 创建一个进程，返回子进程的 PID.
- `int exit(int status)` 终止当前进程，将状态报告给 wait ，没有返回值。
- `int wait(int *status)` 等待子进程退出，`*status` 表示退出状态，返回子进程的 PID 。
- `int kill(int pid)` 终止进程PID。返回0，或-1为错误。
- `int getpid()` 返回当前进程的PID。
- `int sleep(int n)` 暂停n个时钟。
- `int exec(char *file, char *argv[])` 加载一个文件并带着参数执行，仅在出错时返回。
- `char *sbrk(int n)` 为进程内存扩大 n 个字节，返回新内存的起点。
- `int open(char *file, int flags)` 打开一个文件；flags 表示读/写；返回一个fd（文件描述符）。
- `int write(int fd, char *buf, int n)` 从buf写n个字节到文件描述符fd；返回n。
- `int read(int fd, char *buf, int n)` 向 buf 中读取n个字节；返回读取的数量；如果文件结束则返回 0 。
- `int close(int fd)` 释放打开的文件fd。
- `int dup(int fd)` 返回一个新的文件描述符，指的是与 fd 相同的文件。
- `int pipe(int p[])` 创建一个管道，把读/写文件描述符放在p[0]和p[1]中。
- `int chdir(char *dir)` 改变当前目录。
- `int mkdir(char *dir)` 创建一个新的目录。
- `int mknod(char *file, int, int)` 创建一个设备文件。
- `int fstat(int fd, struct stat *st)` 将打开的文件的信息放入`*st`。
- `int stat(char *file, struct stat *st)` 将一个命名的文件的信息放入`*st`。
- `int link(char *file1, char *file2)` 为文件file1创建另一个名字（file2）。
- `int unlink(char *file)` 删除一个文件。

#### 进程和内存

- fork

  - 在父进程中， fork返回新进程的pid，
  - 在子进程中。fork返回0

- cat < input.txt

  ```c
  char *argv[2];
  
  argc[0] = "cat";
  argc[1] = 0;
  if(fork() == 0){
      close(0);
      open("input.txt", O_RDONLY);
      exec("cat", argv);
  }
  ```

- `dirent` 结构体的实现是特定于操作系统的，因为不同的操作系统可能有不同的目录项结构。面是一个常的 `dirent` 结构体的示：

  ```cpp
  struct dirent {
      ino_t d_ino;               // inode number
   	off_t d_off;               // offset to the next dirent
      unsigned short d_reclen;   // length of this record
      unsigned char d_type;      // type of file
      char d_name[256];          // filename
  };
  ```

  这是一个典型的 `dirent` 结构体，包含了以下字段：

  - `d_ino`：inode 号码，用于唯一标文件或目录。
  - `d_off`：下一个 `dirent` 的偏移量。
  - `d_reclen`：当前记录的长度。
  - `d_type`：文件类型可以是常量 `DT_REG`（普通文件）、`DT_DIR`（目录）等。
  - `d_name`：文件或目录的名称。

#### lab1

##### **cat**

```c
char buf[32];
int n;
for(;;)
{
    n = read(0, buf, sizeof buf);
    if(n == 0)
        break;
    if(n < 0)
    {
        fprintf(2, "read error\n");
        exit(1);
    }
    if(write(1, buf, n) != n)
    {
        fprintf(2, "write error\n");
        exit(1);
    }
}
```

##### wc

```c
int p[2];
char*argv[2];
argv[0] = "wc";
argv[1] = 0;
pipe(p);

if(fork() == 0)
{
    close(0);
    dup(p[0]);
    close(p[0]);
    close(p[1]);
    exec("bin/wc", argv);
}else
{
    close(p[0]);
    write(p[1], "hello world!\n",12);
    close(p[1]);
    
}
```

##### 文件系统

```c
#define T_DTR 1
#define T_FILE 2
#define T_DEViCE 3

strcut stat
{
    int dev;		//
    uint ino;		//节点号
    short type;		//类型
    short nlink;
    uint64 size;
}

open("a", O_CREATE|O_WRONLY);
link("a", "b");
```



##### sleep.c

```c
int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(2, "input error!\n");
        exit(1);
    }else {
        int k = atoi(argv[1]); // 字符串转正序
        sleep(k);
    }
    exit(0);
}
```

##### pingpong.c   //父子进程通信

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

int
main(int argc, char *argv[]) {
    int p2c[2], c2p[2];
    pipe(p2c); pipe(c2p);
    char buf[64];

    if (fork() > 0) {
        write(p2c[1], "ping", 4); // 父进程给子进程中写入 ping
        read(c2p[0], buf, 4); // 父进程读取子进程的消息
        printf("%d: received %s\n", getpid(), buf);  // 打印
    }else {
        read(p2c[0], buf, 4); // 子进程读取父进程的消息
        printf("%d: received %s\n", getpid(), buf);
        write(c2p[1], "pong", 4); // 子进程给父进程写消息
    }
    exit(0);
}
```

##### primes 素数

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"

void prime(int rd){
    int n;
    read(rd, &n, 4); // 从rd中读数据，存储在 n 上
    printf("prime %d\n", n);
    int created = 0;
    int p[2];
    int num;
    while(read(rd, &num, 4) != 0){
        if(created == 0){
            pipe(p);
            created = 1;
            int pid = fork();
            if(pid == 0){
                close(p[1]);
                prime(p[0]);
                return;
            }else{
                close(p[0]);
            }
        }
        if(num % n != 0){
            write(p[1], &num, 4);  // 将num 上的数据写入管道中
        }
    }
    close(rd);
    close(p[1]);
    wait(0);
}

int main(int argc, char *argv[]){
    int p[2];
    pipe(p);
    // 父进程
    if(fork() != 0){
        close(p[0]);
        for(int i = 2; i <= 35; i++){
            // 第一个参数是文件描述符
            // 第二个参数是数据的指针
            // 第三个参数是要写入的字节数
            write(p[1], &i, 4);
        }
        close(p[1]);
        wait(0);
    }else{
        // 子进程
        close(p[1]);
        prime(p[0]);
        close(p[0]);
    }
    exit(0);
}
```

##### find.c

```c
#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/fs.h"

char* fmtname(char * path)
{
    static char buf[DIRSIZ+1];
    char *p;
    
    for (p = path + strlen(path); p >= path && *p != '/'; p--); //从后往前
    
    
    p ++;
    if (strlen(p) >= DIRSIZ) return p;
    memmove(buf, p, strlen(p));
    buf[strlen(p)] = 0;
    return buf;
}

void find(char *path,char *name)
{
  char buf[512], *p;
  int fd;
  struct dirent de;  
/*
struct dirent {
  ushort inum;
  char name[DIRSIZ];
};
*/
  struct stat st;
    
/*
#define T_DIR     1   // Directory
#define T_FILE    2   // File
#define T_DEVICE  3   // Device

struct stat {
  int dev;     // File system's disk device
  uint ino;    // Inode number
  short type;  // Type of file
  short nlink; // Number of links to file
  uint64 size; // Size of file in bytes
};
*/

  if((fd = open(path, 0)) < 0){
    fprintf(2, "ls: cannot open %s\n", path);
    return;
  }

  if(fstat(fd, &st) < 0){
    /*
    fstat(fd, &st) 是一个函数调用，用于获取文件描述符 fd 对应文件的相关信息，并将结果存储在结	构体 st 中。

具体解释如下：

fstat: 是一个系统调用函数，用于获取文件状态信息。
fd: 是一个整数值，表示文件描述符。文件描述符是操作系统为每个打开的文件分配的唯一标识符。
&st: 是一个指向 struct stat 类型的指针，用于接收文件状态信息。struct stat 是一个包含了文件的各种属性的结构体。
通过调用 fstat 函数，可以获取文件的各种属性，例如文件大小、访问权限、创建时间等。这些信息可以在 struct stat 结构体中找到，并通过 st 指针进行访问和使用
    */
    fprintf(2, "ls: cannot stat %s\n", path);
    close(fd);
    return;
  }

  switch(st.type){
  case T_FILE:
    if (strcmp(fmtname(path), name) == 0) printf("%s\n", path);
     /*
     int strcmp(const char *str1, const char *str2);
	该函数接受两个参数，别是要比较的两个字符串的指。函数返回一个整数值，表示两个字符串的大小关系。

	strcmp函数的比较规则下：

    如果str1和str2相等，返回值为0。
    如果str1大于str2，返回值大于0。
    如果str1小于str2，返回值小于0。
     */
    break;

  case T_DIR:
    if(strlen(path) + 1 + DIRSIZ + 1 > sizeof buf){
      printf("ls: path too long\n");
      break;
    }
    strcpy(buf, path);
    p = buf+strlen(buf); //位移到最后一位
    *p++ = '/';
    while(read(fd, &de, sizeof(de)) == sizeof(de)){
      if(de.inum == 0)
        continue;
      memmove(p, de.name, DIRSIZ);
      /*
        使用 memmove 将 source 中的数据复制到 destination
         memmove(destination, source, strlen(source) + 1);
      */  
      p[DIRSIZ] = 0;
      // 忽略当前目录和父目录
      if (!strcmp(de.name, ".") || !strcmp(de.name, "..")) continue;
      find(buf, name);
    }
    break;
  }
  close(fd);
}

int
main(int argc, char *argv[])
{
    if (argc != 3) {
        fprintf(2, "usage: find [path] [patten]\n");
        exit(1);
    }
    find(argv[1], argv[2]);
    exit(0);
}
```

##### xargs.c

```c
// Lab Xv6 and Unix utilities
// xargs.c

#include "kernel/types.h"
#include "kernel/stat.h"
#include "user/user.h"
#include "kernel/param.h"

#define MAXN 1024

int
main(int argc, char *argv[])
{
    // 如果参数个数小于 2
    if (argc < 2) {
        // 打印参数错误提示
        fprintf(2, "usage: xargs command\n");
        // 异常退出
        exit(1);
    }
    // 存放子进程 exec 的参数
    char * argvs[MAXARG];
    // 索引
    int index = 0;
    // 略去 xargs ，用来保存命令行参数
    for (int i = 1; i < argc; ++i) {
        argvs[index++] = argv[i];
    }
    // 缓冲区存放从管道读出的数据
    char buf[MAXN] = {"\0"};
    
    int n;
	// 0 代表的是管道的 0，也就是从管道循环读取数据
    while((n = read(0, buf, MAXN)) > 0 ) {
        // 临时缓冲区存放追加的参数
		char temp[MAXN] = {"\0"};
        // xargs 命令的参数后面再追加参数
        argvs[index] = temp;
        // 内循环获取追加的参数并创建子进程执行命令
        for(int i = 0; i < strlen(buf); ++i) {
            // 读取单个输入行，当遇到换行符时，创建子线程
            if(buf[i] == '\n') {
                // 创建子线程执行命令
                if (fork() == 0) {
                    exec(argv[1], argvs);
                }
                // 等待子线程执行完毕
                wait(0);
            } else {
                // 否则，读取管道的输出作为输入
                temp[i] = buf[i];
            }
        }
    }
    // 正常退出
    exit(0);
}
```

#### lab2

##### 为了实现强隔离

- 将资源抽象，应用程序只能通过open、read、write、close等系统调用与文件进行交互，而不是直接写磁盘
- 使用exec建立内存映像
- 文件描述符
- 硬件支持
  - 三种模式
    - 机器模式
    - 监督者：可以执行特权指令 内核模式
    - 用户模式
  - 使用页表为每个进程提供自己的地址空间
  - 为每个进程维护了许多状态，proc结构中
    - p->state 表示进程的创建，就绪、运行、睡眠、还是僵尸
    - p->pagetable 保存进程页表

****

```c
$ git fetch
$ git checkout syscall
$ make clean

```

##### trace

- 在***Makefile\***的**UPROGS**中添加`$U/_trace`

- 将系统调用的原型添加到**user/user.h**，存根添加到**user/usys.pl**，以及将系统调用编号添加到**kernel/syscall.h**

  - `int trace(int);`  ---  user.h
  - `entry("trace");` ---- usys.pl
  - `#define SYS_trace 22` ---- kernel/syscall.h

- 在**kernel/sysproc.c**中添加一个`sys_trace()`函数，它通过将参数保存到`proc`结构体（请参见***kernel/proc.h**）里的一个新变量中来实现新的系统调用。从用户空间检索系统调用参数的函数在**kernel/syscall.c**中，您可以在**kernel/sysproc.c**中看到它们的使用示例。

  ```c
  //proc.h:
  struct proc{
      //...
      int trace_mask; // 用作记录掩码
  };
  
  //sysproc.c
  uint64
  sys_trace(void)
  {
      argint(0, &(myproc()->trace_mask)); // myproc 表示当前进程
      return 0;
  }
  
  //syscall.c
  extern uint64 sys_trace(void);
  static uint64 (*syscalls[])(void) = {
  //...
  [SYS_trace]   sys_trace,
  };
  
  //因为要打印系统调用的名称， 但是缺少系统调用号和名称之间的映射，需要建立二者的映射
  static char *syscalls_name[]= {
  [SYS_fork]   "fork",
  [SYS_exit]    "exit",
  [SYS_wait]    "wait",
  [SYS_pipe]    "pipe",
  [SYS_read]    "read",
  [SYS_kill]    "kill",
  [SYS_exec]    "exec",
  [SYS_fstat]   "fstat",
  [SYS_chdir]   "chdir",
  [SYS_dup]     "dup",
  [SYS_getpid]  "getpid",
  [SYS_sbrk]    "sbrk",
  [SYS_sleep]   "sleep",
  [SYS_uptime]  "uptime",
  [SYS_open]    "open",
  [SYS_write]   "write",
  [SYS_mknod]   "mknod",
  [SYS_unlink]  "unlink",
  [SYS_link]    "link",
  [SYS_mkdir]   "mkdir",
  [SYS_close]   "close",
  [SYS_trace]   "trace",
  };
  
  void
  syscall(void)
  {
    int num; // 声明一个整型变量num，用于存储系统调用号
    struct proc *p = myproc(); //myproc()是一个辅助函数，它返回当前进程的指针
  
    num = p->trapframe->a7; //从当前进程的上下文中获取系统调用号,它存储在trapframe结构的a7寄存器中
      
      // 用于检查系统调用号是否有效，并且对应的系统调用处理函数是否存在。
      //NELEM(syscalls)是一个常量，表示系统调用数组syscalls的长度。
    if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
        
      p->trapframe->a0 = syscalls[num](); //如果系统调用号有效，则调用syscalls数组中对应位置的系统调用处理函数,并将其返回值存储在a0寄存器中
        
        //trace_mask是一个位掩码，每一位代表一种系统调用。如果对应的位为1，表示跟踪该系统调用。
      if ((p->trace_mask & (1 << num)) != 0) {                                 
          printf("%d: syscall %s -> %d \n", p->pid, syscall_names[num], p->trapframe->a0);//如果当前系统调用被跟踪，则输出跟踪信息，包括进程ID、系统调用名称以及系统调用返回值。
      }
        
    } else {
      printf("%d %s: unknown sys call %d\n",
              p->pid, p->name, num);
      p->trapframe->a0 = -1;
    }
  }
  ```

- 修改`fork()`（请参阅***kernel/proc.c\***）将跟踪掩码从父进程复制到子进程。注意修改 kernel/proc.c 中的 `freeproc()` 函数，将 `p->trace_mask = 0;` 释放进程的时候要重置相应内容。

  ```c
  int fork(void){
      //...
      // 将trace_mask 拷贝到子进程hh
      //...
  }
  ```

##### Sysinfo

```c
//kernel/sysinfo.h
struct sysinfo {
  uint64 freemem;   // amount of free memory (bytes)取空闲内存量
  uint64 nproc;     // number of process  进程的数量
};
```



- `$U/_sysinfotest` 添加到 Makefile 的 UPROGS 中

- - `int sysinfo(struct sysinfo*);`  ---  user.h
  - `entry("sysinfo");` ---- usys.pl
  - `#define SYS_sysinfo 23` ---- kernel/syscall.h

- 在*kernel/kalloc.c*中添加一个函数用于获取空闲内存量

  ```c
  struct run {
    struct run *next;
  };
  
  struct {
    struct spinlock lock;
    struct run *freelist;
  } kmem;
  
  //内存是使用链表进行管理的，因此遍历kmem中的空闲链表就能够获取所有的空闲内存，如下
  
  void
  freebytes(uint64 *dst)
  {
    *dst = 0;
    struct run *p = kmem.freelist; // 用于遍历
  
    acquire(&kmem.lock);
    while (p) {
      *dst += PGSIZE;
      p = p->next;
    }
    release(&kmem.lock);
  }
  ```

- 在*kernel/proc.c*中添加一个函数获取进程数

```c
//遍历proc数组，统计处于活动状态的进程即可，循环的写法参考scheduler函数

void
procnum(uint64 *dst)
{
  *dst = 0;
  struct proc *p;
  for (p = proc; p < &proc[NPROC]; p++) {
    if (p->state != UNUSED)
      (*dst)++;
  }
}
```

- 将前两个函数在defs.h添加声明

  ```c
  //kalloc.c
  void freebytes(uint64 *);
  
  //proc.c
  void procnum(uint64 *);
  ```

  

- 在*kernel/sysproc.c* 实现`sys_sysinfo`

```c
#include "sysinfo.h"
uint64
sys_sysinfo(void)
{
  struct sysinfo info;
    //freebytes(&info.freemem);: 这是一个函数调用，用于获取系统当前的可用内存量，并将其存储在info结构体的freemem成员中。
  freebytes(&info.freemem);
    
    //procnum(&info.nproc);: 这是一个函数调用，用于获取当前系统运行的进程数，并将其存储在info结构体的nproc成员中。
  procnum(&info.nproc);

  // 获取虚拟地址
  //这两行代码用于获取用户传递给该系统调用的第一个参数的虚拟地址，并将其存储在变量dstaddr中。
  uint64 dstaddr;
  argaddr(0, &dstaddr);

  // 从内核空间拷贝数据到用户空间
    //copyout函数的第一个参数是当前进程的页表，第二个参数是目标虚拟地址（用户空间的地址），第三个参数是源数据的虚拟地址（内核空间的地址），第四个参数是要拷贝的数据的大小
  if (copyout(myproc()->pagetable, dstaddr, (char *)&info, sizeof info) < 0)
    return -1;

  return 0;
}
```

- 在*kernel/syscall.c* 中添加映射

  ```c
  extern uint64 sys_sysinfo(void);
  static uint64 (*syscalls[])(void) = {
  //...
  [SYS_sysinfo]   sys_sysinfo,
  };
  
  static char *syscalls_name[]= {
  //...
  [SYS_sysinfo]   "sysinfo",
  };
  
  ```


#### lab3

##### Speed up system calls

- 操作系统通过在用户空间和内核之间的共享只读区中的数据来加速某些系统调用。这种做法可以消除用户态到内核态之间切换的消耗。这次实验就使让我们学会将映射插入[页表](https://so.csdn.net/so/search?q=页表&spm=1001.2101.3001.7020)中，我们的第一个任务就是实现对系统调用`getpid()`的优化。
- 所以第一步就是在结构体`struct proc`中加入一个成员`struct usyscall* ucall`

```c
//kernel/proc,h
struct proc{
    //..
    struct usyscall *ucall;
}
```

- 函数`proc_pagetable(struct proc* p)`中将USYSCALL映射到页表中

  ```c
  // kermel/proc.c
  pagetable_t proc_pagetable(struct proc* p)
  {
      //不加 PTE_U权限，PTE只能在特权模式下使用。
      if(mappages(pagetable. USYSCALL, PGSIZE,(uint64)(p->ucall), PTE_U | PTE_R) < 0)
      {
          uvmfree(pagetable,0);
          return 0;
      }
      return pagetable;
  }
  ```

  在映射USYSCALL之前，还没有为成员`ucall`分配内存，在函数`allocproc()`中，需要对该成员分配内存，也是在该函数中对proc的成员分配好内存后，才调用`proc_pagetable()`函数映射的。

  ```c
  /*		kernel/proc.c		*/
  static struct proc*
  allocproc(void)
  {
      ...
      ...
    if (0 == (p->ucall = (struct usyscall*)kalloc()))
    {
  	  freeproc(p);
  	  release(&p->lock);
  	  return 0;
    }
  
    // An empty user page table.
    p->pagetable = proc_pagetable(p);
     // ...
          
    p->ucall->pid = p->pid;
    return p;
  }
  
  ```

3.对内存进行释放

```c
/*		kernel/proc.c		*/
static void
freeproc(struct proc *p)
{
  ...
  if (p->ucall)
	  kfree((void*)p->ucall);
  p->ucall = 0;
  ...
  p->state = UNUSED;
}

```

释放进程的页表，取消映射

```c
/*		kernel/proc.c		*/
void
proc_freepagetable(pagetable_t pagetable, uint64 sz)
{
  uvmunmap(pagetable, TRAMPOLINE, 1, 0);
  uvmunmap(pagetable, TRAPFRAME, 1, 0);
  uvmunmap(pagetable, USYSCALL, 1, 0);
  uvmfree(pagetable, sz);
}

```

##### Print a page table

```c
/*		kernel/exec.c		*/
int exec(char* path, char** argv)
{
    ...
    ...
    if(1 == p->pid) 
    {
        printf("page table %p\n",p->pagetable);
        vmprint(p->pagetable,1);
    }
    return argc;
    ...
    ...
}

```



```c
/*		kernel/vm.c		*/
void vmprint(pagetable_t pagetable, int level)
{
	static char* level1 = "..";
	static char* level2 = ".. ..";
	static char* level3 = ".. .. ..";
	char* buf = 0;
	if (1 == level)
		buf = level1;
	else if (2 == level)
		buf =  level2;
	else if (3 == level)
		buf = level3;
	for (int i = 0;i < 512;i++)
	{
		pte_t pte = pagetable[i];
		if ((pte & PTE_V) && level <= 3)
		{
			uint64 child = PTE2PA(pte);
			printf("%s%d: pte %p pa %p\n",buf,i,pte,child);
			vmprint((pagetable_t)child,level + 1);
		}
	}
}

```

##### Detecting which pages have been accessed

```c
//对进程中的分配的内存进行修改后，RISC-V的page walker会对该内存所在的PTE表项的FLAG进行修改，因此，FLAG会变。
//这个实验的主要任务就是，添加一个标志位PTE_A，然后获取一个页表中被访问的那一项（PTE），被访问的PTE中的FLAG中的标志位A会被硬件置位。
//最后返回所有被访问的PTE。

```

```c
/*		riscv.h			*/
#define 	PTE_A	(1L << 6)

```

```c
/*		kernel/sysproc.c		*/
int sys_pgaccess(void)
{
	uint64 start_addr;	// 测试程序中buf的地址
	int amount;			// ........32
	uint64 buffer;		// ........abits的地址
	if (argaddr(0,&start_addr) < 0 || argint(1,&amount) < 0 || 
			argaddr(2,&buffer) < 0) //获取三个参数
		return -1;
	struct proc* p = myproc();
	uint64 mask = access_check(p->pagetable,start_addr); // 页表是当前进程的页表
	if (copyout(p->pagetable,buffer,(char*)&mask,sizeof(uint64)) < 0)
		return -1;
	return 0;
}

```

```c
/*		kernel/vm.c		*/
uint64 access_check(pagetable_t pagetable, uint64 va)
{
	if (va >= MAXVA)
		panic("walk");
	uint64 mask = 0;
    // 实验中给到最大的页面是32页
	for (int pages = 0; pages < 32; pages++,va += PGSIZE)
	{
		pte_t* pte = 0;
		pagetable_t p = pagetable;
        // 仿照walk函数写的，获取L0级的PTE
		for (int level = 2; level >= 0; level--)
		{
			pte = &p[PX(level,va)];
			if (*pte & PTE_V)
				p = (pagetable_t)PTE2PA(*pte);
		}
		if (*pte & PTE_V && *pte & PTE_A)
		{
			mask |= (1L << pages);
			*pte &= ~(PTE_A);
		}
	}
	return mask;	
}

//defs.h
uint64 access_check(pagetable_t pagetable_t, uint64);
```

#### lab4(traps)

##### gdb命令

- b sum_to     		//在sum_to 部分打断点
- layout asm  	    // 展示asm代码
- layout reg            //展示寄存器代码
- focus  reg             // 关注寄存器窗口
- info breakpoints   // 断点信息
- info  reg   	        // 寄存器信息
- delete                      //删除断点
- c                                 //continue
- layout split                     // 会得到c代码和汇编窗口
- layout  src                //会的到c语言窗口
- apropos                 //内置手册
- apropos  tui              //会显示所有相关的tui命令
- i   frame                 //查看当前栈帧的信息
- i  args                     // 查看函数参数的信息
- backtrace /bt           // 得到整个栈帧的backtrace
- p(print)   *agrv              //查看*argv 的地址
- p x/d(地址)

##### RSIC_V的陷入机制

- `stvec`：内核在这里写入其陷阱处理程序的地址；RISC-V跳转到这里处理陷阱。
- `sepc`：当发生陷阱时，RISC-V会在这里保存程序计数器`pc`（因为`pc`会被`stvec`覆盖）。`sret`（从陷阱返回）指令会将`sepc`复制到`pc`。内核可以写入`sepc`来控制`sret`的去向。
- `scause`： RISC-V在这里放置一个描述陷阱原因的数字。
- `sscratch`：内核在这里放置了一个值，这个值在陷阱处理程序一开始就会派上用场。
- `sstatus`：其中的**SIE**位控制设备中断是否启用。如果内核清空**SIE**，RISC-V将推迟设备中断，直到内核重新设置**SIE**。**SPP**位指示陷阱是来自用户模式还是管理模式，并控制`sret`返回的模式。

上述寄存器都用于在管理模式下处理陷阱，在用户模式下不能读取或写入。在机器模式下处理陷阱有一组等效的控制寄存器，xv6仅在计时器中断的特殊情况下使用它们。

CPU不会切换到内核页表，不会切换到内核栈，也不会保存除`pc`之外的任何寄存器。内核软件必须执行这些任务。CPU在陷阱期间执行尽可能少量工作的一个原因是为软件提供灵活性；例如，一些操作系统在某些情况下不需要页表切换，这可以提高性能。

因此，CPU使用专门的寄存器切换到内核指定的指令地址，即`stvec`，是很重要的。



指令在执行的时候会暂停，分为以下三种情况

- 异常调用：例如系统执行ecall指令
- 异常： 例如除零或使用无效地址
- 中断： 例如设备发出读写请求

trap的大致流程

- 控制权交给内核`ecall`
- 内核保存寄存器状态方便日后回复 
  - `uservec` `trampoline.S`
    - stvec 设置为 uservec
  - 使用`csrrw`先交换`a0` 和 `sscratch` 的内容   `a0` -> 保存了陷阱帧的指针（trapframe）
    - 陷阱帧包含执行当前内核占的指针，当前CPU的hartid， usertrap地址和内核页表的地址
    - uservec 取这些值，将satp 切换到内核页表，并调用usertrap
  - 保存用户寄存器
- 内核执行相应的代码  
  - usertrap 的任务是确定陷阱原因，处理并返回。
  - 改变了stvec 为kernelvec
  - epc  保存 sepc （保存用户程序计数器）
  - 如果陷阱是系统调用， syscall 会处理，
    - 系统调用路径保存在 epc+4的位置
  - 如果是设备中断  devintr会处理，
  - 或者是页错误， 
  - 否则是异常，需要杀死进程
- 内核恢复状态并从trap中返回
  - 第一步调用usertrapret
    - stvec  恢复为 uservec
    - spc 设置为之间保存的用户程序计数器
    - 在trampoline.S 上调用userret， userret中的汇编代码会切换页表
- 代码回到原来的地方
  - userret 复制陷阱帧 从`a0` - > `sscratch`
  - sret 返回用户空间

##### 调用系统调用 （exec）

- exec 需要将参数放在寄存器`a0` 和 `a1`,  并将系统调用号放在 `a7`
- 系统调用号与`syscalls `数组条目相匹配, 它是一个函数指针表
- `ecall`  指令陷入 内核 执行 uservec、usertrap、syscall
- `syscall` 从陷阱帧中保存 `p->trapframe->a7`  检索系统号并用它所以到 `syscalls`
- 第一次系统调用`a7` 中的内容是 8（`SYS_exec`）
- `syscall`将其返回值保存到 `a0`

##### 系统调用参数

- 内核新的系统调用接口需要找到用户代码传递的参数
- 参数最初被放置在：寄存器
- `artint`  `artaddr` `artfd`  来检索系统调用的参数，并用整数，指针，文件描述符的形式保存
- 他们都调用`argraw` 来检错相应的保存的用户寄存器
- 内核实现了安全地将数据传输到用户提供的地址和从用户提供的地址传输数据的功能
  - `fetchstr`
  - `fetchstr` 调用 `copyinstr`来完成这个工作
  - `copyinstr` 从用户页表中的虚拟地址`srcva` 复制 `max` 字节到 `dst`
  - 它使用 `walkaddr` 遍历页表 会检查提供的虚拟地址是否为进程用户地址空间的一部分

##### 从内核空间中陷入

- 因为已经在内核了 kernelvec 依赖于设置内核也表的satp，以及指向有效内核栈 的栈指针
- kernelvec将 寄存器保存在被中断的内核线程的栈上
- kernelvec 之后跳转到kerneltrap 
  - 判断陷阱类型： 设备中断 和异常
  - 调用devintr ；来检查设备中断
  - 如果不是设备中断，则必定是异常，内核调用panic并停止执行
- 如果是定时器中断，且内核线程正在运行 ，那么 kerneltrap 会调用 yield 主动让出CPU（时间片轮转）

##### 页面错误异常

- 许多内核使用页面错误来实现COW
- fork 通过调用uvmcopy 为自己分配物理内部才能，并将父级内存保存在复制到其中，使得子级具有与父级相同的内容
- 如果使得父子进程可以共享父级的物理内存，则效率更高

三种不同的页面错误：

- 加载页面错误：加载指令无法转换其虚拟地址
- 存储页面错误：存储指令无法转换其虚拟地址
- 指令页面错误：当加载指令无法转换其虚拟地址
- `scause` 知识页面错误类型

**COW**

- 先让父子最初共享所有物理页面， 并映射为只读
- 当子级或父级执行存储指令时， 引发页面错误异常，内核响应这个异常，复制了包含错误地址的页面， 更改权限为R/W
- 恢复故障进程执行
- 通常子进程会在fork之后立即调用exec，用心的地址空间替换其地址空间
  - 这样子级只会触发很少的页面错误

**lazy  allocation**

- 应用程序调用`sbrk` ,内核增加地址空间，但在页表中将新地址（虚拟地址）标记为无效（不设置PTE_U）
- 但使用到这个地址时， 通过页面错误， 才会分配物理内存 映射到页表

内核仅在应用程序实际使用它时才会分配内存。

**磁盘分页**

- 如果应用程序需要比可用物理内存更多的内存，内核可以换出一些页面（LRU），将他们写入磁盘，并标记他们的PTE为无效
- 如果CPU读取或写入被换出的页面，就会触发页面错误，然后将磁盘中的页面读到内存

##### Backtrace

这个函数就是实现曾经调用函数地址的回溯，这个功能在日常的编程中也经常见到，编译器报错时就是类似的逻辑，只不过题目的要求较为简单，只用打印程序地址，而实际的报错中往往打印程序文件名，函数名以及行号等信息

返回地址位于栈帧帧指针的固定偏移(-8)位置，并且保存的帧指针位于帧指针的固定偏移(-16)位置。先使用`r_fp()`读取当前的帧指针，然后读出返回地址并打印，再将`fp`定位到前一个帧指针的位置继续读取即可。

根据提示：XV6在内核中以页面对齐的地址为每个栈分配一个页面。使用`PGROUNDUP(fp)(栈底) - PGROUNDDOWN(fp)（栈顶） == PGSIZE`判断当前的`fp`是否被分配了一个页面来终止循环。

- void backtrace();		//defs.h

```c
//riscv.h
static inline uint64
r_fp()
{
    uint64 x;
    asm volatile("mv %0, s0" : "=r" (x));
    return x;
}
```

- 在printf.c 中添加函数 backtrace

  ```c
  //backtrace
  void
  backtrace
  {
      printf("backtrace\n");
      uint64 fp = r_fp();
      while(fp != PGROUNDUP(fp)){
          pritnf("%p\n", *(uint64*)(fp-8));
          fp = *(uint64*)(fp-16);
      }
  }
  void
  panic(){
      ...
          backtrace();
      ...
  }
  
  //sysproc.c
  void sleep(){
      ...
      backtrace();
      ...
  }
  ```


##### Alarm

实现一个进程使用 CPU time 时周期性的发出警告的功能。例如周期性的检查中断/异常需要用到这个功能。

增加一个系统调用 sigalarm(interval, handler)，其中 handler 是一个函数，interval 是一个整数，表示经过的时间。sigalarm 的功能是经过 interval 个 CPU ticks ，调用 handler 函数。如果 sigalarm(0, 0) 内核将会停止周期性调用。

proc 中需要保存 interval ，还需要一个字段表示当前时间 n ，每次调用 n-- 。当 n = 0 时执行函数并更新状态。

如果有 handler 函数正在执行那么就不能执行别的 handler 函数，需要加一个字段判断一下。

`Makefile`

```c
$U_alarmtest
```



`kernel/proc.h`

```c
struct proc{
    ...
    int interval;	//间隔时间
    void (*fn)();	//待处理函数
    int count;		//当前的嘀嗒数
    int handling;	//是否正在执行handler函数
    struct trapframe save_info;  //保存信息
};
```

`kernel/proc.c`

```c
//allocproc
static struct proc*
allocproc(void){
....
p->interval = 0;
p->fn = 0;
p->count = 0;
p->handling = 0;
return p
}

static void
freeproc(struct proc *p){
    ...
    p->interval = 0;
    p->fn = 0;
    p->count = 0;
    p->handling = 0;
}

```

`kernel/syscall.h`

```C
#define SYS_sigalarm 22
#define SYS_sigreturn 23
```



`kernel/syscall.c`

```c
extern uint64 sys_sigalarm(void);
extern uint64 sys_sigreturn(void);

static uint64 (*syscalls[])(void) ={
    ...
    [SYS_sigalarm] 		sys_sigalarm,
    [SYS_sigreturn]		sys_sigreturn,
};
```

`kernel/sysproc.c`

```c
uint64
sys_sigalarm(void){
    struct proc *p = myproc();
    int interval;
    void(*fn)();
    if(argint(0, &interval) < 0)
        return -1;
    if(argaddr(1, (uint64*)&fn) < 0)
        return -1;
    p->interval = 0;
    p->fn = fn;
    p->count = interval;
    return 0;
}

uint64
sys_sigreturn(void){
    struct proc* p = myproc();
    p->handling = 0;
    *(p->trapframe) = p->save_info;
    return 0;
}
```

`kernel/trap.c`

```c
//r_scause = 8 是系统调用的参数
void 
usertrap(void)
{
    ...
    if(which_dev == 2){  //时钟中断
        if(p->interval != 0 && !p->handling) // 判断是否已经存在调用
        {
            if(--p->count == 0)
            {
                p->count = p->interval;
                p->handling = 1;
                p->saved_info = *(p->trapframe); //备份
                p->trapframe->epc = (uint64)p->fn;  //执行调用函数
            }
        }
        yield();
    }
    
    usertrapret();
}
```

`user/user.h`

```c
int sigalarm(int ticks, (void)(*handler));
int sigreturn(void)
```

`user/usys.pl`

```c
entry("sigalarm");

entry("sigreturn");
```

#### lab5 cow

目标:实现 COW 并通过 cowtest 和 usertests .

1. 修改 uvmcopy() 父子进程共享物理页并将该页设置为只读,也就是清除 PTE_W 标志。设置 PTE_W 非法可以参考 uvmclear() .
2. 设置 PTE_COW方便识别. 利用 RSW (reserved for software) 位来表示.
3. 为每一个物理页设置引用计数,统计引用该页的次数.也就是设置数组,数组的每一位表示该页的引用次数. kmem 表示物理页.
4. CHUfreerange 表示释放范围内的物理页,释放的时候将页面的引用次数设置为 1 .
5. 在 usertrap 中捕获 page falut ,为 cow page 分配物理内存.
6. 使用 kalloc() 进行内存分配时，需要将对应 page 的 reference count 设置为 1，使用kfree()释放内存时，只能将reference count为 0 的页面放回空闲列表。
7. 注意读写的时候需要加锁.
8. 处理 copyout() , 从内核复制到用户时需要分配相应的物理内存.

`kernel/riscv.h`

```c
#define PTE_COW (1L << 8)
```

`kernel/vm.c`

```c
uint ref[PHYSTOP/PGSIZE]
int uvmcopy(pagetable_t old, pagetable_t new, uint64 sz){
    ...
    *pte &= ~PTE_W;
    *pte |= PTE_COW;
    if(mappages(new, i, PGSIZE, (uint64)pa, PTE_FLAGS(*pte)) != 0)
        goto err;
}
int 
mappages(pagetable_t pagetable, uint64 va, uint64 size, uint64 pa, int perm){
    if(size == 0)
        panic("mappages:size")
    ...
    ref[pa/PGSIZE]++;
    if(a == last)
        ...
}
void
uvmunmap(pagetable_t pagetable, uint64 va, uint64 npages, int do_free){
    ...
    ref[pa/PGSIZE]--;
    if(do_free && ref[pa/PGSIZE] == 1){
        kfree((void*)pa);
    }
}

int
copyout(pagetable_t pagetable, uint64 dstva, char *src, uint64 len){
    pte_t *pte;
    ...
    pte = walk(pagetable, va0, 0); //返回页表
    if(pte == 0)
        return -1;
    if(!(*pte & PTE_W) && *pte & PTE_COW){
        *pte |= PTE_W;
        *pte &= ~PTE_COW;
        if(ref[pa0/PGSIZE] > 2){
            char *mem;
            if((mem = kalloc()) == 0)
                exit(-1);
            memmove(mem, (char *) pa0, PGSIZE);
            ref[pa0/PGSIZE]--;
            ref[(uint64)mem/PGSIZE]++;
            int flag = PTE_FLAGS(*pte);
            *pte = PA2PTE(mem) | flag;
            pa0 = (uint64)mem;
        }
          
    }
        
    
}

```

`kernel/trap.c`

```c
extern uint ref[PHYSTOP/PGSIZE];

void usertrap(void){
    ...
    else if(r_scause() == 13 || r_scause == 15){
        //发生页面错误， 获取访问地址
        uint64 va = PGROUNDDOWN(r_stval()), pa;
        if(pa =walkaddr(p->pagetable, va) == 0)
            exit(-1);
        pte_t *pte;
        //找不到pte
        if((pte == walk(p->pagetable, va, 0)) == 0 || !(*pte & PTE_COW))
            p->killed = 1;
        else{
             *pte |= PTE_W;
        	 *pte &= ~PTE_COW;
            if(ref[pa0/PGSIZE] > 2){
                char *mem;
                if((mem = kalloc()) == 0)
                    exit(-1);
                memmove(mem, (char *) pa0, PGSIZE);
                ref[pa0/PGSIZE]--;
                ref[(uint64)mem/PGSIZE]++;
                int flag = PTE_FLAGS(*pte);
                *pte = PA2PTE(mem) | flag;
            }
        }
        
    }else{
        ....
    }
}
```

（2）跟着提示一步一步来

**(1).** 在***kernel/riscv.h\***中选取PTE中的保留位定义标记一个页面是否为COW Fork页面的标志位

```c
// 记录应用了COW策略后fork的页面
#define PTE_F (1L << 8)
```

**(2).** 在***kalloc.c\***中进行如下修改

- 定义引用计数的全局变量`ref`，其中包含了一个自旋锁和一个引用计数数组，由于`ref`是全局变量，会被自动初始化为全0。

  这里使用自旋锁是考虑到这种情况：进程P1和P2共用内存M，M引用计数为2，此时CPU1要执行`fork`产生P1的子进程，CPU2要终止P2，那么假设两个CPU同时读取引用计数为2，执行完成后CPU1中保存的引用计数为3，CPU2保存的计数为1，那么后赋值的语句会覆盖掉先赋值的语句，从而产生错误

```c
struct ref_stru {
  struct spinlock lock;
  int cnt[PHYSTOP / PGSIZE];  // 引用计数
} ref;
```

- 在`kinit`中初始化`ref`的自旋锁

```c
void
kinit()
{
  initlock(&kmem.lock, "kmem");
  initlock(&ref.lock, "ref");
  freerange(end, (void*)PHYSTOP);
}
```

- 修改`kalloc`和`kfree`函数，在`kalloc`中初始化内存引用计数为1，在`kfree`函数中对内存引用计数减1，如果引用计数为0时才真正删除

```c
void
kfree(void *pa)
{
  struct run *r;

  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    panic("kfree");

  // 只有当引用计数为0了才回收空间
  // 否则只是将引用计数减1
  acquire(&ref.lock);
  if(--ref.cnt[(uint64)pa / PGSIZE] == 0) {
    release(&ref.lock);

    r = (struct run*)pa;

    // Fill with junk to catch dangling refs.
    memset(pa, 1, PGSIZE);

    acquire(&kmem.lock);
    r->next = kmem.freelist;
    kmem.freelist = r;
    release(&kmem.lock);
  } else {
    release(&ref.lock);
  }
}
void *
kalloc(void)
{
  struct run *r;

  acquire(&kmem.lock);
  r = kmem.freelist;
  if(r) {
    kmem.freelist = r->next;
    acquire(&ref.lock);
    ref.cnt[(uint64)r / PGSIZE] = 1;  // 将引用计数初始化为1
    release(&ref.lock);
  }
  release(&kmem.lock);

  if(r)
    memset((char*)r, 5, PGSIZE); // fill with junk
  return (void*)r;
}
```

- 添加如下四个函数，详细说明已在注释中，这些函数中用到了`walk`，记得在***defs.h\***中添加声明，最后也需要将这些函数的声明添加到***defs.h\***，在cowalloc中，读取内存引用计数，如果为1，说明只有当前进程引用了该物理内存（其他进程此前已经被分配到了其他物理页面），就只需要改变PTE使能`PTE_W`；否则就分配物理页面，并将原来的内存引用计数减1。该函数需要返回物理地址，这将在`copyout`中使用到。

```c
/**
 * @brief cowpage 判断一个页面是否为COW页面
 * @param pagetable 指定查询的页表
 * @param va 虚拟地址
 * @return 0 是 -1 不是
 */
int cowpage(pagetable_t pagetable, uint64 va) {
  if(va >= MAXVA)
    return -1;
  pte_t* pte = walk(pagetable, va, 0);
  if(pte == 0)
    return -1;
  if((*pte & PTE_V) == 0)
    return -1;
  return (*pte & PTE_F ? 0 : -1);
}

/**
 * @brief cowalloc copy-on-write分配器
 * @param pagetable 指定页表
 * @param va 指定的虚拟地址,必须页面对齐
 * @return 分配后va对应的物理地址，如果返回0则分配失败
 */
void* cowalloc(pagetable_t pagetable, uint64 va) {
  if(va % PGSIZE != 0)
    return 0;

  uint64 pa = walkaddr(pagetable, va);  // 获取对应的物理地址
  if(pa == 0)
    return 0;

  pte_t* pte = walk(pagetable, va, 0);  // 获取对应的PTE

  if(krefcnt((char*)pa) == 1) {
    // 只剩一个进程对此物理地址存在引用
    // 则直接修改对应的PTE即可
    *pte |= PTE_W;
    *pte &= ~PTE_F;
    return (void*)pa;
  } else {
    // 多个进程对物理内存存在引用
    // 需要分配新的页面，并拷贝旧页面的内容
    char* mem = kalloc();
    if(mem == 0)
      return 0;

    // 复制旧页面内容到新页
    memmove(mem, (char*)pa, PGSIZE);

    // 清除PTE_V，否则在mappagges中会判定为remap
    *pte &= ~PTE_V;

    // 为新页面添加映射
    if(mappages(pagetable, va, PGSIZE, (uint64)mem, (PTE_FLAGS(*pte) | PTE_W) & ~PTE_F) != 0) {
      kfree(mem);
      *pte |= PTE_V;
      return 0;
    }

    // 将原来的物理内存引用计数减1
    kfree((char*)PGROUNDDOWN(pa));
    return mem;
  }
}

/**
 * @brief krefcnt 获取内存的引用计数
 * @param pa 指定的内存地址
 * @return 引用计数
 */
int krefcnt(void* pa) {
  return ref.cnt[(uint64)pa / PGSIZE];
}

/**
 * @brief kaddrefcnt 增加内存的引用计数
 * @param pa 指定的内存地址
 * @return 0:成功 -1:失败
 */
int kaddrefcnt(void* pa) {
  if(((uint64)pa % PGSIZE) != 0 || (char*)pa < end || (uint64)pa >= PHYSTOP)
    return -1;
  acquire(&ref.lock);
  ++ref.cnt[(uint64)pa / PGSIZE];
  release(&ref.lock);
  return 0;
}
```

- 修改`freerange`

```c
void
freerange(void *pa_start, void *pa_end)
{
  char *p;
  p = (char*)PGROUNDUP((uint64)pa_start);
  for(; p + PGSIZE <= (char*)pa_end; p += PGSIZE) {
    // 在kfree中将会对cnt[]减1，这里要先设为1，否则就会减成负数
    ref.cnt[(uint64)p / PGSIZE] = 1;
    kfree(p);
  }
}
```

**(3).** 修改`uvmcopy`，不为子进程分配内存，而是使父子进程共享内存，但禁用`PTE_W`，同时标记`PTE_F`，记得调用`kaddrefcnt`增加引用计数

```c
int
uvmcopy(pagetable_t old, pagetable_t new, uint64 sz)
{
  pte_t *pte;
  uint64 pa, i;
  uint flags;

  for(i = 0; i < sz; i += PGSIZE){
    if((pte = walk(old, i, 0)) == 0)
      panic("uvmcopy: pte should exist");
    if((*pte & PTE_V) == 0)
      panic("uvmcopy: page not present");
    pa = PTE2PA(*pte);
    flags = PTE_FLAGS(*pte);

    // 仅对可写页面设置COW标记
    if(flags & PTE_W) {
      // 禁用写并设置COW Fork标记
      flags = (flags | PTE_F) & ~PTE_W;
      *pte = PA2PTE(pa) | flags;
    }

    if(mappages(new, i, PGSIZE, pa, flags) != 0) {
      uvmunmap(new, 0, i / PGSIZE, 1);
      return -1;
    }
    // 增加内存的引用计数
    kaddrefcnt((char*)pa);
  }
  return 0;
}
```

**(4).** 修改`usertrap`，处理页面错误

```c
uint64 cause = r_scause();
if(cause == 8) {
  ...
} else if((which_dev = devintr()) != 0){
  // ok
} else if(cause == 13 || cause == 15) {
  uint64 fault_va = r_stval();  // 获取出错的虚拟地址
  if(fault_va >= p->sz
    || cowpage(p->pagetable, fault_va) != 0
    || cowalloc(p->pagetable, PGROUNDDOWN(fault_va)) == 0)
    p->killed = 1;
} else {
  ...
}
```

**(5).** 在`copyout`中处理相同的情况，如果是COW页面，需要更换`pa0`指向的物理地址

```c
while(len > 0){
  va0 = PGROUNDDOWN(dstva);
  pa0 = walkaddr(pagetable, va0);

  // 处理COW页面的情况
  if(cowpage(pagetable, va0) == 0) {
    // 更换目标物理地址
    pa0 = (uint64)cowalloc(pagetable, va0);
  }

  if(pa0 == 0)
    return -1;

  ...
}
```