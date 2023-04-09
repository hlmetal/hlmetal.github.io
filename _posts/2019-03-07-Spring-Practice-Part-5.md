---
layout: post
title:  "服务保护与配置中心"
date:   2019-03-07 09:00:30 +0200
categories: java spring cloud
---

服务熔断与限流实践, 断路器模式、隔舱模式等应用以及基于spring cloud config server、zookeeper、consul、nacos等组件实现配置中心

## 服务熔断与限流
### 断路器模式
* 在断路器对象中封装受保护的方法调用
* 该对象监控调用和断路情况
* 调用失败触发阈值后, 后续调用直接由断路器返回错误, 不再执行实际调用

### 使用AOP实现简单的断路模拟
```java
@Aspect
@Component
@Slf4j
public class CircuitBreakerAspect {
    //阈值
    private static final Integer THRESHOLD = 3;
    private Map<String, AtomicInteger> counter = new ConcurrentHashMap<>();
    private Map<String, AtomicInteger> breakCounter = new ConcurrentHashMap<>();

    @Around("execution(* com.spring.hualan.hualanspring.integration..*(..))")
    public Object doWithCircuitBreaker(ProceedingJoinPoint pjp) throws Throwable {
        String signature = pjp.getSignature().toLongString();
        log.info("Invoke {}", signature);
        Object retVal;
        try {
            if (counter.containsKey(signature)) {
                if (counter.get(signature).get() > THRESHOLD &&
                        breakCounter.get(signature).get() < THRESHOLD) {
                    log.warn("Circuit breaker return null, break {} times.",
                            breakCounter.get(signature).incrementAndGet());
                    return null;
                }
            } else {
                counter.put(signature, new AtomicInteger(0));
                breakCounter.put(signature, new AtomicInteger(0));
            }
            retVal = pjp.proceed();
            counter.get(signature).set(0);
            breakCounter.get(signature).set(0);
        } catch (Throwable t) {
            log.warn("Circuit breaker counter: {}, Throwable {}",
                    counter.get(signature).incrementAndGet(), t.getMessage());
            breakCounter.get(signature).set(0);
            throw t;
        }
        return retVal;
    }
}
```

### [Hystrix](https://github.com/Netflix/Hystrix/wiki/How-To-Use)
Hystrix实现了断路器模式, 在需要做断路保护的方法上面添加`@HystrixCommand`

#### Spring Cloud支持
1. 引入依赖`spring-cloud-starter-netflix-hystrix`
2. 添加注解`@EnableCircuitBreaker`

#### Feign支持
1. 配置: `feign.hystrix.enabled=true`
2. @FeignClient中添加fallback/fallbackFactory

```java
@PostMapping("/product")
@HystrixCommand(fallbackMethod = "fallbackCreateProduct")
public Product createProduct() {
    NewProductRequest productRequest = NewProductRequest.builder()
            .name("apple")
            .price(15)
            .build();
    Product prod = productService.create(productRequest);
    log.info("Product ID: {}", prod != null ? prod.getId() : "-");
    return prod;
}

public Product fallbackCreateProduct() {
    log.warn("Fallback to NULL Product.");
    return null;
}


@FeignClient(name = "product-service", contextId = "product",
        qualifier = "prodcutService", path="/product",
        fallback = FallbackProductService.class)
// 如果用了Fallback，不要在接口上加@RequestMapping，path可以用在这里
public interface ProductService {
    @GetMapping(path = "/", params = "!name")
    List<Product> getAll();

    @GetMapping("/{id}")
    Product getById(@PathVariable Long id);

    @GetMapping(path = "/", params = "name")
    Product getByName(@RequestParam String name);
}

@Slf4j
@Component
public class FallbackCoffeeService implements ProductService {
    @Override
    public List<Product> getAll() {
        log.warn("Fallback to EMPTY.");
        return Collections.emptyList();
    }

    @Override
    public Product getById(Long id) {
        return null;
    }

    @Override
    public Product getByName(String name) {
        return null;
    }
}
```

### 观察服务熔断
#### 如何了解熔断情况
1. 打印日志: 在发生熔断时打印特定日志
2. 看监控: 主动向监控系统埋点, 上报熔断情况或提供与熔断相关的Endpoint, 让第三方系统来拉取信息

#### Hystrix Dashboard
* Sping Cloud提供了Hystrix Metrics Stream(spring-boot-starter-actuator/actuator/hystrix.stream)
* Sping Cloud提供了Hystrix Dashboard(spring-cloud-starter-netflix-hystrix-dashboard), 使用`@EnableHystrixDashboard`开启,访问`/hystrix`

