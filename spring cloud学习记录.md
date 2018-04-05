# spring cloud学习记录

## 简介部分

### 单体架构

- 简介:单体架构对业务场景没有划分,将所有业务场景的表示层,业务逻辑层和数据访问层都放在一个工程里面,最终一起部署
- 不足:
  - 随着业务发展,代码量越来越多,代码的可读性/可维护性/可扩展性下降
  - 并发能力有限
  - 难以测试


### 微服务

#### 特点

- 微服务单元按业务来划分

  开发模式从传统的职能划分变为按业务划分,也就是一个业务一个团队

- 通过HTTP通信
  - 使用RESTful API方式通信,或者通过RabbitMQ/Kafaka等轻量级消息总线通信
  - 数据格式通常为json/XML或者通过Protobuf序列化
  - 当然这种通讯方式不依赖于编程语言,即Java写的服务可以消费Go写的服务
  - 弊端就是其通信机制不可靠,虽然成功率高但是还是会有失败的时候

> 使用Protobuf进行数据序列化时,产生的数据为二进制,需要反序列化才能读懂,但是由于更为轻量,所以也十分受欢迎

- 数据库独立

  服务于服务之间通过HTTP通信的话,每个服务都可以有自己的数据库. 而且可以使用不同的数据库,比如A服务使用MySQL,B服务使用MongoDB等

- 服务集中化管理

  由于按照业务划分服务,服务的数量很多,所以必须集中管理服务. 像Eureka/Zookeeper等都是非常优秀的服务集中化管理框架

- 分布式架构
  - 微服务架构是分布式架构
  - 分布式系统是集群部署的
  - 分布式系统通过网络协议来通信
  - 熔断机制是为了防止"雪崩效应"

> 雪崩效应: 分布式系统中的服务通信依赖于网络,网络不好,必然会对分布式系统带来很大的影响.在分布式系统中,服务之间相互依赖,如果一个服务出现了故障或者是网络延迟,在高并发的情况下,会导致线程阻塞,在很短的时间内该服务的线程资源会被消耗殆尽,最终使得该服务不可用.由于服务的相互依赖,可能会导致整个系统的不可用,这就是"雪崩效应"

- 熔断机制

  为了防止上述的雪崩效应,分布式系统采用了熔断机制. 当服务A出现故障时,请求失败次数超过设定的阈值之后,服务A就会开启熔断器(spring cloud的组件),之后服务A不进行任何的业务逻辑操作,执行快速失败,直接返回请求失败的信息.其他依赖于服务A的服务就不会因为得不到响应而线程阻塞,这时除了服务A和依赖于服务A的部分功能不可用之外,其他功能正常.

  这个spring cloud的熔断组件熔断器还有一个机制,就是自我修复机制. 当服务A熔断后,经过一段时间半打开熔断器,半打开的熔断器会检查一部分请求是否正常,其他请求执行快速失败,检查的请求如果响应成功,则可以判定服务A正常了,就会关闭服务A的熔断器.反之则继续开启熔断器.

#### 优势

- 代码被拆分,服务之间边界明确,可读性和可扩展性增加,新人加入团队只需要了解他接管的服务的代码即可
- 边界明确,服务之间没有任何耦合,易横向扩展
- 服务之间通过HTTP通信,可以使用多种开发语言和技术实现
- 重写某服务的代码容易
- 测试和部署简单

#### 不足

- 复杂

  微服务系统是分布式系统,构建的复杂度远远超过单体系统

- 分布式事物

  分布式事物常常分为两个阶段提交,第一个阶段发起分布式事物,交给事务管理器,事务管理器通知所有节点做准备,所有节点收到指令后做相应准备并且记录日志然后返回准备结果. 第二阶段就是收集到了所有节点返回的数据之后,如果全都成功就执行提交,只要有一个失败就进行回滚

  分布式事物大大降低数据库的性能,而且不可靠,所以一般情况下,尽量少用.

- 服务的划分

  服务划分的粒度难控制


#### 设计原则

