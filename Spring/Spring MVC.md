# Spring MVC

### **Front Controller 패턴**

- 프론트 컨트롤러 서블릿 하나로 클라이언트 요청을 받는다.
- 프론트 컨트롤러가 요청에 맞는 컨트롤러를 찾아서 호출한다.
- 입구를 하나로 하여  공통 처리가 가능하다.
- 프론트 컨트롤러를 제외한 나머지 컨트롤러는 서블릿을 사용하지 않아도 된다.

Spring Web MVC의 **DispatcherServlet**이 FrontController 패턴으로 구현되어 있다.           
                   
### DispatcherServlet

부모 클래스에서 HttpServlet을 상속 받아 사용하며 서블릿으로 동작한다.

Spring Boot는 DispatcherServlet을 서블릿으로 자동 등록하며 모든 경로(urlPatterns=”/”)에 대해 매핑한다. (더 자세한 경로가 우선순위 높음)

**요청 흐름**

- 서블릿이 호출되면 HttpServlet이 제공하는 service() 호출된다.
- Spring MVC는 DispatcherServlet의 부모인 FrameworkServlet에서 service()를 오버라이드 해두었다.
- FrameworkServlet.service()를 시작으로 여러 메소드가 호출되며 DispatcherServlet.doDispatcher()가 호출된다.



forward

: 서버 내부에서 일어나는 호출이기에 클라이언트가 전혀 인지하지 못하며 URL 경로가 변경되지 않는다.

: 다른 서블릿이나 JSP로 이동할 수 있는 기능

: Servlet에서 JSP 호출
              
                
### SpringMVC 구조

**동작 순서**

1. 핸들러 조회 : 핸들러 매핑을 통해 요청 URL에 매핑된 핸들러(컨트롤러)를 조회한다.
2. 핸들러 어댑터 조회 : 1을 통한 핸들러를 실행할 수 있는 핸들러 어댑터를 조회한다.
3. 핸들러 어댑터 실행 : 핸들러 어댑터를 실행한다.
4. 핸들러 실행 : 핸들러 어댑터가 실제 핸들러(컨트롤러)를 실행한다.
5. ModelAndView 반환 : 핸들러 어댑터는 핸들러가 반환하는 정보를 ModelAndView로 변환해서 반환한다.
6. viewResolver 호출 : View Resolver를 찾고 실행한다.
    1. JSP의 경우 : InternalResourceViewResolver가 자동 등록되고 사용된다.
7. View 반환 : View Resolver는 View의 논리 이름을 물리 이름으로 바꾸고 렌더링 역할을 담당하는 뷰 객체를 반환한다.
    1. JSP의 경우 : InternalResourceView(JstlView)를 반환하는데 내부에는 forward() 로직이 있다.
    2. 스프링 부트는 InternalResourceViewResolver 라는 뷰 리졸버를 자동으로 등록하는데, 이때 application.properties 에 등록한 spring.mvc.view.prefix , spring.mvc.view.suffix 설정 정보를 사용해서 등록한다.
8. View 렌더링 : View를 통해서 View를 렌더링한다.
                    
                     
### Spring MVC - 기본 기능

**요청 매핑**

`@RestController`

- `@Controller`는 반환값이 String일 경우 뷰 이름으로 인식하여 뷰를 찾고 뷰가 렌더링된다.
- 그와 달리 `@RestController`는 반환값이 HTTP 메시지 바디에 바로 입력된다.
    
    ```java
    package hello.servlet.basic.requestmapping;
    
    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.RestController;
    
    @RestController
    public class MappingController {
    
        private Logger log = LoggerFactory.getLogger(getClass());
    
        @RequestMapping("/hello-basic")
        public String helloBasic(){
            log.info("helloBasic");
            return "ok";
        }
    }
    ```
    

`@RequestMapping`

- URL 호출이 오면 해당 클래스 또는 메서드가 실행되도록 매핑한다.
- 대부분의 속성을 배열로 제공하므로 다중 설정이 가능하다.
- ex. /hello-basic과 /hello-basic/은 다른 URL이지만 스프링은 같은 요청으로 매핑한다.
- method 속성으로 HTTP 메서드를 지정하지 않으면 HEAD, GET, POST, PUT, PATCH, DELETE 모두 허용
- HTTP 메서드 매핑
    
    ```java
    @RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
    public String mappingGetV1(){
    		log.info("mappingGetV1");
        return "ok";
    }
    ```
    
