### springboot + redis 集成测试

项目地址

~~~htt
https://github.com/tomato2009/spring/tree/master/spring-redis/springboot-redis-01
~~~



#### 1 springboot中redis相关配置

#####1.1 在pom中配置redis的相关依赖包 

~~~xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-redis</artifactId>
    <version>1.3.8.RELEASE</version>
</dependency>
~~~



##### 1.2 在application.yml里面配置redis的配置项。 

~~~xml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/micro-credit?useUnicode=true&characterEncoding=utf8
    username: root
    password: root
    type: com.alibaba.druid.pool.DruidDataSource
    driver-class-name: com.mysql.jdbc.Driver
    filters: stat,wall,log4j
    maxActive: 20
    initialSize: 1
    maxWait: 60000
    minIdle: 1
    timeBetweenEvictionRunsMillis: 60000
    minEvictableIdleTimeMillis: 300000
    validationQuery: select 'x'
    testWhileIdle: true
    testOnBorrow: false
    testOnReturn: false
    poolPreparedStatements: true
    maxOpenPreparedStatements: 20
    druid:
      filter:
        slf4j:
          enabled: true
          statement-create-after-log-enabled: false
          statement-close-after-log-enabled: false
          result-set-open-after-log-enabled: false
          result-set-close-after-log-enabled: false
  redis:
          host: 127.0.0.1
          port: 6379
          password: 123456
          jedis:
            pool:
              max-active: 100
              max-idle: 10
              max-wait: 100000
          timeout: 0
~~~

##### 1.3  springboot中redis相关类

1. 项目操作redis是使用的RedisTemplate方式，另外还可以完全使用JedisPool和Jedis来操作redis。整合的内容也是从网上收集整合而来，网上整合的方式和方法非常的多，有使用注解形式的，有使用Jackson2JsonRedisSerializer来序列化和反序列化key value的值等等，很多很多。这里使用的是我认为比较容易理解和掌握的，基于JedisPool配置，使用RedisTemplate来操作redis的方式。

- redis单独放在一个包redis里，在包里先创建RedisConfig.java文件。

~~~java
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.jedis.JedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import redis.clients.jedis.JedisPoolConfig;

@Configuration
@EnableAutoConfiguration
public class RedisConfig {

    @Bean
    @ConfigurationProperties(prefix = "spring.redis.pool")
    public JedisPoolConfig getRedisConfig(){
        JedisPoolConfig config = new JedisPoolConfig();
        return config;
    }

    @Bean
    @ConfigurationProperties(prefix = "spring.redis")
    public JedisConnectionFactory getConnectionFactory() {
        JedisConnectionFactory factory = new JedisConnectionFactory();
        factory.setUsePool(true);
        JedisPoolConfig config = getRedisConfig();
        factory.setPoolConfig(config);
        return factory;
    }

    @Bean
    public RedisTemplate<?, ?> getRedisTemplate() {
        JedisConnectionFactory factory = getConnectionFactory();
        RedisTemplate<?, ?> template = new StringRedisTemplate(factory);
        return template;
    }

}
~~~



 以上三个方法分别为获取JedisPoolConfig配置、获取JedisConnectionFactory工厂和获取RedisTemplate模板。
 **@Configuration** 注解是用于定义配置类，可替换xml配置文件，被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。
 **@EnableAutoConfiguration** 注解是启用Spring应用程序上下文的自动配置，尝试猜测和配置您可能需要的bean。自动配置类通常基于类路径和定义的bean应用。
 **@ConfigurationProperties** 注解是用于读取配置文件的信息，在这里是读取配置在yml里的redis的相关配置项。
 **@Bean** 注解用在方法上，告诉Spring容器，你可以从下面这个方法中拿到一个Bean

- 在包里创建RedisService接口，在这个接口定义了一些redis的基本操作。在这里我把所有存取操作都封装成了基于json字符串完成，就没有对于list或者对于object等单独定义方法。所有的数据类型的存储都由代码转换成json字符串方式进行。所以这里就只有四个方法。

 

**RedisService.java**

~~~java
public interface RedisService {

