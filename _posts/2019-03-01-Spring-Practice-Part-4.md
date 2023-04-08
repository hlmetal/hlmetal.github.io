---
layout: post
title:  "服务注册与发现"
date:   2019-03-01 09:00:30 +0200
categories: java spring cloud
---

认识微服务、云原生、各种服务注册中心的实践, Spring Cloud的服务注册与发现机制

## 微服务
微服务就是一些协同工作的小而自治的服务
### 微服务优点
1. 异构性: 语言、存储等等
2. 弹性: 一个组件不可用, 不会导致级联故障
3. 扩展性: 单体服务不易扩展, 多个较小的服务可以按需扩展
4. 易于部署
5. 与组织结构对齐
6. 可组合性
7. 可替代性

### 微服务的代价
1. 分布式系统复杂性
2. 开发、测试等诸多研发过程复杂性
3. 部署、监控等诸多运维复杂性

## Cloud Native(云原生)
云原生技术有利于各组织在公有云、私有云和混合云等新型动态环境中，构建和运行可弹性扩展的应用。
### Cloud Native的要求
1. DevOps: 开发和运维一同致力于交付高品质的软件服务于客户
2. 持续交付: 软件的构建、测试、发布等, 要更快、更频发、更稳定
3. 微服务: 以一组小型服务的形式来部署应用
4. 容器化: 提供比传统虚拟机更高的效率

### [12 Factor App](https://12factor.net/zh_cn/)
12 Factor为构建SaaS应用提供方法论, 适用于任意语言和后端服务（数据库、消息队列、缓存等）开发的应用程序
* 使用标准化流程自动配置
* 在各个系统中提供最大可移植性
* 适合部署在现代云计算平台
* 将开发和生产环境差异降至最低, 并使用持续交付实现敏捷开发
* 在工具、架构和开发流程不发生明显变化的前提下实现扩展
#### 12 Factors
1. 基准代码-一份基准代码，多份部署
* 使用版本控制系统加以管理
* 基准代码与应用保持一一对应关系
* 尽管每个应用只对应一份基准代码，但可以同时存在多份部署
2. 依赖-显式声明依赖关系
* 不会隐式依赖系统级的类库
* 通过依赖清单，确切地声明所有依赖项
* 在运行过程中，通过依赖隔离工具来确保程序不会调用系统中存在但清单中未声明的依赖项
3. 配置-代码和配置分离(配置中心)
4. 后端服务-把后端服务当作附加资源
5. 构建、发布、运行-严格分离
* 应用严格区分构建、发布、运行三个步骤
* 部署工具通常提供了发布管理工具
* 每个发布版本必须对应一个唯一的发布ID
6. 进程-以一个或多个无状态进程运行应用
* 应用的进程必须无状态且无共享
* 任何需要持久化的数据都要存储在后端服务内
7. 端口绑定-通过端口绑定提供服务
8. 并发-通过进程模型进行扩展
9. 易处理-快速启动和优雅终止
* 进程应当追求最小启动时间
* 进程一旦接收终止信号就会优雅终止
* 进程应当在面对突然死亡时保持健壮
10. 开发与生产环境等价-尽可能保持开发、pre、prod环境相同
* 想要持续部署就必须缩小本地和线上的差异
* 后端服务是保持开发与线上等价的重要部分
* 应该反对在不同环境间使用不同的后端服务
11. 日志-把日志当作事件流
12. 管理进程-后台管理任务当作一次性进程运行

## Spring Cloud
### 主要功能
1. 服务发现
2. 服务熔断
3. 配置服务
4. 服务安全
5. 服务网关
6. 分布式消息
7. 分布式跟踪
8. 各种云平台支持

### 服务注册与发现
#### Eureka注册服务
Eureka是在AWS上定位服务的REST服务,发布在Spring Cloud Netflix工程里
1. 本地启动简单的Eureka服务
* 引入依赖: `spring-cloud-dependencies``spring-cloud-starter-netflix-eureka-server`
* 声明: `@EnableEurekaServer`
* 默认端口: 8761
2. 将服务注册到Eureka Server
* 引入依赖: `spring-cloud-starter-netflix-eureka-client`
* 声明: `@EnableDiscoveryClient`或`@EnableEurekaClient`
* 配置: `eureka.client.service-url.default-zone``eureka.client.instance.prefer-ip-address`
3. Bootstrap属性: 启动引导阶段加载的属性(bootstrap.properties|yml)
* spring.cloud.bootstrap.name=bootstrap可以修改bootstrap.properties|yml的文件名
* 常用配置: spring.application.name=应用名以及配置中心相关

#### Loadbalancer访问服务
1. 获取服务地址: EurekaClient.getNextServerFromEureka()或DiscoveryClient.getInstances()(spring cloud提供,建议使用)
2. Load Balancer Client
* 通过`@LoadBalaced`为RestTemplate或WebClient增加负载均衡支持, 实际是通过ClientHttpRequestInterceptor实现的
* 无需知道服务方的IP和端口, 通过服务发现找到并调用