- HTTP 메서드를 축약한 어노테이션 사용
    
    ```java
    @GetMapping("/mapping-get-v2")
        public String mappingGetV2(){
            log.info("mappingGetV2");
            return "ok";
        }
    ```
    
- `@RequestMapping`은 URL 경로를 템플릿화 할 수 있는데 `@PathVariable`을 사용하여 매칭되는 부분을 편리하게 조회할 수 있다.
- `@PathVariable`
    - @PathVariable의 이름과 파라미터 이름이 같으면 생략할 수 있다.
    
    ```java
    @GetMapping("/mapping/{userId}")
    public String mappingPath(@PathVariable("userId") String data) { // @PathVariable String userId
    		log.info("mappingPath userId={}",data);
        return "ok";
    }
    ```
    
    - 다중 사용 가능
    
    ```java
    @GetMapping("/mapping/users/{userId}/orders/{orderId}")
    public String mappingPath(@PathVariable String userId, @PathVariable Long orderId) {
    		log.info("mappingPath userId={}, orderId={}", userId, orderId);
    		return "ok";
    }
    ```
    

**요청 매핑 - API 예시**

```java
package hello.springmvc.basic.requestmapping;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/mapping/users")
public class MappingClassController {
 
 // GET /mapping/users
 @GetMapping
 public String users() {
		 return "get users";
 }

 // POST /mapping/users
 @PostMapping
 public String addUser() {
		 return "post user";
 }

 // GET /mapping/users/{userId}
 @GetMapping("/{userId}")
 public String findUser(@PathVariable String userId) {
		 return "get userId=" + userId;
 }

 // PATCH /mapping/users/{userId}
 @PatchMapping("/{userId}")
 public String updateUser(@PathVariable String userId) {
		 return "update userId=" + userId;
 }

 // DELETE /mapping/users/{userId}
 @DeleteMapping("/{userId}")
 public String deleteUser(@PathVariable String userId) {
		 return "delete userId=" + userId;
 }
}
```

- 클래스 레벨에 매핑 정보를 두면 (`@RequestMapping("/mapping/users")`) 메서드 레벨에서 해당 정보를 조합하여 사용한다.

GET 쿼리 파라미터 전송과 POST HTML Form 전송 방식은 둘 다 형식이 같으므로 구분없이 조회할 수 있다. 이를 **요청 파라미터** **조회** 라고 한다.

```java
package hello.springmvc.basic.request;
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.Map;

@Slf4j
@Controller
public class RequestParamController {
 
@RequestMapping("/request-param-v1")
public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
				 String username = request.getParameter("username");
				 int age = Integer.parseInt(request.getParameter("age"));
				 log.info("username={}, age={}", username, age);
				 response.getWriter().write("ok");
		 }
}
```

HttpServletRequest 의 request.getParameter() 를 사용하면 GET 쿼리 파라미터 전송과 POST HTML Form 전송 방식의 요청 파라미터를 조회할 수 있다.

스프링이 제공하는 **@RequestParam** 을 사용하면 요청 파라미터를 매우 편리하게 사용할 수 있다.

```java
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2( @RequestParam("username") String memberName, // request.getParameter("username")
															@RequestParam("age") int memberAge) {
		 log.info("username={}, age={}", memberName, memberAge);
		 return "ok";
}
```

- `@RequestParam` : 파라미터 이름으로 바인딩
- `@ResponseBody` : View 조회를 무시하고 HTTP message body에 직접 해당 내용 입력

```java
@ResponseBody
@RequestMapping("/request-param-v3")
public String requestParamV3(@RequestParam String username, @RequestParam int age) {
			log.info("username={}, age={}", username, age);
			return "ok";
}
```

- HTTP 파라미터 이름과 변수 이름이 같을 경우 `@RequestParam`의 name 속성 생략 가능
- String, int, Integer 등의 단순 타입이면 `@RequestParam`도 생략 가능
    - `@RequestParam` 생략하면 스프링 MVC는 내부에서 `required=false` 적용

```java
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamRequired(@RequestParam(required = true) String username,
																	 @RequestParam(required = false) Integer age) {
			log.info("username={}, age={}", username, age);
			return "ok";
}
```

- `@RequestParam(required = true)` : 파라미터 필수 여부 Default = true
    - 해당 파라미터 존재하지 않으면 400 예외 발생
- 예) “/request-param-required?username=” 요청 들어올 경우
    - 파라미터 이름만 있고 값이 없는 경우 빈문자(””)로 예외 발생하지 않고 통과한다.