    /**
     * set存数据
     * @param key
     * @param value
     * @return
     */
    boolean set(String key, String value);

    /**
     * get获取数据
     * @param key
     * @return
     */
    String get(String key);

    /**
     * 设置有效天数
     * @param key
     * @param expire
     * @return
     */
    boolean expire(String key, long expire);

    /**
     * 移除数据
     * @param key
     * @return
     */
    boolean remove(String key);

}
~~~

在这里execute()方法具体的底层没有去研究，只知道这样能实现对于redis数据的操作。
 redis保存的数据会在内存和硬盘上存储，所以需要做序列化；这个里面使用的StringRedisSerializer来做序列化，不过这个方式的泛型指定的是String，只能传String进来。所以项目中采用json字符串做redis的交互。

**到此，redis在springboot中的整合已经完毕，下面就来测试使用一下。**

 

 ##### 1.4 springboot项目中使用redis

在这里就直接使用springboot项目中自带的单元测试类RedisApplicationTests进行测试。

**SpringbootApplicationTests.java**

~~~java
import com.alibaba.fastjson.JSONObject;
import com.tomatoman.redis.service.RedisService;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.util.ArrayList;
import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest
public class RedisApplicationTests {

	private JSONObject json = new JSONObject();

	@Autowired
	private RedisService redisService;

	@Test
	public void contextLoads() throws Exception {
	}


	/**
	 * 插入字符串
	 */
	@Test
	public void setString() {
		redisService.set("redis_string_test", "springboot redis test");
	}

	/**
	 * 获取字符串
	 */
	@Test
	public void getString() {
		String result = redisService.get("redis_string_test");
		System.out.println(result);
	}

	/**
	 * 插入对象
	 */
	@Test
	public void setObject() {
		Person person = new Person("person", "male");
		redisService.set("redis_obj_test", json.toJSONString(person));
	}

	/**
	 * 获取对象
	 */
	@Test
	public void getObject() {
		String result = redisService.get("redis_obj_test");
		Person person = json.parseObject(result, Person.class);
		System.out.println(json.toJSONString(person));
	}

	/**
	 * 插入对象List
	 */
	@Test
	public void setList() {
		Person person1 = new Person("person1", "male");
		Person person2 = new Person("person2", "female");
		Person person3 = new Person("person3", "male");
		List<Person> list = new ArrayList<>();
		list.add(person1);
		list.add(person2);
		list.add(person3);
		redisService.set("redis_list_test", json.toJSONString(list));
	}

	/**
	 * 获取list
	 */
	@Test
	public void getList() {
		String result = redisService.get("redis_list_test");
		List<String> list = json.parseArray(result, String.class);
		System.out.println(list);
	}

	@Test
	public void remove() {
		redisService.remove("redis_test");
	}

}

class Person {
	private String name;
	private String sex;

	public Person() {

	}

	public Person(String name, String sex) {
		this.name = name;
		this.sex = sex;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public String getSex() {
		return sex;
	}

	public void setSex(String sex) {
		this.sex = sex;
	}
}
~~~



在这里先是用@Autowired注解把redisService注入进来，然后由于是使用json字符串进行交互，所以引入fastjson的JSONObject类。然后为了方便，直接在这个测试类里面加了一个Person的内部类。

一共测试了：对于string类型的存取，对于object类型的存取，对于list类型的存取，其实本质都是转成了json字符串。还有就是根据key来执行remove操作。

![QQæªå¾20180618172457.png](https://github.com/tomato2009/note/blob/master/redis/pic_springboot-redis-01/QQ%E6%88%AA%E5%9B%BE20180618172457.png?raw=true) 



![QQæªå¾20180618172607.png](https://github.com/tomato2009/note/blob/master/redis/pic_springboot-redis-01/QQ%E6%88%AA%E5%9B%BE20180618172607.png?raw=true) 



![QQæªå¾20180618172619.png](https://github.com/tomato2009/note/blob/master/redis/pic_springboot-redis-01/QQ%E6%88%AA%E5%9B%BE20180618172619.png?raw=true) 



 

 

 

 

 

 

 

 

 

 