#**Spring MVC总结**#
  
##web.xml 配置文件结构以及功能##
###1.xml头文件###
```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	version="2.5">
</web-app>
```
###2.其他配置项均写在web-app配置之内###
**	(1)Servlet配置：配置拦截器以及映射拦截地址**
```java
<servlet>
<servlet-name>autox</servlet-name>
<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
<init-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>/WEB-INF/spring-mvc.xml</param-value>
</init-param>
<load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>autox</servlet-name>
	<url-pattern>*.html</url-pattern>
</servlet-mapping>
```
**(2)Filter配置1：配置编码**
```java
	<filter>
		<filter-name>setCharacterEncoding</filter-name>
		<filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>setCharacterEncoding</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
```
**(3)Filter配置2：配置过滤器，此处以添加OpenID过滤器为例**
```java
	<!-- 添加OpenId过滤器 -->
	<filter>  
        <filter-name>DelegatingFilterProxy</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>  
        <init-param>  
            <param-name>targetBeanName</param-name>  
            <param-value>sessionFilter</param-value>  
        </init-param>
        <init-param>  
            <param-name>targetFilterLifecycle</param-name>  
            <param-value>true</param-value>  
        </init-param>  
    </filter>  
	<filter-mapping>  
        <filter-name>DelegatingFilterProxy</filter-name>  
        <url-pattern>/devicemanager/*</url-pattern>  
    </filter-mapping>
    <filter-mapping>  
        <filter-name>DelegatingFilterProxy</filter-name>  
        <url-pattern>/resume/*</url-pattern>  
    </filter-mapping>
```