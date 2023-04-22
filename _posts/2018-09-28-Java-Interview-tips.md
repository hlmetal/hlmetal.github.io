---
layout: post
title:  "java知识盘点"
date:   2018-09-28 09:00:30 +0200
categories: java
---

java基础知识盘点

## 基本类型
8种基本类型(boolean,byte,char,short,long,float,double),其他类型都是引用类型
### int、Integer区别
* int是java的8个原始数据类型之一, 默认为0, 存储在栈空间
* Integer是int的包装类, 默认为null, 存储在堆内存, 有一个int类型的字段存储数据, 通过自动装箱/拆箱(编译阶段),java可以根据上下文自动进行转化;自动装箱valueOf会用到缓存IntegerCache
* *注意: 避免无意中的装箱/拆箱行为, 特别在性能敏感场合*
  
### Integer缓存机制
1. Integer对-128到127之间的数据做了一层缓存, 如果Integer类型的目标值在此之间,就直接从缓存里面获取Integer这个对象实例并返回, 否则创建一个新的Integer对象.这减少频繁创建Integer对象带来的内存消耗, 从而提升性能
2. -128-127的数据在int范围内是使用最频繁的

### 为什么两个Integer不能==判断
如果定义两个Integer对象, 并且这两个Integer的取值范围正好在-128到127之间, 直接用==号来判断可以, 因为两个对象指向的地址是同一个. 但数据超过IntegerCache的取值范围, 就不行

### AtomicInteger的底层实现原理
  <img src= "/assets/files/to_be_supplemented.png" alt="加载错误" title="有待补充"/>

### 判断奇偶数
1. 对2取余(%2)
2. 和1做与操作(&1)

### 类型强转
1. int可以强转为byte, 但int的高24位会被丢弃

### 为什么基本类型的存储都是8位整数倍？
计算机底层按照Byte为单位操作数据, 其他类型设计为8的整数倍, 方便拆分和合并处理

## run()与start()区别
### 启动线程为什么不调run()而是start()
1. start()方法是Java线程约定的内置方法, 能够确保代码在新的线程上下文中运行
2. start()方法包含了创建新线程的特殊代码逻辑. run()方法是自己写的代码,很显然没有这个能力
3. 如果直接调用run()方法, 那么它只是一个普通的方法调用,程序中依然只有一个主线程, 并且只能顺序执行,需要等待run()方法执行结束后才能继续执行后面的代码
4. 创建线程的目的是为了更充分地利用CPU资源, 如果直接调用run()方法就失去了创建线程的意义

### 两次调用start()后果
第一次成功, 第二次抛异常IllegalThreadStateException. 在第一次调用start()的时候, 线程可能处于终止或其他非NEW的状态. 再次调用start()的时候, 相当于让这个正在运行的线程重新运行一遍, 不管是从线程安全的角度来看, 还是从线程本身的执行逻辑来看,它都是不合理的.为了避免这个问题出现,Java会先去判断当前线程的运行状态

## SimpleDateFormat是线程安全的吗
SimpleDateFormat不是线程安全的, 其内部有一个Calendar对象引用, 这个对象主要用来储存SimpleDateFormat的相关日期信息
### 解决办法
1. 可以把SimpleDateFormat定义成局部变量, 这样每个线程调用的时候都创建一个新的实例
2. 可以使用ThreadLocal, 把SimpleDateFormat变成一个线程私有的对象
3. 定义SimpleDateFormat时, 加上同步锁, 这样就能够保证在同一时刻只允许一个线程操作
4. Java8中引入了一些线程安全的日期操作API, 比如LocalDateTimer、DateTimeFormatter


## equalsy与==区别
1. equals
* 字符串, 使用equals表示判断字符串内容是否相同
* Object.equals(), 比较的也是内存地址是否相同
* 重写Object.equals(), 可以定义两个对象是否相等
2. == 
* 基本类型, 使用==表示判断值是否相等
* 引用类型, 使用==表示判断两个对象指向的内存地址是否相同

## 重载和重新区别
1. 重载是指一个类有多个同名方法, 但这些方法的签名不同
2. 重写是指子类继承父类时, 将父类的方法覆盖
3. 重写方法的修饰符必须大于父类方法的修饰符 
4. 重写方法抛出的异常必须小于父类方法抛出的异常

## equals与hashChode
1. 两个对象euqals为true, 则hashCode相等; 两个对象hashCode相等, equals不一定为true
2. hashCode用来在散列存储结构中确定对象的存储地址
3. 相等的对象必须有相等的散列码, 因此重写equals时也必须重写hashCode

## Comparator与Comparable区别

## &与&&

## 类实例化顺序
父类静态代码块 -> 子类静态代码块 -> 父类非静态代码块 -> 父类构造器 -> 子类非静态代码块 -> 子类构造器

## java创建对象的方式
1. new
2. 利用反射, Class.newInstance()
3. 调用对象的clone()方法
4. 利用反序列化, 调用java.io.ObjectInputStream的readObject()方法
5. 使用UnsafeF

## 序列化与反序列化
序列化的核心目的是为了解决网络通信之间的对象传输问题, 即如何把当前JVM进程里面的一个对象跨网络传输到另外一个JVM进程里面
1. 序列化是把内存里面的对象转化为字节流, 以便用来实现存储或者传输
2. 反序列化是从文件或者网络上获取到的对象的字节流,根据字节流里面保存的对象描述信息和状态重写构建出1个新对象
3. 序列化的前提是保证通信双方对于对象的可识别性, 因此需要把对象先转化为通用的解析格式, 比如json、xml等, 然后再转化为数据流进行网络传输,从而实现跨平台和跨语言的可识别性

## char与varchar区别
1. char是固定长度的字符串, varchar是可变长度的字符串
2. 存储效率
* char类型每次修改以后存储空间的长度不变, 所以效率更高
* varchar每次修改数据都需要更新存储空间长度, 效率较低
3. 存储空间不同
* char不管实际数据大小,存储空间是固定的
* varchar存储空间等于实际数据长度
4. 应用
* char适合存储固定长度字符串, 例如存储MD5
* varchar适合存储可变长度字符串


## 匿名内部类
1. 没有名字
2. 不能是抽象的, 必须继承一个抽象类或者实现一个接口
3. 不能定义任何静态成员和静态方法
4. 当所在的方法的形参需要被匿名内部类使用时, 必须声明为final
5. 不能访问外部类方法中的局部变量, 除非该变量被声明为final类型

## break与continue
1. break可以跳出switch语句体, 可以终止本层循环
2. continue可以跳出本次循环, 立即进入下次循环

