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
이 튜토리얼은 Spring MVC를 기반으로 하며 WebMvcLinkBuilder의 static helper method를 사용하여 이러한 링크를 빌드한다. 만약 프로젝트에서 Spring WebFlux를 사용한다면 WebFluxLinkBuilder를 대신 사용해야 한다.

이것은 이전과 매우 유사하지만 몇 가지가 변경되었다.

- 메소드의 반환 타입이 `Employee`에서 `EntityModel<Employee>` 로 변경되었다. `EntityModel<T>` 는 Spring HATEOAS의 제네릭 컨테이너로 데이터뿐만 아니라 링크 모음을 포함한다.
- `linkTo(methodOn(EmployeeController.class).one(id)).withSelfRel()`는 Spring HATEOAS가 `EmployeeController`의 `one()` 메소드에 대한 링크를 빌드하고 자체 링크로 플래그를 지정하도록 요청한다.
- `linkTo(methodOn(EmployeeController.class).all()).withRel("employees")`는 Spring HATEOAS가 집계 루트 `all()`에 대한 링크를 빌드하고 이를 “employees”라고 부르도록 요청한다.
	
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
`CollectionModel`<>은 또다른 Spring HATEOAS Container로 이전의 `EntityModel<>`와 같은 단일 리소스 엔티티 대신 리소스 컬렉션을 캡슐화하는 것을 목표로 한다. `CollectionModel<>`도 링크를 포함할 수 있다.

<aside>
✋ “encapsulating collections”는 무슨 뜻일까? Collections of employees?

</aside>

우리가 REST를 이야기하고 있기 때문에 employee resource collections을 캡슐화(encapsulate) 해야 한다.

그렇기 때문에 모든 employee를 가져온 뒤, `List<EntityModel<Employee>>`로 변환시킨다. (Java 8 Stream)

application을 재시작하고 집계 루트를 가져오면 이와 같은 결과를 볼 수 있다.

```json
{
  "_embedded": {
    "employeeList": [
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
      },
      {
        "id": 2,
        "name": "Frodo Baggins",
        "role": "thief",
        "_links": {
          "self": {
            "href": "http://localhost:8080/employees/2"
          },
          "employees": {
            "href": "http://localhost:8080/employees"
          }
        }
      }
    ]
  },
  "_links": {
    "self": {
      "href": "http://localhost:8080/employees"
    }
  }
}
```

employee resource collections을 제공하는 집계 루트는 최상위 `“self”` 링크를 가진다. “collection”은 `"_embedded"` 섹션 밑에 나열되며 이것이 HAL이 collection을 표현하는 방식이다.

그리고 collection 각각의 멤버들은 관련된 링크뿐만 아니라 정보도 가지고 있다.

이러한 모든 링크들을 추가하는 것은 무슨 의미가 있을까? 이는 시간이 지남에 따라 REST service를 발전시킬 수 있다. 

기존의 링크는 유지관리를 할 수 있지만 향후 새로운 링크를 추가할 수 있다. 최신 클라이언트는 새로운 링크를 활용할 수 있지만 레거시 클라이언트는 이전 링크에서 유지할 수 있다.

이는 service가 재배치 및 이전되는 경우에 특히 유용하다. 링크 구조가 유지되는 한 클라이언트는 계속해서 검색과 상호작용을 할 수 있다. 

### ****Simplifying Link Creation****

이전의 코드에서 단일 employee 링크가 반복해서 생성되는 것을 알아차렸나?

employee에 단일 링크를 제공하고 집계 루트에 대한 “employees” 링크를 생성하는 코드가 두 번 나타났다. 

이것에 우려를 제기하였다면 해결책이 있다.

간단하게 `Employee`객체를 `EntityModel<Employee>`객체로 변환하는 함수를 정의해야 한다. 

메소드를 쉽게 코딩할 수 있지만 Spring HATEOAS의 `RepresentationModelAssembler`인터페이스를 구현하면 이점이 있다.

```java
package payroll;

import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.*;

import org.springframework.hateoas.EntityModel;
import org.springframework.hateoas.server.RepresentationModelAssembler;
import org.springframework.stereotype.Component;

@Component
class EmployeeModelAssembler implements RepresentationModelAssembler<Employee, EntityModel<Employee>> {

  @Override
  public EntityModel<Employee> toModel(Employee employee) {

    return EntityModel.of(employee, //
        linkTo(methodOn(EmployeeController.class).one(employee.getId())).withSelfRel(),
        linkTo(methodOn(EmployeeController.class).all()).withRel("employees"));
  }
}
```

