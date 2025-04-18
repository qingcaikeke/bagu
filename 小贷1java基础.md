// stack入栈null,isEmpty会返回false

**区分空值null和空字符串""?**null不是一个对象，不能调用任何方法字段，指向一块非法内存（代码段下面）

##### unicode-utf-base64

**unicode：**给每个字符一个唯一编码，用二进制表示字符，是一种定长编码。中文，日文，标点，希腊字母等等，现在大概有十几万

​	最常用的前128个字符叫ASCLL（U+0000至U+007F）

**utf-8，utf-16**

表示怎么给unicode编码，节省空间

如 utf-8 用1-3个字节表示一个unicode，ascall只需要一个字节，其他很多字符标点等也只要一个字节

utf-16 用2-4个字节表示一个unicode

redis想存一个string = “你好”，1.首先unicode得到 '你'的码点是U+4F60和'好'的码点是U+597D，

2.然后将其对应utf-8（节省空间） 0x4f60对应UTF-8序列0xE4 0xBD 0xA0; 0x597d对应0xE5 0xA5 0xBD。得到六个字节，

3.存入字符数组: ['你','好'] = [0xE4, 0xBD, 0xA0, 0xE5, 0xA5, 0xBD, **'\0'**]

变长编码怎么编解码：

编码：如用三个字节，他的表示一定是 1110xxx 10xxx 10xxxx，从后向前填充x，多出的x用0补上

解码：如果一个字节的第一位是 0 ，则说明这个字节对应一个字符；如果一个字节的第一位 1，那么连续有多少个 1，就表示该字符占用多少个字节

**base64(文本表示二进制)**

基于 64 个可打印字符来表示二进制数据，是为了让二进制数据能够在只支持文本传输的环境中安全地传输，比如在电子邮件、HTTP 协议等场景中。适用于任何二进制数据（图像，音频等）

用4个ascll字符表示3个字节的二进制，所以编码后的数据长度会比原始二进制数据大约增加 33%

3\*8 = 4*6，每个字节的最前面补两个0，不够3字节用=对齐

6个bit最大64个数，也就是base64

有时我们会在Base64字符末尾会看到`=`，是用来补位的，不够三个字节一组

**哈希算法**

sha256/md5，大概思路好像就是位运算之类的

##### 浮点型的存储方式，以及浮点型怎么表示数的

小数转二进制：乘2取整。计算机用浮点数表示，浮点表示小数点的位置可以变化，

所以首先把小数转换成科学计数法1.a*10^b，然后float用1位标识符号位，8位标识指数位，23位标识小数位

double则是，1位符号，11位指数，52位小数，`log10(2^53)` 约等于 `15.95` 和 `log10(2^24)` 约等于 `7.22` 位（为什么是7和15位）

具体储存的话，首先转二进制，整数部分小数部分都转，然后移动小数点，变成科学计数法，得到指数位，然后加上**偏移量**得到需要储存得整数（偏移量是因为有符号整数计算麻烦）

##### **tomcat的运行原理**

用户发起请求，访问tomcat注册（监听）的端口，监听线程创建socket连接，已连接socket接收到数据去线程池拿一个线程执行用户请求，这个线程会带着封装好的request，去调用我们开发的webAPP（controller service dao）再访问数据库

文件描述符：是应用程序与内核交互的接口,它为应用程序提供了访问文件资源的抽象。 	

动态代理：运行期间动态创建某个接口的实例

垃圾回收器线程是一个守护线程

### 面向对象

##### **什么是面向对象**：

把整个程序运行过程抽取出一个个对象，对象有自己的属性和行为，然后去让对象解决问题，耦合度较低

面向过程就是抽取出一个个函数，按流程去做事，耦合度高，可维护性可复用性可扩展性较差，不适合大型项目。优点是自顶向下编程，便于理解，适合解决，简单，逻辑性强的问题

**封装：**把数据和操作数据的方法封装在一起，形成一个不可分割的独立实体，隐藏内部实现细节，只保留一些对外接口使之与外界联系，以减少耦合，独立开发测试，提高可重用性

公有的get、set方法，私有的对象属性，不允许外部对象直接访问对象的内部信息。但是可以提供一些可以被外界访问的方法来操作属性。使用访问修饰符（public、private、protected），隐藏内部实现细节

数据隐藏和保护：防止非法访问，可以做数据校验；

