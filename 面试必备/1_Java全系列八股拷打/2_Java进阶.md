Java进阶（涉及底层源码细节）
这里能挖掘的很多，这里你最好真的懂，不然禁不住拷打的

## 1. Object类常用方法

这个问题很经典，如果水平不高的话，这个问题不要答太好，因为某个方面
比如多线程这一块你一旦提了势必会考打你


Object 类常见的方法有 `toString( )`，`equals( ) `和 ` hashCode( )`，`wait( )` 和 `notify( ) `等等。

具体来说：
`toString( ) `返回的是**对象的字符串表示**，默认为 「class 名 + @ + hashCode 的十六进制表示」，在实践中，我们一般会在子类将它重写为打印各个字段的值，在调试和打日志中用的多。

`equals()` 方法用于比较两个对象是否相等，返回布尔值。在 `Object` 类中，默认的 `equals()` 实现与 `==` 等价，也就是说，它只是判断两个引用是否指向同一个对象。`hashCode()` 方法默认返回一个整数，这个整数通常由对象的内存地址计算得到。实际应用场景中，特别在使用基于哈希表的数据结构，比如 **`HashSet`**、**`HashMap`**。我们一般会选择重写 `hashCode()`。因为当我们向哈希表中添加一个对象时，基于哈希表的数据结构会先调用 `hashCode()` 确定对象中的桶，再通过 `equals()` 方法判断是否是同一对象。实际开发中如果我们想通过对象的内容比如其各成员变量来判断相等性，我们需要同时重写 `equals()` 和 `hashCode()` 方法。这是因为不同对象即便其内容（成员变量）相同，其内存地址可能不同导致 `equals()` 方法返回非预期结果；其次两个对象即便内容不同，它们的默认的 `hashCode()` 也可能相同。

`wait( )` 和 `notify( ) `，这是线程间通信的方法，必须在同步块（synchronized）中调用，实现线程之间的协调，例如**生产者–消费者模式**。具体实践当中在一个线程里调用 wait() 进入等待，在另一个线程里调用 notify() 或 notifyAll() 唤醒，确保在共享锁对象上调用。

### 1.1 ==和equals()有什么区别？

== 是一个操作符 ，`equals() `是超类 Object 中的方法。对于没有重写 `equals` 方法的子类，`equals `和 = = 是一样的。而 == 在比较时，根据所比较的类的类型不同，功能也有所不同：对于基础数据类型，如 int 类型等，比较的是**具体的值**；而对于**引用数据类型**，比较的是引用的地址是否相同。对于重写的 equals 方法，比的内容取决于这个方法的实现。
基础数据类型指的是Java 内置的最基本的数据类型，不是对象
除了 8 种基本类型以外，其他的类型（类、接口、数组、枚举等）都是引用类型。
引用类型的变量本身存放的不是对象内容，而是对象在内存中的地址（引用）。
基础数据类型（primitive type）在内存中，变量当然也有地址（栈上某个 slot）。但这个地址只是存放值的地方，不参与比较。比较时，直接取出该地址里存放的二进制值（例如 10、20）进行数值比较。
引用数据类型（reference type）
在内存中，栈上的变量保存的不是对象内容，而是一个引用（地址/指针），指向堆中的对象。比较时，== 比较的是这两个引用（即堆中对象的地址值）是否相同。
如果需要比较对象内容，就必须依赖 equals()。

### 1.2 讲一下`equals()`与`hashcode()`，什么时候重写，为什么重写，怎么重写？


equals() 和 hashcode() 都是Object 中的方法。hashcode()会返回哈希值，所谓哈希值就根据一定的规则将与对象相关的信息，比如对象的内存地址，映射成一个数值，这个数值称作为哈希值。

当我们想要自定义类的比较规则时，需要重写 equals( )，但是为了保证类在 HashSet 和 HashMap 等集合中的正确存储，也要同时重写 hashCode( )。

以 HashMap 为例， HashMap底层在添加相同的元素时，会先调用两个对象的 hashCode( ) 是否相同，如果相同还会再用 equals( ) 比较两个对象是否相同

假设有一个 Person 类，有 name 和 age 两个字段，我们现在重写 equals( ) 规定只有两个 Person 的 name 和 age 都相同时，才认为两个 Person 相等。

