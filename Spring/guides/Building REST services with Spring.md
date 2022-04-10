# Building REST services with Spring    

##### [원문](https://spring.io/guides/tutorials/rest/)


### REST
REST는 architecture, benefit 및 기타 모든 것들을 포함한 웹의 원칙을 수용            
            
웹과 그 핵심 프로토콜인 HTTP는 다음과 같은 기능 스택 제공
+ 적절한 조치 (GET, POST, PUT, DELETE, …)
+ caching
+ Redirection과 forwarding (전달)
+ 보안 (암호화 및 인증)    


이러한 것들은 복원력있는 서비스를 구축하는데 중요한 요소        

HTTP 기반으로 구축한 REST API는 다음을 구축할 수 있는 수단 제공          
+ 이전 버전과 호환 가능한 API

+ 진화 가능한 API

+ 확장 가능한 서비스

+ 보안 가능한 서비스

+ stateless부터 stateful한 services까지의 스펙트럼

REST는 유비쿼터스에 있더라도 그 자체로 표준이 아니라 web-scale 시스템을 구축하는데 도움이 될 수 있는 아키텍처의 접근 방식, 스타일, 제약 조건의 집합       

### What You Will Build
Spring portfolio를 사용하여 REST의 stackless 기능을 활용한 RESTful 서비스를 구축

- 회사의 직원들을 관리하는 간단한 payroll 서비스 
- 직원 객체들을 H2 DB에 저장하고 JPA를 통해 접근
- Spring MVC layer로 불리는 인터넷을 통한 접근을 허용하는 것으로 래핑함


### Getting Started
Starting with Spring Initializr
1. https://start.spring.io
2. Gradle and java , but 이 튜토리얼에서는 maven 기반
3. Dependencies : Web, JPA, H2
4. Name : Payroll , Generate and download zip file

### REST 개념을 생략한 아주 간단한 payroll 서비스           

```java
package payroll;

import java.util.Objects;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
 
@Entity
class Employee { // 도메인 객체

  private @Id @GeneratedValue Long id;
  private String name;
  private String role;

  Employee() {}

  // Id 값 아직 없을 때 생성
  Employee(String name, String role) {

    this.name = name;
    this.role = role;
  }

  public Long getId() {
    return this.id;
  }

  public String getName() {
    return this.name;
  }

  public String getRole() {
    return this.role;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public void setName(String name) {
    this.name = name;
  }

  public void setRole(String role) {
    this.role = role;
  }

  @Override
  public boolean equals(Object o) {

    if (this == o)
      return true;
    if (!(o instanceof Employee))
      return false;
    Employee employee = (Employee) o;
    return Objects.equals(this.id, employee.id) && Objects.equals(this.name, employee.name)
        && Objects.equals(this.role, employee.role);
  }

  @Override
  public int hashCode() {
    return Objects.hash(this.id, this.name, this.role);
  }

  @Override
  public String toString() {
    return "Employee{" + "id=" + this.id + ", name='" + this.name + '\'' + ", role='" + this.role + '\'' + '}';
  }
}
```
+ @Entity : JPA 어노테이션으로 해당 객체를 JPA 기반 데이터 저장소에 저장할 준비를 마침
+ @Id : 기본키(primary key)를 의미하며 JPA provider에 의해 자동으로 채워짐
+ @GeneratedValue : @Id와 함께 사용되며 기본키 값의 생성 전략 제공                 


이와 같은 도메인 객체 정의를 통해 *Spring Data JPA* 로 전환하여 데이터베이스 상호작용 처리          

> Spring Data JPA repository는 백엔드 데이터 저장소에 대한 레코드 생성, 읽기, 수정, 삭제 메소드를 지닌 인터페이스             
> 일부 repository는 적절한 경우 데이터 페이징(paging) 및 정렬 제공              
> Spring Data는 인터페이스의 메소드 이름에서 발견되는 규칙을 기반으로 구현         
>                  
> *JPA 외에도 다양한 repository 구현 존재              
> Spring Data MongoDB, Spring Data GemFire, Spring Data Cassandra 등을 사용할 수 있지만 이 튜토리얼에선 JPA 사용*

Spring은 Data 접근에 용이한데 EmployeeRepository 인터페이스를 선언하면 자동으로 아래의 기능들을 사용할 수 있음.             
+ 새로운 Employee 생성
+ 기존의 Employee 수정
+ Employee 삭제
+ Employee 검색 (하나, 전부, 또는 단순하거나 복잡한 속성으로 검색)

이를 위해 도메인 타입이 Employee이고 id 타입이 Long인 Spring Data JPA의 JpaRepository 인터페이스를 상속받은 (extends) 인터페이스를 선언           

