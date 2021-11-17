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