技术应该是随着业务的发展而发展的,不能盲目的使用微服务架构.



## spring cloud简介

首先我们再来讲讲微服务应该具备的功能

### 微服务应该具备的功能

微服务具有以下特点:

- 按照业务划分服务
- 微服务之间相互独立
- 通过HTTP或者消息组件通信,具有容错能力
- 相互之间不耦合,可随时增加和剔除
- 单个服务能够集群化部署,并且具有负载均衡的能力
- 整个微服务系统应该有一个完整的安全机制
- 整个微服务系统有链路追踪的能力
- 有一套完整的实时日志系统

微服务的功能重要体现在:

- 服务的注册和发现
- 服务的负载均衡
- 服务的容错
- 服务网关
- 服务配置的统一管理
- 链路追踪
- 实时日志

### 服务的注册和发现

- 服务的注册

  服务的注册是指,向服务注册中心注册一个服务实例,服务提供者将自己的服务信息(如服务名,IP地址等等)告知服务注册中心

- 服务的发现

  服务的发现是指,当服务消费者需要消费另外一个服务时,服务注册中心能够告知服务消费者它所需要消费服务的实例信息

通常情况下,一个服务既是服务消费者,也是服务提供者. 服务消费一般是通过HTTP或者消息组件.

一个服务实例注册后,需要定时的向服务注册中心发送"心跳",证明自己还处于可用状态,如果一段时间没有发送心跳,服务注册中心则会认为这个服务不可用,从服务注册列表中将其剔除,过一段时间如果心跳恢复,则又会将其加入到服务注册列表之中

### 服务的负载均衡

- 什么是负载均衡

  在微服务架构中,服务之间的相互调用一般是通过HTTP通信协议来实现的.由于网络具有不可靠性,所以为了保证服务的高可用,服务单元往往是集群化部署. 那么每个服务就可能存在多个实例,在服务消费的时候就需要做负载均衡

- 谁来做负载均衡

  服务的负载均衡最流行的一种做法是,由服务消费者来做负载均衡. 所有的服务都向服务注册中心注册,然后定时的发送心跳.同时每个服务也会定时的获取所有服务注册列表信息. 在进行服务消费的时候,消费者服务就根据自己已知的服务注册列表,通过一定的负载均衡策略(轮询,哈希等等,可由开发者配置)进行负载均衡.

服务注册中心不但需要定时接收每个服务的心跳,而且每个服务还会定期的获取服务注册列表的信息,当服务实例数量非常多的时候服务注册中心的负载时非常大的,所以,服务注册中心也需要进行集群部署.把服务注册中心集群化之后,服务注册中心之间实时同步信息,实现了高可用.

### 服务的容错

服务的容错主要是运用熔断机制,这种机制的原理在上面已经说过了,在这就不在赘述了.在这讲一下这种熔断机制除了防止系统的"雪崩",还具有服务降级的能力,如果请求的数量超过了服务的处理能力时,会将其降级,避免服务器负载过高.

### 服务网关

- 服务网关是干什么的

  在微服务系统中,API接口资源通常是由服务网关统一暴露,并且服务网关通常还有请求转发/安全验证等作用.

一般来说,在服务网关层之前需要加上负载均衡层.用户请求通过负载均衡转发到网关层,然后进行一些安全验证转发到具体的服务.

网关层具有以下重要意义:

- 将所有服务的API接口资源聚合,对外统一暴露
- 安全验证
- 监控,实时日志输出,请求记录等
- 流量监控,在流量过高情况下,对服务进行降级
- API接口从内部服务分离出来方便测试



### 服务配置的统一管理

在实际开发中,每个服务都需要大量的配置文件,因此就需要有一个统一管理配置文件的组件,spring cloud 的Config组件/阿里的Diamond组件/携程的Apollo组件等等. 这些组件实现的功能大体相同. 以spring cloud Config来阐述一下服务配置统一管理的工作流程:

