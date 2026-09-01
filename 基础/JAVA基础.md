#### JDK8新特性

1. Lambda 表达式
2. Stream API
3. 新的日期时间 API（java.time 包）

   核心类：LocalDate（日期）、LocalTime（时间）、LocalDateTime（日期+时间）、ZonedDateTime（带时区）、Instant（时间戳
4. 接口默认方法和静态方法
5. Optional 类
6. 方法引用（::）
7. 新的 ConcurrentHashMap 改进
	引入 **CAS + synchronized** 替代 JDK 7 的分段锁，锁粒度更细（Node级别），并发度更高
8. 永久代移除

#### JDK17新特性

1、密封类 (Sealed Classes)。这个特性让你可以精确控制一个类或接口允许哪些子类或实现类，从而更好地守护代码的层次结构，让设计意图更清晰。
2、将模式匹配扩展到了 switch 语句/表达式中。
3、Record 类 



#### 死锁

![[Pasted image 20260705112259.png]]

##### 如何排查死锁:           jstack -l 12345 | grep -A 20 "deadlock"

#### ThreadLocal

ThreadLocal 的 Entry 继承 WeakReference（弱引用），核心目的只有一个：防止key造成内存泄漏。value 的回则仍依赖于 ThreadLocalMap 的主动清理机制。

#### HashMap头插法导致死循环

1. 核心背景
头插法：扩容迁移时，新链表的顺序是原链表的逆序。

并发条件：两个线程（Thread1 和 Thread2）同时触发扩容，并都拿到了指向原链表的头节点引用。

2. 死循环的“三步走”过程
假设原链表是：A -> B -> null（A是头节点）。

第一步（线程2被挂起）：线程2刚拿到数组和A、B的引用，还没开始迁移，就被CPU切换挂起了。

第二步（线程1完成扩容）：线程1用头插法完成迁移。迁移后，新链表顺序变为 B -> A -> null（注意顺序反转了）。

第三步（线程2恢复执行）：这时问题来了——线程2不知道链表已经被线程1反转了，它依然拿着旧引用，认为A后面是B。于是线程2开始迁移：

把A移到新数组（A变成新头节点）。

接着处理B，把B移到A前面（B变成头节点，A在B后面）。

处理完B后，按旧逻辑去找“B的下一个节点”，结果是A（因为被线程1改成了B->A）。

于是线程2又把A移过来，但A已经在链表里了，这就形成了 B -> A -> B -> A... 的环形链表。

3. 最终结果
当后续 get() 操作遍历到这个环时，就会永远循环下去，导致CPU飙升到100%，这就是经典的HashMap死循环。


#### HashMap
HashMap的数据结构： 
底层使用hash表数据结构，即数组和链表或红黑树
1. 当我们往HashMap中put元素时，利用key的hashCode重新hash计算出当前对 象的元素在数组中的下标 
2. 存储时，如果出现hash值相同的key，此时有两种情况。 a. 如果key相同，则覆盖原始值； b. 如果key不同（出现冲突），则将当前的key-value放入链表或红黑树中
3. 获取时，直接找到hash值对应的下标，在进一步判断key是否相同，从而找到 对应值。



**扩容机制**：链表的长度大于 8  **且** 数组长度大于 64 转换为红黑树


#### ArrayList

1. 底层数据结构。ArrayList底层是用动态的数组实现的
2. 初始容量 ArrayList初始容量为0，当第一次添加数据的时候才会初始化容量为10 
3. 扩容逻辑 ArrayList在进行扩容的时候是原来容量的1.5倍，每次扩容都需要拷贝数组 
4. 添加逻辑 确保数组已使用长度（size）加1之后足够存下下一个数据 计算数组的容量，如果当前数组已使用长度+1后的大于当前的数组长度， 则调用grow方法扩容（原来的1.5倍） 确保新增的数据有地方存储之后，则将新元素添加到位于size的位置上。 返回添加成功布尔值。


# 排查类

### 排查内存溢出
JVM启动命令带上以下两个参数
-XX:+HeapDumpOnOutOfMemoryError  
-XX:HeapDumpPath=D:\mycode\my-spring-ai\src\main\resources
IDEA自带的Profiler打开生成的 .hprof文件
![[Pasted image 20260720151418.png]]
或用JProfiler打开快照
![[Pasted image 20260720151346.png]]
### 排查CPU飙升