解耦：外部不用关心细节如何是实现的，只要把需要的参数传过去就行，扩展也只需要修改这个类里面的代码

**继承：**使用已存在的类作为基础建立新类的技术。新类可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。通过使用继承，可以快速地创建新的类，可以提高代码的重用，程序的可维护性，节省大量创建新类的时间 ，提高我们的开发效率。

**什么是多态**：(是什么，解决了什么问题，怎么实现)

1.引用变量的的具体类型和实际发出的方法只有在运行时才能确定，即使用统一的接口操作不同的对象，一个声明类型，一个实际类型（父类的引用指向子类的实例。）

2.代码复用（通用的接口处理不同的对象），可扩展（添加新类型，只需要保证实现同样的接口），接口统一（客户端不需要关注具体类型），只需要知道接口的定义

3.继承实现（子类对象被当作父类对象调用）和接口实现（声明类型是接口，实际调用是实现类）

##### **抽象类和接口的区别？**

相同点：都不能直接实例化，需要继承和实现，单继承和多实现，多实现间接实现多继承

不同点：接口：抽象方法，不能有实现，可以有deafult，成员变量只能是public static final（静态常量）（静态的，属于类，多实例共享）（不可被修改，必须初始化）用于**不同类的解耦**和多态性实现

抽象类：可以有方法定义和方法实现，可以有实例变量（对象的属性）和静态变量（类的属性），一般是**类的一个模板**（基类），供其他类继承和扩展使用，不能用final修饰（必须要继承），提供一个类的模板

**例：**

负载均衡接口定义了一个select方法

abstractBalance实现了这个接口，又定义了一个doselect抽象方法，select会调用doselect，

​	抽象类定义一个模板，这个类想完成selcet方法，要靠自定义的doselect犯法

randomBalance继承了抽象类，隐式继承了select方法，重写了doselect方法

补：接口继承接口相当于扩展了接口的方法，接口层次代表抽象程度

补：实例字段在每个实例中都有自己的一个独立“空间”，但是静态字段只有一个共享空间，所有实例共享该字段。静态字段并不属于实例

##### UML类图

泛化关系(extends，动物 《- 猫)，实现关系(implements)，聚合关系（整体不在，部分还在，电脑=鼠标+键盘+屏幕+主机），组合关系（强依赖，整体不在，部分也不在，公司、部门1、部门2），关联关系（多对多，学生-学校），依赖关系（一个对象在运行时会用到另一个对象）

类和类之间的关系有三种：is-a（继承关系）、has-a（）和use-a（依赖关系）关系

里氏替换就是继承

##### 关键字

**final**：类不能被继承，字段不能修改，方法不能被重写，

​	修饰字段：类似常量的感觉，声明或定义时就得赋初值

**static**：修饰变量，表示变量属于类，无论创建多少个对象，公用一个变量；

​	修饰方法：静态方法表示方法属于类，一般通过类名.方法访问，只能访问静态变量；

​	修饰代码块：静态代码块一般用于给静态变量初始化，类加载时直接执行。用的不多，一般声明时直接创建了

所以单例双重锁中的两个static是必须的，因为构造方法是私有的，要用类.方法创建实例，所以方法得是static得，静态方法只能访问静态变量，所以instance得用static修饰

**常量(final)和静态变量(static)**

常量：声明时就要赋值，不能修改；基本数据类型常量会被嵌入到字节码中，引用类型常量存储在堆中，其引用指向堆中的对象。

静态变量：可以声明赋值或静态代码块赋值，都是在类加载时进行赋值；变量本身储存在堆里，但类的元数据信息，包含静态变量的定义，储存在元空间。当程序访问静态变量时，会依据元空间中的类元数据信息找到堆中的静态变量。

补：元数据信息：变量名，类型，访问修饰符

补：方法的定义信息，也就是字节码、方法名、参数类型、返回类型、访问修饰符等内容，会存于方法区。这些信息是类加载时从字节码文件加载进去的，是类的一部分元数据，在类的生命周期内都会存在。

**super**：

`super`关键字用于在子类中访问父类的方法和变量

如：用`super`调用父类的方法，即使被覆写了也能调

`Comparator<? super T>` 中的 `Comparator` 是 Java 中的一个接口，用于定义对象之间的比较规则。而 `<? super T>` 是一个泛型通配符，它表示该 `Comparator` 可以接受 `T` 类型或者 `T` 的任意父类类型的对象进行比较。

