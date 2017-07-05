---
title: struts2环境搭建
date: 2017-04-21 15:55:11
categories:
- javaee
tags:
- struts2
- javaee
---
## Struts2 环境搭建
### 下包方法
#### maven方式
直接在eclipse中新建一个mavneproject--->选择webapp那项建立工程
在pom.xml中添加struts2的核心依赖
```xml
<dependency>
			<groupId>org.apache.struts</groupId>
			<artifactId>struts2-core</artifactId>
			<version>2.5.10</version>
</dependency>
```
#### 传统方式下载jar包
[Struts2官网](https://struts.apache.org/)中下载最新jar包 当前版本2.5.10
其中关键性jar包为
* asm-5.1.jar 操作字节码 基于子类的动态代理
* asm-commons-5.1.jar
* asm-tree-5.1.jar
* commons-fileupload-1.3.2.jar  io操作的jar
* commons-io-2.4.jar
* commons-lang3-3.4.jar   对java.lang的扩展
* commons-logging-1.1.3.jar 对日志的扩展
* freemaker-2.3.23.jar 对页面模板的扩展
* javassist-3.20.0-GA.jar  对字节码的支持
* log4j-api-2.7.jar 对日志的扩展
* ognl-3.1.12.jar OGNL表达式
* struts2-core-2.5.10.1.jar 核心库（新版包含xwork无需添加xwork）
<!-- more -->

### 创建工程
* 将上文中的jar包拷入WEB-INFO中的lib，加到builPath
* 在WEB-INFO中创建web.xml 添加 注意StrutsPrepareAndExecuteFilter类的位置在2.5中全限定名为这个
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	id="WebApp_ID" version="3.0">
	 <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```
* 在src中创建struts.xml 如果是maven创建的话应该放在src/main/resourse下
```xml
<?xml version="1.0" encoding="UTF-8"?>    
<!DOCTYPE struts PUBLIC    
            "-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"    
    "http://struts.apache.org/dtds/struts-2.5.dtd">
<struts>
	<package name="s" extends="struts-default">
		<action name="hell" class="com.eason.hello.HelloWorld">
			<result>/hello.jsp</result>
		</action>
	</package>
</struts>
```
* 创建类 创建方法 运行...

### 配置struts.xml  在struts.properties文件中创建重复的属性会覆盖   在web.xml中重复的属性会最终覆盖之前的属性
```xml
	<constant name="struts.action.extension" value="li,," /> //配置后缀
	<constant name="struts.devMode" value="true" />					//开启dev模式 修改配置文件不用重启  更多的错误信息
	<constant name="struts.i18n.encoding" value="UTF-8" />	//编码
	<constant name="struts.enable.DynamicMethodInvocation" value="true" /> //是否开启动态方法调用 动作名称！动作方法名  尽量不启用 方法名称放在url中不安全
```
```xml
<package name="s" extends="struts-default">
	name 属性唯一
	extends 继承struts-default 可以struts的核心功能
	namespace 命名空间 添加之后路径名同事改变
	<action name="hello" class="com.eason.hello.HelloWorld" method="hello">
		name 自定义名称 用于url调用 不可以添加.action
		class 类全限定名称
		method 方法名
		<result name="add">/add.jsp</result>
			name 与动作方法的返回值相同
			type dispatcher 请求转发 默认值
					 redirect  重定向 重定向后地址栏会发生改变-显著特征
					 <result name="success" type="redirect">/redirect.jsp</result>
					 chain 转发到另一个动作
					 同package下转发到另一个动作
					 <action name="hello" class="com.eason.l.HelloWorld">
	 				 		<result name="success" type="chain">action</result>
 					 </action>
 				 	 <action name="action" class="com.eason.l.HelloWorld" method="action">
	 						<result name="action">/success.jsp</result>
 					 </action>
					 不同package下
					 <action name="hello" class="com.eason.l.HelloWorld">
						 	<result name="success" type="chain">
								<param name="namespace">/p2</param> 填写package的name
								<param name="actionName">hello</param> 填写方法名称
							</result>
						</action>
					 redirectAction 重定向到另一个动作 同上除了type不相同
					 <action name="hello_*" class="com.eason.l.HelloWorld" method="{1}">
						 <result type="redirect">/direct.jsp</result>
						 <result name="add" type="redirectAction">hello</result>
						 在2.5之后通配符方法需要添加
						 <allowed-methods>add,delete</allowed-methods>
					 </action>
	</action>
</package>
```
### struts.xml分包管理
* 新建xml文件
* 正常写配置信息
* 在struts.xml 引入
```xml
<include file="struts_action.xml"></include>
```

### servletApi获取
* 推荐直接获取
```java
	private HttpServletResponse response;
	private ServletContext application;
	private HttpSession session;
	private HttpServletRequest request;
	request = ServletActionContext.getRequest();
	response = ServletActionContext.getResponse();
	application = ServletActionContext.getServletContext();
	session = request.getSession();
```
* 实现接口方法
```java
public class ServletAPI extends ActionSupport
		implements ServletContextAware, ServletResponseAware, ServletRequestAware {
			@Override
			public void setServletRequest(HttpServletRequest request){
				this.request = request;
			}
			@Override
			public void setServletResponse(HttpServletResponse arg0) {
				this.response = arg0;
			}
			@Override
			public void setServletContext(ServletContext arg0) {
				this.application = arg0;
			}
		}
```
