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

### 07.子组件向父组件通信

#### 子传父应用场景

1. 子组件有一些事件触发时，例如点击了子组件，需要切换父组件内容
2. 子组件有数据需要传递给父组件

#### 子传父操作流程

1. 在子组件中定义要触发的事件

   ```javascript
   emits: ["add", 'sub', 'addN']
   ```

   ```html
   <button @click="increment">+1</button>
   <button @click="decrement">-1</button>
   ```

   ```javascript
   methods: {
     increment() {
   	console.log("+1");
   	// 触发自定义的事件
   	this.$emit("add")
     },
     decrement() {
   	console.log("-1");
   	this.$emit("sub")
     },
     incrementN() {
   	this.$emit("addN", this.number, "lucas", 24)
     }
   }
   ```

2. 在父组件中通过v-on的方式监听子组件触发的事件，并且绑定到自己的方法中

   ```html
   <counter-operation 
     @add="addOne" 
     @sub="subOne"
     @addN="addNum"/>
   ```

3. 在子组件中触发某个事件时，通过父组件监听的方式来响应对应的事件

   ```javascript
   methods: {
     addOne() {
   	this.counter ++
     },
     subOne() {
   	this.counter --
     },      
     addNum(number, name, age) {
   	this.counter += number
   	console.log(name, age);
     }
   }
   ```

## 14.跨组件通信和插槽的使用

非父子组件的通信方式

1. project/inject方式
2. mitt全局事件总线方式

### 1.Provide和Inject

Provide/Inject用于非父子组件之间传递共享数据，比如说有一些嵌套比较深的组件，子组件想要获取父组件的内容。在这种情况下如果我们还是通过props沿着组件链逐级向下传递则会非常麻烦。

对于这种情况我们可以使用Provide/Inject，无论组件之间嵌套有多深入，父组件都可以作为子组件、以及其所有的子孙组件的依赖提供者。其中父组件中有一个`provide选项`用来提供数据，子组件中有一个`Inject选项`用来使用数据

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.6h7nbm584sg0.png" alt="image" style="zoom:67%;" />

#### 使用案例

provide建议统一定义成函数，方便其内部this关键字的使用

provide 定义为一个函数，方便this关键字指向当前的vue实例
如果this关键字直接在对象里面定义，则this关键字指向的是当前对象

1. 父组件内部定义`provide函数`

   ```javascript
     export default {
       components: {
         Home
       },
       provide() {
         // provide 定义为一个函数，方便this关键字指向当前的vue实例
         // 如果this关键字直接在对象里面定义，则this关键字指向的是当前对象
         // 当前对象里面没有array这个数据，因此length会是undefined
         return {
           name: "lucas",
           age: 24,
           length: this.array.length
         }
       },
       data() {
         return {
           array: ["abc", "b", "c"]
         }
       }
     }
   ```

2. 子孙组件

   在子孙作用域通过inject注入的方式使用

   ```html
   <template>
     <div>
       <span>HomeContent: {{name}} - {{age}} - {{length}}</span>
     </div>
   </template>
   
   <script>
     export default {
       inject: ["name", "age", "length"]
     }
   </script>
   ```

#### Provide提供响应式数据

如果希望provide提供的数据是响应式的，则需要使用`computed函数`进行包裹

*注意：computed函数当前版本返回的是一个`ref对象`，如果需要拿到真实数据，需要通过`ref.value`方式才可以实现。不过在下一个vue版本中ref对象将会自动解包*

```html
<template>
  <div>
    <home></home>
    <button @click="pushArray">+name</button>
  </div>
</template>

<script>
  import Home from "./Home.vue"
  
  import { computed } from "vue"

  export default {
    components: {
      Home
    },
    provide() {
      // provide 定义为一个函数，方便this关键字指向当前的vue实例
      // 如果this关键字直接在对象里面定义，则this关键字指向的是当前对象
      return {
        length: computed(() => this.array.length)
      }
    },
    data() {
      return {
        array: ["abc", "b", "c"]
      }
    },
    methods: {
      pushArray() {
        console.log(this.array);
        this.array.push("lucas")
      }
    },
  }
</script>
```

### 2.全局事件总线mitt

#### 应用场景

非父子组件之间、兄弟组件之间的通信以及数据共享

**安装库mitt**

```shell
npm install mitt
```

#### 使用方式

1. eventbus.js

   ```javascript
   import mitt from "mitt";
   
   const emitter  = mitt();
   
   export default emitter
   ```

