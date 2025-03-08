## 邂逅TypeScript

### TypeScript编译环境

TypeScript最终会编译成JavaScript运行，所有我们需要对应的编译环境

```shell
# 安装编译环境
npm install typescript -g

# 查看版本
tsc --version
```

### TypeScript运行环境

1. 通过webpack，配置本地的TypeScript编译环境和开启一个本地服务，可以直接运行在浏览器上

   ```shell
   # 初始化该工程，生成package.json文件
   npm init
   
   # 安装webpack 和webpack cli
   npm install webpack webpack-cli -D
   
   # 安装ts文件加载器
   npm install ts-loader typescript -D
   ```

   

2. 通过ts-node库，为TypeScript的运行提供执行环境

   ```shell
   # 安装ts-node
   npm install ts-node -g
   
   # 安装ts-node需要依赖 tslib 和 @types/node 两个包
   npm install tslib @types/node -g
   
   # 通过ts-node命令直接运行ts文件
   ts-node math.ts
   
   # 搭建webpack本地服务
   npm install webpack-dev-server -D
   ```

### webpack搭建TypeScript运行环境

### TypeScript变量声明

1. ts中变量的声明

   ```typescript
   let name: string = "lucas"
   var age: number = 192
   const height: number = 100
   ```

2. string和String区别

   ```typescript
   // string: TypeScript中的字符串类型
   // String: JavaScript中的字符串包装类类型
   const height: number = 19
   ```

3. 类型推导

   ```typescript
   // 变量的类型推导
   let sex = "male"
   sex = 111 // 这行代码会报错
   ```

## TypeScript数据类型和类型操作

