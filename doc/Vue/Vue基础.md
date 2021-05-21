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

**区别：**v-if 和 v-show都可以表示一个元素能否被渲染，二者区别：v-if的条件表达式为false时，压根不会有对应的元素在DOM中；v-show的条件表达式为false时，仅仅是将元素的display属性设置为Dnone

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

- 作用

  Vue中使用v-model来实现表单与data数据的双向绑定

- 原理

  v-model本质上是一个语法糖，其背后是基于两个操作实现的

  1. v-bind绑定表单的value属性
  2. v-on给表单绑定input事件

- 代码示例

  ```html
  <!--<input type="text" v-model="message">-->
  <input type="text" :value="message" @input="message = $event.target.value">
  ```

- 修饰符

  1. lazy修饰符

     v-model指令默认情况下是实时绑定输入框中的数据的，而lazy修饰符可以让input输入框失去焦点或者键入回车之后再绑定data数据

  2. number修饰符

     默认情况下无论我们在input输入框中输入数字还是字母都会被当做字符串来处理，number修饰符可以让我们在输入数字的时候自动解析成数字来处理

  3. trim修饰符

     如果input输入库中的字符串的前后两端有很多空格，trim修饰符可以自动帮我们去掉空格

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

##### JS高阶函数的使用

```javascript
const array = [1, 2, 3, 4, 5, 6];
let total = array.filter(n => n % 2 ===0).map(n => n * 2).reduce((preValue, n) => preValue + n) ;
console.log(total);
```

#### ES6 

ES5之前没有作用域的概念，ES6以后才添加了作用域的概念，有了let关键字

##### let/var

- ES5中的var没有块级作用域
- ES6中的let有块级作用域

### Vue进阶

##### 全局组件和局部组件

- 全局组件

  注册的全局组件在别的Vue实例中也可以使用

  ```javascript
  //2.注册全局组件
  Vue.component("my-com",component1);
  ```

- 局部组件

  注册的布局组件只能在当前Vue实例中使用

  ```javascript
  //2.注册局部组件
  let app = new Vue({
      el: '#app',
      components: {
          com1: component1
      }
  })
  ```

##### 组件内部数据的存放

- 注意事项

  Vue要求组件内部的数据必须存放在data函数中。为什么data必须是一个函数？因为组件在复用的过程中如果data是一个函数，则每次调用data时会返回一个新的数据，在改变该组件的data时不会影响到其他组件的data

- 示例代码

  ```html
  <template id="cpn">
      <div>
          <p>组件内部的数据:{{context}}</p>
      </div>
  </template>
  
  Vue.component('myCom', {
      template: '#cpn',
      data() {
          return {
          	context: '我是组件内部的数据'
          }
      }
  })
  ```

##### 组件通信（父传子）

- 实现方案

  通过props属性向子组件传递数据

##### 组件通信（子传父）

- 实现方案

  通过自定义事件向父组件发送消息

##### 组件访问（父访问子）

- 需求

  父组件想要获取子组件的data

- 实现方式

  - $children

    父组件直接获取其所有的子组件

    ```javascript
    console.log(this.$children);
    ```

  - $refs

    通过ref属性手动指定需要访问的组件

    ```javascript
    console.log(this.$refs.third.context);
    ```

- 示例代码

##### 组件访问（子访问父）

- 实现方式

  $parent

##### slot插槽

- 应用场景

  - 组件插槽的作用是为了让我们封装的组件具有更加灵活的扩展性
  - 让使用者可以决定组件内部到底使用什么标签

- 分类

  - 具名插槽

    在创建插槽的时候通过name属性给定具体的名字，在使用插槽的时候通过slot属性来指定要替换的插槽

  - 作用域插槽

    > 父组件模板的所有东西都只会在父级作用域内编译，并不会读取到子集作用域的属性；子组件模板的所有东西都只会在子级作用域内编译

    **作用域插槽的本质需求：**是在父组件中拿到子组件中定义的数据

    ```html
    <div id="app">
        <h3>{{message}}</h3>
        <cpn></cpn>
        <cpn>
            <template slot-scope="slot">
                <span>{{slot.data.join(' - ')}}</span>
            </template>
        </cpn>
    </div>
    
    <template id="cpn">
        <div>
            <!--作用域插槽：对外提供一个名字叫list的插槽作用域-->
            <slot :data="languages">
                <ul>
                    <li v-for="item in languages">{{item}}</li>
                </ul>
            </slot>
        </div>
    </template>
    ```

- 示例代码

  ```html
  <div id="app">
      <h3>{{message}}</h3>
      <cpn>
          <button slot="center">中间按钮</button>
      </cpn>
      <cpn>
          <button slot="left">返回</button>
      </cpn>
  </div>
  
  <template id="cpn">
      <div>
          <span>组件内容</span>
          <slot name="left"><span>左侧插槽</span></slot>
          <slot name="center"><span>中间插槽</span></slot>	
          <slot name="right"><span>右侧插槽</span></slot>
      </div>
  </template>
  ```

