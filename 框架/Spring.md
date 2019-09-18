## Spring系列

### 1.一些重要的模块

- **Spring Core：** 基础,可以说 Spring 其他所有的功能都需要依赖于该类库。主要提供 IoC 依赖注入功能。
- **Spring Aspects** ： 该模块为与AspectJ的集成提供支持。
- **Spring AOP** ：提供了面向切面的编程实现。
- **Spring JDBC** : Java数据库连接。
- **Spring JMS** ：Java消息服务。
- **Spring ORM** : 用于支持Hibernate等ORM工具。
- **Spring Web** : 为创建Web应用程序提供支持。
- **Spring Test** : 提供了对 JUnit 和 TestNG 测试的支持。

### 2.@RestController vs @Controller 不懂

Controller返回一个页面，单独使用@Controller不加@ResponseBody的话，一般使用在要返回一个视图的情况，这种就属于比较传统的Spring MVC的应用，对应于前后端不分离的情况。

@RestController返回JSON或者XML形式数据

但是RestController只返回对象，对象数据直接以JSON或XML形式写入HTTP Response，这也是最常用的前后端分离。

### 3.Spring IOC & AOP

#### 1.说说Spring IOC吧

IOC就是控制反转，让IOC容器负责对象的生命周期和对象之间的关系。底层用工厂+反射机制和配置文件来实现。 **IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。**

比如我们用ClassPathXmlApplicationContext的方式去加载配置文件，那我们就可以用getBean拿到我们要的类的类类型，也就是那个类的字节码，然后用类的类类型去获取类中的方法。

BeanFactory是生产Bean 的工厂，它负责生产和管理各个bean实例。ApplicationContext就是一个BeanFactory，那ClassPathXmlApplicationContext是继承BeanFactory，

有好处就有坏处：IOC也让生成对象的过程变得复杂，而且还需要很多的配置。

#### 2.说说AOP

AOP面向切面编程，是一种编程范式，它采用横向抽取机制，他的作用是可以把核心业务逻辑和安全、事务、日志等这些非核心业务逻辑分离，让业务逻辑之间的关系更简洁。他的主要目的是在不修改原来代码的情况下，通过代理类实现增强。

AOP实现的关键在于代理模式，他有静态代理和动态代理，静态代理可以用AspectJ，动态代理有JDK动态代理和CGLIB动态代理。

AspectJ是静态代理是真实存在的，也就是说会在编译阶段生成代理类，运行的时候就直接运行增强之后的AOP对象。  简单来说静态代理就是把被代理的类作为参数传递给代理类的构造方法，让代理类替代理类实现更强大功能。但是在实际的项目中我们不能为每一个需要代理的对象都再去写一个类，太冗余。

动态代理就是说每次在运行的时候在内存中临时为方法生成一个AOP对象，这个对象包含了目标对象的全部方法，并且在切点做了增强处理，并回调原对象方法。并不是真正存在一个这样的类。

①JDK动态代理，面向接口进行代理，动态生成接口的代理类，目标必须有接口。核心是InvocationHandler接口和Proxy类，Proxy会为我们的目标生成代理对象，然后InvocationHandler 通过invoke()方法来调用目标类中的代码，我们可以在invoke方法之前和之后写需要增加的功能。

```java
//客户接口类
public interface CustomerService{
  public void save();
  public void update();
}
public class CustomerServiceImpl implements CustomerService{
  public void save(){
    System.out.println(调用save());
  }
  public void update(){
    System.out.println(调用update()）；
  }
}
//使用JDK动态代理
public class Test implements InvocationHandler{
  //增强
  public void testJdkProxy(){
    //new一个对象，对这个对象生成代理对象，调用代理对象的方法
    //目标，就是我们的被代理对象
    CustomerService sc=new CustomerServiceImpl();
    //生成代理类,它会返回代理对象的实例,参数：目标对象类加载器，目标对象接口，回掉接口
    Proxy.newProxyInstance(sc.getClass().getClassLoador(),sc.getClass.getInterface(),this);
    
    @Override
    public Object invoke(Object proxy,Method method,Object []args){
      //调用原来的方法
      //增强
      method.invoke();
      //增强
    }
    
  }
}
```

②CGLIB不要求我们的目标实现接口，可以用CGLIB的增强器去实现一个代理类，然后我们需要实现MethodInterceptor接口重写intercept去写我们增强的方法。那么Spring AOP会选择使用CGLIB来动态代理目标类。CGLIB（Code Generation Library），是一个代码生成的类库，可以在运行时动态的生成指定类的一个子类对象，并覆盖其中特定方法并添加增强代码，从而实现AOP。CGLIB是通过继承的方式做的动态代理，因此如果某个类被标记为final，那么它是无法使用CGLIB做动态代理的。

如果目标使用了接口，那Spring自动使用JDK，如果没有自动使用CGLIB。他们两个的主要区别在于生成AOP代理对象的时机不同。

#### 3. Spring AOP 和 AspectJ AOP 有什么区别？

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。** Spring AOP 基于代理(Proxying)，而 AspectJ 基于字节码操作(Bytecode Manipulation)。

