# 面试流程

项目介绍STAR模型

 <img src="img/RNotes%EF%BC%9A%E5%89%91%E6%8C%87offer.img/image-20210123144413389.png" alt="image-20210123144413389" style="zoom:70%;" />

# 面试需要的基础知识

#### sizeof空类型

<img src="img/RNotes%EF%BC%9A%E5%89%91%E6%8C%87offer.img/image-20210123151802231.png" alt="image-20210123151802231" style="zoom:67%;" />

#### 复制构造函数无穷递归

```cpp
class A
{
private:
	int value;
public:
    A(int n) {value = n;}
    A(A other) {value = other.value;}
    void Print() {std::cout<<value<<std::endl;}
};

int main()
{
    A a = 10;
    A b = a;
    b.Print();
    return 0;
}
```

在上述代码中，复制构造函数`A(A other)`传入的参数是A的一个实例。由于是传值参数，把形参复制到实参（形参实参结合时）就会调用复制构造函数，这样就会在复制构造函数内调用复制构造函数，就会造成无穷递归直到栈溢出。因此C++标准不允许复制构造函数传值参数，在VS与GCC中，都将编译出错。

 <img src="img/RNotes%EF%BC%9A%E5%89%91%E6%8C%87offer.img/image-20210123153303947.png" alt="image-20210123153303947" style="zoom:50%;" />

复制构造函数的正确写法：`A(const A & other) {value = other.value;}`

 <img src="img/RNotes%EF%BC%9A%E5%89%91%E6%8C%87offer.img/image-20210123153646808.png" alt="image-20210123153646808" style="zoom:50%;" />

#### 1. 赋值运算符函数

```cpp
class CMyString
{
public:
	CMyString(char *pData = nullptr);
	CMyString(const CMyString & str);
	~CMyString(void);
private:
	char * m_pData;
};
```

请为该类型添加赋值运算符函数。

注意点：

- 返回类型为该类引用：保证连续赋值可以成功
- 传参为常量引用
- 分配新内存之前delete旧内存
- 判断是不是自己给自己赋值

```cpp
CMyString& CMyString::operator=(const CMyString & str)//传参为常量引用
{
	if (this != &str)  //判断是不是自己给自己赋值
	{
		delete[] this->m_pData;//分配新内存之前delete旧内存
		this->m_pData = nullptr;

		this->m_pData = new char[strlen(str.m_pData) + 1];
		strcpy(this->m_pData, str.m_pData);
	}
	return *this;//返回类型为该类引用：保证连续赋值可以成功
}
```

**考虑异常安全性的解法**

上边的解法在new分配内存之前执行了delete，如果new失败，那么this->m_pData就成了空指针。即：一旦在赋值运算符函数内部抛出一个异常，CMyString的实例就不再保持有效的状态，这就违背了异常安全性原则。

> 异常安全原则Exception Safety：如果抛出了异常，对象内的任何成员仍然能保持有效状态，没有数据的破坏及资源泄漏。即：出现了异常，对象本身仍要保证安全

解决办法：方案一：先new再delete，即只在分配内容成功之后再释放原来的内容，也就是当分配内存失败时我们能确保CMyString的实例不会被修改。

方案二：创建一个临时实例，再交换临时实例和原来的实例，如下代码。

```cpp
CMyString& CMyString::operator=(const CMyString & str)
{
	if (this != &str)
	{
		CMyString strTemp(str);

		char *pTemp = strTemp.m_pData;
		strTemp.m_pData = this->m_pData;
		this->m_pData = pTemp;
	}//由于strTemp是一个局部变量，出了if就出了该变量的作用域，就会自动调用析构函数，释放内存,由于strTemp.m_pData指向的内存是之前this->m_pData指向的内存，所以这就相当于自动释放原来的内存
	return *this;
}
```

这个代码里在CMyString的构造函数里用new分配内存。如果由于内存不足抛出诸如bad_alloc等异常，但我们还没有修改原来实例，这也就保证了异常安全性

#### 2. 实现Singleton模式

设计一个类，我们只能生成该类的一个实例

以下代码是懒汉单例模式：即在第一次调用的时候才进行实例化，加锁保证线程安全。

单例模式详细介绍在我的本地博客`C++：设计模式.md`

```cpp
std::mutex mtx;
class Singleton
{
private:
	static Singleton* instance;
	Singleton() {}
	Singleton(const Singleton&) = delete;
	Singleton& operator=(const Singleton&) = delete;
public:
	~Singleton() {}
	static Singleton* GetInstance()
	{
		if (instance == nullptr)
		{
			std::lock_guard<std::mutex> lock(mtx);
			instance = new Singleton();
		}
		return instance;
	}
};
Singleton* Singleton::instance = nullptr;
```

#### sizeof数组相关问题

![image-20210123174621216](img/RNotes%EF%BC%9A%E5%89%91%E6%8C%87offer.img/image-20210123174621216.png)