```java
package payroll;

import org.springframework.data.jpa.repository.JpaRepository;

interface EmployeeRepository extends JpaRepository<Employee, Long> {

}
```            
           
Spring Boot application는 최소한 public static void main entry-point와 @SpringBootApplication 어노테이션 지님               
```java
package payroll;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class PayrollApplication {

  public static void main(String... args) {
    SpringApplication.run(PayrollApplication.class, args);
  }
}
```
+ @SpringBootApplication는 meta-annotation으로 component scanning, autoconfiguration, property support를 가능하게 함
+ Servlet container를 실행하고 서비스 제공                   

테스트를 위해 데이터를 미리 로드, 해당 클래스는 스프링에 의해 자동으로 로드됨
```java
package payroll;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
class LoadDatabase {

  private static final Logger log = LoggerFactory.getLogger(LoadDatabase.class);

  @Bean
  CommandLineRunner initDatabase(EmployeeRepository repository) {

    return args -> {
      log.info("Preloading " + repository.save(new Employee("Bilbo Baggins", "burglar")));
      log.info("Preloading " + repository.save(new Employee("Frodo Baggins", "thief")));
    };
  }
}
```
클래스 로드되면
+ Spring Boot는 application context가 로드되면 모든 CommandLineRunner 빈들을 실행
+ 이러한 runner는 방금 생성한 EmployeeRepository 복사본 요청
+ 그것을 사용하여 두개의 엔티티를 생성하고 저장

PayRollApplication을 마우스 오른쪽 버튼으로 클릭하고 실행하면 콘솔 창에서 사전 로드 데이터 확인 가능 (전체 로그가 아닌 핵심 비트)
            
### HTTP is the Platform
web layer로 repository를 감싸기 위해 Spring MVC로 변환해주어야 함
Spring Boot 덕분에 코딩할 인프라 거의 없음
대신 다음과 같은 작업에 집중할 수 있음
```java
package payroll;

import java.util.List;

import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

@RestController
class EmployeeController {

  private final EmployeeRepository repository;

  // 생성자에 의해 EmployeeRepository 주입
  EmployeeController(EmployeeRepository repository) {
    this.repository = repository;
  }


  // Aggregate root
  // tag::get-aggregate-root[]
  @GetMapping("/employees")
  List<Employee> all() {
    return repository.findAll();
  }
  // end::get-aggregate-root[]

  @PostMapping("/employees")
  Employee newEmployee(@RequestBody Employee newEmployee) {
    return repository.save(newEmployee);
  }

  // Single item
  
  @GetMapping("/employees/{id}")
  Employee one(@PathVariable Long id) {
    
    // employee 찾지 못할 경우 EmployeeNotFoundException 발생
    return repository.findById(id)
      .orElseThrow(() -> new EmployeeNotFoundException(id));
  }

  @PutMapping("/employees/{id}")
  Employee replaceEmployee(@RequestBody Employee newEmployee, @PathVariable Long id) {
    
    return repository.findById(id)
      .map(employee -> {
        employee.setName(newEmployee.getName());
        employee.setRole(newEmployee.getRole());
        return repository.save(employee);
      })
      .orElseGet(() -> {
        newEmployee.setId(id);
        return repository.save(newEmployee);
      });
  }

  @DeleteMapping("/employees/{id}")
  void deleteEmployee(@PathVariable Long id) {
    repository.deleteById(id);
  }
}
```
+ @RestController : 각각의 메소드에 의해 리턴되는 데이터는 template으로 렌더링되는 것이 아닌 response body로 직접 기록됨
+ 각각의 작업에 대한 경로 존재
            + @GetMapping : HTTP의 GET
            + @PostMapping : HTTP의 POST
            + @PutMapping : HTTP의 PUT
            + @DeleteMapping : HTTP의 DELETE
           
+ employee를 찾지 못할 경우 EmployeeNotFoundException 발생        
  ```java

            package payroll;

            class EmployeeNotFoundException extends RuntimeException {

              EmployeeNotFoundException(Long id) {
                super("Could not find employee " + id);
              }
            }
  ```
  + EmployeeNotFoundException 발생하면 아래의 추가 정보를 통해 Spring MVC configuration이 HTTP 404와 렌더링 : 

  ```java
  package payroll;

  import org.springframework.http.HttpStatus;
  import org.springframework.web.bind.annotation.ControllerAdvice;
  import org.springframework.web.bind.annotation.ExceptionHandler;
  import org.springframework.web.bind.annotation.ResponseBody;
  import org.springframework.web.bind.annotation.ResponseStatus;

  @ControllerAdvice
  class EmployeeNotFoundAdvice {

  @ResponseBody
  @ExceptionHandler(EmployeeNotFoundException.class)
  @ResponseStatus(HttpStatus.NOT_FOUND)
  String employeeNotFoundHandler(EmployeeNotFoundException ex) {
    return ex.getMessage();
   }
  }
  ```
  + advice의 body는 content를 생성 : 예외 메시지 제공
  + @ResponseBody : 해당 advice는 response body에 바로 렌더링
  + @ExceptionHandler : EmployeeNotFoundException이 발생한 경우에만 응답하도록 advice 구성
  + @ResponseStatus : HTTPStatus.NOT_FOUND는 응답 상태 HTTP 404를 의미
  