이 간단한 인터페이스는 한가지 메소드를 지닌다 : `toModel()`

`toModel()`는 비모델객체(`Employee`)를 모델기반객체(`EntityModel<Employee>`)로 변환하는 것을 기반으로 한다.

Controller에서 이전에 보았던 모든 코드들은 이 클래스로 옮길 수 있다.

그리고 Spring Framework의 `@Component`어노테이션을 적용하면 app이 시작될 때 자동으로 assembler가 생성된다.

Spring HATEOAS의 모든 모델에 대한 추상화(abstract) 기본 클래스는 `RepresentationModel`이다. 하지만 단순성을 위해 모든 POJO를 모델로 쉽게 매핑하는 매커니즘으로 `EntityModel<T>`를 사용하는 것을 추천한다.

이 assembler를 활용하려면 생성자에 assembler를 주입하여 EmployeeController를 변경하면 된다. 

```java
@RestController
class EmployeeController {

  private final EmployeeRepository repository;

  private final EmployeeModelAssembler assembler;

  EmployeeController(EmployeeRepository repository, EmployeeModelAssembler assembler) {

    this.repository = repository;
    this.assembler = assembler;
  }

  ...

}
```

단일 항목 직원 메소드에서 assembler를 사용할 수 있다.

```java
@GetMapping("/employees/{id}")
EntityModel<Employee> one(@PathVariable Long id) {

  Employee employee = repository.findById(id) //
      .orElseThrow(() -> new EmployeeNotFoundException(id));

  return assembler.toModel(employee);

	/* 이전 코드
	return EntityModel.of(employee,
      linkTo(methodOn(EmployeeController.class).one(id)).withSelfRel(),
      linkTo(methodOn(EmployeeController.class).all()).withRel("employees"));
	*/
}
```

이 코드는 거의 동일하지만 `EntityModel<Employee>` 인스턴스를 생성하는 대신 assembler를 위임한다는 점은 다르다. 

```java
@GetMapping("/employees")
CollectionModel<EntityModel<Employee>> all() {

  List<EntityModel<Employee>> employees = repository.findAll().stream() //
      .map(assembler::toModel) 
			/* 이전 코드
			map(employee -> EntityModel.of(employee,
          linkTo(methodOn(EmployeeController.class).one(employee.getId())).withSelfRel(),
          linkTo(methodOn(EmployeeController.class).all()).withRel("employees")))
			*/
      .collect(Collectors.toList());

  return CollectionModel.of(employees, linkTo(methodOn(EmployeeController.class).all()).withSelfRel());
}
```

이 코드 또한 거의 비슷하지만 모든 `EntityModel<Employee>` 생성 로직을 `map(assembler::toModel)`로 대체한다. Java 8 메소드 참조 덕분에 플러그인을 연결하고 Controller를 단순화하는 것이 매우 쉬워졌다.

Spring HATEOAS의 주요 설계 목표는 **Right Thing™**을 더 쉽게 하는 것이다. 이 시나리오에서는 하드 코딩없이 service에 hypermedia를 추가한다.

이 단계에서 실제로 hypermedia 기반 콘텐츠를 생성하는 Spring MVC REST Controller를 생성하였다. HAL을 사용하지 않는 클라이언트는 순수한 데이터를 사용하는 동안 추가 비용을 무시할 수 있다. HAL을 사용하는 클라이언트는 권한 있는 API를 사용할 수 있다. 

그러나 Spring으로 진정한 RESTful service를 구축하는데 필요한 것은 이것만이 아니다.

### ****Evolving REST APIs****

하나의 추가 라이브러리와 몇 줄의 추가 코드로 application에 hypermedia를 추가하였다. 그러나 이것만으로 service를 RESTful하게 만들 수 없다. REST의 중요한 면은 기술 스택도 아니고 단일 표준도 아니다.

- REST는 application을 더욱더 탄력적으로 만드는 아키텍처 제약 조건의 모음이다. 복원력의 핵심 요소는 service를 업그레이드 할 때 클라이언트가 downtime(가동 중지 시간)을 겪지 않는 것이다.
    - 옛날에는 업그레이드는 클라이언트를 깨는 것으로 악명 높았다. 즉, 서버를 업데이트하려면 클라이언트를 업그레이드해야 한다는 것이다. 오늘날과 같은 시대에 업그레이드를 수행하는데 몇 시간 또는 몇 분의 downtime이 발생하면 이는 수백만 달러의 손실로 이어질 수 있다. 일부 회사는 관리자에게 downtime을  최소화할 계획을 제시하도록 한다. 과거에는 로드가 최소인 일요일 오전 2시에 업그레이드를 수행할 수 있었다. 그러나 다른 시간대의 국제 소비자와 거래하는 오늘날의 인터넷 기반 전자 상거래에서는 효과적인 전략이 아니다.
        
        

