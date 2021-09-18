## 13.组件化开发之组件通信

### 04.父传子props数组用法

子组件props数组方式定义

```html
<template>
  <div>
    <h2>{{title}}</h2>
    <p>{{content}}</p>
  </div>
</template>
```

```javascript
props: ['title', 'content']
```

#### 父组件传值的三种方式

1. 直接传递静态变量

   ```html
   <detail title="111" content="111111"></detail>
   ```

2. 传递data中的变量

   通过: 冒号方式传递一个变量

   ```html
   <detail :title="appTitile" :content="appContent"></detail>、
   ```

3. 传递对象

   如果对象中的属性名称和子组件中得props一致的话，可以直接传递对象

   ```html
   <detail v-bind="Obj"></detail>
   ```

   ```javascript
   data() {
       return {
           Obj: {
               title: "333",
               content: "333333"
           }
       }
   }
   ```

### 05.父传子props对象用法

### 06.非prop的attribute

