---
title: linux进程
date: 2018-04-25 07:12:52
tags: linux系统编程
---
## 进程
### fork
> 当程序遇到fork时, 就会fork出一个子进程, 子进程返回0, 父进程返回子进程的pid, 子进程会拷贝父进程的数据空间、堆、栈等资源成副本, 这意味着父子进程间不共享这些存储空间, 因为拥有一样的堆栈数据结构, 子进程会按照当前fork的地方开始执行, 而不是重头开始执行


### 孤儿进程和僵尸进程
> 如果父进程先退出, 子进程还没退出那么会将父进程会变成init进程(即1号进程), 称为孤儿进程; 如果子进程先退出, 父进程还没有退出, 那么子进程必须等到父进程捕捉到子进程的退出状态进行清理回收子进程才真正结束, 否则这个时候子进程就成为僵尸进程
> 通过signal(SIGCHLD, SIG_IGN)通知内核对子进程的结束不关心，由内核回收。如果不想让父进程挂起，可以在父进程中加入一条语句：signal(SIGCHLD,SIG_IGN);表示父进程忽略SIGCHLD信号，该信号是子进程退出的时候向父进程发送的。

```c
#include <sys/types.h>
#include <unistd.h>

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <signal.h>
#include <errno.h>
#include <signal.h>

#include <sys/stat.h>
#include <fcntl.h>
int main(int argc, const char * argv[]) {
    pid_t pid;
    signal(SIGCHLD, SIG_IGN);// 忽略SIGCHLD信号
    printf("before fork pid: %d \n", getpid());
    
    int abc = 10;
    pid = fork();
    
    if (pid == -1) {
        perror("error");
        return -1;
    }
    
    printf("~~pid: %d \n", pid);
    
    if (pid > 0) {
        abc ++;
        printf("parent: pid: %d \n", getpid());
        printf("abc: %d \n", abc);
    }
    else if (pid == 0) {
        abc ++;
        printf("child: %d, parent: %d \n", getpid(), getppid());
        printf("abc: %d\n", abc);
        sleep(110);
    }
    
    printf("after fork \n");
    return 0;
}
```

```bash
// ps -ef|grep TestProcess
501 11001 11002   0  7:27上午 ??         0:00.11 /Users/lichen/Library/Developer/Xcode/DerivedData/TestProcess-dainbcprniakdaelfdtrmjuxyyio/Build/Products/Debug/TestProcess
501 11004 11001   0  7:27上午 ??         0:00.00 /Users/lichen/Library/Developer/Xcode/DerivedData/TestProcess-dainbcprniakdaelfdtrmjuxyyio/Build/Products/Debug/TestProcess

//父进程退出后
501 11004     1   0  7:27上午 ??         0:00.00 /Users/lichen/Library/Developer/Xcode/DerivedData/TestProcess-dainbcprniakdaelfdtrmjuxyyio/Build/Products/Debug/TestProcess
```

### vfork
> vfork与fork的最大区别是 vfork会共享父进程的数据空间

```c
#include <sys/types.h>
#include <unistd.h>

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <signal.h>
#include <errno.h>
#include <signal.h>

#include <sys/stat.h>
#include <fcntl.h>

int main(int argc, const char * argv[]) {
    
    pid_t pid;
    signal(SIGCHLD, SIG_IGN);
    printf("before fork pid: %d \n", getpid());
    
    int abc = 10;
    pid = vfork();
    
    if (pid == -1) {
        perror("error");
        return -1;
    }
    
    printf("~~pid: %d \n", pid);
    
    if (pid > 0) {
        abc ++;
        printf("parent: pid: %d \n", getpid());
        printf("abc: %d \n", abc);
    }
    else if (pid == 0) {
        abc ++;
        printf("child: %d, parent: %d \n", getpid(), getppid());
        printf("abc: %d\n", abc);
        char *const argv[] = {"ls", "-al", NULL};
        int ret = execve("/bin/ls", argv, NULL);
        if (ret == -1) {
            perror("execve: ");
        }
       exit(0);
    }
    
    printf("after fork \n");
    return 0;
}
```

### exit 与 _exit的区别
> exit是c语言方法, 调用后, 会调用终止处理程序和清空I/O缓存区并输出, 再调用内核终止进程; _exit是系统方法, 会跳过中间步骤, 直接调用内核终止程序, 因此在I/O缓存区的内容无法打印到屏幕上

### 守护进程
> int daemon(int nochdir, int noclose);

```c
#include <sys/types.h>
#include <unistd.h>

#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#include <signal.h>
#include <errno.h>
#include <signal.h>

#include <sys/stat.h>
#include <fcntl.h>

// 一下逻辑与 int daemon(int nochdir, int noclose); 函数相似
int main(int argc, const char * argv[]) {
    int i = 0;
    pid_t pid;
    signal(SIGCHLD, SIG_IGN);
    printf("before fork pid: %d \n", getpid());
    
    pid = fork();
    
    if (pid == -1) {
        perror("fork");
        return -1;
    }
    
    pid = setsid();//新建会话id
    
    if (pid == -1) {
        perror("setsid");
        return -1;
    } else if (pid > 0) {
        exit(0);
    }
    
    chdir("/");
    
    for (i = 0; i < 3; i++) {
        close(i);
    }
    
    open("/dev/null", O_RDWR);
    dup(0);
    dup(1);
    
    while (1) {
        sleep(1);
    }
    
    return 0;
}
```