二维数组名的类型

```cpp
int matrix[6][7];
//matrix类型int(*)[7]，matrix是一个指针，指向一个数组长度为7
```

将二维数组当作参数的时候，必须指明所有维数大小或者省略第一维的，但是不能省略第二维或者更高维的大小，这是由编译器原理限制的。大家在学编译原理这么课程的时候知道编译器是这样处理数组的：

 对于数组 *int p\[m][n];*

 如果要取*p\[i][j]*的值*(i>=0 && i<m && 0<=j && j < n)*，编译器是这样寻址的，它的地址为：

 *p + i\*n + j;*

 从以上可以看出，如果我们省略了第二维或者更高维的大小，编译器将不知道如何正确的寻址。但是我们在编写程序的时候却需要用到各个维数都不固定的二维数组作为参数，这就难办了，编译器不能识别阿，怎么办呢？不要着急，编译器虽然不能识别，但是我们完全可以不把它当作一个二维数组，而是把它当作一个普通的指针，再另外加上两个参数指明各个维数，然后我们为二维数组手工寻址，这样就达到了将二维数组作为函数的参数传递的目的，根据这个思想，我们可以把维数固定的参数变为维数随即的参数\

```cpp
void Func(int array[3][10]);
void Func(int array[][10])
```

 变为：

```cpp
void Func(int **array, int m, int n);
```

示例：

```cpp
#include <iostream>
#include <set>
using namespace std;

int findK(int **matrix, const int row, const int col, int k)
{
	multiset<int> mset;
	for (int i = 0; i < row; i++)
	{
		for (int j = 0; j < col; j++)
		{
			mset.insert(*((int*)matrix + i * col+j));  //编译器不知道二维数组的列数，
			//所以编译器并不会为matrix[i][j]寻址，所以不能写成matrix[i][j]，要把二维数组强转为指针手工寻址
		}
	}
	multiset<int>::iterator it = mset.begin();
	for (int i = 0; i < k; i++)
	{
		it++;
	}
	return *it;
}

int main()
{
	//按行和列递增的矩阵m*n，用最优的方式计算第k小的数
	int matrix[6][6] = {
		{1,3,5,7,9,11},
		{2,4,6,8,10,12},
		{3,5,7,9,11,13},
		{4,6,8,10,12,14},
		{5,7,9,11,13,15},
		{6,8,10,12,14,16}
	};

	int k = findK((int**)matrix,6,6, 7);  //matrix类型为 int(*matrix)[6]，强转为二维指针传递过去
	cout << k << endl;
}
```



