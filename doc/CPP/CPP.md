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

**概念：**

指向指针的指针是一种多级间接寻址的形势，或者说是一个指针链。

![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.7gmj89nzbjk.png)

**声明**

```cpp
int **cpp
```

**代码实现**

```cpp
/**
 * 指向指针的指针(也称为指针链)
 */
void point_chain() {
    int value = 1;
    int *ptr = NULL;
    int **pptr = NULL;

    // 获取value的地址
    ptr = &value;
    // 或者指针ptr的地址
    pptr = &ptr;

    cout << "value的值：" << value << endl;
    cout << "*ptr address:" << ptr << "  *ptr value: " << *ptr << endl;
    cout << "**pptr address:" << pptr << "  **pptr value: " << **pptr << endl;
}
```

#### 传递指针给函数

```cpp
int main() {
    int array[MAX] = {1, 2, 3};
    double average = 0;
    pass_point_to_function(array, MAX, &average);
    cout << "the array average value is: " << average;
}

/**
 * 求数组平均值(传递指针给函数)
 * @param array 数组
 * @param size 数组容量
 * @param average 平均数
 */
void pass_point_to_function(const int *array, int size, double *average) {
    double sum = 0;
    for (int i = 0; i < MAX; ++i) {
        sum += array[i];
    }
    *average = sum / size;
}
```

#### 从函数返回指针

C++不允许在函数外返回局部变量的地址，除非定义变量为static变量

```cpp
int main() {
    int *ptr = get_random_array();
    for (int i = 0; i < 10; ++i) {
        cout << *(ptr + i) << endl;
    }
}

/**
 * 获取随机数(从函数返回指针)
 * @return
 */
int * get_random_array() {
    const int length = 10;
    static int array[length];
    srand((unsigned) time(NULL));
    for (int i = 0; i < length; ++i) {
        array[i] = rand();
    }
    return array;
}
```

### 引用

#### 概念

引用可以理解为变量的另一个别名，即它是某个已经存在的变量的另一个名。

#### 应用场景

把引用作为函数参数可以减少数值传递过程中的时间

```cpp
int num = 1;
// 声明引用变量
int& num_ref = num;
int num_copy = num;
cout << "num: " << num << endl;
cout << "num_ref: " << num_ref << endl;
cout << "num_copy: " << num_copy << endl;


cout << "address num: " << &num << endl;
cout << "address num_ref: " << &num_ref << endl;
cout << "address num_copy: " << &num_copy << endl;
```

#### 引用VS指针

1. 引用不能为空指针可以为空指针，引用必须链接到一块合法的内存地址
2. 引用可以理解为静态常量， 不能修改，一旦引用被初始化为一个对象， 就不能修改为另一个对象
3. 引用必须在创建的时候初始化，指针可以在任何时间初始化。

#### 把引用作为参数

#### 把引用作为返回值

### 结构体

结构体应用场景：存储不同数据类型的数据项

我们可以使用成员访问符 . 访问结构体成员变量

```cpp
struct Book {
    char title[50];
    char author[50];
    int book_id;
};

int main() {
    // 定义结构体Book 类型变量book1
    Book book1;
    strcpy(book1.author, "赵四");
    strcpy(book1.title, "成为亚洲舞王的诀窍");
    book1.book_id = 1;

    // 输出结构体book1变量
    cout << book1.book_id << endl;
    cout << book1.title << endl;
    cout << book1.author << endl;
}
```

#### 结构体做为函数参数

```cpp
/**
 * 打印书籍(结构体作为函数参数)
 * @param book
 */
void print_book(struct Book book) {
    // 输出结构体book1变量
    cout << book.book_id << endl;
    cout << book.title << endl;
    cout << book.author << endl;
}
```

#### 指向结构体的指针

```cpp
// 定义指向结构体的指针
struct Book *struct_book_pointer;

// 结构体指针赋值
struct_book_pointer = &book1;
```

```cpp
/**
 * 打印书籍信息(结构体指针的使用)
 * 为了使用结构体指针访问成员变量，必须使用—> 访问成员变量
 * @param book
 */
void print_book(struct Book *book) {
    cout << book->book_id << endl;
    cout << book->author << endl;
    cout << book->title << endl;
}
```

#### typedef关键字

可以使用typedef关键字为结构体类型起别名，定义更加简洁的数据类型

```cpp
typedef struct Book {
    char title[50];
    char author[50];
    int book_id;
} Book;
```

### 类&对象

