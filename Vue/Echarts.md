### Echarts实战

#### 快速上手

##### 通用配置

1. 标题title

2. tooltip

   > 提示框组件（鼠标滑过或者点击图表时会显示提示框）

   1. 触发类型：trigger
   2. 触发时机：trifferOn
   3. 格式化：formatter

3. toolBox

   > Echarts提供的工具栏（导出图片、数据视图、动态类型切换、数据区域缩放、数据重置）

   ```javascript
   toolbox: {
       feature: {
          saveAsImage: {}, //保存图片
                 dataView: {}, // 数据视图
                 restore: {}, // 数据重置
                 dataZoom: {}, // 数据缩放
                 magicType: {  // 动态图表类型切换
             type: ['bar', 'line']
                 }
             }
     }
   ```

4. 图例legend

   > legend：图例，用于筛选数据，需要和series配合使用

   ```javascript
   legend: {
   	data: ['语文', '数学']
   }
   ```

##### 折线图

1. 标记

   最大值、最小值、平均值、标注区间

2. 线条控制

   平滑控制`smooth`、风格 `lineStyle`

3. 填充风格

   areaStyle

4. 紧挨边缘

   boundaryGap

5. 缩放（脱离0值比例）

   scale

6. 堆叠图

   stack

##### 散点图

> 利用散点图可以帮助我们推断出变量之前的相关性

1. 常规散点图的实现

   ```javascript
   let option = {
    xAxis: {
     type: 'value',
     scale: true
    },
    yAxis: {
     type: 'value',
     scale: true
    },
    series: [
     {
      type: 'scatter', // 指明图表的类型为散点图
      data: axisData
     }
    ],
       tooltip: {
     trigger: 'item',
        formatter: (args) => {
      return '身高：' + args.data[0] + ', 体重: ' + args.data[1]
        }
    },
   }
   ```

2. 气泡效果

   1. 散点大小的控制

      symbolSize回调函数实现

   2. 散点颜色的控制

      itemStyle.color实现

3. 涟漪动画效果

   ```javascript
   type: 'effectScatter', // 指明图表为带涟漪动画的散点图
   showEffectOn: 'emphasis', // 出现涟漪动画的时机 render emphasis
   rippleEffect: {
     scale: 10 // 涟漪动画时, 散点的缩放比例
   }
   ```

##### 直角坐标系常用配置

1. dataZoom区域缩放器

   > dataZoom 用于区域缩放, 对数据范围过滤, x轴和y轴都可以拥有, dataZoom 是一个数组, 意味着 可以配置多个区域缩放器

   1. 区域缩放类型

      silder：外置滑块

      inside：内置（依靠鼠标滚轮或者双击缩放）

   2. 产生的作用轴

      xAxisIndex :设置缩放组件控制的是哪个 x 轴, 一般写0即可

      yAxisIndex :设置缩放组件控制的是哪个 y 轴, 一般写0即可

   3. 指明初始缩放状态

      start : 数据窗口范围的起始百分比

      end : 数据窗口范围的结束百分比

##### 饼图

1. 显示数值

2. 南丁格尔图

   `roseType: 'radius'`

3. 选中效果

   `selectedMode: 'single'`

4. 圆环

   `radius: ['50%', '75%']`

##### 地图

1. 缩放拖动：roam
2. 名称显示：label
3. 初始比例缩放：zoom
4. 地图中心点：center

##### 雷达图

> 雷达图可以用来分析多个维度的数据与标准数据对比情况

##### 仪表盘

##### 七种常见图表的使用场景

![image-20210311164251578](https://i.loli.net/2021/03/11/EtNd6QRWPiK1roM.png)

##### 主题

1. 内置主题
2. 自定义主题样式

##### 调色盘

1. 主题调色盘
2. 全局调色盘
3. 局部调色盘

##### 颜色渐变
·
1. 线性渐变
2. 径向渐变

##### 动画

1. 加载动画

   > Echarts内部已经实现了加载数据的动画，我们只需要在合适的时机显示或者隐藏即可

   1. 显示加载动画

      mCharts.showLoading()

   2. 隐藏加载动画

      mCharts.hideLoading()

2. 增量动画

##### Echarts对象

1. Echarts全局对象

   ```javascript
   init() // 初始化对象，获取Echarts实例对象或者指定主题
   registerTheme() //注册主题
   registerMap() // 注册地图
   connect() //实现多个图标相 互关联
   ```

2. Echarts实例对象

   1. setOption：设置配置项以及数据，多次调用setOption会合并新的配置和旧的配置

   2. resize：重新计算和配置图表，一般和window对象的resize事件结合使用

      ```javascript
      window.onresize = function(){
      	myChart.resize();
      }
      ```

   3. on/off绑定或者解绑事件处理函数

      1. 鼠标事件

         ```java
         // 常见事件: 'click'、'dblclick'、'mousedown'、'mousemove'、'mouseup' 等
         // 事件参数 arg: 和事件相关的数据信息
         mCharts.on('click', function (arg) {
          // console.log(arg)
          console.log('饼图被点击了')
         })
         // 解绑事件:
         mCharts.off('click')
         ```

      2. Echarts事件

         ```javascript
         // 常见事件:
         'legendselectchanged'、'datazoom'、'pieselectchanged'、'mapselectchanged'
         // 事件参数 arg: 和事件相关的数据信息
         mCharts.on('legendselectchanged', function (arg) {
         	console.log(arg)
         	console.log('图例选择发生了改变...')
         })
         ```

      3. dispatchAction：主动触发某些行为（通过代码的方式模拟用户的行为）

         ```javascript
         // 触发高亮的行为
         mCharts.dispatchAction({
             type: "highlight",
             seriesIndex: 0,
             dataIndex: 1
         })
         // 触发显示提示框的行为
         mCharts.dispatchAction({
             type: "showTip",
             seriesIndex: 0,
             dataIndex: 3
         })
         ```

      4. clear：清空当前实例，会移除实例中所有的组件和图表，清空之后可以再次setOption

      5. dispose：销毁实例，销毁之后实例无法再次被使用

   ### Koa2快速入手

   #### 快速开始

   ```javascript
   // 1.创建Koa对象
   const Koa = require('koa')
   const app = new Koa();
   // 2.编写响应函数(中间件)
   /**
    * context,web容器的上下文
    * next，下一个中间件
    */
   app.use((context, next) => {
   	console.log(context.request.url);
   	context.response.body = 'Hello Koa'
   })
   // 3.绑定端口号:3000
   app.listen(3000)
   ```

   #### 项目准备

   项目准备

   总耗时中间件

   响应头中间件

   业务逻辑中间件

   允许跨域

   



