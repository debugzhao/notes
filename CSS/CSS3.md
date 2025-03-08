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

1. 

2. 存在属性选择器

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

3. 值属性选择器

   ```html
   <style>
       div[name~="student"] {
           border: 1px solid;
       }
   </style>
   ```

4. 子串值属性选择器

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

1. nth-child

   :nth-child(index)

   ```css
   /* 后代选择器(选中wrap选择器下面的所有层级的li标签) */
   /* 结构伪类选择器：选中wrap下面的第一个标签，并且这个标签是li标签 */
   #wrap li:nth-child(1){
       color: pink;
   }
   ```

   :first-child

   :last-child

   :nth-last-child(index)

   :only-child

2. nth-of-type

   :nth-of-type(index)

   ```css
   /* 后代选择器(选中wrap选择器下面的所有层级的li标签) */
   /* 结构伪类选择器：选中wrap下面的第一个li标签*/
   #wrap li:nth-of-type(1){
       color: pink;
   }
   ```

   :first-of-type

   :last-of-type

   :nth-last-type(index)

   only-of-type

3. 结构性伪类使用注意事项

   index的值从1开始计数

4. not（不选中某一个标签）

   1. 案例

      使每一个a标签右侧有一个border，但是最后一个a标签除外 

   2. 效果图

      ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.1z1pta1axf8g.png)

   3. 代码

      ```html
      <head>
          <meta charset="utf-8">
          <title></title>
          <!-- 使每一个a标签右侧有一个border，但是最后一个a标签除外 -->
          <style type="text/css">
              *{
                  margin: 0;
                  padding: 0;
                  border: none;
              }
              a{
                  font: 20px "微软雅黑";
                  text-decoration: none;
                  color: pink;
                  display: block;
                  float: left;
                  width: 100px;
                  height: 30px;
              }
              a:not(:last-of-type) {
                  border-right: 1px solid #000000;
              }
          </style>
      </head>
      <body>
          <div>
              <a href="#div1">div1</a>
              <a href="#div2">div2</a>
              <a href="#div3">div3</a>
              <a href="#div4">div4</a>
              <a href="#div5">div5</a>
          </div>
      </body>
      ```

5. empty

   1. 应用场景

      选中某一个空内容元素。empty选中的标签内容必须是空的，有空格不会选中该标签，但是有空格并且有属性这种情况可以选中

   2. 代码

      ```html
      <head>
          <meta charset="utf-8">
          <title></title>
          <style type="text/css">
              div{
                  height: 200px;
                  background: #90EE90;
              }
              div:empty{
                  background: #ff007f;
              }
          </style>
      </head>
      <body>
          <div>
              <div></div>
              <div>Second</div>
              <div></div>
              <div>Third</div>
          </div>
      </body>
      ```

##### 伪元素

DOM树是解析HTML标签生成的，伪元素是解析CSS生成的，因此通过伪元素生成的标签不在DOM树里面，即DOM树不能直接操作伪元素生成的标签

一个标签最多只能有两个伪元素，一个是before，一个是after

1. after伪元素

   1. 应用场景

      一般after伪元素是用来清除浮动的

   2. 截图示例

      ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.21c89ww2msow.png)

   3. 代码

      ```html
      <head>
          <meta charset="utf-8">
          <title>::after伪元素</title>
          <style type="text/css">
              #wrap::after{
                  height: 200px;
                  width: 200px;
                  background: pink;
                  display: block;
                  content: "";
              }
          </style>
      </head>
      <body>
          <div id="wrap">div</div>
      </body>
      ```

2. before伪元素

3. first-letter

   选中第一个汉字或者第一个字母

4. first-line

   选中第一行

5. selection

   选中被鼠标左击选中的内容

#### 生命的优先级

### 自定义字体

### 新的UI方案

#### 渐变

```css
/* 线性渐变，改变渐变方向 */
background-image: linear-gradient(to right, red, yellow, green);
/* 改变渐变角度 */
background-image: linear-gradient(90deg, red, yellow, green);
```



### 过渡

### 2D/3D变形

### 动画

### 布局扩展