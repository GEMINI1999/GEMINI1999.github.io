---
layout: post
title: "SpringBoot集成mybatis、kafka、redis、websocket"
date: 2020-12-1 11:02
comments: true
tags: 
        - Spring
        - SpringBoot
        - kafka
        - redis
        - websocket
reward: true
toc: true
---
## pom.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.3.5.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>demo</name>
    <description>Demo project for Spring Boot</description>

    <properties>
        <java.version>1.8</java.version>
    </properties>

    <dependencies>
        <!--springboot redis依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <!--kafka依赖-->
        <dependency>
            <groupId>org.springframework.kafka</groupId>
            <artifactId>spring-kafka</artifactId>
        </dependency>
        <!--fastjson-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.46</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-websocket</artifactId>
        </dependency>
        <!--activemq-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-activemq</artifactId>
        </dependency>
        <!--消息队列连接池-->
        <dependency>
            <groupId>org.apache.activemq</groupId>
            <artifactId>activemq-pool</artifactId>
        </dependency>

        <!--配置关联-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.3</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
            <exclusions>
                <exclusion>
                    <groupId>org.junit.vintage</groupId>
                    <artifactId>junit-vintage-engine</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>

            <!-- mybatis generator 自动生成代码插件 -->
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.5</version>
                <configuration>
                    <!--配置文件的位置-->
                    <configurationFile>src/main/resources/generatorConfig.xml</configurationFile>
                    <overwrite>true</overwrite>
                    <verbose>true</verbose>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>

```
## 编写application.yml和application-dev.yml文件
### application.yml
```yml
spring:
  profiles:
    active: dev
```
### application-dev.yml
```yml
server:
  port: 8080

spring:
  datasource:
    username: root
    password: password
    url: jdbc:mysql://localhost:3306/address_book?serverTimezone=Asia/Shanghai&characterEncoding=utf8&useSSL=false
    driver-class-name: com.mysql.cj.jdbc.Driver
  redis:
    database: 0
    host: 127.0.0.1
    port: 6379
    timeout: 1000s
    password:
    jedis:
      pool:
        #最大空闲连接数
        max-idle: 500
        #最小空闲连接数
        min-idle: 50
        #等待可用连接的最大时间，负数为不限制
        max-wait: -1
        #最大活跃连接数，负数为不限制
        max-active: -1
  kafka:
    bootstrap-servers: 127.0.0.1:9092
    producer:
      # 发生错误后，消息重发的次数。
      retries: 0
      #当有多个消息需要被发送到同一个分区时，生产者会把它们放在同一个批次里。该参数指定了一个批次可以使用的内存大小，按照字节数计算。
      batch-size: 16384
      # 设置生产者内存缓冲区的大小。
      buffer-memory: 33554432
      # 键的序列化方式
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      # 值的序列化方式
      value-serializer: org.apache.kafka.common.serialization.StringSerializer
      # acks=0 ： 生产者在成功写入消息之前不会等待任何来自服务器的响应。
      # acks=1 ： 只要集群的首领节点收到消息，生产者就会收到一个来自服务器成功响应。
      # acks=all ：只有当所有参与复制的节点全部收到消息时，生产者才会收到一个来自服务器的成功响应。
      acks: 1
    consumer:
      # 自动提交的时间间隔 在spring boot 2.X 版本中这里采用的是值的类型为Duration 需要符合特定的格式，如1S,1M,2H,5D
      auto-commit-interval: 1S
      # 该属性指定了消费者在读取一个没有偏移量的分区或者偏移量无效的情况下该作何处理：
      # latest（默认值）在偏移量无效的情况下，消费者将从最新的记录开始读取数据（在消费者启动之后生成的记录）
      # earliest ：在偏移量无效的情况下，消费者将从起始位置读取分区的记录
      auto-offset-reset: earliest
      # 是否自动提交偏移量，默认值是true,为了避免出现重复数据和数据丢失，可以把它设置为false,然后手动提交偏移量
      enable-auto-commit: false
      # 键的反序列化方式
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      # 值的反序列化方式
      value-deserializer: org.apache.kafka.common.serialization.StringDeserializer
    listener:
      # 在侦听器容器中运行的线程数。
      concurrency: 5
      #listner负责ack，每调用一次，就立即commit
      ack-mode: manual_immediate
      missing-topics-fatal: false

  activemq:
    broker-url: tcp://127.0.0.1:61616
    user: admin
    password: admin
    pool:
      enabled: false
  cache:
    redis:
      time-to-live: -1

