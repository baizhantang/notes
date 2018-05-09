---
title: spring boot企业微信点餐课程学习记录
tags: 知识收集
grammar_cjkRuby: true
---

# 日志框架

## 产生日志

- 传统方法
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class LoggerTest {

    //这个Logger类需要选择org.slf4j包下的
    private final Logger logger = LoggerFactory.getLogger(LoggerTest.class);

    @Test
    public void test1() {
        logger.info("info...");
        logger.debug("debug...");
        logger.error("error...");
    }
}
```

- 简便方法

简便方法需要引入和安装第三方插件`lombok`,这个插件的引入需要两步：
1. 引入maven依赖
```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>
```

2. 安装idea的lombok插件（不安装直接maven打包运行是没问题的，但是直接启动项目会报错）

引入了插件之后，代码变成了这样：
```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Slf4j
public class LoggerTest {

    @Test
    public void test1() {
        log.info("info...");
        log.debug("debug...");
        log.error("error...");
        
        //输出变量
        String name = "sunhao";
        Integer age = 18;
        log.info("name={}, age={}", name, age);
    }
}
```

## 输出日志

1. 配置

- application.yml

这种配置方式比较简单，可配置的项目也比较简单。
```yaml
logging:
  pattern:
    console: "%d - %msg%n" #这个配置是控制日志输出的格式的，"日期 - 信息 换行"
  path: /Users/sunhao/Documents/测试日志/  #配置日志输出路径
  file: /Users/sunhao/Documents/测试日志/test/  #配置日志输出路径并且指定文件名
  level: debug #修改默认的日志级别，默认的是info
```



- logback-spring.xml

这种配置方式比较复杂，可配置的项目也比较丰富，一般来说使用这个。下面这个配置实现了每天输出一份日志文件，error日志和其他日志分开。
```xml
<?xml version="1.0" encoding="utf-8" ?>

<configuration>

    <!--配置输出的格式-->
    <appender name="consoleLog" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>
                %d - %msg%n
            </pattern>
        </layout>
    </appender>

    <!--配置一天输出一个日志文件-->
    <appender name="fileInfoLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--过滤error日志-->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>DENY</onMatch>
            <onMismatch>ACCEPT</onMismatch>
        </filter>

        <encoder>
            <pattern>
                %msg%n
            </pattern>
        </encoder>

        <!--配置时间滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/Users/sunhao/Documents/测试日志/test %d.log</fileNamePattern>
        </rollingPolicy>
    </appender>

    <!--配置一个error日志-->
    <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--只接受error日志-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>

        <encoder>
            <pattern>
                %msg%n
            </pattern>
        </encoder>

        <!--配置时间滚动策略-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/Users/sunhao/Documents/测试日志/error %d.log</fileNamePattern>
        </rollingPolicy>
    </appender>

    <!--应用配置-->
    <root level="info">
        <appender-ref ref="consoleLog" />
        <appender-ref ref="fileInfoLog" />
        <appender-ref ref="fileErrorLog" />
    </root>

</configuration>
```