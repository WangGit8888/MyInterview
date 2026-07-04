###RabbitMQ里消息丢失的三种情况
1、生产者侧 :消息根本没发到MQ,网络故障、连接异常关闭、消息发错交换机等。
2、服务端侧:MQ宕机导致内存中消息丢失、未开启持久化，消息只存在于内存中，服务器重启后丢失。
3、消费者侧:消费处理失败但消息已被确认。开启了自动ACK(auto=true)，刚拿到消息还没处理完程序就崩溃了。

###
解决1:
rabbitmq:  
  publisher-confirm-type: correlated   # 开启 confirm 机制  
  publisher-returns: true              # 开启 return 机制  
  template:  
    mandatory: true

![[Pasted image 20260628123648.png]]

    
解决2:
设置交换机、队列、消息持久化

解决3:
@RabbitListener(queues = RabbitMQConfig.EVENT_SYNC_QUEUE,ackMode = "MANUAL")
![[Pasted image 20260628123710.png]]



###如何避免重复消费
1、业务字段去重,给表加业务唯一索引

2、Redis用etIfAbsent 原子操作，第一次设置成功，后续跳过，失败时删除 Redis 标记，让重试能再次处理，设置 TTL（如 30 分钟），自动过期清理

###消息积压了怎么办

一、紧急止损
1-临时扩容消费者,只改配置，不重启服务，将并发数`concurrency`调高
		
2- 检查消费者是否一直在 basicNack + requeue=true 死循环。如果是，立即改成requeue=false 让它进死信，先把队列疏通，数据回头再补。

二、分析积压原因
1-第三方API响应慢(IO等待)
2-数据库写入慢

