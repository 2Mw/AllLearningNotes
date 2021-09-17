# Java项目

[TOC]

## 商城秒杀系统

[BV1SL411H7wN](https://www.bilibili.com/video/BV1SL411H7wN)  P52

学习目标：如何实现高并发高性能的系统，满足高性能，一致性，高可用的特点。

项目结构：

![image-20210824150158587](https://i.loli.net/2021/08/24/oPTY1bdAVijnKrQ.png)

### 两次MD5加密

前端发送到服务器的时候需要加密一次，从后端再到持久层的时候在加密一次。MD5(MD5(PASS+salt)+salt)

添加POM依赖：

```xml
<dependency>
    <groupId>commons-codec</groupId>
    <artifactId>commons-codec</artifactId>
</dependency>

<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
</dependency>
```

### MyBatis-Plus使用以及代码生成

> 可以快速生成Model，mapper，controller，service等java代码

添加依赖：

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3</version>
</dependency>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.4.1</version>
</dependency>

<!--HTML模板引擎-->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
</dependency>

<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
</dependency>
```

添加代码：[代码生成器](https://baomidou.com/guide/generator.html#代码生成器)

修改其中的部分代码配置（比如dsn，包名），运行，找到对应的表生成代码。

配置yaml：

```yaml
mybatis-plus:
  mapper-locations: classpath*:/mapper/**/*Mapper.xml
  type-aliases-package: com.xx.xxx.model
```





### 分布式Session

解决方案：Session复制，前端存储，session粘滞，后端集中存储，redis分布式解决

🔵方法一：使用SpringSession来实现分布式session

添加依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!--lettuce 可能用到的对象池-->
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-pool2</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session-data-redis</artifactId>
</dependency>
```

Spring配置redis：

```yml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    database: 0
    lettuce:
      pool:
        max-active: 8
        max-wait: 10000ms
```

> 配置完之后就已经实现分布式session，当访问网站即可将session存到redis中。

🔵方法二：直接将用户信息存储到redis中去。

首先写一个redis配置类用于操作redis

```java
@Configuration
public class RedisConf {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory){
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer()); // 序列化到redis中为json而非二进制
        // 对Hash类型序列化
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());


        // 注入连接工厂
        redisTemplate.setConnectionFactory(factory);
        return redisTemplate;
    }
}
```

操作redis：

```java
@Autowired
RedisTemplate<String, Object> redisTemplate;

redisTemplate.opsForValue().set("user"+uid, user);
User user = (User) redisTemplate.opsForValue().get("user" + ticket);
```

### 优化登录功能

> 也可以使用拦截器实现

在做每一次业务请求的时候，都需要判断用户是否登录的session，比较麻烦，太重复了，进行参数处理。这里使用到的是**SpringMVC的自定义参数解析器**。

这个参数解析器，如果在controller中的形参中碰到某种类型的参数，即会进行参数解析，将其他参数传入到对应继承了`HandlerMethodArgumentResolver`接口的类中，进行处理，并且返回对应类型的参数。

原来的样子：

```java
@Autowired
IUserService userService;

@RequestMapping("/tolist")
public String toList(HttpServletRequest req, HttpServletResponse rsp, Model model, @CookieValue("userTicket") String ticket){
    if (!StringUtils.hasLength(ticket))return "login";
    //        User user = (User) session.getAttribute(ticket);
    User user = userService.getUserByCookie(ticket,req, rsp);
    if (user == null)return "login";
    model.addAttribute("user", user);
    return "goodsList";
}
```

功能改为：

```java
@RequestMapping("/tolist")
public String toList(Model model, User user){
    //        if (user == null)return "login";
    model.addAttribute("user", user);
    return "goodsList";
}
```

这里将req，rsp等形参变为了`user`，更加注重controller层的业务处理。

首先需要编写一个WebMVC配置类，继承`WebMvcConfigurer`，重写`addArgumentResolvers`方法来进行controller的参数解析。并且由于重写会导致springboot的默认静态资源目录失效，需要对`addResourceHandlers`进行重写，添加对应资源目录。

```java
@Configuration
@EnableWebMvc
public class WebConf implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/**").addResourceLocations("classpath:/static/");
    }

    @Autowired
    UserArgumentResolver userArgumentResolver;

    @Override
    public void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
        resolvers.add(userArgumentResolver);
    }
}
```

再编写对于User参数的解析器，遇到形参有User类型的controller层即开始解析：

这里使用NativeWebRequest来获取对应的HttpServletRequest和HttpServletResponse

```java
@Component
public class UserArgumentResolver implements HandlerMethodArgumentResolver {
    @Autowired
    IUserService userService;

    // 用于条件判断，如果执行返回true之后，才会执行`resolveArgument`方法, false 则不执行
    @Override
    public boolean supportsParameter(MethodParameter methodParameter) {
        return methodParameter.getParameterType().equals(User.class);
    }

    @Override
    public Object resolveArgument(MethodParameter methodParameter,
                                  ModelAndViewContainer modelAndViewContainer,
                                  NativeWebRequest webRequest,
                                  WebDataBinderFactory webDataBinderFactory) throws Exception {
        // 这里用于判断是否登录的流程
        HttpServletRequest request = webRequest.getNativeRequest(HttpServletRequest.class);
        HttpServletResponse rsp = webRequest.getNativeResponse(HttpServletResponse.class);
        String ticket = CookieUtils.getCookie(request, "userTicket");
        if (!StringUtils.hasLength(ticket))return null;
        return userService.getUserByCookie(ticket, request, rsp);
    }
}
```

### 秒杀功能实现

一、检查库存是否充足。二、每个用户只可以买一件，不可以重复买。

即实现库存减一，订单和秒杀订单中添加记录，基本的CURD操作。

### 压力测试

两个测试标准：QPS（Query Per Second），TPS（Transaction Per Second）

使用[JMeter](https://jmeter.apache.org/)工具来进行压力测试。

对于秒杀功能的测试，首先需要准备5000个用户及其Cookie来进行测试前置。

> 获取5000个用户Cookie可以使用Go语言多线程进行获取，速度极快。

将对应的用户和Cookie信息写入到CSV文件中，并且配置JMeter的CSV配置，导入5000个信息。

![image-20210828203212947](https://i.loli.net/2021/08/28/uBnFV42dGQRfgYM.png)

设置HTTP Cookie管理器：

![image-20210828203253745](https://i.loli.net/2021/08/28/grPcQIhJl9T6R2i.png)

开始运行。

> 这里会出现超卖了现象，库存出现负数的情况。

### 缓存

一般放到缓存中的是被频繁读取而且很少进行改动的数据。

🔵页面缓存

将整个HTML页面进行缓存，一般情况下都是前后端分离，这里因为使用到了thymeleaf的问题，将thymeleaf的整个模板+数据库数据放到redis中，并且设置60s秒过期时间。

```java
@RequestMapping(value = "/tolist", produces = "text/html;charset=utf-8")
@ResponseBody
public String toList(Model model, User user, HttpServletRequest request, HttpServletResponse rsp){
    // 缓存查询
    ValueOperations<String, Object> ops = redisTemplate.opsForValue();
    String html = (String) ops.get("goodsList");
    if (StringUtils.hasLength(html))return html;

    //        if (user == null)return "login";
    if (user == null)return null;
    model.addAttribute("user", user);
    model.addAttribute("goodsList", goodsService.findGoodsVo());
    // 手动渲染引擎
    WebContext context = new WebContext(request, rsp, request.getServletContext(), request.getLocale(), model.asMap());
    html = thymeleafViewResolver.getTemplateEngine().process("goodsList", context);
    if (StringUtils.hasLength(html))ops.set("goodsList", html,60, TimeUnit.SECONDS);
    return html;
}
```

🔵URL缓存：

即再redis中存储的字段为：`user:1` `user:2:tel`类似的格式。

🔵对象存储

如何去保证数据库和缓存数据的一致性，当进行数据库操作的时候需要对redis缓存中的信息进行删除清空，更新缓存中的信息，才能够保证前端页面获取到的不是旧的未修改的数据。

🔵页面静态化：

开启`Accept-Encoding:gzip,deflate`选项。

### 解决超卖

出现超卖的情况，可能是一个用户多次下单同一个商品，或者是在更新库存减一的时候没有验证库存量是否大于0，并且防止数据库过量访问。

1. 解决一个用户多次下单同一个商品

   简单的解决办法：在数据库表中字段添加唯一UNIQUE索引，防止重复。

3. 更新的时候使用验证`stock_count > 0`
   
   ```sql
   update t_seckill_goods set stock_count = stock_count - 1 where goods_id = ${goods.id} and stock_count > 0;
   ```

3. 防止数据库单用户过量访问：

   在用户成功下单之后，将数据加入到redis中，用户再次下单的时候检查redis中是否存在数据即可。

### RabbitMQ

添加Springboot依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

yml配置：

```yaml
spring:
  rabbitmq:
    host: localhost
    username: admin
    password: 123456
    virtual-host: /
    port: 5672
    listener:
      simple:
        concurrency: 10     # 消费者最小并发数量
        max-concurrency: 10
        prefetch: 1           # 限制消费这每次只能处理一个消息
        auto-startup: true
        default-requeue-rejected: true    # 被拒绝是否重新进入队列
```

声明交换机、队列和绑定：

```java
@Configuration
public class RabbitMQConfig {

    public static final String EXCHANGE_FANOUT = "fanoutEx01";
    @Bean
    public Queue queue01(){ return new Queue("queue01", true); }

    @Bean
    public Queue queue02(){ return new Queue("queue02", true); }

    @Bean
    public Queue queue03(){ return new Queue("queue03", true); }

    @Bean
    public FanoutExchange fanoutExchange01(){
        return new FanoutExchange(EXCHANGE_FANOUT);
    }

    @Bean
    public DirectExchange directExchange01(){
        return new DirectExchange("Direct01");
    }
    
    // Fanout 绑定

    @Bean
    public Binding binding01(){
        return BindingBuilder.bind(queue01()).to(fanoutExchange01());
    }

    @Bean
    public Binding binding02(){
        return BindingBuilder.bind(queue02()).to(fanoutExchange01()).;
    }

	// 绑定routingKey
    @Bean
    public Binding binding03(){
        return BindingBuilder.bind(queue03()).to(directExchange01()).with("routingKey");
    }
}
```

配置RabbitMQ生产者：

```java
@Service
@Slf4j
public class MQSender {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void send(Object msg){
        log.info("发送消息:{}", msg);
        rabbitTemplate.convertAndSend(msg);
    }
}
```

配置消费者：

```java
@Service
@Slf4j
public class MQReceiver {
    @RabbitListener(queues = {"queue"})
    public void recv(Object msg){
        log.info("Recv Msg: {}", msg);

    }
}
```

### 接口优化

在redis中预减库存减少数据库的访问；通过内存标记等方法优化接口减少redis的访问；请求进入通过消息队列进行异步下单。

🔵预减库存+内存标记：

让对应的Controller实现`InitializingBean`类，并且实现`afterPropertiesSet()`方法，这个方法是在所有类加载完成后进行的初始化操作，即将数据库中的库存数量加载到redis中。每当一个秒杀请求进入的时候就使用redis的`decr`或者`incr`的原子操作进行库存-1。

当一个秒杀商品的库存变为0的时候，为了减少对访问redis，使用内存标记法来标记某个库存是否已经使用完毕，可以使用HashMap来进行标记。

```java
private Map<Long, Boolean> emptyStockMap = new HashMap<>();

@RequestMapping("/doSecKill")
public String doSecKill(Model model, User user, Long goodsId) throws JsonProcessingException {
    if (user == null) return "login";
    model.addAttribute("user", user);

    // 方案2：使用redis进行库存预减
    ValueOperations<String, Object> ops = redisTemplate.opsForValue();
    SeckillOrder seckillOrder = (SeckillOrder) ops.get("user:" + user.getId() + ":" + goodsId);
    if (seckillOrder != null) {
        model.addAttribute("error", RespBeanEnum.REPEAT_ERROR.getMsg());
        return "secKillFail";
    }

    // 内存标记，防止大量访问redis
    if (emptyStockMap.get(goodsId)){
        model.addAttribute("error", RespBeanEnum.EMPTY_STOCK.getMsg());
        return "secKillFail";
    }

    Long stock = ops.decrement("SecKillGoodsStockCount:" + goodsId);
    if (stock < 0) {
        emptyStockMap.put(goodsId, true);
        ops.increment("SecKillGoodsStockCount:" + goodsId);
        model.addAttribute("error", RespBeanEnum.EMPTY_STOCK.getMsg());
        return "secKillFail";
    }

	// 发布消息队列
    SeckillMessage seckillMessage = new SeckillMessage(user, goodsId);
    mqSender.sendSeckillMessage(seckillMessage);

    //        model.addAttribute("order", order);
    //        model.addAttribute("goods", goods);
    model.addAttribute("status", 0);
    return "orderDetail";
}

/**
* 系统初始化，将数据库库存加载到redis
*/
@Override
public void afterPropertiesSet() throws Exception {
    List<GoodsVo> list = goodsService.findGoodsVo();
    if (CollectionUtils.isEmpty(list)) return;
    list.forEach(vo -> {
        redisTemplate.opsForValue().set("SecKillGoodsStockCount:" + vo.getId(), vo.getStockCount());
        emptyStockMap.put(vo.getId(), false);
    });
}
```

🔵使用消息队列处理事件：

构建一个Topic类型的交换机，队列并且绑定

```java
@Configuration
public class RMQTopicConf {
    public static final String QUEUE = "seckillQueue";
    public static final String EXCHANGE = "sckillExchange";


    @Bean
    public Queue queue(){
        return new Queue(QUEUE);
    }

    @Bean
    public TopicExchange topicExchange(){
        return new TopicExchange(EXCHANGE);
    }

    @Bean
    public Binding binding(){
        return BindingBuilder.bind(queue()).to(topicExchange()).with("seckill.#");
    }
}
```

进行消息发送：

> RabbitMQ中存储的类型只能是`String`, `byte[]`以及序列化后的信息，不能直接传入对象，可以使用jackson来进行pojo和string类型的互相转换。

```java
@Service
@Slf4j
public class MQSender {
    @Autowired
    private RabbitTemplate rabbitTemplate;

    public void sendSeckillMessage(Object msg) throws JsonProcessingException {
        log.info("发送消息：{}", msg);
        String s = JSONUtils.Pojo2String(msg);
        rabbitTemplate.convertAndSend(RMQTopicConf.EXCHANGE, "seckill.message", s);
    }
}
```

进行消息接受：

```java
@Service
@Slf4j
public class MQReceiver {
    @Autowired
    private IGoodsService goodsService;
    @Autowired
    private RedisTemplate<String,Object> redisTemplate;
    @Autowired
    private IOrderService orderService;

    @RabbitListener(queues = {RMQTopicConf.QUEUE})
    public void recvSeckillMessage(String s) throws JsonProcessingException {
        log.info("收到消息{}", s);
        SeckillMessage msg = JSONUtils.String2Pojo(s, SeckillMessage.class);
        Long goodsId = msg.getGoodsId();
        User user = msg.getUser();
        GoodsVo goodsVo = goodsService.findGoodsVoById(goodsId);
        if (goodsVo.getStockCount() < 1)return;
        // 参考订单，判断是否重复抢购
        ValueOperations<String, Object> ops = redisTemplate.opsForValue();
        SeckillOrder seckillOrder = (SeckillOrder) ops.get("user:" + user.getId() + ":" + goodsId);
        if (seckillOrder != null)return;
        orderService.seckill(user,goodsVo);
    }
}
```

### Redis实现分布式锁

使用`setIfAbsent()`和lua脚本。。

### 安全优化

不直接显示抢购的URL，比如每个用户使用特定的URL，使用redis可以实现。
