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



- logback-spring.xml

这种配置方式比较复杂，可配置的项目也比较丰富，一般来说使用这个。