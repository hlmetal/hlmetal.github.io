---
layout: post
title:  "Spring初识之MVC"
date:   2019-02-15 09:00:30 +0200
categories: java spring web
---

Spring MVC基本知识与开发实践、包括视图层逻辑、Thymeleaf、RestTemplate、Restful web service、webflux等

## Spring MVC实践
### [认识Spring MVC](https://docs.spring.io/spring-framework/docs/current/reference/html/web.html)

### DispatcherServlet
DispatcherServlet和普通的servlet一样，需要声明和配置，可以在web.xml中或者通过Java代码配置来完成。通过spring的配置来找到它能识别的代理组件用来进行消息映射，view的处理，以及异常处理等等
### Spring MVC中常用注解
1. `@Controller` 
2. `@RestController`
3. `@RequestMapping`
* `@GetMapping`
* `@PostMapping`
* `@PutMapping`
* `@DeleteMapping`
参数：path/method指定映射路径与方法，params/headers限定映射范围，consums/produces限定请求与响应格式
4. `@RequestBody` `@ResponseBody` `@ResponseStatus` 分别表示请求正文、响应正文、响应HTTP状态码
5. `@PathVariable` `@RequestParam` `@RequestHeader` 分别表示url路径上的变量，请求参数，请求http头
6. HttpEntity/ResponseEntity

### 请求处理流程
1. 绑定一些Attribute：WebApplicationContext/LocalResolver/ThemeResolver
2. 处理Multipart,如果是则将请求转为MultipartHttpServletRequest
3. Handler处理，如果找到对应Handler，执行Controller及前后置处理器逻辑
4. 处理返回的Model，渲染视图

### 定义处理方法
```java
// 不存在name参数时才会匹配
@GetMapping(path = "/", params = "!name") 
// 存在name参数时才会匹配
@GetMapping(path = "/", params = "name")
// method定义方法为GET请求
@RequestMapping(path = "/{id}", method = RequestMethod.GET,
        produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
// 路径中id作为参数
@PathVariable Long id

// consumes参数映射的是Header里的Content-Type,此处Content-Type=application/json
// prodcues参数映射的是Header里的Accept,此处Accept=application/json;charset=UTF-8
@PostMapping(path = "/", consumes = MediaType.APPLICATION_JSON_VALUE,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE)
@ResponseStatus(HttpStatus.CREATED)
```
### 定义类型转换
Spring Boot在WebMvcAutoConfiguration实现了WebMvcConfigurer, 当然也可以自己实现WebMvcConfigurer, 自定义的Convertor和Formatter
### 定义校验
通过Validator对绑定结果进行校验, 在绑定对象上使用@Valid注解, SpringMVC就会自动校验，检查结果会通过BindingResult参数传递进来

```java
//请求参数, 封装在RequestParm对象中
@NotEmpty
private String name;
@NotNull
private Money price;

//consumes指定表单提交
@PostMapping(path = "/", consumes = MediaType.APPLICATION_FORM_URLENCODED_VALUE)
@ResponseBody
@ResponseStatus(HttpStatus.CREATED)
public Product addProduct(@Valid RequestParm param,BindingResult result) {//@Valid自动校验param
    // 添加BindingResult参数后, 可以自定义返回错误结果
    if (result.hasErrors()) {
        // 这里先简单处理一下，后续讲到异常处理时会改
        log.warn("Binding Errors: {}", result);
        return null;
    }
}
```

### 文件上传(Multipart)
1. 配置MultipartResovler, Spring Boot自动配置MultipartAutoConfiguration, 支持类型multipart/form-data(MultipartFile类型)
2. 可以在配置文件中配置相关参数, 例如`spring.servlet.multipart.maxFileSize`