现在 new 出两个 name 和 age 都相同的 Person，分别添加到 HashMap 中。我们期望最后 HashMap 中只有一个 Person，但是如果没有按照一定规则重写hashCode( )，HashMap里最后会有两个。因为添加第二个 Person 时，先比较的是两个 Person 的 hashCode( )。在没有重写 hashCode( )的情况 ，那么分别 new 出来的 Person 的**哈希值肯定是不同的**，到这里 HashMap 就会将两个 Person 认定为不同的元素添加进去。

解决的办法就是重写 hashCode( )，最简单的返回 name 和 age 的哈希值的乘积即可。

## 2. Java的Optional类是什么
Java 的 Optional 类是 JDK 8 引入的一个容器类，主要用来解决 空指针异常 (NullPointerException) 的问题。
大概是2009年Tony Hoare谈到null这个概念The Billion Dollar Mistake
Optional 的设计初衷就是：提供一个类型安全的方式来表示“可选值”，减少 NPE(NullPointerException)，把“值可能缺席”明确为 API 契约。灵感来自函数式语言的 Option/Maybe，并在 Java 8 引入以配合 Stream/Lambda 风格


## 3.Java的IO流是什么？
Java 的 IO 流就是一套 用于处理数据输入和输出的类库，它把数据看作“流”来抽象。所谓“流”，就是数据在程序和外部设备之间像水流一样传输
结合源码加设计模式：
IO 的核心是 装饰者模式（BufferedInputStream、DataInputStream 等层层包装）。
字节流和字符流的转换是 适配器模式（InputStreamReader / OutputStreamWriter）。
抽象基类体现了 模板方法模式（父类定义通用逻辑，子类负责具体实现）


## 4. Java的不可变类是什么
在 Java 里，不可变类就是 一旦对象被创建，它的状态就不能再被修改 的类。
最典型的例子就是 String 类。你创建一个 String str = "abc"，它的值是固定的，如果你写 str = str + "d"，其实是创建了一个新对象 "abcd"，原来的 "abc" 并没有被改变

### 4.1为什么String要设计为不可变类
这个主要是出于线程安全和性能方面的考虑。
线程安全体现在，由于 String 是不可变的，多个线程共享一个 String 时不用担心它的同步问题；
性能体现在缓存哈希值和设计常量池上。
String 在被创建时就缓存了自己的哈希值，使用时直接拿出来就行，不用重新计算，这使得 String 适合用来作为 Map 的 Key，可以快速获得 Key 的哈希值，提高查找和比较的效率；
除此之外，基于 String 的不可变，Java 使用**常量池**来尽可能的共享相同的字符串，来节约 String 的存储空间。具体的，当我们使用字面值创建 String 时，会先去查它是否已经存在于常量池中，如果是则直接返回这个已存在字符串的引用，而不会创建新的对象。

### 4.2你谈到字符串常量池，能聊一聊字符串常量池吗
Java 里有一个 字符串常量池，它的作用就是 相同的字符串只存一份。
比如用直接赋值创建了两个字面值一样的字符串，比如字符串“ABC”
这两个字符串字面量其实指向的是常量池里的同一个对象，所以 s1 == s2 是 true。这样做的好处是 节省内存，同时相等比较效率也更高。

但是如果是使用new 方法赋值，这时候会创建新对象


常量池的管理依赖于 intern() 方法：
如果常量池里已经有，就直接返回池里的引用；
如果没有，就把当前字符串放进去并返回它。

### 4.3 StringBuilder、StringBuffer有什么区别？和String又有什么区别
StringBuffer和StringBuilder都是可变字符串，效率比String高
区别在于
StringBuffer它的特点是 线程安全 ——所有修改方法基本都用 synchronized 修饰
StringBuilder 不是线程安全的，但在单线程场景下性能比 StringBuffer 更高
StringBuffer 和 StringBuilder 内部都继承自 AbstractStringBuilder，大部分逻辑都在这个抽象父类里实现

### 4.4 你说StringBuffer和StringBuilder效率比String高，为什么，能举个例子吗
举个例子
String 是不可变类，每次修改都会生成一个新的对象，所以频繁拼接效率低。
StringBuffer和StringBuilder都是可变字符串，对于同样是实现拼接字符串的功能
它直接在内部的 char[] 数组上操作，容量不足时会扩容，但整体上只需要少量内存分配，效率远远高于 String 的反复创建。

