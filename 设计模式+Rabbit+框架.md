### MQ 

[消息队列介绍 | OddFar's Notes](https://note.oddfar.com/pages/2c91a1/#四大核心概念)

**核心概念**：生产者，交换机，队列，消费者，消息可靠性（确认机制，持久化，消息过期机制，死信队列），延迟队列，顺序消费，消费幂等性

1.可以结合线程池消费 2.消息应答需要权衡吞吐量和安全性，可以自动应答和手动应答，手动应答又可以结合批量应答及窗口3.消息可以持久化到磁盘

**多线程异步和MQ异步**

**处理任务的维度：**多线程是进程内的概念，一个进程内多个线程并行处理任务，mq是通过把消息发到不同应用的不同进程来处理，**可靠性**不同，多线程数据存在内存里，程序崩溃数据会丢失，mq具备**分布式能力**，可以把任务发到不同节点储存消费

**消息队列作用（优点）**：削峰、解耦（防止调用过多api，代码复杂）、异步（防止链路过长，线程池也可以实现）

削峰：大量的写请求灌入 MQ 里，后边系统消费后慢慢写，控制在 MySQL 承载范围之内。

生产者不用关心消费者要不要消费、什么时候消费，生产者只需要把东西（消息）给消息队列，生产者的工作就算完成了。

缺点：可用性降低（需要考虑mq挂掉）复杂度提高（需要考虑重复消费，消息丢失等问题）

**消息队列如何保证消息不丢失**

使用一个消息队列，其实就分为三大块：**生产者、中间件、消费者**，所以要保证消息就是保证三个环节都不能丢失数据。

消息生产阶段：生产出来提交到mq，收到mq的ack表示提交成功（生产环境常用的是confirm模式。生产者将信道 channel 设置成 confirm 模式，一旦 channel 进入 confirm 模式，所有在该信道上发布的消息都将会被指派一个唯一的ID，一旦消息被投递到所有匹配的队列之后，rabbitMQ就会发送一个确认给生产者（包含消息的唯一ID）

储存阶段：集群部署，生产者在发布消息时，队列中间件通常会写「多个节点」，也就是有多个副本。包含消息持久化和队列持久化，队列持久化只持久化queue的元数据

消费阶段：消费者接收消息+消息处理之后才给中间件回复ack，而非收到消息就回复

![img](https://mmbiz.qpic.cn/sz_mmbiz_png/J0g14CUwaZfW5jtMU7dzOUhL6avJSwsu2mjicVf2ZjCmpS93xWFUBk07GK7hqvIdMawwKV5YjXF69jnAPyuceKQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

**如何保证消息不重复消费**？

1.解决生产者重复推送，可能是一个controller被调用了两次，没有做**接口幂等性**；也可能是推送消息是mq响应较慢，触发生产者重试

2.mq：消费者回复ack的时候mq挂了，导致重启后mq以为没有消费过，于是再次推送

3.消费者：消费了一条消息没回复ack直接挂了，重启后mq以为消费者没有消费过，于是再次推送

解决方法：1.状态判断，把消费数据（如订单id）记录在 redis 中，消费时先到 redis 中查看是否存在该消息（setnx），若已经消费过，直接丢弃消息。

2.业务判断法（常用）：通常数据消费后都需要插入到数据库中，使用数据库的唯一性约束防止重复消费。每次消费直接尝试插入数据，如果提示唯一性字段重复，则直接丢失消息。

补：幂等性：消费者返回ack时网络中断，该消息从新发送给其他消费者，造成消息重复消费。解决方法：给消息加一个全局唯一id

**顺序消费**：

拆分queue，单消费者消费指定消息，会影响性能

**创建消息队列的参数**：消息队列名称，是否持久化，是否**只允许当前这个创建消息队列**的连接操作消息队列，没有人用队列后，是否要删除队列 ，队列参数（如过期时间等）

**五种工作模式**：简单模式（单生产单消费）工作队列模式（多消费绑定到同一队列，之后轮询分发(一半一半)或公平分发(能者多劳））

发布订阅（Fanout 从这个开始加交换机，消息会广播到所有绑定交换机的队列上）

路由模式（Direct/Routing，队列注册的时候有个binding key ，根据消息的binding key发送到对应的队列去）

主题模式（Topics，点分加通配符，实现更灵活的路由匹配）

**消息过期机制**

允许你为消息设置一个过期时间，在此时间后，如果消息仍然未被消费者消费，将会自动从队列中删除。

**消息确认机制**（消费成功、消费失败、拒绝）

自动确认：一收到消息就回复ack，手动确认：可以指定某条消息回复消费结果（成功，失败，拒绝）

指定是否确认某条消息（id）+是否批量确认+指定拒绝多一个参数（是否重新放回消息队列）

**死信队列**，为了保证消息的可靠性，比如每条消息都成功消费，需要提供一个容措机制，即：失败的消息怎么处理？

通过给正常队列配置死信队列和死信交换机，将死信存到死信队列，然后再起一个消费者消费死信内容

死信**来源**：消息达到TTL，队列长度已满，消息被拒绝（给队列配置TTL，到时间立刻丢弃或进死信队列，给消息配置要等到消费者查看才能发现），消息处理失败

死信交换机，绑定正常队列与死信队列

**延时队列**：延时队列内部是有序的，希望元素在指定时间到了以后或之前取出和处理，如订单十分钟未付款取消，用户三天没登录发消息提醒。

延时队列可以通过死信队列+过期时间实现，也可以通过 Redis 的 zset或Quartz或kafka 的时间轮或Java 的DelayQueue实现

**优先级队列**：队列设置为优先级队列，消息要设置消息的优先级，从而先消费优先级高的

**惰性队列：**将消息存入磁盘中，在消费时再加载到内存中，从而支持更多的消息储存

**消息基于什么传输**

由于TCP连接的创建和销毁开销较大，且并发数受系统资源限制，会造成性能瓶颈。RabbitMQ使用信道的方式来传输数据。信道是建立在真实的TCP连接内的虚拟连接，且每条TCP连接上的信道数量没有限制

**如何处理消息堆积情况?几千万条数据在MQ里积压了七八个小时**

消费者消费速度与生产速度不匹配，看是否消费者出现异常，尽快恢复；可以临时将队列扩大n倍，消费者扩大n倍，尽快消费积压数据。

然后还要查询这个时间内是否存在消息丢失，消息过期，因此可能需要手动查询，把丢失的消息补回来

**场景：订单十分钟未支付取消：**用户下完单后存到数据库，数据库有个字段表示是否支付，然后把订单id放到延迟队列（10min后消费），在这期间，如果用户完成支付，把数据库的字段修改；等到了10min，进行消费，去数据库查订单状态，如果已支付，不用处理，未支付，将状态置为失效

## 设计模式

[两万字盘点那些被玩烂了的设计模式 (qq.com)](https://mp.weixin.qq.com/s/vWkdyKw3QgE2ABfJDqS7pA)

重点：单例，装饰器

手撕：单例，观察者

**设计模式 6 大原则** 

单一职责(一个类和方法只做一件事)、里氏替换(子类继承父类，除添加新增功能外，尽量不要重写父类的方法，即改变原有的功能)、

依赖倒置(细节依赖抽象，下层依赖上层)、接口隔离(建立单一接口，不要建立庞大的接口供所有类调用，如创建和适用分离)、

迪米特原则(一个类最少知道其他类，降低耦合)、开闭原则(对扩展开放，对修改关闭，多用接口，用抽象构建框架)

**分类：创建型、结构型和行为型。**对应了面向对象开发的三个问题：如何**创建**对象、如何**组合**对象、如何处理对象间的**通信和职责分配**。

- **创建型模式**是关于对象创建过程的总结，包括**单例、工厂、抽象工厂、建造者和原型模式**。
- **结构型模式**是针对结构的总结（如何将对象和类组成更大的结构），包括桥接、**适配器**、**装饰者**、**代理**、组合。
- **行为型模式**是对象间的通信和职责划分（多个对象协同完成任务），包括**策略**、解释器、命令、**观察者**、迭代器、**模板方法**和访问者模式。

#### 创建型

**单例模式**：
 volatile禁止重排序 创建对象分三步 a分配空间给对象b初始化对象c指针指向这个对象地址，如果发生重排序，线程1分配空间，返回地址，但还没初始化，线程2在第一个if判断有对象，直接返回了这个空对象，就会发生错误，同时volatile可以保证所有线程看到的数据都是最新的

补：**反射**可以调用私有构造方法，所以可能会破坏单例，解决方案：可以防御，在构造方法里加一个检测，检测到有实例直接抛异常

补：单例模式（懒汉2种、饿汉、双重锁、内部类），多线程并发如何保证单例（饿汉 或 锁或内部类）

用途：网站计数器。应用程序的日志应用。Web项目中的配置对象的读取。数据库连接池。线程池。

    private static final Singleton instance = new Singleton();
    饿汉
    public class Singleton {
        private static volatile Singleton instance;
        private Singleton() {
            // 私有构造方法
        }
    
        public static Singleton getInstance() {
            if (instance == null) { 	//1.外层null减少加锁次数 
                synchronized (Singleton.class) {
                    if (instance == null) { 	//2.内层null防止创建多个对象
                        instance = new Singleton();
                    }
                }
            }
            return instance;
        }
    }
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

**原型模式**

用原型实例指定创建对象的种类，并通过拷贝这些原型，创建新的对象。（深拷贝浅拷贝都有）

不用知道具体的类，只需要有一个实例对象即可

#### 结构型

**适配器模式**

将一个类或接口（被适配对象）通过适配器转换成将客户端所期望的接口（接口不同的类协同工作，电源适配器）

适配器（Adapter）： 实现了目标接口，并在内部持有一个需要适配的对象。适配器的工作是将客户端的调用转发到被适配对象上，同时进行必要的接口转换。 

客户端传入参数给适配器，适配器调用自己的属性，处理参数

例：mybatis提供一个日志接口，不同的日志框架只要适配这个接口就行，内部依赖自己框架的对象，然后通过这个对象实现功能，调用的时候首先构造适配器对象，然后通过适配器对象调用方法（方法是与接口匹配的）

例：把InputStream对象传入适配器InputStreamReader，适配器调用自己的属性StreamDecoder，把InputStream对象转变为StreamDecoder对象，之后再做具体的工作

**装饰器模式**

创建一个包装类，动态的将新功能添加到对象中-------不改变原有对象的情况下扩展其功能，适用于继承比较复杂的场景

一个画画接口(Shape)，一个画圆实现类(draw)，一个画方实现类(draw)，增添颜色功能，一个抽象装饰类(draw,shape)，一个实体装饰类，增添方法，用于draw中。为什么不用继承，shape**子类太多**，继承太麻烦

java io中使用到，用于增强stream流的行为，如BufferedInputStream多了缓冲功能

**代理模式**：创建一个代理类实现原对象功能的增强，接口--实现类--代理对象（继承接口，实现类对象作为成员变量，重写方法（新增方法+调用成员对象原方法））

例：加一个二级缓存功能，先从二级缓存查，查不到就通过被代理的对象查找数据

#### 行为型

**策略模式：**根据不同的对象类型选择不同的方法，原来是if-else，但这样加新的方法需要改代码，策略模式先定义一个**策略接口**（比方说要完成一个多平台通知任务，要有通知功能，要能根据type查询当前实现类是否支持这一功能），然后有**策略实现类**，调用时通过list或map找到需要的实现类，就能调用方法完成任务。之后新增策略只需要让新的算法实现抽象策略

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



### Spring

**Spring MVC**：Spring的一个子模块，提供帮助web开发的一些组件，能把http请求映射到特定的controller，然后返回相应的数据给客户端，提供一个MVC架构，补：controller返回的是modelandview，然后前端控制器渲染，将模型数据填充到request

**Spring：**功能更多的一个框架，提供ioc，aop等

**Spring Boot：**对Spring进行扩展，实现自动配置，简化Spring程序的开发部署

**Spring MVC的工作流程描述一下**

补：mvc不太区分业务流程，而是用很多零散的 service 来组装业务流程。domain 领域就是业务流程的拆解

**UML图**

梳理业务流程。可以使用draw.io进行绘制

类图，对象图，用例图（从用户角色的角度出发）等

**Springboot的注解哪些**

核心注解：@SpringBootApplication@Configuration@ComponentScan

MVC相关：@Controller@RequestMapping  @PathVariable  @RequestParam  @RequestBody@RequestBody，`@RestController`: 是 `@Controller` 和 `@ResponseBody` 的组合

**MyBatis的优点？**

ORM架构：完成java对象到sql参数的映射，把sql语句和代码分离，在xml映射文件中通过标签来写sql

**如何理解ioc**

**所谓控制**就是对象的创建、初始化、销毁。

原来创建对象是程序员手动控制，依赖关系通过直接在类内部创建对象。这样如果有大量组件就会产生紧密的耦合，导致复杂度增大，测试和维护困难（有一个student类，有一个score类，需要先new strudent(),new score,student.setScore(score)）

ioc把控制权转移到容器，把对象注入到容器，然后通过注解或xml把依赖关系描述出来，让容器管理创建并装配bean

**补：**AppConfig标注了@Configuration，表示它是一个配置类，它包含一个component注解
此外，AppConfig还标注了@ComponentScan，它告诉容器，自动搜索当前类所在的包以及子包，把所有标注为@Component的Bean自动创建出来，并根据@Autowired进行装配。

**bean注入的方式**

属性注入，setter方法注入，构造器注入

**Bean 的注册过程**

容器扫描配置文件或注解，把bean定义加载到容器，根据bean作用域（单例，原型）创建单例并管理其生命周期

补：原型：每次请求都会创建一个新的实例，使用完成后立即销毁。

补：clone创建全新的对象，内容相同，地址不同

**如何创建并装配一个bean**

配置类中的方法上加@bean注解，指示spring初始化的时候调用这个方法，把返回对象注册成一个bean

**如何解决bean的循环依赖**

三级缓存，一级保存可使用的bean，二级保存半成品（早期对象），三级保存创建工厂，二级缓存作为循环出口

具体过程 1.创建一个空bean存入一级，2.进行属性赋值，如果发现循环依赖，把当前bean对象提前暴漏，完成初始化后放入二级缓存，

3.继续进行依赖注入，如果发现循环依赖，从二级缓存中获取已经初始化的bean实例

**核心点：**

bean生命周期：实例化、依赖注入、初始化、销毁等

bean作用域：单例、原型、会话、请求等，可以考虑使用Map来存储不同作用域的Bean实例。

属性注入



**spring bean注入的设计模式**  todo

[Spring 中经典的 9 种设计模式，打死也要记住啊！ - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/114244039#概览)



**AOP**：底层是动态代理，通过责任链模式管理通知的执行顺序，前置通知，后置通知，环绕通知等

在 OOP 中最小的单元就是“Class 对象”，但是在 AOP 中最小的单元是“切面”。一个“切面”可以包含很多种类型和对象，对它们进行模块化管理，例如事务管理。

AOP能够将那些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来**，便于**减少系统的重复代码**，**降低模块间的耦合度**，并**有利于未来的可拓展性和可维护性**。

核心概念：切面（Aspect）切面类，包含一组通知和切点

连接点（Join Point）：即需要在哪个地方执行这段方法增强代码

通知（Advice）：函数@Before@after

切点（Pointcut）：execution，切点是一组连接点的集合

```
@Aspect
public class LogAspect implements InvocationHandler{
 	private Object target; // 被代理的对象
    @Before("execution(* com.example.UserService.*(..))")
    public void logBefore() {
        System.out.println("日志记录：用户管理操作开始");
    }
}
```

补：动态代理的底层是反射

**那SpringAOP如何实现动态代理**

JDK：通过反射，生成一个代理类，并获得接口信息，代理类实现了原来那个类的全部接口，并对接口的所有方法做了代理

CGLib：操作字节码，重写类的方法，把额外的逻辑编到方法中，实现方法的增强

切面类实现InvocationHandler，构造函数接收被代理的对象，然后invoke调用方法完成增强

**拦截器和过滤器（filter）的区别**

过滤器在Spring外面，不涉及业务逻辑，一般检验请求和相应的内容，如字符编码

拦截器在Spring里面，是更细粒度的控制，可以做权限检查，登录验证等