## try-with-resource
1. 代码更加优雅, 行数更少, 不用再写finally关闭资源
2. 资源自动关闭, 不用担心忘记关闭, 以及内存泄漏问题
```java
try(BufferedReader br = new BufferedReader(new FileReader( "D:/test.txt"))) {
  System.out.println(br.readLine());
} catch (IOException e) {
  // exception handling
}
```

## 守护线程

## 多继承问题
java不支持多继承, 因为子类继承多个父类的相同方法或属性, 将不知道具体继承哪个的, java提供的接口和内部类可以实现类似多继承效果


## 静态内部类和非静态内部类
1. 静态内部类可以有静态成员(方法, 属性),而非静态内部类则不能有静态成员(方法, 属性)
2. 静态内部类只能够访问外部类的静态成员和静态方法, 而非静态内部类则可以访问外部类的所有成员
3. 实例化静态内部类与非静态内部类的方式不同
4. 调用静态内部类的方法或静态变量,可以通过类名直接调用

## 深拷贝和浅拷贝
1. 浅拷贝: 只复制某个对象的指针, 而不复制对象本身, 两个引用指针指向同一内存地址
2. 深拷贝: 创建一个一模一样的新对象
3. 克隆方法: 实现Cloneable接口, 重写clone()方法、 序列化

## Object中定义了哪些方法
1. getClass() 获取类结构信息
2. hashCode() 获取哈希码
3. equals(Object) 默认比较对象地址值是否相等, 子类可以重写比较规则
4. clone() 用于对象克
5. toString() 对象转变成字符串
6. notify() 多线程中唤醒功能
7. notifyAll() 多线程中唤醒所有等待线程的功能
8. wait() 让持有对象锁的线程进入等待
9. wait(long timeout) 让持有对象锁的线程进入等待,设置超时毫秒数时间
10. wait(long timeout, int nanos) 让持有对象锁的线程进入等待, 设置超时纳秒数时间
11. finalize() 垃圾回收前执行的方法

### nofity与wait为什么要在synchronized代码块中
wait和notify用来实现多线程之间的协调. wait表示让线程进入到阻塞状态, notify表示让阻塞的线程唤醒. 它们成对存在. 
1. 在通过共享变量来实现多个线程通信的场景里面,参与通信的线程必须要竞争到这个共享变量的锁资源,才有资格对共享变量做修改,修改完成后就释放锁,那么其他的线程就可以再次来竞争同一个共享变量的锁来获取修改后的数据,从而完成线程之前的通信

### wait与sleep是否会触发锁释放及CPU资源释放
1. Object.wait()方法会释放锁资源以及CPU资源
2. Thread.sleep()方法,不会释放锁资源, 但是会释放CPU资源


## public、private、protected、default访问区别
1. public: 当前类、子类、同包、其他包
2. protected: 当前类、子类、同包
3. default: 当前类、同包
4. private: 当前类

## Math类中的小数取整
1. round(): 返回四舍五入, 负 .5 小数返回较大整数, 如 -1.5 返回 -1
2. ceil(): 返回小数所在两整数间的较大值, 如 -1.5 返回 -1.0。
3. floor(): 返回小数所在两整数间的较小值, 如 -1.5 返回 -2.0

## 值传递和引用传递
1. 值传递是对基本型变量而言,传递的是该变量的一个副本,改变副本不影响原变量
2. 引用传递一般是对于对象型变量而言, 传递的是该对象地址的一个副本, 并不是原对象本身,对引用对象进行操作会同时改变原对象

## this与super关键字作用
1. this
* 对象内部指代自身的引用
* 解决成员变量和局部变量同名问题
* 可以调用成员变量, 不能调用局部变量
* 可以调用成员方法
* 在普通方法中可以省略this
* 在静态方法当中不允许出现this
2. super
* 调用父类的成员或者方法
* 调用父类的构造函数

## String、StringBuffer、StringBuilder区别
* String是JAVA非常基础和重要的类, 提供了构造和管理字符串的各种功能。它是immutable类, 其所有属性都是final的, 由于不可变性, 因此在拼接、裁剪字符串时都会产生新的String对象. 存储在字符串常量池
* StringBuffer则是为解决上述产生许多中间对象问题而提供的一个类, 通过append()方法来动态构造数据, 它是一个线程安全的类, 加上了synchronized关键字. 存储在堆内存空间
* StringBuilder与StringBuffer基本没有区别, 只是去掉了synchronized关键字, 不再是线程安全的, 减少了性能开销, 是绝大多数情况下处理字符串的选择. 存储在堆内存空间

### ## String s="a"与String ns = new String("a")区别
1. 前者先从常量池找有没有"a", 有就让s指向它; 没有就在常量池创建, 并让s指向它
2. 对于后者
* 字符串常量"a"不存在, 创建2个对象, 在堆中创建new String()这个对象, 再创建字符串常量"a"
* 字符串常量"a"存在, 创建1个对象

### String s = "a" + "b" + "c"创建了几个对象
只创建1个对象, Java编译器对字符串常量直接相加的表达式进行了优化,不等到运行期去进行加法运算, 而是在编译时就去掉了加号,直接将其编译成一个这些常量相连的结果, 即"a"+"b"+"c"相当于直接定义一个 "abcd=" 的字符串

### String类为什么不能继承
1. 使用final修饰
2. 效率, 作为最常用的类之一, 禁止被继承可以提高效率
3. 安全性, String类中有很多调用底层的本地方法, 调用了操作系统API, 禁止被继承可以防止恶意代码

### String类常用方法
1. indexOf()：返回指定字符的索引
2. charAt()：返回指定索引处的字符
3. replace()：字符串替换
4. trim()：去除字符串两端空白
5. split()：分割字符串, 返回一个分割后的字符串数组
6. getBytes()：返回字符串的byte类型数组
7. length()：返回字符串长度
8. toLowerCase()：将字符串转成小写字母
9. toUpperCase()：将字符串转成大写字符
10. substring()：截取字符串
11. equals()：字符串比较

## String反转
1. charAt倒循环输出到新对象
2. reverse()方法

### StringBuffer和StringBuilder底层实现
底层都是利用可修改的char数组(java 9之后为byte数组), 继承自AbstractStringBuilder, 默认大小是16

