#### @Bean

```java
@Bean(name = "BeanName")  //在spring容器中注册person对象
public Person person(){
	return new Person("natasha",21);
}
```

#### @ComponentScan

```java
/**
 * value:要扫描哪些包
 * excludeFilters:要排除包中的哪些类
 */
@Configuration //该配置类相当于配置文件
@ComponentScan(value = "com.hacker",excludeFilters =
    @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class, Service.class})) //包扫描
public class MyConfig {
}
```

#### @Scope

```java
/**
  * value:
  *      prototype 多例模式 但是在多实例模式中，什么时候调用该对象，什么时候创建该对象
  *      singleton 单例模式，默认在单例模式中，随着IOC容器启动，单例对象也会被创建并注入到容器中
 */
@Scope(value = "prototype")
@Bean(name = "BeanName")  //在spring容器中注册person对象
public Person person(){
    System.out.println("创建Person对象...");
    return new Person("natasha",21);
}
```

#### @Lazy

```java
/**
* @Lazy 注解为懒加载注解，只针对单例模式
* 开启懒加载注解后在IOC容器启动完成后不立即创建Bean对象，而是什么时候用，什么时候创建对象
*/
@Lazy
@Bean(name = "person")  //在spring容器中注册person对象
public Person person01(){
    System.out.println("Person对象创建完成...");
    return new Person("natasha",21);
}
```

#### @Conditional

```java
@Conditional({WindowsConditional.class})
@Bean(name = "bill")  //在spring容器中注册person对象
public Person person03(){
    System.out.println("bill对象创建完成...");
    return new Person("bill",62);
}
```

##### 自定义Condition条件

```java
public class LinuxConditional implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        //获取环境配置信息
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        return property.contains("Linux");
    }
}
```

#### @Import

```java
//直接import要导入的组件
@Import({Red.class,Green.class})
```

#### Bean的生命周期

1. bean创建
2. bean初始化（可以自定义初始化策略）
3. bean销毁（可以自定义销毁策略）

##### @PostConstruct

```java
@Component
public class Dog {
    public Dog(){
        System.out.println("Dog对象初始化方法...");
    }
    //对象创建并初始化完成后调用
    @PostConstruct
    public void init(){
        System.out.println("dog对象创建并初始化完成后调用....");
    }
    //对象销毁之前调用
    @PreDestroy
    public void destroy(){
        System.out.println("dog对象销毁之前调用...");
    }
}
```

#### @Value

直接给属性赋值

```java
@Value("张三")
private String name;
@Value("18")
private Integer age;
```

加载配置文件

```java
@Value("${person.nickname}")
private String nickname;
    
@PropertySource(value = "classpath:/application.properties")
@Configuration
public class MyConfigPropertyValue {

    @Bean
    public Person01 person01(){
        return new Person01();
    }
}
```

#### @Autowired

```java
@Qualifier("bookDao") //指定需要装配的组件的id，而不是类名
@Autowired(require = false) //不是强制装配bean
```

#### @Profile

> 作用：spring为我们提供的可以根据当前环境，动态的激活和切换一系列组件的功能

#### AOP

> 在程序的运行期间将某段代码动态的切入到某个方法的指定位置的编程方式

##### 依赖支持

```xml
<!--导入AOP依赖支持-->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>4.3.12.RELEASE</version>
</dependency>
```

##### 业务逻辑类

```java
public class MathCalculator {

    public int divide(int i,int j){
        System.out.println("divide目标方法已被调用...");
        return i/j;
    }
}
```

##### 切面类

```java
@Aspect
public class LogAspects {

    /**
     * 抽取公共的切入点表达式
     * 1.当前类引用
     * 2.外部类引用
     */
    @Pointcut("execution(* com.hacker.aop.MathCalculator.*(..))")
    public void pointCut(){ }

    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint){
        Object[] args = joinPoint.getArgs();
        System.out.println("前置通知 "+joinPoint.getSignature().getName()+" method start... parameters is"+ Arrays.toString(args) +"...");
    }

    @After("com.hacker.aop.LogAspects.pointCut()")
    public void logEnd(JoinPoint joinPoint){
        System.out.println("后置通知 "+joinPoint.getSignature().getName()+" method end...");
    }

    /**
     * 注意：如果有多个参数传入则，joinPoint必须写在第一个参数的位置上
     * @param object 用Object来封装返回值
     */
    @AfterReturning(value = "pointCut()",returning = "object")
    public void logReturn(JoinPoint joinPoint,Object object){
        System.out.println("返回通知 "+joinPoint.getSignature().getName()+" method return...result is "+object+"...");
    }

    @AfterThrowing(value = "pointCut()",throwing = "exception")
    public void logException(JoinPoint joinPoint,Exception exception){
        System.out.println("异常通知 "+joinPoint.getSignature().getName()+" method exception "+exception+"...");
    }
}
```

##### 配置类

```java
/**
 * AOP思想：在程序的运行期间将某段代码动态的切入到某个方法的指定位置的编程方式
 * 应用场景：定义一个业务逻辑类，调用该方法时进行日志输出。
 * 通知方法:
 * 前置通知:在目标方法运行之前运行
 * 后置通知:在目标方法运行结束运行(无论方法是正常结束还是异常结束)
 * 返回通知:在目标方法运行正常返回通知
 * 异常通知:在目标方法运行出现异常通知
 * 环绕通知:在目标方法运行执行前后通知
 *
 * 1.给切面类的目标方法添加注解
 * 2.将切面类和业务逻辑类都加入spring容器中
 * 3.告诉spring容器哪个是切面类 @Aspect
 * 4.开启基于注解的切面编程 @EnableAspectJAutoProxy
 */
@EnableAspectJAutoProxy
@Configuration
public class MyConfigAOP {

    @Bean
    public MathCalculator mathCalculator(){
        return new MathCalculator();
    }
    
    @Bean
    public LogAspects logAspects(){
        return new LogAspects();
    }
}
```

##### 测试类

```java
    @Test
    public void testAOP(){
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfigAOP.class);
        MathCalculator bean = context.getBean(MathCalculator.class);
        bean.divide(1,1);
        context.close();
    }
```

