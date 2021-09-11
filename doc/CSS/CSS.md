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

1. 链接伪类使用说明

   link： 表示作为超链接，并指向一个**未访问**的所有地址的锚

   visited: 表示作为超链接，并指向一个**已访问**的所有地址的锚

2. 使用方式

   ```html
   <style type="text/css">
       a{
           text-decoration: none;
       }
       a:link{
           color: deeppink;
       }
       a:visited{
           color: pink;
       }
   </style>
   
   <a href="#">点我点我点我</a>
   ```

3. 注意事项

   超链接标签使用时需要遵循 `l(link)v(visited)h(hover)a(actived)原则`，即link和visited的使用必须用在hover和actived的前面，因为link和visited伪类选择器已经包含了a标签的所有的样式，如果这两个伪类选择器放在了hover和actived之后会覆盖掉他们的样式

4. target的使用

   target可以代表一个id选择器组件

   ```html
   <style type="text/css">
       *{
           margin: 0;
           padding: 0;
       }
       a{
           text-decoration: none;
       }
       div{
           width: 200px;
           height: 200px;
           background: pink;
           display: none;
           text-align: center; /* 水平居中 */
           font: 50px/200px "微软雅黑";
       }
       :target{
           display: block;
       }
   </style>
   
   <body>
       <a href="#div1">div1</a>
       <a href="#div2">div2</a>
       <a href="#div3">div3</a>
   
       <div id="div1">div1</div>
       <div id="div2">div2</div>
       <div id="div3">div3</div>
   </body>
   ```

##### 动态伪类

1. hover

   鼠标悬浮到元素上会触发该伪类选择器

   ```css
   a:hover{
   	font-size: 40px;
   }
   ```

2. active

   鼠标点击时会触发该伪类选择器

   ```css
   a:active{
       font-size: 50px;
   }
   ```

##### 表单相关伪类

1. enabled 匹配可编辑的表单

   ```css
   input:enabled{
   	background: hotpink;
   }
   ```

2. disabled 匹配被禁用的表单

   ```css
   input:disabled{
   	background: gray;
   }
   ```

3. checked 匹配被选中的表单

   ```css
   input:checked{
       width: 200px;
       height: 200px;
   }
   ```

4. focus 匹配获得焦点的表单

   ```css
   input:focus{
       background: pink;
   }
   ```

##### 结构性伪类

#### 生命的优先级

### 自定义字体

### 新的UI方案

### 过渡

### 2D/3D变形

### 动画

### 布局扩展