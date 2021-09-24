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

1. 对象或者数组默认值问题

   ```javascript
     // 注意这里如果props属性是对象类型的话，default默认值返回的是函数
     // 多个组件返回的是不同的对象
     // 否则会出现多个组件引用同一个对象的问题
     info: {
   	type: Object,
   	// 对象或者数组必须从一个工厂函数里面获取
   	default() {
   	  return {name: "lucas"}
   	}
     }
   ```

2. 自定义验证函数

   ```javascript
   props: {
     content: {
   	type: String,
   	required: true,
   	default: "123",
   	// 自定义验证函数
   	validator(value) {
   	  return ["success", "warning", "danger"].includes(value)
   	}
     }
   }
   ```

   

### 06.非prop的attribute