## vector、ArrayList、LinkedList区别
* Vector是线程安全的动态数组, 内部使用对象数组存储数据, 可根据需要自动增加容量(扩容1倍), 当数组满时, 会创建新数组, 并拷贝原有数组数据
* ArrayList时应用更加广泛的动态数组, 默认长度为10, 本身不是线程安全的, 与Vector类似也是对象数组保存数据.也会自动扩容, 增加之原来的1.5倍长度. 自动扩容时创建一个新的数组, 使用Arrays.copyOf()方法把原数组中的数据拷贝到新数组中
* ArrayList和Vector都是根据索引来取数据, 查询数据快而插入数据慢
* LinkedList即双向链表, 不是线程安全的, 也没有扩容问题. 获取数据需要根据索引序号,向前或者向后遍历. 插入数据时只需要记录本项的前后项即可, 速度快

### 应用场景
* Vector和ArrayList内部以数组形式顺序存储元素, 适合随机访问的场合, 除了尾部插入、删除操作外, 性能较低
* LinkedList进行节点插入、删除比较高效, 但随机访问性能则比上述慢

## Hashtable、HashMap、TreeMap区别
*  Hashtable继承自Dictionary类, 是线程安全的, 使用synchronized实现, 效率比较低, 现在更多使用ConcurrentHashMap来实现同步, 效率高很多
* HashMap继承自AbstractMap, 是应用更加广泛的哈希表, 可以存储null键和值, 不是同步的, 在进行put/get操作时能达到常数时间的性能, 默认大小16
* TreeMap也继承自AbstracMap, 利用红黑树实现, 并且实现了SortMap接口, 能够对保存的记录根据键进行排序(默认升序), 也可以自定义实现Comparator接口来实现排序方式, 在进行get/put/remove时可以达到O(logn)的时间复杂度

## HashMap和ConcurrentHashMap
### ConcurrentHashMap
1. 1.7 由Segment数组和HashEntry数组组成, 即把桶数组切分成更小的segment数组, 每个segment有若干HashEntry. 每个segment配一把锁(segment继承自ReentrantLock, 即可重入锁), 当线程占用其中一段的锁时, 其他段仍能够被其他线程访问
2. 1.8 由Node数组+链表+红黑树的结构组成, 在锁的实现上抛弃了原来的segment(segment仍然存在, 只是仅用做保证序列化时的兼容性);
* 采用CAS+synchronized实现了更加细粒度的锁(synchronized的实现得到了优化, 锁不是加在整个节点上, 而是链表头上减少了内存开销)

### HashMap源码解读(数组+链表+红黑树)
* HashMap可以看作是Node数组和链表的复合结构, 数组被分为一个个桶, 通过哈希值决定键值对在数组上的寻址, 哈希值相同的则以链表形式存储, 链表长度超过默认阈值(8)就会转换成树形结构, 树节点小于6则转为链表
* h = key.hashCode() ^ h >>> 16(高位移位到低位进行异或运算)：哈希值的主要差异来自于高位, 但hashmap中的哈希寻址是忽略容量上的高位的, 这样处理可以有效避免哈希碰撞
* 容量、负载因子、树化

#### 属性
1. loadFactor: 负载因子, 默认0.75
2. threshold: 链表长度,  
3. size: 实际键值对数量
4. modCount: 内部结构变化次数
5. DEFAULT_INITIAL_CAPACITY: 默认容量, 16

#### 动态扩容
当HashMap里的元素个数超过阈值(负载因子(0.75)*容量大小(16))的时候会自动触发扩容, 扩容到原来的2倍, 最大为Integer.MAX_VALUE. 
* 创建新的HashMap -> 将数据拷贝到新HashMap

#### 负载因子为什么是0.75
负载因子表示hashmap中的元素填充程度
1. 负载因子的值越大, 触发扩容的元素个数就越多. 虽然整体空间利用率会比较高, 但是Hash冲突的概率也会增加
2. 反之, hash冲突小, 但浪费内存多, 扩容频率高

#### 为什么用红黑树
1. 不用二叉树是因为红黑树作为一种平衡树, 查询时间复杂度O(logn)要小于O(n)
2. 不用平衡二叉树是因为平衡二叉树比红黑树的平衡性要求更严格, 旋转次数更多, 保持平衡的效率低, 所以插入删除效率比红黑树低

#### 为什么链表长度大于8才转树结构
1. 红黑树平均查找长度log(n),如果长度为8,平均查找长度为log(8)=3,链表平均查找长度为n/2, 当长度为8时,平均查找长度为8/2=4, 这才有转换成树的必要
2. 链表长度如果小于等于6, 6/2=3,而log(6)=2.6,虽然速度也很快, 但转化为树结构和生成树时间并不会短

#### 扩容导致的死循环问题
1. 在1.7中, HashMap的数据插入采用的是头插法,新插入的数据会从链表的头节点进行插入, 而在扩容时, 旧HashMap中链表元素顺序为A,B,C, 新HashMap在插入时元素顺序为C,B,A
2. 在1.8之后改为尾插法, 解决了此问题

### 哈希冲突解决
1. 开放寻址法: 包含线性探测、平方探测等. 就是从发生冲突的位置开始,按照一定的次序从hash表中找到一个空闲的位置, 然后把发生冲突的元素存入到这个空闲位置中. ThreadLocal就用到了线性探测法来解决hash冲突
2. 链式寻址法: 这是一种非常常见的方法, 简单理解就是把存在hash冲突的key以单向链表的方式来存储
3. 再hash法: 就是当通过某个hash函数计算的key存在冲突时, 再用另外一个hash函数对这个key做hash，一直运算直到不再产生冲突
4. 建立公共溢出区: 就是把hash表分为基本表和溢出表两个部分, 凡事存在冲突的元素,一律放入到溢出表中


### 为什么ConcurrentHashMap不允许null, 而HashMap可以
1. ConcurrentHashMap允许插入null,就会出现歧义. 
* 值没有在集合中, 即key不存在, 所以返回的结果就是null
* 值就是null, 即key存在但value是null, 所以返回的结果就是它原本的null
2. 歧义原因是在多线程环境下
* 假设线程1在取值时使用containsKey(key), 判断是否存在key, 期望结果是false
* 但在线程1返回前, 线程2调用put()插入一个value = null的值, 线程1返回的结果变成true
3. HashMap主要是单线程环境下使用, 可以通过HashMap的containsKey(key)方法来区分这个null到底是插入值是null, 还是本就没有才返回的null

