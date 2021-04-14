---
title: JPA
date: 2020-08-12 00:10:30
tags: 
  - JPA
categories: 
  - 框架
description: JPA
comments: true
top_img: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/img6.jpg
cover: https://cdn.jsdelivr.net/gh/liuyeweiQX/picBed/img/img6.jpg
---
### JPA是什么

JPA（Java Persistence API）Java持久化API。是一套sun公司Java官方制定的ORM方案，是规范，是标准，sun公司自己并没有实现。

**关键点：ORM，标准 概念(关键字)**

### ORM是什么

ORM（Object Relational Mapping）对象关系映射。问：ORM有什么用？在操作数据库之前，先把数据表与实体类关联起来。然后通过实体类的对象操作（增删改查）数据库表，这个就是ORM的行为！所以：ORM是一个实现使用对象操作数据库的设计思想！通过这句话，我们知道JPA的作用就是通过对象操作数据库的，不用编写sql语句。

### JPA的实现者

既然我们说JPA是一套标准，意味着，它只是一套实现ORM理论的接口。没有实现的代码。那么我们必须要有具体实现者才可以完成ORM操作功能的实现！ 

市场上主流的JPA框架（实现者）有：Hibernate(JBoss)、EclipseTop(Eclipse社区)、OpenJPA(Apache基金会)其中Hibernate是众多实现者之中，性能最好的。

### JPA的作用是什么

JPA是ORM的一套标准，既然JPA为ORM而生，那么JPA的作用就是实现使用对象操作数据库，不用写SQL

数据库是用sql操作的，那用对象操作，由谁来产生SQL?

答：JPA实现框架

### 核心概念 

- 实体（pojo）表示关系数据库中的一个表

- 每个实体实例对应着该表中的一行

- 类必须用javax.persistence.Entity注解

- 类必须含有一个public或者protected的无参构造函数

- 当实体实例被当做值以分离对象的方式进行传递（例如通过会话bean的远程业务接口）则该类必须实现Serializable（序列化）接口

- 唯一的对象标志符，简单主键（javax.persistence.Id），复合主键（javax.persistence.EmbeddledId和javax.persistence.IdClass）

### 关系 

- 一对一：@OneToOne

- 一对多：@OneToMany

- 多对一：@ManyToOne

- 多对多：@ManyToMany

### EntityManager

- 管理实体的一个类，接口

- 定义与持久性上下文进行交互的方法

- 创建和删除持久实体类，通过实体的主键查找实体

- 允许在实体类上进行查询
                                                                                                                                                          