```cpp
class Box {
public:
    // 成员变量声明
    double length;
    double breadth;
    double height;

    // 成员函数声明
    double get();
    void set(double length, double breadth, double height);
};

double Box::get() {
    return length * breadth * height;
}

void Box::set(double length, double breadth, double height) {
    this->breadth = breadth;
    this->length = length;
    this->height = height;
}
```

#### 类成员函数

#### 类访问修饰符

1. private
2. public
3. protected

#### 构造函数&析构函数

1. 构造函数

2. 析构函数

   类的析构函数是一种特殊的成员函数，它会在每次删除创建的对象时执行。**析构函数有助于在退出程序之前释放资源（关闭文件、释放内存）**

   ```cpp
   class Line {
   private:
       double length;
   public:
       void setLength(double length) {
           this->length = length;
       }
       double getLength() {
           return this->length;
       }
       Line(double length) {
           this->length = length;
           cout << "Line object is created" << endl;
       }
       ~Line() {
           cout << "Line object is destroyed" << endl;
       }
   };
   
   int main() {
       Line line(100.0);
       cout << "the line'length is " << line.getLength() << endl;
   }
   ```


#### 拷贝构造函数

**概念**

拷贝构造函数是一种特殊的构造函数，它在创建对象时，使用的是同一类中之前创建的对象来初始化新创建的对象（和原型模式很类似）。

**应用场景**

1. 通过使用同类型的对象来初始化新创建的对象
2. 复制对象把它作为参数传递给函数
3. 复制对象，并从函数返回这个对象

**代码实现**

#### 友元函数

**概念**

类的友元函数是定义在类的外部， 但是有权访问类的私有（**private**）成员和受保护（**protected**）成员。尽管友元函数的原型在类中声明过，但是**友元函数不是成员函数**。

友元可以是一个函数，该函数称为友元函数；友元可以是一个类，该类称为友元类，在这种情况下，整个类的所有成员都是友元。

**友元关键字**

```cpp
friend void printWidth();
```

**代码实现**

```cpp
class Student {
private:
    int age;
public:
    void setAge(int age) {
        this->age = age;
    }
    // 友元函数的定义
    friend void printAge(Student student);
};

/**
 * 注意，友元函数不是任何类的函数
 * @param student
 */
void printAge(Student student) {
    // 注意：友元函数可以直接访问该类的任何成员
    cout << "the student age is :" << student.age << endl;
}

int main() {
    Student student{};
    student.setAge(18);
    printAge(student);
}
```

#### 内联函数

#### this指针

在C++中， 每个成员都能够通过this指针来访问自己的地址

```cpp
int compare(Box box){
    return this->Volume() > box.Volume();
}
```

#### 指向类的指针

一个指向类的指针和指向结构的指针类似，访问指向类的指针需要使用成员访问符号 -> 

```cpp
int main(void)
{
   Box Box1(3.3, 1.2, 1.5);    // Declare box1
   Box *ptrBox;                // Declare pointer to a class.
   // 保存第一个对象的地址
   ptrBox = &Box1;
   // 现在尝试使用成员访问运算符来访问成员
   cout << "Volume of Box1: " << ptrBox->Volume() << endl;
}
```

#### 静态成员

可以使用static关键字来定义静态成员变量，无论创建了多少类对象，静态成员都只有一个副本

```cpp
class Box {
public:
    // 静态成员变量声明
     static int objectCount;
}

// 初始化Box的静态成员变量
int Box::objectCount = 0;

int main(void) {
   // 调用静态成员变量
   cout << "Total objects: " << Box::objectCount << endl;
}
```

### 继承

```cpp
class Shape {
    
}

class Rectangle: public Shape {
   
}
```

### 重载运算符&重载函数

### 多态

**虚函数**

如果基类中area函数没有被virtual关键字修饰，编译器绑定的的是基类的函函数，调用派生类的area函数时，编译器将会调用基类的area函数，这会导致程序出错。这种现象被称之为**静态多态**，或者**早绑定**。

但是用**virtual**关键字修饰基类中的方法时，就是动态绑定， 也称为晚绑定。

