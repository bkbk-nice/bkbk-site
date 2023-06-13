---
slug: springcloud
title:  SpringCloud 
authors: [bkbk]
tags: [SpringCloud]
---
 
 


:::caution 目录
**项目结构**   

**SpringCloud配置**   

**nacos、Ribbon配置**   

**远程调用OpenFeign**  

**sentinel**

**openfeign-fallback**
::: -->
<!--truncate-->

<!-- ``` js title="目录" {1-4}
项目结构
SpringCloud配置
Ribbon配置
远程调用OpenFeign
``` -->



[spring官网](https://spring.io)  

## 项目结构
``` xml title="school-parent 版本依赖与管理"
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.3.12.RELEASE</version>
    <relativePath></relativePath>
</parent>

<groupId>com.bkbk</groupId>
<artifactId>school-parent</artifactId>
<version>1.0-SNAPSHOT</version> 
<modules>
    <module>student-service</module>
    <module>group-service</module>
</modules>
<packaging>pom</packaging>  
<dependencies>  
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.5.0</version>
    </dependency> 
</dependencies>


<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-alibaba-dependencies</artifactId>
            <version>2.2.9.RELEASE</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Hoxton.SR12</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency> 
    </dependencies>
</dependencyManagement>
```



``` xml title="子模块" {1-4}
 引入依赖的公共模块 
 Nacos属于Springcloudalibaba的依赖 
 Openfeign属于SpringCloud的依赖 
 版本已经在父项目中规定完成
<parent>
    <artifactId>school-parent</artifactId>
    <groupId>com.bkbk</groupId>
    <version>1.0-SNAPSHOT</version>
</parent>
<modelVersion>4.0.0</modelVersion> 
<artifactId>student-service</artifactId>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
<dependency>
    <groupId>com.bkbk</groupId>
    <artifactId>commons</artifactId>
    <version>1.0-SNAPSHOT</version>
    <scope>compile</scope>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency> 
```

## SpringCloud、Ribbon配置
``` yml title="配置"
server:
  port: 6003
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/school?serverTimezone=GMT%2B8&characterEncoding=utf8&useSSL=true
    username: root
    password: root # 你的数据库密码
  cloud:
    nacos:
      server-addr: localhost:8848
  application:
    name: student-service
# ribbon规则 (nacos中包含) 需要rpc时添加
student-service:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```


## 远程调用OpenFeign
``` js title="调用"
//第一种 
//配置类
@Bean
@LoadBalanced
public RestTemplate restTemplate(){
    RestTemplate restTemplate = new RestTemplate();
    return restTemplate;
}
@LoadBalanced 注解原理(使用ribbon)
这里主要的逻辑就是给 RestTemplate 增加拦截器，在请求之前对请求的地址进行替换，或者根据具体的负载策略选择服务地址，然后再去调用

@Autowired
private RestTemplate restTemplate;
ResultVo resultVo= restTemplate.getForObject("http://student-service/student/getById?id=1",ResultVo.class);
System.out.println(resultVo.getData());

//第二种
@Autowired
private DiscoveryClient discoveryClient;
List<ServiceInstance> instances = discoveryClient.getInstances("student-service");
for (ServiceInstance instance : instances) {
    System.out.println(  instance.getHost());
    System.out.println(   instance.getUri() );
}

//第三种 OpenFeign
@EnableFeignClients //启动类注解
//编写接口
@FeignClient(name="student-service")
public interface StudentFeignClient { 
    @GetMapping("/student/getById")
    ResultVo getById(@RequestParam("id") Integer id);  
}
//调用
for (Integer id : group.getList()) {
    ResultVo resultVo = studentFeignClient.getById(id);
    if(resultVo.getStatus()!=0){
        System.out.println("id " + id + " not exist");
    }else{
        System.out.println(resultVo.getData());
    }
} 
```
:::tip 
**同一个服务修改port可以启动多次，在启动设置中允许多个运行**  
::: -->


## sentinel

``` xml title="导入依赖,版本受父项目管理"
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>  
```

``` yml title="yml配置"
spring: 
  cloud:
    nacos:
      server-addr: localhost:8848
    sentinel:
      transport:
        dashboard: localhost:8080
```
**在dashboard中设置规则**

## openfeign_fallback
``` yml title="yml配置" {1}
在调用失败后调用fallback方法
feign:
  sentinel:
    enabled: true
```

``` java title="fallback类实现接口" 
@FeignClient(name="student-service", fallback= StudentFeignClientFallback.class)
public interface StudentFeignClient { 
    @GetMapping("/student/getById")
    ResultVo getById(@RequestParam("id") Integer id);  
}

@Component
public class StudentFeignClientFallback implements StudentFeignClient {
    @Override
    public ResultVo getById(Integer id) {
        System.err.println("断路");
        return ResultVo.fail("请求失败");
    }
} 
```