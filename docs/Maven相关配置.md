###jar包的统一版本控制###
**一 发现问题**
```java
在pom.xml中添加依赖时语法如下
<dependency>
 <groupId>org.springframework</groupId>
 <artifactId>spring-core</artifactId>
 <version>1.2.6</version>
</dependency>
<dependency>
 <groupId>org.springframework</groupId>
 <artifactId>spring-aop</artifactId>
 <version>1.2.6</version>
</dependency>
```
以上内容没错，但有这样一个问题，在spring的依赖中，我们需要引用一系列版本的spring，如版本1.2.6。每次都写不利于维护。

**二 解决办法**
在pom.xml定义properties标签
```java
<properties>
 <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
 <spring.version>1.2.6</spring.version>
 <developer.organization><![CDATA[xy公司]]></developer.organization>
</properties>
```
以上内容就改成了
```java
<dependency>
 <groupId>org.springframework</groupId>
 <artifactId>spring-core</artifactId>
 <version>${spring.version}</version>
</dependency>
<dependency>
 <groupId>org.springframework</groupId>
 <artifactId>spring-aop</artifactId>
 <version>${spring.version}</version>
</dependency>
```
###Maven安装中央仓库没有的jar包###
一 概述
    使用疱丁分词器，发现中央仓库中没有paoding-analysis这个jar包，而且如果只是单纯的将从其他处获取的jar包拷贝到本地仓库时不行的，pom文件依然报错，也无法导入到项目的classpath中。需要用maven将jar包导入到本地仓库中才可以，不能通过手动拷贝的方式。以下将介绍如何将中央仓库中没有的jar包导入到本地仓库。

二 环境
windows 7
Apache Maven 3.1.1

三 步骤
1.命令格式如下
```java
mvn install:install-file   
-DgroupId=包名    
-DartifactId=项目名    
-Dversion=版本号    
-Dpackaging=jar    
-Dfile=jar文件所在路径
```

2.执行安装
```java
mvn install:install-file -Dfile=E:\JavaEE\开发库\paoding-analysis.jar -DgroupId=net.paoding -DartifactId=paoding-analysis -Dversion=2.0.4-beta -Dpackaging=jar -DgeneratePom=true -DcreateChecksum=true
```

3.pom依赖设置
```java
<dependency>  
    <groupId>net.paoding</groupId>  
    <artifactId>paoding-analysis</artifactId>  
    <version>2.0.4-beta</version>  
    <scope>runtime</scope>  
</dependency></span>  
```