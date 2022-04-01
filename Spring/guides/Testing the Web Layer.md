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
	+ @Configuration : Bean 정의된 클래스에 지정
	+ @EnableAutoConfiguration : classpath를 기준으로 Bean 추가
	+ @EnableWebMvc : 해당 application을 web application으로 지정하고 DispatcherServlet 설정과 같은 주요 동작 활성화
	+ @Component : 하위 패키지의 컴포넌트, 설정, 서비스들을 검색

### Test the Application
##### 변화를 줄 때마다 application이 작동하는 것을 확신하기 위하여 테스트 자동화 작업
```java
package com.example.testingweb;

import org.junit.jupiter.api.Test;

import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class TestingWebApplicationTests {

	@Test
	public void contextLoads() {
	}

}
```
Application 내용을 실행할 수 없는 경우 실패하는 간단한 검사 테스트
+ @SpringBootTest : Spring Boot가 메인 설정 클래스(ex. @SpligBootApplication이 있는 클래스)를 찾아 Spring application context를 실행하도록 함

##### context가 controller를 생성하고 있음을 확인하기 위해 assertion 추가할 수 있음
```java
package com.example.testingweb;

import static org.assertj.core.api.Assertions.assertThat;

import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class SmokeTest {

	@Autowired
	private HomeController controller;

	@Test
	public void contextLoads() throws Exception {
		assertThat(controller).isNotNull();
	}
}
```
+ @Autowired : controller는 테스트 메소드가 실행되기 전에 주입됨
+ AssertJ : assertThat() 등과 같은 메소드를 제공하며 
