---
title: '@CreateDate 与 @CreationTimeStamp的区别'
date: '2019-4-18 23:38:00'
---

# @CreateDate 与 @CreationTimeStamp

## 比较

### 共同点

- 都能够标记在日期属性上，实现实体创建时自动补充日期属性
- 都支持多种时间类型
- 能够支援不同的数据库

### 不同点

- @CreateDate 来自 SpringJpaAuditing 框架，使用不会导致与库、框架耦合，并且能够被 SpringBoot 自动配置，仅需要引入依赖和在 SpringApplication 上加入 @EnableJpaAuting 的注解，解耦充分

- @CreationTimeStamp 来自 hibernate 框架，目前常用于 Jpa 接口的实现，因此如果项目使用 Jpa 的话就不适合使用 hibernate 框架的注解，会导致与 hibernate 耦合，使得 JPA 的灵活性下降

