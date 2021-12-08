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

2. 

#### Null指针

#### 指针的算数运算

#### 指针VS数组

#### 指针数组

#### 指向指针的指针

#### 传递指针给函数

#### 从函数返回指针

