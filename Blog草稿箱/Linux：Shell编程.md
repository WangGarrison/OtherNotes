# 函数
shell脚本的函数不写返回类型，括号里不写参数，调用函数不写括号，参数直接空格隔开加函数名后面，例如

```shell
foo()
{
	echo "foo start"
	echo "foo \$#=$#"  # $#: 参数个数
	echo "foo \$1=$1"
	echo "foo \$2=$2"
}
foo "123" "abc"`
```

shell脚本没有主函数，从第一句命令开始执行，函数体先越过

**函数参数**

函数内部可以这样取得参数：

- $#：参数个数
- $1：第一个参数
- $2：第二个参数

注意：没有$10这个概念，只有$1~$9，$10解释成$1后面加了个0，一般也不会传10个参数

函数嵌套调用示例：

main不像C/C++里的main，这里就是一个普通函数名为main

```shell
#main.sh
#!/bin/bash  
# '#!'是一个特殊的表示符，其后跟着解释此脚本的shell路径。

foo()
{
	echo "foo start"
	echo "foo \$#=$#"  # $#: 参数个数
	echo "foo \$1=$1"
	echo "foo \$2=$2"
}
main()
{
	echo "hello"
	foo "123" "abc" #给foo函数传参
}

main  
exit 0

#bash man.sh
#chmod u+x main.sh
#./main.sh	
```

**函数返回值**

函数返回值：return 返回，如果不加return，将以最后一条命令运行结果，作为返回值

```shell
#!/bin/bash
add()
{
	if [ $# != 2 ]
    then
    	echo "参数个数不是2个"
    	return 0
    fi
    
	res=`expr $1 + $2`
	return res
}

add 2 3
val=$? 
echo "val=$val"
```

$?是上一行语句执行的结果

shell里成功是0，失败是错误码

shell里没有局部变量这一说法，add函数里的变量res，直接定义在解释器中，所以只要用的还是这个解释器，就可以直接使用res，如下，add外部可以直接使用res

```shell
#!/bin/bash
add()
{
	if [ $# != 2 ]
    then
    	echo "参数个数不是2个"
    	return 0
    fi
    
	res=`expr $1 + $2`
	return res
}

add 2 3
echo "res=$res"   # 可以执行成功
```

如果不想让函数外使用这个变量可以：撤销变量`unset res`或者定义变量前加`local`

# 脚本调用脚本

```shell
#!/bin/bash
#a.sh

echo "a.sh pid=$$"  #$$当前脚本pid
mystr="hello"

echo $mystr
./b.sh $mystr #给b脚本传参
exit 0
```

一个脚本调用另一个脚本，另一个脚本会新启动一个解释器进程，所以两者变量并不共享，要通过传参方式来使用另一个脚本的变量

如何让两个脚本使用同一个解释器呢？

答：调用另一个脚本命令前加一个点空格：`. ./b.sh`，这个点是点命令；或者前面加source空格：`source ./b.sh`

在同一个解释器进程中的两个脚本可以互通变量

把变量提升为环境变量，不同的解释器进程都可以访问到：`export mystr` 

# C语言中调用脚本

![image-20210126114435530](img/Linux%EF%BC%9AShell%E7%BC%96%E7%A8%8B.img/image-20210126114435530.png)

# awk

awk：格式化报文或从一个大的文本文件中抽取数据包

<img src="img/Linux%EF%BC%9AShell%E7%BC%96%E7%A8%8B.img/image-20210126115817695.png" alt="image-20210126115817695" style="zoom:50%;" />

![image-20210126120724506](img/Linux%EF%BC%9AShell%E7%BC%96%E7%A8%8B.img/image-20210126120724506.png)

# sed

编辑匹配到的文本

![image-20210126121614508](img/Linux%EF%BC%9AShell%E7%BC%96%E7%A8%8B.img/image-20210126121614508.png)

