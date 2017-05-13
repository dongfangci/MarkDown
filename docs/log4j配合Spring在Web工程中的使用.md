##log4j配合Spring使用##
###各工具版本:###
JDK 1.8  
Maven 3.5  
Tomcat 7  
Eclipse Mars Release (4.5.0)  
Spring 4.0.2 RELEASE 
log4j 1.2.17c  
###1：Spring（Spring-core）默认使用JCL（commons-logging）作为log文件的输出框架,但是JCL可以在运行时发现Project classpath下的其他log框架。因此可以在project classpath下创建log4j的配置文件log4j.properties或 log4j.xml，Maven结构中即为src/main/resources文件夹，如下图所示：###
![](http://i.imgur.com/TRzqGku.png)   
###2：依赖管理 pom.xml文件###
```
<properties>  
	<spring.version>4.1.6.RELEASE</spring.version>  
	<log4j.version>1.2.17</log4j.version>  
</properties>
```
```
<dependencies>

	<!-- Spring -->
	<dependency>
		<groupId>org.springframework</groupId>
		<artifactId>spring-webmvc</artifactId>
		<version>${spring.version}</version>
	</dependency>

	<!-- Log4j -->
	<dependency>
		<groupId>log4j</groupId>
		<artifactId>log4j</artifactId>
		<version>${log4j.version}</version>
	</dependency>

</dependencies>
```  
###3:log4j.properties文件内容###
```
log4j.rootLogger=DEBUG, stdout, file

log4j.appender.stdout=org.apache.log4j.ConsoleAppender   
log4j.appender.stdout.Target=System.out  
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout  
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n


log4j.appender.file=org.apache.log4j.RollingFileAppender  
log4j.appender.file.File=${catalina.home}/logs/myapp.log  
log4j.appender.file.MaxFileSize=5MB  
log4j.appender.file.MaxBackupIndex=10  
log4j.appender.file.layout=org.apache.log4j.PatternLayout  
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
```  
**注意： **   
1.上面File类型的log存放的地址定义为catalina.home/logs,此地址为Tomcat安装目录下的logs文件夹  
2.若改为catalina_home/logs(catalina_home已经在环境变量中配置为Tomcat安装目录)，则log存储的路径为Tomcat安装的目录所在的根目录下的logs文件夹。  
3.若改为/logs,则log存储的路径为Tomcat安装的目录的bin文件夹（bin文件夹下新建logs目录）。  
4.通过在web.xml中重新配置log4j可以实现将log文件存在其他位置   
```   
< context-param>   
     <param-name>webAppRootKey</param-name>   
     <param-value>webApp.root</param-value>   
   </context-param>   
  <context-param>   
   <param-name>log4jConfigLocation</param-name>   
     <param-value>classpath:log4j.properties</param-value>  
  </context-param>  
< listener>      
      <listener-class>org.springframework.web.util.Log4jConfigListener</listener-class>    
  </listener>  
```  

log4j.appender.logfile.File=${webApp.root}/WEB-INF/logs/app.log