mybatis: #配置xml文件和实体类地址
  mapper-locations: classpath:mapping/*.xml
  type-aliases-package: com.example.entity
  configuration:
    #开启驼峰命名
    map-underscore-to-camel-case: true

#showSql
logging:
  level:
    com:
      example:
        mapper : debug
```
## Mybatis集成
### 在entity文件夹下编写实体类
```java
@Data
@Component
public class Address implements Serializable {
    private Integer id;
    private String name;
    private String phoneNumber;
    private String address;
    private String emailAddress;
    private String remarks;
}
```
### 在resources资源文件夹下的mapping文件夹下创建AddressMapper.xml文件
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.dao.AddressMapper">
  <resultMap id="BaseResultMap" type="com.example.entity.Address">
    <id column="id" jdbcType="INTEGER" property="id" />
    <result column="name" jdbcType="VARCHAR" property="name" />
    <result column="phone_number" jdbcType="VARCHAR" property="phoneNumber" />
    <result column="address" jdbcType="VARCHAR" property="address" />
    <result column="email_address" jdbcType="VARCHAR" property="emailAddress" />
    <result column="remarks" jdbcType="VARCHAR" property="remarks" />
  </resultMap>
  <sql id="Base_Column_List">
    id, name, phone_number, address, email_address, remarks
  </sql>
  <select id="selectByPrimaryKey" parameterType="java.lang.Integer" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List" />
    from address_book
    where id = #{id,jdbcType=INTEGER}
  </select>
  <delete id="deleteByPrimaryKey" parameterType="java.lang.Integer">
    delete from address_book
    where id = #{id,jdbcType=INTEGER}
  </delete>
  <insert id="insert" parameterType="com.example.entity.Address">
    insert into address_book (id, name, phone_number, 
      address, email_address, remarks
      )
    values (#{id,jdbcType=INTEGER}, #{name,jdbcType=VARCHAR}, #{phoneNumber,jdbcType=VARCHAR}, 
      #{address,jdbcType=VARCHAR}, #{emailAddress,jdbcType=VARCHAR}, #{remarks,jdbcType=VARCHAR}
      )
  </insert>
  <insert id="insertSelective" parameterType="com.example.entity.Address">
    insert into address_book
    <trim prefix="(" suffix=")" suffixOverrides=",">
      <if test="id != null">
        id,
      </if>
      <if test="name != null">
        name,
      </if>
      <if test="phoneNumber != null">
        phone_number,
      </if>
      <if test="address != null">
        address,
      </if>
      <if test="emailAddress != null">
        email_address,
      </if>
      <if test="remarks != null">
        remarks,
      </if>
    </trim>
    <trim prefix="values (" suffix=")" suffixOverrides=",">
      <if test="id != null">
        #{id,jdbcType=INTEGER},
      </if>
      <if test="name != null">
        #{name,jdbcType=VARCHAR},
      </if>
      <if test="phoneNumber != null">
        #{phoneNumber,jdbcType=VARCHAR},
      </if>
      <if test="address != null">
        #{address,jdbcType=VARCHAR},
      </if>
      <if test="emailAddress != null">
        #{emailAddress,jdbcType=VARCHAR},
      </if>
      <if test="remarks != null">
        #{remarks,jdbcType=VARCHAR},
      </if>
    </trim>
  </insert>
  <update id="updateByPrimaryKeySelective" parameterType="com.example.entity.Address">
    update address_book
    <set>
      <if test="name != null">
        name = #{name,jdbcType=VARCHAR},
      </if>
      <if test="phoneNumber != null">
        phone_number = #{phoneNumber,jdbcType=VARCHAR},
      </if>
      <if test="address != null">
        address = #{address,jdbcType=VARCHAR},
      </if>
      <if test="emailAddress != null">
        email_address = #{emailAddress,jdbcType=VARCHAR},
      </if>
      <if test="remarks != null">
        remarks = #{remarks,jdbcType=VARCHAR},
      </if>
    </set>
    where id = #{id,jdbcType=INTEGER}
  </update>
  <update id="updateByPrimaryKey" parameterType="com.example.entity.Address">
    update address_book
    set name = #{name,jdbcType=VARCHAR},
      phone_number = #{phoneNumber,jdbcType=VARCHAR},
      address = #{address,jdbcType=VARCHAR},
      email_address = #{emailAddress,jdbcType=VARCHAR},
      remarks = #{remarks,jdbcType=VARCHAR}
    where id = #{id,jdbcType=INTEGER}
  </update>
</mapper>
```
### 创建dao文件夹,编写dao/mapper文件
```java
@Repository
public interface AddressMapper {
    int deleteByPrimaryKey(Integer id);
    int insert(Address record);
    int insertSelective(Address record);
    Address selectByPrimaryKey(Integer id);
    int updateByPrimaryKeySelective(Address record);
    int updateByPrimaryKey(Address record);
}
```
### 创建service文件夹编写实现类和接口（这里由于是Demo并未写接口）
```java
@Service
public class AddressService {
    @Autowired
    AddressMapper addressMapper;
    public Address sel(int id){
        return addressMapper.selectByPrimaryKey(id);
    }

}
```
### 创建controller文件夹编写接口
```java
@Slf4j
@RestController
@RequestMapping("/testBoot")
public class AddressController {
    @Autowired
    private AddressService addressService;

    @RequestMapping("/getAddress/{id}")
    public String getAddress(@PathVariable int id){
        return addressService.sel(id).toString();
    }
}
```
## Redis集成
### 配置Redis
```java
@Configuration
@EnableCaching//启用缓存，这个注解很重要；
//继承CachingConfigurerSupport，为了自定义生成KEY的策略。可以不继承。
public class RedisConfig extends CachingConfigurerSupport {
    @Bean(name = "redisTemplate")
    public RedisTemplate<String,Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory);
        //序列化对象
        //字符串序列化
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        //jdk序列化
        JdkSerializationRedisSerializer jdkSerializationRedisSerializer = new JdkSerializationRedisSerializer();
        //Json序列化
        GenericJackson2JsonRedisSerializer genericJackson2JsonRedisSerializer = new GenericJackson2JsonRedisSerializer();

        //使用StringRedisSerializer来序列化和反序列化redis的key值
        template.setKeySerializer(stringRedisSerializer);
        // hash的key也采用String的序列化方式
        template.setHashKeySerializer(stringRedisSerializer);
        // value序列化方式采用jackson
        template.setValueSerializer(genericJackson2JsonRedisSerializer);
        // hash的value序列化方式采用jackson
        template.setHashValueSerializer(genericJackson2JsonRedisSerializer);

        template.afterPropertiesSet();
        return template;
    }
}

```
### 调用方法
```java
@Autowired
private RedisTemplate<String, Object> redisTemplate;

redisTemplate.opsForValue().set(key, value);
redisTemplate.opsForValue().get(key);
```
## Kafka集成
### 消费者
```java
@Component
@Slf4j
@EnableKafka
public class KafkaConsumer {

    @KafkaListener(topics = KafkaProducer.TOPIC_TEST, groupId = KafkaProducer.TOPIC_GROUP1)
    public void topic_test(ConsumerRecord<?, ?> record, Acknowledgment ack, @Header(KafkaHeaders.RECEIVED_TOPIC) String topic) {

        Optional message = Optional.ofNullable(record.value());
        if (message.isPresent()) {
            Object msg = message.get();
            log.info("topic_test 消费了： Topic:" + topic + ",Message:" + msg);
            ack.acknowledge();
        }
    }

    @KafkaListener(topics = KafkaProducer.TOPIC_TEST, groupId = KafkaProducer.TOPIC_GROUP2)
    public void topic_test1(ConsumerRecord<?, ?> record, Acknowledgment ack, @Header(KafkaHeaders.RECEIVED_TOPIC) String topic) {

        Optional message = Optional.ofNullable(record.value());
        if (message.isPresent()) {
            Object msg = message.get();
            log.info("topic_test1 消费了： Topic:" + topic + ",Message:" + msg);
            ack.acknowledge();
        }
    }
}
```
### 生产者
```java
@Component
@Slf4j
public class KafkaProducer {
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    //自定义topic
    public static final String TOPIC_TEST = "topic.test";
    public static final String TOPIC_GROUP1 = "topic.group1";
    public static final String TOPIC_GROUP2 = "topic.group2";

    public void send(Object obj) {
        String obj2String = JSONObject.toJSONString(obj);
        log.info("准备发送消息为：{}", obj2String);
        //发送消息
        ListenableFuture<SendResult<String, Object>> future = kafkaTemplate.send(TOPIC_TEST, obj);
        future.addCallback(new ListenableFutureCallback<SendResult<String, Object>>() {
            @Override
            public void onFailure(Throwable throwable) {
                //发送失败的处理
                log.info(TOPIC_TEST + " - 生产者 发送消息失败：" + throwable.getMessage());
            }

            @Override
            public void onSuccess(SendResult<String, Object> stringObjectSendResult) {
                //成功的处理
                log.info(TOPIC_TEST + " - 生产者 发送消息成功：" + stringObjectSendResult.toString());
            }
        });
    }
}

```
## Websocket集成
### 配置
```java
@Configuration
public class WebSocketConfig {
    @Bean
    public ServerEndpointExporter serverEndpointExporter(){
        return new ServerEndpointExporter();
    }
}

```
### 具体用例
> 自己发送给所有人(不包括自己)
```java
/**
 *
 * 前后端交互的类实现消息的接收推送(自己发送给所有人(不包括自己))
 *
 * @ServerEndpoint(value = "/test/oneToMany") 前端通过此URI 和后端交互，建立连接
 */
