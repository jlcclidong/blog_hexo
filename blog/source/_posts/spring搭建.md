---
title: springmvc搭建
date: 2017-07-04 17:02:30
categories:
- javaee
tags:
- spring
- javaee
---
#### 创建工程
* 直接在eclipse中新建一个mavneproject--->选择webapp那项建立工程
* 在pom.xml中添加spring的核心依赖
```xml
<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>4.3.9.RELEASE</version>
</dependency>
<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>4.3.9.RELEASE</version>
</dependency>
```

* 在web.xml中添加配置
```xml
<servlet>
		<servlet-name>spring-mvc</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
<servlet-mapping>
		<servlet-name>spring-mvc</servlet-name>
		<url-pattern>/</url-pattern>
</servlet-mapping>
```

* 在web.xml 同级别创建spring-mvc-servlet.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- Bean头部 -->
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:mvc="http://www.springframework.org/schema/mvc" xmlns:context="http://www.springframework.org/schema/context"
	xmlns:util="http://www.springframework.org/schema/util"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd  
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd  
            http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc-3.0.xsd              
            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd">

	<!-- 激活@Controller模式 -->
	<mvc:annotation-driven />
	<!-- 对包中的所有类进行扫描，以完成Bean创建和自动依赖注入的功能 需要更改 -->
	<context:component-scan base-package="li.hello" />

	<bean
		class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter" />
	<!-- 错误异常的统一处理 id必须是这个-->
	<bean id="handlerExceptionResolver" class="handler.ExceptionHandler" />

	<bean id="viewResolver"
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix">
			<value>/WEB-INF/jsp/</value>
		</property>
		<property name="suffix">
			<value>.jsp</value>
		</property>
	</bean>
</beans>
```

#### 编写Controller
* 新建类 类的包名应该在上面扫描的包注册过

```java
@RequestMapping("/phone/app/test") //请求的地址
@Controller                        //必须标记为controller 才可以被识别
public class Test {

	// http://172.16.0.35:8080/lidong/phone/app/test?name=13&password=133
	// {
	// "name": "13",
	// "password": "133"
	// }
	@RequestMapping(value = "")
	public View test0(Model model, @RequestParam(defaultValue = "", name = "name") String name,
			@RequestParam(required = true) String password) {
		model.addAttribute("name", name);
		model.addAttribute("password", password);
		return new MappingJackson2JsonView();
	}

	// http://172.16.0.35:8080/lidong/phone/app/test/test1?a=3&b=4&c=34&c=111&c=15
	// {
	// "a": "3",
	// "b": 4,
	// "haha": 1,
	// "c": [
	// "34",
	// "111",
	// "15"
	// ]
	// }
	@RequestMapping(value = "/test1", method = RequestMethod.POST)
	public View test1(Model model, String a, int b, String[] c) {
		ArrayList<String> arrayList = new ArrayList<>();
		for (int i = 0; i < c.length; i++) {
			arrayList.add(c[i]);
		}
		model.addAttribute("a", a);
		model.addAttribute("b", b);
		model.addAttribute("c", arrayList);
		model.addAttribute("haha", 1);
		return new MappingJackson2JsonView();
	}

	// http://172.16.0.35:8080/lidong/phone/app/test/test2/lidong/123
	// {
	// "name": "lidong",
	// "password": "123"
	// }
	@RequestMapping(value = "/test2/{name}/{password}", method = RequestMethod.GET)
	public View test2(Model model, @PathVariable String name, @PathVariable String password) {
		model.addAttribute("name", name);
		model.addAttribute("password", password);
		return new MappingJackson2JsonView();
	}

	@RequestMapping(value = "/test3")
	public View test3(Model model, MultipartFile file, String name, String password) {
		String url = "testRecord/" + name + "/" + password + "/" + file.getOriginalFilename();
		String filePath = FileUtils.getAbsoluteFilePath(url);
		copytFile(file, filePath);
		model.addAttribute("result", 1);
		return new MappingJackson2JsonView();
	}

	@RequestMapping(value = "/test4")
	public View test3(Model model, MultipartFile[] file, String name, String password) {
		for (int i = 0; i < file.length; i++) {
			String url = "testRecord/" + name + "/" + password + "/" + file[i].getOriginalFilename();
			String filePath = FileUtils.getAbsoluteFilePath(url);
			copytFile(file[i], filePath);
		}
		model.addAttribute("result", 1);
		return new MappingJackson2JsonView();
	}

  //spring中转移文件的方法
	private void copytFile(MultipartFile file, String dest) {
		try {
			File destFile = new File(dest);
			if (destFile.exists()) {
				destFile.delete();
			}
			if (!destFile.getParentFile().exists()) {
				destFile.getParentFile().mkdirs();
			}
			file.transferTo(destFile);
		} catch (IllegalStateException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}

```

#### springmvc 中返回json的两种方式
* 添加maven依赖
```xml
<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-core</artifactId>
			<version>2.5.4</version>
</dependency>
<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.5.4</version>
</dependency>
```
* 两种方法
  * 第一种@RequestBody
  ```java
  @RequestMapping("test")
	@ResponseBody
	public Map<String, Object> hello() {
		Map<String, Object> map = new HashMap<String, Object>();
		map.put("haha", "222");
		return map;
	}
  ```
  * 第二种
  ```java
  @RequestMapping("test1")
	public View hello1(Model model) {
		model.addAttribute("haha", "222");
		return new MappingJackson2JsonView();
	}
  ```

#### springmvc的错误统一处理
* 如第一步注册
* 实现HandlerExceptionResolver
```java
public class ExceptionHandler implements HandlerExceptionResolver {

	public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler,
			Exception ex) {
		try{
			throw ex;
		}catch (Exception e) {
			ModelAndView modelAndView = new ModelAndView();
			modelAndView.setView(new MappingJackson2JsonView());
			modelAndView.addObject("result", "0");
			modelAndView.addObject("desc", "");
			return modelAndView;
		}
	}
}
```