SOAP 기반 service와 CORBA 기반 서비스는 믿을 수 없을 정도로 취약했다. 기존, 신규 클라이언트를 모두 지원할 수 있는 서버를 출시하는 것이 어려웠다. REST 기반 사례를 사용하면, 특히 Spring 기술을 사용하면 훨씬 쉽다.

### ****Supporting changes to the API****

설계 문제를 상상해보자 : Employee 기반 레코드가 있는 시스템을 출시했는데 이 시스템이 큰 타격을 입었다. 시스템을 수많은 기업에 판매하였는데 갑자기 직원의 이름을 firstName과 lastName으로 분리해야하는 상황이 발생한다.

Employee 클래스를 열어 name 필드를 firstName과 lastName으로 바꾸기 전에 잠시 멈추고 생각해 보아라. 클라이언트는 손해를 입을까? 업그레이드하는데 시간이 얼마나 걸릴까? service에 접근하는 모든 클라이언트를 제어해야 하나?

REST 이전의 오래된 전략이 있다 : 데이터베이스에서 column을 절대 삭제하지 마라

데이터베이스 테이블에 항상 column(field)를 추가할 수 있다. 그러나 하나를 (name 필드)를 제거하지 마라. RESTful service의 원칙도 동일하다. 

JSON에 새로운 필드를 추가하되 제거하지 않는다. 

```json
{
  "id": 1,
  "firstName": "Bilbo",
  "lastName": "Baggins",
  "role": "burglar",
  "name": "Bilbo Baggins",
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

이 형식이 `firstName`, `lastName`, `name`을 어떻게 표시하는지 보았는가?

정보의 복제를 자랑하지만 목적은 기존 클라이언트와 신규 클라이언트를 모두 지원하는 것이다. 

즉, 클라이언트를 동시에 업그레이드 하지 않고 서버를 업그레이드 할 수 있다는 뜻이다.
그리고 이 정보를 이전 방식과 새로운 방식으로 모두 표시해야할 뿐만 아니라 들어오는 데이터를 양방향으로 처리해야 한다.

```java
package payroll;

import java.util.Objects;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
class Employee {

  private @Id @GeneratedValue Long id;
  private String firstName;
  private String lastName;
  private String role;

  Employee() {}