```cpp
/**
 * 基类
 */
class Shape {
    protected:
        double height, width;
    public:
        Shape(double height, double width) {
            this->height = height;
            this->width = width;
        }

        virtual double area() {
            cout << "bass class area function is called.";
            return 0.0;
        }
};

/**
 * 矩形派生类
 */
class Rectangle: public Shape {
    public:
        Rectangle(double height, double width): Shape(height, width) { }
        double area() override {
            cout << "Rectangle class area function is called." << endl;
            return height * width;
        }

};

/**
 * 三角形派生类
 */
class Triangle: public Shape {
    public:
        Triangle(double height, double width): Shape(height, width) { }
        double area() override{
            cout << "Triangle class area function is called." << endl;
            return height * width / 2;
        }
};

int main() {
    Shape *shape;
    Rectangle rectangle(2, 2);
    Triangle triangle(2, 2);

    // 矩形类地址
    shape = &rectangle;
    // 调用矩形类的area函数
    const double rectangle_area = shape->area();
    cout << "矩形面积：" << rectangle_area << endl;

    // 三角形类地址
    shape = &triangle;
    // 调用三角形类的area函数
    const double triangle_area = shape->area();
    cout << "三角形面积：" << triangle_area << endl;
}
```

**纯虚函数**

纯虚函数也可以理解为是一个抽象的虚函数。

```cpp
class Shape {
   protected:
      int width, height;
   public:
      Shape( int a=0, int b=0){
         width = a;
         height = b;
      }
      // pure virtual function
      virtual int area() = 0;
};
```

### 数据抽象

### 数据封装

### 接口&抽象类

### 文件和流

### 异常处理

```cpp
double division(int a, int b);

int main() {
    int a = 1;
    int b = 0;
    double result = 0.0;
    try {
        result = division(1, 0);
        cout << "result: " << result << endl;
    } catch (const char *error) {
        cerr <<  "异常信息：" << error << endl;
    }
    return 0;
}

double division(int a, int b) {
    if (b == 0) {
        throw "Division by zero condition!";
    }
    return a / b;
}
```

#### 自定义异常

```cpp
class MyException: public exception {
public:
    const char* what() const throw() override {
        return "c++ exception";
    }
};

int main() {
    try {
        throw MyException();
    } catch (MyException& e) { // MyException& 声明一个异常类的引用
        cout << "MyException caught" << endl;
        cout << e.what() << endl;
    }
}
```

### 动态内存

#### C++内存类型

1. 栈内存

   在函数内部声明的所有局部变量都将占用栈内存

2. 堆内存

   堆内存是程序未使用的内存，用于在程序运行期间动态分配内存

#### 操作内存的运算符

```cpp
// 初始化一个空指针
double *ptr = NULL;
// 为ptr指针分配内存
ptr = new double;

// 分配内存检查
doube *ptr = NULL;
if (!(ptr = new double)) {
    cerr << "Error: out of memory" << endl;
    exit(1);
}

// 释放ptr指针所占用的内存
delete ptr;
```

#### 数组的动态内存分配

```cpp
// 数组动态分配内存
char *ptr_arrary = new char[20]; 
// 删除ptr_arrary指针所占用的内存
delete [] ptr_arrary;
```

**二维数组动态分配内存**

```cpp
int main() {
   int **p;
   int i,j;
   // 开始分配4行8列二维数组内存
   p = new int *[4];
    for (i = 0; i < 4; ++i) {
        p[i] = new int[8];
    }

    for (i = 0; i <4; ++i) {
        for (j = 0; j < 8; ++j) {
            p[i][j] = i * j;
        }
    }

    // 打印数据
    for (i = 0; i <4; ++i) {
        for (j = 0; j < 8; ++j) {
            if (j == 0) {
                cout << endl;
            }
            cout << p[i][j] << "\t";
        }
    }

    // 开始释放申请的堆内存
    for (i = 0; i < 4; ++i) {
        delete [] p[i];
    }
    delete [] p;
    return 0;
}
```

#### 对象的动态内存分配

```cpp
class Student {
public:
    Student() {
        cout << "Construction method 被调用" << endl;
    }
    ~Student() {
        cout << "Deconstruction method被调用" << endl;
    }

};
int main() {
    Student* students = new Student[4];
    delete [] students;
}

Construction method 被调用
Construction method 被调用
Construction method 被调用
Construction method 被调用
Deconstruction method被调用
Deconstruction method被调用
Deconstruction method被调用
Deconstruction method被调用
```

### 命名空间

命名空间用来区分不同库中的相同的类、函数、变量（和Java中的package类似）

```cpp
namespace first_space {
    void fun() {
        cout << "first_space: fun is called" << endl;
    }
}

namespace second_space {
    void fun() {
        cout << "second_space: fun is called" << endl;
    }
}

int main() {
    second_space::fun();
}
```

### 模板

#### 函数模板