@Slf4j
@ServerEndpoint(value = "/test/oneToMany")
@Component
public class OneToManyWebSocket {

    /** 记录当前在线连接数 */
    private static AtomicInteger onlineCount = new AtomicInteger(0);

    /** 存放所有在线的客户端 */
    private static Map<String, Session> clients = new ConcurrentHashMap<>();

    /**
     * 连接建立成功调用的方法
     */
    @OnOpen
    public void onOpen(Session session) {
        onlineCount.incrementAndGet(); // 在线数加1
        clients.put(session.getId(), session);
        log.info("有新连接加入：{}，当前在线人数为：{}", session.getId(), onlineCount.get());
    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose(Session session) {
        onlineCount.decrementAndGet(); // 在线数减1
        clients.remove(session.getId());
        log.info("有一连接关闭：{}，当前在线人数为：{}", session.getId(), onlineCount.get());
    }

    /**
     * 收到客户端消息后调用的方法
     *
     * @param message
     *            客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        log.info("服务端收到客户端[{}]的消息:{}", session.getId(), message);
        this.sendMessage(message, session);
    }

    @OnError
    public void onError(Session session, Throwable error) {
        log.error("发生错误");
        error.printStackTrace();
    }

    /**
     * 群发消息
     *
     * @param message
     *            消息内容
     */
    private void sendMessage(String message, Session fromSession) {
        for (Map.Entry<String, Session> sessionEntry : clients.entrySet()) {
            Session toSession = sessionEntry.getValue();
            // 排除掉自己
            if (!fromSession.getId().equals(toSession.getId())) {
                log.info("服务端给客户端[{}]发送消息{}", toSession.getId(), message);
                toSession.getAsyncRemote().sendText(message);
            }
        }
    }
}
```
> 自己发送给另一个人
```java
/**
 * 前后端交互的类实现消息的接收推送(自己发送给另一个人)
 *
 * @ServerEndpoint(value = "/test/oneToOne") 前端通过此URI 和后端交互，建立连接
 */
@Slf4j
@ServerEndpoint(value = "/test/oneToOne")
@Component
public class OneToOneWebSocket {

