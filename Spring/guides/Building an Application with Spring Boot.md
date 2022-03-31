# Building an Application with Spring Boot

##### [원문](https://spring.io/guides/gs/spring-boot/)

### requirement     
+ JDK 1.8 or later       
+ Gradle 4+     
+ IntelliJ IDEA        

### Create an Application class
```java
package com.example.springboot;

import java.util.Arrays;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args); // Application 실행
	}

	@Bean
	public CommandLineRunner commandLineRunner(ApplicationContext ctx) {
		return args -> {

			System.out.println("Let's inspect the beans provided by Spring Boot:");

			String[] beanNames = ctx.getBeanDefinitionNames();
			Arrays.sort(beanNames);
			for (String beanName : beanNames) {
				System.out.println(beanName);
			}
		};
	}
}
```
+ @SpringBootApplication : @EnableAutoConfiguration, @ComponentScan, @Configuration과 동일한 기능을 하는 어노테이션        
	+ @EnableAutoConfiguration : Spring Boot의 auto-configuration mechanism 실행 
 		+ auto-configuration : 추가된 jar dependencies 기반으로 Spring Application 자동적으로 설정
 	+ @ComponentScan : Application이 위치한 com.example.springboot 하위 패키지 클래스에 component scan 실행(자동 bean 등록)
 	+ @Configuration : Application context를 위한 bean 설정 클래스에 표기
                
### Create a Web Controller for a simple Web Application
```java
package com.example.springboot;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

	@GetMapping("/")
	public String index() {
		return "Greetings from Spring Boot!";
	}
}
``` 
    
+ @RestController : @Controller, @ResponseBody를 결합한 기능 -> web request에 view가 아닌 data 반환
	+ @Controller : Component Scan으로 Spring Bean 자동 등록
	+ @ResponseBody : viewResolver 대신 HttpMessageConverter 동작
		+ HTTP의 BODY에 문자 내용 직접 반환
		+ 기본 문자처리 : StringHttpMessageConverter
		+ 기본 객체처리 : MappingJackson2HttpMessageConverter
+ @GetMapping : HTTP GET request가 들어올 때 @GetMapping을 작성한 메소드 호출    

### Add Unit Tests
##### 추가한 endpoint(Controller)에 대한 Unit Test(단위 테스트)
```java
// build.gradle의 dependency 추가
testImplementation('org.springframework.boot:spring-boot-starter-test') // Junit4 사용을 위함
```      
##### endpoint를 통해 Servlet request 및 response를 mock하는 간단한 단위 테스트 
```java
package com.example.springboot;

import static org.hamcrest.Matchers.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;

@SpringBootTest
@AutoConfigureMockMvc
public class HelloControllerTest { // 앞의 HelloController에 대한 테스트

	@Autowired
	private MockMvc mvc;

	@Test
	public void getHello() throws Exception {
		mvc.perform(MockMvcRequestBuilders.get("/").accept(MediaType.APPLICATION_JSON)) // perform() : requeset 전송, accept(MediaType.APPLICATION_JSON) : response JSON으로 반환
				.andExpect(status().isOk()) // andExpect() : request 전송 결과(response)를 검증 및 확인, isOk() : 상태 코드 200
				.andExpect(content().string(equalTo("Greetings from Spring Boot!"))); // content() : response 정보 검증
	}
}
```
MockMvc : DispatcherServlet에 HTTP request를 보내고 결과를 확인 (원하는 결과 제대로 반환하는지 확인)
	+ MockMvc 인스턴스를 주입하기 위해 @SpringBootTest와 @AutoConfigureMockMvc 사용 또는 @WebMvcTest 사용
+ @SpringBootTest : Spring Container와 Test 함께 실행, Application context를 생성 (의존성 제공), 통합 테스트를 제공하는 기본적인 spring test annotation, @WebMvcTest와 달리 @Service, @Repository 어노테이션이 붙은 객체 또한 메모리에 올림
	+ @WebMvcTest : web layer만 생성하여 테스트, @SpringBootTest와 같이 사용될 수 없음, Controller 테스트에 적합
+ @AutoConfigureMockMvc : Mock 테스트에 필요한 의존성 제공, MockMvc 생성
+ @Autowired : 생성된 MockMvc bean 주입 받음
+ @Test : 해당 메서드마다 단위 테스트 진행

### Full-stack Integration Test
##### Spring Boot를 사용하여 통합 테스트 진행, HTTP request cycle mocking (Rest)
```java
package com.example.springboot;

import org.junit.jupiter.api.Test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.http.ResponseEntity;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT) // 내장된 server가 임의의 port로 시작 -> 실제 Servlet 환경 구성,,처럼?
public class HelloControllerIT {

	@Autowired
	private TestRestTemplate template;

    	@Test
    	public void getHello() throws Exception {
        	ResponseEntity<String> response = template.getForEntity("/", String.class);
        	assertThat(response.getBody()).isEqualTo("Greetings from Spring Boot!"); // "/" request -> HelloController의 index() 반환값과 동일한지 확인
    	}
}
```
     
@SpringBootTest의 WebEnvironment 설정
	+ Mock 서블릿 환경이 아닌 내장 Server 구동하여 테스트하기 위하여 설정
	+ WebEnvironment.MOCK : Default값으로 Mock 서블릿 환경으로 Server 구동 X
	+ WebEnvironment.RANDOM_PORT : Server가 랜덤포트로 구동
실제 port는 TestRestTemplate의 기본 url에서 자동적으로 설정
+ TestRestTemplate : 테스트용 클라이언트
	+ 위의 WebEnvironment 설정으로 내장 Server 구동되어 MockMvc 사용할 수 없으니 테스트를 위한 HTTP request를 보낼 수 있는 클라이언트 필요
	
### Add Production-grade Services
##### management service 추가
```java
// build.gradle의 dependency 추가
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```   
##### Restart the application
```cmd
./gradlew bootRun
```
+ RESTful end point들이 추가됨 -> Spring Boot가 제공하는 management services
+ http://localhost:8080/actuator/health 또는 http://localhost:8080/actuator로 접근 가능
