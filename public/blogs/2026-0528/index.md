
## 1. RabbitMQ 消息可靠性保障

在 RabbitMQ 中，一条消息从生产者发送到消费者处理完成，通常会经历多个可靠性保障环节。

生产者发送消息到交换机后，可以通过 Confirm 机制确认消息是否成功到达交换机。  
如果交换机无法根据路由规则将消息投递到队列，则会触发 Return 回调。  
消息进入队列后，可结合队列持久化和消息持久化机制提升数据安全性。  
消费者消费消息后，需要手动发送 ACK 确认；如果处理失败，则可以 NACK、重新入队，或者转入死信队列。

## 2. 死信队列生成流程

以下几种情况会导致消息成为死信：

- 消息被消费者拒绝，并且不重新入队
- 消息在队列中超过 TTL 仍未被消费
- 队列达到最大长度，后续消息被丢弃

当消息成为死信后，会被发送到死信交换机，再由死信交换机路由到对应的死信队列，最后交由专门的消费者进行处理。

## 3. 延迟交换机 Delay Exchange

RabbitMQ 本身并不直接支持延迟队列，但可以通过安装延迟消息插件来实现延迟交换机功能。

其核心思路是：生产者发送消息时指定 delay 时间，消息先暂存在延迟交换机中；等延迟时间到达后，再根据路由键投递到目标队列，由消费者进行处理。

### 3.1 延迟交换机配置示例

```java
@Bean
public CustomExchange checkOrderStatusExchange() {
    return new CustomExchange(
            ConstantUtil.EXCHANGE_CHECK_ORDER_STATUS,
            "x-delayed-message",
            true,
            false,
            Map.of("x-delayed-type", "topic")
    );
}
```

### 3.2 队列配置示例

```java
@Bean
public Queue checkOrderQueue() {
    return new Queue(ConstantUtil.QUEUE_CHECK_ORDER_STATUS);
}
```

### 3.3 绑定配置示例

```java
@Bean
public Binding checkOrderQueueToExchange() {
    return BindingBuilder.bind(checkOrderQueue())
            .to(checkOrderStatusExchange())
            .with(ConstantUtil.ROUTING_CHECK_ORDER_STATUS)
            .noargs();
}
```

## 4. 秒杀异步下单功能

在高并发秒杀场景下，通常会采用异步下单模式来提升系统吞吐量并削峰填谷。

用户发起抢购请求后，系统先校验库存和购买资格，然后加锁防止重复下单，接着生成订单号并将订单消息发送到 MQ。  
接口可以快速向前端返回“排队中”或“提交成功”的响应，由消费者异步完成订单创建和数据库写入操作。

## 5. ObjectMapper

`ObjectMapper` 是 Jackson 提供的 JSON 处理核心工具类，可用于将 Java 对象转换为 JSON 字符串，也可以将 JSON 字符串反序列化为 Java 对象。

### 5.1 Java 示例

```java
ObjectMapper objectMapper = new ObjectMapper();
String json = objectMapper.writeValueAsString(order);
```

### 5.2 JSON 示例

```json
{
  "id": 1001,
  "userId": 2001,
  "orderNo": "KILL202605280001",
  "status": "PENDING"
}
```

## 6. 前端定时轮询 setInterval

前端可以通过 `setInterval` 周期性执行某个方法，常用于轮询查询订单状态。

例如，用户提交秒杀请求后，可以每隔 1.5 秒调用一次 `loadKillOrder` 方法；当获取到最终结果或页面销毁时，应调用 `clearInterval` 清除定时器，避免资源浪费。

### 6.1 JavaScript 示例

```javascript
this.timer = setInterval(this.loadKillOrder, 1500);
clearInterval(this.timer);
```