PayRollApplication의 public static void main에서 마우스 오른쪽 클릭한 후 실행

만약 존재하지 않는 직원을 검색한다면
```terminal
$ curl -v localhost:8080/employees/99
```
Custom message인 Could not find employee 99 와 HTTP 404 에러 확인 할 수 있음
```terminal
*   Trying ::1...
* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> GET /employees/99 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 404
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 26
< Date: Thu, 09 Aug 2018 18:00:56 GMT
<
* Connection #0 to host localhost left intact
Could not find employee 99
```

> 만약 Windows 명령 프롬프트를 사용하여 cURL 명령을 실행하는 경우 아래의 명령문이 제대로 작동하지 않을 수 있음 
> 작은 따옴표로 묶인 arguments를 지원하는 터미널을 선택하거나 큰 따옴표를 사용한 뒤 JSON 내부에서 escape 처리

새로운 Employee record 생성         
```terminal
$ curl -X POST localhost:8080/employees -H 'Content-type:application/json' -d '{"name": "Samwise Gamgee", "role": "gardener"}'
```
새로 생성된 employee를 저장하고 우리에게 다시 보냄
```terminal
{"id":3,"name":"Samwise Gamgee","role":"gardener"}
```

기존의 employee를 수정
```terminal
$ curl -X PUT localhost:8080/employees/3 -H 'Content-type:application/json' -d '{"name": "Samwise Gamgee", "role": "ring bearer"}'
```
변경 사항이 반영된 것을 확인
```terminal
{"id":3,"name":"Samwise Gamgee","role":"ring bearer"}
```
        
> 서비스를 구성하는 방식은 상당한 영향을 미칠 수 있음            
> 이 상황에서 우리는 update(수정)라고 했지만 (replace)교체가 더 나은 설명             
> 예를 들어 이름이 제공되지 않으면 null이 대신됨

employee를 삭제              
```terminal
$ curl -X DELETE localhost:8080/employees/3

# Now if we look again, it's gone
$ curl localhost:8080/employees/3
Could not find employee 3
```
            
잘 작동되지만 이는 RESTful service라고 할 수 없음             

### What makes something RESTful?
지금까지 employee 데이터와 관련된 핵심 작업을 처리하는 web-based service를 다루었는데 이는 RESTful이라고 하기 충분하지 않음
+ /employees/3 와 같은 URL는 REST라고 할 수 없음
+ 단순히 GET, POST 등을 사용하는 것은 REST라고 할 수 없음
+ 모든 CRUD 작업을 배치하는 것은 REST라고 할 수 없음

사실 지금까지 우리가 빌드한 것은 RPC(Remote Procedure Call)라고 서술하는 것이 나음               
이 서비스와 상호 작용하는 방법을 알 수 있는 방법이 없기 때문                   
Roy Fielding의 이 진술은 REST와 RPC의 차이점에 대한 단서를 추가로 제공         
> 나는 HTTP 기반 인터페이스를 REST API로 호출하는 사람들의 수에 좌절감을 느끼고 있다. 
> 오늘날의 예는 SocialSite REST API다. 그것은 RPC이다. RPC라고 외친다. 
>
> hypertext가 제약 조건이라는 개념에서 REST 아키텍처 스타일을 명확히 하려면 무엇을 해야할까?
> 다시 말해, 만약 application 상태 엔진(API)이 hypertext에 의해 구동되지 않는다면 이는 RESTful하지 않으며 REST API라고 할 수 없다. 
> - Roy Fielding
> - https://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven

hypermedia를 포함하지 않는 표현의 부작용은 클라이언트가 API를 탐색하기 위해 URI를 하드 코딩해야한다는 것          
이는 웹에서 전자 상거래가 부상하기 이전과 동일한 취약점을 초래하기에 JSON 출력에 약간의 도움이 필요하다는 신호   
                     