甚至，编译器在处理 String s = s + "b"; 这种语句时，底层其实也是 新建一个 StringBuilder → append → toString。所以如果我们直接用 StringBuilder.append()，就能避免多余的中间对象创建

## 5.Java的访问修饰符
在 Java 里，一共有四种访问修饰符，分别是 public、protected、default（也叫包访问权限，不写就是默认的）、private。它们主要用来控制类、方法、变量在不同范围内的可见性
public 对所有可见
protected对同包可见，对子类可见
default只对同一个包里的类可见
private
只有在同一个类内部可见，其他类都不可见，包括子类

### 5.1 你说protected对同包可见，对子类可见那么我新建一个文件，该文件import了一个包，包里有一个protected类，我能用这个类吗？

不能

## 6.聊一聊Java异常吧

### 6.1 什么是异常
在 Java 里，异常就是程序运行过程中出现的错误事件。Java 把这些情况都封装成了 Throwable 类型及其子类，通过 异常处理机制来终止出错代码，转而执行异常处理逻辑，避免程序直接崩溃
Java 的异常体系从 Throwable 开始，分为两大类：

Error：严重错误，比如 OutOfMemoryError、StackOverflowError，一般是 JVM 层面的问题，程序没法处理。
Exception：常见的可捕获异常。又分为：
Checked Exception（受检异常）：必须在编译期显式处理，比如 IOException、SQLException。编译器会强制要求 try-catch 或 throws。
Unchecked Exception（运行时异常）：也叫 RuntimeException，比如 NullPointerException、IndexOutOfBoundsException。这种异常一般是代码逻辑 bug，不强制捕获


对于Exception，则有相关异常处理机制，
异常处理机制
try-catch-finally：捕获并处理异常。
throws：方法声明时把异常抛出去，让调用者处理。
throw：在方法内部主动抛出异常对象。
finally：无论是否异常都会执行，常用于释放资源


### 6.2 throw 和 throws 有啥区别？直接 try catch 不好吗，为啥还要抛呢？
throw 用在方法体内，表示某个地方需要抛出一个异常，是一个具体的动作。而 throws 用在方法声明后面，表示这个方法可能会抛出哪些异常，是一个声明。

这个需要分情况对待。首先，对于能在方法内处理的异常，可以直接用 try catch，但对于在当前方法内无法处理的异常，我们只能选择抛出，将它交给方法调用者去处理。其次，有时多个方法可能会抛出相同类型的异常，如果每个方法都用 try catch 会非常冗余，这时可以在这些方法中只 throw，在调用这些方法的地方整体 try catch，统一进行处理。

还有的话就是，有时候得把异常交给上层调用的人来处理。

### 6.3 try catch会影响性能吗？为什么抛出异常的时候会影响性能？
几乎不会，也就是程序没有异常时，try catch 加和不加性能几乎一样，只有在程序有异常需要捕获时才会影响性能。

原因在于，处理异常的 catch 语句在 class 文件中是用异常表实现的。异常表有四个字段，分别是 From, To, Target 和 Type。From 和 To 分别对应 try 块对应的行号，如果其中有异常抛出，会跳转到 Target 对应的行号，也就是 catch 语句对应的位置，而这个异常的类型记录在 Type 中。如果程序在 try 中没有异常抛出，最终会通过一条 goto 指令跳转到 try catch 块后的语句继续执行，这一条 goto 指令性能的消耗可以忽略不计。如果有异常，才会去查异常表，再跳转到对应的 catch 块位置去执行，这个过程可能会影响性能。

### 6.4 try-catch-finally 中，如果 catch 中 return 了，finally 还会执行吗？
会的，无论 catch 块中有异常还是有返回语句，最终一定会在方法真正结束之前执行 finally。但是需要考虑一种特殊情况，就是如果 try 块中抛出了异常但没有被 catch 捕获，且此时 finally 中也抛出了异常，那么 finally 中的异常会覆盖掉 try 中的异常，成为这个方法最终抛出的异常。实践中需要注意一下这个异常覆盖问题。

## 7.Java的序列化




## 8.Java的反射



## 9.Java的动态代理



## 10.Java的泛型是什么