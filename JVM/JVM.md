![[60678c30454422b4ac001aa37f0f8418.png]]

### JVM由哪几部分组成
1、ClassLoader（类加载器）
2、Runtime Data Area（运行时数据区，内存分区）
3、Execution Engine（执行引擎）
4、Native Method Library（本地库接口）

![[429385d189bf7fd5d26f8b6a3d748cc5.png]]

---

### 栈

1. 局部变量表
2. 操作数栈
3. 动态链接
4. 方法返回地址

┌──────────────────────────────────────────────────────┐
│                   栈帧 (Stack Frame)                
│                  (每个方法一个)                     
├──────────────────────────────────────────────────────┤
│                                                   
│  ┌──────────────────────────────────────────────┐    │
│  │  1. 局部变量表 (Local Variable Table)     
│  │     ├─ 方法参数                          
│  │     ├─ 局部变量                         
│  │     └─ 槽位 (Slot) 索引                   
│  └──────────────────────────────────────────────┘   │
│                                                     
│  ┌──────────────────────────────────────────────┐   │
│  │  2. 操作数栈 (Operand Stack)              
│  │     ├─ 运算中间结果                       
│  │     ├─ 方法调用参数                        
│  │     └─ 返回值                             
│  └──────────────────────────────────────────────┘   │
│                                                     
│  ┌──────────────────────────────────────────────┐   │
│  │  3. 动态链接 (Dynamic Linking)         
│  │     └─ 指向运行时常量池的方法引用            
│  └──────────────────────────────────────────────┘   │
│                                                     
│  ┌──────────────────────────────────────────────┐   │
│  │  4. 方法返回地址 (Return Address)            
│  │     ├─ 正常返回：调用者的PC指针             
│  │     └─ 异常返回：异常处理器地址              
│  └──────────────────────────────────────────────┘   │
│                                                     
│  ┌──────────────────────────────────────────────┐   │
│  │  5. 附加信息 (可选)                          
│  └──────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘

### 堆

物理名字（固定）：JVM启动时，两块区域就固定命名为 S0 和 S1，这个名字永远不会变。

逻辑角色（动态）：From 和 To 是逻辑角色，每次Minor GC（YGC）后，这两个角色会发生调换。

### 方法区(类是懒加载)

1. 类信息
2. 运行时常量池


### 类加载机制###

#### 一、加载Load
1、通过一个类的全限定明获取定义此类的二进制字节流。
2、将这个字节流所代表的静态存储结构转化为方法去的运行时数据结构。
2、在内存中生成一个代表这个类的java.lang.Class对象  
#### 二、链接Link
###### 1.验证(Verify)
目的在于确保Class文件的字节流中包含信息符合当前虚拟机要求，确保被加载类的正确性，不会危害虚拟机自身安全

主要包括四种验证，文件格式验证，元数据验证，字节码验证，符号引用验证。

##### 2.准备(Prepare)
为类变量分配内存并且设置该类变量的默认初始值，即零值。
这里不包含用final修饰的static，因为final在编译的时候就会分配了，准备阶段会显式初始化；
这里不会为实例变量分配初始化，类变量会分配在方法区中，而实例变量会随对象一起分配到Java堆中。
##### 2.解析(Resolve)
将常量池内的符号引用转换为直接引用的过程。
事实上，解析操作往往会伴随JVM在执行完初始化之后再执行。
符号引用就是一组符号来描述引用的目标。符号引用的字面量形式明确定义在《java虚拟机规范》Class文件格式中。直接引用就是直接指向目标的指针、相对偏移量或一个间接定位到目标的句柄。

#### 三、初始化
执行类构造器 `<clinit>()`
## 垃圾回收

### 对象分配过程

1、new的对象先放伊甸园区。此区有大小限制。
2、当伊甸园的空间填满时，程序又要创建对象，JVM的垃圾回收器将对伊甸园区进行垃圾回收(Minor GC),将伊甸园区中的不再被其他对象引用的对象进行销毁。再加载新的对象放到伊甸园区
3、然后将伊甸园中的剩余对象移动到幸存者0区
4、如果再次触发垃圾回收，此时上次幸存下来的放到幸存者0区的，如果没有回收，就会放到幸存者1区
5、如果再次经历垃圾回收，此时会重新放回幸存者0区，接着再去幸存者1区
6、啥时候去养老区呢?可以设置次数。默认是15次。
可以设置参数：`-XX:MaxTenuringThreshold=<N>`进行设置

总结:针对幸存者s0，s1区的总结:复制之后有交换，谁空谁是to，s0 s1 是物理分区,from to 是逻辑分区会对调


吞吐量（越高越好）
100% │
     │ Parallel GC  ██████████████████████░░░░  98%
 95% │ G1 GC        █████████████████░░░░░░░░  93%
 90% │ ZGC          ████████████████░░░░░░░░░  90%
     │
     └────────────────────────────────────→
       吞吐量        低延迟程度
   
