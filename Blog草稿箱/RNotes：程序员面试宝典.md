# 程序设计基本概念

1. ```c
   int i = 1;
   
   int main()
   {
       int i = i;
   }
   ```

   main()里面的i是未定义的值：主函数中与全局的同名了，就近原则，==全局的被屏蔽==，所以主函数中i是未初始化的局部变量

2. ![image-20210122162618852](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210122162618852.png)

3. ```cpp
   #include <iostream>
   using namespace std;
   int func(int x)
   {
       int count = 0;
       while(x)
       {
           count++;
           x=x&(x-1);
       }
       return count;
   }
   int main()
   {
       cout<<func(9999)<<endl;
       return 0;
   }
   ```

   答案：8。func函数功能是求形参的==二进制1的个数==，9999二进制：10 0111 0000 1111，所以答案是8

4. ![image-20210122164622931](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210122164622931.png)

   ![image-20210122164747997](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210122164747997.png)

   !和++是同级运算符，但结合顺序是从右向左的，本题中由于!在x左边，++在x右边，所以肯定先算!再算++，如果是`!++x`，则是先算++再算!，例如:

   ```cpp
   #include <iostream>
   using namespace std;
   int main()
   {
       int x = 0;
       int b = !++x;  //先算++，再算!，b的值是0
       cout<<b<<endl;
   }
   ```

   ![image-20210122165317432](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210122165317432.png)

   <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210122165453663.png" alt="image-20210122165453663" style="zoom:67%;" />

5. ```c
   #include <stdio.h>
   int main()
   {
       int b = 3;
       int arr[] = {6,7,8,9,10};
       int *ptr = arr;
       *(ptr++)+=123;
       printf("%d,%d\n", *ptr,*(++ptr));
   }
   ```

   输出结果：8, 8  *(ptr++)先用再加，==printf入栈从右向左==，所以入栈时候先算++ptr

6. <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210123095139966.png" alt="image-20210123095139966" style="zoom:100%;" />

   浮点数在内存里和整数的存储方式不同，==浮点数按阶码方式存储==，(int&)a：将a的引用强转为int类型，1.0f在内存中的存储为：0  011  1111  1  000  0000  0000  0000  0000  0000（0x3f80 0000)，转换为10进制就是1065353216；而0.0f内存中存储的为0000 0000，所以将引用强转为int类型还是0

7. ![image-20210123102740445](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210123102740445.png)

   %08x:  宽度为8，不足的用0补齐，以十六进制格式输出数值

   i        :   0xf7，i是unsigned，所以==左边补0==，为0000  00f7

   *b    :   0xf7，b是有符号类型的，所以左边补1，为ffff  fff7

8. ![image-20210123104811022](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210123104811022.png)

<img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210123104852351.png" alt="image-20210123104852351" style="zoom:50%;" />

9. ![image-20210123105241671](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210123105241671.png)

10. ```c
    int f(int x, int y)
    {
        return (x&y)+((x^y)>>1);
    }
    int main()
    {
        cout<<f(729, 271)<<endl;  //500
    }
    ```

    函数 f功能是==求平均值==：x&y是x和y相同的位进行&，1&1=1，所以结果是相同位的和的一半；x^y是不同位的和，>>1是除以2，所以((x^y)>>1)是不同位的和的一半。这俩个加起来就是相同位的和的一半加不同位的和的一半，所以结果就是所有位加起来的一半，即两数平均值

11. 利用位运算实现==加法==：

    ```c
    //方法一：第10题求平均值的2倍
    int Add1(int x, int y)
    {
        return 2*(x&y) + (x^y); 
    }
    ```

    ```c
    //方法二：递归,这个没看懂
    int Add2(int a, int b)
    {
        if(b == 0)	return a;  //没有进位的时候完成运算
        int sum,carry;
        sum = a^b;  //完成第一步，没有进位的加法运算
        carry = (a & b)<<1; //完成第二步，进位并且左移运算
        return Add(sum,carry);   //进行递归，相加
    }
    ```


12. 不用if、?:、switch等判断语句，求a、b中的较大值

    ```c
    int max = (a+b+abs(a-b))/2;  //大数+小数+大数-小数，再除以2
    ```

13. 有2数据，写一个交换数据的宏

    ```c
    #include <stdio.h>
    #include <string.h>
    
    #define swap(a,b)\
    {	char tempBuf[10];	memcpy(tempBuf, &a, sizeof(a));	memcpy(&a, &b, sizeof(b));	memcpy(&b, tempBuf, sizeof(b));}
    //注意：宏函数换行要用\
    ```

    memcpy原型

    ```c
    void *memcpy(void *destin, void *source, unsigned n);
    //从源source中拷贝n个字节到目标destin中
    ```

     <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210124163229099.png" alt="image-20210124163229099" style="zoom:50%;" />

14. <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210124170407340.png" alt="image-20210124170407340" style="zoom:67%;" />

15. <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210124170941806.png" alt="image-20210124170941806" style="zoom:67%;" />

# 预处理、const与sizeof

