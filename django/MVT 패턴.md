# MVT 패턴
> Django의 디자인 패턴
## M
Model
+ 데이터베이스의 데이터를 다룬다.
+ ORM(Object Relational Mapping : 객체 관계 매핑)을 이용하여 SQL문 없이 CRUD 작업을 처리
## V
View
+ Client 요청을 받아 Client에게 응답할 때까지의 처리를 담당
+ Client가 요청한 작업을 처리하는 흐름(Work flow)을 담당
+ 구현 방식은 함수기반 방식(FBV)와 클래스기반 방식(CBV)
+ template 호출
## T
Template
+ Client에게 보여지는 부분(응답화면)의 처리를 담당
+ HTML로 변환
