# Testing the Web Layer
### Application 개발 및 JUnit을 통한 테스트
+ application context 가동
+ Spring의 MockMvc를 이용한 web layer 테스트

### Create a Simple Application
```java
package com.example.testingweb;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class HomeController {

	@RequestMapping("/")
	public @ResponseBody String greeting() {
		return "Hello, World";
	}

}
```
@RequestMapping
+ Default는 모든 HTTP 작업과 매핑되어 GET, POST, PUT 등을 명시하지 않음
+ 매핑을 좁히려면 @GetMapping 또는 @RequestMapping(method=GET) 사용

### Run the Application
```java
package com.example.testingweb;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class TestingWebApplication {

	public static void main(String[] args) {
		SpringApplication.run(TestingWebApplication.class, args);
	}
}
```
Spring Initializer가 기본적으로 application class 생성해줌
+ @SpringBootApplication : 아래의 기능들을 포함
+ 