    /** 记录当前在线连接数 */
    private static AtomicInteger onlineCount = new AtomicInteger(0);

    /** 存放所有在线的客户端 */
    private static Map<String, Session> clients = new ConcurrentHashMap<>();

    /**
     * 连接建立成功调用的方法
     */
    @OnOpen
    public void onOpen(Session session) {
        onlineCount.incrementAndGet(); // 在线数加1
        clients.put(session.getId(), session);
        log.info("有新连接加入：{}，当前在线人数为：{}", session.getId(), onlineCount.get());
    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose(Session session) {
        onlineCount.decrementAndGet(); // 在线数减1
        clients.remove(session.getId());
        log.info("有一连接关闭：{}，当前在线人数为：{}", session.getId(), onlineCount.get());
    }

    /**
     * 收到客户端消息后调用的方法
     *
     * @param message
     *            客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        log.info("服务端收到客户端[{}]的消息[{}]", session.getId(), message);
        try {
            MyMessage myMessage = JSON.parseObject(message, MyMessage.class);
            if (myMessage != null) {
                Session toSession = clients.get(myMessage.getUserId());
                if (toSession != null) {
                    this.sendMessage(myMessage.getMessage(), toSession);
                }
            }
        } catch (Exception e) {
            log.error("解析失败：{}", e);
        }
    }

    @OnError
    public void onError(Session session, Throwable error) {
        log.error("发生错误");
        error.printStackTrace();
    }

    /**
     * 服务端发送消息给客户端
     */
    private void sendMessage(String message, Session toSession) {
        try {
            log.info("服务端给客户端[{}]发送消息[{}]", toSession.getId(), message);
            toSession.getBasicRemote().sendText(message);
        } catch (Exception e) {
            log.error("服务端发送消息给客户端失败：{}", e);
        }
    }

}

```
> 自己发送给自己
```java
/**
 * 前后端交互的类实现消息的接收推送(自己发送给自己)
 *
 * @ServerEndpoint(value = "/test/one") 前端通过此URI和后端交互，建立连接
 */
@Slf4j
@ServerEndpoint(value = "/test/one")
@Component
public class OneWebSocket {