#### 聚合集群熔断信息
* Spring Cloud提供了Netflix Turbine(`spring-cloud-starter-netflix-turbines`), 通过`@EnableTurbine`开启,访问`/turbine.stream?cluster=集群名`

### [Resilience4j](https://resilience4j.readme.io/docs/getting-started)
Resilience4j是一款受Hystrix启发的轻量级且易于使用的容错库
#### 核心组件
1. resilience4j-circuitbreaker  熔断
2. resilience4j-ratelimiter 频率控制
3. resilience4j-retry   自动重试
4. resilience4j-bulkhead    依赖隔离和负载保护
5. resilience4j-cache   应答缓存
6. resilience4j-timelimiter 超时控制

#### 实现服务熔断
1. 引入依赖: `resilience4j-spring-boot2`
2. 在需要断路保护的方法上添加注解: `@CircuitBreaker(name= '断路器名称')`
3. 配置
```java
resilience4j.circuitbreaker.backends.断路器名称.failure-rate-threshold=50
resilience4j.circuitbreaker.backends.断路器名称.wait-duration-in-open-state=5000
resilience4j.circuitbreaker.backends.断路器名称.ring-buffer-size-in-closed-state=5
resilience4j.circuitbreaker.backends.断路器名称.ring-buffer-size-in-half-open-state=3
resilience4j.circuitbreaker.backends.断路器名称.event-consumer-buffer-size=10
```

#### 实现服务限流
##### Bulkhead
1. Bulkhead可以防止下游依赖被并发请求冲击、防止发生连环故障
2. 引入依赖: `resilience4j-spring-boot2`
3. 在需要限流的方法上添加`Bulkhead(name="名称")`
4. 配置
```java
resilience4j.bulkhead.backends.名称.max-concurrent-call=1
resilience4j.bulkhead.backends.名称.max-wait-time=5
```

##### RateLimiter
1. RateLimiter限制特定时间段内的执行次数
2. 引入依赖: `resilience4j-spring-boot2`
3. 在需要限流的方法上添加`@RateLimiter(name="名称")`
4. 配置
```java
resilience4j.ratelimiter.limiters.名称.limit-for-period=3
resilience4j.ratelimiter.limiters.名称.limit-refresh-period-in-millis=30000
resilience4j.ratelimiter.limiters.名称.timeout-in-millis=1000
resilience4j.ratelimiter.limiters.名称.subscribe-for-events=true
resilience4j.ratelimiter.limiters.名称.register-health-indicator=true
```

#### 以编程方式实现熔断和限流
```java
private CircuitBreaker circuitBreaker;
private Bulkhead bulkhead;
private RateLimiter rateLimiter;

public ProductController(CircuitBreakerRegistry circuitBreakerRegistry,
                            BulkheadRegistry bulkheadRegistry, RateLimiterRegistry rateLimiterRegistry) {
    circuitBreaker = circuitBreakerRegistry.circuitBreaker("all");
    bulkhead = bulkheadRegistry.bulkhead("all");
    rateLimiter = rateLimiterRegistry.rateLimiter("item");
}

@GetMapping("/menu")
public List<Product> readAll() {
    return Try.ofSupplier(
            Bulkhead.decorateSupplier(bulkhead,
                    CircuitBreaker.decorateSupplier(circuitBreaker,
                            () -> productService.getAll())))
            .recover(CircuitBreakerOpenException.class, Collections.emptyList())
            .recover(BulkheadFullException.class, Collections.emptyList())
            .get();
}

@GetMapping("/{id}")
public Product getItem(@PathVariable("id") Long id) {
    Product product = null;
    try {
        product = rateLimiter.executeSupplier(() -> productService.get(id));
        log.info("Get Product: {}", product);
    } catch(RequestNotPermitted e) {
        log.warn("Request Not Permitted! {}", e.getMessage());
    }
    return product;
}
```

## 配置中心
### 基于git的配置中心
#### Spring Cloud Config Server
1. Spring Cloud Config Server提供针对外置配置的HTTP API
2. 依赖`spring-cloud-config-server`
3. 注解`@EnableConfigServer`
4. 支持Git/SVN/Vault/JDBC...
5. 使用Git作为后端存储
* 配置: `spring.cloud.config.server.git.uri`
* 配置文件要素
    * {application},即客户端的spring.application.name
    * {profile},即客户端的spring.profiles.active
    * {label},配置文件的特定标签, 默认master
