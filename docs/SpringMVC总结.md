#**Spring MVC总结**#
----------  
##web.xml 配置文件结构以及功能##
###1.xml头文件,这是使用Web Moudle 3.0以上的配置头文件###
```
<web-app xmlns="http://java.sun.com/xml/ns/javaee"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee   
          http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"  
    version="3.0">  
    <display-name>Servlet 3.0 Web Application</display-name>  
</web-app>
```
###2.其他配置项均写在web-app配置之内###
**	(1)Servlet配置：配置拦截器以及映射拦截地址**
```
<servlet>
<servlet-name>SpringMVC</servlet-name>
<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
<init-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>/WEB-INF/spring-mvc.xml</param-value>
</init-param>
<load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
	<servlet-name>SpringMVC</servlet-name>
	<url-pattern>*.html</url-pattern>
</servlet-mapping>
```
注意上面配置的spring-mvc.xml文件的路径为/WEB-INF/spring-mvc.xml，即是和web.xml文件处于同一目录下。更为常见的配置路径如下：
```
<param-value>classpath:spring-mvc.xml</param-value>  
```
配置的路径为classpath，文件放在src/main/resources文件夹即可。


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
##Spring-mvc.xml配置文件配置##
当web.xml配置文件配置的spring-mvc.xml文件的路径如下时：
```
<param-value>classpath:spring-mvc.xml</param-value>  
```
配置的路径为classpath，文件放在src/main/resources文件夹即可。

##后台Controller向html(或其他前端页面如ftl)传递数据##
###1.使用SpringMVC自带的ModelMap(org.springframework.ui.ModelMap)###
```java
	//后台Controller
	@RequestMapping("/newIndex")
	public String index(HttpServletRequest request, ModelMap modelMap) {
		modelMap.put("deviceNames", deviceNames.toArray());
		...
		modelMap.put("internalNumbers", internalNumbers.toArray());	
		return "devicemanager/newIndex";
	}

----------------------------------------------------------------------------
	//前端newIndex静态页面
	<div class="col-lg-1" style="padding-left: 0px;">
		<select id="deviceNameSelect" class="form-control">
			<option value="all">全部</option> <#if deviceNames?exists> <#list
			deviceNames as deviceName>
			<option value="${deviceName}">${deviceName}</option> </#list>
			</#if>
		</select>
	</div>
```
###2.使用自定义XXModel###
XXModel是自定义的类，放在Model层，用于将前端页面需要的参数值封装在一个对象之中
```java
	//静态页面内容，含有click的js
	otherJs=["/js/bootstrap/js/bootbox.js",
		"/js/usermanager/addUser.js",
		"/js/core/core.js"]

	<div class="row" style="padding-top: 5px; padding-left: 680px; padding-right: 5px;">
		<label for="addUserInfoLabel" class="col-sm-2 control-label">
			<button id="addUserInfo" class="btn btn-primary" onClick="updateUserInfo()">修改用户信息</button>
		</label>
		<div class="col-sm-10"></div>
	</div>
-----------------------------------------------------------------------------
	//利用js中的ajax来处理从静态页面获得的信息，将信息封装为data后传递给后台Controller，
	//success之后处理Controller返回的Map。
	function updateUserInfo() {
		var data = {};
		data.corpMail = $("#corpMail").val();
		data.userName = $("#userName").val();
		data.jobNumber = $("#jobNumber").val();
		data.telephone = $("#telephone").val();
		if (!isNullOrEmpty(data.corpMail) && !isNullOrEmpty(data.userName)) {
			$.ajax({
				url : '/usermanager/updateUserInfo.html',
				type : 'POST',
				dataType : 'json',
				data : data,
				success : function(json) {
					if (json.retCode == 200) {
						bootbox.alert("修改用户信息成功！");
					} else {
						bootbox.alert("修改用户信息失败，请稍后再试！");
					}
				}
			});
		} else {
			bootbox.alert("请填写完整的用户信息！");
		}
	}
--------------------------------------------------------------------------------
	//自定义Model
	public class UserModel {
		private String userID;
		private String corpMail;
		private String userName;
		private String jobNumber;
		private String telephone;
		//各个属性的get、set方法
		get()...
		set()...
	}

	//后台Controller接收js传来的数据，并用自定义类UserModel去匹配各个参数
	@RequestMapping("/updateUserInfo")
	@ResponseBody
	public Map<String, Object> updateUserInfo(UserModel userModel) {
		
		int status = userDaoServiceImpl.updateUserInfo(userModel);
		Map<String, Object> retMap = new HashMap<String, Object>();
		if (status > 0) {
			retMap.put("retCode", 200);
			retMap.put("retDesc", "操作成功");
		} else {
			retMap.put("retCode", -1);
			retMap.put("retDesc", "插入数据库失败");
		}
		return retMap;
	}
```

