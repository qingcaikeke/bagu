



## 设计模式

[两万字盘点那些被玩烂了的设计模式 (qq.com)](https://mp.weixin.qq.com/s/vWkdyKw3QgE2ABfJDqS7pA)

#### 总览

重点：单例，装饰器

手撕：单例，观察者

**一两句话讲讲什么是设计模式**

软件设计中大家遇到的常见问题的解决方案，防止写出屎山代码，便于扩展维护

**设计模式 6 大原则** 

单一职责(一个类和方法只做一件事)、里氏替换(子类继承父类，除添加新增功能外，尽量不要重写父类的方法，即改变原有的功能)、

依赖倒置(细节依赖抽象，下层依赖上层)、接口隔离(建立单一接口，不要建立庞大的接口供所有类调用，如创建和适用分离)、

迪米特原则(一个类最少知道其他类，降低耦合)、开闭原则(对扩展开放，对修改关闭，多用接口，用抽象构建框架)

**分类：创建型、结构型和行为型。**对应了面向对象开发的三个问题：如何**创建**对象、如何**组合**对象、如何处理对象间的**通信和职责分配**。

- **创建型模式**是关于对象创建过程的总结，包括**单例、工厂、抽象工厂、建造者和原型模式**。
- **结构型模式**是针对结构的总结（如何将对象和类组成更大的结构），包括桥接、**适配器**、**装饰者**、**代理**、组合。（一个类变成另一个类）
- **行为型模式**是对象间的通信和职责划分（多个对象协同完成任务），包括**策略**、解释器、命令、**观察者**、迭代器、**模板方法**和访问者模式。

#### 单例模式

正常就是私有构造，公有get(方法1)，但会有线程安全问题，所以方法加锁(方法2)，但这样太重了，所以双重检查(方法3)，饿汉，类加载，声明时直接初始化静态变量(方法4)，内部类类似于双重检查(方法5)，是4的优化，利用了内部类只有在被调用的时候才加载。

方法1，3用的最多

应用场景：线程池，连接池，日志对象，配置对象。保证资源合理利用和数据一致性

##### 手撕

懒汉使用时才创建，饿汉类加载就创建。

饿汉

```
private static final Singleton instance = new Singleton();
```

双重检查，懒汉：防止重排序：分配空间，初始化，instance对象指向该地址

```java
public class Singleton {
    private static volatile Singleton instance; //static，volatile
    private Singleton() {
        // 私有构造方法
    }

    public static Singleton getInstance() { //static必须，
        if (instance == null) { 	//1.外层null减少加锁次数 
            synchronized (Singleton.class) { // 锁啥
                if (instance == null) { 	//2.内层null防止创建多个对象
                    instance = new Singleton(); // volatile防止这行重排序
                }
            }
        }
        return instance;
    }
}
```

#### 真题：

常见的设计模式：按创建，结构，行为型来回答

单例，工厂，建造者

代理

策略，观察者，责任链



单例模式有没有线程不安全的情况，有，方法1

反射能破坏单例吗：**反射**可以调用私有构造方法，所以可能会破坏单例，解决方案：可以防御，在构造方法里加一个检测，检测到有实例直接抛异常



#### 创建型

**简单工厂**：分离对象的创建和使用，客户端不用知道具体的创建的流程

**工厂方法模式**：提供一个创建对象的接口，但允许子类决定实例化哪个类。这样，工厂模式使得一个类的实例化延迟到其子类,实现开闭原则。（对扩展开放，对修改关闭）

```
interface AnimalFactory ->> class CatFactory implements AnimalFactory  //将创建过程延迟到子类
AnimalFactory animalFactory = new CatFactory();
Animal cat = animalFactory.createAnimal();
```

**抽象工厂：**针对多个产品，于创建一系列相关或相互依赖对象，而无需指定它们的具体类。以形成一个产品族。

举例：配置信息创建工厂，工厂创建实例

mybatis中：使用者通过session执行sql语句，使用者只需调用factory的openssion就能创建，但实践上创建会话是一个复杂的过程，源码做了封装，首先有一个 interface SqlSessionFactory ，里面有一个open方法，defaultFactory实现了这个方法，具体的实现策略是调用子集扩展的openFromDataSorce方法，里面有一大坨

spring中：创建bean，有一个interface BeanFactory

**建造者模式**：builder.build，用于一步一步的创建复杂对象，分离构造和装配，将对象的构建细节隐藏在具体的建造者类。是按照指定的蓝图去构建产品，通过组装零部件而产生一个新产品。角色包括：产品，抽象建造者，具体建造者，指挥者。

客户端只需要知道想要的产品，不需要知道产品的内部组成细节，解耦产品本身与产品创建过程

**补：**工厂和建造者都是为了分离创建过程和使用过程。或者说都是把创建逻辑封装起来

**原型模式**

用原型实例指定创建对象的种类，并通过拷贝这些原型，创建新的对象。（深拷贝浅拷贝都有）

不用知道具体的类，只需要有一个实例对象即可

#### 结构型

适配器是让已经开发的适配不同用户的；装饰器是增加新功能的；代理也是增加新功能的

**适配器模式**

将一个类或接口（被适配对象）通过适配器转换成将客户端所期望的接口（接口不同的类协同工作，电源适配器）

适配器（Adapter）： 实现了目标接口，并在内部持有一个需要适配的对象。适配器的工作是将客户端的调用转发到被适配对象上，同时进行必要的接口转换。 

客户端传入参数给适配器，适配器调用自己的属性，处理参数

例：mybatis提供一个日志接口，不同的日志框架只要适配这个接口就行，内部依赖自己框架的对象，然后通过这个对象实现功能，调用的时候首先构造适配器对象，然后通过适配器对象调用方法（方法是与接口匹配的）

例：把InputStream对象传入适配器InputStreamReader，适配器调用自己的属性StreamDecoder，把InputStream对象转变为StreamDecoder对象，之后再做具体的工作

创建适配器类实现客户接口，这样客户就可以直接调了
注入已经编写好的原来的服务逻辑当属性
重写客户接口，接口内调原来逻辑的方法

**装饰器模式**

创建一个包装类，动态的将新功能添加到对象中-------不改变原有对象的情况下扩展其功能，适用于继承比较复杂的场景

一个画画接口(Shape)，一个画圆实现类(draw)，一个画方实现类(draw)，增添颜色功能，一个抽象装饰类(draw,shape)，一个实体装饰类，增添方法，用于draw中。为什么不用继承，shape**子类太多**，继承太麻烦

java io中使用到，用于增强stream流的行为，如BufferedInputStream多了缓冲功能

**代理模式**：创建一个代理类实现原对象功能的增强，接口--实现类--代理对象（继承接口，实现类对象作为成员变量，重写方法（新增方法+调用成员对象原方法））

例：加一个二级缓存功能，先从二级缓存查，查不到就通过被代理的对象查找数据

#### 行为型

**策略模式：**根据不同的对象类型选择不同的方法，原来是if-else，但这样加新的方法需要改代码，策略模式先定义一个**策略接口**（比方说要完成一个多平台通知任务，要有通知功能，要能根据type查询当前实现类是否支持这一功能），然后有**策略实现类**，调用时通过list或map找到满足要求的的实现类(可以多个)，就能调用方法完成任务。之后新增策略只需要让新的算法实现抽象策略

**本质上把类的不同行为进行封装，使得程序可以进行动态的扩展和替换，增加了程序的灵活性。**

spring中@PathVariable、@RequestParam、@RequestBody等注解，SpringMVC判断使用了哪种注解并处理注解
spring中对返回值的判断也用了策略模式，识别他是那种类型，调用对应的方法

**例：**根据星期几采用不同的折扣策略，环境接收日期构造自己的策略属性，调用方法实现策略的切换

**例：**假设现在有一个需求，需要将消息推送到不同的平台，短信通知，app通知

首先设计一个策略接口，然后短信通知实现，app通知实现，Notifier接口，xxNotifier实现，然后加载所有的MessageNotifier，循环遍历，一旦support返回true，就掉要该类型的notify

```
public interface MessageNotifier {
	boolean support(int notifyType);//是否支持该类型
    void notify(User user, String content);
}
public class SMSMessageNotifier implements MessageNotifier {}
public class AppMessageNotifier implements MessageNotifier {}

private List<MessageNotifier> messageNotifiers;
public void notifyMessage(User user, String content, int notifyType) {
    for (MessageNotifier messageNotifier : messageNotifiers) {
        if (messageNotifier.support(notifyType)) {
            messageNotifier.notify(user, content);
        }
    }
}
```

**模板方法**：父类定义一个操作的框架，具体的一些步骤等待子类实现，使得子类可以在不改变算法结构的情况下重新定义算法的某些步骤。

**责任链模式**：将很多对象由每一个对象对其下一家的引用连接起来形成一条链

责任链和策略相比是互斥的，过滤一个再过滤下一个，如果中间某一个返回了，就不需要再走后面的了（也可以一直走，代表完成所有流程）

责任链模式：一堆handler，每个负责处理不同的请求，有指向下一个请求的指针，并不在自己的处理范围内，就找下一个handler

spring中：HandlerInterceptor拿到所有的Interceptor[]，然后for循环依次调用preHandle

```
public abstract class ApprovalHandler {
	protected ApprovalHandler next;//下一个对象
	public abstract void approval(ApprovalContext approvalContext);//调用本对象处理
    protected void invokeNext(ApprovalContext approvalContext) {//调用下一个对象处理
        if (next != null) {
            next.approval(approvalContext);
        }
    }	
}
```

补：Service Provider Interface (SPI) 机制是一种基于接口的服务发现机制。允许开发者定义一组接口，并在运行时动态替换其实现，允许在不修改代码的情况下，通过替换实现类来改变程序的行为。

**观察者模式**：对象间存在**一对多的依赖**关系，一个对象被修改，自动通知依赖他的所有对象，实现**状态变更的及时感知**

主题接口（addObserver，notifyObserver（遍历list，调用update））观察者接口（update）

### Mabatis

#### **MaBatis中的设计模式**

**工厂模式**：SqlSessionFactory-》DefaultSqlSessionFactory（+Configuration） -》实例化

**单例**：Configuration，LogFactory， ErrorContext

**创建者**：到处都是 XxxxBuilder，所有关于 XML ⽂件的解析到各类对象的封装，都是⽤建造者以及建造者助⼿来完成对象的封装。

它的核心目的的就是不希望把过多的关于对象的属性设置，写到其他业务流程中，⽽是⽤建造者的⽅式提供最佳的边界隔离

**适配器：**有太多的⽇志框架，Mybatis 定义了⼀套统⼀的⽇志接⼝，为所有的其他⽇志⼯具接⼝做相应的适配操作。

**代理模式**：一切框架的基石，dao层调用的crud方法，会由代理类接管，调用一系列操作，返回数据库的执行结果

**模板方法**：`BaseExecutor` 实现了一系列算法的骨架，其中一些方法是抽象的，由具体的子类实现。即允许⼦类在不修改结构的情况下重写算法的特定步骤

**装饰器模式**：把原对象放到包含行为的特殊对象中，来为原对象绑定新的行为。比方说原来有 DataService》MyDataService，想加一个缓存功能CacheDecorator，把mydata注入到cache中，有缓存走缓存，没缓存走mydata

**观察者模式**：注册监听器来监听执行过程中发生的事件，包括SQL 执行前后、事务提交或回滚前后等

手撕观察者：

```
// 主题接口
interface Subject {
    void addObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers(String message);
}
//主题实现类
class ConcreteSubject implements Subject {
    private List<Observer> observers = new ArrayList<>();
    //省略add和remove
    @Override
    public void notifyObservers(String message) {
        for (Observer observer : observers) {
            observer.update(message);
        }
    }
    // 某些操作触发通知
    public void doSomething() {
        // 做一些事情
        // 通知观察者
        notifyObservers("Something happened!");
    }
}
// 观察者接口
interface Observer {
    void update(String message);
}
//观察者实现类
```

### MaBatis

##### 如何防止sql注入的

预编译固定 sql 语句结构，#{}占位，参数值安全处理后，插入预编译的 sql

**MyBatis的优点？**

ORM架构：完成java对象到sql参数的映射，把sql语句和代码分离，在xml映射文件中通过标签来写sql

### idea

bookmarks：可以看到断点位置

structure：当前类文件的结构，可以看到 方法名-入参-返回值，字段名-类型-初值

### Spring

[Spring面试题，41道Spring八股文（1.3万字63张手绘图），面渣逆袭必看👍 | 二哥的Java进阶之路](https://javabetter.cn/sidebar/sanfene/spring.html#_23-spring-事务的种类)

**Spring扫描规则：**启动类所在包及其子包

**创建新项目顺序：**配置文件 - 启动类 - 实体类(对应数据库的) - controller，service，mapper

#### MVC流程

![image-20250221151940241](C:\Users\16776\AppData\Roaming\Typora\typora-user-images\image-20250221151940241.png)

找handler本质就是根据url找controller

#### **执行web请求流程**

1. **接收请求**

   点击 "Run" 启动boot项目，通常会启动一个内嵌的Web服务器，比如Tomcat。

   内嵌的Web服务器会维护一个线程池，用于处理接收到的HTTP请求。线程池可配置，以便在高并发情况下有效地处理多个请求。

   当有HTTP请求到达时，Web服务器从线程池中获取一个线程来处理这个请求。

2. **获取Session：**

​		在处理请求之前，Web服务器通常会创建一个`HttpServletRequest`对象，其中包含了当前请求的相关信息。

​		请求session由web服务器中的session管理器管理

​		如果请求携带了Session ID（通常存储在Cookie中），Web服务器会尝试从Session管理器中获取对应的Session。

​		获得请求信息和Session，Web服务器会将请求传递给Spring框架。

3. **经过拦截器：**

   拦截器允许在请求处理前或处理后执行一些逻辑，例如权限验证、日志记录等。

   在拦截器中，可以访问请求和响应对象，以及Session信息，并在需要的情况下将信息存储到`ThreadLocal`中。

4. **执行业务逻辑返回：**

   一旦通过拦截器，执行业务逻辑，请求将被传递到对应的Controller，service，mapper。执行完成后返回响应，web服务器的线程释放回线程池

**Spring MVC**：Spring的一个子模块，提供帮助web开发的一些组件，能把http请求映射到特定的controller，然后返回相应的数据给客户端，提供一个MVC架构，补：controller返回的是modelandview，然后前端控制器渲染，将模型数据填充到request

**Spring：**功能更多的一个框架，提供ioc，aop等

**Spring Boot：**对Spring进行扩展，实现自动配置，简化Spring程序的开发部署

补ddd：mvc不太区分业务流程，而是用很多零散的 service 来组装业务流程。domain 领域就是业务流程的拆解

##### @Autowired 和 @Resource 注解有什么区别？

auto是spring框架的注解，通过byName注入；resource是java本身的注解，通过byType注入

#### **IOC原理及实现**

**控制**就是对象的创建、初始化、销毁。

**反转**就是原来创建对象是程序员手动控制，依赖关系通过直接在类内部创建对象。这样如果有大量组件就会产生紧密的耦合，导致复杂度增大，测试和维护困难（有一个student类，有一个score类，需要先new strudent(),new score,student.setScore(score)）

**实现方式：**ioc把控制权转移到容器，把bean对象注入到容器，然后通过注解或xml把依赖关系描述出来，让容器管理创建并装配bean

**补：**AppConfig标注了@Configuration，表示它是一个配置类，它包含一个component注解
此外，AppConfig还标注了@ComponentScan，它告诉容器，自动搜索当前类所在的包以及子包，把所有标注为@Component的Bean自动创建出来，并根据@Autowired进行装配。

### bean

+	什么是bean：交给容器管理的对象，通过注解或xml标记

+ 将类声明bean的注解：@component@controller@Service

+ @bean和@component：前者作用于方法，配置类中的方法上加@bean注解，指示spring初始化的时候调用这个方法，把返回对象注册成一个bean

```java
@Configuration
public class XXConfiguration {
    @Bean
    public XX jsonTemplate() {
		解析properties，注入属性，返回对象
        return view;
    }
```

+ 注入bean的注解：@Autowired 和 @Resource 

  auto是spring框架的注解，默认根据bytype注入，如果一个接口有多个实现类，注入变为byName

  resource是java本身的注解，默认byName

+ bean注入的方式

  属性注入，setter方法注入，构造函数注入；推荐使用构造函数注入，保证依赖完整，所有必需依赖在对象创建时就被注入，避免了空指针异常的风险

```java
构造函数注入
public UserService(UserRepository userRepository) {
    this.userRepository = userRepository;
}
set方法注入
@Autowired
public void setUserRepository(UserRepository userRepository) {
    this.userRepository = userRepository;
}
字段注入
@Autowired
private UserRepository userRepository;
```

**Bean 的注册过程**

容器扫描配置文件或注解，把bean定义加载到容器，根据bean作用域（单例，原型）创建并管理其生命周期

注册

本质上就是创建对象，然后注册到beanDefinitionRegistry，beanDefenition包括bean的ClassName，还有是否是Singleton、对象的属性和值等

**bean生命周期：**实例化、依赖注入、初始化、销毁等

**bean的循环依赖**

三级缓存，一级保存可使用的bean，二级保存半成品（早期对象），三级保存创建工厂，二级缓存作为循环出口

具体过程 1.创建一个空bean存入一级，2.进行属性赋值，如果发现循环依赖，把当前bean对象提前暴漏（半成品bean），完成初始化后放入二级缓存，

3.继续进行依赖注入，如果发现循环依赖，从二级缓存中获取已经初始化的bean实例

**bean作用域：**单例、原型、会话、请求等，可以考虑使用Map来存储不同作用域的Bean实例。

**怎么实现bean单例**：一个表存已经创建的bean，更新这个表靠类似单例模式，双重验证，判断加过没，加锁，再判断（单例池） 

补：原型：每次请求都会创建一个新的实例，使用完成后立即销毁。

补：clone创建全新的对象，内容相同，地址不同

**spring bean注入的设计模式**  todo

[Spring 中经典的 9 种设计模式，打死也要记住啊！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/114244039#概览)

### **AOP**：

底层是动态代理，通过责任链模式管理通知的执行顺序，前置通知，后置通知，环绕通知等

在 OOP 中最小的单元就是“Class 对象”，但是在 AOP 中最小的单元是“切面”。一个“切面”可以包含很多种类型和对象，对它们进行模块化管理，例如事务管理。

AOP能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

核心概念：切面（Aspect）切面类，包含一组通知和切点

连接点（Join Point）：即需要在哪个地方执行这段方法增强代码

通知（Advice）：函数@Before@after

切点（Pointcut）：execution，切点是一组连接点的集合

```
1.原本的方法代码，也叫被增强的业务类
2.定义切面类，加aspect注解，定义增强逻辑，before注解指明拦截什么(自定义注解的全类名)，方法入参joinPoint
3.调用原本方法，切面类拦截，执行前逻辑，执行原本方法，执行后逻辑
```

```java
@Aspect
public class LogAspect implements InvocationHandler{
 	private Object target; // 被代理的对象
    @Before("execution(* com.example.UserService.*(..))") //什么时候拦截
    public void logBefore(ProceedingJoinPoint joinPoint) {
        System.out.println("日志记录：用户管理操作开始");
    }
}
```

补：动态代理的底层是反射

补：@Target(ElementType.METHOD) @Retention(RetentionPolicy.RUNTIME)

自定义注解的两个参数，一个表示注解用在哪，类属性方法

一个表示注解什么阶段可见：仅源文件保留；class文件保留但不会被虚拟机读取；一直保留，运行时可以通过反射获取，aop必须设成runtime

#### jdk和cglib(**AOP实现原理**)

一句话总结：jdk创建的代理对象是一个实现了接口的对象；cglib创建的代理对象是一个类的子类对象

jdk是基于接口实现的。利用反射，把原类的类加载器和原类的接口数组，以及编写的增添新逻辑的InvocationHandler传入，创建一个代理对象。代理对象调用与原接口方法名相同的方法

cglib基于继承实现。它通过字节码技术在运行时创建被代理类的子类，并覆盖其中的方法，在方法调用前后插入额外的逻辑。会动态生成代理类（子类）的字节码

如果目标对象实现了接口，优先考虑使用 JDK 动态代理；如果目标对象是具体类且没有实现接口，则使用 CGLIB 动态代理。

```java
interface Subject {
    void request();
}
class RealSubject implements Subject {
    @Override
    public void request() {
    }
} 
class ProxyHandler implements InvocationHandler {
    private Object target;
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // todo 方法增强
        Object result = method.invoke(target, args);
        return result;
    }
}
public static void main(String[] args) {
        RealSubject realSubject = new RealSubject();
        ProxyHandler proxyHandler = new ProxyHandler(realSubject);
        Subject proxySubject = (Subject) Proxy.newProxyInstance(
                Subject.class.getClassLoader(),
                new Class<?>[]{Subject.class},
                proxyHandler
        );
        proxySubject.request();
}
```



```java
// 实现MethodInterceptor接口
class CglibProxyInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("Before method call");
        Object result = proxy.invokeSuper(obj, args);
        System.out.println("After method call");
        return result;
    }
}    
    public static void main(String[] args) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(ConcreteSubject.class); // 具体类
        enhancer.setCallback(new CglibProxyInterceptor());
        ConcreteSubject proxySubject = (ConcreteSubject) enhancer.create();
        proxySubject.request();
    }
```

JDK：通过反射，生成一个代理类，并获得接口信息，代理类实现了原来那个类的全部接口，并对接口的所有方法做了代理

CGLib：操作字节码，重写类的方法，把额外的逻辑编到方法中，实现方法的增强

切面类实现InvocationHandler，构造函数接收被代理的对象，然后invoke调用方法完成增强

#### spring中的事务

底层就是数据库对事务的支持，提交和回滚都是数据库实现的，Spring只提供了事务管理接口

声明式事务：@Transactional，基于动态代理，在目标方法执行完之后根据执行情况提交或者回滚事务。

**事务传播机制**

用于定义一个事务方法中调用另一个事务方法时，应该如何处理

例：有方法 A 和方法 B，方法 A 没有事务，方法 B 使用`PROPAGATION_REQUIRED`。当方法 A 调用方法 B 时，方法 B 会创建一个新事务来执行；若方法 A 本身在一个事务中，方法 B 会加入到方法 A 的事务中。

required：默认，没事务就创建一个，有事务就加入这个事务

support：有事务就用当前的，没事务就以非事务执行

mandatory：必须在已存在的事务中执行，否则抛异常

requiresNew：总是创建一个新事务，如果当前有事务，就把当前事务先挂起，两事务独立

spring中大小子事务回滚情况：大事务就是主方法，小/子事务就是被调用方法，以reqired为例，大事务回滚，子事务一定回滚，小事务回滚，大事务也要回滚，除非主方法捕获了子方法异常并做了处理

**事务失效**

非public方法；事务传播设置错误；异常被捕获

同一个类中非事务方法调用事务方法：因为底层需要调用代理对象，而同一个类中调用的是this对象

**拦截器和过滤器（filter）的区别**

过滤器在Spring外面，不涉及业务逻辑，一般检验请求和相应的内容，如字符编码

拦截器在Spring里面，是更细粒度的控制，可以做权限检查，登录验证等

### SpringBoot

##### SpringBoot 的 Starter：

简化依赖管理的，简化项目搭建流程。是一组定义好的依赖结合，引过来直接用

自动配置：扫描类路径下的配置和资源，自动创建配置需要的bean

```java
@Configuration
@EnableConfigurationProperties(MyStarterProperties.class)
public class MyServiceAutoConfiguration {
    @Bean
    @ConditionalOnMissingBean
    public MyService myService(MyStarterProperties properties) {
        
    }
}
@ConfigurationProperties(prefix = "mystarter")
public class MyStarterProperties {
    private String xx;
}
```

spring-boot-starter-web = SpringMVC + tomcat

**Springboot的注解哪些**

核心注解：@SpringBootApplication@Configuration@ComponentScan

MVC相关：@Controller@RequestMapping  @PathVariable  @RequestParam  @RequestBody@RequestBody，`@RestController`: 是 `@Controller` 和 `@ResponseBody` 的组合

`@ConfigurationProperties`

在 Bean 上添加上了这个注解，会读取配置文件中的前缀(application.properties)，自动给 Bean 中的属性注入值。

### SpringCloud

有哪些组件

服务注册（注册中心），服务调用，降级熔断，负载均衡

网关（请求路由，权限，限流）
