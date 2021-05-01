## strtok函数

```c
#include <string.h>

char *strtok(char str[], const char *delim);//函数原型
```

分解字符串为一组字符串。str为要分解的字符串，delim为分隔符字符（所有delim中包含的字符都会被滤掉，并将被滤掉的地方设为一处分割的节点。如果传入字符串，则传入的字符串中每个字符均为分割符）。首次调用时，str指向要分解的字符串，之后再次调用要把str设成NULL。

==作用于字符串str，以包含在delim中的字符为分界符，将str切分成一个个子串；如果，str为空值NULL，则函数保存的指针SAVE_PTR在下一次调用中将作为起始位置。==

该函数返回被分解的第一个子字符串，如果没有可检索的字符串，则返回一个空指针。

例如：

```c
strtok("abc,def,ghi",",");
```

第一次返回的是"abc"的首地址，再次接着分割"def"要这样调用`strtok(NULL,",");`

使用：

```c
#include <string.h>
#include <stdio.h>
 
int main () {
   char str[80] = "This is - www.runoob.com - website";
   char *token;
   
   /* 获取第一个子字符串 */
   token = strtok(str, "-");
   
   /* 继续获取其他的子字符串 */
   while( token != NULL ) 
   {
      printf( "%s\n", token );
      token = strtok(NULL, "-");
   }
    
   return(0);
}
```

特别要注意分割处理后原字符串 str 会变，变成第一个子字符串"This is"

## strtok实现(我的代码)

```c
#include "pch.h"
#include <stdio.h>
#include <string.h>
//myStrtok

char *myStrtok(char str[], const char *delim)//用delim分割str，第一次返回分割后的子串, 当str为NULL接着上次的分割
{
	static char *oldstr = str;
	static int i = 0;
	static int len = strlen(str);

	if (str != NULL)
	{
		while (str[i] == *delim) i++;
		char *temp = &str[i];

		while (i < len && str[i] != *delim)
		{
			i++;
		}
		str[i] = '\0';

		return temp;
	}
	else if (i < len)
	{
		i++;//跳过上次的'\0'
		while(i<len && oldstr[i] == *delim)	i++;			

		char *tmp = &oldstr[i];
		while (i < len && oldstr[i] != *delim)
		{
			i++;
		}
		oldstr[i] = '\0';

		return tmp;
	}
	else
	{
		return NULL;
	}
}

int main()
{
	char str[30] = ",,sad,,,,hsad,,saf,saf,,,,g,d" ;
	char *buf = myStrtok(str, ",");
	while (buf != NULL)
	{
		printf("%s\n", buf);
		buf = myStrtok(NULL, ",");		
	}	
	return 0;
}
```