- config server(配置服务)读取配置文件仓库的信息,仓库可以是本地也可以是远程
- 配置服务启动后,读取的配置文件信息存于内存之中.
- 当其他服务启动需要读取配置信息的时候,向配置服务读取.
- 服务的配置信息修改且修改完成后,发送post请求给配置服务进行刷新,同时其他服务向配置服务重新读取配置


### 服务的链路追踪

服务的链路追踪其实就是记录一个请求从头到尾经过了多少个服务,并且记录先后顺序. 

目前常见的链路追踪组件有Google的Dapper,Twitter的Zipkin以及阿里的Eagleeye(鹰眼)等.

### spring cloud 相关

#### 常用组件

- Spring Cloud Netflix
  - 服务注册和发现组件Eureka

  - 熔断组件Hystrix

    - Hystrix Dashboard组件提供了单个服务熔断器的健康状态数据的界面展示
    - Hystrix Turbine组件提供了多个服务熔断器的健康状态数据的界面展示

  - 负载均衡组件Ribbon

    Ribbon是一个负载均衡组件,通常和Eureka/Zuul/RestTemplate/Feign配合使用

  - 路由网关组件Zuul

- spring cloud config

  配置文件统一管理,通常和spring cloud bus相互配合,刷新指定client或者全部client的配置文件

- spring cloud security

  用来做用户验证和权限认证,通常和spring security oauth2组件搭配使用

- spring cloud sleuth

  链路追踪使用

- spring cloud stream

  数据流操作包,可以封装RabbitMq/ActiveMq/Kafka/Redis等消息组件

### Dubbo相关

Dubbo是阿里巴巴开源的一个分布式服务框架

- 核心内容
  - RPC远程调用,封装了长连接框架,如Netty/Mina等等,采用多线程模式
  - 集群容错
  - 服务发现
- 架构流程
  - 服务提供者向服务中心注册服务
  - 服务消费者订阅服务
  - 服务消费者发现服务
  - 服务消费者远程调度服务提供者进行服务消费,在调度过程中,使用了负载均衡策略,失败容错等功能
  - 服务消费者和提供者,在内存中记录服务的调用次数和调用时间,并定时每分钟发送一次统计数据到监控中心

#### spring cloud和Dubbo比较

| 微服务关注点 | spring cloud            | Dubbo     |
| ------------ | ----------------------- | --------- |
| 配置管理     | config                  | -         |
| 服务发现     | eureka/consul/zookeeper | zookeeper |
| 负载均衡     | ribbon                  | 自带      |
| 网关         | zuul                    | -         |
| 分布式追踪   | spring cloud sleuth     | -         |
| 容错         | Hystrix                 | 不完善    |
| 通信方式     | HTTP,message            | RPC       |
| 安全模块     | spring cloud security   | -         |

- spring cloud 项目模块多
- spring cloud 更新速度快
- spring cloud 学习成本高
- Dubbo 倾向于spring xml的配置方式, spring cloud采用基于注解和Javabean配置方式的敏捷开发
- spring cloud使用HTTP Restful风格通信方式,不依赖平台和语言, Dubbo使用远程调用,对语言和平台强依赖

### kubernetes

kubernetes是一个容器集群管理系统,为容器化的应用程序提供部署运行/维护/扩展/资源调度/服务发现等功能.

kubernetes是Google运行Borg大规模系统长时间的一个经验总结.具有以下特点:

- 大容量
- 永不过时
- 随时随地运行

kubernetes是开源免费的,它提供以下功能:

- 自动包装
  自动配置容器
- 自我修复
- 横向扩展
  可以根据机器CPU的使用率来调整容器的数量,只需开发人员在管理界面输入几个命令即可
- 服务发现和负载均衡
- 自动部署或回滚
- 配置管理
  部署和更新应用程序的配置,不需要重新打镜像
- 存储编排
- 批量处理

| 微服务关注点 | spring cloud            | kubernetes              |
| ------------ | ----------------------- | ----------------------- |
| 配置管理     | config                  | kubernetes configmap    |
| 服务发现     | eureka/consul/zookeeper | kubernetes services     |
| 负载均衡     | ribbon                  | kubernetes services     |
| 网关         | zuul                    | kubernetes services     |
| 分布式追踪   | spring cloud sleuth     | open tracing            |
| 容错         | Hystrix                 | kubernetes health check |
| 安全模块     | spring cloud security   | -                       |
| 分布式日志   | ELK                     | EFK                     |
| 任务管理     | spring batch            | kubernetes jobs         |

