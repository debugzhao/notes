### 1 什么是CSS

#### 1.1 样式种类

1. 外部样式
2. 内部样式
3. 行内样式

优先级：行内样式 > 内部样式 > 内部样式（就近原则）

### 2 选择器

#### 2.1基本选择器

1.  标签选择器
2. 类选择器
3. ID选择器

#### 2.2 层次选择器

1. 后代选择器  （改变所有后代的样式，儿子 孙子 曾孙子）

   ```css
   /*改变body下面的所有P标签的样式*/
   body p{
   	background-color: red;
   }
   ```

2. 子选择器 （只改变儿子的样式）

   ```css
   /*改变body下面的所有第一层后代的标签的样式*/
   body>p{
       background-color: red;
   }
   ```

3. 相邻兄弟选择器 （改变相邻下方的第一个兄弟的样式）

   ```html
   <style>
       /*改变body下面的所有P标签的样式*/
       .active + p{
           background-color: red;
       }
   </style>
   
   <body>
       <p>p1</p>
       <p class="active">p2</p>
       <p>p3</p>
       <p>p4</p>
       <ul>
           <li><p>p3.1</p></li>
           <li><p>p3.2</p></li>
           <li><p>p3.3</p></li>
       </ul>
   </body>
   ```

4. 通用兄弟选择器（只改下方，不改对上方兄弟的样式）

   ```html
   <head>
       <style>
           .active ~ p{
               background-color: red;
           }
       </style>
   </head>
   <body>
       <p>p1</p>
       <p class="active">p2</p>
       <p>p3</p>
       <p>p4</p>
       <ul>
           <li><p>p4.1</p></li>
           <li><p>p4.2</p></li>
           <li><p>p4.3</p></li>
       </ul>
       <p>p5</p>
   </body>
   ```

#### 2.3 结构伪类选择器

```html
<style>
    /*选中ul的第一个li标签*/
    ul li:first-child {
        background-color: red;
    }
    /*选中ul的最后一个li标签*/
    ul li:last-child {
        background-color: green;
    }

</style>
```

#### 2.4 属性选择器

```css
<style>
    /*选中存在class属性的标签*/
    a[id] {
        background-color: red;
    }
    /*选中class属性是google的标签*/
    a[class=ame] {
        font-size: 20px
    }
</style>
```

### 3 盒子模型

#### 3.1 内外边距

#### 3.2 圆角边框

```css
.box1 {
    width: 200px;
    height: 100px;
    border: 2px solid red;
    margin: auto;
    border-radius: 100px 100px 0px 0px;
}
```

#### 3.3 盒子阴影 

### 4 浮动

#### 4.1 标准文档流

#### 4.2 display

```css
.box2 {
    width: 100px;
    height: 100px;
    border: 1px solid red;

    /*不显示元素*/
    display: none;
    /*行内元素*/
    display: inline;
    /*将块级元素变成行内元素（不换行），但是保持块级元素的特性（有宽高的值）*/
    display: inline-block;
    /*将行内元素变成块级元素*/
    display: block;
}
```

#### 4.3 float

```css
p {
    display: inline-block;
    /*向左浮动*/
    float: left;
}
```

#### 4.4 clear

```css
p {
    /*左侧不允许有浮动元素*/
    clear: left;
    /*右侧不允许有浮动元素*/
    clear: right;
    /*左右两侧不允许有浮动元素*/
    clear: both;
    
}
```

#### 4.5 父级边框坍塌问题

