### Vue基础

#### Vue常用指令

##### v-once

该指令表示元素或者组件只渲染一次，不会随着组件的改变而改变

##### v-html

```html
<h3 v-html="url"></h3>
```

解析HTML标签的字符串，渲染成HTMl标签

##### v-pre

v-pre指令用于跳过这个标签元素和它的子元素的编译过程，显示最原始的Mustache语法

##### v-bind

作用：动态绑定属性，可以缩写为一个冒号：

```html
<a :href="url">百度一下</a>
<h3 :class="{fontColor: isColor, bgcColor: isBgc}">Hello Vue</h3>
<button @click="changeFontColor">改变字体颜色</button>
<button @click="changeBgcColor">改变背景颜色</button>
```

##### v-on

- 作用：绑定事件监听器（点击事件/鼠标悬停事件/拖拽事件/键入事件）

- 语法糖：@

- 修饰符

  - .stop：停止冒泡

    ```html
    <div @click="divClick">
        {{message}}
        <button @click.stop="btnClick">点击</button>
    </div>
    ```


  - .prevent：组织默认行为

  - .{keyCode | keyAlias}：从特定键触发时才触发回调

  - .native：监听组件跟元素的原生事件

  - .once 只触发一次

##### v-if

作用：表示一个元素是否能够被渲染，与之相结合使用的是 `v-else` 和 `v-else-if`

##### v-show

**区别：**v-if 和 v-show都可以表示一个元素能否被渲染，二者区别：v-if的条件表达式为false时，压根不会有对应的元素在DOM中；v-show的条件表达式为false时，仅仅是将元素的display属性设置为none

**如何选择：**

- 当需要频繁在显示于隐藏之间切换时，用 v-show指令
- 当只有一次在显示于隐藏之间切换时，用v-if指令

##### v-for

```html
<ul>
    <li v-for="(value, key, index) in object">
        {{index}}.{{key}}:{{value}}
    </li>
</ul>
```

##### v-model

#### Vue基础知识

##### 计算属性

某些情况下，我们需要对数据操作再展示，这个时候可以考虑用计算属性来实现。计算属性表面上看起来是一个方法，但是本质上是一个属性（是get方法的语法糖实现）

```javascript
//计算属性，看起来是一个方法，本质上是一个属性(get方法的语法糖的实现)
computed: {
    fullName() {
        return this.firstName + ' ' + this.lastName
    }
}
```

计算属性和Method对比：计算属性底层对方法进行了一层缓存封装，多次调用相同的数据的方法最终只会调用一次，相对来说相比于Method性能会更高一些

##### 过滤器

1. 过滤器概念

   Vue对数据提供一个过滤器，用以在不改变原数据的情况下输出前端需要的数据格式

2. 使用方式

   ```html
   <!--普通方法实现格式化小数-->
   <td>{{formatPrice(book.price)}}</td>
   <!--过滤器实现格式化小数(注意传递参数问题)-->
   <td>{{book.price | formatPriceFilter}}</td>
   ```

3. 注意事项

   在一个{{message}}模板中，可以定义多个filter函数对message处理

   可以给filter过滤器函数传入多个参数，但是过滤器默认的第一个参数是是模板中message的值，所以如果要传入自定义参数的话，得从参数的第二个位置往后传

#### ES6 

ES5之前没有作用域的概念，ES6以后才添加了作用域的概念，有了let关键字

##### let/var

- ES5中的var没有块级作用域
- ES6中的let有块级作用域



