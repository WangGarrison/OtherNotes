# 仿写Bash

## 1.仿Bash写一个命令解析器

用户输入一个命令：

1. bash获取到命令以及传递 的参数
2. 对获取的数据做一些基本处理和解析
3. 创建子进程
4. 父进程可以通过wait()等待子进程结束，子进程通过exec去替换命令对应的可执行文件



Bash启动起来先打印提示符

  ![image-20201030134706844](img/Linux%EF%BC%9A%E4%BB%BF%E5%86%99Bash.img/image-20201030134706844.png)

![image-20201030134527214](img/Linux%EF%BC%9A%E4%BB%BF%E5%86%99Bash.img/image-20201030134527214.png)

接收命令把命令传入

根据分析可以写出整体框架

```c
while(1)
{
    /*
    1.显示终端提示符信息
    2.等待用户输入命令
    3.将用户输入的命令拆解成字符指针数组
      对命令进行划分：内置命令(cd ,exit)——直接在Bash中实现, cd与exit的执行要直接影响于该Bash环境
    				外部命令——执行第四步
    4.创建子进程，子进程exec替换外部命令程序，父进程wait阻塞
    */
}
```

```bash
touch mybash.c
mkdir mybin #放置将来的可执行文件
```

```c
//mybash.c
#include <stdio.h>
#include <stdlib.h>
#include <assert.h>
#include <unistd.h>
#include <string.h>

#include <sys/types.h>
#include <pwd.h>
#include <sys/utsname.h>

#define ARGVNUM  20
#define CMDLEN   128


void ShowInfo()//显示提示信息
{
    //获取用户信息
    struct passwd * pw = getpwuid(getuid());
    assert(pw != NULL);
    
    //获取主机名
    struct utsname host;
    uname(&host);
    
    //获取路径
    char path[128] = {0};
    getcwd(path,127);   //getcwd方法将当前工作目录的绝对路径填充到path中
    //需要的是最后一个目录的名称, ubuntu直接打印绝对目录就可，无需下面这两句
    char *dirname = NULL;
    if(strlen(path) == 1) dirname = path;//根目录
    else if(strcmp(path, pw->pw_dir) == 0)	dirname = "~";//用户的家目录
    else//一般的目录
    {
     	dirname = path + strlen(path);//dirname走到最后面
        while(*dirname != '/')	dirname--;//dirname要变成最后一个目录名
        dirname++;//最后呈现出来没有/的目录名
    }    
    
    //判断$, #
    char flag = '$';//普通用户$, root->#
    if(pw->pw_uid == 0) flag = '#';//0是root用户
    
    printf("My: [%s@%s %s]%c ",pw->pw_name,host.nodename,dirname,flag);
}

void CutCmd(char *cmd,char *Argv[])//切割用户命令cmd，存储到Argv字符指针数组中
{
    int index = 0;
    char *p = strtok(cmd, " ");//strtok替换空格为\0以此达到切分
    while(p != NULL)
    {
        Argv[index++] = p;
        p = strtok(NULL, " ");//下一次strtok从上一个空格之后开始
	}
}

void CarryOutCd(char *path)//cd命令的实现
{
    //1.cd     cd后面什么都不加进入的是家目录
    //2.cd ~   进入家目录
    //3.cd -   返回上一次所在位置
    //4.cd 路径 
    char newpath[CMDLEN] = {0};
    static char oldpath[CMDLEN] = {0};//记录上一次的目录路径
    if(path == NULL)//要进入家目录
    {
        struct passwd *pw = getpwuid(getuid());
        strcpy(newpath, pw->pw_dir);//获取家目录
    }
    else if(strncmp(path, "~", 1) == 0)//用家目录去替换~, ~/code --> home/code
    {
        struct passwd *pw = getpwuid(getuid());
        strcpy(newpath, pw->pw_dir);
        strcat(newpath, "/");
        strcat(newpath, path+1);      
	}
    else if(strncmp(path,"-",1) == 0)//返回上一级
    {
        if(strlen(oldpath) == 0)//没有上次的目录
        {
            printf("OLDPWD NOT SET\n");
            return;
        }
        strcpy(newpath, oldpath);
    }
    else
    {
        strcpy(newpath, path);
	}
    //切换之前的位置就是切换成功之后的上一次位置
    char nowpath[128] = {0};
    getcwd(nowpath,127);
    if(-1 == chdir(newpath))//切换失败
    {
        perror(newpath);
    }    
    else//更新oldpath
    {
        memset(oldpath, 0, 128);//置空oldpath
        strcpy(oldpath, nowpath);
	}
}

void CarryOutCmd(char *Argv[])//实现外部命令
{
	pid_t pid = fork();
	assert(pid != -1);
    if(pid == 0)//子进程
    {
        //exec();
        //exec执行失败,子进程必须结束
        char path[128] = "/home/wanggarrison/Desktop/tulun/mubash/mybin";
        
        
        execv();
        
        perror(Argv[0]);
        exit(0);
    }
    else//父进程
    {
        wait(NULL);//等待子进程结束
    }
}

int main()
{
    while(1)
    {
        ShowInfo();//显示提示信息
        
        char cmd[CMDLEN] = {0};//保存用户输入的命令
        fgets(cmd,CMDLEN-1,stdin);//因为遇到空格也要继续输入所以用fgets
        					 //从stdin里获取命令放入到cmd中，一次最多获取127个字符
        					 //fegts方法：用户输入的字符串以及最后的回车符都会被存储到cmd中
        cmd[strlen(cmd) - 1] = 0;//去掉末尾的回车符
        
        if(strlen(cmd) == 0)	continue;//用户只输入了个回车
        
        char *Argv[ARGVNUM] = {0};//数组，存储的是字符指针类型,存分割的用户命令
        
        CutCmd(cmd,Argv);//分割用户命令,cmd切割到Argv指针数组中去
        
        //区分命令是否为内置命令
        if(strncmp(Argv[0],"cd",2) == 0)//是cd命令
        {
            CarryOutCd(Argv[1]);
		}
        else if(strncmp(Argv[0],"exit",4) == 0)//是exit命令
        {
            exit(0);
		}
        else//外部命令
        {
			CarryOutCmd(Argv);//执行外部命令
		}              
	}
}
```

## 2.仿照系统上的命令实现命令的代码

## 学到的

vim 末行模式按u可以回滚撤销

ctrl+r取消回滚

gcc -o main main.c   或gcc main.c -o main==//-o后面必须是指定的文件，如果错写成.c文件会覆盖掉.c文件的==