![image-20210318211517788](https://i.loli.net/2021/03/18/QxBAUIPJzTDNqvy.png)

**解决方案：**

1. 给父级元素增加高度

   ```css
   #father {
       height: 1000px
   }
   ```

2. 在所有元素的最下方增加一个空的div，清除浮动

   ```css
   <div class="clear"></div>
   
   .clear {
       clear: both;
       margin: 0;
       padding: 0;
   }
   ```

3. 使用overflow

   ```css
   div {
       height: 100px;
       width: 100px;
       border: 1px solid red;
       /*超出边框的部分自动隐藏掉*/
       overflow: hidden;
   }
   ```

4. **给父级元素添加一个伪类（推荐使用）**

   ```css
   /*该效果类似方法2*/
   #father::after {
       content: '';
       display: block;
       clear: both
   }
   ```

### 5 定位

#### 5.1 相对定位

*相对定位：相对于原来的自己进行偏移，并且它仍然在标准文档流中，原来的位置会被保留，不会出现位置塌陷的情况 *

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    <style>
        div {
            margin: 10px;
            padding: 5px;
            font-size: 12px;
        }
        .father {
            padding: 0;
            border: 1px solid red;
        }
        .box1 {
            background-color: black;
            border: 1px dashed #6aff00;
            position: relative;
            top: -20px;
        }
        .box2 {
            background-color: blue;
            border: 1px dashed #e21dc1;
            position: relative;
            left: -30px;
        }
    </style>
</head>
<body>

    <div class="father">
        <div class="box1">盒子1</div>
        <div class="box2">盒子2</div>
    </div>
</body>
</html>
```

#### 5.2 绝对定位

1. 父级元素没有进行定位的前提下，相对于浏览器进行定位
2. 如果父级元素存在定位，子级元素通常会相对于父级元素进行定位（并且只在父级元素内移动）
3. **子绝父相**

#### 5.3 固定定位

```css
div:nth-of-type(2) {
    width: 50px;
    height: 50px;
    background-color: yellow;
    position: fixed;
    right: 0;
    bottom: 0;
}
```

#### 5.4 z-index

z-index层级概念，最底层是第0层，最高无限层级

#### box-sizing

1. **content-box**（css默认属性）

   宽度和高度的计算值都不包含内容的边框（border）和内边距（padding）。

2. **border-box**

   **将其应用于所有元素是安全且明智的**

   ```css
   * {
     box-sizing: border-box;
   }
   ```
   
   `width` = border + padding + 内容的宽度
   
   添加了 box-sizing: border-box;属性，以确保框不会由于额外的内边距（填充）而损坏

#### flex： 弹性布局

父级元素定义display:flex,子元素宽度用flex来定义,flex:1 是均分父级元素。占的比例相同

#### 水平居中/垂直居中

##### 水平居中

1. 块级元素水平居中

   设置块级元素在水平方向上居中（**注意这里必须给块级元素设置宽度**）

   ```css
   width: 50%;
   margin: auto;
   ```

2. 元素内的文本水平居中

   ```css
   text-align: center
   ```

3. 行内元素水平居中（**注意：需要将行内元素设置为块级元素**）

   ```css
   display: block;
   margin: 0 auto;
   ```

4. 左对齐/右对齐

   **通过绝对定位实现（如果父级元素没有设置定位，则该元素相对于浏览器进行定位）**

   ```css
   position: absolute;
   right: 0
   ```

   **通过float浮动实现（注意父级边框坍塌的问题）**

   ```css
   float: left
   ```

##### 垂直居中

1. 设置固定的上下边距

   ```css
   padding: 70px 0;
   ```

2. 通过line-height属性实现

   注意：line-height的属性值必须和height属性值相等

   ```css
   height: 100px;
   line-height: 100px;
   ```

3. 使用弹性盒模型实现

   ```css
   display: flex;
   justify-content: center; // 水平方向居中
   align-items: center:  // 垂直方向居中
   ```

#### 选择器

##### 后代选择器（空格）

**后代选择器匹配指定元素的所有指定后代**

```css
div p {
    
}
```

##### 子选择器（>）

子选择器匹配指定元素所有的**子**级元素，**孙子元素不算**

```css
div > p {
    
}
```

##### 相邻兄弟选择器（+）

相邻兄弟选择器匹配所有作为指定元素的相邻同级的元素。兄弟（同级）元素必须具有相同的父元素，“相邻”的意思是“**紧随其后**”。

```css
// 下面的例子选择紧随 <div> 元素之后的第一个 <p> 元素：
div + p {
  background-color: yellow;
}
```

##### 通用兄弟选择器（~）

通用兄弟选择器匹配的是指定元素的**所有指定同级兄弟元素**（只有在指定元素的后方兄弟元素才会被选中）

#### 伪类（:）

##### 概念

伪类用于定义元素的特殊状态

##### 应用场景

1. 设置鼠标悬停在元素上的样式
2. 为已访问链接和未访问链接设置不同的样式
3. 设置元素获取焦点时的样式

##### 实例

1. 锚伪类

   *`a:hover` 必须在 CSS 定义中的 `a:link` 和 `a:visited` 之后，才能生效！`a:active` 必须在 CSS 定义中的 `a:hover` 之后才能生效！伪类名称对大小写不敏感。*

   ```css
   /* 未访问的链接 */
   a:link {
     color: #FF0000;
   }
   
   /* 已访问的链接 */
   a:visited {
     color: #00FF00;
   }
   
   /* 鼠标悬停链接 */
   a:hover {
     color: #FF00FF;
   }
   
   /* 已选择的链接 */
   a:active {
     color: #0000FF;
   }
   ```

2. 简单的提示悬停的工具

   ```css
   p {
     display: none;
     background-color: yellow;
     padding: 20px;
   }
   
   div:hover p {
     display: block;
   }
   ```

3. :first-child

   ```css
   /* 匹配第一个p元素，忽略层级关系的限制 */
   p:first-child {
     color: blue;
   }
   ```

4. 通过后代选择器选中第一个后代

   ```css
   p i:first-child {
     color: blue;
   }
   ```

5. 通过伪类选择器选中第一个元素，然后再通过后代选择器选中第一个元素的所有后代

   ```css
   p:first-child i {
       color: bule;
   }
   ```

##### [所有的CSS伪类](https://www.w3school.com.cn/css/css_pseudo_classes.asp)

#### 伪元素（::）

##### 概念

伪元素用于设置元素的指定部分样式

##### 应用场景

1. 设置元素的首字母、首行的样式
2. 在元素的内容之前、内容之后插入

##### 实例

1. 设置P标签的第一行元素的样式

   ```css
   p::first-line {
     color: #ff0000;
     font-variant: small-caps;
   }
   ```

2. 为每一个h1元素之前插入一张图片

   ```css
   h1::before {
       content: url(smile.gif)
   }
   ```

3. 设置用户选择元素的样式

   ```	css
   ::selection {
       color: red;
       background: yellow;
   }
   ```

所有CSS3的伪元素

| 选择器         | 例子            | 例子描述                      |
| -------------- | --------------- | ----------------------------- |
| ::after        | p::after        | 在每个 <p> 元素之后插入内容。 |
| ::before       | p::before       | 在每个 <p> 元素之前插入内容。 |
| ::first-letter | p::first-letter | 选择每个 <p> 元素的首字母。   |
| ::first-line   | p::first-line   | 选择每个 <p> 元素的首行。     |
| ::selection    | p::selection    | 选择用户选择的元素部分。      |

#### 不透明度

##### 应用场景

指定元素的透明度/不透明度

##### 实例

1. 悬停透明效果

   ```css
   img {
       opacity: 0.5
   }
   img:hover {
       opacity: 1.0
   }
   ```

#### 导航栏

##### 垂直导航栏

##### 水平导航栏

#### 下拉菜单

#### 属性选择器

##### 所有CSS属性选择器

| 选择器            | 案例                | 案例描述                                                |
| ----------------- | ------------------- | ------------------------------------------------------- |
| [attr]            | [target]            | 选择带有 target 属性的所有元素。                        |
| [attr= "value"]   | [target=_blank]     | 选择带有 target="_blank" 属性的所有元素。               |
| [attr~= "value"]  | [title~=flower]     | 选择带有包含 "flower" 一词的 title 属性的所有元素。     |
| [attr\|= "value"] | [lang\|=en]         | 选择带有以 "en" 开头的 lang 属性的所有元素。            |
| [attr^= "value"]  | a[href^="https"]    | 选择其 href 属性值以 "https" 开头的每个 <a> 元素。      |
| [attr$= "value"]  | a[href$=".pdf"]     | 选择其 href 属性值以 ".pdf" 结尾的每个 <a> 元素。       |
| [attr*= "value"]  | a[href*="w3school"] | 选择其 href 属性值包含子串 "w3school" 的每个 <a> 元素。 |

#### 计数器

#### 网站布局

#### 单位

#### 特异性