2. App.vue

   ```javascript
   <template>
     <div>
       <button @click="click">按钮</button>
     </div>
   </template>
   
   <script>
     import emitter from "./utils/eventbus"
   
     export default {
       methods: {
         click() {
           console.log("click");
           // 通过事件总线发射一个事件
           emitter.emit("clicekEvent", {
             eventName: "clicekEvent",
             username: "lucas"
           })
         }
       }
     }
   </script>
   ```

3. HomeContent.vue

   ```javascript
   <script>
     import emitter from "./utils/eventbus"
   
     export default {
       created() {
         emitter.on("clicekEvent", (info) => {
           console.log(info);
         })
       }
     }
   </script>
   ```

### 3.认识插槽的作用

#### 插槽的定义

```html
<template>
  <div class="nav-bar">
    <div class="left">
      <slot name="left"/>
    </div>
    <div class="center">
      <slot name="center"/>
    </div>
    <div class="right">
      <slot name="right"/>
    </div>
  </div>
</template>
```

#### 插槽的使用

```html
<template>
  <div>
    <nav-bar>
      <template v-slot:left>
        <button>左边按钮</button>
      </template>
      <template v-slot:center>
        <span>中间title</span>
      </template>      
      <template v-slot:right>
        <i>i标签</i>
      </template>      
    </nav-bar>
  </div>
</template>
```

#### 插槽语法糖

```html
<template>
  <div>
    <nav-bar>
      <template #left>
        <button>左边按钮</button>
      </template>
      <template #center>
        <span>中间title</span>
      </template>      
      <template #right>
        <i>i标签</i>
      </template>      
    </nav-bar>
  </div>
</template>
```

#### 作用域插槽

##### Vue中的渲染作用域

父级模板中的所有内容都是在父级作用域中编译的；子级模板中的所有内容都是在子级作用域中编译的。也就是**模板中的所有内容不可以跨作用域编译**

##### 作用域插槽使用场景

需求：父组件中的插槽可以访问到子组件中的内容

当一个组件被用来渲染数组元素时，我们通过插槽形式来实现，并且希望插槽中没有显示每项的内容。这个需求可以利用Vue提供的作用域插槽来实现

##### 代码实现

1. 父组件

   ```javascript
   <template>
     <div>
       <show-names :names="names">
         <template v-slot="slotProps">
           <span>{{slotProps.item}}- {{slotProps.index}}</span>
         </template>
       </show-names>
     </div>
   </template>
   ```

2. 子组件

   ```javascript
   <template>
     <div>
       <template v-for="(item, index) in names" :key="index">
         <!-- 自定义插槽的属性 -->
         <slot :item="item" :index="index"></slot>
       </template>
     </div>
   </template>
   ```

## 15.动态|异步|keepalived|生命周期

### 1.动态组件的基本使用

我们可以通过动态组件实现组件之间的切换，`component组件`是vue内置的动态组件，可以通过特殊`is属性`来实现

##### 代码实现

```javascript
<template>
  <div>
    <button v-for="item in tabs" 
            :key="item"
            @click="btnClick(item)"
            :class="{active: currentName === item, inactive: currentName !== item}">
      {{item}}
    </button>

    <!-- 动态组件实现切换组件案例 -->
    <component :is="currentName"></component>
  </div>
</template>

<script>
  import Home from "./pages/Home.vue"
  import About from "./pages/About.vue"
  import Category from "./pages/Category.vue"

  export default {
    components: {
      Home, About, Category
    },
    data () {
      return {
        tabs: ["Home", "About", "Category"],
        currentName: "Home"
      }
    },
    methods: {
      btnClick(item) {
        this.currentName = item
      }
    }
  }
</script>

<style scoped>
  .active {
    color: red
  }
  .inactive {
    color: grey;
    font-style: italic;
  }
</style>
```

### 2.动态组件的参数传递

父组件代码实现

```html
<!-- 动态组件实现切换组件案例 -->
<component 
  :is="currentName" 
  name="lucas"
  :age="18"
  @pageClick="pageClick"
/>
```

子组件代码实现

```javascript
props: {
  name: {
	type: String,
	default: ""
  },
  age: {
	type: Number,
	default: 18
  }
},
```

### 3.keepalive缓存组件

##### keepalive组件功能

使用keepalive组件内部包裹的组件有**缓存效果**

```html
<keep-alive>
  <component 
	:is="currentName" 
	name="lucas"
	:age="18"
	@pageClick="pageClick"
  />    
</keep-alive>
```

##### keepalive组件属性

<img src="https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.6d86osl4dg40.png" alt="image" style="zoom:67%;" />

1. include

   该属性支持String | RegExp | Array，只有名称匹配的组件才可以被缓存

2. exclude

   该属性支持String | RegExp | Array，任何名称被匹配的组件都不会被缓存