hypermedia 기반 출력을 돕는 것을 목표로 하는 Spring HATEOAS를 소개한다.              
이전의 서비스를 RESTful하게 업그레이드 하려면 pom.xml의 dependencies에 Spring HATEOAS 추가 
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```
이 작은 라이브러리는 RESTful service를 정의하고 클라이언트가 사용하기 적합한 형식으로 렌더링하는 구성 제공          
모든 RESTful service의 중요한 요소는 관련 작업의 링크를 추가하는 것이다. controller를 더욱더 RESTful하게 만들려면 다음과 같이 링크를 추가 :        
```java
@GetMapping("/employees/{id}")
EntityModel<Employee> one(@PathVariable Long id) {

  Employee employee = repository.findById(id) //
      .orElseThrow(() -> new EmployeeNotFoundException(id));

  return EntityModel.of(employee, //
      linkTo(methodOn(EmployeeController.class).one(id)).withSelfRel(),
      linkTo(methodOn(EmployeeController.class).all()).withRel("employees"));
}
```
> 이 튜토리얼은 Spring MVC를 기반으로 하며 WebMvcLinkBuilder의 정적 helper method를 사용하여 이러한 링크를 빌드

이전의 것과 유사하지만 몇 가지 변경됨 :            
+ 메소드의 반환 타입이 Employee에서 EntityModel<Employee>로 변경. EntityModel<T>은 데이터뿐만 아니라 링크 모음을 포함하는 Spring HATEOAS의 제네릭 컨테이너
+ linkTo(methodOn(EmployeeController.class).one(id)).withSelfRel()는 Spring HATEOAS가 EmployeeController의 one() 메소드에 대한 링크를 빌드하고 자체 링크로 플래그하도록 요청
+ linkTo(methodOn(EmployeeController.class).all()).withRel("employees")는 Spring HATEOAS가 집계 루트 all()에 대한 링크를 빌드하고 "employee"라고 부르도록 요청
	
"링크를 빌드하는 것(build a link)"은 무엇을 의미할까? Spring HATEOAS의 주요 타입 중 하나는 Link이다.           
이것은 URI와 rel(Relation)을 포함한다. 링크는 웹에 힘을 실어주는데 World Wide Web 이전에는 다른 정보 시스템이 정보 또는 링크를 렌더링했지만 웹을 하나로 묶은 것은 이러한 종류의 관계 메타데이터가 있는 문서의 연결이었다.     
	     
Roy Fielding은 웹을 성공으로 이끈 동일한 기술로 API를 구축할 것을 권장하며 링크가 그 중 하나이다.         
application을 재시작하고 Bilbo employee record를 검색하면 이전과는 약간 다른 응답을 받을 것이다.
              
RESTful한 employee 표현
```json
{
  "id": 1,
  "name": "Bilbo Baggins",
  "role": "burglar",
  "_links": {
    "self": {
      "href": "http://localhost:8080/employees/1"
    },
    "employees": {
      "href": "http://localhost:8080/employees"
    }
  }
}	
```
압축해제된 출력은 이전에 본 데이터 요소(id, name, role)뿐만 아니라 두개의 URI를 포함하는 _link 항목도 보여줌.     
이 전체 문서는 *HAL* 을 통해 형식이 지정됨
	
HAL은 데이터뿐만 아니라 hypermedia 컨트롤도 인코딩할 수 있는 경량 미디어타입으로 소비자가 탐색할 수 있는 API의 다른 부분에 대해 경고한다.
	
아 경우에선 집계 루트에 대한 링크와 "self" 링크 (코드의 this문과 같은 종류)가 있다.          
집계 루트 또한 더 RESTful하게 만들려면 최상위 링크를 포함하는 동시에 RESTful 구성요소도 포함해야 한다.
	
그래서 우린 집계 루트 가져오기를
```java
@GetMapping("/employees")
List<Employee> all() {
  return repository.findAll();
}
```
집계 루트 리소스 가져오기로 바꾼다.
```java
@GetMapping("/employees")
CollectionModel<EntityModel<Employee>> all() {

  List<EntityModel<Employee>> employees = repository.findAll().stream()
      .map(employee -> EntityModel.of(employee,
          linkTo(methodOn(EmployeeController.class).one(employee.getId())).withSelfRel(),
          linkTo(methodOn(EmployeeController.class).all()).withRel("employees")))
      .collect(Collectors.toList());

  return CollectionModel.of(employees, linkTo(methodOn(EmployeeController.class).all()).withSelfRel());
}
```	
+ CollectionModel<> : 또다른 Spring HATEOAS 콘테이너로 이전의 EntityModel<>와 같은 단일 리소스 엔티티 대신 리소스 컬렉션을 캡슐화하는 것을 목표로 함. CollectionModel<>도 링크 포함할 수 있음.
	