为什么可以比较父类就一定可以比较子类，因为面向对象里子类可以完全视为父类，一定有父类的属性和方法

**作用域：**字段不加修饰符表示包作用域

### java基础

##### **jdk，jre，jvm**

jdk（development kit）开发工具包，给开发人员使用的，包含开发程序所需要的一切工具，编译调试等

jre(runtime environment) 运行已编译程序需要的环境，包含虚拟机，基础类库等(io操作，网络通信，数据结构)

jvm：虚拟机，把编译出来的字节码文件，**解释**成具体系统平台上的机器指令，让其在各个平台运行（字节码->机器码）

**java和c++**

自动内存管理（分配，回收），不提供指针直接访问内存(用的是引用，能找到那个位置)，类单继承

补：基本类型和包装类型，包装类型必须用equals

**java怎么执行的**

源代码编译成class字节码，通过jvm类加载器加载，jvm解释器将字节码解释成对应操作系统的机器码

##### 基本数据类型和拆装箱

byte(1) short int(4) long(8) - float(4，小数点后七位) double(8，小数点后十五位) - Boolean(1) - **char(2)**

为什么int的范围？max_val = 2147483647，min = -2147483648，是二十一亿多

总共32位，第一位符号位，

2^31-1 ~ -2^31；+0已经有表示了，-0实际上表示minVal；

负数是源码按位取反+1，所以想表示min，加1之前应该是31个1，取反之前就31个0，加符号位1，也就是-0

一般基本数据类型存在栈里，包装存在堆里；但会有逃逸分析，如果判断对象范围不超过方法或线程，可能存堆里，减少堆内存垃圾回收开销

如果基本数据类型是方法的局部变量，就存栈中；如果是类的实例变量，就存堆中；如果是静态变量，就存方法区

**int到Integer的转换称之为装箱**

Integer.valueOf(int) ，装箱在-128~127之间会用缓存，得到同一个对象，之外会用new，new一定是创建了一个新的对象

new Integer(int)，new会调用构造方法，把int赋值给一个final的属性，得到一个新的对象

对象.intValue()，实现插箱

**Integer a=null; System,out.println(a==127) 会报错吗？**

会报npe，因为integer会自动拆箱，调用intValue方法

**integer是不可变类型**，因为里面的int是final的，所以把integer作为参数递归传递，下一层不会改变当前层的值，会导致错误，可以使用只有一个元素的Intege[]

**BigDecimal**

可以实现对浮点数的运算，不会造成精度丢失。通常情况下，大部分需要浮点数精确运算结果的业务场景（比如涉及到钱的场景）都是通过 `BigDecimal` 来做的。

底层：去掉小数点bigInteger表示，再记录小数点位置

##### equals,==,hashCode

**对象相等和引用相等**

对象在堆里，对象引用在栈里

对象相等比较内存中的内容是否一样，引用相等比较内存地址是否一样

==比较引用相等（基本数据类型值相等，引用对象地址相等），

equals比较对象相等(内容相等)，没重写比较地址，重写比较内容（一般为比较类的属性是否一致），`String` 中的 `equals` 方法是被重写过的。equals只能比较对象

**为什么要有hashcode**

用于快速判断元素在哈希集中应存放的位置，如果发现hashcode相同，再用equals判断对象是否真的相同，减少equals次数，提高执行效率

**为什么重写 equals() 时必须重写 hashCode() 方法？**

equals代表两个对象相等，内容相同的对象在哈希集合中的位置必须相同，所以hashcode必须相同。

存map时，先算hash相等再用equals判断是存在链后面还是别的位置，**例：**重写hashCode方法：return 31*name.hashCode()+age

##### 深拷贝、浅拷贝、引用拷贝

浅拷贝创建新对象，但如果对象内部属性是引用类型，直接复制引用；深拷贝把内部对象也复制一份

##### object类的常见方法

getClass，hashCode，equals，clone，toString，wait，notify，finalize(垃圾回收)

##### String相关

**String、StringBuffer、StringBuilder 的区别？**

String是不可变类型，频繁改变不应该用String，会消耗资源。

字符串对象通过“+”的字符串拼接方式，实际上是通过 `StringBuilder` 调用 `append()` 方法实现的，拼接完成之后调用 `toString()` 得到一个新的 `String` 对象 

Stringbuffer 线程安全，builder不安全（缓冲区这玩意多线程总用），buffer底层使用了synchornize关键字