3. max

   该属性支持Number | String，最多可以缓存多少组件实例

### 4.webpack的import函数分包

通过import函数导入的模块，后续webpack会进行**分包打包**

```javascript
import ('./11.异步组件的使用/utils/math').then(({ sum }) => {
    console.log(sum(3, 4))
})
```

### 5.定义异步组件和代码分包

#### 异步组件使用场景

如果我们的项目很大， 对于**某些组件**，我们希望通过**异步的方式进行加载**（目的是可以对其进行分包处理），Vue3中提供了一个`defineAsyncComponent`函数可以实现该需求

defineAsyncComponent函数接收两种类型的参数

1. 工厂函数

   该工厂函数需要返回一个Promise对象

2. 对象

   接收一个对象类型，对异步函数进行配置

#### 异步组件实现

```javascript
<script>
    // 导入defineAsyncComponent函数
    import { defineAsyncComponent } from 'vue'
	// 同步导入
    import About from "./components/About"
	// 异步导入
    const AsyncCategory = defineAsyncComponent(() => import('./components/AsyncCategory.vue'))

    export default {
        components: {
            About, AsyncCategory
        }
    }
</script>
```

### 6.异步组件和suspense结合使用

suspense是Vue3的内置组件，可以简化使用defineAsyncComponent函数的配置，其提供了两个插槽

1. default

   如果default插槽的内容可以显示，那么直接显示

2. fallback

   如果default插槽的内容不可以显示，那么显示fallback插槽内容

```html
<suspense>
	<template #default>
		<async-category/>        
	</template>
	<template #fallback>
		<loading/>
	</template>                        
</suspense>
```

### 7.获取元素和组件refs

#### $refs使用场景

在某些场景下，我们想要直接获取元素对象或者组件示例，我们可以通过`ref属性`实现。在Vue中不推荐我们直接操作DOM，这个时候我们可以给元素或者组件绑定一个ref属性

#### $refs代码实现

```html
<template>
    <div>
        <!-- 绑定到元素上 -->
        <h2 ref="title">H2标题</h2>
        <!-- 绑定到组件上 -->
        <my-component ref="component"/>
        <button @click="btnClick">$refs</button>
    </div>
</template>

<script>
    import { defineAsyncComponent } from 'vue'
    const MyComponent = defineAsyncComponent(() => import('./MyComponent.vue'))

    export default {
        components: {
            MyComponent
        },
        methods: {
            btnClick() {
                // 获取元素内容
                console.log(this.$refs.title.innerText);
                // 获取组件定义的内部属性
                console.log(this.$refs.component.message);
                // 调用组件定义的方法
                this.$refs.component.sayHello();
            }
        }
    }
</script>
```

#### $parent $root

我们可以$parent和$root获取父组件和根组件

注意：在Vue3中$children已经

### 8.组件和组件实例的关系

### 9.理解什么是组件的生命周期

![生命周期函数](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/生命周期函数.5y8i5zizbyc0.png)

#### 什么是生命周期

每个组件都可能会经历从创建、挂载、更新、卸载等一系列过程，在这个过程中的某一个阶段，可根绝需求添加一些我们自己的逻辑（比如组件创建完成之后去请求后端接口）；所以Vue给我们提供了组件的声明周期函数

#### 生命周期函数

声明周期函数本质是一些钩子函数，会在某个时间段在Vue源码内部被调用；通过对声明周期函数的回调，我们可以知道目前组件正在经历什么样的阶段；

#### 生命周期函数应用场景

1. created

   请求后端接口可以在created钩子函数进行

2. beforeUpdate

   通过该生命周期函数可以获取数据更新之前的值

3. updated

   通过该生命周期函数可以获取数据更新之后的值

4. unmount

   取消事件的注册可以在unmount生命周期下进行

### 10.生命周期函数的演练

```javascript
    beforeCreate() {
      console.log("home beforeCreate");
    },
    created() {
      console.log("home created");
    },
    beforeMount() {
      console.log("home beforeMount");
    },
    mounted() {
      console.log("home mounted");
    },
    beforeUpdate() {
      console.log("改变之前的数据：", this.$refs.data.innerText);
      console.log("home beforeUpdate");
    },
    updated() {
      console.log("改变之后的数据：", this.$refs.data.innerText);
      console.log("home update");
    },
    beforeUnmount() {
      console.log("home beforeUnmount");
    },
    unmounted() {
      console.log("unmounted");
    },
```

### 11.缓存组件的生命周期

对于缓存组件来说，进入或者离开一个组件时，不会执行created/unmounted生命周期函数。但是有时候我们确实有希望监听到进入组件/离开组件的需求，这个时候可以使用actived/deactived这两个生命周期函数满足该需求

