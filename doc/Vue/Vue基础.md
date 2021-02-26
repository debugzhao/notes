### Vue基础

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







##### v-on

##### v-if

##### v-show

##### v-for

##### v-model