```cpp
/*
	int a[10]; a就相当于int *，如果是对它加1（a + 1）是相当于a + 1 * sizeof(int)。
	但是&a的类型则相当于int **，是所谓指向数组的指针，是数组元素类型的二级指针，
	对它加1是相当于 &a + 1 * sizeof(a)的，所以会偏移一个数组长度。
*/
int main()
{
	int arr[6] = { 1,2,3,4,5,6 };

	int *ptr = (int *)(&arr + 1);  //加到数组末尾了
	printf("%d\n", *(ptr - 1));    //最后一个元素6
	
	int * ptr2 = (arr + 1);  //加到第二个元素
	printf("%d\n", *(ptr2 - 1));  //第一个元素1

	cout << *(arr + 1) << endl;  //偏移一个元素: 2
	cout << *(&arr + 1) << endl; //偏移整个数组长度: 随机数

	cout << sizeof(arr) << endl;  //24
    cout << sizeof(&arr) <<endl;  //4
}
```

#### 3. 数组中重复的数字


在一个长度为 n 的数组 nums 里的所有数字都在 0～n-1 的范围内。数组中某些数字是重复的，但不知道有几个数字重复了，也不知道每个数字重复了几次。请找出数组中任意一个重复的数字。

示例 1：

```c
输入：
[2, 3, 1, 0, 2, 5, 3]
输出：2 或 3 
```

**我的思路：**

- 一个萝卜一个坑，遍历数组，每次把这个位置上的数字交换到它本该在的位置，交换过程中有重复的发生，则返回
- 如果所有萝卜都归位则说明没有重复数字
- 时间复杂度O(n)，空间复杂度O(1)

```c
int findRepeatNumber(int* nums, int numsSize)
{
    if(nums == NULL || numsSize<=0) return -1;
    
    int temp;
    int i = 0;
    for(; i < numsSize; ++i)
    {
        while(nums[i] != i)  //交换nums[i]与nums[nums[i]];
        {
            if(nums[i] == nums[nums[i]])    return nums[i];
            temp = nums[i];  
            nums[i] = nums[temp]; 
            nums[temp] = temp; 
        }
    }
    return -1;
}
```

**如果不能修改原数组，怎样找出重复数字呢？**

**我的思路：**

- 创建辅助数组，一个萝卜一个坑，把原数组填充到新数组中去，如果出现重复填充，则返回
- 时间复杂度O(n)，空间复杂度O(n)

```c
int findRepeatNumber(int* nums, int numsSize)
{
    if(nums == NULL || numsSize <= 0)  return -1;
	
    //创建辅助数组
    int *nums2 = (int*)malloc(sizeof(int)*numsSize);
    int i = 0;

    //初始化辅助数组
    for(; i < numsSize; ++i)    nums2[i] = -1;

    //一个萝卜一个坑
    for(i = 0; i < numsSize; ++i)
    {
        if(nums[i] == nums2[nums[i]])   
        {
            free(nums2);
            return nums[i];
        }
        nums2[nums[i]] = nums[i];    
    }
    free(nums2);
    return -1;
}
```

#### 4. 二维数组中的查找

在一个 n * m 的二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个高效的函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

示例:

```c
现有矩阵 matrix 如下：
[
  [1,   4,  7, 11, 15],
  [2,   5,  8, 12, 19],
  [3,   6,  9, 16, 22],
  [10, 13, 14, 17, 24],
  [18, 21, 23, 26, 30]
]
给定 target = 5，返回 true。
给定 target = 20，返回 false。
```

**我的思路：**

- 从右上角开始找，target比当前大，就向下找，比当前小，就向左找

C版本：

```c
bool findNumberIn2DArray(int** matrix, int matrixRowSize, int* matrixColSize, int target)
{
    if(matrix == NULL || matrixRowSize == 0 || *matrixColSize == 0)   return false;
    
    int i = 0;  //行初始坐标
    int j = *matrixColSize - 1; //列初始坐标
    
    while( i < matrixRowSize && j >=0)
    {
        if(matrix[i][j] == target)	
            return true;
        else if(target > matrix[i][j])
            i++;
        else
            j--;
    }
    return false;
}
```

C++版本：

```cpp
class Solution {
public:
    bool findNumberIn2DArray(vector<vector<int>>& matrix, int target) 
    {
        if (matrix.size()==0)   return false;
        
        int i = 0;  //行初始坐标
        int j = matrix[0].size() - 1; //列初始坐标
        
        while(i < matrix.size() && j >= 0)
        {
            if(target == matrix[i][j])  return true;
            else if(target > matrix[i][j])  i++;
            else j--;
        }
        return false; 
    }
};
```

#### 指向常量字符串的指针

![image-20210127175719467](img/RNotes%EF%BC%9A%E5%89%91%E6%8C%87offer.img/image-20210127175719467.png)

为了节省内存，C/C++把常量字符串放到单独的一个内存区域。当几个指针赋值给相同的常量字符串时，它们实际上会指向相同的内存地址

![image-20210127180020987](img/RNotes%EF%BC%9A%E5%89%91%E6%8C%87offer.img/image-20210127180020987.png)

