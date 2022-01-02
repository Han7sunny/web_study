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
  + admin 페이지에서 Model 관리하고 싶다면 admin.py에 Model 클래스 등록
    ```python
    admin.site.register(Model_클래스)
    ```
+ Model 클래스를 이용한 데이터 CRUD
  + Model Manager : Model 클래스와 연결된 DB table에 SQL 실행할 수 있는 인터페이스
  + 각 Model 클래스의 class 변수 objects 이용하여 사용 : Model_클래스.objects.메소드
    + 전체 조회
      + all()
    + 조건 조회
      + filter(**kwargs) : 조건 파라미터를 만족하는 조회 결과를 QuerySet으로 반환
      + exclude(**kwargs) : 조건 파라미터를 만족하지 않는 조회 결과를 QuerySet으로 반환, 조회결과 없으면 빈 QuerySet 반환
      + get(**kwargs) : 조건 파라미터를 만족하는 조회결과를 Model 객체로 반환, 조회결과 1개일 때 사용
        + 조회결과 없으면 DoesNotExist 예외 발생, 1개 이상일 경우 MultipleObjectsReturned 예외 발생
  + 
## V
View
+ Client 요청을 받아 Client에게 응답할 때까지의 처리를 담당 -> Client가 요청한 작업을 처리하는 흐름(Work flow) 담당
+ 1개의 HTTP 요청에 대하여 1개의 View가 실행되어 요청된 작업 처리
  + urls.py에 사용자가 View를 요청할 URL을 mapping
    + path(요청 URL, View 함수, name='이름')
+ 구현 방식은 함수기반 방식(FBV)와 클래스기반 방식(CBV)
  + app 내의 views.py에 작성
  + 함수 기반 View
  + 클래스 기반 View
+ template 호출
## T
Template
+ Client에게 보여지는 부분(응답화면)의 처리를 담당
+ HTML로 변환
