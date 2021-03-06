### Spring
#### Spring框架中用到了哪些设计模式
- 工厂设计模式：Spring使用工厂模式通过BeanFactory、ApplicationContext 创建 bean 对象。
- 代理设计模式：Spring AOP功能的实现
- 单例设计模式：Spring中的Bean默认都是单例的
- 模板方法模式：Spring中的jdbcTemplate 等以Template结尾的对数据库操作的类，使用了模板模式
- 包装其设计模式：我们的项目需要连接多个数据库，而且不同的客户在对每次访问中根据需要回去访问不同的数据库。
- 观察者模式：Spring事件驱动魔性就是观察者模式很经典的一个应用
- 适配器模式：Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller。

#### SpringMVC工作原理

- 客户端发送请求，直接请求到前端控制器(DispatcherServlet **作用** 接受请求，响应结果)
- DispatcherServlet根据请求信息调用处理器映射器 HandlerMapping，解析请求对应的 Handler。
- 解析到对应的Handler（也就是Controller控制器）后，开始由HandlerAdpatyer适配器处理。
- 处理器适配器（HandlerAdapter）会根据Handler来调用真正的处理器，并处理对应的业务逻辑。
- 处理器处理完业务后，会返回一个ModelAndView对象给前端控制器，Model是返回的数据对象，View是逻辑上的View。
- 前端控制器会拿着这个MV对象到视图解析器View resolver 进行视图解析
- 找到实际的View后返回给前端控制器
- 将模型数据填充至视图进行渲染视图
- DispatherServlet相应客户

![](/img/frame/springMVC.png)

#### Restful 

REST，全称Representational State Transfer，直接翻译就是：表现层状态转化。通俗来讲，就是“资源在网络中以某种表现形式进行状态转化”。

**资源**

“资源”是RESTful中最核心的概念之一。在RESTful概念中，互联网中的每一样信息都可以定义为资源，比如文本、图片、音频、视频等。而这些资源又都可以对应一个特定的URI(统一资源定位符)，URI为每一个资源的地址或独一无二的识别符。

**表现层**

针对上面的“资源”，我们要进行相应的呈现，而且可以采用多种的呈现形式，而这些呈现形式就叫做“表现层”。就拿文本为例，我们可以呈现为JSON格式、XML格式、HTML格式，甚至二进制格式等。这就是表现层所做的事情。

**状态转化**

资源通常放在服务器端，而客户端对服务器资源的增、删、改、查等操作，便涉及到资源状态的转化。这个过程便是“ 状态转化”。在HTTP中，提供了四种常见的操作方式：GET、POST、PUT、DELETE。

####  @RestController vs @Controller

单独使用 @Controller 不加 @ResponseBody的话一般使用在要返回一个视图的情况，这种情况属于比较传统的Spring MVC 的应用，对应于前后端不分离的情况。

@RestController只返回对象，对象数据直接以 JSON 或 XML 形式写入 HTTP 响应(Response)中，这种情况属于 RESTful Web服务，这也是目前日常开发所接触的最常用的情况（前后端分离）。

所以 @Controller + @ResponseBody =@RestController


#### Spring Ioc和AOP的理解

**IoC**：控制反转。 是一种设计思想，就是**将原本在程序中手动创建对象的控制权，交由Spring框架来管理** Ioc容器是Spring用来实现IoC的载体，IoC容器实际是一个Map，存放各种对象。
**Ioc容器就像一个工厂一样，当我们需要创建 一个对象的时候，只需要配置好文件/注解即可，完全不用考虑对象是如何被创建出来的。**

源码解读：https://javadoop.com/post/spring-ioc

**依赖倒置原则(DI)：**

假设我们设计一辆汽车：假设我们设计一辆汽车：先设计轮子，然后根据轮子大小设计底盘，接着根据底盘设计车身，最后根据车身设计好整个汽车。这里就出现了一个“依赖”关系：汽车依赖车身，车身依赖底盘，底盘依赖轮子。  问题来了，如果**修改轮子**那么之前的设计都需要改。

如果换一种思路:我们先设计汽车的大概样子，然后根据汽车的样子来设计车身，根据车身来设计底盘，最后根据底盘来设计轮子。这时候，依赖关系就倒置过来了：轮子依赖底盘， 底盘依赖车身， 车身依赖汽车。这时**修改轮子** 就不会需要改那么多。

这就是**依赖倒置原则**。IoC就是依赖倒置原则的一种代码设计的思路。

IoC的三种依赖注入:(配置文件怎么弄??)

- 接口注入
- setter 方法注入
- 构造方法注入


**AOP：**  面向切面编程。

将与业务无关**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、管线控制等）封装起来**。便于**减少系统的重复代码，降低模块间的耦合度，** 并**有利于未来的可扩展性和可维护性。**

Spring AOP就是基于动态代理的，如果要代理的对象，实现了某个接口， 那么Spring AOP会使用JDK Proxy,去创建代理对象，而对于没有实现接口的对象，会使用**Cglib**.

**动态代理和Cglib异同**

java动态代理是利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。

而cglib动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

- 如果目标对象实现了接口，默认情况下会采用JDK的动态代理来实现AOP
- 如果目标实现了接口，可以强制使用CGLIB实现AOP
- 如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换

如何强制使用CGLIB实现AOP？

- JDK动态代理只能对实现了接口的类生成代理，而不能针对类
- CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法 **该类或者方法最好不要声明成final**

JDK代理是不需要第三方库的，只要有JDK环境就可以，要求如下：
- 实现InvocationHandler
- 使用Proxy.newProxyInstance产生代理对象
-  被代理的对象必须要实现接口

CGLib 必须依赖于CGLib的类库，但是它需要类来实现任何接口代理的是指定的类生成一个子类，覆盖其中的方法，是一种继承但是针对接口编程的环境下推荐使用JDK的代理