```javascript
    activated() {
      console.log("home activated")
    },
    deactivated() {
      console.log("home deactivated")
    }
```



### 12.组件的v-model的基本使用

### 13.组件的v-model的计算属性的实现

### 14.自定义组件的多个v-model

## 16.动态|异步|keepalived|生命周期 组件v-model

## 17.Vue3实现动画-animate-gsap

### 1.Vue3动画的基本使用

```vue
<template>
  <div>
    <button @click="animationShow" ref="buttonText">隐藏</button>
    <!-- vue内置组件 -->
    <transition name="animation">
      <h2 v-if="isShow" >Hello Animation</h2>
    </transition>
  </div>
</template>

<script>
  export default {
    name: "App",
    data() {
      return {
        isShow: true
      }
    },
    methods: {
      animationShow() {
        this.isShow = !this.isShow;
        if(this.isShow) {
          this.$refs.buttonText.innerHTML = "隐藏"
        }else {
          this.$refs.buttonText.innerHTML = "显示"
        }
      }
    }
  }
</script>

<style scoped>
  .animation-enter-from, .animation-leave-to {
    opacity: 0;
  }
  .animation-enter-to, .animation-leave-from {
    opacity: 1;
  }
  .animation-enter-active, .animation-leave-active {
    transition: opacity 1s ease;
  }
</style>
```

### 2.transition的执行原理

1. transition的执行原理

   当插入或者删除在transition组件中的元素时，Vue会做如下处理

   1. 自动嗅探目标元素是否应用了CSS过渡或者动画，如果有的话那么在恰当的实际添加/删除CSS类名
   2. 如果transition组件提供了钩子函数，那么这些钩子函数会在恰当的时机被调用
   3. 如果没有嗅探到CSS过渡动画，并且也没有找到JavaScript钩子函数。那么DOM的插入，删除操作会立即执行