```cpp
/**
 * inline：表示这个函数是内联函数(提高程序的运行效率，内联函数在程序编译时就会在被调用的地方进行替换，以空间换时间，目的是为了提高程序的运行效率)
 * const：常量参数(常量参数保证在使用时不能被改变)
 * const&: 引用常量参数(引用参数的目的是为了减少数值传递过程中消耗的时间，如果不用引用参数，编译器会重新申请一块内存空间)
 */
template <typename T> inline T max_value(T const& a, T const& b) {
    return  a > b ? a : b;
}

int main() {
    int i = 30;
    int j = 20;
    cout << "max num is: " << max_value(i, j) << endl;

    double f1 = 13.5;
    double f2 = 20.7;
    cout << "Max(f1, f2): " << max_value(f1, f2) << endl;

    string s1 = "Hello";
    string s2 = "World";
    cout << "Max(s1, s2): " << max_value(s1, s2) << endl;
}
```

#### 类模板

```cpp
template <class T>
class MyStack {
private:
    vector<T> list;

public:
    /**
     * 出栈
     */
    void push(T const&);

    /**
     * 出栈
     */
    void pop();

    /**
     * 返回栈顶元素(副本)
     * @return
     */
    T top() const;

    int isEmpty() const {
        return list.empty();
    }
};

template <class T>
void MyStack<T>::push(T const& element) {
    list.push_back(element);
}

template <class T>
void MyStack<T>::pop(){
    if (list.empty()) {
        throw out_of_range("Stack<>::pop(): empty stack");
    }
    list.pop_back();
}

template <class T>
T MyStack<T>::top() const {
    if (list.empty()) {
        throw out_of_range("Stack<>::pop(): empty stack");
    }
    // 返回最后一个元素的副本
    return list.back();
}


int main() {
    try {
        // int类型的栈
        MyStack<int> int_stack;
        // string类型的栈
        MyStack<string> string_stack;

        // 操作int类型的栈
        int_stack.push(7);
        cout << int_stack.top() << endl;
        cout << int_stack.isEmpty() << endl;


        // 操作string类型的栈
        string_stack.push("hello");
        cout << string_stack.top() << endl;
        string_stack.pop();
        string_stack.pop();
        cout << "测试输出" << endl;

    } catch (exception const& ex) {
        cerr <<  "exception: " << ex.what() << endl;
    }
}
```

### 预处理器

预处理器实际上是一些指令，指示编译器在实际编译之前做的一些操作

#### #define 预处理

```cpp
// 创建符号常量
#define PI 3.14159
```

#### 参数宏

可以使用#define来定义一个带有参数的宏

```cpp
#define MIN(a,b) (a < b ? a : b)
```

#### 条件编译

有几个指令可以用来有选择地对部分程序源代码进行编译。这个过程被称为条件编译。

```cpp
#if 0
    // 不进行编译的代码
#endif
```

#### #和##运算符

#### C++中预定义宏

| 宏        | 功能         |
| --------- | ------------ |
| \__LINE__ | 显示当前行号 |
| \__FILE__ | 显示文件名   |
| \__DATE__ | 显示日期     |
| \__TIME__ | 显示时间     |

### 信号处理

### 多线程

#### 创建线程

```c++
#include <iostream>
#include <pthread.h>

using namespace std;

#define THREAD_NUM 5

typedef struct thread_data {
    int thread_id;
    char *message;
} thread_data;

void* say_hello(void *data_ptr) {
    thread_data *my_data = (thread_data *) data_ptr;
    cout << "thread_id: " << my_data->thread_id << endl;
    cout << "message: " << my_data->message << endl;
    pthread_exit(nullptr);
}

int main() {

    int result_code = -1;
    // 生命线程id的变量
    pthread_t thread_ids[THREAD_NUM];
    // 用数组来保存i的值
    int index[THREAD_NUM] = {};
    thread_data thread_data_array[THREAD_NUM];

    for (int i = 0; i < THREAD_NUM; ++i) {
        cout <<"main() : creating thread, " << i << endl;
        thread_data_array[i].thread_id = i;
        thread_data_array[i].message = (char *)(&"this is message " [ i]);
        index[i] = i;
        // 传入的时候必须强制转换为void* 类型，即无类型指针
        result_code = pthread_create(&thread_ids[i], nullptr, say_hello, (void *)&(thread_data_array[i]));
        if (result_code != 0) {
            cout << "创建线程失败, error_code: " << result_code << endl;
            exit(-1);
        }
    }
    // 等待所有线程退出之后，进程才退出, 避免出现进程结束子线程还没有结束的情况
    pthread_exit(nullptr);
}
```

#### 终止线程

#### 向线程传递参数

#### 连接和分离线程

### STL教程

#### 容器

#### 迭代器
