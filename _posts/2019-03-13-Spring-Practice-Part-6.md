---
layout: post
title:  "消息与服务链路追踪简单实践"
date:   2019-03-13 09:00:30 +0200
categories: java spring cloud
---

通过RabbitMQ、Kafka实现简单的消息订阅与发送, 以及使用zipkin实现简单的链路追踪

## 消息
### Spring Cloud Stream
一款用于构建消息驱动的微服务应用程序的轻量级框架
#### 特性
1. 声明式编程模型
2. 引入多种概念抽象:发布订阅、消息组、分区
3. 支持多种消息中间件: RabbitMQ、Kafka等

#### Binder
Spring Cloud Stream应用程序与MQ之间的一层抽象, 把不同的消息中间件统一封装成binder, 应用程序只需要和Binder交互, 并不需要太多关注底层细节
* RabbitMQ
* Kafka
* Kafka Streams
* Amazon Kinesis
* RocketMQ

#### Binding
应用程序与Binder之间的一个桥梁，是应用中消息的生产者和、消费者与消息系统之间的桥梁
1. @EnableBinding
2. @Input   订阅消息 
3. @Output  发送消息

#### 消息组
对同一消息, 每个组中都会有一个消费者收到消息
#### 分区
#### 发送与接收消息
1. 生产消息: 使用MessageChannel中的send()方法、@SendTo
2. 消费消息: @StreamListener指定消费队列、@Payload代表消息体、@Headers、@Header消息头信息   

### [RabbitMQ](https://www.rabbitmq.com/getstarted.html)
1. 依赖: `spring-cloud-starter-stream-rabbit`
2. 配置: 
* `spring.cloud.stream.rabbit.binder.*`
* `spring.cloud.stream.rabbit.bindings.<channelName>.consumer.*`
* `spring.cloud.stream.rabbit.bindings.<channelName>.group.*`
* `spring.rabbitmq.host=localhost`
* `spring.rabbitmq.port=5672`
* `spring.rabbitmq.username=root`
* `spring.rabbitmq.password=d1207`
3. 通过Docker启动
* `docker pull rabbitmq:management-alpine`(包含rabbitmq管理界面)
* `docker run --name rabbitmq -d -p 5672:5672 -p 15672:15672 -e RABBITMQ_DEFAULT_USER=root -e RABBITMQ_DEFAULT_PASS=d1207 rabbitmq:management-alpine`

```java
/** Watiter服务 */
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=spring
spring.rabbitmq.password=spring

spring.cloud.stream.bindings.finishedOrders.group=waiter-service

@EnableBinding(Barista.class)
public class WaiterServiceApplication {public static void main(String[] args) {SpringApplication.run(WaiterServiceApplication.class, args);}}
	
public interface Barista {
    String NEW_ORDERS = "newOrders";
    String FINISHED_ORDERS = "finishedOrders";

    @Input
    SubscribableChannel finishedOrders();

    @Output
    MessageChannel newOrders();
}

@Component
@Slf4j
public class OrderListener {
    // 消费订单已制作完成的消息
    @StreamListener(Barista.FINISHED_ORDERS)
    public void listenFinishedOrders(Long id) {
        log.info("We've finished an order [{}].", id);
    }
}

//支付
public boolean updateState(CoffeeOrder order, OrderState state) {
    if (order == null) {
        log.warn("Can not find order.");
        return false;
    }
    if (state.compareTo(order.getState()) <= 0) {
        log.warn("Wrong State order: {}, {}", state, order.getState());
        return false;
    }
    order.setState(state);
    orderRepository.save(order);
    log.info("Updated Order: {}", order);
    if (state == OrderState.PAID) {
        // 有返回值，如果要关注发送结果，则判断返回值
        // 一般消息体不会这么简单
        // 发送订单已支付的消息
        barista.newOrders().send(MessageBuilder.withPayload(order.getId()).build());
    }
    return true;
}

/** Barista服务 */
spring.rabbitmq.host=localhost
spring.rabbitmq.port=5672
spring.rabbitmq.username=spring
spring.rabbitmq.password=spring

spring.cloud.stream.bindings.finishedOrders.group=waiter-service

@EnableBinding(Waiter.class)
public class BaristaServiceApplication {public static void main(String[] args) {SpringApplication.run(BaristaServiceApplication.class, args);}}

public interface Waiter {
    String NEW_ORDERS = "newOrders";
    String FINISHED_ORDERS = "finishedOrders";

    @Input(NEW_ORDERS)
    SubscribableChannel newOrders();

    @Output(FINISHED_ORDERS)
    MessageChannel finishedOrders();
}

@Component
@Slf4j
@Transactional
public class OrderListener {
    @Autowired
    private CoffeeOrderRepository orderRepository;
    @Autowired
    @Qualifier(Waiter.FINISHED_ORDERS)
    private MessageChannel finishedOrdersMessageChannel;
    @Value("${order.barista-prefix}${random.uuid}")
    private String barista;
    // 消费Waiter发过来的消息
    @StreamListener(Waiter.NEW_ORDERS)
    public void processNewOrder(Long id) {
        CoffeeOrder o = orderRepository.getOne(id);
        if (o == null) {
            log.warn("Order id {} is NOT valid.", id);
            return;
        }
        log.info("Receive a new Order {}. Waiter: {}. Customer: {}",
                id, o.getWaiter(), o.getCustomer());
        o.setState(OrderState.BREWED);
        o.setBarista(barista);
        orderRepository.save(o);
        log.info("Order {} is READY.", id);
        // 发送订单已制作完成的消息
        finishedOrdersMessageChannel.send(MessageBuilder.withPayload(id).build());
    }

}

```