##### 箭头函数中的this

箭头函数中的this是如何查找的？向外层作用域一层一层查找this，直到有this的定义为止

##### 前端渲染&后端渲染

1. 前端渲染

   前后端分离开发

2. 后端渲染

   通过JSP或者PHP技术，当用户请求服务器的时候，服务器就返回一个已经渲染好的HMTL/CSS网页，这种就是服务端渲染/后端渲染

##### 前端路由&后端路由

1. 前端路由

2. 后端路由

   由后端处理URL和页面之间的映射关系

### Webpack

##### 概念

webpack是现代JavaScript应用的**模块化打包**工具。在项目开发中，我们通常使用webpack来处理js代码，并且webpack会自动帮我们管理js代码之间的依赖关系。

例如：将ES6规范的代码转换成ES5规范的代码；将TS代码转换成js代码；将scss代码、less代码转换成css代码；将.vue文件转换成.js文件等等

![image-20210302140026481](https://i.loli.net/2021/03/02/cMmPTvqrdRf7h6z.png)

##### 使用

```shell
# webpack打包命令
webpack ./src/main.js ./dist/bundle.js
```

```javascript
// 使用CommonJS模块化规范导入
const {add, multi} = require('./mathUtils')

console.log(add(20, 30));
console.log(multi(20, 30));

// 使用ES6模块化规范导出
import {name, age} from './info'

console.log(name);
console.log(age);
```

##### webpack配置

1. package.json文件 （执行 npm init命令会生成该文件）

   如果一个项目依赖于node环境，则会有package.json文件进行管理配置相关的包

   ```json
   "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1",
       // 执行脚本里面的命令会优先执行本地的webpack命令
       "build": "webpack" 
     },
   ```

2. 本地安装webpack开发时依赖

   ```shell
   npm install webpack@3.6.0 --save-dev
   ```

   此时`package.json`文件就会出现如下配置项

   ```json
   "devDependencies": {
       "webpack": "^3.6.0"
     }
   ```

##### loader处理css文件 

1. loader使用场景

   将scss代码、less代码转换成css代码

   打包css文件至指定的js文件

2. [安装loader](https://www.webpackjs.com/loaders/css-loader/)

   ```shell
   npm install --save-dev css-loader
   ```

##### ES6转ES5

引入babel依赖

### Vue-CLI

##### CLI2

1. 安装

   ```shell
   # 在安装CLI3的基础上执行如下指令
   npm install @vue/cli-init -g
   ```

2. 创建项目

   ```shell
   vue init webpack project-name
   ```

3. 创建项目过程详解

   ![image-20210302193834196](https://i.loli.net/2021/03/02/GF9lf1v43EnLdYb.png)

   

##### CLI3

1. 安装

   ```shell
   npm install -g @vue/cli
   ```

2. 创建项目

   ```shell
   vue create project-name
   ```

##### runtime-complier和runtime-only区别

1. runtime-comliler

   使用runtime-comliler编译项目的过程如下：template（组件） -> ast（抽象语法树） -> render（render函数）-> vdom（虚拟dom）-> UI（前端UI）

   <img src="https://i.loli.net/2021/03/02/u1lqJsNybIh49xM.png" alt="image-20210302222039225" style="zoom: 80%;" />

2. runtime-only

   使用runtime-only编译项目的过程如下：render（render函数）-> vdom（虚拟dom）-> UI（前端UI）

   如上图所示runtime-only的编译过程少了前两个步骤，所以相对来说**性能更高**，**代码量更少**

### Vue-Router

#### 认识路由

##### 概念

路由（routing）就是通过互联网把数据从源地址发送到目的地址的过程

##### 路由机制

1. 路由

   路由决定了数据包从源地址发送到目的地址的路径

2. 转发

   转发就是将数据从源地址选择合适的路径发送到目的地址

#### 基本使用

##### 使用步骤

1. 创建vue组件

2. 配置vue组件和路径映射关系

   ```javascript
   export default new Router({
     // 通过history方式修改URL
     mode: 'history',
     routes: [
       {
         // 配置首页重定向路径
         path: '',
         redirect: '/home'
       },
       {
         path: '/home',
         component: Home
       },
       {
         path: '/about',
         component: About
       }
     ]
   })
   ```

3. 使用路由 `router-link` 和 `router-view`

**改变URL但是不让页面重新刷新**

1. 改变URL的hash值

   ```java
   location.hash = 'aaa'
   ```

2. 修改history对象的URL属性

   ```
   history.pushState({}, '', 'home')
   ```

##### router-link常用属性

1. tag属性

   可以指定router-link渲染成什么组件

2.  replace属性

   使用replace属性不会留下history记录，所以不能使用后退键返回到上一个页面

3. active-class属性

   当router-link匹配路由成功时，会自动给当前元素设置一个名字是router-link-active的class，所以通过设置active-class属性可以修改默认的class名称

##### 通过代码实现路由跳转

```javascript
methods: {
    homeClick() {
      this.$router.push('/home')
    },
    aboutClick() {
      this.$router.push('/about')
    }
}
```

##### 动态路由的使用

##### $router 和  $route

1. $router

   $router是vue-router插件创建的router对象

2. $route

   $route是当前处于活跃状态的路由

##### 路由懒加载

1. 解释

   用到时才会加载某个路由对应的组件（性能相对于直接全部加载会更高）

2. 代码示例

#### 嵌套路由

##### 代码示例

```javascript
{
  path: '/home',
  component: () => import('../components/Home'),
  children: [
    {
      path: '',
      redirect: 'news'
    },
    {
      path: 'news',
      component: () => import('../components/HomeNews')
    },
    {
      path: 'message',
      component: () => import('../components/HomeMessage')
    }
  ]
}
```

#### 参数传递

##### 通过params方式传递参数

传递的是一个字符串，该参数会以URI的方式拼接在URL中

```html
<router-link :to="'/user/' + userId">用户</router-link>
```

##### 通过query方式传递参数

传递的是一个对象，该参数会以&符号拼接在URL中

```html
<router-link :to="{path: '/profile', query: {name: '赵静超', age: 24}}">档案</router-link>
```

#### 全局导航守卫（拦截器）

```javascript
// 调用router对象的beforeEach方法实现全局导航守卫
router.beforeEach( ((to, from, next) => {
  document.title = to.matched[0].meta.title;
  next();
}))
```

#### keep-alive

1. keep-alive是Vue内置的一个组件，可以是被包含的标签处于活跃状态（可以避免被重新渲染）

   ```javascript
   export default {
     name: "Home",
     data() {
       return{
         path: '/home'
       }
     },
     created () {
       console.log('Home组件被创建')
     },
     // 这两个函数只有在当前组件被保持活跃状态，使用了keep-alive标签时，才有效
     destroyed () {
       console.log('Home组件被销毁')
     },
     activated () {
       this.$router.push(this.path)
       console.log('Home组件的路由处于活跃状态')
     },
     deactivated () {
       console.log('Home组件的路由处于不活跃状态！')
     },
     beforeRouteLeave(to, from, next){
       console.log(this.$route.path);
       this.path = this.$route.path;
       next();
     }
   }
   ```

2. router-view也是一个组件，如果该组件被keep-alive标签包起来，那么路径匹配到的视图的数据会被缓存进内存中

   ```html
   <keep-alive><router-view/></keep-alive>
   ```

### Promise

##### 应用场景

对异步网络请求的封装

```javascript
new Promise(((resolve, reject) => {
    setTimeout(() => {
        // 请求成功的时候调用resolve方法
        // resolve('Hello Vue.js')

        // 请求失败的时候调用reject方法
        reject('error message')
    }, 2000)
})).then(data => {
    console.log(data)
}).catch(error => {
    console.log(error)
})
```

##### Promise的三种状态

1. pending等待状态

   正在进行网络请求的时候就处于等待状态

2. fullfill满足状态

   当我们主动调用resolve方法就处于满足状态，并且会回调then方法

3. rejetc拒绝状态

   当我们主动调用reject方法时就处于拒绝状态，并且会回调catch方法 

##### Promise链式调用

```javascript
new Promise(((resolve, reject) => {
    setTimeout(() => {
        console.log('开始进行第1次网络请求，消耗两秒...')
        resolve('第1次网络请求成功，调用请求成功后的回调函数...')
    }, 2000)
})).then(data => {
    console.log(data)

    return new Promise(resolve => {
        setTimeout(() => {
            resolve('开始进行第二次网络请求，耗时2秒...')
        }, 2000)
    })
}).then(data1 => {
    console.log(data1 + '第二次网络请求成功')

    return new Promise(((resolve, reject) => {
        setTimeout(() => {
            reject('开始进行第三次网络请求，耗时2秒...')
        }, 2000)
    }))
}).catch(data2 => {
    console.log(data2 + '第三次网络请求失败')
})
```

##### Promise.all方法

```javascript
// 需求场景：这里同时有两个网络请求，当两个网络请求都成功时，再处理某一个业务
// 提示：并不知道哪个网络请求先完成

Promise.all([
    new Promise(((resolve, reject) => {
        setTimeout(() => {
            console.log('result1结束')
            resolve('result1')
        }, 2000)
    })),
    new Promise(((resolve, reject) => {
        setTimeout(() => {
            console.log('result2结束')
            resolve('result2')
        }, 1000)
    }))
]).then(results => {
    console.log(results)
})
```

### VueX

##### 概念

Vuex是一个专门为vue.js应用程序开发的**状态管理工具**

##### 本质

Vuex采用的是**集中式、响应式 存储管理**所有组件的状态（变量），并且以相应的规则以一种可预测的方式发生变化

##### 应用场景

1. 用户的登陆状态（token）、userId、用户位置信息
2. 购物车中的商品信息
3. **Vuex一般存放需要在多个界面共享的数据**

##### Vuex管理共享状态流程

<img src="https://i.loli.net/2021/03/04/ECgnbd4eVNIrcvJ.png" alt="image-20210304201723647" style="zoom:80%;" />

##### Vuex核心概念

1. State（存放状态）

   单一状态树

2. Getters（类似计算属性）

3. Mutations（处理同步操作）

   1. 组成部分

      1. 事件类型
      2. 回调函数

   2. 实现方式

      传入事件类型，然后调用回调函数

      ```javascript
      // 定义事件类型
      mutations: {
          increment(state) {
              state.counter ++;
          }
      }
      
      methods: {
          addition(){
              // 传入事件类型，调用回调函数
              this.$store.commit('increment')
          }
      }
      ```

   3. 携带参数提交事件

      ```javascript
      mutations: {
        incrementCount(state, count){
          state.counter += count;
        }
      }
      ```

      ```javascript
      methods: {
        addCount(count){
          this.$store.commit('incrementCount', count)
        }
      }
      ```

   4. 提交风格

4. Action（处理异步操作）

   ```javascript
   // 提交同步方法
   mutations: {
     //mutations对象里面只能实现同步方法
     syncUpdateInfo(){
       this.state.info.name = 'lucas';
     }
   },
   ```

   ```javascript
   // 分发同步方法
   actions: {
     /**
      * 异步修改store中的信息
      * @param context 上下文
      */
     asyncUpdateInfo1(context){
       setTimeout(() => {
         context.commit('syncUpdateInfo');
       }, 2000)
     }
   }
   ```

   ```javascript
   methods: {
     updateInfoFun1(){
       this.$store.dispatch('asyncUpdateInfo1')
     }
   }
   ```

5. Mudule（分模块保存数据）

### Axios  

##### 基本GET请求

```javascript
axios({
  url: 'http://123.207.32.32:8000/home/multidata'
}).then(data => {
  console.log(data)
})
```

##### 带有参数的GET请求

```javascript
axios({
  url: 'http://152.136.185.210:8000/api/w6/home/data',
  params: {
    type: 'pop',
    page: 1
  }
}).then(data => {
  console.log(data)
})
```

##### 并发请求

```javascript
axios.all([
  axios({
    url: 'http://123.207.32.32:8000/home/multidata'
  }),
  axios({
    url: 'http://152.136.185.210:8000/api/w6/home/data',
    params: {
      type: 'pop',
      page: 1
    }
  })
]).then(data => {
  console.log(data)
})
```

##### 全局配置

![image-20210305173655512](https://i.loli.net/2021/03/05/iKztlwXPyO3CWn5.png)

##### Axios的封装

```javascript
import axios from 'axios'

export function request(config){
  //1.创建axios实例
  let instance = axios.create({
    baseURL: 'http://123.207.32.32:8000',
    timeout: 5000
  })
  //2.发送真正的网络请求
  return instance(config);
}
```

```javascript
import {request} from './network/request'

request({
  url: '/home/multidata'
}).then(data => {
  console.log(data)
}).catch(err => {
  console.log(err)
})
```

##### Axios拦截器的使用

1. 请求拦截器

   1. 请求成功拦截器使用场景
      1. 拦截掉一些不符合服务器的请求
      2. 每次发送网络请求的时候，在界面上展示loading图标
      3. 某些请求（登陆请求）必须在headers中携带Token，如果token失效或者没有携带Token则将页面重定向到登陆 页面
   2.  请求失败拦截器

2. 响应拦截器

   1. 响应成功拦截器
   2. 响应失败拦截器

3. 代码示例

   ```javascript
   export function request(config){
     //1.创建axios实例
     let instance = axios.create({
       baseURL: 'http://123.207.32.32:8000',
       timeout: 5000
     })
   
     //请求拦截器
     instance.interceptors.request.use(config => {
       console.log(config);
       return config;
     }, error => {
       console.log("请求失败：" + error)
     })
   
     //响应拦截器
     instance.interceptors.response.use(result => {
       return result.data;
     }, error => {
       console.log('响应失败' + error)
     })
     //2.发送真正的网络请求
     return instance(config);
   }
   ```



