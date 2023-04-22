---
layout: post
title:  "重新认识Java"
date:   2023-03-31 09:00:30 +0200
categories: java 
---

谈谈对java的粗浅理解, java不仅是一种语言, 也是一个平台, 它包括java运行环境、JVM、以及write once, run everywhere等特性

## JDK主要版本的特性
### [java8](https://www.oracle.com/java/technologies/javase/8-whats-new.html)
#### lambda表达式
#### StreamAPI（流的操作）
1. 创建流（集合）->筛选、过滤、去重、排序、映射map->收集(集合) 
* 普通收集: toList/toSet
* 高级收集: toMap/groupingby

#### 新日期时间API
* local Date Time、时间戳、格式化

#### Opational类
* 优化对null的处理，使代码更加健壮
* 链式取值: 层层嵌套对象取值，只有上层对象不为空时才能读取其属性值，然后继续调用，取出最终结果

### [java9](https://docs.oracle.com/javase/9/whatsnew/toc.htm#JSNEW-GUID-C23AFD78-C777-460B-8ACE-58BE5EA681F6)
#### 模块化
* javaSE程序更容易轻量级部署:jlink
* 改进组件间的依赖管理:module-info.java、requires\exports
* 改进性能和安全性：可读性、可访问性

#### jshell
交互式解释器.使Java 可以像 Python一样在 Shell 中运行代码并直接得出结果
#### StreamAPI增强
* takeWhile、dropWhile、ofNullable
* iterate的新重载方法可自定义终止逻辑

#### 不可变集合新创建方式of()方法
#### 垃圾收集器默认G1（之前是Parallel）
* 与 Parallel GC 相比，它减少了暂停时间

### [java11](https://www.oracle.com/java/technologies/javase/11-relnote-issues.html#NewFeature)
#### 字符串API增强
* isBlank(判断非空), strip(去空格), lines(换行)等等API
#### var可用于lambda表达式
#### 标准HttpClientAPI， 与Apache HttpClient、Jetty类似，支持Http2.0,也兼容1.1
#### 简化Java编译运行命令，以前先编译javac再运行java；现在只需一个命令java
#### ZGC（java15正式发布）
* 进一步减少停顿，可以将未使用的已提交内存返回给操作系统

#### 飞行记录器
* 是一种低开销的事件信息收集框架，主要用于对应用程序和 JVM 进行故障检查、分析

### [java17](https://www.oracle.com/java/technologies/javase/17-relnote-issues.html#NewFeature)
#### 新增文本块
* 之前版本要定义字符串，如JSON，相对麻烦，在17中更加便捷易读，更方便格式调整

#### switch表达式
* 允许switch有返回值, 不再有break，让switch块更易读

#### 密封类
* 可以更好的控制哪些类可以对超类进行继承，子类则可以是密封的、非密封的或final的

#### instanceof模式匹配
* 可以将类型转换和变量声明都在if中处理从而简化代码

#### Helpful NullPointerExceptions
* 准确显示发生NPE的精确位置

#### 增强的伪随机数生成器
* 增加了新的接口类型和实现，让在代码中使用各种 PRNG 算法变得容易许多
* 增加了 RandomGenerator 接口，为所有的 PRNG 算法提供统一的 API，并且可以获取不同类型的 PRNG 对象流
* 增加了一个新类 RandomGeneratorFactory 用于构造各种 RandomGenerator 实例

## JVM
### 内存区域划分
  <img src= "/assets/files/JVM内存模型.jpg" alt="加载错误" title="JVM内存模型"/>

* **程序计数器**: 每线程一个，任何时间一个线程只有一个方法执行，其会存储当前方法的JVM指令地址
* **Java虚拟机栈**: 每线程一个， 存储栈帧，一个栈桢对应一次方法调用，一个方法中调用了另一个方法，则会创建一个新的栈帧，成为新的当前帧
* **本地方法栈**: 每线程一个， 支持本地方法调用
* **堆**: 每进程一个， 被所有线程共享，所有对象实例都在堆中，堆还可以细分为老年代、新生代等
    * 新生代:即大部分java对象创建、销毁的区域，其内部又分为Eden、survivor区域；Eden又分为多个缓存区，是对象初始分配的区域， survivor区则存放GC中存活下来的对象
    * 老年代:即存放长生命周期的对象，大多数都是从survivor区拷贝过来的对象，也会存放在新生代Eden区中放不下的超大对象
