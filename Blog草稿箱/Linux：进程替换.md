#### 进程替换

进程替换要执行的程序，用新的程序指令替换当前进程的指令代码，进程替换的系统调用是**exec函数族**

当进程调用一种exec函数时，该进程执行的程序完全替换为新程序，而新程序则从其main函数开始执行。因为调用exec并不创建新进程，所以前后的进程ID并未改变。exec只是用一个全新的程序替换了当前进程的正文、数据、堆和栈段

进程替换的系统调用：

```c
int execve(const char *pathname, char *argv[],char *envp[]);
```

pathname：要替换的新程序的路径+文件名

argv：给新程序main方法传递的参数列表

envp：给新程序的main方法传递的环境变量列表

#### 封装了该系统调用的库函数

```c
#include <unistd.h>
int execl(const char* pathname, char *argv[0], char *argv[1],......,NULL);
int execv(const char* pathname, char *argv[]);
int execle(const char* pathname, char *argv[0], char *argv[1],......,NULL, char *envp[]);
int execlp(const char* filename, char *argv[0], char *argv[1],......,NULL);
int execvp(const char* filename, char *argv[], char *envp[]);
```

execlp与execvp（最后两个）如果没有给路径，则默认从环境变量PATH指定的路径搜索

exec这些方法执行成功没有返回值，如果执行失败返回-1

exec执行成功，exec之后的指令不会再执行，从exec指定的新程序的main方法开始执行

#### 例：

```c
//exec.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main()
{
    printf("i am exec :: %d\n",getpid()); 
    
    execl("./main", "./main", "1", "2", NULL);//进程替换
    
    printf("Hello World!\n");	
    
    exit(0);
}
```

```c
//main.c
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char *argv[])
{
    printf("i am main :: %d\n",getpid());
    
    int i=0;
    for(;i<argc;i++)
    {
        printf("argv[%d] = %s\n",i,argv[i]);
    }
    exit(0);
}
```

   执行exec可执行文件

 ![image-20201028190313995](img/Linux%EF%BC%9A%E8%BF%9B%E7%A8%8B%E6%9B%BF%E6%8D%A2.img/image-20201028190313995.png)

![image-20201028190226914](img/Linux%EF%BC%9A%E8%BF%9B%E7%A8%8B%E6%9B%BF%E6%8D%A2.img/image-20201028190226914.png)

执行完execl之后，exec.c里execl之后的不会再执行，而是从指定的./main程序的main方法开始执行，没有换进程，仍然是同一个进程，所以上图打印出来的pid是一样的

fork之后，为了使用子进程作为主进程，子进程的代码就会使用exec做进程替换

进程替换有利于模块化编程