- `@RequestParam(required = false)` : 파라미터 값 없을 경우 null 입력된다.
    - 따라서 int가 아닌 Integer로 입력 받거나 `defaultValue` 사용

```java
@ResponseBody
@RequestMapping("/request-param-default")
public String requestParamDefault(
							@RequestParam(required = true, defaultValue = "guest") String username,
							@RequestParam(required = false, defaultValue = "-1") int age) {
			log.info("username={}, age={}", username, age);
			return "ok";
}
```

- `defaultValue`를 사용하면 파라미터에 값이 없는 경우 기본 값을 적용할 수 있다.
    - 이미 기본값이 있기 때문에 `required` 는 의미가 없다.
    - 파라미터 값이 빈 문자인 경우에도 설정한 기본값이 적용된다.
        - 예) “/request-param-default?username=” → username = guest

```java
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamMap(@RequestParam Map<String, Object> paramMap) {
		log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));
		return "ok";
}
```

- 파라미터를 `Map`, `MultiValueMap`으로 조회할 수 있다.
- 파라미터의 값이 1개가 확실하다면 `Map`을 사용해도 되지만 그렇지 않다면 `MultiValueMap` 사용

요청 파라미터를 받아 필요한 객체를 만들고 그 객체에 값을 넣어주어야 하는데 이러한 과정을 `@ModelAttribute`가 자동화해준다.

우선 요청 파라미터를 바인딩 받을 객체 클래스를 만들어준다.

```java
package hello.springmvc.basic;
import lombok.Data;

@Data
public class MyData {
		private String username;
		private int age;
}
```

- Lombok의 `@Data`
    - `@Getter` , `@Setter` , `@ToString` , `@EqualsAndHashCode` , `@RequiredArgsConstructor` 를
    자동으로 적용해준다.

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute MyData myData) {
		log.info("username={}, age={}", myData.getUsername(), myData.getAge());
		return "ok";
}
```

model.addAttribute(myData) 코드도 함께 자동 적용된다. → 뒤에 model에서 자세히 설명

스프링 MVC는 `@ModelAttribute` 어노테이션이 있으면

 1. MyData  객체를 생성한다.

1. 요청 파라미터의 이름으로 MyData  객체의 *property*를 찾고 해당 *property*의 setter를 호출하여 파라미터의 값을 입력(바인딩)한다.
    - *property*란?
        
        객체에 getUsername() , setUsername() 메서드가 있으면, 이 객체는 username 이라는 프로퍼티를 가지고 있다.
        username 프로퍼티의 값을 변경하면 setUsername() 이 호출되고, 조회하면 getUsername() 이 호출된다.
        

`@ModelAttribute` 은 `@RequestParam`과 마찬가지로 생략할 수 있는데 스프링은 생략시 다음과 같은 규칙을 적용한다.

- String , int , Integer 같은 단순 타입은 `@RequestParam`
- 나머지는 `@ModelAttribute` (argument resolver 로 지정해둔 타입은 제외

**HTTP message body에 데이터를 직접 담아 요청**

HTTP API에서 주로 사용하며 JSON, XML, TEXT가 있다.

데이터 형식은 주로 JSON을 사용하며 POST, PUT, PATCH 방식이 있다.

요청 파라미터와 다르게 HTTP message body를 통해 데이터가 직접 넘어오는 경우`@RequestParam` , `@ModelAttribute` 를 사용할 수 없다. (HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정)

**단순 텍스트**

```java
@Slf4j
@Controller
public class RequestBodyStringController {

@PostMapping("/request-body-string-v1")
public void requestBodyString(HttpServletRequest request,HttpServletResponse response) throws IOException {
			ServletInputStream inputStream = request.getInputStream();
			String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
			log.info("messageBody={}", messageBody);
			response.getWriter().write("ok");
		}
}
```

기본적으로 HTTP message body의 데이터를 `request.getInputStream()`를 통해 직접 읽을 수 있다.

- `request.getInputStream()` : HTTP message body의 내용을 byte 코드로 반환한다.
- byte 코드를 우리가 읽을 수 있는 문자(String)로 보기 위해 문자표(Charset) 지정해야 한다. 해당 코드에서는 UTF-8 Charset을 지정해주었다.

```java
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {
			String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
			log.info("messageBody={}", messageBody);
			responseWriter.write("ok");
}
```

- `InputStream`(Reader): HTTP 요청 메시지 바디의 내용을 직접 조회
- `OutputStream`(Writer): HTTP 응답 메시지의 바디에 직접 결과 출력

```java
@PostMapping("/request-body-string-v3")
public HttpEntity<String> requestBodyStringV3(HttpEntity<String> httpEntity) {
			String messageBody = httpEntity.getBody();
			log.info("messageBody={}", messageBody);
			return new HttpEntity<>("ok");
}
```

- `HttpEntity`
    - HTTP header, body 정보를 편리하게 조회할 수 있다.
    - 요청 파라미터를 조회하는 기능과 관계 없음(`@RequestParam`, `@ModelAttribute`)
    - 응답에도 사용하며 메시지 바디 정보 직접 반환 가능하다.
    - 스프링 MVC 내부에서 HTTP 메시지 바디를 읽어서 HTTP 메시지 컨버터(HttpMessageConverter)로 문자나 객체로 변환한다.
    

```java
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) {
			log.info("messageBody={}", messageBody);
			return "ok";
}
```

`@RequestBody` : HTTP 메시지 바디 정보를 편리하게 조회할 수 있다.

- `@RequestHeader` : 헤더 정보 조회

`@ResponseBody` : HTTP 메시지 바디에 응답 결과를 직접 담아 전달할 수 있다.

**JSON**

```java
package hello.springmvc.basic.request;
import com.fasterxml.jackson.databind.ObjectMapper;
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.util.StreamUtils;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;
import javax.servlet.ServletInputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
/**
 * {"username":"hello", "age":20}
 * content-type: application/json
 */