* **方法区**: 每进程一个，被所有线程共享，用于存储元数据，例如：类结构信息、运行时常量池、字段、方法代码等
* **运行时常量池**: 方法区中的一部分，存放各种常量信息，例如: 字面量、符号引用

#### 为什么用元空间代替永久代
1. 在1.7中, 永久代内存是有上限的, 虽然我们可以通过参数来设置,但是JVM加载的class总数、大小是很难确定的, 所以很容易出现OOM问题. 但元空间是存储在本地内存里面, 内存上限比较大,可以很好的避免这个问题
2. 永久代的对象是通过FullGC进行垃圾收集, 替换成元空间以后，简化Full GC. 可以在不进行暂停的情况下并发地释放类数据, 同时也提升了GC的性能

### Java对象在JVM中的组成
1. 对象头
* Mark Word(标记字段): 用于存储对象自身运行时的数据,如hashCode、GC年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等. 在64位操作系统中占8个字节, 32位操作系统中占4个字节
* ClassPointer(类型指针): 指向当前实例对象所属哪个类. 开启指针压缩时占4个字节,未开启则占8个字节
* 数组长度: 只有对象数组才会存在. 占4个字节
2. 示例数据: 主要用来存储对象中的字段信息
3. 对齐填充: 用来补充实现Java对象大小的倍数对齐. Java对象的大小需要按照8字节或8字节的倍数来对齐,从而避免伪共享问题

#### 一个空对象有多大
1. 在开启压缩指针的情况下, 是16字节. 其中Markword占8字节、类型指针占4字节, 对齐填充4字节
2. 在关闭压缩指针的情况下, 默认16个字节, 不需要对齐填充


#### 对象最大年龄为什么是15
对象头里面有4个bit位来存储GC年龄, 4个bit位能够存储的最大数值是15

### HotSpotVM的核心
#### Class Loader Subsystem(类加载子系统)
编译好的.class文件会装载到Class Loader Subsystem, 它的主要功能是查找并验证类文件、完成相关内存分配和对象赋值
#### Runtime Data Areas(运行时数据区)
类文件加载到内存之后由Runtime Data Areas来完成数据存储和数据交换
1. 运行时数据区又分为线程共享内存区和线程隔离内存区
* 线程共享内存区: 包括方法区和堆区, 它们是程序员能够通过编写代码直接操作的内存区
* 线程隔离内存区: 包括栈区、程序计数器和本地方法栈，它们是完全由JVM来调度的内存区域
2. 通过Runtime Data Areas的五个内存区就能完成Java程序程序逻辑的执行和数据交换

#### Execution Engine(执行引擎)
主要包含即时编辑器和垃圾回收器
1. 即时编译器(JIT): 用来将字节码翻译成操作系统能够执行的CPU指令, 可以通过JVM参数来设置选择解释执行或编译执行
2. 垃圾回收器(GC): 主要负责对Runtime Data Areas的数据进行管理和回收, 其实就是对各种垃圾回收算法的实现, 可以通过JVM参数来设置
* 复制算法
* 标记清除算法
* 标记整理算法

### JVM常用参数
1. 堆栈配置
* -Xmx3550m 最大堆大小为 3550m
* -Xms3550m 设置初始堆大小为 3550m
* -Xmn2g 设置年轻代大小为 2g
* -Xss128k 每个线程堆栈大小为 128k
* -XX:MaxPermSize 设置持久代大小为 16m
* -XX: NewRatio 新生代老年代比例(默认1:2)
* -XX: SurvivorRatio Eden和Survivor比例调整(默认8:2)
* -XX:MaxTenuringThreshold 设置对象最大年龄,如果设置为0话,则新生代对象不经过Survivor区, 直接进入老年代
2. 垃圾收集
* -XX:+UseG1GC 选择垃圾收集器
* -XX: G1GCThreads 设置垃圾收集器线程数
3. 辅助信息
* -XX: +PrintGC
* -XX: +PrintCGDetails

