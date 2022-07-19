---
title: spring cloud nacos config
date: 2022-07-19 18:35:00
tags: spring cloud nacos
---

版本对应关系参考：[版本说明 Wiki](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/版本说明)

demo使用版本（后面学习spring-cloud 也是用该版本）。

- spring-boot `2.4.2`
- spring-cloud `2020.0.1`
- spring-cloud-alibaba `2021.1`

[nacos-config](https://github.com/alibaba/spring-cloud-alibaba/blob/2.2.x/spring-cloud-alibaba-docs/src/main/asciidoc-zh/nacos-config.adoc)

[官方文档](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)

[参考项目](https://github.com/oneredka/learn_springcloud/tree/master/spring-cloud-nacos-config)

## 配置管理

1. `[application.properties`](application.properties)  添加配置

```yaml
server.port=8222
spring.application.name=icarus
spring.cloud.nacos.config.server-addr=127.0.0.1:8848
```

说明：之所以需要配置 `spring.application.name` ，是因为它是构成 Nacos 配置管理 `dataId`字段的一部分。

在 `Nacos Spring Cloud` 中，`dataId` 的完整格式如下：

```
${prefix}-${spring.profiles.active}.${file-extension}
```

- `prefix` 默认为 `spring.application.name` 的值，也可以通过配置项 `spring.cloud.nacos.config.prefix`来配置。
- `spring.profiles.active` 即为当前环境对应的 profile，详情可以参考 [Spring Boot文档](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html#boot-features-profiles)。 **注意：当 `spring.profiles.active` 为空时，对应的连接符 `` 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`**
- `file-exetension` 为配置内容的数据格式，可以通过配置项 `spring.cloud.nacos.config.file-extension` 来配置。目前只支持 `properties` 和 `yaml` 类型。

1. 添加依赖

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
   </dependency>
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
       <version>2021.1</version>
   </dependency>
   <!-- 区别于官方demo, 基于当前使用的spring boot, cloud 版本，需要添加 -->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-bootstrap</artifactId>
       <version>3.0.1</version>
   </dependency>
   ```

2. 通过 Spring Cloud 原生注解 `@RefreshScope` 实现配置自动更新

   ```java
   @RestController
   @RequestMapping("/config")
   @RefreshScope
   public class ConfigController {
   
       @Value("${publicKey:123456}")
       private String publicKey;
   
       /**
        * <http://localhost:8222/config/get>
        */
   
       @RequestMapping("/get")
       public String get() {
           return publicKey;
       }
   }
   ```

3. 访问 `http://localhost:8222/config/get`，此时返回内容为 `123456`

4. 发布配置

   ```java
   curl -X POST "<http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=icarus.properties&group=DEFAULT_GROUP&content=publicKey=147258>"
   ```

5. 再次访问 `http://localhost:8222/config/get`，此时返回内容为 `147258`，说明程序中的 `publicKey` 读取到了远程配置，值已经被动态更新了

该demo只能成功一次，后面nacos上的配置确实改了，但是访问url不能得到最新的配置，不知道为啥。。。

再次设置config

1. 启动类更改

   ```java
   @SpringBootApplication
   public class SpringCloudNacosApplication {
   
       public static void main(String[] args) {
           /**
            * 手动在nacos 上添加配置
            * user.name=nacos-config-properties
            * user.age=90
            */
           ConfigurableApplicationContext applicationContext = SpringApplication.run(SpringCloudNacosApplication.class, args);
           String  userName = applicationContext.getEnvironment().getProperty("user.name");
           String  userAge = applicationContext.getEnvironment().getProperty("user.age");
           System.out.println("user name: " + userName + ", user age: " + userAge);
           //SpringApplication.run(SpringCloudNacosApplication.class, args);
       }
   }
   ```

2. `properties` 配置文件

   ```bash
   server.port=8222
   spring.application.name=icarus
   spring.cloud.nacos.config.server-addr=127.0.0.1:8848
   ```

3. 手动增加配置

   ```bash
   Data ID: icarus.properties
    
   Group: DEFAULT_GROUP
   配置格式: Properties
   配置内容：
   publicKey=666
   user.name=icaru
   suser.age=18
   ```

4. 启动项目，输出配置内容

5. 更改nacos配置内容格式为 `yaml` 。在 `nacos` 上手动配置的时候进行选择， `properties` 配置文件增加配置

   ```bash
   # nacos 上的配置格式为yaml, 默认为properties
   spring.cloud.nacos.config.file-extension=yaml
   ```

6. 手动增加配置

   ```bash
   Data ID: icarus.properties
    
   Group: DEFAULT_GROUP
   配置格式: YAML
   配置内容：
   publicKey: 666
   user:
     age: 18
     name: icarus
   ```

7. 启动项目，输出配置内容
