
### 执行流程

①读取MyBatis配置文件：mybatis-config.xml加载运行环境和映射文件 
②构造会话工厂SqlSessionFactory，一个项目只需要一个，单例的，一般由 spring进行管理 
③会话工厂创建SqlSession对象，这里面就含了执行SQL语句的所有方法 
④操作数据库的接口，Executor执行器，同时负责查询缓存的维护 
⑤Executor接口的执行方法中有一个MappedStatement类型的参数，封装了 映射信息 
⑥输入参数映射 
⑦输出结果映射