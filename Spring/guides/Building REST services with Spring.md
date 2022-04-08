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
