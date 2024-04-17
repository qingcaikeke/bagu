Java的反射是指程序在运行期可以拿到一个**对象**的所有信息。

在运行期，对某个实例一无所知的情况下，如何调用其方法。

这3个生命周期分别对应于：Java源文件(.java文件) ---> .class文件 ---> 内存中的字节码。

java编译生成.class字节码文件，字节码文件加载到jvm中，jvm解析生成Class对象，Class对象可以用于实例化一个对象

@ConfigurationProperties，

在 Bean 上添加上了这个注解，指定好配置文件（properties）中的前缀，就会自动给 Bean 中的属性注入值。

在application.[properties](https://so.csdn.net/so/search?q=properties&spm=1001.2101.3001.7020)文件中有如下配置文件

1. `config.username=jay.zhou`

2. `config.password=333`

   

   @ConfigurationProperties(prefix = "config")

   ``
   public class TestBean{``

       private String username;
       
       private String password;

使用反射读取注解，一种方法是先判断存在，再使用。一种是直接get，如果不存在返回null

invoke调用

InvocationHandler 调用处理程序 ，决定了这个代理类到底是多了什么功能

使用反射处理注解

```
// 判断@Report是否存在于Person类:
Person.class.isAnnotationPresent(Report.class);
```

声明类型可能与实际类型不同，方法执行时取决于实际类型 -多态

执行哪个方法是由运行时动态决定的

 Income[] incomes = new Income[] {
            new Income(3000),
            new Salary(7500),
            new StateCouncilSpecialAllowance(15000)
        };

income.getTax();





这种尽量引用高层类型，避免引用实际子类型的方式，称之为面向抽象编程。

面向抽象编程的本质就是：

- 上层代码只定义规范（例如：`abstract class Person`）；
- 不需要子类就可以实现业务逻辑（正常编译）；
- 具体的业务逻辑由不同的子类实现，调用者并不关心。



抽象类与接口：能否定义字段，单继承还是多继承

接口继承接口相当于扩展了接口的方法

接口层次代表抽象程度

接口的default方法不要求一定被重写

实例字段在每个实例中都有自己的一个独立“空间”，但是静态字段只有一个共享“空间”，所有实例都会共享该字段。举个例子：

静态字段并不属于实例,实例可以访问静态字段

不推荐用`实例变量.静态字段`去访问静态字段，因为在Java程序中，实例对象并没有静态字段。在代码中，实例对象能访问静态字段只是因为编译器可以根据实例类型自动转换为`类名.静态字段`来访问静态对象。

调用实例方法必须通过一个实例变量，而调用静态方法则不需要实例变量，通过类名就可以调用。静态方法类似其它编程语言的函数。

静态方法属于`class`而不属于实例，因此，静态方法内部，无法访问`this`变量，也无法访问实例字段，它只能访问静态字段。

静态方法经常用于工具类。例如：

- Arrays.sort()
- Math.random()

不加字段表示包作用域



String比较：String.equals;

String.contains;    String.indexof;	String.lastindexof;	String.trim()	;String.split; String.join();

其他类型转换为String：String.valueof()； String转换为其他类型：integer.paresint()



`Object` 类中一些常用的方法：

- `equals(Object obj)`: 用于比较两个对象是否相等。
- `hashCode()`: 返回对象的哈希码值。
- `toString()`: 返回对象的字符串表示。
- `getClass()`: 返回对象的类。
- `clone()`: 用于创建并返回此对象的副本。
- `wait()`, `notify()`, 和 `notifyAll()`: 用于多线程的同步。
- `finalize()`: 用于在对象被垃圾回收之前调用。



`Class` 类是Java反射机制的核心类

- `getName()`: 返回类的名称。
- `newInstance()`: 创建类的新实例。
- `getMethods()`: 返回一个包含 `Method` 对象的数组，表示类或接口的所有公共方法。
- `getFields()`: 返回一个包含 `Field` 对象的数组，表示类的所有公共字段。
- `getConstructors()`: 返回一个包含 `Constructor` 对象的数组，表示类的所有公共构造方法。

你可以使用这个`myClass`对象来获取关于`MyClass`类的信息，比如类的名称、方法和字段。

这样的灵活性使得你能够在运行时根据需要操作和了解类的结构，而不需要在代码编写时就确定所有的类信息。



method就要比field 多个class参数，因为重载可以改变方法参数



右侧structure，git下面

下载一个新项目，可以在聚合项目的根目录执行mvn clean install,相当于按聚合中指明的模块顺序，先去执行clean，再去install

maven ：统一构建项目的结构，也是依赖管理工具

mvn所在的命令必须在pom.xml所在的文件使用

compile：编译，生成target包

clean：清理之前编译的结果（删除target）

maven识别测试方法不止对注解有要求，还对测试类有要求XaaTest，对方法名有要求testAXXX

package:打成jar包（会编译核心程序，测试程序，并进行测试）保存到target文件下

补：build=compile+test+package

pom引入的所有依赖都要去本地仓库找，

install将当前工程生成的jar包保存到本地仓库，会按照坐标保存到指定位置

依赖版本统一维护properties标签，${}

**mvnrepository.com**中去找依赖，不要想当然的去改版本号

下载错误解决：可能下载过程存在污染或损害，要先删除缓存（根据坐标去本地仓库中找对应的 . lastUpdated文件）

.bat:脚本文件



搭建后端工程 jdk17

1.父工程：公共依赖版本锁定，打包方式pom

其他module要注意配置父工程



**ctrl +shift+r**全局查找替换

创建容器

docker run -d --name mysql -p 3306:3306 

-v mysql_data:/var/lib/mysql -v mysql_conf:/etc/mysql 

--restart=always --privileged=true -e MYSQL_ROOT_PASSWORD=1234 mysql:8.0.30

-v把主机上名为xx的卷挂载到容器的xx目录，目的是为了持久化

卷挂载是双向的，对主机文件修改，容器内会变，对容器内修改，主机会变

`ps` 命令是“Process Status”的缩写

云服务器打开端口防火墙，本地连接远程数据库

vi/vim是linux原始的文本编辑器，可以用于创建文件

mkdir创建文件 pwd文件路径

ls看文件夹内容 :wq :wq! (强制保存并推出):q!（直接退出） :q（未修改）



存到redis：key是token val是用户信息

2.common：公共部分（），工具类，公共服务

3.model：实体类（1.dto（Data Transfer Object）接收请求参数

2.与数据库表名对应（entity）3.vo封装结果响应）

4.manager：后台功能

Invalid bound statement (not found)执行sql查询找不到指定的语句 mapper.xml中返回值错误，函数名错误，连接redis错误



spring扫描规则，启动类所在包及子包

项目流程：1.创建配置文件

2.启动类

3.实体类

4.controller service mapper



1. **启动项目：**
   - 点击 "Run" 启动项目，通常会启动一个内嵌的Web服务器，比如Tomcat。
2. **Web服务器管理线程池：**
   - 内嵌的Web服务器会维护一个线程池，用于处理接收到的HTTP请求。这个线程池可以配置，以便在高并发情况下有效地处理多个请求。
3. **接收到请求，从线程池中获取线程：**
   - 当有HTTP请求到达时，Web服务器从线程池中获取一个线程来处理这个请求。
4. **获取Session：**
   - 在处理请求之前，Web服务器通常会创建一个`HttpServletRequest`对象，其中包含了当前请求的信息，包括Session。
   - 如果请求携带了Session ID（通常存储在Cookie中），Web服务器会尝试从Session管理器中获取对应的Session。
5. **请求传递给Spring框架：**
   - 一旦获取了请求信息和Session，Web服务器会将请求传递给Spring框架。
6. **经过拦截器：**
   - 请求可能会经过一系列的拦截器。拦截器允许你在请求处理前或处理后执行一些逻辑，例如权限验证、日志记录等。
   - 在拦截器中，你可以访问请求和响应对象，以及Session信息，并在需要的情况下将信息存储到`ThreadLocal`中。
7. **传递到Controller：**
   - 一旦通过拦截器，请求将被传递到对应的Controller。Controller是Spring框架中负责处理业务逻辑的组件。
8. **业务逻辑执行：**
   - 在Controller中，你可以执行具体的业务逻辑，可能包括从数据库中读取数据、处理用户输入等。
9. **响应返回：**
   - Controller执行完业务逻辑后，返回一个响应。这个响应可能是HTML页面、JSON数据等。
10. **线程释放回线程池：**

- 一旦处理完成，Web服务器会将使用的线程释放回线程池，以供处理下一个请求。

