#### 视口

##### 概念

视口（ViewPort）是用户在网页上的可见区域，视口的大小随设备而异

##### 设置视口

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

#### 网格视图

##### 概念

许多网页都是基于响应式布局设计的，这就意味着可以更轻松地在网页上放置元素

##### 构建响应式网格视图

1. 首先设置全局的CSS样式

   ```css
   /*确保元素的实际高度 = 内容高度 + 内边距 + 边框宽度*/
   * {
       box-sizing: border-box;
   }
   ```

2. 

#### 媒体查询

##### 概念

##### 示例

#### 图像

##### 使用 width 属性

如果 width 属性设置为百分比，且高度设置为 "auto"，则图像将进行响应来放大或缩小：

```css
img {
  width: 100%;
  height: auto;
}
```



#### 视频