### [Kafka](https://kafka.apache.org/documentation/#gettingStarted)
1. 依赖: `spring-cloud-starter-stream-kafka`
2. 配置:
* `spring.cloud.stream.kafka.binder.*`
* `spring.cloud.stream.kafka.bindings.<channelName>.consumer.*`
* `spring.kafka.*`
3. 通过Docker启动, kafka启动需要依赖zookeeper,根据官方指引, 需配置`docker-compose.yml`, 然后使用`docker-compose up -d`启动
```java
---
version: '2'
services:
  zookeeper:
    image: zookeeper:latest

  kafka:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 9092:9092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka:29092,PLAINTEXT_HOST://localhost:9092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
```

```java
/** Watiter服务
 * 只改变了依赖及配置, 其他没变
 */
spring.cloud.stream.kafka.binder.brokers=localhost
spring.cloud.stream.kafka.binder.defaultBrokerPort=9092

spring.cloud.stream.bindings.finishedOrders.group=waiter-service

/** Barista服务
 * 改变了依赖及配置
 * 发送消息使用注解形式, 其他未作变动
 */
spring.cloud.stream.kafka.binder.brokers=localhost
spring.cloud.stream.kafka.binder.defaultBrokerPort=9092

spring.cloud.stream.bindings.newOrders.group=barista-service

// 发送消息改为使用注解形式
@StreamListener(Waiter.NEW_ORDERS)
@SendTo(Waiter.FINISHED_ORDERS)
public Long processNewOrder(Long id) {
    CoffeeOrder o = orderRepository.getOne(id);
    if (o == null) {
        log.warn("Order id {} is NOT valid.", id);
        throw new IllegalArgumentException("Order ID is INVAILD!");
    }
    log.info("Receive a new Order {}. Waiter: {}. Customer: {}",
            id, o.getWaiter(), o.getCustomer());
    o.setState(OrderState.BREWED);
    o.setBarista(barista);
    orderRepository.save(o);
    log.info("Order {} is READY.", id);
    return id;
}
```

### 定时任务
1. TaskScheduler
* Trigger
* TriggerContext
2. 配置定时任务
* 开启定时任务支持`@EnableScheduling`
* 配置定时任务`<task:shcedule/>`或 `@Scheduled`
3. spring中的事件:ApplicationEvent
* 发送事件: ApplicationEventPublisherAware、ApplicationEventPublisher.publishEvent()
* 监听事件: ApplicationListener或@EventListener