```java
@PostMapping(path = "/", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
@ResponseBody
@ResponseStatus(HttpStatus.CREATED)
public List<Product> batchAddProduct(@RequestParam("file") MultipartFile file) {
    List<Product> products = new ArrayList<>();
    if (!file.isEmpty()) {
        BufferedReader reader = null;
        try {
            reader = new BufferedReader(
                    new InputStreamReader(file.getInputStream()));
            String str;
            while ((str = reader.readLine()) != null) {
                String[] arr = StringUtils.split(str, " ");
                if (arr != null && arr.length == 2) {
                    products.add(productService.save(arr[0],
                            Money.of(CurrencyUnit.of("CNY"),
                                    NumberUtils.createBigDecimal(arr[1]))));
                }
            }
        } catch (IOException e) {
            log.error("exception", e);
        } finally {
            IOUtils.closeQuietly(reader);
        }
    }
    return products;
}
```


### ApplicationContext(应用上下文)
#### Web上下文层次
1. Servlet WebApplicationContext
* Controllers
* ViewRsolver
* HandlerMapping
2. Root WebApplicationContext
* Serivces
* Repositories

### 视图解析实现的基础
视图解析主要是通过ViewResovler接口实现的
#### DispatcherServlet中的视图解析逻辑
1. initStrategies()
* initViewResolvers() 初始化对应的ViewResolver
2. doDispatch()
* ProcessDispatchResult() 没有返回视图的话，尝试RequestToViewNameTranslator
* resolverViewName() 解析View对象

### Spring MVC支持的视图
1. Jackson-based JSON/XML
2. Thymeleaf & FreeMarker
3. JSP & JSTL
4. ...

#### 配置MessageConverter
通过WebMcvConfigurer的configureMessageConverters(), spring boot 自动查找HttpMessaggeConverters进行注册
#### Spring MVC 对Jackson的支持
1. JacksonAutoConfiguration
* Spring Boot通过@JsonComponent注册JSON序列化组件
* Jackson2ObjectMapperBuilderCustomizer

2. JacksonHttpMessageConvertersConfiguration
* 增加jackson-dataformat-xml以支持XML序列化

#### Spring MVC 对Thymeleaf的支持
1. 使用Thymeleaf
* 添加Thymeleaf依赖 `spring-boot-starter-thymeleaf`
* Spring Boot的自动配置 `ThymeleafAutoConfiguration`
2. Thymeleaf一些默认配置
```java
spring.thymeleaf.cache=false
spring.thymeleaf.check-template=true
spring.thymeleaf.check-template-location=truee
spring.thymeleaf.enabled=true
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.mode=HTML
spring.thymeleaf.servlet.content-type=text/html
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```
3. 简单示例
```java
<html xmlns:th="http://www.thymeleaf.org">
<body>
<h2>User Information</h2>
<table>
    <thead>
    <tr>
        <th>User Name</th>
        <th>Phone</th>
    </tr>
    </thead>
    <tbody>
    <tr th:each="user : ${userList}">
        <td th:text="${user.userName}">nico</td>
        <td th:text="${user.phone}">2134123</td>
    </tr>
    </tbody>
</table>
</body>
</html>
```

```java
@ModelAttribute
public List<User> userList() {
    UserExample example = new UserExample();
    return userService.selectByExample(example);
}

@GetMapping(path = "/user_list")
public ModelAndView showCreateForm() {
    return new ModelAndView("user");
}
```

### Spring MVC中静态资源配置
1. 核心逻辑: WebMvcConfigurer.addResourceHandlers()
2. 常用配置
* 静态资源路径, 默认从根路径开始去找 `spring.mvc.static-path-pattern=/**`
* 静态资源位置 `spring.resources.static-locations=classpath:/META-INF/resources/,classpath:/static/,classpath:/public/`

### Spring MVC中缓存配置
1. 核心逻辑: ResourceProperties.Cache
2. 常用配置
* 最大缓存时间 `spring.resources.cache.cachecontrol.max-age=10000`
* 不做缓存 `spring.resources.cache.cachecontrol.no-cache=true`
* 公共缓存,缓存资源时间`spring.resources.cache.cachecontrol.s-max-age=30000`