Parallel GC: 最高吞吐量，最差延迟
ZGC:         最低吞吐量，最好延迟
G1:          两者之间


### 内存泄漏的8种情况
 
 1. 静态集合类
 2. 单例模式
 3. 内部类持有外部类
 4. 各种连接如数据库连接网络连接IO连接
 5. 变量不合理的作用域
 6. 改变哈希值
 7. 缓存泄漏
 8. 监听器和回调

### 哪些对象可以作为GCRoot

1、虚拟机栈引用。方法中的局部变量
2、静态变量引用。类的静态属性。
3、常量池引用。字符串常量、Class对象
4、JNI引用。Native方法中的全局/局部变量
5、活跃线程。Thread对象本身
6、同步监视器。被synchronized持有的对象
7、JVM内部引用。系统类加载器、异常对象等



口诀:**栈中变量静态量，常量JNI活跃线，同步锁和类加载，都是Root要记全。**

技巧:由于Root采用栈方式存放变量和指针，所以如果一个指针，他保存了堆内存里面的对象，但是自己又不存放在堆内存里面，那它就是一个Root。



Minor GC:触发条件是Eden 区满
Full GC:触发条件是老年代满


### 对象的内存布局

对象头:

实例数据:

对齐填充:

### 创建对象的方式
1、new
2、反射
3、clone
4、反系列化

### 创建对象的步骤

1、检查加载。
2、分配内存
3、初始化零值
4、设置对象头
5、执行`<init>`方法

从JVM底层看，对象创建分为五步：第一步是类加载检查，确保类已加载初始化；第二步是分配内存，根据堆是否规整采用指针碰撞或空闲列表，配合TLAB(Thread-Local Allocation Buffer)优化并发分配；第三步是零值初始化，把内存空间清零，保证字段默认值；第四步是设置对象头，写入哈希码、分代年龄、锁状态等元数据；第五步是执行`<init>`方法，执行构造器完成对象初始化，最后返回引用。另外，如果开启了逃逸分析，部分对象会直接在栈上分配，不进堆



![[Pasted image 20260711085221.png]]
#### 为什么CMS被废弃

1. 难以解决浮动垃圾。CMS在执行并发标记和清理时，应用线程仍在运行，会产生新的垃圾
2. 内存碎片化。CMS使用“标记-清除”算法，回收后不整理内存。长时间运行后，老年代会充满碎片，可能导致无法为大对象分配连续空间，进而不得不触发代价高昂的Full GC。
![[Pasted image 20260710074115.png]]

![[Pasted image 20260710074327.png]]


![[Pasted image 20260710140855.png]]
### G1回收过程

目的:在延迟可控的情况下尽可能获得性能高的吞吐量
##### 阶段一：年轻代GC
1.第一阶段,扫描根。
2.第二阶段,处理dirty card queue更新Rset
3.第三阶段:处理RSet
4.复制对象
5.处理引用

CardTable
RSet
CSet

一个Region通常包含几千个Card（因为Region大小是1MB~32MB，每个Card是512字节）

| 概念             | 粒度级别              | 说明                                                |
| -------------- | ----------------- | ------------------------------------------------- |
| **Card Table** | **Card级别**（512字节） | 全局的、细粒度的脏标记数组。                                    |
| **RSet**       | **Region级别**      | 每个Region拥有一个独立的RSet，记录“谁引用了本Region”。              |
| **CSet**       | **Region级别**      | 是一个Region的集合（Set of Regions），由G1算法选出的待回收Region名单。 |
 
##### 阶段二：并发标记过程

1. 初始标记（STW，很短）
2. 并发标记（与应用线程一起跑）
3. 最终标记（STW，很短）
4. 筛选回收（STW，清理+拷贝）

![[Pasted image 20260711114918.png]]

##### 阶段三：混合回收

不单回收Old区，而是同时回收一部分Eden、Survivor和Old区的Region

![[Pasted image 20260826092700.png]]

**总结**：G1把堆拆成Region，通过并发标记统计每个Region的垃圾占比，然后在可预测的停顿时间内，优先回收垃圾最多的Region，并把存活对象拷贝到干净Region中，以此消除碎片并控制延迟。

#### ZGC
ZGC收集器是一款基于Region内存布局的，不设分代的，使用了读屏障、染色指针和内存多重映射等技术实现可并发的标记-压缩算法的，以低延迟为首要目标的一款垃圾回收器。


#### 为什么要改永久代为原空间

1. 为永久代设置空间大小很难确定
2. 对永久代进行调优是很困难的


#### 方法区 栈 堆 关系

![[Pasted image 20260720220043.png]]



#### 三色标记算法