  Employee(String firstName, String lastName, String role) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.role = role;
  }

  public String getName() {
    return this.firstName + " " + this.lastName;
  }

  public void setName(String name) {
    String[] parts = name.split(" ");
    this.firstName = parts[0];
    this.lastName = parts[1];
  }

  public Long getId() {
    return this.id;
  }

  public String getFirstName() {
    return this.firstName;
  }

  public String getLastName() {
    return this.lastName;
  }

  public String getRole() {
    return this.role;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public void setFirstName(String firstName) {
    this.firstName = firstName;
  }

  public void setLastName(String lastName) {
    this.lastName = lastName;
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
    return Objects.equals(this.id, employee.id) && Objects.equals(this.firstName, employee.firstName)
        && Objects.equals(this.lastName, employee.lastName) && Objects.equals(this.role, employee.role);
  }

  @Override
  public int hashCode() {
    return Objects.hash(this.id, this.firstName, this.lastName, this.role);
  }

  @Override
  public String toString() {
    return "Employee{" + "id=" + this.id + ", firstName='" + this.firstName + '\'' + ", lastName='" + this.lastName
        + '\'' + ", role='" + this.role + '\'' + '}';
  }
}
```

해당 클래스는 `Employee` 이전 버전과 매우 유사하다. 변경사항을 살펴보자.

- `name` 필드가  `firstName`과 `lastName`으로 변경되었다.
- 이전 name 속성의 “가상” getter인 `getName()`이 정의된다. `firstName`과 `lastName` 필드를 사용하여 값을 생성한다.
- 이전 name 속성의 “가상” setter인 `setName()` 또한  정의된다. 들어오는 string(문자열)을 분석하여 적절한 필드에 저장한다.

새로운 생성자를 사용하기 위해 데이터베이스를 미리 로드(preload)하는 방법을 변경해야 하는 걸 잊지 마라.

```java
log.info("Preloading " + repository.save(new Employee("Bilbo", "Baggins", "burglar")));
log.info("Preloading " + repository.save(new Employee("Frodo", "Baggins", "thief")));
```

올바른 방향으로 나아가는 또다른 단계는 각 REST 메소드가 적절한 response를 반환하고 있는지 확인하는 것이다. POST 메소드를 수정해보자.

```java
@PostMapping("/employees")
ResponseEntity<?> newEmployee(@RequestBody Employee newEmployee) {

  EntityModel<Employee> entityModel = assembler.toModel(repository.save(newEmployee));

  return ResponseEntity //
      .created(entityModel.getRequiredLink(IanaLinkRelations.SELF).toUri()) //
      .body(entityModel);
}
```

- 새로운 `Employee` 객체는 이전과 같이 저장된다. 그러나 결과 객체는 `EmployeeModelAssembler`를 통해 래핑된다.
- Spring MVC의 `ResponseEntity`는 **HTTP 201 Created** 상태 메시지를 생성하는데 사용된다. 이 유형의 response는 일반적으로 Location response header가 포함되며 모델의 자체 관련 링크에서 파생된 URI를 사용한다.
- 또한 저장된 객체의 모델 기반 버전을 반환한다.

이러한 조정을 적용하면 동일한 endpoint을 사용하여 새로운 employee 리소스를 생성하고 레거시 name 필드를 사용할 수 있다. : 

```powershell
$ curl -v -X POST localhost:8080/employees -H 'Content-Type:application/json' -d '{"name": "Samwise Gamgee", "role": "gardener"}'
```

```powershell
> POST /employees HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> Content-Type:application/json
> Content-Length: 46
>
< Location: http://localhost:8080/employees/3
< Content-Type: application/hal+json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Fri, 10 Aug 2018 19:44:43 GMT
<
{
  "id": 3,
  "firstName": "Samwise",
  "lastName": "Gamgee",
  "role": "gardener",
  "name": "Samwise Gamgee",
  "_links": {
    "self": {
      "href": "http://localhost:8080/employees/3"
    },
    "employees": {
      "href": "http://localhost:8080/employees"
    }
  }
}
```

결과 객체가 HAL(`name`과 `firstName`,`lastName` 모두)로 렌더링될 뿐만 아니라 **Location** header가 *[http://localhost:8080/employees/3](http://localhost:8080/employees/3로)* 로 채워진다.

hypermedia 기반 클라이언트는 이 새로운 리소스를 “서핑”하고 상호작용을 계속하도록 선택할 수 있다. 

PUT 메소드도 비슷한 조정이 필요하다. :

```java
@PutMapping("/employees/{id}")
ResponseEntity<?> replaceEmployee(@RequestBody Employee newEmployee, @PathVariable Long id) {

  Employee updatedEmployee = repository.findById(id) //
      .map(employee -> {
        employee.setName(newEmployee.getName());
        employee.setRole(newEmployee.getRole());
        return repository.save(employee);
      }) //
      .orElseGet(() -> {
        newEmployee.setId(id);
        return repository.save(newEmployee);
      });

  EntityModel<Employee> entityModel = assembler.toModel(updatedEmployee);

  return ResponseEntity //
      .created(entityModel.getRequiredLink(IanaLinkRelations.SELF).toUri()) //
      .body(entityModel);
}
```

`save()` 작업에서 생성된 `Employee` 객체는 `EmployeeModelAssembler`에 의해 `EntityModel<Employee>`객체로 래핑된다. 

`getRequiredLink()` 메소드를 사용하여 SELF rel로 `EmployeeModelAssembler`에 의해 생성된 링크를 검색할 수 있다. 이 메소드는 `toUri()` 메소드를 통해 URI로 변환되어야 하는 링크를 반환한다.

HTTP 200 OK보다 더 자세한 HTTP response 코드를 원하기에 Spring MVC의 `ResponseEntity` wrapper를 사용할 것이다. 리소스의 URI를 연결할 수 있는 편리한 정적 메소드 `created()`가 있다. **HTTP 201 Created**가 새로운 리소스를 반드시 생성하는 것은 아니기 때문에 올바른 의미를 전달하는지 논쟁의 여부가 있다. 그러나 **Location** response header가 미리 로드되어 있으므로 함께 실행한다.

```powershell
$ curl -v -X PUT localhost:8080/employees/3 -H 'Content-Type:application/json' -d '{"name": "Samwise Gamgee", "role": "ring bearer"}'

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> PUT /employees/3 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
> Content-Type:application/json
> Content-Length: 49
>
< HTTP/1.1 201
< Location: http://localhost:8080/employees/3
< Content-Type: application/hal+json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Fri, 10 Aug 2018 19:52:56 GMT
{
	"id": 3,
	"firstName": "Samwise",
	"lastName": "Gamgee",
	"role": "ring bearer",
	"name": "Samwise Gamgee",
	"_links": {
		"self": {
			"href": "http://localhost:8080/employees/3"
		},
		"employees": {
			"href": "http://localhost:8080/employees"
		}
	}
}
```

마지막으로 DELETE 메소드를 적절하게 수정한다.

```java
@DeleteMapping("/employees/{id}")
ResponseEntity<?> deleteEmployee(@PathVariable Long id) {

  repository.deleteById(id);

  return ResponseEntity.noContent().build();
}
```

```powershell
$ curl -v -X DELETE localhost:8080/employees/1

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> DELETE /employees/1 HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 204
< Date: Fri, 10 Aug 2018 21:30:26 GMT
```

Employee 클래스의 필드를 변경하려면 기존의 콘텐츠를 새로운 column로 적절하게 마이그레이션 할 수 있도록 데이터베이스팀과 조정이 필요하다.

이제 신규 클라이언트가 향상된 기능을 활용할 동안 기존 클라이언트를 방해하지 않는 업그레이드를 할 준비가 되었다. 

### ****Building links into your REST API****

지금까지 bare bone link로 진화 가능한 API를 구축했다. API를 성장시키고 클라이언트에게 더 나은 service를 제공하려면 Hypermedia의 개념을 application 상태의 엔진으로 받아들여야 한다.

비즈니스 로직은 필연적으로 프로세스와 관련된 규칙을 구축한다. 이러한 시스템의 위험은 종종 서버측의 로직을 클라이언트에게 전달하고 강력한 결합을 구축한다는 것이다. REST는 이러한 연결, 결합을 끊고 이를 최소화한다.

클라이언트의 변경사항을 유발하지 않고 상태 변경에 대처하는 방법을 보여주기 위해 주문을 이행하는 시스템을 추가하는 것을 상상해보자.

첫번째로 Order 레코드를 정의하자.

```java
package payroll;