### Spring MVC 异常处理
1. 核心接口 `HandlerExceptionResovler`
2. 实现类 `SimpleMappingExceptionResovler``DefaultHandlerExceptionResolver``ResponseStatusExceptionResolver`
3. 处理方法 方法上添加 `@ExceptionHandler`
4. 添加位置 `@Controller``@RestController``@ControllerAdvice``@RestControllerAdvice`
```java
@RestControllerAdvice
public class GlobalControllerAdvice {
    @ExceptionHandler(ValidationException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public Map<String, String> validationExceptionHandler(ValidationException exception) {
        Map<String, String> map = new HashMap<>();
        map.put("message", exception.getMessage());
        return map;
    }
}
//...
public ...(@Valid @RequestBody Parm param BindingResult result) {
if (result.hasErrors()) {
            log.warn("Binding Errors: {}", result);
            throw new ValidationException(result.toString());
        }
```

### Spring MVC的拦截器
1. 核心接口 `HandlerInterceptor`
2. 方法 `boolean preHandle()``void postHandle()``void afterCompletion()`
3. 针对@ResponseBody和ResponseEntity的情况,提供了ResponseBodyAdvice
4. 针对异步请求接口,提供了AsynHandlerInterceptor接口(void afterConcurrentHandlingStarted())
5. 配置拦截器
* 常规方法 WebMvcConfigurer.addInterceptors()
* springboot中配置: 创建一个带`@Configuration`的WebMvcConfigurer配置类,自己实现addInterceptors方法

```java
//配置拦截器
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new PerformanceInteceptor())
            .addPathPatterns("/user/**").addPathPatterns("/product/**");
}

// 拦截器
public class PerformanceInteceptor implements HandlerInterceptor {
    private ThreadLocal<StopWatch> stopWatch = new ThreadLocal<>();

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        StopWatch sw = new StopWatch();
        stopWatch.set(sw);
        sw.start();
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
        stopWatch.get().stop();
        stopWatch.get().start();
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
        StopWatch sw = stopWatch.get();
        sw.stop();
        String method = handler.getClass().getSimpleName();
        if (handler instanceof HandlerMethod) {
            String beanType = ((HandlerMethod) handler).getBeanType().getName();
            String methodName = ((HandlerMethod) handler).getMethod().getName();
            method = beanType + "." + methodName;
        }
        log.info("{};{};{};{};{}ms;{}ms;{}ms", request.getRequestURI(), method,
                response.getStatus(), ex == null ? "-" : ex.getClass().getSimpleName(),
                sw.getTotalTimeMillis(), sw.getTotalTimeMillis() - sw.getLastTaskTimeMillis(),
                sw.getLastTaskTimeMillis());
        stopWatch.remove();
    }
}
```

## 访问Web资源
### 通过RestTemplate访问web资源
```java
@Autowired
private RestTemplate restTemplate;
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
    //return new RestTemplate();
    return builder.build();
}
```

### 常用方法
1. GET请求 `getForObject()``getForEntity()`
2. POST请求 `postForObject()``postForEntity()`
3. PUT请求 `put()`
4. DELETE请求 `delete()`

### 构造URI
1. 构造URI `UriComponentsBuilder`
2. 构造相对于当前请求的URL `ServletUriComponentsBuilder`
3. 构造指向Controller的URI `MvcUriComponentsBuilder`

```java
URI uri = UriComponentsBuilder
        .fromUriString("http://localhost:8080/user/{id}")
        .build(1);
ResponseEntity<User> c = restTemplate.getForEntity(uri, User.class);
log.info("Response Status: {}, Response Headers: {}", c.getStatusCode(), c.getHeaders().toString());
log.info("User: {}", c.getBody());

String userUri = "http://localhost:8080/user/";
User request = User.builder()
        .name("nico")
        .phone(2143234421)
        .build();
User response = restTemplate.postForObject(userUri, request, User.class);
log.info("New User: {}", response);

String s = restTemplate.getForObject(userUri, String.class);
log.info("String: {}", s);
```

