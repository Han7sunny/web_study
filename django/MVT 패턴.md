# MVT 패턴
> Django의 디자인 패턴
## M
Model
+ 데이터베이스의 데이터를 다룬다.
+ ORM(Object Relational Mapping : 객체 관계 매핑)을 이용하여 SQL문 없이 CRUD 작업을 처리
  + 객체와 관계형 데이터베이스의 데이터를 자동으로 연결         
                    
      |객체|데이터베이스|
      |----|----------|
      |Model 클래스|테이블|
      |클래스 변수|컬럼|
      |객체|행(데이터 1개)|
  + models.py에 Model 클래스 작성 (django.db.models.Model 상속, 테이블 이름 : app이름_class이름)
  + Model 클래스 작성 후 DB에 테이블 생성
    ```cmd
    python manage.py makemigrations # 변경 사항 DB에 넣기 위한 내역을 가진 migration 파일 생성
    python manage.py migrate # DB에 적용
    ```
 
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