```java

@EnableScheduling
public class CustomerServiceApplication {public static void main(String[] args) {SpringApplication.run(CustomerServiceApplication.class, args);}}


public class CustomerController implements ApplicationEventPublisherAware {
    private ApplicationEventPublisher applicationEventPublisher;

    @PostMapping("/order")
    public CoffeeOrder createAndPayOrder() {
        NewOrderRequest orderRequest = NewOrderRequest.builder()
                .customer("Li Lei")
                .items(Arrays.asList("capuccino"))
                .build();
        CoffeeOrder order = coffeeOrderService.create(orderRequest);
        log.info("Create order: {}", order != null ? order.getId() : "-");
        order = coffeeOrderService.updateState(order.getId(),
                OrderStateRequest.builder().state(OrderState.PAID).build());
        log.info("Order is PAID: {}", order);
        // 发送事件
        applicationEventPublisher.publishEvent(new OrderWaitingEvent(order));
        return order;
    }

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.applicationEventPublisher = applicationEventPublisher;
    }

}
public class CoffeeOrderScheduler {
    @Autowired
    private CoffeeOrderService coffeeOrderService;
    private Map<Long, CoffeeOrder> orderMap = new ConcurrentHashMap<>();

    // 监听事件
    @EventListener
    public void acceptOrder(OrderWaitingEvent event) {
        orderMap.put(event.getOrder().getId(), event.getOrder());
    }
    // 定时调度
    @Scheduled(fixedRate = 1000)
    public void waitForCoffee() {
        if (orderMap.isEmpty()) {
            return;
        }
        log.info("I'm waiting for my coffee.");
        orderMap.values().stream()
                .map(o -> coffeeOrderService.getOrder(o.getId()))
                .filter(o -> OrderState.BREWED == o.getState())
                .forEach(o -> {
                    log.info("Order [{}] is READY, I'll take it.", o);
                    coffeeOrderService.updateState(o.getId(),
                            OrderStateRequest.builder()
                                    .state(OrderState.TAKEN).build());
                    orderMap.remove(o.getId());
                });
    }
}
```

## 服务链路追踪
### 通过Dapper了解链路治理
1. 链路治理关注什么
* 系统中有哪些服务
* 服务间依赖关系是什么样的
* 请求的具体执行路径
* 请求各个环节执行是否正常及耗时情况等
* ......
2. Google Dapper的相关术语
* Span - 基本的工作单元
* Trace - 由一组Span构成的树形结构
* Annotation - 用于及时记录事件
    * cs - Client Sent
    * cr - Client Received
    * sr - Server Received
    * ss - Server Sent

### Spring Cloud Sleuth with Zipkin实现链路追踪
#### [Adding to the Project](https://cloud.spring.io/spring-cloud-sleuth/1.0.x/#_sleuth_with_zipkin_via_http)
1. 依赖: spring-cloud-starter-zipkin(spring-cloud-starter-sleuth)
2. 配置: 
* `spring.zipkin.base-url==http://localhost:9411`
* `spring.zipkin.discovery-client-enabled=false`
* `spring.zipkin.sender.type=web|rabbit|kafka`
* `spring.zipkin.compression.enabled=false`
* `spring.sleuth.sampler.probability=1.0`
3. Docker启动zipkin: `docker pull openzipkin/zipkin``docker run --name -d -p 9411:9411 openzipkin/zipkin`
4. 追踪消息链路:
* 如需通过MQ埋点, 需增加RabbitMQ或kafka依赖
* 配置`spring.zipkin.sender.type=rabbit``sping.zipkin.rabbitmq.queue=zipkin`
5. 让Zipkin能通过RabbitMQ接收消息
* 环境变量: `RABBIT_ADDRESSES=<RabbitMQ地址>``RABBIT_USER``RABBIT_PASSWORD`
* 运行Zipkin镜像: `docker run --name rabbit -zipkin -d -p 9411:9411 --link rabbitmq -e RABBIT_ADDRESSES=rabbitmq:5672 -e RABBIT_USER=root -e RABBIT_PASSWORD=d1207 openzipkin/zipkin`
6. 启动项目, 进入localhost:9411/zipkin下观察请求链路情况

### 服务治理
一个企业实施的用以保障事情正确完成的流程, 即遵循最佳实践, 体系架构原则, 治理条例, 法律和其他决定因素. SOA治理是指用于管理SOA的采用和实现的流程
1. 宏观上
* 架构设计是否合理
* 哪些链路是关键链路
* 链路的容量水位趋势
* 系统变更的管理与审计
2. 微观上
* 系统依赖了什么
* 系统有哪些配置
* 系统主观与客观质量