spring cloud的特点:

- 优点
  - 采用Java,Java程序员易上手
  - 有大量的类库和资源
- 缺点
  - 不支持跨语言
  - 关注微服务的功能点

kubernetes的特点:

- 优点
  - 支持多种语言
  - 应用于多种场合,开发,测试,等等
  - 除了提供基本的构建微服务功能之外,还提供了环境,资源限制,管理应用程序的生命周期等等,更像是一个平台
- 缺点
  - 学习成本非常高
  - 不断学习




## 实战部分

- 在变量上加入`@Value("${属性名}")`注解就可以取到一个配置值

- 在配置文件中可用`${random}`生成随机数, 例如`${random.int}`
  `${random.int(10)}`是生成10以内的随机数

- 配置文件相关

  - 自定义属性

    配置文件:

    ```yaml
    my:
      name: sunhao
      age: 21
    ```


~~~java
使用配置:

```java
@Value("${my.name}")
private String name;

@GetMapping("/getInfo")
public String info() {
    return name;
}
```
~~~

  - 将属性提取成实体类

    配置文件:

    ```yaml
    my:
      name: sunhao
      age: 21
      number: ${random.int}
    ```

    提取实体类:

    ```java
    @ConfigurationProperties(prefix = "my")
    @Component
    public class Sunhao {
        private String name;
        private int age;
        private int number;
        //省略了getter setter
    }
    ```

    使用实体类:

    ```java
    @RestController
    @EnableConfigurationProperties({Sunhao.class})
    public class ConfigController {
        @Autowired
        private Sunhao sunhao;

        @GetMapping("/getInfo")
        public String info() {
            return sunhao.getName();
        }
    }
    ```

  - 将配置文件提取到实体类

    配置文件(独立的文件):

    ```yaml
    my:
      name: sunhao
      age: 21
      number: ${random.int}
    ```

    提取实体类:

    ```java
    @Configuration
    @PropertySource(value = "classpath:sunhao.yml")
    @ConfigurationProperties(prefix = "my")
    public class Sunhao {
        private String name;
        private int age;
        private int number;
        //省略了getter setter
    }
    ```

    使用实体类:

    ```java
    @RestController
    @EnableConfigurationProperties({Sunhao.class})
    public class ConfigController {
        @Autowired
        private Sunhao sunhao;

        @GetMapping("/getInfo")
        public String info() {
            return sunhao.getName();
        }
    }

    ```

  - 指定使用哪一个文件使用

    ```yaml
    spring:
      profiles:
        active: dev
    ```

- Actuator相关

  Spring Boot的Actuator提供了运行状态监控的功能，书上讲的不详细，而且有些地方配置是有问题的。留待以后在网上学习之后补充

- jpa、Redis、swagger2相关

  后两个在书上讲的不太详细，也留待以后在网上查询学习之后补充

- spring cloud常用组件实战

  在这个部分只记录关键配置或者关键的原理

  - eureka组件

    - 为了防止eureka自己向自己注册，需要编写如下两条配置关闭：

      ```
      eureka:
        client:
          registerWithEureka: false
          FetchRegistry: false
      ```

    - 服务续约

      Eureka Client 在默认的情况下每隔三十秒向server发一次心跳，证明自己还活着。如果server90秒没有收到心跳则会把该服务移出列表。这个时间间隔是可以更改的，但是官方并不建议这么做。同时隔30秒client就会向server查询服务注册表。

    - 服务下线

      在程序关闭的时候client可以向server发送下线信息，server收到下线信息之后会更新注册表。但是这个下线信息需要在程序关闭的时候调用

      ```java
      DiscoveryManager.getInstance().shutdownComponent()
      ```

    - 6

      ​

  ​