@Slf4j
@Controller
public class RequestBodyJsonController {
private ObjectMapper objectMapper = new ObjectMapper();

@PostMapping("/request-body-json-v1")
public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
			ServletInputStream inputStream = request.getInputStream();
			String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
			log.info("messageBody={}", messageBody);
			HelloData data = objectMapper.readValue(messageBody, HelloData.class);
			log.info("username={}, age={}", data.getUsername(), data.getAge());
			response.getWriter().write("ok");
		}
}
```

- `request.getInputStream()` : `HttpServletRequest` 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와 문자로 변환한다.
- `objectMapper.readValue()` : 문자로 변환된 JSON 데이터를 Jackson 라이브러리인 `objectMapper` 를 사용해서 자바 객체로 변환한다.

```java
package hello.springmvc.basic;
import lombok.Data;

@Data
public class MyData {
		private String username;
		private int age;
}
```

```java
@ResponseBody
@PostMapping("/request-body-json-v2")
public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {
			MyData data = objectMapper.readValue(messageBody, MyData.class);
			log.info("username={}, age={}", data.getUsername(), data.getAge());
			return "ok";
}
```

- `@RequestBody` : HTTP 메시지 바디에서 데이터를 읽어 매개변수에 저장한다.
- 위의 방식과 동일하게 문자로 된 JSON 데이터를 Jackson 라이브러리인 `objectMapper`를 사용해서 자바 객체로 변환한다.
    
    → `@ModelAttribute`처럼 한번에 객체로 변환할 수 없을까?
    

```java
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody MyData data) {
			log.info("username={}, age={}", data.getUsername(), data.getAge());
			return "ok";
}
```

- `@RequestBody`에 직접 만든 객체를 지정할 수 있다.
- `HttpEntity` , `@RequestBody` 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.
- `@RequestBody`은 생략이 불가능하다.
    - 스프링은 `@ModelAttribute` , `@RequestParam` 해당 생략시 다음과 같은 규칙을 적용한다.
        - String , int , Integer 같은 단순 타입은 `@RequestParam`
        - 나머지는 `@ModelAttribute` (argument resolver 로 지정해둔 타입 제외)
        - 따라서 이 경우 MyData에 `@RequestBody`를 생략하면 `@ModelAttribute` 가 적용된다.
    - 따라서 생략하면 HTTP 메시지 바디가 아니라 요청 파리미터를 처리하게 된다.
    

```java
@ResponseBody
@PostMapping("/request-body-json-v4")
public String requestBodyJsonV4(HttpEntity<MyData> httpEntity) {
			MyData data = httpEntity.getBody();
			log.info("username={}, age={}", data.getUsername(), data.getAge());
			return "ok";
}
```

- `HttpEntity`를 사용해도 된다.
- 스프링 MVC 내부에서 HTTP 메시지 바디를 읽어서 HTTP 메시지 컨버터(HttpMessageConverter)로 문자나 객체로 변환한다.

```java
@ResponseBody
@PostMapping("/request-body-json-v5")
public MyData requestBodyJsonV5(@RequestBody MyData data) {
			log.info("username={}, age={}", data.getUsername(), data.getAge());
			return data;
}
```

- `@ResponseBody` : HTTP 메시지 바디에 객체를 직접 넣어줄 수 있다.
    - 물론 이 경우에도 `HttpEntity`를 사용해도 된다.

**HTTP 메시지 컨버터**는 문자 뿐만 아니라 JSON도 객체로 변환한다.

- `@RequestBody` 요청
    - JSON 요청 → HTTP 메시지 컨버터 → 객체
- `@ResponseBody` 요청
    - 객체 → HTTP 메시지 컨버터 → JSON 응답

**HTTP 응답 - 정적 리소스, 뷰 템플릿**

서버(스프링)에서 응답 데이터를 만드는 방법은 크게 3가지이다.

- 정적 리소스
    
    정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것이다.
    
    예) 웹 브라우저에 정적인 HTML, CSS, JS을 제공할 때는, 정적 리소스를 사용한다.
    
    *src/main/resources* 는 리소스를 보관하는 곳이며 classpath의 시작 경로이다.
    
    Spring Boot는 classpath의 다음 디렉토리에 있는 정적 리소스를 제공한다.
    
    - /static , /public , /resources , /META-INF/resources
    - 따라서 다음 디렉토리에 리소스를 넣어두면 Spring Boot가 정적 리소스로 서비스를 제공한다.
    - 예) 해당 경로에 파일이 들어있으면
        
        *src/main/resources/static/basic/hello-form.html*
        
        웹 브라우저에서 다음과 같이 실행하면 된다. 
        
        [*http://localhost:8080/basic/hello-form.html*](http://localhost:8080/basic/hello-form.html)
        
- 뷰 템플릿 사용
    
    뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.
    
    일반적으로 HTML을 동적으로 생성하는 용도로 사용하지만, 다른 것들도 가능하다. 
    
    뷰 템플릿이 만들 수 있는 것이라면 뭐든지 가능하다.
    
    Spring Boot는 기본 뷰 템플릿 경로(***src/main/resources/templates***)를 제공한다.

- *Spring Boot* **Thymeleaf** *설정*
    
    해당 라이브러리를 추가하면 (이미 추가되어 있다.)
    
    ```java
    `implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'`
    ```
    
    Spring Boot가 자동으로 **ThymeleafViewResolver** 와 필요한 스프링 빈들을 등록한다. 
    
    그리고 다음 설정은 기본 값이기에 변경이 필요할 때만 설정하면 된다.
    
    ```java
    spring.thymeleaf.prefix=classpath:/templates/
    spring.thymeleaf.suffix=.html
    ```
    

뷰 템플릿 생성 : *src/main/resources/templates/response/hello.html*

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
 <meta charset="UTF-8">
 <title>Title</title>
</head>
<body>
<p th:text="${data}">empty</p>
</body>
</html>
```