![[Pasted image 20260720153153.png]]
### 排查死锁
#### JVM层死锁
jps -l 找出pid
jstack -l 11832 
![[Pasted image 20260720142631.png]]
#### 数据库层死锁

SHOW ENGINE INNODB STATUS

![[Pasted image 20260720143902.png]]


### 排查接口慢

#### Arthas

jad
trace
jad → mc → retransform

#### Maven的生命周期

1. Clean Lifecycle（清理生命周期）
2. Default Lifecycle（默认生命周期/构建生命周期）核心
	  validate → compile → test → package → verify → install → deploy
3. Site Lifecycle
话术:

Maven 有三套生命周期，最核心的是 Default Lifecycle，它的关键阶段是 `validate → compile → test → package → verify → install → deploy`，执行后面的阶段会自动执行前面的所有阶段。每个阶段本身不干活，真正干活的是绑定在上面的插件，比如 `compile` 阶段绑定了 `maven-compiler-plugin`。实际开发中最常用的命令是 `mvn clean package` 和 `mvn clean install`——`clean` 负责清理上一次构建的残留文件，确保构建干净；`install` 会把打好的包安装到本地仓库，供其他模块引用。`Generate Sources` 是 `validate` 和 `compile` 之间的一个阶段，用于在编译前生成额外的源代码，比如根据 `.proto` 文件生成 Java 类。

#### 仲裁机制

Maven 有两套依赖仲裁规则，按顺序优先级：

1. **路径最短优先**：依赖树中路径最短的版本胜出。
2. **声明顺序优先**：路径长度相同时，在 `pom.xml` 中先声明的优先。

#### 项目中怎么解决依赖冲突的

1. 使用 exclusions 排除（最常用）
2. 在 dependencyManagement 中统一管理版本（推荐）
3. 直接在 `<dependencies>` 中显式声明
4. 升级/降级依赖版本


### 对象的创建过程

1. Class Loading（类加载）

2. Class Linking（链接）

	Verification（验证）

	Preparation（准备：静态变量赋默认值）

	Resolution（解析）

3. Class Initializing（初始化：静态变量赋初始值，执行 static {} 块）

4. 申请对象内存（在堆里划出一块空间）

5. 成员变量赋默认值（这是JVM自动做的，在内存清零时完成）

6. 设置对象头（Mark Word + 类型指针）

7. 调用构造方法 `<init>`（成员变量赋初始值，执行构造器里的代码）

![[Pasted image 20260817165850.png]]

##### 给对象的属性赋值的操作
1. 属性的默认初始化
2. 显示初始化
3. 代码块中初始化
4. 构造器中初始化

#### ConcurrentHashMap锁机制

|特性|JDK 1.7|JDK 1.8|
|---|---|---|
|锁机制|**Segment 分段锁**（ReentrantLock）|**桶节点锁**（synchronized + CAS）|
|锁粒度|粗（一个 Segment 锁住多个桶）|细（只锁住一个桶）|
|最大并发度|默认 16，固定|数组长度，可动态扩容|
|读操作|无锁（volatile 保证可见性）|无锁（volatile 保证可见性）|
|size 计算|全局锁，阻塞|LongAdder 累加，非阻塞|
|红黑树|不支持|支持（链表长度 ≥8 时树化）|







### Cookie 和 Session

|维度|Cookie|Session|
|---|---|---|
|**存储位置**|**客户端**（你的浏览器硬盘/内存）|**服务器端**（服务器内存/Redis/数据库）|
|**存储大小**|很小，一般 **4KB** 左右|较大，理论上无限制（取决于服务器内存）|
|**安全性**|**低**。数据明文可见，容易被篡改或窃取（XSS攻击）|**高**。数据只在服务器，用户无法看到和修改|
|**生命周期**|可长期存在（设置Expires），甚至几年|通常较短（默认30分钟），关闭浏览器或超时即失效|
|**主要作用**|身份凭证（存Session ID）、记录用户偏好（如字体大小）|存储敏感用户数据（如登录状态、购物车内容）|


Cookie是浏览器（客户端）本地存东西的小盒子，主要用来记住“你是谁”。(会员卡)

Session是服务器内存里存东西的记录本，主要用来记住“你干了什么”。(后台档案)