    /**
     * 记录当前在线连接数
     */
    private static AtomicInteger onlineCount = new AtomicInteger(0);

    /**
     * 连接建立成功调用的方法
     */
    @OnOpen
    public void onOpen(Session session) {
        onlineCount.incrementAndGet(); // 在线数加1
        log.info("有新连接加入：{}，当前在线人数为：{}", session.getId(), onlineCount.get());
    }

    /**
     * 连接关闭调用的方法
     */
    @OnClose
    public void onClose(Session session) {
        onlineCount.decrementAndGet(); // 在线数减1
        log.info("有一连接关闭：{}，当前在线人数为：{}", session.getId(), onlineCount.get());
    }

    /**
     * 收到客户端消息后调用的方法
     *
     * @param message 客户端发送过来的消息
     */
    @OnMessage
    public void onMessage(String message, Session session) {
        log.info("服务端收到客户端[{}]的消息:{}", session.getId(), message);
        this.sendMessage("Hello, " + message, session);
    }

    @OnError
    public void onError(Session session, Throwable error) {
        log.error("发生错误");
        error.printStackTrace();
    }

    /**
     * 服务端发送消息给客户端
     */
    private void sendMessage(String message, Session toSession) {
        try {
            log.info("服务端给客户端[{}]发送消息{}", toSession.getId(), message);
            toSession.getBasicRemote().sendText(message);
        } catch (Exception e) {
            log.error("服务端发送消息给客户端失败：{}", e);
        }
    }
}
```
### 前端测试代码
```html
<!DOCTYPE HTML>
<html>
<head>
    <title>My WebSocket</title>
</head>

<body>
<input id="text" type="text" />
<button onclick="send()">Send</button>
<button onclick="closeWebSocket()">Close</button>
<div id="message"></div>
</body>

<script type="text/javascript">
    var websocket = null;

    //判断当前浏览器是否支持WebSocket, 主要此处要更换为自己的地址
    if ('WebSocket' in window) {
        websocket = new WebSocket("ws://localhost:8080/test/oneToOne");
    } else {
        alert('Not support websocket')
    }

    //连接发生错误的回调方法
    websocket.onerror = function() {
        setMessageInnerHTML("error");
    };

    //连接成功建立的回调方法
    websocket.onopen = function(event) {
        //setMessageInnerHTML("open");
    }

    //接收到消息的回调方法
    websocket.onmessage = function(event) {
        setMessageInnerHTML(event.data);
    }

    //连接关闭的回调方法
    websocket.onclose = function() {
        setMessageInnerHTML("close");
    }

    //监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
    window.onbeforeunload = function() {
        websocket.close();
    }

    //将消息显示在网页上
    function setMessageInnerHTML(innerHTML) {
        document.getElementById('message').innerHTML += innerHTML + '<br/>';
    }

    //关闭连接
    function closeWebSocket() {
        websocket.close();
    }

    //发送消息
    function send() {
        var message = document.getElementById('text').value;
        websocket.send(message);
    }
</script>
</html>
```
### kafka调用接口
```java
@RequestMapping("/kafka")
@Slf4j
@RestController
public class KafkaController {

    @Autowired
    private KafkaProducer kafkaProducer;

    @GetMapping(value = "/send")
    @ResponseBody
    @Transactional(rollbackFor = Exception.class)
    public void sendMsg(){
        kafkaProducer.send("this is a test kafka topic message!");
    }
}
```