三色标记算法（Tri-color Marking） 是JVM垃圾回收器（如CMS、G1、ZGC）在并发标记阶段，用来追踪存活对象的核心算法。它的本质是将对象分为三种颜色，通过颜色变化来区分“存活”“待扫描”“可回收”，从而实现在业务线程不暂停的情况下，安全地标记出所有存活对象。

|颜色|含义|状态描述|
|---|---|---|
|**白色（White）**|**未访问 / 可回收**|尚未被垃圾回收器触及的对象。标记结束后仍为白色的对象，将被判定为**垃圾**并回收。|
|**灰色（Gray）**|**正在访问中**|对象本身已被标记为存活，但**它的所有引用字段还没有被扫描完毕**。它处于“待处理”队列中，是算法推进的动力。|
|**黑色（Black）**|**已访问完成**|对象已被标记为存活，并且**它的所有引用字段都已扫描完毕**。它不会再被访问，是算法完成后的安全状态。|

### CPU飙升/Full GC频繁 /OOM

| 维度       | CPU飙升                  | Full GC频繁               | OOM                        |
| -------- | ---------------------- | ----------------------- | -------------------------- |
| **现象**   | 应用响应慢，top -H看到CPU占用高   | GC日志显示FGC次数暴增，STW时间长    | 抛出 `OutOfMemoryError`，应用崩溃 |
| **常见原因** | 死循环、频繁GC、锁竞争、大量计算      | 老年代/元空间不足、内存泄漏、CMS并发失败  | 堆内存/元空间/直接内存耗尽             |
| **排查工具** | `top -H` + `jstack`    | `jstat` + GC日志 + `jmap` | `jmap` + MAT + `jcmd`      |
| **排查入口** | 找CPU最高的线程 → 看堆栈 → 定位代码 | 看GC频率 → dump堆 → 查引用链    | dump堆 → 找最大对象 → 定位泄漏源头     |
| **解决手段** | 修复死循环、优化算法、减少GC        | 调整堆大小、换G1、修复内存泄漏        | 增加堆内存、修复泄漏、减少缓存            |

三者本质都是内存/计算资源问题，但排查路径不同：CPU飙升看 jstack 定位热点代码；Full GC频繁看 jstat + GC日志分析内存分配；OOM看 jmap dump + MAT定位泄漏源头。实际线上经常是连锁反应——比如内存泄漏导致Full GC频繁，GC线程又把CPU跑满。

### JVM调优常用参数