**为什么String是不可变类型**

底层用char[] 储存，被标记为private final，也没有提供暴露给外部的修改方法，同时String类也被标记为final，导致其不能被继承，避免子类破坏不可变性（子类重写父类方法，创建声明对象父类，实际对象子类，调用方法调用的是子类的）

补：后改用byte[]储存字符串，char2字节，byte1字节

**为什么要这么设计？**

多线程并发安全问题；性能优化，需要创建新String时，可以复用

**String s1 = new String("abc");这句话创建了几个字符串对象？**

如果常量池没有"abc"，创建两个对象，"abc"在字符串常量池，s1在堆里

String str1 = "str"; String str2 = "ing";//常量池

String str3 = "str" + "ing"; // 常量池 

String str4 = str1 + str2; // 堆

字符串常量池：减少内存消耗，避免字符串重复创建

**String常用方法**

```java
String.valueOf(int i);
Integer.parseInt(String s);
String s = " ffff";
s.split(String regex,int limit)
s.trim(); String.split;	String.join();
String.equals;	String.contains;    String.indexof;	String.lastindexof;        
```

##### **异常**

throwable(父类)分exception（子类）和error（子类），异常是程序本身可以处理的，通过捕获并解决；错误是程序本身没法处理

Exception 分 Checked(受检查异常，必须抛，FileNotFound）和Uncheckd(空指针，数组越界，参数错误)

RuntimeException属于Unchecked，即编译器不要求程序员手动处理可能出现的异常，如果运行时发生了该问题，程序会终止并抛出响应的信息，如NullPointerException，ArrayIndexOutOfBoundsException

Error包括OOM，StackOverflow等

try-with-resources：适用于必须关闭的资源，防止泄漏，正常写在finally中，调用close

throw写方法内部的

throws写方法声明，告诉会抛啥，让调用方处理

##### **反射**

程序运行期间拿到一个对象的信息，对一个对象一无所知的情况下如何调用其方法

在程序运行时动态的获取类信息，包括字段，方法，构造函数等，并创建对象实例，可以用于运行时根据需要加载外部的类和资源，实现插件化结构，也可以用于检查注解，泛型等信息（创建对象，调用方法，访问字段）

框架中大量使用动态代理，动态代理基于反射，注解的实现也用到了反射

**优点：**灵活，扩展性强，不改变现有代码的前提下，通过反射加载使用新类和方法。使得框架可以在运行时动态地创建和管理对象，实现依赖注入和 AOP 等功能。

**缺点：**性能开销大，要动态解析，在运行时进行额外的查找和调用，因此比直接调用方法和属性开销大；能绕过访问修饰符，因此存在安全问题

补：`.java`文件**编译**生成`.class`文件，首次访问某个类，触发类**加载**，JVM读取.class文件，解析生成Class对象，Class对象可以用于实例化一个对象

3个生命周期分别对应于：Java源文件(.java文件) ---> .class文件 ---> 内存中的字节码

Class对象如何获得：类名.class；对象.getClass；Class.forName(全类名)

**反射代码示例：**

```java
    public static void main(String[] args) throws Exception {
        // 获取 Class 对象
        Class<?> clazz = Class.forName("MyClass");
        // 获取构造函数
        Constructor<?> constructor = clazz.getConstructor(String.class);
        // 创建对象
        Object obj = constructor.newInstance("Hello, Reflection!");
        // 获取方法
        Method method = clazz.getMethod("printMessage");
        // 调用方法
        method.invoke(obj);
    }
```

##### 注解

是一种特殊的注释，需要解析才能生效，解析方法有两种，**编译器直接扫描**@Override

或**运行期间通过反射处理**@Value@Component，扫描指定包路径，找到带有特定注解的类@Componet，实例化成bean

**自定义注解**：使用@interface定义，定义作用域，生效时间，定义字段和默认值，使用时通过反射解析注解信息

**SPI和API**：(Service Provider Interface) 专门提供给服务提供者或者扩展框架功能的开发者去使用的一个接口。

实现方提供接口和具体实现，调用方直接调用，这就是API

调用方定义接口规则，不同厂商根据该接口完成实现，是SPI

当接口存在于调用方这边时，就是 SPI ，由接口调用方确定接口规则，然后由不同的厂商去根据这个规则对这个接口进行实现，从而提供服务。**缺点：**要遍历加载所有的实现类，不能做到按需加载，这样效率还是相对较低的。

有点像策略模式，不修改客户端代码的情况下，动态地替换服务实现，如日志，在切换日志具体实现的时候我们是不需要更改项目代码的，只需要在 Maven 依赖里面修改一些 pom 依赖就好了。加载进来哪个包就用哪个

**语法糖**：编译级别，增强for，枚举，try with resources，lambda

**手写一个泛型类**

泛型是指允许在、接口、方法定义时使用一个类型参数，实现代码通用性，可读性，避免了重复编码和强制类型转换，同时在编译时提供类型检查，因此更安全

```java
public class MyGeneric<T> {   
     private T data;    
     public MyGeneric(T data) {
        this.data = data;    
     }     
	public T getData() { return data; }     
	public void setData(T data) {this.data = data;    }
}
Generic<Integer> genericInteger = new Generic<Integer>(123456);
```

### jdk源码

##### **集合概述**

集合是因为数组具有局限性，集合具有 大小可变、支持泛型、具有内建算法等优势

集合由collection和map两大接口派生

​	map接口下有：hashMap，LinkedHashMap(双线链表连接插入顺序)，TreeMap(key顺序)

​	collection接口下有：list，set，queue

​		collection接口提供了一些集合的共性方法的抽象，add，remove，isEmpty，size等

list：有序(插入顺序)，linkedList双向链表，ArrayList

set：唯一，hashSet，TreeSet（有序），linkedHashSet

queue：先入先出，linkedList实现

#### List

##### **List的实现类**

ArrayList、LinkedList、Vector、stack

1. ArrayList：基于动态数组实现，优势在于支持随机访问和快速插入/删除元素，适用于频繁读取和遍历的场景。（不安全）
2. LinkedList：基于双向链表实现，内存不连续，优势在于支持快速插入/删除元素，适用于频繁插入/删除元素的场景。（不安全）
3. Vector：和ArrayList类似，但由于其**线程安全性**，适用于多线程环境。
4. Stack：基于Vector实现，是一个后进先出（LIFO）的数据结构，适用于需要按照后进先出顺序处理元素的场景。

一般用ArrayList更多，因为支持随机访问，同时link插入删除也没快到哪去，因为需要遍历找到位置

**ArrayList和LinkedList区别？**

底层实现不同(数组、链表)导致增删查效率不同，占用空间不同，进而导致使用场景不同

**List和Set的区别？**有序无序，可否存放重复元素

list按插入顺序保存，需要保持元素顺序，会储存重复数据，通过索引值直接访问、插入或删除元素，适用于需要频繁进行这些操作的场景用list

不关心顺序，需要去重，快速判断元素是否存在，需要交并补等集合运算用set

##### ArrayList

**ArrayList和Array**

是动态数组和静态数组，能否扩容，泛型保证类型安全，只能存对象

**arrayList扩容机制**：

无参构造时创建的是空数组，添加第一个元素容量扩容为10

有参构造，参数容量给多少就创建多大的数组

add时如果当前容量不足以容纳新元素，就扩容

扩容时，扩为就容量1.5倍，然后旧元素复制到新元素中

##### 线程安全的List

Collections.synchronizedList：方法加锁，适合频繁读写 

CopyOnWriteArrayList：ArrayList的一种线程安全变体，其中所有可变操作（add、set等）都是通过创建底层数组的新副本来实现的。适合读多于写，适合不希望加锁遍历，但同时需要排除并发线程干扰

在对列表遍历时，生成一个快照，基于快照遍历

进行修改（如添加、删除元素）时，不直接在原数组上进行操作，而是先复制一份原数组，然后在新复制的数组上进行修改，最后将原数组的引用指向新的数组。

#### Map:

**hashtable和hashmap区别是什么？**

hashtable线程安全，所有方法都加了synchornize，同一时刻只能由一个线程操控该对象，所以效率比较低，已经几乎不用了，用ConcurrentHashMap，它是通过原子操作和局部加锁实现

**HashSet/Map 如何检查重复?**

底层是hashMap，先算code，没有一样的认为唯一，有的话再用equals方法

##### **CurrentHashMap**

jdk7：分段锁，整个哈希表划分成多个小的哈希表

jdk8：volatile修饰的节点数组+cas（槽位为空时）+synchornize（哈希冲突时，链转树时）

概念：tab表示储存节点的数组，Node<K,V>表示节点，位于数组的某个桶表示位于数组的某个索引位置

初始化：判断是否有其他线程正在初始化，cas尝试拿到执行权

get：不加任何锁，因为tab有volatile修饰，保证每次获取值都是最新的

put：计算hash值确定放到tab的哪个桶，**如果桶为空，直接使用cas插入**(比较当前桶位置的内存值是否为null)

```java
transient volatile Node<K,V>[] table;
if ((null == tabAt(table, i))) {
    if (casTabAt(table, i, null, newNode))
        return null;
}
```

如果桶不为空，synchornize锁头节点，遍历链表，再覆盖或插入

##### HashMap底层

HashMap底层是基于**数组和链表**（拉链法）实现的。HashMap将key通过hash算法映射到数组中，然后在对应的链表中查找value。当多个key的hash值相同时，会在同一个数组位置上使用链表来存储这些key-value。但是，当链表长度太长时，会影响HashMap的性能，因此在JDK1.8中，当链表长度超过阈值时，会将链表转换为红黑树，以提高查找效率。

**HashMap插入一个元素的过程**

根据key的哈希算应该在table的哪个位置，看该位置是否为null，为null新建节点并插入；否则看table[i]的首个位置的key和要插入的是否一样（hashCode和equals都得一样），一样覆盖val，不一样一直往后找（数组或链表），都不一样就插入节点，然后根据容量，负载因子和当前元素数量是否需要扩容

**线程安全吗？**因为多个线程同时访问HashMap时可能会导致数据不一致的问题。

**1.8：**两个线程要插，都判断槽位为null，第一个判断完，没插，切换到第二个；源码是new节点然后放到槽里，这样第二个线程就会把第一个线程覆盖

**1.7：**扩容会出现死循环，因为用了头插法，也可能会丢数据

1.put涉及到计算哈希值再计算位置，完成插入等多个操作，不是原子操作

2.扩容涉及到重算哈希值，重算位置，复制等 步骤，可能多线程同时扩容

3.也可能一个线程遍历，另一个线程插入删除

3.链表转红黑树是非原子的

4.读写冲突，一个线程put一个线程get

**HashMap源码**

初始容量默认16，可以传入初始容量参数，然后设置为大于他的2的n次方

容量（n-1）&（hash(key)）计算储存位置，哈希桶中用数组或红黑树（链转树）

**扰动函数**：减少哈希碰撞，原哈希码^原哈希码右移16位

初始化容量：比所需存放元素大的最小的2的n次方

**负载因子**：默认0.75，超过多少扩容，所存元素/容量

**扩容方法**：二倍扩容，高位多了一位1，原哈希值与扩容新增出来的长度16，进行**&运算**，等于0下标不变，等于1在原来的位置+16（put的时候看是否超过阈值，然后扩容）

容量永远是n = 2的k次方，n-1二进制全是1，所以扩容后一部分元素不用动，一部分在原来的位置加上原容量n

**链转树：**桶容量大于64，链长度大于8，两个条件都满足

```java
class MyHashMap<K, V> {
    // 默认初始容量
    private static final int DEFAULT_CAPACITY = 16;
    // 存储数据的数组
    private Node<K, V>[] table;
    // 当前存储的键值对数量
    private int size;
    static class Node<K, V> {
        final int hash;
        final K key;
        V value;
        Node<K, V> next;
    }
    public void put(K key, V value) {
        int hash = hash(key);
        int index = (table.length - 1) & hash;
        Node<K, V> newNode = new Node<>(hash, key, value, null);
        if(table[index] == null){
        	table[index] = newNode;
        }else{
        	Node<K, V> current = table[index];
        	while(true){
        		if (Objects.equals(current.key, key)) {
                    // 如果键已存在，更新值
                    current.value = value;
                    return;
                }
                if (current.next == null) {
                    break;
                }
                current = current.next;                
        	}
            // 键不存在，插入到链表尾部
            current.next = newNode;        	
        }
        size++;
    }
}
```

##### LinkedHashMap

在hashmap的基础上，增加了一个双向链表，使之可以按插入顺序找到下一个元素，可以用于简单实现LRU

##### **TreeMap**

哈希map遍历Key时，其顺序是不可预测的。而treeMap底层是**红黑树**，按key进行排序，增删查的复杂度都是logN，同时他不需要使用hashcode，但是需要重写一个Comparator，负责比较传入的两个元素`a`和`b`，如果`a<b`，则返回负数，通常是`-1`
或是在key中重写ComparateTo方法

**补：**hashmap插入删除和查询的复杂度O(1)，treemap复杂度O(logn)， 元素稀疏且无序时用hashmap，元素稠密且有序时用treemap

**红黑树是什么？**

二叉查找树可能退化成链表，严格的平衡树插入删除的过程可能涉及到复杂的树的变化，影响查找效率

红黑树减少翻转，同时保证了根到叶的最长路径不会超过最短路径的两倍

红黑树是自平衡的二叉查找树，增删改查效率稳定olgn，满足1.根节点**黑色**，叶节点黑色2.红节点的两个子节点是黑色3.任意节点到叶节点的路径包含相同数目的黑色节点

补：红黑树主要用于在内存中维护有序的动态集合（epoll），而B树和B+树主要用于实现外存储的索引结构，例如数据库和文件系统。

##### Queue

**阻塞队列，优先级队列，延迟队列**

阻塞队列：常用于生产者消费者结构；队空，获取元素的线程等待；队满，储存元素的线程等待

​	内部靠reentrantLoct和condition实现

优先级队列：堆实现；

延迟队列（DelayQueue）：优先级队列实现顺序 + ReentrantLock保证线程安全(并发存取)

用途：希望任务可以按找我们预期的时间执行，如下单15分钟后未支付取消，如提交了三个任务，希望分别在1s，2s，3s后执行

核心方法：getDelay：查看当前任务还有多久到期；compareTo：比较任务执行时间大小，用于排序

入队(存)：加锁，存，看是不是队中第一个元素，是的话通知其他线程来抢

出队(取)：多线程抢锁，抢到锁的才有资格取；看队列是否位空，为空等待；不为空看到没到期，到期直接返回，队首有没有其他的线程等着执行

todo？

队里存放的是task，有一个runnable属性表示执行对象，所以存的是线程？

##### lambda表达式

[函数式编程 - Java教程 - 廖雪峰的官方网站](https://liaoxuefeng.com/books/java/functional/index.html)

函数式编程：把函数作为基本运算单元，函数可以作为变量，可以接收函数，还可以返回函数。

函数式接口（单方法接口）：只有一个方法的接口（只有一个抽象方法）

接收函数式接口作为参数时，可以把实例化的匿名类改写为Lambda表达式，能大大简化代码(不用new对象再重写方法)。即lambda可以为函数式接口中的唯一抽象方法提供实现。

(参数列表) -> {表达式或代码块}

只有一行return就直接表达式，比较复杂就 ->{代码块}

参数列表描述了输入参数，可以省略类型（编译器自动推断）。箭头符号将参数列表与表达式或代码块分隔开来，返回值类型也是编译器推断

```java
Arrays.sort(array, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return s1.compareTo(s2);
    }
});
Arrays.sort(array,(s1,s2)->s.compareTo(s2));
```

**api示例**

Function<T,R>：T代表参数类型，R代表返回值类型

```java
public static <T> Comparator<T> comparingInt(ToIntFunction<? super T> keyExtractor) {
```

返回一个比较t类型的比较器，需要传入一个函数，函数接收t或者t的父类，返回比较器需要的键（比较器根据键确定排序位置）

```java
Arrays.sort(intervals, Comparator.comparingInt(interval -> interval[0]));
```

```java
Arrays.sort(intervals,(a,b)->(a[0]-b[0]));
```

```java
runnable是个函数式接口，创建匿名对象 -> lambda创建匿名函数 -> 方法引用
Thread thread1 = new Thread(new Runnable() {
    @Override
    public void run() {
        printAB.printA();
    }
});
Thread thead2 = new Thread(() -> printAB.printA());
Thread thead3 = new Thread(printAB::printA);
```

**方法引用**

双冒号"::"是方法引用，会根据上下文推断参数类型，因此特别适用于直接引用已有方法的情况。

方法引用的一般形式是：ClassName::methodName

方法参数需要传入一个函数或函数式接口，一般使用lambda，虽然比创建匿名类简单，但这样还需要自己写逻辑，有的时候可以直接使用方法引用，只要方法参数需要的函数和已有的函数方法签名相同就行（参数类型和返回值类型）

如果引用的是本类的实例方法，要注意实例方法的函数参数其实有一个隐含的this参数

如果引用的是构造方法，要注意实际上返回了一个this实例，即返回值类型为this

##### 回调函数

一种函数式编程，把回调函数作为参数传递给另一个函数，并在被调用函数执行完毕后被调用。常用于事件处理，异步编程。

##### Stream

[使用Stream - Java教程 - 廖雪峰的官方网站](https://liaoxuefeng.com/books/java/functional/stream/index.html)

stream 储存顺序输出的对象实例，通过惰性计算，把一个stream转成另一个stream，而不是修改原`Stream`本身。

stream的对象实例不是预先储存在内存中的，而是实时计算出来的

stream是惰性计算的，一个`Stream`转换为另一个`Stream`时，实际上只存储了转换规则，并没有任何计算发生，所以必须要通过聚合操作才能触发计算。对`s3`进行`reduce()`聚合计算，会不断请求`s`3输出它的每一个元素。因为`s3`的上游是`s2`，它又会向`s2`请求元素，`s2`向`s1`请求元素，最终，`s1`从 Array / Collection 实例中请求到真正的元素，并经过一系列转换，最终被`reduce()`聚合出结果。

```java
Stream创建
基于数组 Arrays.stream(arr) 
基于Collection (List,set,Queue) list.stream()
```

```java
常用运算：map，filter(不满足条件的就被“滤掉”了)
排序.sorted(String::compareToIgnoreCase) 去重.distinct() 截取.skip(2).limit(3)
把一种操作运算，映射到一个序列的每一个元素上
List.of("  Apple ", " pear ", " ORANGE", " BaNaNa ")
        .stream()
        .map(String::trim) // 去空格
        .map(String::toLowerCase) // 变小写
        .forEach(System.out::println); // 打印
```

```java
聚合操作：reduce()——collect()——count()——max(Compator)——min(Compator)
输出List：collect(Collectors.toList());
输出Array：stream().toArray(String[]::new);
输出为Map：Map<String, String> map = stream
                .collect(Collectors.toMap(
                        // 把元素s映射为key:
                        s -> s.substring(0, s.indexOf(':')),
                        // 把元素s映射为value:
                        s -> s.substring(s.indexOf(':') + 1)));
分组输出：使用`Collectors.groupingBy()`，它需要提供两个函数：一个是分组的key，这里使用`s -> s.substring(0, 1)`，表示只要首字母相同的`String`分到一组，第二个是分组的value，这里直接使用`Collectors.toList()`，表示输出为`List`
Map<String, List<String>> groups = list.stream()
                .collect(Collectors.groupingBy(s -> s.substring(0, 1), Collectors.toList()));                      
```

##### 数组转集合List

Integer[] myArray = {1, 2, 3};

`Arrays.asList(myArray)`或`Arrays.asList(1，2，3)`，必须传对象数组，不可直接修改list，对原数组的修改可以同步到list

如需修改list，可使用

```java
List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
List myList = Arrays.stream(myArray).collect(Collectors.toList());
List<Integer> list = List.of(array);
```

List转int[]

```java
int[] nums = list.stream().mapToInt(Integer::intValue).toArray();
```

反转list

```java
Collections.reverse(list);
```

遍历**map**

```java
for (Map.Entry<Integer, String> entry : map.entrySet()) {}
for (Integer i : leftMap.keySet()) {}
```

##### iterator

如果要遍历集合的同时修改集合的结构，就不能用for，如删除集合中所有偶数，删除元素后，其他元素的索引会发生变化

**底层原理**

首先集合类中实现的 `iterator()` 方法获取迭代器对象，迭代器对象中维护了两个游标

一个指向当前元素的游标 `cursor`，以及一个 `lastRet` 表示上一个元素的索引位置

核心方法：

- `hasNext()`：判断是否还有下一个元素，通常通过比较 `cursor` 和集合的大小来确定。

- `next()`：返回当前元素，并将 `cursor` 向后移动一位，同时更新 `lastRet` 的值。

- `remove()`：删除上一个通过 `next()` 返回的元素，一般会检查 `lastRet` 是否为有效的索引，然后调用集合的 `remove` 方法删除对应元素，并重置相关变量。

  比如123，删除2，先找到2，cursor指向3，lastRet指向2，调用remove，这个时候cursor仍然指向3，lastRet指向1

```java
Iterator<Integer> iterator = list.iterator();
   while (iterator.hasNext()) {
       Integer element = iterator.next();
       if (element % 2 == 0) {
           iterator.remove();
       }
   }
```