## CountDownLatch与CyclicBarrier区别
1. CountDownLatch: 一个或多个线程,等待其他多个线程完成某件事情之后才能执行
*  举例: 运动会跑步比赛, 裁判只有等待多名运动员到齐(多个子线程)才能发令(主线程)
2. CyclicBarrier: 多个线程互相等待,直到到达同一个同步点,再继续一起执行
* 举例: 舞蹈队排练, 人员需要相互等待都到齐了, 才能开始
3. <img src= "/assets/files/CoutdownLatch与CyclicBarrier.png" alt="加载错误" title="CoutdownLatch与CyclicBarrier"/>
* CountDownLatch的计数器只能使用一次, 而CyclicBarrier的计数器可以使用reset()方法重置
* CyclicBarrier能处理更为复杂的业务场景,比如计算发生错误,可以结束阻塞,重置计数器,重新执行程序
* CyclicBarrier提供getNumberWaiting()方法,可以获得CyclicBarrier阻塞的线程数量,还提供isBroken()方法, 可以判断阻塞的线程是否被中断
* CountDownLatch会阻塞主线程,CyclicBarrier不会阻塞主线程,只会阻塞子线程

## Fork/Join
用于并行执行任务的框架, 一个大任务分割成若干个小任务,最终汇总每个小任务结果后得到大任务结果(分而治之)
1. 大任务拆分成小任务, 放到不同队列执行, 交由不同的线程分别执行
2. 有的线程优先把自己负责的任务执行完了,有的线程还在处理自己的任务.为了提高效率就需要"工作窃取"
* 某个线程从其他队列中窃取任务进行执行. 一般就是指快线程抢慢线程的任务来做
* 为了减少锁竞争,通常使用双端队列,即快线程和慢线程各在一端

## ThreadLocal原理
ThreadLocal是一种线程隔离机制,它提供了多线程环境下对于共享变量访问的安全性.
* 一般在多线程访问共享变量的场景中, 是对共享变量加锁, 从而保证在同一时刻只有一个线程能够对共享变量进行更新, 并且基于Happens-Before规则里面的监视器锁规则, 又保证了数据修改后对其他线程的可见性. 
* ThreadLocal用了一种空间换时间的设计思想, 每个线程里面都有一个容器来存储共享变量的副本,然后每个线程只对自己的变量副本来操作,这样既解决了线程安全问题,又避免了多线程竞争加锁的开销
```java
public class Thread implements Runnable {
ThreadLocal.ThreadLocalMap threadLocals = null;
}

static ThreadLocal<String> localVariable = new ThreadLocal<>();
```
1. Thread类有一个类型为ThreadLocal.ThreadLocalMap类的实例变量threadLocals,即每个线程都有一个属于自己的ThreadLocalMap
2. ThreadLocalMap内部维护着Entry数组, 每个Entry代表一个完整的对象, 其中key是ThreadLocal本身,value 是ThreadLocal的泛型值
3. 每个线程在往ThreadLocal里设置值的时候, 都是往自己的ThreadLocalMap里存, 读也是以某个ThreadLocal 作为引用,在自己的map里找对应的key,从而实现了线程隔离

### ThreadLocal内存泄漏问题
```java
static class ThreadLocalMap {
  static class Entry extends WeakReference<ThreadLocal<?>> {
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
      super(k);
      value = v;
    }
  }
}
```
1. 原因: ThreadLocalMap中使用的key是ThreadLocal的弱引用, 弱引用比较容易被回收. 如果没有强引用指向key, 就会被回收, 导致Entry的key为null, value没有外部强引用指向它应该也被回收, 但是ThreadLocalMap与Thread生命周期一致, 而Entry又属于ThreadLocalMap, 从而导致Entry中的key没了, value还在, Entry对象不能被回收, 这就会造成了内存泄漏问题
2. 解决办法
* 使用完ThreadLocal后及时调用remove()方法释放内存空间
* ThreadLocal变量尽可能定义为static final
3. ThreadLocal内部优化
* 调用set()方法时, ThreadLocal会进行采样清理、全量清理, 扩容时还会继续检查
* 调用get()方法时, 如果没有直接命中或者向后环形查找时也会进行清理
* 调用remove()时, 除了清理当前Entry, 还会向后继续清理

### 应用场景
1. 数据库连接池
2. 会话管理

## synchronized和ReentrantLock区别
1. synchronized是jvm内部的内置锁, 提供了互斥的语义和可见性, 当一个线程获取锁后, 其他线程只能等待或阻塞；可以修饰方法或特定代码块
2. ReentrantLock即重入锁, 语义与前者基本相同, 但提供了更多的实用方法, 能够做到前者不能做到的细节控制, 例如: 设置公平性
* 可重入: 就是获得锁的线程在释放锁之前再次去竞争同一把锁的时候, 不需要加锁就可以直接访问

### Lock(J.U.C包的接口)
Lock有很多实现类, 例如ReentrantLock
1. Lock通过lock与unlock方法实现锁粒度控制, 即在lock与unlock之间的代码都是加锁的
2. Lock提供了非阻塞竞争锁的方法tryLock(), 通过返回true/false告诉当前线程, 是否其他线程正在使用锁
3. 乐观锁机制, 采用CAS自旋锁进行优化
4. Lock提供了公平和非公平锁两种机制, synchronized只提供了非公平锁机制
* 公平锁是指线程竞争锁资源的时候, 如果已经有其他线程正在排队或者等待锁释放,那么当前竞争锁的线程是无法去插队的
* 非公平锁就是不管是否线程再排队等待锁, 当前线程都会去尝试竞争一次锁

#### ReentrantLock
1. ReentrantLock是通过互斥变量, 使用CAS机制来实现的. 没有竞争到锁的线程, 使用AbstractQueuedSynchronizer这样一个队列同步器来存储. 底层是通过双向链表来实现的. 当锁被释放之后, 会从该队列里面的头部唤醒下一个等待锁的线程
2. 在该队列里面有一个成员变量来保存当前获得锁的线程, 当同一个线程下次再来竞争锁的时候, 就不会去走锁竞争的逻辑, 而是直接增加重入次数

##### AQS为什么使用双向链表
1. 没有竞争到锁的线程会加入阻塞队列
* 阻塞等待的前提是当前线程所在节点的前置节点是正常状态,以避免链表中存在异常线程导致无法唤醒后续线程
* 因为要判断前置节点的状态, 如果没有指针指向前置节点, 就需要从Head 节点开始遍历, 性能非常低
2. 处于阻塞队列中的线程允许被中断
* 被中断线程仍处于链表中, 后续的锁竞争中需要把这个节点从链表里移除
* 因为需要删除节点, 所以使用双向链表更快
3. 避免线程阻塞和唤醒带来的开销

### synchronized
* 底层实现完全依赖JVM, 通过进入和退出Monitor对象来实现代码块同步和方法同步的, 包括一对monitorenter/monitorexit指令,  而monitor的实现则是依赖操作系统内部的互斥锁, 需要进行用户态到内核态的切换, 是一个重量级操作
* 实例对象的对象头里的Mark Word指针指向了moniter

