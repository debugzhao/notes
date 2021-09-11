## CSS3教程

### 选择器

#### 基本选择器

1. 通配符选择器

   *

2. 元素选择器

   body（任何一个HTML标签）

3. 类选择器

   .

4. ID选择器

   #

5. 后代选择器

   空格

6. 基本选择器的扩展

   1. 子元素选择器
   2. 相邻兄弟选择器
   3. 通用兄弟选择器

#### 属性选择器

1. 存在属性选择器

   ```html
   <style>
       div[name] {
           border: 1px solid;
       }
   </style>
   
   <div id="wrap">
       <div name="student stu1">学生</div>
       <div name="student_zhangsan">张三</div>
       <div name="student_lisi">李四</div>
   </div>
   ```

2. 值属性选择器

   ```html
   <style>
       div[name~="student"] {
           border: 1px solid;
       }
   </style>
   ```

3. 子串值属性选择器

   [attr|=val] : 选择attr属性的值是val（包括val）或以val-开头的元素。

   [attr^=val] : 选择attr属性的值以val开头（包括val）的元素。

   [attr$=val] : 选择attr属性的值以val结尾（包括val）的元素

   [attr*=val] : 选择attr属性的值中包含字符串val的元素。

#### 伪类与伪元素选择器

##### 链接伪类

##### 动态伪类

##### 表单相关伪类

##### 结构性伪类

#### 生命的优先级

### 自定义字体

### 新的UI方案

### 过渡

### 2D/3D变形

### 动画

### 布局扩展