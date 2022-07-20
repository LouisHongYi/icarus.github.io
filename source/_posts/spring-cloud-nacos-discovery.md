---
title: spring cloud nacos discovery
date: 2022-07-19 18:38:49
tags: spring cloud nacos
---

版本对应关系参考：[版本说明 Wiki](https://github.com/spring-cloud-incubator/spring-cloud-alibaba/wiki/版本说明)

demo使用版本（后面学习spring-cloud 也是用该版本）。

- spring-boot `2.4.2`
- spring-cloud `2020.0.1`
- spring-cloud-alibaba `2021.1`

[nacos-discovery](https://github.com/alibaba/spring-cloud-alibaba/blob/2.2.x/spring-cloud-alibaba-docs/src/main/asciidoc-zh/nacos-discovery.adoc)

[官方文档](https://nacos.io/zh-cn/docs/quick-start-spring-cloud.html)

[参考项目](https://github.com/oneredka/learn_springcloud/tree/master/spring-cloud-nacos-discovery)

## 手动模拟

### 新建Const

```java
/**
 * @className: Consts
 * @description: TODO description
 * @tag
 * @author: louis
 * @date: 2022/7/19
 **/
public class Consts {
    public static final String NACOS_SERVER_ADDR = "127.0.0.1:8848";
    public static final String NAMESOACE = "icarus-namespace";
    public static final String SERVICE_NAME = "icarus";

    public static final String IP_1 = "11.11.11.11";
    public static final String IP_2 = "2.2.2.2";

    public static final int PORT_1 = 9001;
    public static final int PORT_2 = 9002;

    public static final String CUSTER_1 = "jack";
    public static final String CUSTER_2 = "rose";

}
```

### 模拟服务注册

```java
package com.example.serviceprovider.self;

import com.alibaba.nacos.api.exception.NacosException;
import com.alibaba.nacos.api.naming.NamingFactory;
import com.alibaba.nacos.api.naming.NamingService;
import com.alibaba.nacos.api.naming.pojo.Instance;

import java.util.List;
import java.util.Properties;

/**
 * @className: ServiceProvider
 * @description: 模拟注册两个实例到注册中心
 * @tag
 * @author: louis
 * @date: 2022/7/19
 **/
public class ServiceProvider {

    public static void main(String[] args) throws NacosException {
        Properties properties = new Properties();
        properties.setProperty("serverAddr", Consts.NACOS_SERVER_ADDR);
        properties.setProperty("namespace", Consts.NAMESOACE);

        NamingService namingService = NamingFactory.createNamingService(properties);
        namingService.registerInstance(Consts.SERVICE_NAME, Consts.IP_1, Consts.PORT_1, Consts.CUSTER_1);
        namingService.registerInstance(Consts.SERVICE_NAME, Consts.IP_2, Consts.PORT_2, Consts.CUSTER_2);
        List<Instance> instances = namingService.getAllInstances(Consts.SERVICE_NAME);
        System.out.println("getAllInstances after registered\\ninstance size="
                + instances.size() + "\\ninstance list =" + instances);
        try {
            int n = System.in.read();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

输出结果

```java
15:55:23.559 [nacos.publisher-com.alibaba.nacos.client.naming.event.InstancesChangeEvent] DEBUG com.alibaba.nacos.common.notify.NotifyCenter - [NotifyCenter] the com.alibaba.nacos.client.naming.event.InstancesChangeEvent@5bc8cec9 will received by com.alibaba.nacos.client.naming.event.InstancesChangeNotifier@4f32a3ad
getAllInstances after registered
instance size=2
instance list =[Instance{instanceId='2.2.2.2#9002#rose#DEFAULT_GROUP@@icarus', ip='2.2.2.2', port=9002, weight=1.0, healthy=true, enabled=true, ephemeral=true, clusterName='rose', serviceName='DEFAULT_GROUP@@icarus', metadata={}}, Instance{instanceId='11.11.11.11#9001#jack#DEFAULT_GROUP@@icarus', ip='11.11.11.11', port=9001, weight=1.0, healthy=true, enabled=true, ephemeral=true, clusterName='jack', serviceName='DEFAULT_GROUP@@icarus', metadata={}}]
```

### 模拟服务发现

```java
package com.example.serviceprovider.self;

import com.alibaba.nacos.api.exception.NacosException;
import com.alibaba.nacos.api.naming.NamingFactory;
import com.alibaba.nacos.api.naming.NamingService;
import com.alibaba.nacos.api.naming.listener.Event;
import com.alibaba.nacos.api.naming.listener.EventListener;
import com.alibaba.nacos.api.naming.listener.NamingEvent;
import com.alibaba.nacos.api.naming.pojo.Instance;

import java.util.List;
import java.util.Properties;

/**
 * @className: ServiceCustomer
 * @description: 创建一个消费服务，向注册中心订阅一个服务，拿到服务列表，模拟五次服务请求
 * @tag
 * @author: louis
 * @date: 2022/7/19
 **/
public class ServiceCustomer {
    public static void main(String[] args) throws NacosException {
        Properties properties = new Properties();
        properties.setProperty("serverAddr", Consts.NACOS_SERVER_ADDR);
        properties.setProperty("namespace", Consts.NAMESOACE);

        NamingService namingService = NamingFactory.createNamingService(properties);
        namingService.subscribe(Consts.SERVICE_NAME, new EventListener() {
            @Override
            public void onEvent(Event event) {
                NamingEvent namingEvent = (NamingEvent) event;
                printInstance(namingEvent);
                mockConsume(namingService, Consts.SERVICE_NAME);
            }
        });

        try {
            int n = System.in.read();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void mockConsume(NamingService namingService, String serviceName) {
        int i = 0, loop = 5;
        try {
            while (i++ < loop) {
                Instance instance = namingService.selectOneHealthyInstance(serviceName);
                if (instance != null) {
                    System.out.println("get one healthy instance of service:" + serviceName
                            + "\\nip=" + instance.getIp()
                            + ",\\nport=" + instance.getPort()
                            + ",\\ncuster=" + instance.getClusterName());
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void printInstance(NamingEvent namingEvent) {
        List<Instance> instances = namingEvent.getInstances();
        System.out.println("all instance of service:\\n");
        for (Instance instance : instances) {
            System.out.println("ip=" + instance.getIp()
                    + ",port=" + instance.getPort()
                    + ",custer=" + instance.getClusterName());
        }
        System.out.println("=====");
    }
}
```

我们可以理解为，注册中心持有一个容器，这个容器可以是 `map` ,也可以是 `mysql` 数据库，现在   `userService` 向注册中心进行注册，该服务有两个提供者，对应着两个实例. `ServiceHolder` 为 `userService`  提供了空间，并将其两个实例挂在该空间下。服务注册成功后，提供者将与注册中心维持心跳，保证能及时剔除失效实例。 消费者向注册中心订阅某个服务，并提交一个监听器，当注册中心发生变更时，监听器收到通知，并更新本地维护的实例列表.

注册与发现的 `demo` 没什么好讲的，最好按照提供的 `demo`源码上来。主要是版本的一些坑。

## 使用注解

### 生产者注册服务

配置文件内容

```bash
server.port=8001
spring.application.name=service-provider
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

`@EnableDiscoveryClient` 开启服务发现

```bash
package com.example.serviceprovider;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@SpringBootApplication
@EnableDiscoveryClient
public class ServiceProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServiceProviderApplication.class, args);
    }

    @RestController
    class EchoController {
        @RequestMapping(value = "/echo/{string}", method = RequestMethod.GET)
        public String echo(@PathVariable String string) {
            return "Hello Nacos Discovery, this is provider-> " + string;
        }
    }
}
```

### 消费者发现服务

配置文件内容

```bash
server.port=8002
spring.application.name=service-customer
spring.cloud.nacos.discovery.server-addr=127.0.0.1:8848
```

`@EnableDiscoveryClient` 开启服务发现

```java
packagecom.example.servicecustomer;

importorg.springframework.beans.factory.annotation.Autowired;
importorg.springframework.boot.SpringApplication;
importorg.springframework.boot.autoconfigure.SpringBootApplication;
importorg.springframework.cloud.client.discovery.EnableDiscoveryClient;
importorg.springframework.cloud.client.loadbalancer.LoadBalanced;
importorg.springframework.context.annotation.Bean;
importorg.springframework.web.bind.annotation.GetMapping;
importorg.springframework.web.bind.annotation.PathVariable;
importorg.springframework.web.bind.annotation.RestController;
importorg.springframework.web.client.RestTemplate;

@SpringBootApplication
@EnableDiscoveryClient
public classServiceCustomerApplication {

public static voidmain(String[] args) {
        SpringApplication.run(ServiceCustomerApplication.class, args);
    }

@LoadBalanced
    @Bean
publicRestTemplate restTemplate() {
return newRestTemplate();
    }

@RestController
public classTestController {

private finalRestTemplate restTemplate;

@Autowired
publicTestController(RestTemplate restTemplate) {this.restTemplate = restTemplate;}

@GetMapping(value = "/echo/{str}")
publicString echo(@PathVariableString str) {
returnrestTemplate.getForObject("<http://service-provider/echo/>" + str, String.class);
        }
    }
}
```