#### 1.内存大小调整
|                            |                                                                                                                                                                                                                                                                                |                                                                                                                                                                                                                              |     |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| **`-Xms` / `-Xmx`**        | 分别设置堆内存的**初始值**和**最大值**[](https://developer.huaweicloud.cn/tags/200754/post_1)[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)[](https://github.com/web3-private/2025JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md#1#1)。               | **生产环境建议将两者设为相同**，避免JVM运行时频繁伸缩堆大小，带来不必要的性能开销[](https://cloud.tencent.cn/developer/article/2021088?from=15425#1)[](https://github.com/web3-private/2025JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md#1#1)。        |     |
| **`-Xmn`**                 | 设置**新生代（Young Generation）** 的大小[](https://developer.huaweicloud.cn/tags/200754/post_1)[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)[](https://github.com/web3-private/2025JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md#1#1)。       | 新生代大小对GC影响很大。调大新生代可减少对象晋升老年代，但会压缩老年代空间。官方建议可作为堆的 **3/8** 左右作为参考[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)。                                                                                 |     |
| **`-XX:NewRatio`**         | 设置**老年代与新生代**的内存大小比例[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)[](https://github.com/web3-private/2025JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md#1#1)[](https://bbs.huaweicloud.com/blogs/986dc76b519547608f68f3501197c4a0)。   | 例如 `-XX:NewRatio=2` 表示老年代:新生代 = 2:1，即新生代占堆的1/3[](https://github.com/web3-private/2025JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md#1#1)[](https://docs.oracle.com/en/java/javase/17/gctuning/ergonomics.html#1)。 |     |
| **`-XX:SurvivorRatio`**    | 设置新生代中**Eden区与Survivor区**的比例[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)[](https://cloud.tencent.cn/developer/article/2021088?from=15425#1)[](https://cloud.tencent.cn/developer/article/2461395?policyId=1003#1)。                             | 例如 `-XX:SurvivorRatio=8` 表示 `Eden:S0:S1 = 8:1:1`，这是JDK的默认值[](https://cloud.tencent.cn/developer/article/2021088?from=15425#1)[](https://cloud.tencent.cn/developer/article/2461395?policyId=1003#1)。                         |     |
| **`-Xss`**                 | 设置**每个线程的栈空间**大小[](https://developer.huaweicloud.cn/tags/200754/post_1)[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)[](https://cloud.tencent.cn/developer/article/2021088?from=15425#1)。                                                        | JDK 5+ 默认1M。如果线程数很多，可以适当减小该值，以支持创建更多线程[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)。                                                                                                          |     |
| **`-XX:MaxMetaspaceSize`** | 设置**元空间（Metaspace）** 的最大大小（JDK 8+）[](https://cloud.tencent.com.cn/developer/article/2481951)[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)[](https://github.com/web3-private/2025JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md#1#1)。 | 元空间使用本地内存，若不设置上限，可能会耗尽系统内存。建议设置一个合理上限[](https://cloud.tencent.cn/developer/article/2021088?from=15425#1)[](https://github.com/web3-private/2025JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md#1#1)。               |     |

#### 2.垃圾回收器选择

|   |   |   |
|---|---|---|
|**`-XX:+UseG1GC`**|启用 **G1（Garbage-First）** 垃圾回收器[](https://cloud.tencent.com.cn/developer/article/2497446?policyId=1003#1)[](https://manpages.debian.org/testing/openjdk-21-jre-headless/java.1.en.html#7)[](https://github.com/web3-private/2025JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md#1#1)。|**服务器端应用**，尤其大内存（>6GB）、需要**可控、可预测的GC停顿时间**的场景，是很多大厂的线上选择[](https://cloud.tencent.com.cn/developer/article/2497446?policyId=1003#1)[](https://manpages.debian.org/testing/openjdk-21-jre-headless/java.1.en.html#7)。|
|**`-XX:+UseParallelGC`**|启用 **Parallel GC**（并行收集器）[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)[](https://manpages.debian.org/testing/openjdk-21-jre-headless/java.1.en.html#7)[](https://bbs.huaweicloud.com/blogs/986dc76b519547608f68f3501197c4a0)。|对**吞吐量**要求高、停顿时间要求不高的后台任务或计算密集型应用[](https://manpages.debian.org/testing/openjdk-21-jre-headless/java.1.en.html#7)[](https://bbs.huaweicloud.com/blogs/986dc76b519547608f68f3501197c4a0)。|
|**`-XX:+UseZGC`**|启用 **ZGC** 垃圾回收器（JDK 11+）[](https://manpages.debian.org/testing/openjdk-21-jre-headless/java.1.en.html#7)[](https://github.com/web3-private/2025JavaGuide/blob/main/docs/java/jvm/jvm-parameters-intro.md#1#1)。|追求**极低停顿时间**（毫秒级）的大型应用，堆大小从8MB到16TB均支持[](https://manpages.debian.org/testing/openjdk-21-jre-headless/java.1.en.html#7)。|

#### 3.问题留痕

|                                       |                                                                                                                                                                                                                 |                                                                                                                                                                                                               |
| ------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`-XX:+HeapDumpOnOutOfMemoryError`** | 当发生 **OOM（OutOfMemoryError）** 时，自动**导出堆内存快照（Heap Dump）**[](https://cloud.tencent.com.cn/developer/article/2497446?policyId=1003#1)[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)。 | **生产环境必加参数**！配合`-XX:HeapDumpPath`指定存储路径，是事后分析内存泄漏的黄金数据[](https://cloud.tencent.com.cn/developer/article/2497446?policyId=1003#1)[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)。 |
|                                       |                                                                                                                                                                                                                 |                                                                                                                                                                                                               |

4.行为目标调优

|                                |                                                                                                                                                                                                                                                 |                                                                                                                                            |
| ------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **`-XX:MaxGCPauseMillis`**     | 设置**期望的最大GC停顿时间**目标（毫秒）[](https://docs.oracle.com/en/java/javase/17/gctuning/ergonomics.html#1)。                                                                                                                                                | 这是一个“软”目标，JVM会尽力达成，例如`-XX:MaxGCPauseMillis=200`。设置过小可能导致频繁GC，反而不利[](https://docs.oracle.com/en/java/javase/17/gctuning/ergonomics.html#1)。 |
| **`-XX:GCTimeRatio`**          | 设置**吞吐量目标**，即GC时间与应用时间的比例[](https://docs.oracle.com/en/java/javase/17/gctuning/ergonomics.html#1)。                                                                                                                                              | 例如`-XX:GCTimeRatio=19`表示GC时间占比目标为`1/(1+19)=5%`[](https://docs.oracle.com/en/java/javase/17/gctuning/ergonomics.html#1)。                    |
| **`-XX:MaxTenuringThreshold`** | 设置对象在新生代**经历多少次Minor GC后晋升到老年代**的年龄阈值[](https://cloud.tencent.com.cn/developer/article/2481951)[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)[](https://cloud.tencent.cn/developer/article/2021088?from=15425#1)。 | 适当增大阈值，能让存活对象在新生代呆更久，避免过早晋升到老年代[](https://help.aliyun.com/zh/sae/jvm-parameter-configuration-recommend#1#1)。                               |