#### 5. 替换空格

请实现一个函数，把字符串 s 中的每个空格替换成"%20"。

示例 1：

```cpp
输入：s = "We are happy."
输出："We%20are%20happy."
```

**我的思路一：**

 <img src="img/RNotes%EF%BC%9A%E5%89%91%E6%8C%87offer.img/image-20210127201747543.png" alt="image-20210127201747543" style="zoom:50%;" />

算法一代码：

```c
char* replaceSpace(char* s)
{
    if(s == NULL)   return NULL;

    //先计算该字符串的长度，如果是空格就多加两个
    int length = 0;
    int i = 0;
    while(s[i] != '\0')
    {
        if(s[i] == ' ')	{ length += 2; }
        length++;
        i++;
    }
    length++; //结尾的\0
    char * new_str = (char*)malloc(length*sizeof(char));
    i = 0;
    int j = 0;
    while(s[i] != '\0')
    {
        if(s[i] == ' ')
        {
            new_str[j++] = '%';
            new_str[j++] = '2';
            new_str[j++] = '0';
            i++;
        }
        else
        {
            new_str[j++] = s[i++];
        }
    }
    new_str[j] = '\0';
    return new_str;
}
```

算法二代码：如果题目告诉str1空间够用就不需要开新数组，直接原地从后面开始赋值；如果str1空间不够就开

![image-20210127203014396](img/RNotes%EF%BC%9A%E5%89%91%E6%8C%87offer.img/image-20210127203014396.png)

```c
char* replaceSpace(char *str1)
{
	//先统计空格数量，利用i定位到字符串末尾，j=i+空格数*2；
	//再从后往前替换，遇到非空格的直接放，遇到空格替换为0 2 %, j往前移动3个
	int i, j;
	int blank = 0;//空格数量
	for (i = 0; str1[i] != '\0'; i++)
	{
		if (str1[i] == ' ')
			blank++;
	}
    char * str2 = (char*)malloc((i+blank*2+1)*sizeof(char));
	for (j = i + blank * 2; i >= 0; i--)//i是旧串末尾，j是新串的末尾
	{
		if (str1[i] != ' ')
		{
			str2[j] = str1[i];
			j--;			
		}
		else//是空格
		{
			str2[j--] = '0';
			str2[j--] = '2';
			str2[j--] = '%';
		}
	}
    return str2;
}
```

**我的思路二：**

- 利用C++string类的push_back，直接新建一个string往进放

```cpp
class Solution {
public:
    string replaceSpace(string s) 
    {
        string new_str;
        for(auto & it : s)
        {
            if(it == ' ')  //it类型推导出是char&
            {
                new_str.push_back('%'); 
                new_str.push_back('2');
                new_str.push_back('0');
            }
            else
            {
                new_str.push_back(it);
            }
        }
        return new_str;
    }
};
```

#### 头指针传递二级指针原因

![image-20210127204031578](img/RNotes%EF%BC%9A%E5%89%91%E6%8C%87offer.img/image-20210127204031578.png)

#### 6.从尾到头打印链表

输入一个链表的头节点，从尾到头反过来返回每个节点的值（用数组返回）。

**示例 1：**

```
输入：head = [1,3,2]
输出：[2,3,1]
```

**我的思路一：**

- 先反转链表，再进行输出，但是这种方法不好，因为这会改变原链表的结构，通常打印是只读操作，一般希望修改原数据

**我的思路二：**

- 利用栈，先入栈，再出栈

```cpp
class Solution {
public:
    vector<int> reversePrint(ListNode* head) 
    {
        vector<int> ve;  //存放新数据
        std::stack<ListNode*> st;  ////利用栈，入栈再出栈
		
        while(head != nullptr)
        {
            st.push(head);
            head = head->next;
        }

        while(!st.empty())
        {
            ve.push_back(st.top()->val);
            st.pop();
        }
        return ve;
    }
};
```

**我的思路三：**

- 递归：递归本质上也是一种栈，先递推到尾部，再回溯，回溯的时候往数组里放，就实现了尾部的放在数组头
- 缺点：当链表很长时，会导致递归函数调用层级很深，函数栈消耗大量空间

```cpp
class Solution {
public:
    vector<int> ve;  //注意：定义在反转函数外
    vector<int> reversePrint(ListNode* head) 
    { 
        //递归本质上也是一种栈，先递推到尾部，再回溯
        recur(head);
        return ve;        
    }
    void recur(ListNode * head)
    {
        if(head == nullptr) return;
        recur(head->next);  //递推到尾部
        ve.push_back(head->val); //回溯的时候往数组中放
    }
};
```

#### 7. 重建二叉树

输入某二叉树的前序遍历和中序遍历的结果，请重建该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如:

    前序遍历 preorder = [3,9,20,15,7]
    中序遍历 inorder = [9,3,15,20,7]
    返回如下的二叉树：
        3
       / \
      9  20
        /  \
       15   7
T81
