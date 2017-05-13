#Maven相关问题总结#
##1.安装配置问题#
安装配置参考官网：[https://maven.apache.org/install.html](https://maven.apache.org/install.html "Maven安装")  
1.先保证已经正确安装好JDK，并且设置好JAVA_HOME的环境变量，Windows环境下查看环境变量命令为 set JAVA_HOME ，环境变量地址为JDK安装目录如：C:\Program Files\Java\jdk1.7.0_51
2.将Maven的bin目录添加到系统的path中
3.cmd命令行命令  mvn -v 显示如下表示安装配置成功
```
Apache Maven 3.3.3 (7994120775791599e205a5524ec3e0dfe41d4a06; 2015-04-22T04:57:37-07:00)
Maven home: /opt/apache-maven-3.3.3
Java version: 1.8.0_45, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "mac os x", version: "10.8.5", arch: "x86_64", family: "mac"
```
###2.修改配置文件###
主要修改本地仓库存储位置以及远程仓库地址
1.在合适的目录创建本地仓库作为本地仓库存储位置，拷贝Maven安装目录C:\Program Files\apache-maven-3.5.0\conf下的settings.xml文件到本地仓库，并修改配置文件，本地仓库目录结构如下所示：
![](http://i.imgur.com/kXqCC6z.png)

2.修改settings.xml文件的本地仓库存储位置
```
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
  <localRepository>D:\Maven_Repository\repository</localRepository>
```
3.修改settings.xml文件的远程仓库地址（默认远程仓库地址较慢）
```
  <mirrors>
    <!-- mirror
     | Specifies a repository mirror site to use instead of a given repository. The repository that
     | this mirror serves has an ID that matches the mirrorOf element of this mirror. IDs are used
     | for inheritance and direct lookup purposes, and must be unique across the set of mirrors.
     |
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
     -->
    <mirror>
        <id>nexus-aliyun</id>
        <mirrorOf>*</mirrorOf>
        <name>Nexus aliyun</name>
        <url>http://maven.aliyun.com/nexus/content/groups/public</url>
    </mirror>
  </mirrors>
```
##2.与Eclipse集成使用问题##
####1.将Eclipse默认使用内嵌的Maven改为外部安装的Maven###
Window-> Preferrence -> Maven -> Installations
![](http://i.imgur.com/A0FepzI.png)
并配置使用的setting文件位置：
Window-> Preferrence -> Maven -> User Settings
![](http://i.imgur.com/Fs7wELK.png)
###2.修改Eclipse的Installed JRES###
Eclipse默认运行在JRE上，而Maven需要使用JDK，修改如下：
Window-> Preferrence -> Java -> Installed JRES 添加JDK目录如下图所示
![](http://i.imgur.com/rlRyrGO.png)
不修改的话在使用Eclipse的maven进行build时会报错：
```
[ERROR] COMPILATION ERROR:
[ERROR] No compiler is provided in this environment. Perhaps you are running on a JRE rather than a JDK?
```
###3.Eclipse新建Maven-Web项目时报错Cannot change version of project facet Dynamic Web Module to 3.0?####  
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
之后右键项目Maven->Update 即可成功
####4.Eclipse中项目右键菜单中点击Maven->Update Projects时JDK被重置###
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
###5.使用Eclipse Maven插件将Maven-Web工程打成war包###
由于新建工程时选择的时web工程，，pom.xml文件在开头含有如下配置：
```
  <modelVersion>4.0.0</modelVersion>
  <groupId>web</groupId>
  <artifactId>example</artifactId>
  <packaging>war</packaging>
  <version>0.0.1-SNAPSHOT</version>
  <name>example Maven Webapp</name>
  <url>http://maven.apache.org</url>

	...

  <build>
    <finalName>example</finalName>
  </build>
```
因此直接就可以使用mvn package打包
1.右键pom.xml文件，选择 Run as-> Maven clean(清除之前的打包信息)
2.右键pom.xml文件，选择 Run as-> Maven build... 输入如下参数：
![](http://i.imgur.com/ijx5b5v.png)
点击Run即可，打包成功后会在项目的target目录下生成war包如下图所示
![](http://i.imgur.com/N4HOpV5.png)