뷰 템플릿 호출 컨트롤러

```java
package hello.springmvc.basic.response;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class ResponseViewController {

	@RequestMapping("/response-view-v1")
	public ModelAndView responseViewV1() {
			ModelAndView mav = new ModelAndView("response/hello").addObject("data", "hello!");
			return mav;
	}

	// 1️⃣
	@RequestMapping("/response-view-v2")
	public String responseViewV2(Model model) {
			model.addAttribute("data", "hello!!");
			return "response/hello";
	}
	
	// 2️⃣
	@RequestMapping("/response/hello")
	public void responseViewV3(Model model) {
			model.addAttribute("data", "hello!!");
	}
}
```

1️⃣ String을 반환하는 경우 - View or HTTP 메시지

- `@ResponseBody`가 없으면 *response/hello*로 뷰 리졸버가 실행되어 뷰를 찾고(*response/hello.html*) 렌더링한다.
- `@ResponseBody`가 있으면 뷰 리졸버를 실행하지 않고 HTTP 메시지 바디에 직접 “*response/hello*” 라는 문자가 입력된다.

2️⃣ Void를 반환하는 경우

- **참고로 이 방식은 권장하지 않는다.**
- `@Controller` 를 사용하고, `HttpServletResponse` , `OutputStream`(Writer) 같은 HTTP 메시지
바디를 처리하는 파라미터가 없으면, 요청 URL을 참고해서 논리 뷰 이름으로 사용
    - 요청 URL : */response/hello*
    - 실행 : *templates/response/hello.html*

**HTTP API, 메시지 바디에 직접 입력**

