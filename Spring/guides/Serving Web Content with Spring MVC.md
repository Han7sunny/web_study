# Serving Web Content with Spring MVC
##### [원문](https://spring.io/guides/gs/serving-web-content/)         

 ### What You Will Build
 
 + http://localhost:8080/greeting 로 HTTP GET 요청을 받는 정적인 홈페이지를 지니는 application
    + HTML로 보여지는 웹 페이지로 HTML의 body에 "Hello, World!" 포함
 
 + query string에 name parameter를 선택적으로 사용하여 사용자 지정 가능
    + 해당 URL는 http://localhost:8080/greeting?name=User        
 
 + name parameter는 default 값인 World를 override하며 이는 변경된 "Hello, User!"로 응답

### What You Need
+ JDK 1.8 or later

+ Gradle 4+
+ IntelliJ IDEA

### Starting with Spring Initializr
1.  https://start.spring.io.
2.  Gradle , java 선택.
3.  Dependencies : Spring Web, Thymeleaf, Spring DevTools.
4.  Generate and Download zip file

### Create a Web Controller
web site 구축에 대한 Spring의 접근 방식에서 HTTP 요청은 controller에 의해 처리    
```java
package com.example.servingwebcontent;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class GreetingController {

	@GetMapping("/greeting")
	public String greeting(@RequestParam(name="name", required=false, defaultValue="World") String name, Model model) {
		model.addAttribute("name", name); // name 매개변수의 값이 Model 객체에 추가
		return "greeting"; // view template에 접근
	}

}
```
+ GreetingController는 "/greeting" GET request 처리를 View의 이름으로 반환(예제 : "greeting")
+ @Controller : @Controller 어노테이션을 통해 controller를 쉽게 식별
+ @GetMapping : "/greeting"에 대한 HTTP GET request를 greeting() 메소드와 매핑
+ @RequestParam : query string parameter인 name의 값을 greeting() 메소드의 name parameter에 바인딩
	+ 현재 예제에서 query string의 parameter는 required=false로 필수적인 값 아님. request 없으면 defaultValue인 "World" 사용         

+ View는 HTML 내용을 rendering
+ greeting() 메소드 본문의 구현은 HTML 서버측의 렌더링을 수행하기 위해 view technology (해당 예제에선 Thymeleaf)에 의존
```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head> 
    <title>Getting Started: Serving Web Content</title> 
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8" />
</head>
<body>
    <p th:text="'Hello, ' + ${name} + '!'" />
</body>
</html>
```
+ Thymeleaf는 greeting.html(GreetingController의 greeting() 메소드의 반환 view)의 구문 분석
+ th:text 표현식을 평가하여 GreetingController에서 설정된 name 매개변수의 값을 ${name}에 렌더링      

### Spring Boot Devtools
+ web application 개발의 일반적인 기능은 변경 사항을 코딩하고, application을 재시작하며, 변경사항을 보기 위해 browser를 새로 고침함
+ 이러한 전체적인 프로세스는 시간 소모가 크다. 이러한 새로 고침 주기의 속도를 높이기 위해 Spring Boot는 spring-boot-devtools로 알려진 편리한 모듈 제공 

#### Spring Boot Devtools:
+ Hot Swapping 활성화
+ caching 비활성화하도록 template engine 전환
+ browser 자동으로 새로 고침하기 위해 LiveReload 활성화
+ production 대신 개발을 기반으로 한 기타 합리적인 기본값들

### Run the Application
Spring Initializr로부터 제공되는 application class
```java
package com.example.servingwebcontent;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class ServingWebContentApplication {

    public static void main(String[] args) {
        SpringApplication.run(ServingWebContentApplication.class, args);
    }

}
```
+ @SpringBootApplication : 아래와 같은 기능을 함
	+ @Configuration : application context에 대한 bean 정의
	+ @EnableAutoConfiguration : classpath 설정, 다른 bean, 다양한 속성 설정을 기반으로 bean 추가를 Spring Boot에 지시
		+ 예를 들어, spring-webmvc가 classpath에 있다면 해당 어노테이션(@EnableAutoConfiguration)은 해당 application을 web application으로 지정하고 DispatcherServlet 설정과 같은 주요 동작을 활성화
	+ @ComponentScan : com/example 패키지에서 다른 component, configuration, service를 찾도록 지시하여 controller 찾도록 함 -> 위의 GreetingController

### Test the Application
- web site가 실행되고 있을 때, http://localhost:8080/greeting 에 접근하면 "Hello, World!"를 볼 수 있음.
- http://localhost:8080/greeting?name=User 에 접근하면 query string parameter로 name을 제공하여  “Hello, World!” 에서 “Hello, User!” 변경된 것을 볼 수 있음.
- 이것으로 GreetingController의 @RequestParam 설정이 예상대로 작동하고 있다는 것을 보여줌.
- name parameter는 default 값으로 World가 지정되어 있지만 query string을 통해 명시적으로 override 할 수 있음.

### Add a Home Page
+ HTML, JavaScript, CSS를 포함한 정적 리소스는 올바른 위치에 소스 코드를 작성하여 Spring Boot application에서 사용 가능
+ 기본적으로 Spring Boot는 /static (또는 /public)에 있는 classpath의 resources에서 정적 콘텐츠 제공
+ index.html는 welcome page로 ~~~ (pdf 참조)
