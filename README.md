# 스프링 동작 원리
## DispatcherServlet
[WEB-INF] 디렉토리 아래 [web.xml] 이라는 톰켓 설정 파일에 들어가보면 다음과 같은 설정이 있는데, 스프링 프로젝트가 실행되면 가장 먼저 [web.xml]을 읽고 위에서부터 태그를 해석한다.
 ```xml
<!-- The definition of the Root Spring Container shared by all Servlets and Filters -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/spring/root-context.xml</param-value>
	</context-param>
	
	<!-- Creates the Spring Container shared by all Servlets and Filters -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>

	<!-- Processes application requests -->
	<servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>/WEB-INF/spring/appServlet/servlet-context.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
		
	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
 ```
위 설정은 [web.xml]의 대표적인 설정으로, /경로로 들어오는 모든 요청에 대해서 DispatcherServlet의 어떤 메서드가 실행이 되면서 스프링이 요청을 받아들이게 되고, URL에 맞는 컨트롤러를 찾아가게끔 해주는 일을 한다. 컨트롤러를 찾아가도록하게 해주는 것은 어노테이션이라고 부르는데, 이는 나중에 설명하도록 하겠다. 
![screen-shot-initialize-1](https://postfiles.pstatic.net/MjAxODEyMjFfMjM2/MDAxNTQ1Mzc1MjQ3NjQ2.mXixsGbpFL2lpnTvAwDJLv2aaYcEbVbeaLjefnVcKI0g.JWmQ5ByhKarxuOm2Ec3HnP7Vbqcq13Db6IXqDvE7Ivcg.PNG.cheoljin1234/image.png?type=w773)

## 1.[root-context.xml]
-이 파일은 스프링의 환경설정 파일로, 현재는 아무것도 적혀있지 않으니 넘어간다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<!-- Root Context: defines shared resources visible to all other web components -->
		
</beans>
```

## 2.[servlet-context.xml]
  - [web.xml]에서 DispatcherServlet(스프링의 내부 컨틀롤러)객체를 읽고 그 객체의 내부 메서드를 
      실행하면서 이 파일을 참조하게 되는데,  InternalResourceViewResolver는 뷰리졸버의 구현체인 
      UrlBasedViewResolver를 상속받았는데, 이는 사용자에게 보여줄 view를 만들어줄 때, prefix와
      suffix를 설정해준다.
      
  - <context:component-scan>태그는 어노테이션을 해석한다. 이 때, base-packge를 입력해줘야
       하는데, 이는 스프링의 동작방식을 설명하면서 얘기하도록 하겠다.
     
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->
	
	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven />

	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
	<resources mapping="/resources/**" location="/resources/" />

	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
	
	<context:component-scan base-package="com.sts.test3" />
</beans:beans>

```
     
## 3.[HomeController.java]
- [servlet-context.xml]에서 <context:component-scan>태그의 base-package로 com.sts.test3를
      입력해 주었는데, 이 패키지 안에는 HomeController.java라는 객체가 들어있다. 이것은 이 패키지 안
      에 어노테이션이용한 무언가가 있다는 의미인데, 이 자바 파일의 소스 코드를 잘 살펴보면,
      @Controller와 @RequestMapping이라는 어노테이션이 클래스네임이랑 메서드명 위에 적혀있다.
      이 컨트롤러 어노테이션은 디스패쳐서블릿이 클라이언트로부터 어떤 요청이 들어왔을 때 처리하도록 
      클래스를 읽어들일 수 있게 하고, 리퀘스트매핑 어노테이션을 통해 요청의 URL에 해당하는 메서드를
      실행할 수 있도록 하게 해준다.  return 문자열 home은 [servlet-context.xml]에서 지정한 view의
      위치를 리턴할 수 있게 해준다.
```xml
package com.sts.test3;

import java.text.DateFormat;
import java.util.Date;
import java.util.Locale;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

/**
 * Handles requests for the application home page.
 */
@Controller
public class HomeController {
	
	private static final Logger logger = LoggerFactory.getLogger(HomeController.class);
	
	/**
	 * Simply selects the home view to render by returning its name.
	 */
	@RequestMapping(value = "/", method = RequestMethod.GET)
	public String home(Locale locale, Model model) {
		logger.info("Welcome home! The client locale is {}.", locale);
		
		Date date = new Date();
		DateFormat dateFormat = DateFormat.getDateTimeInstance(DateFormat.LONG, DateFormat.LONG, locale);
		
		String formattedDate = dateFormat.format(date);
		
		model.addAttribute("serverTime", formattedDate );
		
		return "home";
	}
	
}
```

## 4.[WEB-INF/views/home.jsp]
- 위 모든 스프링의 동작 과정을 통해 컨트롤러에서 리턴된 경로의 view 객체를 뷰리졸버가 반환을하게 
      되면서 hello world라는 화면이 뜨게 된다
```xml
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ page session="false" %>
<html>
<head>
	<title>Home</title>
</head>
<body>
<h1>
	Hello world!  
</h1>

<P>  The time on the server is ${serverTime}. </P>
</body>
</html>

```