HTTP API를 제공하는 경우에는 HTML이 아니라 **데이터를 전달**해야 하므로, HTTP 메시지 바디에 JSON 같은 형식으로 데이터를 실어 보낸다

> HTML이나 뷰 템플릿을 사용해도 HTTP 응답 메시지 바디에 HTML 데이터가 담겨서 전달된다. 여기서 설명하는 내용은 정적 리소스나 뷰 템플릿을 거치지 않고, **직접 HTTP 응답 메시지를 전달하는 경우**를 말한다.

- `@ResponseBody` , `HttpEntity` 를 사용하면, 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 응답 데이터를 출력할 수 있다.

```java
package hello.springmvc.basic.response;
import hello.springmvc.basic.HelloData;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@Slf4j
@Controller
//@RestController
public class ResponseBodyController {

	 @GetMapping("/response-body-string-v1")
	 public void responseBodyV1(HttpServletResponse response) throws IOException {
			 response.getWriter().write("ok");
	 }

	 /**
	 * HttpEntity, ResponseEntity(Http Status 추가)
	 * @return
	 */
	 @GetMapping("/response-body-string-v2")
	 public ResponseEntity<String> responseBodyV2() {
			 return new ResponseEntity<>("ok", HttpStatus.OK); // 200
	 }

	 @ResponseBody
	 @GetMapping("/response-body-string-v3")
	 public String responseBodyV3() {
			 return "ok";
	 }

	 @GetMapping("/response-body-json-v1")
	 public ResponseEntity<HelloData> responseBodyJsonV1() {
			 HelloData helloData = new HelloData();
			 helloData.setUsername("userA");
			 helloData.setAge(20);
			 return new ResponseEntity<>(helloData, HttpStatus.OK);
	 }

	 @ResponseStatus(HttpStatus.OK)
	 @ResponseBody
	 @GetMapping("/response-body-json-v2")
	 public HelloData responseBodyJsonV2() {
			 HelloData helloData = new HelloData();
			 helloData.setUsername("userA");
			 helloData.setAge(20);
			 return helloData;
	 }
}
```

- **responseBodyV1**
서블릿을 직접 다룰 때 처럼 `HttpServletResponse` 객체를 통해서 HTTP 메시지 바디에 직접 ok 응답 메시지를 전달한다. `response.getWriter().write("ok")`

- **responseBodyV2**
`ResponseEntity` 엔티티는 `**HttpEntity` 를 상속** 받았는데, `HttpEntity`는 **HTTP 메시지의 헤더, 바디 정보**를 가지고 있다. `ResponseEntity` 는 여기에 더해서 **HTTP 응답 코드를 설정**할 수 있다.
HttpStatus.CREATED 로 변경하면 201 응답이 나가는 것을 확인할 수 있다.

- **responseBodyV3**
`@ResponseBody` 를 사용하면 view를 사용하지 않고, HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접 입력할 수 있다. `ResponseEntity` 도 동일한 방식으로 동작한다.
- **responseBodyJsonV1**
`ResponseEntity` 를 반환한다. HTTP 메시지 컨버터를 통해 객체가 JSON 형식으로 변환되어 반환된다.
- **responseBodyJsonV2**
    
    `@ResponseBody` 를 사용하여 HTTP 메시지 컨버터를 통해서 객체가 JSON 형식으로 변환되어 HTTP 메시지 바디에 직접 입력할 수 있다.
    `ResponseEntity` 는 HTTP 응답 코드를 설정할 수 있는데, `@ResponseBody` 를 사용하면 이런 것을 설정하기 까다롭다.
    `@ResponseStatus(HttpStatus.OK)` 어노테이션을 사용하면 응답 코드도 설정할 수 있다.
    물론 어노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다. 프로그램 조건에 따라서 **동적으로 변경하려면** `ResponseEntity` 를 사용하면 된다.
    

**@RestController**

`@Controller` 대신에 `@RestController` 어노테이션을 사용하면, **해당 컨트롤러에 모두
`@ResponseBody` 가 적용되는 효과**가 있다. 따라서 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 데이터를 입력한다. 이름 그대로 **Rest API(HTTP API)를 만들 때 사용하는 컨트롤러**이다.
참고로 `@ResponseBody` 는 클래스 레벨에 두면 전체에 메서드에 적용되는데, `@RestController`
에노테이션 안에 `@ResponseBody` 가 적용되어 있다

### **HTTP 메시지 컨버터**

뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 HTTP 메시지
바디에서 직접 읽거나 쓰는 경우 HTTP 메시지 컨버터를 사용하면 편리하다.