#### Feign访问服务
声明式REST Web服务客户端
1. Spring Cloud的支持: `spring-cloud-starter-openfeign`
2. 开启Feign支持: `@EnableFeignClients`
3. 定义feign接口: `@FeignClient`
```java
@FeignClient(name = "product-service", contextId = "fruit", path = "/fruit")
// 不要在接口上加@RequestMapping
public interface CoffeeService {
    @GetMapping(path = "/", params = "!name")
    List<Product> getAll();

    @GetMapping("/{id}")
    Product getById(@PathVariable Long id);

    @GetMapping(path = "/", params = "name")
    Product getByName(@RequestParam String name);
}
```
4. Spring Cloud提供的默认配置都在`FeignClientsConfiguration`(Encoder/Decoder/Logger/Contract/Client.../)
5. 通过配置定制Feign: `feign.client.config.*`
6. 其他配置: 
* 启用okhttp或httpclient支持`feign.okhttp.enabled=true``feign.httpclient.enabled=true`(需要引入相应依赖)
* 压缩支持`feign.compression.response|request.enabled=true`等等

#### 使用[Zookeeper]作为服务注册中心(https://zookeeper.apache.org/)
简单、多副本、有序、快
1. 依赖: `spring-cloud-starter-zookeeper-discovery`
2. 配置: `spring.cloud.zookeeper.connect-string=localhost:2181`
3. 通过Docker启动：`docker pull zookeeper``docker run --name zookeeper -p 2181:2181 -d zookeepr`
4. 启动项目后,使用`docker exec -it zookeeper bash`进入zookeeper,并使用zk client `./zkCli.sh`命令连上zookeeper
5. 使用`ls /services` 查看注册的项目, 进入项目目录查看注册的节点信息

#### 使用[Consul]作为服务注册中心(https://developer.hashicorp.com/consul)
1. 关键特性:服务发现、健康检查、KV存储、多数据中心支持、安全的服务间通信
2. 好用的功能: HTTP API、DNS(xxx.service.consul)、与Nginx联动
3. 简单实践
* 依赖: `spring-cloud-starter-consul-dicovery`
* 配置: `spring.cloud.consul.host|port|dicovery.prefer-ip-address`
* 通过Docker启动: `docker pull consul``docker run --name consul -d -p 8500:8500 -p 8600:8600/udp consul`
* 通过consul的ui界面查看注册信息

#### 使用[Nacos]作为服务注册中心(https://nacos.io/zh-cn/)
一个更易于构建云原生应用的动态服务发现、配置管理和服务管理平台
1. 功能: 动态服务配置、服务发现和管理、动态DNS服务
2. 简单实践
* 依赖: `spring-cloud-alibaba-dependencies``spring-cloud-starter-alibaba-nacos-discovery`
* 配置: `spring.cloud.nacos.discovery.server-addr`
* 通过Docker启动: `docker pull nacos/nacos-server``docker run --name nacos -d -p 8848:8848 -e MODE=standalone nacos/nacos-server`(localhost:8848/nacos, 用户名密码: nacos)

#### 自定义DiscoveryClient
1. 已经接触过的Spring Cloud类
* EurekaDiscoveryClient
* ZookeeperDiscoveryClient
* ConsulDiscoveryClient
* NacosDiscoveryClient
* RibbonLoadBalancerClient
2. 自定义DiscoveryClient需要做的:
* 返回该DiscoveryClient能提供的服务名列表
* 返回指定服务对应的ServiceInstance列表
* 返回DiscoveryClient的顺序
* 返回HealthIndicator里的描述
3. 自定义RibbonClient支持需要做的:
* 实现自定义`ServerList<T extends Server>`, Ribbon提供了`AbstractServerList<T extends Server>`
* 提供一个配置类, 声明ServerListBean实例

```java
product:
  services:
    - localhost:8080

@ConfigurationProperties("product")
@Setter
public class FixedDiscoveryClient implements DiscoveryClient {
    public static final String SERVICE_ID = "product-service";
    // product.services
    private List<String> services;

    @Override
    public String description() {
        return "DiscoveryClient that uses service.list from application.yml.";
    }

    @Override
    public List<ServiceInstance> getInstances(String serviceId) {
        if (!SERVICE_ID.equalsIgnoreCase(serviceId)) {
            return Collections.emptyList();
        }
        // 这里忽略了很多边界条件判断，认为就是 HOST:PORT 形式
        return services.stream()
                .map(s -> new DefaultServiceInstance(s,
                        SERVICE_ID,
                        s.split(":")[0],
                        Integer.parseInt(s.split(":")[1]),
                        false)).collect(Collectors.toList());
    }

    @Override
    public List<String> getServices() {
        return Collections.singletonList(SERVICE_ID);
    }
}

public class FixedServerList implements ServerList<Server> {
    @Autowired
    private FixedDiscoveryClient discoveryClient;

    @Override
    public List<Server> getInitialListOfServers() {
        return getServers();
    }

    @Override
    public List<Server> getUpdatedListOfServers() {
        return getServers();
    }

    private List<Server> getServers() {
        return discoveryClient.getInstances(FixedDiscoveryClient.SERVICE_ID).stream()
                .map(i -> new Server(i.getHost(), i.getPort()))
                .collect(Collectors.toList());
    }
}
```