1. 宏替换
   <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210124173053356.png" alt="image-20210124173053356" style="zoom:67%;" />

    <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210124173005981.png" alt="image-20210124173005981" style="zoom: 50%;" />

   #define(x,y) (x-y) //给x-y加上括号后

    <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210124173844763.png" alt="image-20210124173844763" style="zoom: 50%;" />

2. ![image-20210124174913519](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210124174913519.png)

3. ![image-20210124175915365](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210124175915365.png)

4. 在const成员函数中，用mutable修饰成员变量名后，就可以修改类的成员变量了

   <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210125190231183.png" alt="image-20210125190231183" style="zoom:67%;" />

5. <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210125192249309.png" alt="image-20210125192249309" style="zoom:70%;" />

   结构体字节对齐：

   - 前面所有成员字节相加要能整除当前成员的字节数
   - 结构体最终大小一定能整除单个类型大小

   取消内存对齐：`#pragma pack(1)`（以1对齐）

   <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210125191828597.png" alt="image-20210125191828597" style="zoom:50%;" />

6. sizeof是一个关键字，计算字符串时算\0，strlen是个求字符串长度的函数，不算\0

   sizeof是个类似宏定义的特殊关键字，括号内的内容在编译过程中是不被编译的，而是被替代类型，如下图中，a的值还是1，因为`sizeof(a=2)`被替代为`sizeof(int)`,赋值语句压根没被编译

    <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210125193602322.png" alt="image-20210125193602322" style="zoom:50%;" />

   sizeof注意点：

   （1）unsigned影响的只是最高位bit的意义（正/负），数据长度是不会变的，所以：`sizeof(unsigned int)==sizeof(int)==4`

   （2）对函数使用sizeof，在编译阶段会被函数返回值的类型替代。如下图，函数并不会被真实调动：

    	<img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210125194508432.png" alt="image-20210125194508432" style="zoom:50%;" />

   （3）sizeof空类、空struct结果是1，多重继承的空类所占空间还是1

   （4）含有虚函数的类或虚继承的类涉及虚表指针，占4字节

7. ![image-20210125200017433](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210125200017433.png)

   内联是以代码膨胀（复制）为代价，仅仅省去了函数调用的开销

# 指针与引用

1. 指针和引用的区别

   - 非空区别。指针可以为NULL，引用必须非空，定义引用时必须进行初始化，没有二级引用，引用效率比指针高
   - 合法性区别。指针使用之前应该做合法性检测，使用引用不需要
   - 可修改区别。指针可以被重新赋值，引用不可以，因此引用定义时必须进行初始化
   - 应用区别。如果总是指向一个对象并且一旦指向一个对象后就不会改变指向，可以使用引用。其他情况使用指针

2.  指针p并没有指向实际的地址，在这种情况下给它赋值是错误的，因为赋的值不知道该放到哪里去，导致出错

    <img src="img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126153945814.png" alt="image-20210126153945814" style="zoom:50%;" />

3. const常量定义时必须进行初始化

   ![image-20210126154341121](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126154341121.png)

4. ![image-20210126154802070](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126154802070.png)

5. ![image-20210126155155747](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126155155747.png)

   ![image-20210126155609930](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126155609930.png)

6. ![image-20210126161219170](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126161219170.png)

7. 主函数返回值类型可以写void或者int，写成int如果不写return语句会默认返回0，其他函数返回值类型写成void可以不写return语句，但是返回值类型是int之类的必须写return语句，不写会出错

8. ![image-20210126171857785](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126171857785.png)

9. ![image-20210126170214914](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126170214914.png)

   ![image-20210126171139076](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126171139076.png)

10. ![image-20210126173010467](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126173010467.png)

11. 三数比大小，函数指针标准写法：

    ![image-20210126173622686](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126173622686.png)

12. ![image-20210126175213405](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126175213405.png)

13. ![image-20210126180441003](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126180441003.png)

14. 迷途指针/悬挂指针：当delete掉一个指针所指向的空间时，并没有把该指针置为NULL，该指针就成为一个迷途指针。当delete一个指针的时候，实际上仅是让编译器释放指针所指的内存，但指针本身依然存在，这时它就是一个迷途指针。使用`myPtr = NULL`可以使其变为空指针

15. ![image-20210126182731665](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210126182731665.png)

16. 句柄和指针的区别和联系

    ![image-20210127140128922](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210127140128922.png)

    > 句柄（handle）是C++程序设计中经常提及的一个术语。它并不是一种具体的、固定不变的数据类型或实体，而是代表了程序设计中的一个广义的概念。句柄一般是指获取另一个对象的方法——一个广义的指针，它的具体形式可能是一个整数、一个对象或就是一个真实的指针，而它的目的就是建立起与被访问对象之间的唯一的联系

# 循环、递归与概率

1. 递归关注三点：

   - 参数、终止条件
   - 本级递归做的事情
   - 返回值

2. 一个等边三角形，每次在外面添加新的三角形。n的前三个值的结果如下图所示。在100次迭代之后会有多少个小三角形?

   ![image-20210127143343151](img/RNotes%EF%BC%9A%E7%A8%8B%E5%BA%8F%E5%91%98%E9%9D%A2%E8%AF%95%E5%AE%9D%E5%85%B8.img/image-20210127143343151.png)

3. T116