### 常量

#### 字面量

常量是固定值，也叫做字面量

#### 整数常量

整数常量可以是十进制、八进制（0开头）或十六（0X开头）进制的常量

#### 定义常量

1. 使用 **#define** 预处理器定义常量

   ```cpp
   #define LENGTH 10   
   ```

2. 使用 **const** 关键字

   ```cpp
   const int  LENGTH = 10;
   ```

### 修饰符类型

#### 速记符号

C++ 允许使用速记符号来声明**无符号短整数**或**无符号长整数**。您可以不写 int，只写单词 **unsigned、short** 或 **long**，**int** 是隐含的。

```cpp
unsigned x;
unsigned int y;
```

#### extern 存储类

extern 修饰符通常用于当有两个或多个文件共享相同的全局变量或函数的时候

```cpp
#include <iostream>
 
int count ;
extern void write_extern();
 
int main()
{
   count = 5;
   write_extern();
}
```

```cpp
#include <iostream>
 
extern int count;
 
void write_extern(void)
{
   std::cout << "Count is " << count << std::endl;
}
```

### 函数

#### 函数参数

1. 传值调用

   把参数的**实际值**赋值给函数的形式参数

   ```cpp
   void swap1(int x, int y) {
       int temp;
       temp = x;
       x = y;
       y = temp;
   
       std::cout << "交换过程中x的值：" << x << std::endl;
       std::cout << "交换过程中y的值：" << y << std::endl;
   }
   
   // swap1(x, y);
   ```

2. 指针调用

   把参数的**地址**赋值给函数的形式参数

   ```cpp
   void swap2(int *x, int *y) {
       int temp;
       temp = *x; // 保存地址 x的值
       *x = *y;
       *y = temp;
   
       std::cout << "交换过程中x的值：" << *x << std::endl;
       std::cout << "交换过程中y的值：" << *y << std::endl;
   }
   
   // swap2(&x, &y);
   ```

3. 引用调用

   把参数的**引用**赋值给函数的形式参数

   ```cpp
   void swap3(int &x, int &y) {
       int temp;
       temp = x;
       x = y;
       y = temp;
   
       std::cout << "交换过程中x的值：" << x << std::endl;
       std::cout << "交换过程中y的值：" << y << std::endl;
   }
   
   // swap3(x, y);
   ```

#### [Lambda函数与表达式](https://www.runoob.com/cplusplus/cpp-functions.html)

### 数组

#### 指向数组的指针



#### 传递数组给函数

#### 从函数返回数组

### 指针

#### 概念

指针是一个变量，其值为另一个变量的内存地址。

#### 基本用法

1. 指针的声明

   ```cpp
   int *ip;     // 整形指针
   double *dp;  
   float *fp;
   char *cp;
   ```


#### Null指针

##### 概念

C++支持空指针。NULL指针是一个定义在标准库中的值为零的常量。

##### 应用场景

在声明指针变量的时候，如果没有确切的地址可以赋值，可以暂时为指针赋值为NULL（*未初始化的变量会存在垃圾值，会导致程序难以调试*）

##### 空指针判断

```cpp
int  *ptr = NULL;
// 如果pr指针非空
if (pr) {}
// 如果pr指针为空
if (!pr) {}
```

#### 指针的算数运算

指针是一个用16进制数值表示的地址，因此我们可以对指针进行算数运算（++ -- + -）

##### 指针递增运算

```cpp
int int_array[MAX] = {10, 100, 1000};
int *ptr = NULL;

// 数组地址赋值给指针
ptr = int_array;
for (int i = 0; i < MAX; ++i) {
    cout << "指针地址：" << ptr << endl;
    cout << "指针对应的数值：" << *ptr << endl;
    ptr ++;
}
```

##### 指针递减运算

```cpp
int int_array[MAX] = {10, 100, 1000};
int *ptr = NULL;

// 数组地址赋值给指针
ptr = &int_array[MAX - 1];
for (int i = MAX; i > 0; --i) {
    cout << "指针地址：" << ptr << endl;
    cout << "指针对应的数值：" << *ptr << endl;
    ptr --;
}
```

##### 指针的比较

指针可以用关系用算符进行比较，如果p1和p2指向的是两个相关的变量，例如是同一个数组中不同的数值，可以对p1和p2进行大小比较（实际上比较的是16进制内存地址）

```cpp
int  var[MAX] = {10, 100, 200};
int  *ptr;

// 指针中第一个元素的地址
ptr = var;
int i = 0;
while ( ptr <= &var[MAX - 1] ){
    cout << "Address of var[" << i << "] = ";
    cout << ptr << endl;

    cout << "Value of var[" << i << "] = ";
    cout << *ptr << endl;

    // 指向上一个位置
    ptr++;
    i++;
}
```

#### 指针VS数组

很多情况下指针和数组是可以互换的，例如一个指向数组开头的指针，可以通过指针的算数运算来遍历数组

#### 指针数组

```cpp
// 在这里把ptr声明成一个数组，由MAX个整形指针组成。因此ptr中的每个元素都是指向int类型的指针。
int *ptr[MAX];
```

```cpp
void pointArray() {
    int array[MAX] = {1, 2, 3};
    // 声明一个指针数组
    int *ptr_array[MAX];
    
    // 给指针数组赋值
    for (int i = 0; i < MAX; ++i) {
        ptr_array[i] = &array[i];
    }

    for (int i = 0; i < MAX; ++i) {
        cout << "address：" << ptr_array[i] << " ";
        cout << "value：" << *ptr_array[i] << endl;
    }
}
```

#### 指向指针的指针

#### 传递指针给函数

#### 从函数返回指针

