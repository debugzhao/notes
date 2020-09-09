#### 计算属性

对于任何复杂的逻辑，建议使用计算属性实现。

```javascript
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>

var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
```

计算属性VS方法

定义一个方法也可以实现上面的结果

```javascript
<p>Reversed message: "{{ reversedMessage() }}"</p>

// 在组件中
methods: {
  reversedMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

**区别：** **计算属性是基于它们的响应式依赖进行缓存的**。只在相关响应式依赖发生改变时它们才会重新求值。这就意味着只要 `message` 还没有发生改变，多次访问 `reversedMessage` 计算属性会立即返回之前的计算结果，而不必再次执行函数

#### v-bind

**绑定：**既可以绑定元素的内容（用插值表达式实现），也可以绑定元素的属性（用v-bind实现）

##### 动态绑定style

实现方式

1. 对象语法

   ```html
   <div id="app">
       <!--使用v-bind动态绑定style，通过对象语法形式实现-->
       <h2 :style= "{fontSize: finalFonts,backgroundColor: finalBackgroundColor}">{{message}}</h2>
       <!--定义一个方法，返回style对象-->
       <h2 :style= "getStyles()">{{message}}</h2>
   </div>
   ```

2. 数组语法

   ```html
   <div id="app">
       <!--使用v-bind动态绑定style，通过数组语法形式实现-->
       <h2 :style= "[baseStyle,baseStyle1]">{{message}}</h2>
   </div>
   ```

#### v-model

##### 1.修饰符

1. lazy修饰符

   默认情况下v-model是在input事件触发后实时进行数据的同步，lazy修饰符是在表单输入回车/或者失去焦点的时候才会进行数据同步

   ```html
   <select name="fruits" v-model.lazy="fruits" multiple>
   ```

2. number修饰符

   当在进行表单输入时，默认会以字符串形式进行传值，输入数字时也会转换成字符串。number修饰符可以将输入框中的数字内容转换成数字类型

   ```html
   <select name="fruits" v-model.number="fruits" multiple>
   ```

3. trim修饰符

   会削减调表单首尾的空格

   ```html
   <select name="fruits" v-model.trim="fruits" multiple>
   ```

#### 组件化开发

##### 1.创建组件过程

1. 创建组件构造器 Vue.extend()

   ```javascript
   //1.创建组件构造器
   let component1 = Vue.extend({
      template: `
           <div>
               <h2>二级标题</h2>
               <h4>四级标题</h4>
               <p>正文内容</p>
           </div>
      `
   });
   ```

2. 注册组件（全局组件、局部组件）

   ```javascript
   //注册全局组件
   Vue.component("my-com",component1);
   
   //注册局部组件
   let app = new Vue({      
       el: '#app',
       data: {
           message: "hello vue"
       },
       components: {
           com1: component1
       }
   })
   ```

3. 组件的使用

##### 2.父传子

父组件向子组件传值，在子组件中使用props属性接收，同时在自定义的组件标签中用v-bind标签绑定即可

组件中定义props属性时，最好不要用驼峰方式命名，因为在使用组件时 v-bind不支持解析驼峰命名的变量（可以解析 c-info这样的变量命名方式）

**注意事项：**

1. 避免直接改变props属性里面的值，取而代替的是用data或者计算属性进行实现，因为每当父组件重新渲染时，该值都会被重新覆盖

##### 3.子传父

> 子组件向父组件传值场景：子组件里面触发事件，通知到父组件并且父组件予以相应

##### 4.父子组件之间进行互相访问

1. 父组件访问子组件

   - this.$children

   - this.$refs

     ```html
     <div id="app">
         <cpn></cpn>
         <cpn></cpn>
         <cpn ref="ref1"></cpn> <!--设置组件的引用属性id为ref1-->
         <button @click="btnClick">按钮</button>
     </div>
     
     btnClick(){
     	console.log(this.$refs.ref1); //取子组件时 取引用属性为ref1的子组件
     }
     ```

2. 子组件访问父组件

   - this.$parent（访问父组件）
   - this.$root（访问根组件）

#### slot插槽

> 插槽可以让我们封装的组件更加具有扩展性，可以让使用者决定展示组件内部的哪些内容

##### 1.基本使用方式

```html
<div id="app">
    <cpn><h1>扩展标题</h1></cpn>
    <cpn><button>扩展按钮</button></cpn>
    <cpn></cpn>
</div>

<template id="cpn">
    <div>
        <h3>组件标题</h3>
        <slot><button>默认插槽标题</button></slot>
    </div>
</template>
```

*如果有多个元素需要扩展，则同时放入插槽位置即可*

##### 2.具名插槽的使用

```html
<div id="app">
    <cpn><button slot="center">按钮</button></cpn>
</div>

<template id="cpn">
    <div>
        <slot name="left"><span>左边</span></slot>
        <slot name="center"><span>中间</span></slot>
        <slot name="right"><span>右边</span></slot>
    </div>
</template>
```

##### 3.编译作用域

父组件模板的所有东西会在父级作用域内编译，子组件模板的所有东西都会在子级作用域内编译

##### 4.作用域插槽

父组件替换插槽的标签，但是内容是由子组件提供的