### 锁升级
为了减少获得锁和释放锁来的性能开销, 引入了偏向锁、轻量级锁, 锁的状态从无锁-偏向锁-轻量级锁-重量级锁, 根据竞争激烈的程度从低到高不断升级
* 无锁, 即所有线程都能访问并修改统一资源, 但只有一个线程能够修改成功
* 偏向锁, JVM默认,  即在没有多线程竞争时, 偏向于第一个获得它的线程, 如果在接下来的执行过程中, 该锁没有被其他的线程获取, 则持有偏向锁的线程将永远不需要同步
* 轻量级锁, 即当有其他线程试图获取锁时, 偏向锁就会升级为轻量级锁, 其他线程会通过自旋形式获取锁, 而不会阻塞
* 重量级锁, 即在轻量级锁获取失败时, 会进一步升级为重量级锁, 其他线程试图获取锁时, 都会被阻塞, 只有持有锁的线程释放锁之后才会唤醒这些线程, 它是原始synchronized的实现

### 偏向锁实现
1.检查对象头中Mark Word是否为可偏向状态, 如果不是则直接升级为轻量级锁
2.如果是, 判断Mark Work中的线程ID是否指向当前线程, 如果是, 则执行同步代码块
3.如果不是, 则进行CAS操作竞争锁, 如果竞争到锁, 则将Mark Work中的线程ID设为当前线程ID, 执行同步代码块
4.如果竞争失败, 升级为轻量级锁

### 自旋锁
自旋锁,  即当线程a持有锁, 而线程b竞争锁时, 线程b不会阻塞, 而是进行自循环等待, 当线程a释放锁时, 线程b可以马上获取锁。自旋机制可以减少线程阻塞和唤醒时的上下文切换带来的开销；适用于锁竞争不是很激烈且线程占有锁的时间较短的场景, 如果锁竞争激烈或者持有锁的线程需要很长时间执行同步块, 此时自旋锁一直占用cpu做无用功, 其消耗就要远大于线程阻塞挂起操作

## 死锁
死锁是一种特定的程序状态, 不仅会发正在线程之间, 而且在资源独占的进程之间也可能出现;大多数情况下是指两个或多个线程之间互相持有对方需要的锁, 而永远处于阻塞状态
### 死锁的定位
* 通常是利用jstack等工具获取线程栈, 然后定位相互之间的依赖关系, 进而找到死锁;死锁问题一般不能在线解决,只能重启、修正程序错误; 
* JavaAPI提供了findDeadlockedThreads方法也可以进行定位
* 若某个线程进入了死循环, 导致其他线程一直等待, 进而发生死锁, 这种情况下可以查看线程 cpu 使用情况, 排查出使用 cpu 时间片最高的线程, 再打出该线程的堆栈信息, 排查代码

### 死锁的原因
* 互斥条件, 一个资源只能有一个线程占有
* 互斥条件长期持有, 也即在使用结束之前, 不会主动释放, 也不会被其他线程抢占
* 循环依赖关系, 两个或多个线程之间出现锁的链条环

### 死锁的预防：
* 尽量避免使用多个锁, 并且在需要时才持有锁
* 必须使用多个锁时, 设计好锁的获取顺序(银行家算法)
* 使用带超时的方法, 指定超时时间, 并为无法得到锁时准备退出逻辑

## 强引用、弱引用、软引用、幻象(虚)引用区别
* 强引用：即最常见普通对象引用, 只要还有强引用指向一个对象, 那么该对象就还活着, GC就不会处理它。当对象没有其他引用关系或者超出了引用作用域或者引用被赋值为null时, 对象就可以被垃圾收集了
* 软引用：相对强引用较弱, 能够使对象豁免一些垃圾收集, 只有当JVM认为内存不足时才会试图去回收这些引用指向的对象, 以保证不会耗尽内存而抛出Error. 通常用在内存敏感的缓存, 如果内存还有空闲, 就保留缓存, 否则清理掉. *可以与引用队列联合使用, 如果引用所指向对象被回收, JVM就会把该引用加入引用队列, 之后可以调用相应方法检查对象是否被回收*
* 弱引用：不能让对象豁免垃圾收集,只是提供一种访问在弱引用状态下对象的途径, 如果获取时对象还在就使用此对象, 否则重现实例化,也常用于缓存的实现.*可以与引用队列联合使用, 如果引用所指向对象被回收, JVM就会把该引用加入引用队列, 之后可以调用相应方法检查对象是否被回收*
* 幻象(虚)引用：指向的对象不能被访问, 就跟没有引用一样. 可以用来跟踪对象被GC回收的活动.*必须与引用队列联合使用, 当GC准备回收对象时, 若发现还有虚引用, 就会在回收之前把虚引用加入引用队列。*

## final、finally、finalize区别
* final可以用来修饰类、方法、变量, final不等于immutable
   * 修饰类代表此类不可被继承扩展；
   * 修饰方法代表此方法不可被重写；
   * 修饰变量代表此变量是不可修改的
* finally是保证重点代码一定被执行的机制, 通常可以使用try-finally或try-catch-finally来进行类似关闭文件流、关闭JDBC链接、unlock锁等操作
* finalize是java.lang.Object中的一个方法, 目的是保证对象被垃圾收集前完成特定资源的回收.在java9以被标记为弃用

## 代理机制
代理就是在访问实际对象时引入一定程度的间接性, 因为这种间接性, 可以增加许多操作, 例如：预处理消息、过滤消息等
### 代理分类
#### 静态代理: 由程序员自己创建代理类, 在程序运行前代理类就已生成
#### 动态代理:
1. JDK: 使用Proxy类的newProxyInstance方法来动态生成,动态生成的代理类必须与被代理类实现相同接口, 也即有相同行为
* 拿到目标对象的引用, 并且通过反射获取目标对象的所有接口
* 重新生成一个新的代理类, 实现目标类所有的接口方法；
* 把增强的逻辑代码加入到新生成的代理类源代码中。
* 动态编译代理类的源代码并生成.字节码, 也就是class文件。
* 加载并执行新生成的代理对象
2. cglib: 使用Enhancer.create方法来动态生成,动态生成一个子类,该子类继承被代理类, 并重写其中非final方法
3. 两者区别
* 前者基于反射机制, 只能对接口进行代理, 因为动态生产的代理类继承自Proxy,而JAVA不支持多继承
* 后者基于ASM框架的FastClass机制, 通过修改字节码来生成子类, 不能给final类生成代理

