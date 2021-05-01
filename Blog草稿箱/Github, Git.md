# git一般流程

**==git clone URL/git pull==     ==添加代码==     ==git add main.cpp==     ==git commit -m "注释"==     ==git push==**

![image-20210501140223410](img/Github,%20Git.img/image-20210501140223410.png)

# 不用代码版本管理工具的缺点

大家在开发项目的过程中，如果直接在本地系统上维护源码目录，经常会碰见下面的问题：

- 不小心把源代码的目录或文件删了，写了好久的代码没了！
- 按需求添加新功能，写了好多代码，但净是编译错误，改都改不完，想回到之前的版本，开始大面
  积删除或者屏蔽代码，很崩溃，如果此时有个代码版本管理工具，该多好！
- 新功能添加完了，编译运行一切很顺利，功能也正常，但有时候运行会出现以前没见过的运行错
  误，非必现的，想查看和之前代码的差异，看看都在哪些源文件中修改了代码，该怎么办？
- 团队开发项目，但是项目成员都不在一起，各自写的代码该如何添加到一块，还能避免错误，不会
  出现谁把谁的代码给覆盖了？  

# git与github

**git**：是目前世界上最先进的**分布式版本控制系统**（对比集中式版本控制系统SVN），没有之一，Linux创始人Linus Torvalds开发

**github**：一个免费的代码远程托管仓库

SVN集中式版本控制系统与git分布式控制系统区别：

- 集中式：SVN server维护了代码所有的版本，clinet端没有仓库，SVN server挂掉就都用不成了
- 分布式：每个结点相对独立，每个client都会维护一个本地仓库，大家在自己的仓库独立地开发，最后再进行分枝合并

<img align='left' src="img/Github,%20Git.img/image-20210429165446053.png" alt="image-20210429165446053" style="zoom: 43%;" />

# git常用操作命令

```shell
#创建一个目录
mkdir github
cd github

#克隆仓库
git clone git@github.com:WangGarrison/Algorith

#查看日志
git log

#查看文本信息
cat README.md

#查看远程仓库名称，默认名称是origin，在.git目录下的config里可以更改名字
git remote

#ls：列出当前目录下内容
ls -a  #列出所有的，包含隐藏文件

#查看当前分枝
git branch

#更改分枝名字
git branch -m oldname newname 

#查看状态，查看在你上次提交之后是否有对文件进行再次修改
git status

#将当前目录下所有修改的添加到本地仓库
git add .  #git add main.cpp
```

# gitclone

- 把远程仓库的代码克隆到本地，远程仓库名字为origin
- 本地仓库生成一个默认的主干分支master

<img align='left' src="img/Github,%20Git.img/image-20210430143251473.png" alt="image-20210430143251473" style="zoom:50%;" />

# git add  |  git commit  |  git push

> 工作区：当前存放项目代码的目录  
>
> 暂存区：git add把工作区修改的内容添加到暂存区当中  
>
> 本地仓库：git commit把本地暂存区的修改提交到本地代码仓库分支中（不同分支代表不同的代
> 码版本）  
>
> 远程仓库：通过git push把本地仓库的某一个分支上的代码推送到远程仓库的某个分支上  
>
> HEAD指针：本地仓库每一个分支上的代码修改都会生成一个commit id信息，HEAD指针指向最
> 近一次的commit提交，通过这个commit id可以进行版本回退  

git工作区：本地仓库的代码目录

git add：把工作区的代码改动，提交到暂存区中

git commit：将暂存区的提交到本地git仓库的分支上

git push：本地仓库推送到远程仓库 git push origin master 将本地的master推送到远程的origin

git pull：从远程仓库把代码拉取到本地

<img align='left' src="img/Github,%20Git.img/image-20210430145621297.png" alt="image-20210430145621297" style="zoom:90%;" />

# 版本回退

![image-20210430155442788](img/Github,%20Git.img/image-20210430155442788.png)

```shell
#查看回退日志，查看HEAD指针的改动日志
git reflog

#在git add之前，把工作区的代码用版本库中的代码覆盖掉，注意命令中的--不能去掉，否则成切换分支的命令了
git checkout -- 

#把git add之后，暂存区的内容全部撤销
git reset HEAD

#强制推送本地仓库代码到远程仓库
git push -f 

#查看工作区file文件和仓库中该文件最新版本的代码有什么区别
git diff HEAD --
```

代码检查好再往远程仓库上提交，本地仓库代码错了能回退，能修改，push到远程，别人可能把你错的代码clone走了，引起不好的后果

如果远程仓库要回退，先拉到本地，在本地修改好/回退好，再推送给远程

如果本地仓库代码版本落后于远程仓库代码版本，直接git push是不行的，得用git push -f 强制推

# 冲突产生与解决

![image-20210430175408426](img/Github,%20Git.img/image-20210430175408426.png)

小张与啊亮都pull了V1版本的代码，之后啊亮push了V3版本的代码，这之后如果小张准备push代码，那么就会报错，因为仓库已经变成了V3版本，与小张拉代码之前版本不一样了，所以会报错，拒绝小张的push

- 如果小亮和小张修改的代码不是同一处：小张再pull一下，这样就会把V3与小张的V2合并到小张的本地仓库，小张就可以再次push了

- 如果小亮和小张修改的代码是同一处：小张先git pull，自动合并不了了，冲突内容会显示在修改的文件中，需要手动去修改，手动修改之后再去重新push提交
- ![image-20210501110248981](img/Github,%20Git.img/image-20210501110248981.png)

# 分支

**分支简介**

<img align='left' src="img/Github,%20Git.img/image-20210501135745806.png" alt="image-20210501135745806" style="zoom:50%;" />

在开发的时候往往是团队协作，多人进行开发，因此光有一个分支是无法满足多人同时开发的需求的，并且在分支上工作并不影响其他分支的正常使用，会更加安全，Git鼓励开发者使用分支去完成一些开发任务

**分支命令**

```shell
#查看本地分支，查询的带*的表示当前分支
git branch

#查看远程分支
git branch -r

#查看所有分支（本地和远程）
git branch -a

#查看当前本地分支隶属的远程分支
git branch -vv

#创建分支
git branch name

#创建分支并切换
git checkout -b sortdev

#切换分支
git checkout name

#合并分支，把分支brancha合并到当前分支
git merge brancha

#删除分支
git branch -d name  #小写d如果该分支没有合并到master会提示你不能删，大写D是强制删

#推送当前分支的代码到远程origin仓库，本地的brancha分支推到远程的master分支
git push origin    brancha:master
git push 远程仓库名 本地分支名:远程分支名
```

# 帮助命令

```shell
git help
```



