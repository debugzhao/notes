1. 引入lodash实现对象的深拷贝
2. 引入vue-quill-editor实现富文本编辑器

#### 项目优化策略

1. 生成打包报告

2. 移除整个项目中的console方法

   ```
   npm install --dev babel-plugin-transform-remove-console
   ```

   ```javascript
   module.exports = {
     "presets": [
       "@vue/cli-plugin-babel/preset"
     ],
     "plugins": [
       [
         "component",
         {
           "libraryName": "element-ui",
           "styleLibraryName": "theme-chalk"
         }
       ],
       'transform-remove-console'
     ]
   }
   ```
   
3. 通过external节点加载外部CDN

   默认情况下，通过import语法导入的第三方依赖包，都会直接打包合并到一个文件中，就会导致打包成功后单文件体积过大的问题。

   问了解决该问题，可以通过webpack的external节点来配置并加载外部的CDN资源，凡是生命在external节点中的第三方依赖都不会被打包（**如果需要用到这些依赖，则回去windows全局对象节点中寻找该依赖**）

4. 不同环境指定不同的打包入口

   1. 配置不同的打包入口

      开发模式下的打包入口文件为src/main-dev.js发布模式下的打包入口文件为src/main-prod.js

   2. 通过自定义chainWebpack自定义打包入口

      1. `vue.config.js`文件总引入如下配置

         ```javascript
         config.set('externals', {
             vue: 'Vue',
             'vue-router': 'VueRouter',
             axios: 'axios',
             lodash: '_',
             echarts: 'echarts',
             nprogress: 'NProgress',
             'vue-quill-editor': 'VueQuillEditor'
         })
         ```

      2. `index.html`文件中引入如下配置

         ```html
               <!-- nprogress 的样式表文件 -->
               <link rel="stylesheet" href="https://cdn.staticfile.org/nprogress/0.2.0/nprogress.min.css" />
               <!-- element-ui 的样式表文件 -->
               <link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
         
               <!--js 文件 -->
               <script src="https://cdn.staticfile.org/vue/2.5.22/vue.min.js"></script>
               <script src="https://cdn.staticfile.org/vue-router/3.0.1/vue-router.min.js"></script>
               <script src="https://cdn.staticfile.org/axios/0.18.0/axios.min.js"></script>
               <script src="https://cdn.staticfile.org/lodash.js/4.17.11/lodash.min.js"></script>
               <script src="https://cdn.staticfile.org/echarts/4.1.0/echarts.min.js"></script>
               <script src="https://cdn.staticfile.org/nprogress/0.2.0/nprogress.min.js"></script>
         ```

         

5. element ui按需加载

6. 路由懒加载

7. 首页内容定制

   ```javascript
   // 添加一个isProd配置项
   config.plugin('html').tap(args => {
       args[0].isProd = true
       return args
   })
   ```

   ```html
   <title><%= htmlWebpackPlugin.options.isProd ? '' : 'dev - ' %>电商后台管理系统</title>
   
   <% if(htmlWebpackPlugin.options.isProd){ %>
       <!--外部CDN资源-->
   <% } %>
   ```

   

   

8. 全局引入nprogress进度条

   ```
   npm install --save nprogress
   ```

   

