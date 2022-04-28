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
