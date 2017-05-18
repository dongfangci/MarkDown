#JavaWeb相关疑惑解答#
###1.JavaWeb与Java程序的区别###
###2.JavaWeb的classpath###
顾名思义是java编译后class文件的存放目录，web项目默认是在WEB-INF/classes.
首先要知道Web项目的目录结构，可以在Tomcat的目录下查看运行的项目的目录，也是项目打包之后，target目录下以项目名命名的目录：
如下所示：
![](http://i.imgur.com/OqE0bOL.png)  
项目名称为example，即为根目录/
下级目录包含/WEB-INF、   /META-INF、   index.jsp  三个文件
其中/WEB-INF目录下面包含classes、jsp、lib、web.xml文件
classes的目录之下即为classpath，也就是说classpath目录为/WEB-INF/classes，由于Web项目设置部署输出路径为
![](http://i.imgur.com/3z30DLY.png)

即所有maven工程的 src/main/resources 资源文件夹以及源码文件夹src/main/java/下面的文件编译后都会在此，所以在开发时常将相应的xml配置文件放于src或其子目录下；  
###引用classpath路径下的文件，只需在文件名前加classpath:(需保证该文件确实位于classpath路径下);
```
<param-value>classpath:applicationContext-*.xml</param-value> 
```
或者引用其子目录下的文件,如  :
```
<param-value>classpath:context/conf/controller.xml</param-value> 
```
### classpath 和 classpath* 区别： 
classpath：只会到你的class路径中查找找文件; 
classpath*：不仅包含class路径，还包括jar文件中(class路径)进行查找. 
classpath* 的使用：当项目中有多个classpath路径，并同时加载多个classpath路径下（此种情况多数不会遇到）的文件，*就发挥了作用，如果不加*，则表示仅仅加载第一个classpath路径，代码片段： 
```
<param-value>classpath*:context/conf/controller*.xml</param-value>  
```
另外： 
"**/" 表示的是任意目录； 
"**/applicationContext-*.xml"  表示任意目录下的以"applicationContext-"开头的XML文件。  
程序部署到tomcat后，src目录下的配置文件会和class文件一样，自动copy到应用的 WEB-INF/classes目录下 
classpath:与classpath*:的区别在于， 

前者只会从第一个classpath中加载，而 
后者会从所有的classpath中加载  

如果要加载的资源， 
不在当前ClassLoader的路径里，那么用classpath:前缀是找不到的， 
这种情况下就需要使用classpath*:前缀 

在多个classpath中存在同名资源，都需要加载， 
那么用classpath:只会加载第一个，这种情况下也需要用classpath*:前缀 

注意： 
用classpath*:需要遍历所有的classpath，所以加载速度是很慢的，因此，在规划的时候，应该尽可能规划好资源文件所在的路径，尽量避免使用 classpath* 
我们可以查看项目根路径下的.classpath文件如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<classpath>
	<classpathentry kind="src" output="target/classes" path="src/main/java">
		<attributes>
			<attribute name="optional" value="true"/>
			<attribute name="maven.pomderived" value="true"/>
		</attributes>
	</classpathentry>
	<classpathentry excluding="**" kind="src" output="target/classes" path="src/main/resources">
		<attributes>
			<attribute name="maven.pomderived" value="true"/>
		</attributes>
	</classpathentry>
	<classpathentry kind="src" output="target/test-classes" path="src/test/java">
		<attributes>
			<attribute name="optional" value="true"/>
			<attribute name="maven.pomderived" value="true"/>
		</attributes>
	</classpathentry>
	<classpathentry kind="con" path="org.eclipse.m2e.MAVEN2_CLASSPATH_CONTAINER">
		<attributes>
			<attribute name="maven.pomderived" value="true"/>
			<attribute name="org.eclipse.jst.component.dependency" value="/WEB-INF/lib"/>
		</attributes>
	</classpathentry>
	<classpathentry kind="con" path="org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8">
		<attributes>
			<attribute name="maven.pomderived" value="true"/>
		</attributes>
	</classpathentry>
	<classpathentry kind="output" path="target/classes"/>
</classpath>
```
###进一步解释###
把配置文件如：struts.xml、applicationContext.xml等放到src/main/resources目录,然后使用“classpath：xxx.xml”来读取，都放到src目录准没错
曾经试过把配置文件放到WEB-INF目录，然后以路径“/WEB-INF/xxxx.xml”来成功读取配置文件
但是在用ClassPathXmlApplicationContext()函数不能读取这样的路径，用上面的方法才能成功读取配置文件
回答：
 
【01】 src路径下的文件在编译后会放到WEB-INF/clases路径下吧。默认的classpath是在这里。直接放到WEB-INF下的话，是不在classpath下的。用ClassPathXmlApplicationContext当然获取不到。
【02】 如果单元测试的话，可以在启动或者运行的选项里指定classpath的路径的。用maven构建项目时候resource目录就是默认的classpath
【03】 classPath即为java文件编译之后的class文件的编译目录一般为web-inf/classes，src下的xml在编译时也会复制到classPath下
（1）ApplicationContext ctx = new ClassPathXmlApplicationContext("xxxx.xml");  //读取classPath下的spring.xml配置文件
（2）ApplicationContext ctx = new FileSystemXmlApplicationContext("WebRoot/WEB-INF/xxxx.xml");   //读取WEB-INF 下的spring.xml文件
而classpath可以通过如下代码获取:
```
MyClass.class.getClassLoder().getResource("").getPath();
```
 			