### 高阶用法
1. RestTemplate.exchange()方法传递HTTP Heander
```java
RequestEntity<Void> req = RequestEntity.get(uri)
				.accept(MediaType.APPLICATION_XML)
				.build();
ResponseEntity<String> resp = restTemplate.exchange(req, String.class);
```
2. 类型转换使用`@JsonComponent`注解，并定义JsonSerializer/JsonDeserializer
```java
@JsonComponent
public class MoneySerializer extends StdSerializer<Money> {
    protected MoneySerializer() {
        super(Money.class);
    }

    @Override
    public void serialize(Money money, JsonGenerator jsonGenerator, SerializerProvider serializerProvider) throws IOException {
        jsonGenerator.writeNumber(money.getAmount());
    }

    @Override
    public Money deserialize(JsonParser p, DeserializationContext ctxt) throws IOException, JsonProcessingException {
        return Money.of(CurrencyUnit.of("CNY"), p.getDecimalValue());
    }
}
```
3. 解析泛型对象 exchange()方法中传入`ParamterizedTypeReference<>`参数
```java
ParameterizedTypeReference<List<Product>> ptr =
        new ParameterizedTypeReference<List<Product>>() {};
ResponseEntity<List<Product>> list = restTemplate
        .exchange(uri, HttpMethod.GET, null, ptr);
```

### 简单定制RestTemplate
1. RestTemplate支持的HTTP库
* 通用接口`ClientHttpRequestFactory`,默认实现`SimpleClientHttpRequestFactory`
* Apache HttpComponents提供的`HttpComponentsClientHttpRequestFactory`
* Netty提供的`Netty4ClientRequestFactory`
* OkHttp提供的`OkHttp3ClientRequestFactory`
2. 优化底层请求策略
* 连接管理 PoolingHttpClientConnectionManager、keepAlive策略

```java
public HttpComponentsClientHttpRequestFactory requestFactory() {
    PoolingHttpClientConnectionManager connectionManager =
                new PoolingHttpClientConnectionManager(30, TimeUnit.SECONDS);
        connectionManager.setMaxTotal(200);
        connectionManager.setDefaultMaxPerRoute(20);

    CloseableHttpClient httpClient = HttpClients.custom()
        .setConnectionManager(connectionManager)
        .evictIdleConnections(30, TimeUnit.SECONDS)
        .disableAutomaticRetries()
        // 有 Keep-Alive 认里面的值，没有的话永久有效
        //.setKeepAliveStrategy(DefaultConnectionKeepAliveStrategy.INSTANCE)
        // 换成自定义的
        .setKeepAliveStrategy(new CustomConnectionKeepAliveStrategy())
        .build();

    HttpComponentsClientHttpRequestFactory requestFactory =
        new HttpComponentsClientHttpRequestFactory(httpClient);

    return requestFactory;
}
```

* 超时设置 connectTimeOut/readTimeOut
* SSL校验

### WebClient
1. WebClient是一个以Reactive方式处理HTTP请求的非阻塞客户端
2. 支持的HTTP库
* Reactive Netty——ReactorClientHttpConnector
* Jetty ReactiveStream HttpClient——JettyClientHttpConnector
3. 基本用法
* 创建WebClient: `WebClient.create()``WebClient.builder()`
```java
@Bean
public WebClient webClient(WebClient.Builder builder) {
    return builder.baseUrl("http://localhost:8080").build();
}
```
* 发起请求: get()/put()/post()/delete()/patch()
* 获取结果: retrieve()/exchange()
* 处理Http Status: onStatus()
* 应答正文: bodyToMono()/bodyToFlux()

