---
title: SpringBoot整合redis
date: 2017-3-15 9:47:20
categories:
tags:
    - SpringBoot
    - redis
---

redis在系统架构中正在担任越来越重要的角色，由于其简单的集群配置，较为可靠的稳定性，还有快速的读取和插入性能，在系统中一般当作缓存或队列使用。SpringBoot中集成了jedis，能够方便的连接与操作redis，这里介绍一下具体的配置过程。

<!--more-->

#### 引入依赖
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

#### 配置redis连接参数，启动redis，默认端口6379
在application.yml中配置
```yml
spring:
  redis:
    database: 0
    host: localhost
    port: 6379
    pool:
      max-active: 8
      max-wait: -1
      min-idle: 0
    timeout: 0
    password:
```

#### 配置需要用到的Bean
```Java
@Configuration
public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new JdkSerializationRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());
        template.setHashValueSerializer(new JdkSerializationRedisSerializer());
        template.setConnectionFactory(factory);
        return template;
    }


    @Bean
    public ValueOperations<String, Object> valueOperations(RedisTemplate<String, Object> redisTemplate){
        return redisTemplate.opsForValue();
    }

    @Bean
    public ListOperations<String, Object> listOperations(RedisTemplate<String, Object> redisTemplate){
        return redisTemplate.opsForList();
    }

    @Bean
    public SetOperations<String, Object> setOperations(RedisTemplate<String, Object> redisTemplate){
        return redisTemplate.opsForSet();
    }

    @Bean
    public HashOperations<String, String, Object> hashOprations(RedisTemplate<String, Object> redisTemplate){
        return redisTemplate.opsForHash();
    }

    @Bean
    public ZSetOperations<String, Object> zstOprations(RedisTemplate<String, Object> redisTemplate){
        return redisTemplate.opsForZSet();
    }
}
```
这里说明一下各个实例的作用：

1.`RedisTemplate`是最重要的一个实例，其他具体操作的实例都来自于它。构造`RedisTemplate`时需要传入`RedisConnectionFactory`的实例，其中存储了redis连接的信息，而且SpringBoot已经帮我们实例化好了，直接用就可以。而且在装配时，方法参数可以直接引用装配好的实例（ByType方式），不需要使用@Autowired来引入。还需要设置`RedisTemplate`的键和值的序列化反序列化方式。这里采用JDK的序列化方式。

2.剩余实例的作用：
`ValueOperations`:操作String数据类型
`ListOperations`:操作list
`SetOperations`:操作Set
`HashOperations`:操作hash数据类型
`ZSetOperations`:操作SortedSet有序集合

#### 单元测试
```Java
@RunWith(SpringRunner.class)
@SpringBootTest
public class DemoApplicationTests {

	@Autowired
	RedisTemplate<String, Object> redisTemplate;

	@Autowired
	ValueOperations<String, Object> valueOperations;

	@Autowired
	ListOperations<String, Object> listOperations;

	@Autowired
	SetOperations<String, Object> setOperations;

	@Autowired
	HashOperations<String, String, Object> hashOprations;

	@Autowired
	ZSetOperations<String, Object> zSetOperations;

	@Test
	public void testRedisTemplate() {

		User user = new User(0,"小白");

		valueOperations.set("小红","18");

		valueOperations.set(user.getId().toString(),user);

		Assert.assertEquals("18",valueOperations.get("小红"));
		Assert.assertEquals(user,valueOperations.get("0"));


		User user1 = new User(1,"小李");

		listOperations.leftPush("users",user);

		listOperations.leftPush("users",user1);

		Assert.assertEquals(user,listOperations.rightPop("users"));
		Assert.assertEquals(user1,listOperations.rightPop("users"));


		setOperations.add("users", user);
		setOperations.add("users", user1);
		setOperations.add("users", user);
		System.out.println(setOperations.members("users"));
		redisTemplate.delete("users");

		zSetOperations.add("users",user,90.0);
		zSetOperations.add("users",user1,80.0);
        System.out.println(zSetOperations.range("users",0,-1));
        redisTemplate.delete("users");

        hashOprations.put("users",user.getId().toString(),user);
        hashOprations.put("users",user1.getId().toString(),user1);
        Assert.assertEquals(user,hashOprations.get("users",user.getId().toString()));
        Assert.assertEquals(user1,hashOprations.get("users",user1.getId().toString()));
        redisTemplate.delete("users");
	}

}
```
这里仅仅做了简单的插入和删除操作，更多的命令还需以后补充。我用断言进行了测试。需要注意，`redisTemplate`能够进行针对全局的操作，比如hasKey()判断键值是否存在，delete()删除一个键，expire()为一个键设置过期时间。
还发现一个问题就是如果一个键已经存在比如`"users"`,如果创建一个数据类型不同的`"users"`，那么会报错：
```
WRONGTYPE Operation against a key holding the wrong kind of value
```
这个在生产中必须注意。
有一个查询redis命令的地址，在此记录一下：http://doc.redisfans.com/