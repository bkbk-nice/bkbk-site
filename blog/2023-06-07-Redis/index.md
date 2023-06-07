---
slug: redis
title:  Redis 
authors: [bkbk]
tags: [Redis]
---

##  Redis


Redis(Remote Dictionary seve)，即远程字典服务，
是一个开源的使用ANSIC语言纳写、支持网络、可基于内存亦可特久化的日志型、Key-value库，
Redis是一种面向"Key-Value"类型的内存数据库，可以满足我们对海量数据的快速读写要求- Redis的Key只能是字符串，value可以是多种类型  
Redis特点  
 高性能:Redisi读的速度是11w次/s，写的速度是8.1w次/s    
 原子性:保证数据的准确性  
 持久存储:支特两种持久化方案，RDB和AOF  
 支持主从模式和集群模式，(3.0开始支持集群)  
- Redis是—个单线程的服务(6.x的新特性开始支持多线程)
- Redlis是一个NoSQL数据库
- Redis的应用场景
https;//www.jianshu.com/p/40dbc78711c8

<!--truncate-->
:::info  net
**本机网络需要配置虚拟机网络适配器并配置网段**   
:::


 



<!-- :::note  
**content**  
:::

:::tip 
**content**  
::: -->





<!-- :::caution 
**content**  
:::

:::danger  
**content**  
::: -->



 
``` xml title="导入依赖"
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

 
``` yml title="yml"
spring:
  redis: 
      database: 0
      host: 192.168.78.134
      port: 6379
      password:
      jedis:
        pool:
          #最大连接数
          max-active: 8
          #最大阻塞等待时间(负数表示没限制)
          max-wait: -1
          #最大空闲
          max-idle: 8
          #最小空闲
          min-idle: 0
          #连接超时时间
      timeout: 10000
```

 
``` js  title="redisconfig"
@Configuration 
public class RedisConfig extends CachingConfigurerSupport {
    @Bean(name = "redisTemplate")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory){

        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        //参照StringRedisTemplate内部实现指定序列化器
        redisTemplate.setConnectionFactory(redisConnectionFactory);
        redisTemplate.setKeySerializer(keySerializer());
        redisTemplate.setHashKeySerializer(keySerializer());
        redisTemplate.setValueSerializer(valueSerializer());
        redisTemplate.setHashValueSerializer(valueSerializer());
        return redisTemplate;
    }

    private RedisSerializer<String> keySerializer(){
        return new StringRedisSerializer();
    }

    //使用Jackson序列化器
    private RedisSerializer<Object> valueSerializer(){
        return new GenericJackson2JsonRedisSerializer();
    }
}
```

  
``` java title="redistemplate" 

  // highlight-start
   @Autowired
    private RedisTemplate redisTemplate;
  // highlight-end 

    @GetMapping("/all")
    public ResultVo getAllStudent(){ 

        ResultVo all = new ResultVo();
        if(redisTemplate.opsForValue().get("all")==null){
            all = studentService.getAllStudent();
           // redisTemplate.opsForValue().set("all", all ,6, TimeUnit.SECONDS);
            redisTemplate.opsForValue().set("all", all );
            System.out.println("mysql");
            System.out.println(all);
        } else{
            System.out.println("redis");
            System.out.println(redisTemplate.opsForValue().get("all"));
        } 
        return studentService.getAllStudent();
    }
```
## spring-cache

 
``` xml title="导入依赖"
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

``` java title="添加启动类注解"
@SpringBootApplication
@MapperScan("com.bkbk.school.mapper")
@EnableCaching
public class SchoolApplication {

    public static void main(String[] args) {
        SpringApplication.run(SchoolApplication.class, args);
    }

}
```


```java title="crud" 
     
    @CacheEvict("all-stu")
    @GetMapping("/delcache")
    public ResultVo delchahe() {
        return ResultVo.success();
    } 

    @CachePut("all-stu")
    @GetMapping("/putcache")
    public ResultVo putchahe() {
       return ResultVo.success();
    } 


    @Cacheable("all-stu")
    @GetMapping("/all")
    public ResultVo getAllStudent(){  
        return studentService.getAllStudent();
    }

```



 
 