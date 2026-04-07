# 系统技术选型

## 语言版本

Java 18

## 整体框架

Spring Boot：3.1.x 或 3.2.x

## 存储

MySQL：8.0.x

## 构建

Gradle：7.5+

## 依赖包

0.基础
- spring-boot-starter-web

1.开发效率与代码质量
- lombok
- mapstruct
- spring-boot-starter-validation

2.持久层与数据库
- flyway-core
- spring-boot-starter-data-jpa
- mybatis-plus-boot-starter
- mysql-connector-j

3.监控与运维
- spring-boot-starter-actuator

4.接口文档
- springdoc-openapi-starter-webmvc-ui

5.常用辅助组件
- hutool-all

6.测试
- spring-boot-starter-test
- archunit-junit5
- junit-jupiter
- org.testcontainers: mysql
- rest-assured
- awaitility