2. 过渡动画常用类名

   ![image](https://cdn.jsdelivr.net/gh/Andre235/-community@master/src/image.433ibaexy3y0.png)

3. transition组件的name属性命令规则

   如果transition组件没有name属性时，那么所有的class会以 v- 作为默认前缀

   如果我们添加了name属性，例如<transition name="why"> ，那么所有的class会以 why- 作为前缀

### 3.animation帧动画实现

```vue
<template>
  <div style="width: 400px; margin: 0 auto">
    <div>
      <button @click="isShow = !isShow" ref="buttonText">隐藏</button>
    </div>
    <!-- vue内置组件 -->
    <transition name="animation">
      <h2 v-if="isShow" style="display: inline-block">Hello Animation</h2>
    </transition>
  </div>
</template>

<script>
  export default {
    name: "App",
    data() {
      return {
        isShow: true
      }
    },
  }
</script>

<style scoped>
  .animation-enter-active {
    /* 进入的时候执行bounce帧动画 */
    animation: bounce 1s ease;
  }

  .animation-leave-active {
    /* 离开的时候执行bounce的反转帧动画 */
    animation: bounce 1s ease reverse;
  }

  @keyframes bounce {
    /* 第一帧缩放0 倍 */
    0% {
      transform: scale(0);
    }

    /* 第二帧缩放1.2倍 */
    50% {
      transform: scale(1.2);
    }

    /* 第三帧缩放1倍 */
    100% {
      transform: scale(1);
    }
  }
</style>
```

### 4.过渡动画和帧动画的结合使用

Vue为了知道过渡的完成，内部是在监听transitionend和animationed事件，到底使用哪个取决于元素应用的CSS规则（如果我们只使用其中的一个，那么vue可以自动识别类型，并设置监听）

但是如果我们同时使用了过渡和动画呢，并且在这种情况下可能某一个动画执行结束时，另外的动画还没有结束。在这种情况下，我们可以设置transition组件的type属性为animation或者transition，来明确告知vue监听的类型

```html
<transition name="animation" type="transition">
    <h2 v-if="isShow" style="display: inline-block">Hello Animation</h2>
</transition>
```

### 5.过渡模式（mode属性）

mode属性一般应用在两个元素相互切换时使用

1. mode="in-out"

   先等元素进来之后，另外的元素再出来

2. mode="out-in"

   先等元素出去之后，另外的元素再进来

##### 元素切换使用示例

```vue
<transition name="animation" mode="out-in">
    <h2 v-if="isShow">Hello Animation</h2>
    <h2 v-else>你好啊 李银河</h2>
</transition>
```

##### 动态组件切换使用示例

```vue
<transition name="animation" mode="out-in">
    <component :is="isShow ? 'home' : 'page'"/>
</transition>

<script>
  import Home from "@/15_animation_01/pages/Home";
  import Page from "@/15_animation_01/pages/Page";

  export default {
    name: "App",
    components: {
      Home, Page
    },
    data() {
      return {
        isShow: true
      }
    },
  }
</script>
```

##### 初次渲染显示动画（appear属性）

```html
<transition name="animation" mode="out-in" appear>
    <component :is="isShow ? 'home' : 'page'"/>
</transition>
```

### 6.认识animate.css动画库

animate.css是一个已经实现好的，跨平台的动画库，通常应用在强调、滑动、注意力引导等场景

#### animate.css的使用

1. 安装animate.css动画库

   ```shell
   npm install animate.css
   ```

2. 在main.js文件中导入动画库

   ```javascript
   import 'animate.css'
   ```

3. 使用

   我们可以在transition组件指定不同的属性来自定义不同时期的class类名，其中常用的属性有

   1. enter-from-class
   2. enter-active-class
   3. enter-to-class
   4. leave-from-class
   5. leave-active-class
   6. leave-to-class

   代码实现

   ```vue
   <transition
     appear
     enter-active-class="animate__animated animate__fadeInDown"
     leave-active-class="animate__animated animate__fadeInUp">
     <h2 v-if="isShow">animation第三方库使用</h2>
   </transition>
   ```

### 7.gsap库的使用

gsap是the greensock animation platform的缩写，它是一个JavaScript动画库，可以通过它来实现给CSS、SVG、Canvas等设置动画，并且它是浏览器兼容的

#### 使用方式

1. 安装gsap库

   ```shell
   npm install gsap
   ```

2. 导入gsap库

   ```javascript
   // 在需要使用gsap的vue文件中直接引入即可
   import gsap from 'gsap'
   ```

3. transition组件默认生命周期钩子函数

   ```vue
   <template>
     <div style="width: 400px; margin: 0 auto">
       <div>
         <button @click="show" ref="buttonText">隐藏</button>
       </div>
   
       <transition
         @before-enter="beforeEnter"
         @enter="enter"
         @after-enter="afterEnter"
         @before-leave="beforeLeave"
         @leave="leave"
         @after-leave="afterLeave">
         <h2 v-if="isShow">gsap第三方库使用</h2>
       </transition>
     </div>
   </template>
   
   <script>
     export default {
       name: "App",
       data() {
         return {
           isShow: true
         }
       },
       methods: {
         show() {
           this.isShow = !this.isShow
           if(this.isShow) {
             this.$refs.buttonText.innerHTML = "隐藏"
           }else {
             this.$refs.buttonText.innerHTML = "展示"
           }
         },
         beforeEnter() {
           console.log("beforeEnter")
         },
         enter() {
           console.log("enter")
         },
         afterEnter() {
           console.log("afterEnter")
         },
         beforeLeave() {
           console.log("beforeLeave")
         },
         leave() {
           console.log("leave")
         },
         afterLeave() {
           console.log("afterLeave")
         }
       }
     }
   </script>
   
   <style scoped>
   
   </style>
   ```

4. 使用gsap库实现简单的动画效果

   当我们使用JavaScript来执行过渡动画时，需要进行done函数回调，否则他们将会同步执行，过渡动画将会立即完成

   添加`:css="false"`，会使vue跳过CSS检测，除了性能略高之外，还可以避免过渡过程中其他CSS规则对过渡动画的影响

   ```vue
   <template>
     <div style="width: 400px; margin: 0 auto">
       <div>
         <button @click="show" ref="buttonText">进入</button>
       </div>
   
       <transition @enter="enter" @leave="leave" :css="false">
         <h2 v-if="isShow">gsap第三方库使用</h2>
       </transition>
     </div>
   </template>
   
   <script>
     import gsap from 'gsap'
   
     export default {
       name: "App",
       data() {
         return {
           isShow: true
         }
       },
       methods: {
         show() {
           this.isShow = !this.isShow
           if(this.isShow) {
             this.$refs.buttonText.innerHTML = "离开"
           }else {
             this.$refs.buttonText.innerHTML = "进入"
           }
         },
         enter(el, done) {
           console.log("enter")
           gsap.from(el, {
             scale: 0,
             x: 200,
             onComplete: done
           })
         },
         leave(el, done) {
           console.log("leave")
           gsap.to(el, {
             scale: 0,
             x: 200,
             onComplete: done
           })
         },
       }
     }
   </script>
   
   <style scoped>
   
   </style>
   ```

   

## 18.Vue3实现动画-animate-gsap（2）

## 19.vue3的Mixin和CompositionAPI



