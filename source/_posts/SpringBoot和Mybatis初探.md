---
title: SpringBoot和Mybatis初探
date: 2018-12-05 14:48:36
excerpt: 整个SpringBoot非常方便的可以实现Restful服务，然而在实际的应用中，实现Restful服务只是表面，如果需要运营起来，需要数据层的支持...
category: Tec
photos: /images/Spring.png
---
## 前言
整个SpringBoot非常方便的可以实现Restful服务，然而在实际的应用中，实现Restful服务只是表面，如果需要运营起来，需要数据层的支持。目前服务端的数据层包括关系型数据库例如MySQL与非关系型数据库例如MangoDB。非关系型数据库可能更加方便实现对象存储，对应用编码更加方便，但是对底层存储的理解不利，所以在刚开始还是推荐使用关系型数据库。
这里我们将使用SpringBoot的Restful项目来结合Mybatis来实现一个有数据存储的Restful服务。
## Mybatis是什么？
[这里是Mybatis的介绍](http://www.mybatis.org/mybatis-3/zh/index.html)
MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。
简言之：MyBatis可以方便的帮助JAVA应用使用关系型数据库。
## 写代码之前
在实际代码之前，我们需要给项目配置Mybatis，这里的项目管理使用的maven管理，有一个pom.xml来管理依赖，所以我们开始吧：
给项目加入MyBatis依赖
```
<!--加入mybatis的支持-->
<dependency>
	<groupId>org.mybatis.spring.boot</groupId>
	<artifactId>mybatis-spring-boot-starter</artifactId>
	<version>1.1.1</version>
</dependency>
```
给项目加入MySQL连接的依赖
```
<!--加入mysql connector-->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
</dependency>
```
在加入依赖之后，需要做的是首先是配置数据库端口和用户
当然，我这里使用的是MySQL来作为数据库。端口是3306，用户是root。

所以开始配置：
[这里是一个参考](https://blog.csdn.net/Winter_chen001/article/details/78622141)
[这里是一个更好的参考](https://www.bysocket.com/?p=1610)


完成数据库相关配置之后，我们需要在Spring项目中完成数据库的连接配置，不然Spring如何知道该连接哪个数据库呢。

- 在application.properties中添加下面的数据（如果没有改文件，请在java平级建立一个resource文件夹下新建一个改文件）
```
server.port=5000
########################################################
###datasource
########################################################
spring.datasource.url = jdbc:mysql://localhost:3306/blog?useUnicode=true&characterEncoding=UTF-8
spring.datasource.username = root
spring.datasource.password = seven107001
spring.datasource.driverClassName = com.mysql.jdbc.Driver
spring.datasource.max-active=20
spring.datasource.max-idle=8
spring.datasource.min-idle=8
spring.datasource.initial-size=10
```
然后SpringBoot就知道用什么（依赖）去连接哪个数据库（配置）了。

## 开始编码吧！
我们在源代码里面加入dao层，并加入一个Mapper。（注意mapper是一个Interface）
![](/images/SpringBoot+Mybatis初探1.png)
在Mapper中的代码如下：
![](/images/SpringBoot+Mybatis初探2.png)
```
package blog.dao;

import blog.domain.BlogItem;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;

import java.util.LinkedList;

@Mapper
public interface BlogMapper {
    @Select("SELECT * FROM blogs")
    LinkedList<BlogItem> getAllBlogItem();

    @Insert("INSERT INTO blogs (title, subTitle, summary, content, dateString) VALUES(#{title}, #{subTitle}, #{summary}, #{content}, #{dateString})")
    int insert(@Param("title") String title,
               @Param("subTitle") String subTitle,
               @Param("summary") String summary,
               @Param("content") String content,
               @Param("dateString") String dateString);
}
```
是不是很简单，我们已经用注释的方式完成了从数据库获取和插入的操作。

## 测试
1. 新建一个Test文件，并在类上加注释：
```
@RunWith(SpringRunner.class)
@SpringBootTest(classes = App.class, webEnvironment = SpringBootTest.WebEnvironment.**RANDOM_PORT**)
```
因为使用SpringBoot来跑，所以这里测试需要有一个Application类传入，随机端口。

2. 然后在类中加入一个自动注入的Mapper：
```
@Autowired
private BlogMapper mapper;
```
这时候mapper我们就不需要自己去初始化了，毕竟自动注入

3. 最后我们写一个测试方法吧：
```
@Test
public void testInsertBlog() {
    mapper.insert("标题", "副标题", "总结", "主要内容", "一小时以前");
}
```
这里我们测试了插入，如果是数据库本身配置没有问题的话，就已经完成了SpringBoot对数据库的操作了。

## 结语
其实Spring+Mybatis真的不难，在做之前感觉挺复杂畏难心理，实际做的时候发现和iOS数据库操作基本一致，基本都是三步：加入依赖，连接，写很容易理解的代码。