Spring AOP 已经集成了 AspectJ ，AspectJ 应该算的上是 Java 生态系统中最完整的 AOP 框架了。AspectJ 相比于 Spring AOP 功能更加强大，但是 Spring AOP 相对来说更简单，

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 AspectJ ，它比Spring AOP 快很多。

### 4.Spring如何装配Bean，bean的注入方式有哪些

这是装配方式：
xml配置，<bean id="">
@Bean注解配置
自动装配，开启注解扫描用AutoWired注解。
这是注入方式：
构造器注入，还有set方法注入。

### 5. Bean的作用域

单例（Singleton）：在Spring IOC容器中仅存在一个Bean实例，默认。
原型（Prototype）：每次从容器中调用bean都返回一个新的实例。
会话（Session）：同一个http session共享一个bean，不同session使用不同的bean，就是在Web应用中，为每个会话创建一个bean实例。
请求（Request）：每次http请求都会创建新的bean，该bean只会在当前的HTTP request内有效，为每个请求创建一个bean实例。
globalSession：一般用于Portlet应用环境，在基于 portlet 的 web 应用中才有意义。 默认情况下Spring中的bean都是单例的。

#### 5.1 Spring 中的单例 bean 的线程安全问题了解吗？

大部分时候我们并没有在系统中使用多线程，所以很少有人会关注这个问题。单例 bean 存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

常见的有两种解决办法：

1. 在Bean对象中尽量避免定义可变的成员变量（不太现实）。
2. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）。

#### 5.2  @Component 和 @Bean 的区别是什么？

1. 作用对象不同: `@Component` 注解作用于类，而`@Bean`注解作用于方法。

2. `@Component`通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 bean,`@Bean`告诉了Spring这是某个类的示例，当我需要用它的时候还给我。

3. `@Bean` 注解比 `Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring`容器时，则只能通过 `@Bean`来实现。

   `@Bean`注解使用示例：

   ```java
   @Configuration
   public class AppConfig {
       @Bean
       public TransferService transferService() {
           return new TransferServiceImpl();
       }
   }
   ```

   上面的代码相当于下面的 xml 配置

   ```java
   <beans>
       <bean id="transferService" class="com.acme.TransferServiceImpl"/>
   </beans>
   ```

   下面这个例子是通过 `@Component` 无法实现的。

   ```java
   @Bean
   public OneService getService(status) {
       case (status)  {
           when 1:
                   return new serviceImpl1();
           when 2:
                   return new serviceImpl2();
           when 3:
                   return new serviceImpl3();
       }
   }
   ```

   

### 6. bean的生命周期

Bean容器找到配置文件中 Spring Bean 的定义。
Bean容器利用Java Reflection API创建一个Bean的实例。
如果涉及到一些属性值 利用set方法设置一些属性值。
如果Bean实现了BeanNameAware接口，调用setBeanName()方法，传入Bean的名字。
如果Bean实现了BeanClassLoaderAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例。
如果Bean实现了BeanFactoryAware接口，调用setBeanClassLoader()方法，传入ClassLoader对象的实例。
与上面的类似，如果实现了其他*Aware*接口，就调用相应的方法。
如果有和加载这个Bean的Spring容器相关的BeanPostProcessor对象，执行postProcessBeforeInitialization()方法
如果Bean实现了InitializingBean接口，执行afterPropertiesSet()方法。
如果Bean在配置文件中的定义包含init-method属性，执行指定的方法。
如果有和加载这个Bean的Spring容器相关的BeanPostProcessor对象，执行postProcessAfterInitialization()方法
当要销毁Bean的时候，如果Bean实现了DisposableBean接口，执行destroy()方法。
当要销毁Bean的时候，如果Bean在配置文件中的定义包含destroy-method属性，执行指定的方法。

### 7. SpringMVC的工作流程：

1.客户端request请求到前端控制器（DispatcherServlet）
2.前端控制器请求查找Handler到HandlerMapping
3.HandlerMapping返回一个执行链
4.然后前端控制器再去找HandlerAdapter，
5.HandlerAdapter去执行Handler
6.返回Model and View
7.前端控制器会让视图解析器去解析
8.视图解析器再返回View给前端控制器
9.前端控制把View给到jsp返回给用户。

### 8. IOC源码部分，一个Bean是怎么出来的

比如在xml文件中有一个这样的类：

```java
<bean id="XiaoWang" class="com.springstudy.talentshow.SuperInstrumentalist">
    <property name="instruments">
        <list>
            <ref bean="piano"/>
            <ref bean="saxophone"/>
        </list>
    </property>
</bean>
```







### 6.AOP的一些概念，如果没有问可以不说

通知：就是拦截到需要增强的方法后需要做的事情，也叫通知。就是增强的功能。
AOP中的通知有前置通知Before（在目标方法被调用之前调用通知功能），后置通知After（在目标方法被调用或者抛出异常之后都会调用通知功能），返回通知After-turning（在目标方法成功执行之后调用通知），异常通知After-throwing（在目标方法抛出异常之后调用通知），环绕通知Around（通知包裹了被通知的方法，在目标方法被调用之前和调用之后执行自定义的行为。）

连接点：那些能拦截到的就是方法
切入点：需要增强的方法，就是切入点。是链接点中的某一个
目标：增强之后的类
切面：就是定义需要对哪些方法进行怎样的增强











