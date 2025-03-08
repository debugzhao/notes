1. #### R文件生成失败情况

   ![](C:\Users\Administrator\Desktop\md\Android\image\R文件生成失败.png)

   解决方式：Build--->clean Project

2. #### res目录 

   1. drawable目录

      存放位图文件(PNG、JPG、GIF)

      9 Patch文件（9 Patch工具生成的）

      ​	适合用作背景图片（拉伸不变形）

   2. layout目录（布局文件）

      activity_main.xml

   3. mipmap目录

      存放系统启动图标文件

      存放规则：需要适配系统分辨率的存放该目录下，减少性能浪费 

   4. values目录

      保存资源文件（字符串资源、颜色资源、样式资源）



---

### Android布局

### 自定义View

```java
//自定义RabbitView
RabbitView extends View
//创建RabbitView构造方法

MainActivity类
//创建FrameLayout布局
FrameLayout frameLayout = findViewById(R.id.mylayout);
//创建RabbitView对象
final RabbitView rabbit = new RabbitView(this);
//将自定义的view对象添加布局中
frameLayout.addView(rabbit);
```

### 布局管理器

- 作用

  控制组件是如何摆放的（可以适配任何分辨率的屏幕）

- 分类

  相对布局

  线性布局

  帧布局

  表格布局

  ~~绝对布局（不方便屏幕自适应，已被弃用）~~

  网格布局（可以代替表格布局）

#### 相对布局

##### 相对布局常用属性

| 于组件参考位置   | 于layout对齐方式  | 于组件对齐方式 | 与layout居中方式 |
| ---------------- | :---------------- | -------------- | ---------------- |
| layout_above     | alignParentBotton | alignBottom    | centerHorizontal |
| layout_below     | alignParentLeft   | alignLeft      | centerVertical   |
| layout_toLeftOf  | alignParentRight  | alignRight     | centerInParent   |
| layout_toRightOf | alignParentTop    | alignTop       |                  |

#### 线性布局

概念：进行垂直、水平布局管理

常用属性：

| orientation        | gravity(显示位置)        |
| ------------------ | ------------------------ |
| horizontal水平线性 | center                   |
| vertical 垂直线性  | right\|left\|bottom\|top |

```java
//平均分配剩余空间的尺寸
android:layout_weight="1"
```

padding 内边距

#### 帧布局

概念：一帧一帧层层覆盖 

| foreground                     | foregroundGravity |
| ------------------------------ | ----------------- |
| 为当前帧布局管理器设置前景图像 | 设置前景图像位置  |

#### 表格布局

#### 网格布局

特点：即可跨行、又可跨列

常用属性

#### 嵌套布局





