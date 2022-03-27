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
		SpringApplication.run(Application.class, args); // application 실행 
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
+ @SpringBootApplication : @EnableAutoConfiguration, @ComponentScan, @Configuration 기능을 하는 어노테이션
 + @EnableAutoConfiguration : Spring Boot의 auto-configuration mechanism 실행 
  + auto-configuration : 추가된 
 + @ComponentScan : Application이 위치한 com.example.springboot 패키지 이하의 클래스에 component scan 실행
 + @Configuration 
