
#### JDK17新特性

1、密封类 (Sealed Classes)。这个特性让你可以精确控制一个类或接口允许哪些子类或实现类，从而更好地守护代码的层次结构，让设计意图更清晰。
2、将模式匹配扩展到了 switch 语句/表达式中。
3、Record 类 



#### 死锁

![[Pasted image 20260705112259.png]]

##### 如何排查死锁:           jstack -l 12345 | grep -A 20 "deadlock"