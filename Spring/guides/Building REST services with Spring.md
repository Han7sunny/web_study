# Building REST services with Spring
##### [원문](https://spring.io/guides/tutorials/rest/)


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

+ stateless 부터 stateful한 services까지의 스펙트럼

REST는 유비쿼터스에 있더라도 그 자체로 표준이 아니라 web-scale 시스템을 구축하는데 도움이 될 수 있는 아키텍처의 접근 방식, 스타일, 제약 조건의 집합

이 튜토리얼에서는 Spring portfolio를 사용하여 REST의 stackless 기능을 활용하며 RESTful 서비스를 구축

회사의 직원들을 관리하는 간단한 payroll 서비스 
직원 객체들을 H2 DB에 저장하고 JPA를 통해 접근
Spring MVC layer로 불리는 인터넷을 통한 접근을 허용하는 것으로 래핑함


### Getting Started
Starting with Spring Initializr
1. https://start.spring.io
2. Gradle and java ,, 이 튜토리얼에서는 maven 기반
3. Dependencies : Web, JPA, H2
4. Name : Payroll , Generate and download zip file

```java
package payroll;

import java.util.Objects;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
class Employee {

  private @Id @GeneratedValue Long id;
  private String name;
  private String role;

  Employee() {}

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
+ @GeneratedValue 