import java.util.Objects;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.Table;

@Entity
@Table(name = "CUSTOMER_ORDER")
class Order {

  private @Id @GeneratedValue Long id;

  private String description;
  private Status status;

  Order() {}

  Order(String description, Status status) {

    this.description = description;
    this.status = status;
  }

  public Long getId() {
    return this.id;
  }

  public String getDescription() {
    return this.description;
  }

  public Status getStatus() {
    return this.status;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public void setDescription(String description) {
    this.description = description;
  }

  public void setStatus(Status status) {
    this.status = status;
  }

  @Override
  public boolean equals(Object o) {

    if (this == o)
      return true;
    if (!(o instanceof Order))
      return false;
    Order order = (Order) o;
    return Objects.equals(this.id, order.id) && Objects.equals(this.description, order.description)
        && this.status == order.status;
  }

  @Override
  public int hashCode() {
    return Objects.hash(this.id, this.description, this.status);
  }

  @Override
  public String toString() {
    return "Order{" + "id=" + this.id + ", description='" + this.description + '\'' + ", status=" + this.status + '}';
  }
}
```

- ORDER는 테이블에 유효한 이름이 아니기 때문에 클래스는 테이블의 이름을 변경하는 JPA의 `@Table` 어노테이션이 필요하다.
- `description`필드와 `status`가 포함된다.

주문(Order)은 고객이 주문을 제출하고 이행되거나 취소된 시점부터 일련의 상태 변화를 거쳐야 한다. 이것은 Java의 `enum`으로 잡을 수 있다.

```java
package payroll;

enum Status {

  IN_PROGRESS, //
  COMPLETED, //
  CANCELLED
}
```

이 `enum`은 주문(Order)이 가질 수 있는 다양한 상태를 잡는다.

주문(Order)과 데이터베이스의 상호작용을 지원하기 위해서 해당하는 Spring Data repository를 정의해야 한다.

```java
interface OrderRepository extends JpaRepository<Order, Long> {
}
```

이제 `OrderController`를 정의할 수 있다.

```java
@RestController
class OrderController {

  private final OrderRepository orderRepository;
  private final OrderModelAssembler assembler;

  OrderController(OrderRepository orderRepository, OrderModelAssembler assembler) {

    this.orderRepository = orderRepository;
    this.assembler = assembler;
  }

  @GetMapping("/orders")
  CollectionModel<EntityModel<Order>> all() {

    List<EntityModel<Order>> orders = orderRepository.findAll().stream() //
        .map(assembler::toModel) //
        .collect(Collectors.toList());

    return CollectionModel.of(orders, //
        linkTo(methodOn(OrderController.class).all()).withSelfRel());
  }

