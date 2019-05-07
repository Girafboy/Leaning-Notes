# 第6章 存储器层次架构
> 存储技术：
  - DRAM和SRAM(断电丢失，Main Memory用的是DRAM)、ROM（断电不丢失，flash memory基于EEPROM，用于手机、SSD硬盘等）
  - 磁盘存储
    - 构造：platter->surface->track->sector->bytes
    - 时间计算：
      - access time = seek time + rotational latency + transfer time  
      - T/avg seek = 3~9ms  
      - T/avg rotation = 1/RPM X 60s/1min (RPM- Revolution Per Minute)  
      - T/avg transfer = 1/RPM X 60s/1min X 1/(average#sectors/track)  
    - DMA(Direct Memory Access)
  - SSD
  - 速度 SRAM > DRAM > Disk  
  - 价格 SRAM > DRAM > SSD > 旋转磁盘  
> 局部性（Locality）：
  - 时间局部性：重复引用相同变量的程序
  - 空间局部性：步长越小越好
  - 对于指令：循环体越小，局部性越好
> 层次结构（Hierarchy）：
  - Regs -> L1 cache(SRAM) -> L2 cache(SRAM) -> L3 cache(SRAM) -> Main Memory(DRAM) -> Local disks -> Remote(distributed file systems, Web serves)
> Cache Memories：
  - General organization of cache (S, E, B, m).
  - C = S x E x B (S=2^s B=2^b t=m-(s+b)
  - cache: S set -> E line -> 1 valid bit & t tag bits & B bytes 
  - address: t bits Tag & s bits Set index & b bits Block offset
  - Type:
    - Direct-Mapped Caches(E=1)
    - Set Associative Caches
    - Fully Associative Caches(E=C/B)
  - write hit: write-through v.s. write-back
  - write miss: write-allocate v.s. not-write-allocate
  - i-cache d-cache / unified cache
  - Performance:
    - miss rate
    - hit rate
    - hit time
    - miss penalty
> Cache-Friendly Code
  - Example:  
    ```  
    for(i=0; i<M; i++)
      for(j=0; j<N; j++)
        sum += a[i][j];
    ```
    ```
    for(j=0; j<M; j++)
      for(i=0; i<N; i++)
        sum += a[i][j];
    ```
> Memory mountain
# 第7章 链接
> Compiler Driver
>   - Source files:
>
>     - *.c (ASCII source file)
>   - Translators:
>     - cpp(C preprocessor) -> *.i (ASCII intermediate file)
>     - ccl(C compiler) -> *.s (ASCII assembly-language file)
>     - as(assembler) -> *.o (relocatable object file)
>   - Linker:
>     - ld(linker program) -> *.elf (executable object file)
>     Static Linking
>   - Symbol resolution
>   - Relocation
>   Object File
>   - 分类：
>     - Relocatable object file
>       - Windows -> *.obj
>       - Linux -> *.o
>     - Executable object file
>       - Windows -> *.exe
>       - Linux -> /bin/bash
>     - Shared object file 
>       - Windows -> *.dll
>       - Linux -> *.so
>   - Format (COFF)
>     - Windows -> Portable Executable(PE)
>     - MacOS-X -> Mach-O
>     - x86-64 Linux & Unix -> Executable and Linkable Format(ELF)
>   - Relocatable Object Files
>     - ELF header:
>       1. 生成该文件的系统的字的大小和字节顺序
>       2. ELF header的大小
>       3. 目标文件的类型（relocatable， executable or shared）
>       4. 机器类型（e.g. x86-64)
>       5. section header table的文件偏移，以及其中entry的大小和数量
>     - .text:(Ndx=1)
>       - 已编译的机器代码
>     - .rodata:
>       - 只读数据
>     - .data:(Ndx=3)
>       - 已初始化的全局和静态变量
>     - .bss:（Better save space）
>       - 未初始化的全局和静态变量
>       - 初始化为0的全局和静态变量
>       - 占位符，不占据实际空间；运行时，内存分配，初始值为0
>     - .symtab:
>       - 符号表：定义和引用的函数和全局变量的信息
>       - 类型：
>         - 全局符号： 非静态函数和全局变量
>         - 外部符号： 引用的非静态函数和全局变量
>         - 局部符号： static属性的函数和全局变量（对外部模块不可见）
>       - Entry:
>         - name: String table offset
>         - type: Function or Data
>         - binding: Local or Global
>         - value: Section offset or absolute address
>         - section: <=> Ndx
>           - pseudosection:（仅出现在可重定位目标文件）
>             - ABS: 不该被重定位的符号
>             - UNDEF： 未定义符号
>             - COMMON： 未被分配位置的未初始化数据（未初始化的全局变量，其他是.bss），也即弱全局符号
>       - Resolution:
>         - Rules:（Linux链接器）
>           1. 不允许多个同名的强符号
>           2. 强符号和弱符号同名，选择强符号
>           3. 多个弱符号同名，任意选择一个
>         - static library:
>           - Linux中archive
>           - E-可重定位目标文件集合，U-未解析符号集合，D-已定义符号集合
>           - *.a位置很重要
>     - .rel.text：
>       - .text节中的位置列表，链接时修改位置
>       - 任何调用外部函数或者引用全局变量的指令
>       - 可执行目标文件中通常省略
>     - .rel.data:
>       - 被引用或定义的所有全局变量的重定位信息
>     - .debug:
>       - 局部变量和类型定义、定义和引用的全局变量、原始C源文件
>       - 以-g选项调用才生成
>     - .line:
>       - 源文件中行号与.text中机器指令的映射
>       - 以-g选项调用才生成
>     - .strtab:
>       - .symtab和.debug中的符号表
>       - section header中的section name
>     - Section header table:
>       - 描述了不同section的位置和大小
>       - 每个section有固定大小的entry
>   - Relocation:
>     - Entries:(.rel.text & .rel.data)
>       - offset: 要被修改的重定位处的节偏移
>       - type:
>         - R_X86_64_PC32: PC相对地址，PC指向下一条指令
>         - R_X86_64_32： 32位绝对寻址
>         - R_X86_64_64:    64位绝对地址
>       - symbol: 某全局变量或者函数
>       - addend: 偏移调整
>     - 算法：
>       - 首先： refptr = s + r.offset（要被修改的重定位位置）
>       - PC相对引用：
>         1. refaddr = ADDR(s) + r.offset（要被修改的地方的运行时地址 = 当前section的运行时地址 + 偏移量）
>         2. *refptr = (unsigned)(ADDR(r.symbol) + r.addend - refaddr)（把要被修改的地方的值改为运行时相对地址）
>       - 绝对引用：
>         1. *refptr = (unsigned)(ADDR(r.symbol) + r.addend) （把要被修改的地方的值改为绝对地址）
>   - Executable Object Files
>     - 文件格式：
>       1. ELF头还包括了程序入口点（entry point）
>       2. .text,.rodata和.data与可重定位目标文件相似
>       3. .init定义了函数_init，程序初始化代码调用
>       4. 不再需要.rel节
>     - 对齐要求：vaddr mod align = off mod align（起始地址和节偏移量对齐）
>     - 加载
>   - Dynamic Linking with Shared Libraries
>     - 目标： 解决static library的缺陷（显式更新，浪费内存）
>     - Linux系统接口：
>     ```
>     #include <dlfcn.h>
>     ```

    //成功返回指向句柄的指针，出错返回NULL
    //filename: ./*.so
    //flag: RTLD_NOW(立即解析对外部符号的引用)，RTLD_LAZY(推迟符号解析直到执行来自库中的代码)
    void *dlopen(const char *filename, int flag);
    
    //成功返回指向符号的指针，出错返回NULL
    void *dlsym(void *handle, char *symbol);
    
    //成功返回0，出错返回-1
    int dlclose(void *handle);
    
    //对前面三者调用失败返回错误消息，调用成功返回NULL
    const char *dlerror(void);
    ```
    - 技术实现——PIC(Position-Independent Code)
      - GOT(Global Offset Table):数据段
        - 8 bytes per entry
        - GOT[0] GOT[1] 解析函数地址使用
        - GOT[2] ld-linux.so的入口点
        - GOT[3] 系统start up
        - 从GOT[4]开始指向PLT entry第二条指令或运行时地址
      - PLT(Procedure Linkage Tabl代码段
        - 16 bytes per entry
        - PLT[0] 跳转到dynamic linker
        - PLT[1] 调用系统启动函数__libc_start_main
        - 从PLT[2]开始调用用户代码函数
  - Tools:
    - AR: 静态库
    - STRINGS：列出一个目标文件中所有可打印字符串
    - STRIP：从目标文件删除符号表信息
    - NM：列出一个目标文件的符号表中定义的符号
    - SIZE：列出目标文件中节的名字和大小
    - READELF：显示一个目标文件的完整结构（包含SIZE和NM）
    - OBJDUMP：所有二进制工具之母。反汇编.text
# 第8章 异常控制流
> 通俗的理解：
  - ECF（Exceptional Control Flow）就是在正常指令进行（控制流control flow)的过程中应对突发变化做出的反应。
> Exceptions：
  - Instruction curr --Exception--> Exception processing --> I curr / I next / Abort
  - Type:  

    类别|原因|异步同步|返回行为
    :--:|:--|:--:|:--
    Interrupt|来自IO设备的信号|Async|返回到下一条指令
    Trap|有意的异常|Sync|返回到下一条指令
    Fault|潜在可恢复的错误|Sync|可能返回到当前指令
    Abort|不可恢复的错误|Sync|不返回
    
    - Interrupt: 网络适配器、磁盘控制器、定时器芯片等，中断引脚触发中断
    - Trap：syscall(read,fork,execve,exit,etc.)  
      %rax包含系统调用号  
      参数寄存器传递，而非栈传递，最多6个  
      返回时%rcx,%r11被破坏，%rax包含返回值，负数返回值对应errno  
      > System Call Error Handling
      error-handling wrappers:  
      ```
      void unix_error(char *msg)
      {
        fprintf(stderr, "%s: %s\n", msg, strerror(errno);
        exit(0);
      }
      
      pid_t Fork(void)
      {
        pid_t pid;
        
        if((pid = fork()) < 0)
          unix_error("Fork error");
        return pid;
      }
      ```
      ```
      pid = Fork();
      ```
    - Fault: 缺页异常（page fault exception）
    - Abort: DRAM或SRAM位损坏时发生奇偶校验错误
  - 常见异常：
  
    Exception number | Description | Exception Class
    :--: | :-- | :--
    0 | 除法错误（通常选择终止程序） | Fault
    13 | 一般保护故障（Segmentation Fault） | Fault
    14 | 缺页 | Fault
    18 | Machine check（机器故障）| Abort
    32-255 | 操作系统定义的异常 | Interrupt or Trap
    
> Processes
>   - 两个关键抽象：
>     1. 一个独立的逻辑控制流
>     2. 一个私有的地址空间
>   - 区分两个概念：
>     1. 并发(concurrency)：time slice有交叉
>     2. 并行(parallel)：多核处理和多机协同
>   - 理解：
>     1. User Mode & Kernel Mode
>     2. Context Switches
>   - Process Control:
>     1. 获取进程ID
>     ```
>     #include <sys/types.h> //定义了pid_t为int
>     #include <unistd.h>
>     ```

    pid_t getpid(void);//返回当前进程PID
    pid_t getppid(void);//返回父进程PID
    ```
    2. 创建和终止进程
    ```
    #include <stdlib.h>
    void exit(int status);//不返回，有退出状态
    ```
    ```
    #include <sys/types.h>
    #include <unistd.h>
    //调用一次，返回两次；子进程返回0，父进程返回子进程的PID，出错返回-1
    //并发执行，相同但独立的地址空间，共享文件
    pid_t fork(void);
    ```
    3. 回收子进程
    ```
    #include <sys/types.h>
    #include <sys/wait.h>
    
    //pid>0 => 等待这一个子进程
    //pid=-1 => 等待所有子进程
    
    //默认options=0,等待一个子进程终止就返回，返回已终止子进程的PID
    //options=WNOHANG => 没有可回收子进程就立即返回
    //options=WUNTRACED => 等待一个terminated or stopped子进程
    //options=WCONTINUED => 等待一个terminated子进程 or SIGCONT继续一个stopped子进程
    
    //WIFEXITED(status) => exit或者return正常终止，则返回true
      //WEXITSTATUS(status) => 返回退出状态
    //WIFSIGNALED(status) => 因未捕获的信号终止，则返回true
      //WTERMSIG(status) => 返回导致终止的信号编号
    //WIFSTOPPED(status) => 子进程当前是STOPPED，则返回true
      //WSTOPSIG(status) => 返回引起停止的信号编号
    //WIFCONTINUED(status) => 子进程收到SIGCONT重新启动，则返回真
    
    //没有子进程，则返回-1，设置errno为ECHILD
    //被信号中断，则返回-1，设置errno为EINTR
    pid_t waitpid(pid_t pid, int *statusp, int options)
    
    //wait(&status)等价于waitpid(-1, &status, 0)
    pid_t wait(int *statusp);
    ```
    4. 让进程休眠
    ```
    #include <unistd.h>
    //时间到返回0，否则返回剩下的秒数
    unsigned int sleep(unsigned int secs);
    //休眠直到收到信号，总是返回-1
    int pause(void);
    ```
    5. 加载并运行程序
    ```
    #include <unistd.h>
    //成功不返回，错误返回-1
    //argv和envp都是以null结尾的指针数组，其中每个指针指向栈中的一个命令行/环境变量字符串
    //全局变量environ指向envp[0]
    int execve(const char *filename, const char *argv[], const char *envp[]);
    ```
    ```
    #include <stdlib.h>
    
    //在环境变量数组中搜索“name=value”，若存在则返回指向value的指针，否则返回NULL
    char *getenv(const char *name);
    //成功返回0，失败返回-1.
    //若存在且overwrite非零则覆盖重写，不存在则添加
    int setenv(const char *name, const char *newvalue, int overwrite);
    //不返回，删除对应字符串
    void unsetenv(const char *name);
    ```
> Signals
>   - Sending Signals
>     - 一个进程可以发送信号给自己
>     - process group:
>     ```
>     #include <unistd.h>
>     //返回调用进程的进程组ID
>     pid_t getpgrp(void);
>     //将进程pid的进程组改为pgid
>     //pid=0 => 使用当前进程pid
>     //pgid=0 => 使用pid作为进程组ID
>     //成功返回0，失败返回-1
>     int setpgid(pid_t pid, pid_t pgid);
>     ```
>     - 使用程序  
>     `linux> /bin/kill -9 15213 //发给进程`
>     `linux> /bin/kill -9 -15213 //发给进程组的每个进程`
>     - 键盘发送
>       - Ctrl+C 发送SIGINT到前台进程组的每个进程，终止作业
>       - Ctrl+Z 发送SIGTSTP到前台进程组的每个进程，停止（挂起suspend）作业
>     - kill函数
>     ```
>     #include <sys/types.h>
>     #include <signal.h>
>     //pid=0 => 发送给调用进程所在进程组的每个进程，包括自己
>     //pid<0 => 发送给进程组中每个进程
>     //pid>0 => 发送给某个进程
>     //成功返回0，错误返回-1
>     int kill(pid_t pid, int sig);
>     ```
>     - alarm函数
>     ```
>     #include <unistd.h>
>     //向自己发送SIGALRM信号
>     //返回前一次闹钟剩余的秒数，若以前没有设定闹钟就返回0
>     //secs=0 => 不安排新的闹钟
>     //任何调用都将取消待处理（pending）的闹钟
>     unsigned int alarm(unsigned int secs);
>     ```
>   - Receiving Signals
>   ```
>   #include <signal.h>
>   typedef void (*sighandler_t)(int);
>   //可以修改信号相关联的默认行为，SIGSTOP和SIGKILL不可修改
>   //handler=SIG_IGN => 忽略signum的信号
>   //handler=SIG_DFL => 恢复signum的默认行为
>   ```

  sighandler_t signal(int signum, sighandler_t handler);
  ```
  - Blocking and Unblocking Signals
  ```
  #include <signal.h>
  //how=SIG_BLOCK => blocked=blocked | set
  //how=SIG_UNBLOCK => blocked=blocked & ~set
  //how=SIG_SETMASK => block=set
  //oldset非空 => blocked位向量之前的值保存在oldset中
  //成功返回0，出错返回-1
  int sigprocmask(int how, const sigset_t *set, sigset_t *oldset);
  int sigemptyset(sigset_t *set);
  int sigfillset(sigset_t *set);
  int sigaddset(sigset_t *set, int signum);
  int sigdelset(sigset_t *set, int signum);

  //signum是set的成员返回1，不是返回0，出错返回-1
  int sigismember(const sigset_t *set, int signum);
  ```
  - Signal Handlers
    - Safe：
      1. G0.处理程序要尽可能简单
      2. G1.在处理程序中只调用异步信号安全（async-signal-safe）的函数
      
      异步信号不安全的|异步信号安全的
      :--:|:--:
      printf sprintf|write sio_putl sio_puts sio_error
      malloc|--
      exit|_exit
      --|fork execve waitpid signal
       
      3. G2.保存和恢复errno（处理程序要返回时才有必要，_exit终止就不需要了）
      4. G3.阻塞所有的信号，保护对共享全局数据结构的访问（也即保证事务的原子性）
      5. G4.用volatile声明全局变量（也即保证数据的一致性，不要缓存，必须每次从内存读取）
      6. G5.用sig_atomic_t声明flags
      `volatile sig_atomic_t flag;`
    - Correct:
      1. 信号不会排队等待，信号block后，后来的pending信号被简单丢弃
    - Portable：
      1. signal的函数语义各有不同
      2. 系统调用可以被中断  
      解决方案：Posix标准
  ```
      #include <signal.h>
      
      //成功返回0，出错返回-1
      int sigaction(int signum, struct sigaction *act, struct sigaction *oldact);
      ```
  - Concurrency Bugs：
    - race: 并发交错使得有时先delete后add，有时先add后delete
    - 运用显式block达到Synchronizing Flows来避免这种问题
  - Explicitly Waiting:
    1. Wasteful
    `while(!pid);`
    2. Race
    ```
    while(!pid)
      pause();
    ```
    3. Too slow
    ```
    while(!pid)
      sleep(1);
    ```
    4. Proper
    ```
    #include <signal.h>
    //返回-1
    //暂时用mask替换当前阻塞集合，然后挂起该进程，直到收到一个信号。
    //若信号行为终止，不返回就直接终止
    //若信号行为运行处理程序，从处理程序返回，恢复调用原有的阻塞集合
    int sigsuspend(const sigset_t *mask);
    ```
> Nonlocal Jumps
>   ```
>   #include <setjmp.h>
>   ```

  //自己调用返回0，longjump调用返回非零
  int setjmp(jmp_buf env);
  //sig-是可以被信号处理程序使用的版本，savesigs表明是否保存信号到上下文
  int sigsetjmp(sigjmp_buf env, int savesigs);

  //retval设置了到setjmp的返回值
  void longjmp(jmp_buf env, int retval);
  void siglongjmp(sigjmp_buf env, int retval);
  ```
> Tools:
  - STRACE: 打印一个正在运行的程序和它的子进程调用的每个系统调用的轨迹。
  - PS： 列出系统中的进程
  - /proc： 一个虚拟文件系统
# 第10章 系统级I/O
> Unix I/O
  - 将所有的I/O设备（例如网络、磁盘和终端）都优雅地映射为文件。
  - 打开文件使用一个descriptor标识
  - 每个进程创建开始都有三个打开的文件：standard input(descriptor 0 or STDIN_FILENO), standard output(descriptor 1 or STDOUT_FILENO), standard error(descriptor 2 or STDERR_FILENO)
  - 文件位置file position可以通过seek操作，初试为0
  - 读文件即从文件复制n(n>0)字节到内存,file position响应从k变为k+n,当k大于等于文件字节总数时触发EOF（end-of-file）条件。**文件结尾处并没有EOF符号**  
    写操作就是从内存复制n(n>0)字节到文件，从当前file position开始，然后更新file position
  - 关闭文件后释放descriptor
> Files
  - regular file: 包含任意数据。
    - text file 只包含ASCII或Unicode的普通文件
    - binary file 所有其他文件
  - directory： 包含一组链接（link）的文件，每个链接将filename映射到一个文件（包括目录）
  - socket： 用来与另一个进程进行跨网通信的文件
> 文件操作：
  - ```
    #include <sys/types.h>
    #include <sys/stat.h>
    #include <fcntl.h>
    
    //成功返回new file descriptor, 出错返回-1
    //flags：O_RDONLY,OWRONLY,O_RDWR,O_CREAT(不存在则新建),O_TRUNC(已存在则截断),O_APPEND
    //访问权限mode & ~umask   umask(some_mode);
    //9种权限：S_IR(W or X)USR(GRP or OTH)  
              读/写/执行 * 拥有者/所在组成员/其他任何人
    int open(char *filename, int flags, mode_t mode);
  ```
    ```
    #include <unistd.h>
    //成功返回0，出错返回-1
    int close(int fd);
    ```
    ```
    #include <unistd.h>
    //出现short count的原因：
      //读时遇到EOF，先返回读取值，此后read通过返回0发出EOF信号
      //从终端读文本行，一次读一行
      //读和写socket，网络延迟
    //成功返回读的字节数，若EOF返回0，出错返回-1
    ssize_t read(int fd, void *buf, size_t n);
    //成功返回写的字节数，出错返回-1
    ssize_t write(int fd, const void *buf, size_t n);
    ```
> RIO
>   - 无缓冲的RIO
>   ```
>   #include "csapp.h"
>   //成功返回传送的字节数，若EOF返回0，出错返回-1
>   ssize_t rio_readn(int fd, void *usrbuf, size_t n);
>   sszie_t rio_writen(int fd, void *usrbuf, size_t n);
>   ```
>   - 带缓冲的RIO
>   ```
>   #include "csapp.h"
>   ```

  //把fd和rp处的一个类型为rio_t的读缓冲区联系起来
  void rio_readinitb(rio_t *rp, int fd);

  //成功返回读的字节数，EOF返回0，出错返回-1
  ssize_t rio_readlineb(rio_t *rp, void *usrbuf, size_t maxlen);
  sszie_t rio_readnb(rio_t *rp, void *usrbuf, size_t n);
  ```
> Metadata
  ```
  #include <unistd.h>
  #include <sys/stat.h>

  //检索关于文件的信息
  //st_size 文件字节数大小
  //st_mode 文件访问许可位
    //S_ISREG(st_mode) 是一个普通文件？
    //S_ISDIR(st_mode) 是一个目录文件？
    //S_ISSOCK(st_mode) 是一个socket文件？
  //成功返回0，出错返回-1
  int stat(const char *filename, struct stat *buf);
  int fstat(int fd, struct stat *buf);
  ```
> Directory Contents
  ```
  #include <sys/types.h>
  #include <dirent.h>

  //成功返回directory stream指针，否则为NULL
  DIR *opendir(const char *name);

  //成功返回下一个directory entry,否则为NULL
  struct dirent *readdir(DIR *dirp);

  //成功返回0，错误返回-1
  int closedir(DIR *dirp);
  ```
> Sharing Files
  - Descriptor table: 每个进程都有独立的此表，每个entry使用fd索引，指向File table中的一个entry
  - File table：标识打开文件的集合，所有进程共享。每个entry包括pos、refcnt和指向v-node table中一个entry的指针。关闭fd减少对应refcnt，当refcnt=0时删除该entry。
  - v-node table：所有进程共享，每个entry包含stat中大部分信息，包括st_mode和st_size。
> Redirection
  ```
  #include <unistd.h>
  //复制oldfd到newfd并覆盖
  //若newfd已经打开，复制前先关闭newfd
  //成功返回非负的fd，出错返回-1
  int dup2(int oldfd, int newfd);
  ```
> Standard I/O
  

  ```