### 反射
1. 反射是指在程序运行状态中, 能够取得任何类的内部信息, 并能直接操作任意对象的属性及方法, 即动态获取信息以及动态调用对象方法的功能
* 使用JDBC时, 如果要创建数据库的连接, 则需要先通过反射机制加载数据库的驱动程序
* 多数框架都支持注解/XML配置, 从配置中解析出来的类是字符串, 需要利用反射机制实例化
2. 优点
* 增加程序的灵活性, 可以在运行的过程中动态对类进行修改和操作
* 提高代码的复用率, 比如动态代理, 就是用到了反射来实现
* 可以在运行时轻松获取任意一个类的方法、属性, 并且还能通过反射进行动态调用
3. 缺点
* 反射会涉及到动态类型的解析, 所以JVM无法对这些代码进行优化, 导致性能要比非反射调用更低
* 使用反射以后, 代码的可读性会下降
* 反射可以绕过一些限制访问的属性或者方法, 可能会导致破坏了代码本身的抽象性

#### Class.forName和ClassLoader区别
ClassLoader负责加载类的字节码到JVM中, Class.formName内部调用了ClassLoader, 并且可通过参数控制类的初始化

### Spring的AOP
* 我们用上边的做法去实现目标方法的增强, 实现代码的解耦, 是没有问题的, 但是还是需要自己去生成代理对象, 自己手写拦截器, 在拦截器里自己手动的去把要增强的内容和目标方法结合起来, 这用起来还是有点繁琐
* Spring的AOP就不用自己去写, 只需要在配置文件里进行配置, Spring根据不同的情况去决定是使用JDK还是cglib

## 接口和抽象类
* 接口是对行为的抽象, 是抽象方法的集合, 不能实例化, 没有非常量成员, 没有非静态方法实现, 利用接口可实现API定义和实现相分离的目的
* 抽象类除了不能实例化之外, 与普通类没有太大区别, 可以有抽象方法, 也可以没有, 抽象类大多用于抽取相关类的共同方法和成员变量以达到代码重用的目的
* 接口可以继承接口、抽象类可以实现接口、抽象类可以继承实体类

## IO流
1. java中IO流分为字节流和字符流
2. 分别由四个抽象类表示: InputStream, OutputStream, Reader, Writer

### 文件拷贝方式
1. io类库中的FileInputStream/FileOutputStream
2. nio类库中的transferTo/transferFrom(底层调用的就是sendfile)
3. 标准类库本身提供的Files.copy

###  传统IO流程
当使用输入输出流进行读写时, 先在内核态将数据从磁盘拷贝进内核缓存, 然后切换到用户态将数据从内核缓存读取到用户缓存
  <img src= "/assets/files/传统IO流程.jpg" alt="加载错误" title="传统IO流程"/>

* 用户应用进程调用read函数, 向操作系统发起IO 调用, 上下文从用户态切换为核态(切换 1)
* DMA控制器将数据从磁盘中读取到内核缓冲区
* CPU将内核缓冲区数据, 拷贝到用户应用缓冲区, 上下文从内核态转为用户态(切换 2), read 函数返回
* 用户应用进程通过write函数, 发起IO调用, 上下文从用户态转为内核态(切换3)
* CPU将用户缓冲区中的数据, 拷贝到socket缓冲区
* DMA控制器将数据从socket缓冲区拷贝到网卡设备(网卡缓冲区再把数据传输到目标服务器上), 上下文从内核态切换回用户态(切换 4), write函数返回

#### 缺陷
* 会进行多次上下文切换, IO效率低(4次上下文切换, 4次数据拷贝)
   * 使用缓存, 减少IO次数
   * 使用transferTo方法, 减少上下文切换
   * 尽量减少不必要的转换, 例如：编解码, 序列化反序列化

### 零拷贝
零拷贝并不是没有数据拷贝过程, 而是说减少用户态/内核态之间的切换次数以及CPU数据拷贝的次数
#### 虚拟内存与物理内存
1. 虚拟内存是指将硬盘的一块区域划分来作为内存
2. 物理内存是指物理内存条中的内存空间
3. CPU里有一个内存管理单元MMU, 虚拟内存不是直接送到内存总线,而是先给到MMU,由MMU把虚拟地址映射到物理地址
4. 多个虚拟内存可以指向同一个物理地址, 可以将内核空间和用户空间的虚拟地址映射到同一个物理地址, 这样就可以减少IO的数据拷贝次数
  <img src= "/assets/files/虚拟内存.jpg" alt="加载错误" title="虚拟内存"/>

#### mmap(memory map) + write(减少1次cpu拷贝)
利用内存映射(虚拟内存), 将内核的读缓冲区地址与用户缓冲区地址进行映射
  <img src= "/assets/files/zerocopy-mmap.jpg" alt="加载错误" title="zerocopy-mmap"/>

1. 用户进程通过mmap()方法向操作系统发起调用, 上下文从用户态转向内核态
2. DMA控制器把数据从硬盘中拷贝到读缓冲区, 上下文从内核态转为用户态, mmap调用返回
3. 用户进程通过write()方法发起调用, 上下文从用户态转为内核态
4. CPU将读缓冲区中数据拷贝到socket缓冲区
5. DMA控制器把数据从socket缓冲区拷贝到网卡, 上下文从内核态切换回用户态, write()返回

#### sendfile(减少2次上下文切换, 减少1次cpu拷贝)
在两个文件描述符之间传输数据, 它是在操作系统内核中操作的, 避免了数据从内核缓冲区和用户缓冲区之间的拷贝
  <img src= "/assets/files/zerocopy-sendfile.jpg" alt="加载错误" title="zerocopy-sendfile"/>

1. 用户进程发起sendfile调用, 上下文从用户态转向内核态
2. DMA控制器将数据从磁盘中拷贝到内核缓冲区
3. CPU将读缓冲区中数据拷贝到socket缓冲区
4. DMA控制器异步的将数据从socket缓冲区拷贝到网卡
5. 上下文从内核态切换回用户态, sendfile调用返回

#### 增强sendfile(减少2次上下文切换, 减少2次cpu拷贝)
引入SG-DMA技术, 即DMA拷贝加入scatter/gather操作, 可以直接从内核缓冲区将数据读取到网卡
  <img src= "/assets/files/zerocopy-enhanceSendfile.jpg" alt="加载错误" title="zerocopy-enhanceSendfile"/>

1. 用户进程发起sendfile调用, 上下文从用户态转向内核态
2. DMA控制器将数据从硬盘中拷贝到内核缓冲区
3. CPU将内核缓冲区中的文件描述符信息(包括内核缓冲区的内存地址和偏移量)发送到socket缓冲区
4. DMA控制器根据文件描述符信息, 直接将数据从内核缓冲区拷贝到网卡
5. 上下文从内核态切换回用户态, sendfile调用返回

