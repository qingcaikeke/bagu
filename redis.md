应用场景：消息队列在存取消息时，必须要满足三个需求，分别是**消息保序、处理重复的消息和保证消息可靠性**。

可靠性：消费者程序从 List 中读取一条消息后，List 就不会再留存这条消息了。所以，如果消费者程序在处理消息的过程出现了故障或宕机，就会导致消息没有处理完成，那么，消费者程序再次启动后，就没法再次从 List 中读取消息了

- 消息保序：使用 LPUSH + RPOP；
- 阻塞读取：使用 BRPOP；（消费者不知道list中是否有消息写入，只能一直循环）
- 重复消息处理：生产者自行实现全局唯一 ID；
- 消息的可靠性：使用 BRPOPLPUSH，**让消费者程序从一个 List 中读取消息，同时，Redis 会把这个消息再插入到另一个 List（可以叫作备份 List）留存**

List 作为消息队列有什么缺陷？List 作为消息队列有什么缺陷？



