# Testing the Web Layer
[원문](https://spring.io/guides/gs/testing-web/)
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
+ AssertJ : assertThat() 등과 같은 메소드를 제공하며 test assertion을 표현
> Spring Test의 좋은 기능은 application context가 test 간에 cache됨
> 테스트 케이스에 여러 메소드가 있거나 동일한 설정에 여러 테스트 케이스가 있는 경우 application을 시작하는데 한 번의 비용만 발생
> @DirtiesContext 어노테이션을 통해 cache를 제어할 수 있음

##### Application의 행동(behavior)을 확인하는 테스트
+ Application을 시작하고 연결을 수신한 뒤 HTTP request를 보내고 response를 확인(assert)
```java
package com.example.testingweb;

import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.boot.web.server.LocalServerPort;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class HttpRequestTest {

	@LocalServerPort
	private int port;

	@Autowired
	private TestRestTemplate restTemplate;

	@Test
	public void greetingShouldReturnDefaultMessage() throws Exception {
		assertThat(this.restTemplate.getForObject("http://localhost:" + port + "/",
				String.class)).contains("Hello, World");
	}
}
```
+ @LocalServerPort : 테스트 환경에서 충돌을 피하는데 유용한 랜덤 포트를 사용하여 server를 시작(WebEnvironment=RANDOM_PORT)하고 @LocalServerPort로 포트를 주입
+ @Autowired : Spring Boot가 자동으로 TestRestTemplate 제공하므로 @Autowired만 추가해주면 됨
    
##### Server 없이 전체 Spring application context 테스트 실행
	
```java
package com.example.testingweb;

import static org.hamcrest.Matchers.containsString;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.web.servlet.MockMvc;

@SpringBootTest
@AutoConfigureMockMvc
public class TestingWebApplicationTest {

	@Autowired
	private MockMvc mockMvc;

	@Test
	public void shouldReturnDefaultMessage() throws Exception {
		this.mockMvc.perform(get("/")).andDo(print()).andExpect(status().isOk())
				.andExpect(content().string(containsString("Hello, World")));
	}
}
```
거의 전체 Spring application context이 시작되며 서버 실행 비용 없이(서버 실행 없이) 실제 HTTP 요청을 처리하는 것과 동일한 방식으로 코드 호출됨
+ @AutoConfigureMockMvc : Spring의 MockMvc 주입되도록 요청

##### Web layer만 테스트 

```java
@WebMvcTest
public class WebLayerTest {

	@Autowired
	private MockMvc mockMvc;

	@Test
	public void shouldReturnDefaultMessage() throws Exception {
		this.mockMvc.perform(get("/")).andDo(print()).andExpect(status().isOk())
				.andExpect(content().string(containsString("Hello, World")));
	}
}
```
+ @WebMvcTest : 테스트 범위를 web layer로만 좁힘
Test assertion이 이전 케이스와 동일하지만 이번 테스트에선 Spring Boot가 전체 context가 아닌 web layer만을 인스턴스화 함
여러 controller가 있는 application에서 @WebMvcTest(HomeController.class)를 사용하여 하나만 인스턴스화 할 수 있음

##### dependency가 없었던 이전의 controller와 달리 인사말을 저장하기 위한 별도의 component 도입 (service)
```java
package com.example.testingweb;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;


@Controller
public class GreetingController {

	private final GreetingService service;

	public GreetingController(GreetingService service) {
		this.service = service;
	}

	@RequestMapping("/greeting")
	public @ResponseBody String greeting() {
		return service.greet();
	}

}
```
```java
package com.example.testingweb;

import org.springframework.stereotype.Service;

@Service
public class GreetingService {
	public String greet() {
		return "Hello, World";
	}
}
```
Spring은 자동으로 controller에 service를 DI함 (Controller의 생성자)
