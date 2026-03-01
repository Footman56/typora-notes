# 交换机类型

## 直连交换机

默认交换机，路由健与队列名完全匹配交换机，完全匹配，单播，一个key 可以绑定多个queue 队列，当匹配到key1时 queue1和queue2都会收到消息

## 扇形交换机

直接广播

## 主题交换机

topic 可以进行模糊匹配

## 头部交换机

配置AMQP消息的header 不是路由健， 几乎不用

![image-20260301214005755](https://raw.githubusercontent.com/Footman56/images-2/master/img202603012140803.png)

# 确保消息不丢失

可能丢失的三种情况：生产者到rabbitmq服务器、rabbitmq消息持久化、消息者拉起消息

## confirm 消息确认机制（生产者）

Confirm 模式是 RabbitMQ 提供的一种消息可靠性保障机制。当生产者通过 Confirm 模式发送消息时，它会等待 RabbitMQ的确认，确保消息已
经被正确地投递到了指定的 Exchange 中。
消息正确投递到 queue 时，会返回 ack。
消息没有正确投递到 queue 时，会返回 nack。如果 exchange 没有绑定 queue，也会出现消息丢失。

## 消息持久化

消息存储到磁盘

## ACK 确认机制