```java
webClient.get()
        .uri("/product/{id}", 1)
        .accept(MediaType.APPLICATION_JSON_UTF8)
        .retrieve()
        .bodyToMono(Product.class) //单个对象
        .doOnError(t -> log.error("Error: ", t))
        .subscribeOn(Schedulers.single())
        .subscribe(c -> log.info("Product 1: {}", c));

Mono<Product> p = Mono.just(
        Product.builder()
                .name("apple")
                .price(Money.of(CurrencyUnit.of("CNY"), 25.00))
                .build()
);
webClient.post()
        .uri("/product/")
        .body(p, Product.class)
        .retrieve()
        .bodyToMono(Product.class)
        .subscribeOn(Schedulers.single())
        .subscribe(c -> log.info("Product Created: {}", c));

webClient.get()
        .uri("/product/")
        .retrieve()
        .bodyToFlux(Product.class) //多个对象
        .toStream()
        .forEach(c -> log.info("Product in List: {}", c));
}
```

## Web开发进阶
### RESTful Web Service
#### Richardson成熟度模型
* Lv0: The Swamp of POX
* Lv1: Resources
* Lv2: Http Verbs
* Lv3: Hypermedia Controls
* Glory of REST

#### 如何实现Restful Web Service
1. 识别资源
* 找到领域名称: 能用CURD操作的名词, 例如User、Product
* 将资源组织为集合(即集合资源)
* 将资源合并为复合资源
* 计算或处理函数
2. 选择合适的资源粒度
* 站在服务端角度: 网络效率、表述的多少、客户端的易用程度
* 站在客户端角度: 可缓存性、修改频率、可变性
3. 设计URI
* 使用域或子域对资源进行合理的分组或划分
* 在URI路径部分使用"/"来表示资源之间的层次关系, 例如: user/id, product/order/id
* 在URI路径部分使用","和";"来表示非层次元素
* 使用连字符"-"和下划线"_"来改善长路径中名称的可读性
* 在URI的查询部分使用"&"来分隔参数
* 在URI中避免出现文件扩展名(.php/.aspx/.jsp等)
4. 选择合适的HTTP方法和返回码
* GET   用于信息获取(安全/幂等, 即不会改变资源，且返回结果一样)
* POST  用于创建、更新、删除资源等(非安全/非幂等)
* DELETE    删除资源(非安全/幂等)
* PUT   更新或完全替换一个资源(非安全/幂等)
* HEAD  获取与GET一样的HTTP请求头信息, 但没有响应体(安全/幂等)
* OPTIONS   获取资源支持的HTTP方法列表(安全/幂等)
* TRACE     让服务器返回其收到的HTTP头(安全/幂等)

```java
/user/ GET 获取user
/user/ POST 创建user
/user/{id}  GET 获取特定user
/user/{id}  PUT 修改特定user
/user/{id}  DELETE 删除特定user
```
* 200   OK
* 201   Created
* 202   Accepted
* 301   Moved Permanently
* 303   See Ohter
* 304   Not Modified
* 307   Temporary Redirect
* 400   Bad Request
* 401   Unauthorized
* 403   Forbidden
* 404   Not Found
* 410   Gone
* 500   Internal Server Error
* 503   Serivce Unavailable
5. 设计资源的表述(JSON/XML/HTML)
* JSON: MappingJackson2HttpMessageConverter\GsonHttpMessageConverter\JsonbHttpMessageConverter
* XML: MappingJacson2XmlHttpMessageConverter\Jaxab2RootElementHttpMessageConverter
* HTML
* ProtoBuf: ProtobufHttpMessageConverter

#### 超媒体驱动HATEOAS(Hypermedia As The Engine Of Application State)
1. HATEOAS vs SOA'WSDL
* HATEOAS 表述中的超链接会提供服务所需要的各种REST接口信息。无需事先约定如何访问服务
* WSDL 传统服务契约必须事先约定服务的地址与格式
2. 常用超链接类型
* self  指向当前资源本身的连接
* edit  指向一个可以编辑当前资源的连接
* collection    如果当前资源包含在某个集合中，指向该集合的连接
* search    指向一个可以搜索当前资源与相关资源的连接
* related   指向一个与当前资源相关的连接
* first     集合遍历相关类型，指向第一个资源的连接
* last      指向最后一个资源的连接
* previous  指向上一个资源的连接
* next      指向下一个资源的连接

