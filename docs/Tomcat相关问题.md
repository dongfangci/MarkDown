#Tomcat相关问题#
##1.Tomcat安装配置问题##
Windows64位系统下载64bit版本，之后解压到目录，要求之前已经配置好JAVA_HOME的环境变量。
现在的版本目前没有配置CATALINA_HOME的环境变量
##2.与Eclipse Web项目集成的问题##
Tomcat最初安装后默认的配置文件C:\Program Files\apache-tomcat-7.0.77\conf\server.xml部分如下所示：
```
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">

        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
               prefix="localhost_access_log." suffix=".txt"
               pattern="%h %l %u %t &quot;%r&quot; %s %b" />

      </Host>
```
<Host>标签中没有Context标签
###1.为项目添加Tomcat服务器###
右键项目 Run As -> Run on Server
![](http://i.imgur.com/QkjbXXD.png)
在Configure runtime environment 中配置Tomcat安装的路径即可。
此时Eclipse将自动生成Servers工程如下图所示：
![](http://i.imgur.com/xxIdyY2.png)

此时查看C:\Program Files\apache-tomcat-7.0.77\conf\server.xml会发现发生改变（也可以直接查看Servers目录下的server.xml）
```
      <Host appBase="webapps" autoDeploy="true" name="localhost" unpackWARs="true">

        <!-- SingleSignOn valve, share authentication between web applications
             Documentation at: /docs/config/valve.html -->
        <!--
        <Valve className="org.apache.catalina.authenticator.SingleSignOn" />
        -->

        <!-- Access log processes all example.
             Documentation at: /docs/config/valve.html
             Note: The pattern used is equivalent to using pattern="common" -->
        <Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs" pattern="%h %l %u %t &quot;%r&quot; %s %b" prefix="localhost_access_log." suffix=".txt"/>

      <Context docBase="example" path="" reloadable="true" source="org.eclipse.jst.jee.server:example"/>
	</Host>
```
发现相比之前添加了Context标签，Context标签包含项目名信息以及eclipse的相关信息
###2.修改Eclipse默认的Tomcat项目部署目录###
默认的目录为Eclipse的Workspace下的子目录，不方便观察，因此改为Tomcat的相关目录。
在Servers视图，Remove刚刚发布的项目
![](http://i.imgur.com/FsWlECH.png)
双击Servers，选择ServerLocations的目录为中间一项，Deploy path改为webapps。
![](http://i.imgur.com/TPTkHs8.png)
Web Modules一栏选择path为/,而不是项目名
![](http://i.imgur.com/UIH725R.png)
###3.手工部署war包到Tomcat目录下###
1.参考Maven相关问题总结使用Eclipse Maven插件将Maven-Web工程打成war包。也可以直接使用cmd的命令行，将cmd目录切换至项目的pom.xml文件所在的目录，执行以下命令进行打包，得到的war包在项目的target目录下，名字为项目名。
```
mvn package
```
若想打包过程中跳过测试，采用下面的命令
```
mvn package -Dmaven.test.skip=true
```
2.将war包拷贝到Tomcat安装目录的webapps目录下
3.单击bin目录下的start.bat，启动Tomcat。
4.在浏览器中打开localhost:8080/项目名称/index.jsp(localhost:8080/项目名称)即可.  
#####5.若想直接通过localhost:8080/index.jsp(localhost:8080)访问，不需要添加项目名，则需要修改Tomcat的server.xml文件，注意备份文件（Eclipse已经修改过server.xml）,修改<Host>标签下的Context标签（若没有则添加）
```
<Context path="/" docBase="example" debug="0" privileged="true"/>
```
其中dcoBase为项目名称。
#####6.若想直接通过localhost访问，省略8080，####则同样需要修改Server.xml文件，将端口号改为http协议默认的80即可
```
    <Connector connectionTimeout="20000" port="80" protocol="HTTP/1.1" redirectPort="8443"/>
```
8080是tomcat安装之后默认的访问端口号，而http协议的默认端口号位80 ，如访问www.baidu.com，浏览器其实访问的是www.baidu.com:80,因此将Tomcat端口号改为80之后可以直接通过localhost访问，无需添加端口号。
#####7.若想通过静态IP访问Tomcat，修改<Host>标签的name属性即可
```
<Host appBase="webapps" autoDeploy="true" name="192.168.0.1" unpackWARs="true">
...
</Host>
```
###注意###
一旦修改过server.xml文件，在Eclipse中重新Run on Server时会失败，要么在项目中重新添加Server，要么将配置文件改回Eclipse修改后的状态。
