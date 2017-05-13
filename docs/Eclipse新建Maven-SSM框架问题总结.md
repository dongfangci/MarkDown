# 使用Eclipse搭建Web工程遇到的问题 #
##框架使用##
1.Spring 4.0.2  
2.Mybatis 3.2.6  
3.MySQL 5.7.18  
4.Maven 3.5.0  
5.Tomcat 7.0.77
##遇到的问题总结##
####1.Eclipse新建Maven-Web项目,缺少src/main/java等文件夹  
Properties -> Java Build Path -> Libraries -> JRE System Library 选择工作空间默认的JRE即可自动添加src文件夹
之后右键项目 Maven-> Update Project  

####2.建好工程后index.jsp报错####
第一种：直接在pom.xml文件中添加jar包支持：  
```
<!-- 导入java ee jar 包 -->  
        <dependency>  
            <groupId>javax</groupId>  
            <artifactId>javaee-api</artifactId>  
            <version>7.0</version>  
        </dependency> 
```   
添加完成之后ctrl+s保存一下，就会发现项目不报错了！   
第二种：添加tomcat支持  
第一步：选中项目点击右键，选择“Build Path”，选择“configure build path”。  
第二步：点击Libraries选项卡，点击Add Library按钮  
第三步：选择Server Runtime，接着选择tomcat，如图  
第四步：点击Finish，这时候也会看到项目没有报错信息了！
####3.Eclipse中项目右键菜单中点击Maven->Update Projects时JDK被重置###
如果没有在Maven中配置过JDK版本，只是在Eclipse中项目的Properties配置里修改了Java版本，当运行Maven->Update Projects时，Java版本又会被切回默认的1.5。  
运行Maven->Update Project时，会根据maven和pom.xml配置调用eclipse插件重建Eclipse工程文件（.project、.classpath和.settings）。因此Eclipse中的配置会被重置。  
方式一： 可以修改项目的配置，指定编译使用的java版本。
如下配置pom.xml文件：
```
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-compiler-plugin</artifactId>  
            <configuration>  
                <source>1.6</source>  
                <target>1.6</target>  
                <encoding>UTF-8</encoding>  
            </configuration>  
        </plugin>  
    </plugins>  
</build>  
```  
方式二：配置Maven默认使用的JDK版本  
%M2_HOME%/conf/settings.xml文件中加入如下配置：
```
<profiles>   
       <profile>     
            <id>jdk-1.6</id>     
            <activation>     
                <activeByDefault>true</activeByDefault>     
                <jdk>1.6</jdk>     
            </activation>     
            <properties>     
                <maven.compiler.source>1.6</maven.compiler.source>     
                <maven.compiler.target>1.6</maven.compiler.target>     
                <maven.compiler.compilerVersion>1.6</maven.compiler.compilerVersion>     
            </properties>     
    </profile>    
</profiles> 
```
####4.Eclipse新建Maven-Web项目时报错Cannot change version of project facet Dynamic Web Module to 3.0?####  
若是无法选择3.0版本，先将Properties -> Project Facets的Java选为1.7或者更高版本，Apply之后Disable 选项Dynamic Web Module，Apply，之后再选择3.0版本，此时依然报错，因为Maven自动生成的web.xml文件对应的版本为2.3，因此更新web.xml文件为  
```
<web-app xmlns="http://java.sun.com/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee   
          http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"  
    version="3.0">  
    <display-name>Servlet 3.0 Web Application</display-name>  
</web-app>
```
####5.困扰时间最长的一个问题是使用Junit测试连接数据库时一直连接不上，报错####
jdbc.properties那个文件直接复制每行后面会多几个空格，导致数据库连接失败，单元测试通不过。删除空格后Maven->Update Project即可成功