### Netty中的零拷贝
### Kafka中的零拷贝

## IO模型
### 阻塞IO
应用程序发起IO调用，如果内核的数据还没准备好,应用程序进程就一直在阻塞等待,一直等到内核数据准备好
  <img src= "/assets/files/阻塞IO模型.jpg" alt="加载错误" title="阻塞IO模型"/>

### 非阻塞IO(NIO)
  <img src= "/assets/files/NIO模型.jpg" alt="加载错误" title="NIO模型"/>

1. 应用进程向操作系统内核, 发起recvfrom读取数据
2. 操作系统内核数据没有准备好, 返回EWOULDBLOCK错误码
3. 应用程序进程轮询调用,继续向操作系统内核发起recvfrom读取数据
4. 操作系统内核数据准备好了, 从内核缓冲区拷贝到用户空间
5. 完成调用,返回成功提示
**频轮询导致频繁的系统调用, 消耗大量CPU资源**

### IO多路复用
#### select
通过调用select 函数, 同时监控多个fd, 在select函数监控的fd中,只要有任何一个数据状态准备就绪, select 函数就会返回可读状态, 应用进程再发起recvfrom请求去读取数据
  <img src= "/assets/files/IO多路复用select.jpg" alt="加载错误" title="IO多路复用select"/>

* 监听的IO最大连接数有限, Linux一般为1024
* select函数返回后, 要通过遍历fdset, 找到就绪的描述符fd（仅知道有I/O事件发生,却不知道哪几个流,所以遍历所有流）

#### poll
解决了连接数限制问题, 其他和select一样, 但是锁着连接数的增大, 效率也会线性下降
#### epoll
采用监听事件回调机制, 而不是同时监控多个fd, 去掉了遍历fd的操作. 先通过epoll_ctl()来注册一个fd,一旦基于某个fd就绪时,内核会采用回调机制,迅速激活这个fd,当进程调用epoll_wait()时便得到通知
  <img src= "/assets/files/IO多路复用epoll.jpg" alt="加载错误" title="IO多路复用epoll"/>

#### 三者区别
<head>
	<meta charset="utf-8"> 
	<link rel="stylesheet" href="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/css/bootstrap.min.css">  
	<script src="https://cdn.staticfile.org/jquery/2.1.1/jquery.min.js"></script>
	<script src="https://cdn.staticfile.org/twitter-bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>
<table class="table table-bordered table-striped">
	<caption>IO多路复用</caption>
	<thead>
		<tr>
      <th>对比项</th>
			<th>select</th>
			<th>poll</th>
			<th>epoll</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>底层实现</td>
			<td>数组</td>
			<td>链表</td>
      <td>红黑树+双向链表</td>
		</tr>
    <tr>
			<td>获取就绪fd</td>
			<td>遍历</td>
			<td>遍历</td>
      <td>事件回调</td>
		</tr>
		<tr>
			<td>时间复杂度</td>
			<td>O(n)</td>
			<td>O(n)</td>
      <td>O(1)</td>
		</tr>
    <tr>
			<td>最大连接数</td>
			<td>1024</td>
			<td>无限制</td>
      <td>无限制</td>
		</tr>
		<tr>
			<td>fd数据拷贝</td>
			<td>每次调用select, 将fd从用户空间拷贝到内核空间</td>
			<td>每次调用poll, 将fd从用户空间拷贝到内核空间</td>
      <td>使用内存映射(mmap),不需要从用户空间频繁拷贝到内核空间</td>
		</tr>
	</tbody>
</table>

### 信号驱动
信号驱动IO不再以主动询问的去确认数据是否就绪, 而是向内核发送一个信号, 然后应用进程可以去做别的事, 不用阻塞. 当内核数据准备好后, 再通过信号通知应用进程, 应用进程收到信号之后立即调用recvfrom去读取数据
  <img src= "/assets/files/信号驱动IO.jpg" alt="加载错误" title="信号驱动IO"/>

### 异步非阻塞IO(AIO)
应用进程发出系统调用后,立即返回. 返回的不是处理结果, 而是表示提交成功类似的意思. 等内核数据准
备好, 将数据拷贝到用户进程缓冲区,发送信号通知应用进程
  <img src= "/assets/files/AIO.jpg" alt="加载错误" title="AIO"/>

### Netty-Reactor
在NIO多路复用的基础上提出的一个高性能IO设计模式. 核心思想是把响应IO事件和业务处理进行分离,通过一个或多个线程来处理IO事件
### Netty
基于Java NIO封装的高性能的网络通信框架
1. 提供了比NIO更简单易用的API, 我们可以利用这些封装好的API快速开发自己的网络通信程序
2. 在NIO的基础上还做了很多优化, 比如零拷贝机制、内存池管理等, 总体运行性能会比原生的NIO更高
3. 支持多种通信协议, 如HTTP、WebSocket等, 并且针对数据通信的拆包、黏包问题提供了内置解决方案
4. 可以使用很少的代码实现Reactor多线程模型以及主从线程模型
5. Zookeeper、Dubbo、RocketMQ等都用到Netty, 经历了大型项目的使用和考验更加成熟稳定

#### Netty核心组件
1. 网络通信层
* Bootstrap: 负责客户端启动并用来链接远程Netty Server
* ServerBootStrap: 负责服务端监听, 用来监听指定端口
* Channel: 相当于完成网络通信的载体
2. 事件调度层
* EventLoopGroup: 本质上是一个线程池, 主要负责接收I/O请求, 并分配线程执行处理请求
* EventLoop: 相当于线程池中的线程
3. 服务编排层
* ChannelPipeline: 负责将多个ChannelHandler链接在一起
* ChannelHandler: 针对I/O的数据处理器. 数据接收后,通过指定的Handler进行处理
* ChannelHandlerContext: 用来保存ChannelHandler的上下文信息

#### Pipeline工作原理
Netty中的Pipeline本质上是一个双向链表,它采用了责任链模式. 在Netty中每个Channel都有且仅有一个ChannelPipeline与之对应. 而ChannelPipeline中又维护了一个由ChannelHandlerContext组成的双向链表. 链表的头叫HeadContext,链表的尾叫TailContext.每个ChannelHandlerContext中又关联着一个ChannelHandler
1. Pipeline初始化: 通过调用Channel的handler()方法,然后在handler()方法中传入ChannelInitializer的对象, 通过SocketChannel构建出一个新的Pipeline对象. 每次调用addLast()方法, 都会在Pipelie的末端插入一个ChannelHandlerContext
2. 


