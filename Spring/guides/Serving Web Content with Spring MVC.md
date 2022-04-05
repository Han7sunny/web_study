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
		model.addAttribute("name", name);
		return "greeting";
	}

}
```
+ @Controller : @Controller 어노테이션을 통해 controller를 쉽게 식별
