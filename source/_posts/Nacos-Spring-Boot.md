---
title: Nacos spring boot
date: 2022-07-19 18:35:00
tags: Nacos springboot

---



## 1. 预备环境

- jdk1.8 +
- Maven3.2.x +

[下载最新的Nacos](https://github.com/alibaba/nacos/releases)

![Untitled](nacos-folder.png)

## 2. **文件夹解释**

```java
bin里面是启动和关闭nacos命令文件；
conf存储的nacos相关的配置文件；
logs日志信息
target里有一个springboot的jar包
```

## 3. 启动Nacos

1. **在数据库中创建nacos的配置表**

在本地mysql中创建数据库 `nacos` ,  在该 `schema` 下 执行 `conf` 目录下的 `nacos-mysql.sql` 文件。

1. 更改配置文件中数据库信息

在 `conf` 目录下的 `application.properties` 文件中更改。

```bash
#*************** Config Module Related Configurations ***************#
### If use MySQL as datasource:
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=root

### Connection pool configuration: hikariCP
db.pool.config.connectionTimeout=30000
db.pool.config.validationTimeout=10000
db.pool.config.maximumPoolSize=20
db.pool.config.minimumIdle=2
```

1. 启动

进入bin 目录

```bash
startup.cmd -m standalone
其中-m standalone指定为单机模式，否则以cluster集群模式启动
```

控制集群还是单机

```bash
bin文件夹下找到startup.cmd文件 ，
set MODE=“cluster” (集群模式)，
set MODE=“standalone”  （单机模式）
```

1. **进入nacos管理控制台**

启动nacos服务后，接下来登录管理控制台看看吧

地址: http://localhost:8848/nacos

账号/密码: nacos/nacos

搭建一个 spring-boot-nacos 项目。 https://github.com/oneredka/learn_springcloud.gitzhu

**注意**版本 [0.2.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.boot/nacos-config-spring-boot-starter) 对应的是 Spring Boot 2.x 版本，版本 [0.1.x.RELEASE](https://mvnrepository.com/artifact/com.alibaba.boot/nacos-config-spring-boot-starter)对应的是 Spring Boot 1.x 版本。，`nacos-config-spring-boot-starter:0.2.7`依赖最高支持 `spring boot 2.3.x` 。

## 4. 配置管理

1. 在 `application.properties`中配置 `Nacos server` 的地址

   ```yaml
   server.port=8221
   nacos.config.server-addr=127.0.0.1:8848
   ```

2. 向 `nacos` 注入属性，并设置为动态更新

   ```java
   /**
    *@className:ConfigController
    *@description:配置管理demo
    *@tag
   *@author:louis
    *@date:2022/7/13
    **/
   @Controller
   @RequestMapping("config")
   public classConfigController {
   //通过 Nacos的 @NacosValue注解设置属性值。
   @NacosValue(value = "${useLocalCache:false}", autoRefreshed =true)
   private booleanuseLocalCache;
   
   @GetMapping(value = "/get")
   @ResponseBody
   public booleanget() {
   returnuseLocalCache;
       }
   }
   ```

3. 启动类增加注解 `@NacosPropertySource*(dataId = "example", autoRefreshed = *true*)`

4. 项目启动后，通过cmd调用 [`Nacos Open API`](https://nacos.io/zh-cn/docs/open-api.html) 向 `Nacos server` 发布配置：dataId 为`example`，内容为`useLocalCache=true` 。

```bash
curl -X POST "<http://127.0.0.1:8848/nacos/v1/cs/configs?dataId=example&group=DEFAULT_GROUP&content=useLocalCache=true>"
```

1. 访问 `http://localhost:8221/config/get`，此时返回内容为`true`，说明程序中的`useLocalCache`值已经被动态更新了。

## 服务的注册与发现

1. 在 `application.properties`中配置 `Nacos server` 的地址

   ```yaml
   server.port=8221
   nacos.config.server-addr=127.0.0.1:8848
   ```

2. 增加服务发现的依赖

   ```xml
   <dependency>
       <groupId>com.alibaba.boot</groupId>
       <artifactId>nacos-discovery-spring-boot-starter</artifactId>
       <version>0.2.7</version>
   </dependency>
   ```

3. 调用 `<http://localhost:8080/discovery/get?serviceName=example-service>](<http://localhost:8221/discovery/get?serviceName=example>)`，此时返回为空 JSON 数组`[]`

4. 编写服务发现类

   ```java
   /**
    * @className: DiscoverController
    * @description: 服务发现
    * @tag
    * @author: louis
    * @date: 2022/7/14
    **/
   @Controller
   @RequestMapping("discovery")
   public class DiscoveryController {
   
       @NacosInjected
       private NamingService namingService;
   
       @GetMapping(value = "/get")
       @ResponseBody
       public List<Instance> get(@RequestParam String serviceName) throws NacosException {
           return namingService.getAllInstances(serviceName);
       }
   }
   ```

5. 项目启动后，,通过调用 [`Nacos Open API`](https://nacos.io/zh-cn/docs/open-api.html)  向 `Nacos server` 注册一个名称为 `example-service` 服务

   ```bash
   curl -X POST "<http://127.0.0.1:8848/nacos/v1/ns/instance?serviceName=example-service&ip=127.0.0.1&port=8221>"
   ```

6. 调用  [`http://localhost:8221/discovery/get?serviceName=example-service`](http://localhost:8221/discovery/get?serviceName=example-service) 结果

   ```json
   [
   	{
   		"instanceId":"127.0.0.1#8221#DEFAULT#DEFAULT_GROUP@@example-service",
   		"ip":"127.0.0.1",
   		"port":8221,
   		"weight":1,
   		"healthy":true,
   		"enabled":true,
   		"ephemeral":true,
   		"clusterName":"DEFAULT",
   		"serviceName":"DEFAULT_GROUP@@example-service",
   		"metadata":{
   		},
   		"instanceHeartBeatInterval":5000,
   		"instanceIdGenerator":"simple",
   		"instanceHeartBeatTimeOut":15000,
   		"ipDeleteTimeout":30000
   	}
   ]
   ```

   