### 线程模型
1. 单线程: 
* 由同一个线程来负责处理IO事件以及业务逻辑, handler的执行过程是串行
* 如果有任何一个handler处理线程阻塞, 就会影响整个服务的吞吐量
2. 多线程: 
* 把处理IO就绪事件的线程和处理Handler业务逻辑的线程进行分离. 
* 每个Handler由一个独立线程来处理. 即便是存在Handler线程阻塞问题,也不会对IO线程造成影响
* 但所有的IO操作都是由一个Reactor来完成的, 而且Reactor运行在单个线程里面, 并发较高的场景Reactor就成了性能瓶颈
3. 主从多线程
* 单个Reactor拆分成了Main Reactor和多个SubReactor, Main Reactor负责接收客户端的链接, 然后把接收到的连接请求随机分配到多个subReactor里面

#### 三个组件
1. Reactor: 把I/O事件分发给对应的Handler
2. Acceptor: 处理客户端连接请求
3. Handlers: 执行非阻塞读/写,也就是针对收到的消息进行业务处理

## 设计模式
1. 创建型: 即创建对象过程中的各种问题和解决方案的总结, 例如：工厂模式、单例模式、构建器模式
* 构建器模式： 例如, 创建HttpRequest的过程, 以及其他创建复杂对象的场景(builder().build())
2. 结构型: 即对软件设计结构的总结, 例如： 适配器模式、装饰者模式、代理模式
* 装饰者模式： 例如, InputStream, 它本身是个抽象类, java基础类库提供了FileInputStream、BufferedInputStream等子类, 对其从不同角度进行了功能扩展
3. 行为型: 即对类或对象之间交互、职责划分的总结, 例如： 策略、解释器、观察者、模板方法模式

### Spring框架中用到的设计模式
* 工厂模式: BeanFactory类, 其getBean()方法用来创建对象, ApplicationContext继承自BeanFactory
* 单例模式: spring中的bean默认都是单例
* 装饰器模式: spring中以Wrapper命名的基本都是, 比如BeanWrapper用来访问bean的属性方法
* 策略模式: spring中bean的实例化用到, 比如原生对象和代理对象实例化逻辑不一样, simpleInstantiationStrategy时spring默认的实例化策略
* 适配器模式: spring中以Adapter命名的基本都是, 比如MVC模块中的HandlerAdapter
* 代理模式: springAOP
* 模板方法模式: spring中以Template命名的基本都是, 比如RestTemplate,JdbcTemplate
* 观察者模式: spring中以Listener命名的基本都是, 比如ApplicationListener
* 桥接模式: Java SPI(用来被第三方实现或扩展的API, 如数据库驱动, JDK只定义了Driver接口, 没有提供实现, 由第三方数据库厂商自己来完成)

### 抽象工厂和工厂方法
1. 工厂方法模式针对的是一个产品的等级结构, 只有一个抽象产品类, 可以派生出多个具体产品类
2. 抽象工厂模式针对的是多个产品的等级结构, 有多个抽象产品类, 每个抽象产品类可以派生出多个具体产品类

### 单例模式
#### 单例被破坏的五个场景
1. 多线程破坏单例, 即创建出多个对象. 只出现在懒汉模式中. 使用双检锁或使用静态内部类
2. 指令重排破坏单例, 导致先给对象引用赋值内存地址, 再初始化对象. 此时线程1初始化完成前, 线程2已经判断instance不为空, 也即直接返回instace对象, 但调用时就会报错
* 正常顺序: memory = allocate()(分配对象的内存空间指令) -> ctorInstance(memory)(初始化对象) -> instance = memory(将已分配存地址赋值给对象引用)
* 指令重拍后: memory = allocate() -> instance = memory -> ctorInstance(memory)
* 解决办法: 使用volatile定义instance
3. 克隆破坏单例
Java的所有类都继承自Object, 而Object又有clone()方法, 深clone导致每次创建新对象, 若再单例对象中调用此方法就破坏了单例. 可以重写clone()方法, 将其改为浅clone
4. 反序列化破坏单例
反系列化时对象被重新创建. 在反序列化中会调用readResolve()方法, 重写此方法将返回值设置为已存在的单例对象
5. 反射破坏单例
反射机制是可以拿到对象的私有的构造方法, 也即反射可以任意调用私有构造方法创建单例对象. 可以在狗崽方法上加入条件判断解决, 或者使用枚举式单例(反射不能访问枚举)

#### 单例模式适用场景
1. 如果某个共享资源, 使用频次非常高, 而且不可替代性也很强,就应该被设计为单例. 比如Spring中的IoC容器、JDK的Runtime
2. 要经常被赋值传递的对象, Vo、Pojo等就不适合设置为单例

#### 双检锁
```java
if (null == instance) {
  synchronized(LazyDoubleCheckLockSingleton.class) {
    if (null == instance) {
    instance = new LazyDoubleCheckLockSingleton();
    }
  }
}
```
上述代码中加锁以保证线程安全, 检查以保证只有1个实例
1. 为什么需要双重检查
* 假设去掉外层检查, 在线程1创建好对象后, 其他线程每次都会阻塞. 加上后只有第一次出现并发时会阻塞, 提高性能
* 假设去掉内层检查, 两个线程同时访问getInstance()方法时,同时满足条件,两个线程会按顺序执行synchronized代码块中的逻辑. 假设线程1先执行创建对象, 线程2获得锁后依然会创建对象并覆盖之前创建的对象. 加上后就能避免重复创建

### 策略模式


## 内存溢出和内存泄漏
### 内存溢出
创建的对象大于内存剩余空间
### 内存泄漏
在业务代码执行过程中, 有些对象应该被回收, 但又有其他对象的引用引用这个对象, 导致GC不能自动回收. 当垃圾对象越堆越多,可用内存越来越少, 若可用内存无法存放新的垃圾对象, 最终导致内存泄漏. 内存泄漏最终会导致内存溢出

 ## 项目访问量激增
 1. 预估量
 2. 全链路压测. 由于是微服务,并且用到了比较多的组件, 比如MQ、redis、mysql等. 所以整个链路会比较长, 压测不能局限在单个接口, 需要进行一个全链路压测. 找到链路瓶颈、哪个服务、哪个接口、哪个组件服务拖慢了性能
* 代码是否有低级错误, 有不应该的重复请求, 查询语句是否有走到索引, 有哪些数据可以做预热
* 如果代码或用法没有问题, 那么就是机器瓶颈, 对应的去加机器, 对于热点接口以及服务进行扩容
4. 降级. 对于复杂但不是主流的场景,进行静态降级或者暂时不可用