  @GetMapping("/orders/{id}")
  EntityModel<Order> one(@PathVariable Long id) {

    Order order = orderRepository.findById(id) //
        .orElseThrow(() -> new OrderNotFoundException(id));

    return assembler.toModel(order);
  }

  @PostMapping("/orders")
  ResponseEntity<EntityModel<Order>> newOrder(@RequestBody Order order) {

    order.setStatus(Status.IN_PROGRESS);
    Order newOrder = orderRepository.save(order);

    return ResponseEntity //
        .created(linkTo(methodOn(OrderController.class).one(newOrder.getId())).toUri()) //
        .body(assembler.toModel(newOrder));
  }
}
```

- 지금까지 구축한 Controller와 동일한 REST Controller 설정이 포함되어 있다.
- `OrderRepository`와 `OrderModelAssembler`(아직 구현되지 않은)를 주입한다.
- 처음 두 개의 Spring MVC 경로는 집계 루트(`all()`)와 단일 항목(`one()`) 주문 리소스를 처리한다.
- 세번째 Spring MVC 경로는 IN_PROGRESS 상태에서 시작해서 새 주문을 처리한다.
- 모든 Controller 메소드는 Hypermedia(또는 그러한 유형의 wrapper)를 적절하게 렌더링하기 위해 Spring HATEOAS의 **`RepresentationModel`** 서브 클래스 중 하나를 반환한다.

`OrderModelAssembler`를 구축하기 전에 어떤 일이 일어나야 하는지 논의해보자.

Status.IN_PROGRESS, Status.COMPLETED, 그리고 Status.CANCELLED 사이의 상태 흐름을 모델링하고 있다. 이러한 데이터를 클라이언트에게 제공할 때 자연스러운 것은 클라이언트가 payload를 기반으로 할 수 있는 작업에 대한 결정을 내리도록 하는 것이다.

그러나 그것은 잘못된 것이다.

이 흐름에서 새로운 상태를 도입하면 어떻게 되는가? UI의 다양한 버튼 배치는 아마도 잘못되었을 것이다. 국제적 지원을 코딩하고 각 상태에 대한 로케일별 텍스트를 표시하는 동안 각 상태의 이름을 변경하면 어떻게 될까? 그러면 대부분의 클라이언트가 손상될 가능성이 크다. 

**application 상태의 엔진으로** **HATEOAS** 또는 **Hypermedia**를 입력한다. 클라이언트가 payload 구문 분석을 하는 대신 유효한 작업을 알리는 링크를 제공한다.

payload의 데이터에서 상태 기반 작업을 분리한다. 즉, CANCEL 및 COMPLETE가 유효한 작업인 경우, 링크 목록에 동적으로 추가한다. 클라이언트는 링크가 존재할 때 사용자에게 해당 버튼을 표시하기만 하면 된다. 이렇게 하면 클라이언트가 이러한 작업이 유효한 때를 알 필요가 없어져 서버와 클라이언트가 상태 전환 논리에서 동기화 되지 않을 위험이 줄어든다.

Spring HATEOAS `RepresentationModelAssembler` 컴포넌트의 개념을 이미 수용했으므로 이러한 로직을 `OrderModelAssembler`에 넣는 것은 이 비즈니스 규칙을 잡는데 완벽한 판이 될 것이다.

```java
package payroll;

import static org.springframework.hateoas.server.mvc.WebMvcLinkBuilder.*;

import org.springframework.hateoas.EntityModel;
import org.springframework.hateoas.server.RepresentationModelAssembler;
import org.springframework.stereotype.Component;

@Component
class OrderModelAssembler implements RepresentationModelAssembler<Order, EntityModel<Order>> {