* 访问配置内容
    * HTTP请求: `GET /{application}/{profile}[/{label}]`/`GET /{application}-{profile}.yml|properties]`

#### Spring Cloud Config Client
1. 依赖 `spring-cloud-starter-config`
2. 发现配置中心: 在bootstra.properties|yml中配置
* 通过配置固定uri `spring.cloud.config.uri=http://localhost:8888`
* 通过服务发现`spring.cloud.config.discovery.enabled=true``spring.cloud.config.discovery.service-id=configserver`
* 访问不上就快速失败 `spring.cloud.config.fail-fast=true`
3. 配置刷新: 在配置Bean上添加`@RefreshScope` , 当配置更新后可以使用/actuator/refresh刷新

#### Spring Cloud的配置抽象
Spring Cloud Config在分布式系统中，提供外置配置支持
1. 实现: 类似于Spring应用中的Environment和PropertySource, 在上下文中增加Spring Cloud Config的PropertySource
2. PropertySource
* Spring Cloud Config Client中的CompositePropertySource
* Zookeeper中的ZookeeperPropertySource
* Consul中的ConsulPropertySource/ConsulFilesPropertySource
3. 通过PropertySourceLocator找到PropertySource
4. 通过EnvirommentRepository支持Git/SVN/JDBC等backend
5. 功能特性: SSH、代理访问、配置加密...
6. 配置刷新: 
* 单机可以用/actuator/refresh
* 集群则使用Spring Cloud Bus-RefreshRemoteApplicationEvent
7. 配置组合顺序: 应用名-profile.yml->应用名.yml->application-profile.yml->application.yml


### 基于Zookeeper的配置中心
1. 依赖: `spring-cloud-starter-zookeeper-config`
2. 启用: 在bootstrap.properties|yml中配置
* `spring.cloud-zookeeper.connect-string=localhost:2181`
* `spring.cloud-zookeeper.config.enabled=true`
3. 配置项: 在zookeeper根节点创建
* 特定应用,特定环境的配置`./cofing/应用名,profile(dev,test,pre)/key=value`
* 全局配置`./cofing/application`, 即所有服务都能使用该节点下的配置
4. 定制
* 配置根节点名称 `spring.cloud-zookeeper.config.root=config`
* 全局配置节点名称 `spring.cloud-zookeeper.config.default-context=application`
* 配置应用名和环境的分隔符 `spring.cloud-zookeeper.config.profile-separator=','`
5. 配置刷新: zookeeper自动监听节点值的变化
6. 通过ZookeeperConfigBootstrapConfiguration注册ZookeeperPropertySourceLocater提供ZookeeperPropertySource
7. 通过ZookeeperConfigAutoConfiguration注册ConfigWatcher监听配置更新

```java
//在Zookeeper中创建配置节点和数据
create /config/product-service,dev/order.discount 70

//java代码中配置设置
@ConfigurationProperties("order")
@RefreshScope
@Data
@Component
public class OrderProperties {
    //默认值
    private Integer discount = 100;
}
```

### 基于Consul的配置中心
1. 依赖: `spring-cloud-starter-consul-config`
2. 启用: boootstrap.properties|yml中配置
* `spring.cloud.consul.host=localhost`
* `spring.cloud.consul.port=8500`
* `spring.cloud.consul.config.enabled=true`
* `spring.cloud.consul.config.format=KEY_VALUE|YAML|PROPERTIES|FILES`
3. Consul中数据存储：通过consul的ui界面建立如下节点
* /config/应用名,profile/data
* /config/application,profile/data
4. 定制:
* spring.cloud.consul.config.data-key=data
* spring.cloud.consul.config.root=config
* spring.cloud.consul.config.default-context=application
* spring.cloud.consul.config.profile-separator=','
5. 自动刷新配置(默认开启)
* spring.cloud.consul.config.watch.enabled=true
* spring.cloud.consul.config.watch.delay=1000

### 基于Nacos的配置中心
1. 依赖: `spring-cloud-starter-alibaba-nacos-config`
2. 启用: boootstrap.properties|yml中配置
* `spring.cloud.nacos.config.server-addre=127.0.0.1:8848`
* `spring.cloud.nacos.config.enabled=true`
3. Nacos中数据存储: 通过nacos的ui界面-配置管理中新建配置dataId=应用名.yaml
4. 配置项: 
* `${prefix}-${spring.profile.active}.${file-extension}`
* `spring.cloud.nacos.config.prefix`
* `spring.cloud.nacos.config.file-extension=yaml`
* `spring.cloud.nacos.config.group`