#### Spring Data Rest
##### 认识HAL(Hypertext Application Language)
1. HAL是一个基于JSON的扩展，为API中的资源提供简单一致的连接
2. HAL模型
* 链接
* 内嵌资源
* 状态

##### spring-boot-starter-rest
1. 常用注解 
* `@RepositoryRestResouce`
2. 常用类
* `Resouce<T>`
* `PagedResouce<T>`
```java
@RepositoryRestResource(path = "/user")
public interface UserRepository extends JpaRepository<User, Long> {
    List<Coffee> findByNameInOrderById(List<String> list);
    Coffee findByName(String name);
}
```
3. 如何访问HATEOAS服务
* 配置JacksonJSON: 注册HAL
* 操作超链接

```java
// 在RestTemplate中注册此Bean
@Bean
public Jackson2HalModule jackson2HalModule() {
    return new Jackson2HalModule();
}

// 编辑link
private Link getLink(URI uri, String rel) {
    ResponseEntity<CollectionModel<Link>> rootResp =
            restTemplate.exchange(uri, HttpMethod.GET, null,
                    new ParameterizedTypeReference<CollectionModel<Link>>() {
                    });
    Link link = rootResp.getBody().getRequiredLink(rel);
    log.info("Link: {}", link);
    return link;
}
// 请求user数据的方法
private void readUsers(Link userLink) {
    ResponseEntity<PagedModel<EntityModel<User>>> userResp =
            restTemplate.exchange(userLink.getTemplate().expand(),
                    HttpMethod.GET, null,
                    new ParameterizedTypeReference<PagedModel<EntityModel<User>>>() {
                    });
    log.info("Users Response: {}", userResp.getBody());
}

// 调用
public void hateoasTest() {
    Link userLink = getLink(ROOT_URI, "user");
    readUsers(userLink);
}
```

### 会话(Session)解决方案
#### 常见的会话(Session)解决方案
1. 粘性会话Sticky Session 通过Load Balance实现,来自一个用户的请求的会话尽可能落在同一台机器上, 如果服务器变动, 则用户请求会落在其他机器上，这时用户原来持有的session就不生效了
2. 会话复制Session Replication 每台机器上的session都做一个复制,不管请求在哪台机器上都能获取这个session
3. 集中会话Centralized Session 使用JDBC或Redis等集中存储session信息,不管请求落在哪台机器上,只要持有相同的JSESSIONID,就能取得相同的会话

#### Spring Session 
1. 简化集群中用户会话管理, 无需绑定容器特定解决方案
2. 支持的存储: Redis、MongoDB、JDBC、Hazelcast
3. 实现原理: 通过定制的HttpServletRequest返回定制HttpSession(核心类包括SessionRepositoryRequestWrapper等)
4. 基于Redis的HttpSession
* 引入依赖`spring-session-data-redis`
* 基本配置`@EnableRedisHttpSession`

### WebFlux
#### 什么是WebFlux
WebFlux是用于构建基于Reactive技术栈之上的Web应用程序, 它是基于Reactive Stream API, 运行在非阻塞服务器上(Netty)
#### 为什么会有WebFlux
1. 对于非阻塞Web应用程序的需要
2. 函数式编程的需要

#### WebFlux性能
1. 请求耗时并没有很大改善
2. 仅需少量固定数量的线程和较少的内存即可实现扩展

#### WebFlux和WebMvc取舍
1. 现有MVC应用不必修改
2. 应用依赖大量阻塞式持久化API和网络API, 使用MVC
3. 已使用非阻塞技术栈的，建议使用WebFlux
4. 使用Lambda结合轻量级函数式框架的, 可以考虑WebFlux

#### WebFlux编程模型
1. 基于注解的控制器
* 常用注解: `@Controller``@RequestMapping` `@RequestBody/@ResponseBody`等
* 返回值必须是`Mono<T>/Flux<T>`类型
2. 基于函数式Endpoints