### 监控和诊断JVM堆内外内存使用
* 利用图形化工具，例如：JConsole、VisualVM、JMC
* 利用命令行工具， 例如: jstat、jmap、jstack、jinfo
* GC日志
* jps获得进程号, top -H pid获得进程中所有线程的CPU耗时, jstack pid查看线程堆栈信息
* 在线分析工具: [fastthread](https://fastthread.io/)

### OOM
JVM 内存不够用，也即没有空闲内存，并且垃圾收集器也无法提供更多内存
* 在抛出异常之前，通常垃圾收集器会被触发，尽可能清理出空间；不过，也不是在任何情况下垃圾收集器都会被触发。
    * 例如，在去分配一个超大对象时，超过了堆的最大值，JVM 可以判断出垃圾收集并不能解决这个问题，会直接抛出 OutOfMemoryError
* 原因
    * 堆内存不足：例如：内存泄漏、堆大小不合理
    * java栈和本地方法栈：例如: 递归方法中没有退出条件，会一直进行压栈操作，JVM在判断内存不足进行扩展栈空间失败时就会抛出

### GC
#### 类型
* SerialGC: 单线程垃圾收集器，在Client模式下JVM的默认GC， 采用标记-整理算法
* ParNewGC: 是一个新生代GC实现，实际上是SerialGC的多线程版本, 通常与老年代的CMS GC配合使用
* CMSGC: 老年代GC, 以获得最短回收停顿时间为目标的GC, 采用标记-清除算法，存在内存碎片化问题
* ParallelGC：吞吐量优先GC，算法与SerialGC类似，新生代老年代GC都是并行进行的
* G1GC：兼顾吞吐量和停顿时间的GC, java9默认GC，其内存结构不是条带状划分，而是像棋盘一样划分为一个个region， 采用标记-整理算法
* ZGC：支持 T bytes 级别的堆大小，并且保证绝大部分情况下，延迟都不会超过 10 ms

#### 对象实例收集
* 引用计数法：即为对象添加引用计数，若计数为0，则表示对象可回收
* 可达性分析：即从一个被选为GC ROOTS的对象开始向下搜索，如果一个对象到GCROOTS之间没有引用链相连，则这个对象就是不可达，该对象经过两次标记仍为可回收对象，则该对象则面临回收。Java虚拟机栈、本地方法栈中引用的对象，方法去中静态属性和常量引用的对象都可以作为GCROOTS的对象
* 方法区中的元数据等收集

#### 算法
1. 复制算法: 将内存分为2份, 每次使用其中1分, 等待该份内存满后, 标记存活对象, 复制到到另一份内存. 其他对象清除,该份内存变为空闲状态,依次循环.
* 内存利用率50%, 且复杂大量对象时耗时长
* 新生代GC一般都是该算法, 将Eden区中活着的对象复制到survivor区中的to区域
2. 标记-清除: 存活的对象打上标记, 没有被标记的对象就是需要被回收的垃圾对象
* 会产生比较多的内存碎片. 随着时间的推移, 无法再大量分配连续的内存空间,会导致更加频繁地触发GC
* 老年代一般使用该算法
3. 标记-整理: 先标记出存活的对象,然后把所有存活的对象整理到内存空间的另一端, 没有被标记的对象就是需要被回收的垃圾对象

#### 流程
1. java创建的对象通常分配在Eden区域
2. 在Eden空间占用达到阈值后,会触发minorGC, 存活的对象将被复制到Survivor中的S1区(from)
3. 当再次出发minorGC时，Eden中存活对象和From区对象将被复制到S2区(to)
4. 在上述过程中, 对象每复制一次其生命周期都会加1, 当对象生命周期达到阈值, 就会晋升到老年代
5. 当老年代满了, 就会触发Full GC清理整个堆内存

##### Survivor存在意义
1. 如果没有Survivor，Eden区每进行一次 Minor GC, 存活对象直接送到老年代, 老年代很快被填满,触发 Major GC. 同时老年代内存空间远大于新生代,进行一次Full GC耗时比Minor GC长得多.
2. Survivor的存在意义就是减少被送到老年代的对象, 进而减少 Full GC发生
3. Survivor的预筛选保证只有经历16次Minor GC 还能在新生代中存活的对象,才会被送到老年代

##### Survivor这是两个区域的意义
设置两个Survivor区最大的好处就是解决了内存碎片化问题. S1中来自S0和Eden两部分的存活对象占用连续的内存空间, 避免了内存碎片化

### JVM调优
#### 调优时机
1. 堆内存(老年代)持续上涨达到设置的最大内存值
2. Full GC次数频繁
3. GC停顿时间过长(超过1秒)
4. 应用出现OutOfMemory等内存异常
5. 应用中有使用本地缓存且占用大量内存空间
6. 系统吞吐量与响应性能下降

#### 调优目标
通常关注内存占用、延时、吞吐量三个目标；大多数情况下调优侧重其中1到2个目标

#### 调优策略
1. 选择合适的GC
2. 调整内存大小——GC非常频繁
3. 调整内存区域大小比率——某一个区域GC频繁
4. 调整对象升老年代的年龄——老年代GC频繁
5. 调整大对象的标准——老年代GC频繁, 单个对象都很大
6. 调整GC的触发时机——经常Full GC, 程序卡顿严重
7. 调整JVM本地内存大小——GC次数、时机、回收的对象都正常, 堆内存空间充足, 但是发生OOM

#### 步骤
1. 确定应用需求和问题
2. 分析GC日志等来判断和掌握JVM和GC状态——>思考GC类型选择是否符合应用需求
3. 确定调优目标(量化数据)
4. 确定具体调整参数和软硬件配置
5. 验证


## 类加载器
把.class文件加载到JVM内存, 并生成Class对象
### 分类
1. Bootstrap ClassLoader(启动类加载器)
* 由C++语言实现(HotSpot),负责加载`<JAVA_HOME>\lib`目录或-Xbootclasspath参数指定路径中的类库
* 用原生代码来实现的, 并不继承自java.lang.ClassLoader
2. Extension ClassLoader(扩展类加载器):
* 负责加载`<JAVA_HOME>\lib\ext`目录或java.ext.dirs系统变量指定路径中的所有类库
3. Application ClassLoader(应用程序类加载器)
* 负责加载应用程序中的类, 可以直接使用这个类加载器. 一般没有自定义类加载器默认就是用这个类加载器

### 层次结构
Bootstrap ClassLoader <- Extension ClassLoader <- Application ClassLoader <- 自定义
### 双亲委派
如果一个类加载器收到类加载请求, 它首先不会自己去尝试加载这个类, 而委派给父类加载器完成,每个类加载器都是如此,只有当父加载器在自己的搜索范围内找不到指定类时,子加载器才会尝试自己去加载
* 自定义的Test.class需要加载 -> Application ClassLoader 委派给父类加载器 Extension ClassLoader -> Extension ClassLoader 又委派给父类加载器 Bootstrap ClassLoader
* Bootstrap ClassLoader 找不到Test.class -> 通知子类加载器 Extension ClassLoader 自己加载 -> Extension ClassLoader 找不到Test.class -> 通知子类加载器 Application ClassLoader 自己加载

#### 为什么需要双亲委派
为了保证Java核心库的类型安全, 对于Java核心库的类的加载工作由Bootstrap ClassLoader来统一完成,保证了 Java应用所使用的都是同一个版本的Java核心库的类
#### 双亲委派模型的破坏
1. 继承ClassLoader抽象类, 重写loadClass
2. 使用线程上下文加载器, 可以通过java.lang.Thread类的setContextClassLoader()方法来设置当前类使用的类加载器类型

## JMM(Java内存模型)——并发
线程和主内存之间的抽象关系, JMM中定义了JVM在计算机内存中的工作方式, 也即JMM是抽象, JVM是实现
   <img src= "/assets/files/Java内存模型.jpg" alt="加载错误" title="Java内存模型"/>

* 线程之间的共享变量存储在主内存中
* 每个线程都有一个私有的本地内存(JMM的一个抽象概念,并不真实存在). 它涵盖了缓存、写缓冲区、寄存器以及其他的硬件和编译器优化, 本地内存中存储了该线程以读/写共享变量的拷贝副本
* 从更低的层次来说，主内存就是硬件的内存, 而为了获取更好的运行速度，虚拟机及硬件系统可能会让工作内存优先存储于寄存器和高速缓存中
* 线程对变量的所有操作都必须在工作内存中进行, 而不能直接读写主内存中的变量. 不同的线程之间无法直接访问对方工作内存中的变量. 线程间变量值的传递均需要通过主内存来完成

### 主内存与工作内存之间的交互
一个变量如何从主内存拷贝到工作内存、如何从工作内存同步到主内存, JMM定义了八种操作来完成.
1. lock(作用于主内存的变量): 把一个变量标识为一条线程独占状态
2. unlock(作用于主内存变量): 把一个处于锁定状态的变量释放出来,释放后的变量才可以被其他线程锁定
3. read(作用于主内存变量): 把一个变量值从主内存传输到线程的工作内存中, 以便随后的load动作使用
4. load(作用于工作内存变量): 它把read操作从主内存中得到的变量值放入工作内存的变量副本中
5. use(作用于工作内存变量): 把工作内存中的一个变量值传递给执行引擎, 每当虚拟机遇到一个需要使用变量的值的字节码指令时将会执行这个操作
6. assign(作用于工作内存变量): 它把一个从执行引擎接收到的值赋值给工作内存的变量, 每当虚拟机遇到一个给变量赋值的字节码指令时执行这个操作
7. store(作用于工作内存变量): 把工作内存中的一个变量的值传送到主内存中, 以便随后的write的操作
8. write(作用于工作内存变量): 它把store操作从工作内存中一个变量的值传送到主内存的变量中

### volatile
volatile关键字作为一个修饰用来修饰变量, 它保证变量对所有线程可见性, 禁止指令重排, 但不保证原子性. 保证可见性和禁止指令重排都跟内存屏障有关. 为了实现volatile的内存语义,JMM采取以下保守策略
1. 在每个volatile写操作前面插入一个StoreStore屏障
2. 在每个volatile写操作后面插入一个StoreLoad屏障
3. 在每个volatile读操作后面插入一个LoadLoad屏障
4. 在每个volatile读操作后面插入一个LoadStore屏障


### 指令重排
```java
public class PossibleReordering {
    static int x = 0, y = 0;
    static int a = 0, b = 0;
    public static void main(String[] args) throws Exception {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(2,  2, 3, TimeUnit.SECONDS,
                new LinkedBlockingDeque<>(2), Executors.defaultThreadFactory(), new ThreadPoolExecutor.DiscardPolicy());
        int flag = 1000;
        for (int i = 0; i <= flag; i++) {

            threadPoolExecutor.execute(() -> {
                a = 1;
                x = b;
                //System.out.println(Thread.currentThread().getName() + "——" + x + "," + y);
            });
            threadPoolExecutor.execute(() -> {
                b = 1;
                y = a;
                //System.out.println(Thread.currentThread().getName() + "——" + x + "," + y);
            });
            // 执行结果包括0,0; 0,1; 1,0; 1,1
            System.out.println(x + "," + y);
        }
        threadPoolExecutor.shutdown();
    }
}
```
在实际运行过程中, 代码指令并不严格按照代码语句顺序执行, 而是乱序执行. 为了减少获取下一条指令所需数据产生的等待时间, 提高执行效率

### 内存屏障
一种CPU指令, 用于控制特定条件下的重排序和内存可见性问题. Java编译器在生成指令序列的适当位置会插入内存屏障指令来禁止特定类型的处理器重排序

<table class="table table-bordered table-striped">
	<thead>
		<tr>
            <th>屏障类型</th>
			<th>指令示例</th>
			<th>说明</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>LoadLoad</td>
			<td>Load1; LoadLoad; Load2</td>
			<td>在Load2及后续读取操作要读取的数据被访问前, 保证Load1读取操作完毕</td>
		</tr>
        <tr>
			<td>StoreStore</td>
			<td>Store1; StoreStore; Store2</td>
			<td>在Store2及后续写入操作执行前, 保证Store1写入操作对其他处理器可见</td>
		</tr>
		<tr>
			<td>LoadStore</td>
			<td>Load1; LoadStore; Store2</td>
			<td>在Store2及后续写入操作刷新到内存前, 保证Load1读取操作完毕</td>
		</tr>
        <tr>
			<td>StoreLoad</td>
			<td>Store1; StoreLoad; Load2</td>
			<td>在Load2及后续读取操作执行前, 保证Store1写入操作对所有处理器可见</td>
		</tr>
	</tbody>
</table>


### Happens-before
在多线程环境下, 因指令重排序的存在会导致数据的可见性问题即线程1修改某个共享变量对线程2不可见. happen-before提供**跨越线程的内存可见性保证**. 并不表示指令执行的先后顺序, 也就是说只要不对结果产生影响,仍然允许指令的重排序
* 程序顺序规则: 一个线程内执行的每个操作,都保证先于后面的操作
* volatile变量规则: 对于volatile变量的写操作, 保证先于随后对改变量的读取操作
* 锁规则: 对一个锁的解锁操作, 保证先于后续对该锁的加锁操作. 即线程1写操作对线程2读操作可见
* 线程启动规则: 线程1中对线程2的start()方法的调用先于此线程2的每一个动作
* 线程中断规则: 线程1中对线程2的interrupt()方法的调用先于线程2中的中断事件检测代码的发生
* 线程终结规则：线程1中对线程2的join()方法的调用,线程B终止之前对共享变量的修改在线程A可见
* 对象终结规则: 一个对象的初始化完成先于其finalizer()方法的开始
* 传递性规则: a操作先于b操作, b操作先于c操作, 那么a操作先于c操作

## 语言特性
### 面向对象
#### 基本要素
* 多态即重写、重载、向上转型等
* 继承是代码复用的基础，是一种比较紧耦合的关系，父类修改，子类也会变动，不能过度滥用
* 封装的目的是隐藏事务内部的实现细节，以便提高安全性和简化编程，它提供了合理的边界，以防止外部调用者接触到内部细节。比如多线程环境下暴露内部状态，导致并发修改的问题

### 面向对象设计原则
* 单一职责，即类或对象只有单一职责，若承担多种义务可以考虑拆分
* 开关原则， 即设计要对扩展开放，对修改关闭，也就是说应当保证平滑的扩展，避免因新增同类功能而修改已有实现
* 里氏替换， 即进行继承关系抽象时，凡可以用父类的地方，都可以用子类替换
* 接口分离， 即接口中不要定义太多方法，应视情况拆分出多个接口，将行为解耦，在未来维护中某个接口设计有变，不会影响使用其他接口的子类
* 依赖反转， 即高层次模块不应该依赖低层次模块，而应该基于抽象
 
### 泛型
### 反射
### lambda

## 性能分析
### 后台服务变慢如何诊断
* 首先分析是突然变慢还是长时间运行后变慢、是否重复出现
* 问题有可能来自于自身服务， 也可能是受系统中其他服务影响
* 先检查应用自身的错误日志，看是否大量出现了某种异常，如果没有，再监控CPU、内存等资源占用情况； 还可以监控GC日志，检查是否出现full GC等恶劣情况等


## 核心类库
### 异常机制
#### java.lang.Throwable
* Exception：程序正常运行期间可以预料到的意外情况，能够并且应该被捕获并做出相应处理，通常是编程导致的轻微错误
    * 例如：NullPointException
* Error：在正常情况下不大可能出现情况，绝大部分的error都会导致程序处于非正常、不可恢复状态， 通常是JVM无法解决的严重错误
    * 例如：OutOfMemoryError、NoClassDefFoundError

#### Excepotion和Error区别
#### 检查异常和不检查异常区别
1. 前者是在编写源代码期间就必须显示地捕获处理的异常,这是编译检查的一部分
* 例如: IOException、ClassCastException
* 抛出检查异常的时候需要上声明,这会直接破坏方法签名导致版本不兼容. 通常会使用RuntimeException包装
2. 后者又称为运行时异常, 在编译时不会强制检查,但运行时会出错,通常可以通过编码来避免逻辑错误,具体根据需要来判断是否需要捕获
* 例如: ClassNotFoundException、NPE、IndexOutOfBoundsException、 IllegalArgumentException、 ArithmeticException
* 可以去掉一些不需要的异常处理代码, 但可能忽略某些应该处理的异常,导致带来一些隐藏很深的Bug

#### try-catch-finally
* try块中是可能发生异常的业务代码；
* catch块中则是对捕获异常的处理，例如重试、打印堆栈信息、回滚事务等等；
* finally块中则是处理一些资源回收等操作

#### 实践注意事项
* try-catch代码段会产生额外的性能开销， 尽量不要用一的大的try包住整段代码，而仅仅放入可能发生异常的代码，也不要用异常来控制代码流程，这远比if/else低效
* 尽量不要捕获类似Exception的通用异常，而应当捕获特定异常，这不仅能够更加直观的体现出异常信息，保证程序不会捕获我们不需要捕获的异常，而且对于上层调用方来说也能够更好的根据不同业务情况处理异常
* 不要生吞异常，如果不把异常抛出来或输出到日志，在程序出现错误时就没办法快速判断和修复问题
* 遵循throw early, catch late原则

#### 反应式编程（异步）异常处理
* traceId串联

### BIO/NIO/AIO
#### BIO
* java.io提供的IO功能，例如File抽象，输入输出流等；是同步、阻塞的方式，即在读或写完成之前，线程会一直阻塞，优点是简单直观，缺点是效率和扩展性有限

#### NIO
* java.nio提供了Channel、Selector、Buffer等，可以构建多路复用、同步非阻塞的IO
* NIO由Buffer、Channel、Selector组成
* 利用单线程轮询事件的机制确定准备就绪的channel来决定做什么，仅select阶段是阻塞的

#### AIO
* java.nio2是一种异步非阻塞的IO方式，基于事件和回调机制
* 基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作

#### java.net中提供的部分网络API，如Socket、ServerSocket等也可以看多是同步阻塞的IO
 
### 并发
#### 线程安全
保证多线程环境下共享的、可修改的数据的正确性. 方式例如：封装、不可变
##### 基本特性
* 原子性: 即相关操作中途不会被其他线程干扰，一般通过同步机制实现
* 可见性: 即一个线程修改了共享变量，其他线程能够立即知晓，使用volatile保证可见性
* 有序性: 即保证线程内串行语义，而不是并行，以避免指令重排

##### 保证线程安全方式
1. 针对原子性
* Java提供了非常多的Atomic类. 比如AtomicInteger. 这些类都是通过CAS来保证原子性
* Java提供了各种锁机制来保证锁内的代码块在同一时刻只能被一个线程执行. 比如synchronized
2. 针对可见性
* 使用synchronized关键字加锁
* 使用volatile关键字
3. 针对有序性
* 使用synchronized关键字
* 通过Lock接口

#### 线程
从操作系统角度，可以简单理解为系统调度的最小单元，一个进程中有多个线程，是人物的真正执行者，有自己的栈、寄存器、本地存储等，线程之间共享文件描述符、虚拟地址空间等；java线程不允许启动两次，第二次调用会抛运行时异常异常illegalThreadStateException
##### 创建
1. 继承Thread类, 实现简单, 但扩展性不足, Java是单继承的语言，如果一个类已经继承了其他类,就无法通过
这种方式实现自定义线程
2. 实现Runnable接口, 扩展性好, 非常适合多线程处理一份资源的场景, 构造线程实例的过程相对繁琐
3. 实现Callable接口, 扩展性好,能支持回调并得到返回值,而且可以抛出受检查异常

#### 安全的中断线程
使用interrupt(), 配合isInterrupted(). stop()会强行终止, 不管任务有没有完成

##### 生命周期(线程状态流转)
* 新建(new): 被创建但尚未启动
* 就绪(Runnable): 已在JVM中执行,可能正在运行,也可能在等待分配cpu片段,在就绪队列中排队
* 阻塞(Blocked): 等待阻塞(wait())、同步阻塞(等待monitor lock)、其他阻塞(sleep()或join())
* 等待(Waiting): 等待其他线程采取操作,例如: 生产者消费者模式中,若任务条件尚未满足,就会让消费者线程等待.生产者线程去准备任务数据,之后通知消费者线程继续工作
* 超时等待(TimedWaiting): 在指定的时间后自动唤醒
* 终止(Dead): 线程完成任务终止运行, 即线程死亡

#### Executor框架
##### 组成
* Executor基础接口， 只有一个execute方法
* ExecutorService继承了Executor，提供了更完善的功能，包括shutdown、submit等方法
* Executors提供了各种方便的静态工厂方法
* ThreadPoolExecutor、ScheduledThreadPoolExecutor等线程池的实现

##### 线程池
通常都是用Executors提供的创建线程池的方法去创建不同配置的线程池，主要主要区别在于不同的ExecutorService以及初始参数
1. newCachedThreadPool
* 用于处理大量短时任务, 因为没有最大线程数限制
* 其特点是会缓存线程并重用, 无缓存线程时就创建新的线程
* 默认线程keepAlive为60s, 超过60s就会被终止并移出缓存
* 使用synchronousQueue作为工作队列,它不能存储任何元素, 每提交一个任务到队列里都需要分配一个线程处理
2. newFixedThreadPool
* 重用指定数目的线程,内部使用无界队列
* 若工作任务超过指定数目线程, 就会等待空闲线程出现
* 如果有线程终止,就会创建新线程, 以补足指定的线程数目
3. newSingleThreadExecutor
* 创建线程数为1的线程池, 保证任务顺序执行
* 一次只有一个任务执行, 内部使用无界队列
4. newScheduledThreadPool/newSingleThreadScheduledExecutor:
* 创建单个或多个线程以进行定时的工作调度
5. newWorkStealingPool: 内部构造一个ForkJoinPool, 利用工作窃取算法并行处理

##### 线程池状态
1. RUNNING
* 该状态的线程池会接收新任务,并处理阻塞队列中的任务
* 调用线程池的shutdown()方法,可以切换到SHUTDOWN状态
* 调用线程池的shutdownNow()方法,可以切换到STOP状态
2. SHUTDOWN
* 该状态的线程池不会接收新任务,但会处理阻塞队列中的任务
* 队列为空,并且线程池中执行的任务也为空时, 进入TIDYING状态
3. STOP
* 该状态的线程不会接收新任务,也不会处理阻塞队列中的任务, 而且会中断正在运行的任务
* 线程池中执行的任务为空时, 进入TIDYING状态
4. TIDYING
* 该状态表明所有的任务已经运行终止, 记录的任务数量为0
* 调用线程池的terminated(), 进入TERMINATED状态
5. TERMIATED
* 该状态表示线程池彻底终止

##### 线程池实现
1. ThreadPoolExecutor构造函数:
* corePoolSize: 线程池核心线程数最大值
* maximumPoolSize: 线程池最大线程数大小
* keepAliveTime: 线程池中非核心线程空闲存活时间大小
* unit: 线程空闲存活时间单位
* workQueue: 存放任务的阻塞队列
* threadFactory: 用于设置创建线程的工厂, 可以给创建的线程设置有意义的名字,方便排查问题
* handler: 线程池饱和策略事件,主要有四种类型
2. 四种饱和拒绝策略
* AbortPolicy: 抛出一个异常, 默认该策略
* DiscardPolicy: 直接丢弃任务
* DiscardOldestPolicy: 丢弃队列里最老的任务,将当前这个任务继续提交给线程
* CallerRunsPolicy: 交给线程池调用所在的线程进行处理
3. 流程
* 提交任务, 判断核心线程是否已满, 是则进入下一步, 否则创建核心线程执行任务
* 判断任务队列是否已满, 是则进入下一步, 否则将任务加入工作队列
* 判断是否达到最大线程数, 是则进入下一步, 否则创建非核心线程执行任务
* 按照拒绝策略处理

##### 线程池实践
* 避免任务堆积、避免过度扩展线程、避免死锁等
* 线程池大小选择
    * 主要进行计算任务，通常线程数=cpu核心数
    * 主要进行I/O任务，通常线程数 = CPU核数 × 目标CPU利用率 ×（1 + 平均等待时间/平均工作时间）
  
#### 并发工具类
##### 主要内容
* 提供了比 synchronized 更加高级的各种同步结构，包括 CountDownLatch、CyclicBarrier、Semaphore
    * CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同: CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；
    * 而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行； CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的；
    * Semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限
* 各种线程安全的容器，比如最常见的 ConcurrentHashMap
* 各种并发队列实现，比如典型的 ArrayBlockingQueue
* Executor 框架，可以创建各种不同类型的线程池

#### Future与FutureTask
FutureTask实现了RunnableFuture接口, RunnableFuture继承了Runnable和Future接口. 可以将Runnable看作生产者, Future看作消费者, FutureTask被两者共享, 生产者运行run方法计算结果, 消费者运行get方法获取结果

### 安全
#### 注入攻击
攻击者将不可信的动态内容注入到程序中，从而产生与期望不一致的结果
* SQL注入，是最常见的注入攻击方式，往往是因为后端SQL语句是通过界面输入信息拼接而来，利用输入信息篡改SQL
    * 输入校验，限定合法输入内容
    * 利用PreparedStatement，而不是动态SQL
    * 限制数据库查询、修改等权限
* XML注入， 跟SQL注入类似，利用用户输入修改XML的数据格式或添加新的XML节点，导致解析XML异常
    * 输入正则校验
    * 输入转码处理
* 操作系统命令注入

#### 中间人攻击
攻击者与通讯的两端分别创建独立的联系，并交换其所收到的数据，使通讯的两端认为他们正在通过一个私密的连接与对方直接对话，但事实上整个会话都被攻击者完全控制
#### Dos攻击
常见表现为利用大量机器发送请求，将目标网站的带宽和其他资源耗尽，导致无法正常响应合法用户的请求
#### 安全机制
##### 运行时安全机制
* 类加载过程中的字节码验证，以防止恶意代码或不符合规范代码影响运行
* 利用SecurityManger，限制代码的运行时行为能力， 例如限制对文件系统的操作权限

##### 安全框架API
* 加密解密API、授权鉴权API

##### JDK集成的安全工具
* keytool，管理密钥、证书等

## 工具
#### 辅助工具
* jlink:自定义jre /减少内存消耗/
* jdeps

#### 编译器
* javac
* sjavac

#### 诊断工具
* jmap
* jstack
* jconsole
* jcmd