  @Override
  public EntityModel<Order> toModel(Order order) {

    // Unconditional links to single-item resource and aggregate root

    EntityModel<Order> orderModel = EntityModel.of(order,
        linkTo(methodOn(OrderController.class).one(order.getId())).withSelfRel(),
        linkTo(methodOn(OrderController.class).all()).withRel("orders"));

    // Conditional links based on state of the order

    if (order.getStatus() == Status.IN_PROGRESS) {
      orderModel.add(linkTo(methodOn(OrderController.class).cancel(order.getId())).withRel("cancel"));
      orderModel.add(linkTo(methodOn(OrderController.class).complete(order.getId())).withRel("complete"));
    }

    return orderModel;
  }
}
```

이 resource assembler는 항상 단일 항목 리소스에 대한 자체 링크와 집계 루트에 대한 링크를 포함한다. 그러나 이것은 OrderController.cancel(id) 및 아직 정의되지 않은 OrderController.complete(id)에 대한 두 개의 조건 링크부 또한 포함하고 있다. 이 링크는 주문의 상태가 **Status.IN_PROGRESS**인 경우에만 표시된다.

클라이언트가 단순히 기존 JSON 데이터를 읽는 대신 링크를 읽는 기능과 HAL를 채택할 수 있다면, 주문 시스템에 대한 도메인 지식의 필요성을 교환할 수 있다. 이것은 자연스럽게 클라이언트와 서버 간의 결합을 감소시킨다. 그리고 프로세스에서 클라이언트를 중단시키지 않고 주문 이행의 흐름을 조정할 수 있다.

주문 이행을 마무리하려면 취소 작업을 위해 `OrderController`에 다음을 추가한다. :

```java
@DeleteMapping("/orders/{id}/cancel")
ResponseEntity<?> cancel(@PathVariable Long id) {

  Order order = orderRepository.findById(id) //
      .orElseThrow(() -> new OrderNotFoundException(id));

  if (order.getStatus() == Status.IN_PROGRESS) {
    order.setStatus(Status.CANCELLED);
    return ResponseEntity.ok(assembler.toModel(orderRepository.save(order)));
  }

  return ResponseEntity //
      .status(HttpStatus.METHOD_NOT_ALLOWED) //
      .header(HttpHeaders.CONTENT_TYPE, MediaTypes.HTTP_PROBLEM_DETAILS_JSON_VALUE) //
      .body(Problem.create() //
          .withTitle("Method not allowed") //
          .withDetail("You can't cancel an order that is in the " + order.getStatus() + " status"));
}
```

취소를 허용하기 전에 주문 상태를 확인한다. 만약 유효한 상태가 아니라면 hypermedia 지원 에러 컨테이너인 [RFC-7807](https://tools.ietf.org/html/rfc7807) 문제를 반환한다. 전환이 실제로 유효하다면 주문을 CANCELLED로 전환한다.

주문 완료를 위해 `OrderController`에 다음을 추가한다. :

```java
@PutMapping("/orders/{id}/complete")
ResponseEntity<?> complete(@PathVariable Long id) {

  Order order = orderRepository.findById(id) //
      .orElseThrow(() -> new OrderNotFoundException(id));

  if (order.getStatus() == Status.IN_PROGRESS) {
    order.setStatus(Status.COMPLETED);
    return ResponseEntity.ok(assembler.toModel(orderRepository.save(order)));
  }

  return ResponseEntity //
      .status(HttpStatus.METHOD_NOT_ALLOWED) //
      .header(HttpHeaders.CONTENT_TYPE, MediaTypes.HTTP_PROBLEM_DETAILS_JSON_VALUE) //
      .body(Problem.create() //
          .withTitle("Method not allowed") //
          .withDetail("You can't complete an order that is in the " + order.getStatus() + " status"));
}
```

이것은 적절한 상태가 아니면 주문 상태가 완료되지 않도록 유사한 로직을 구현한다.

이전에 로드했던 Employee와 함께 일부 Order를 미리 로드하도록 LoadDatabase 업데이트

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
  CommandLineRunner initDatabase(EmployeeRepository employeeRepository, OrderRepository orderRepository) {

    return args -> {
      employeeRepository.save(new Employee("Bilbo", "Baggins", "burglar"));
      employeeRepository.save(new Employee("Frodo", "Baggins", "thief"));

      employeeRepository.findAll().forEach(employee -> log.info("Preloaded " + employee));

      // update
      orderRepository.save(new Order("MacBook Pro", Status.COMPLETED));
      orderRepository.save(new Order("iPhone", Status.IN_PROGRESS));

      orderRepository.findAll().forEach(order -> {
        log.info("Preloaded " + order);
      });
      
    };
  }
}
```

새로운 order service를 사용하기 위해 몇 가지 작업만 수행하면 된다.

```powershell
$ curl -v http://localhost:8080/orders
```

```json
{
  "_embedded": {
    "orderList": [
      {
        "id": 3,
        "description": "MacBook Pro",
        "status": "COMPLETED",
        "_links": {
          "self": {
            "href": "http://localhost:8080/orders/3"
          },
          "orders": {
            "href": "http://localhost:8080/orders"
          }
        }
      },
      {
        "id": 4,
        "description": "iPhone",
        "status": "IN_PROGRESS",
        "_links": {
          "self": {
            "href": "http://localhost:8080/orders/4"
          },
          "orders": {
            "href": "http://localhost:8080/orders"
          },
          "cancel": {
            "href": "http://localhost:8080/orders/4/cancel"
          },
          "complete": {
            "href": "http://localhost:8080/orders/4/complete"
          }
        }
      }
    ]
  },
  "_links": {
    "self": {
      "href": "http://localhost:8080/orders"
    }
  }
}
```

이 HAL 문서는 현재 상태에 기반하여 각 주문에 대한 다른 링크를 즉시 보여준다.

- 첫번째 주문은 **COMPLETED**로 탐색 링크만 있다. 상태 전환 링크는 표시되지 않았다.
- 두번째 주문은 **IN_PROGRESS**로 **cancel** 링크와 **complete** 링크가 추가로 있다.

주문을 취소해보자 :

```powershell
$ curl -v -X DELETE http://localhost:8080/orders/4/cancel

> DELETE /orders/4/cancel HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 200
< Content-Type: application/hal+json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Mon, 27 Aug 2018 15:02:10 GMT
<
{
  "id": 4,
  "description": "iPhone",
  "status": "CANCELLED",
  "_links": {
    "self": {
      "href": "http://localhost:8080/orders/4"
    },
    "orders": {
      "href": "http://localhost:8080/orders"
    }
  }
}
```

이 response는 성공했음을 나타내는 **HTTP 200** 상태 코드를 보여준다. response HAL 문서는 해당 주문을 새로운 상태(**CANCELLED**)로 표시한다. 그리고 상태 변경 링크(**cancel**, **complete**)가 사라졌다.

같은 작업을 다시 시도한다면 ...

```powershell
$ curl -v -X DELETE http://localhost:8080/orders/4/cancel

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> DELETE /orders/4/cancel HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 405
< Content-Type: application/problem+json
< Transfer-Encoding: chunked
< Date: Mon, 27 Aug 2018 15:03:24 GMT
<
{
  "title": "Method not allowed",
  "detail": "You can't cancel an order that is in the CANCELLED status"
}
```

**HTTP 405 Method Not Allowed** response를 볼 수 있다. **DELETE**는 잘못된 작업이 되었다. 문제 응답 객체는 이미 “**CANCELLED**” 상태의 주문을 “**cancel**” 할 수 없음을 분명히 나타낸다.

또한 동일한 주문을 완료하려는 작업 또한 실패한다 :

```powershell
$ curl -v -X PUT localhost:8080/orders/4/complete

* TCP_NODELAY set
* Connected to localhost (::1) port 8080 (#0)
> PUT /orders/4/complete HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.54.0
> Accept: */*
>
< HTTP/1.1 405
< Content-Type: application/problem+json
< Transfer-Encoding: chunked
< Date: Mon, 27 Aug 2018 15:05:40 GMT
<
{
  "title": "Method not allowed",
  "detail": "You can't complete an order that is in the CANCELLED status"
}
```

이 모든것이 준비되면 order service에서 사용 가능한 작업을 조건부로 표시할 수 있다. 또한 잘못된 작업을 방지할 수 있다.

하이퍼미디어 및 링크 프로토콜을 활용하여 클라이언트를 더 견고하게 구축할 수 있으며 단순 데이터 변경으로 손상될 가능성이 적어진다. 그리고 Spring HATEOAS는 클라이언트에게 제공하는데 필요한 하이퍼미디어를 쉽게 구축할 수 있다.

### Summary

이 튜토리얼에서 REST API를 빌드하기 위한 다양한 전술에 참여했다. 결과적으로 REST는 단지 예쁜 URI와 XML 대신 JSON을 반환하는 것이 아니다. 

대신, 다음의 전술들은 제어하거나 제어하지 않을 수 있는 기존의 클라이언트를 service가 중단시킬 가능성을 줄이는데 도움을 준다 :

- 기존의 필드들을 제거하지 말고 그것들을 지원해라
- 클라이언트가 URI를 하드코딩할 필요가 없도록 rel-based link(상대 기반 링크)를 사용해라
- 기존의 링크를 가능한 오래 유지해라. URI를 변경해야하는 경우에도 기존 클라이언트가 새로운 기능에 대한 경로를 갖도록 rels를 유지해라.
- payload 데이터가 아닌 링크를 사용하여 다양한 상태 구동 작업을 사용할 수 있을 때 클라이언트에게 지시해라.

각 리소스 타입에 대한 **`RepresentationModelAssembler`** 구현을 구축하고 모든 Controller에서 이러한  구성 요소를 사용하는 것은 약간의 노력으로 보일 수 있다. 그러나 이런 서버측의 추가 설정(Spring HATEOAS 덕분에 쉬워진)은 API를 발전시키면서 제어하는 클라이언트(더 중요하게는 제어하지 않는)가 쉽게 업그레이드 할 수 있도록 한다.
	
