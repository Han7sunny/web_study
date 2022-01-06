# MVT 패턴
> Django의 디자인 패턴
## M
Model
+ 데이터베이스의 데이터를 다룬다.
+ ORM(Object Relational Mapping : 객체 관계 매핑)을 이용하여 SQL문 없이 CRUD 작업을 처리
  + django ORM이 지원하는 DBMS : mysql, oracle, postgresql, sqlite3 
  + 객체와 관계형 데이터베이스의 데이터를 자동으로 연결         
                    
      |객체|데이터베이스|
      |----|----------|
      |Model 클래스|테이블|
      |클래스 변수|컬럼|
      |객체|행(데이터 1개)|
      
+ Model 클래스 
  + models.py에 Model 클래스 작성 (DB 테이블 이름 : app이름_class이름 -> Model의 Meta 클래스 db_table 속성을 통해 테이블 이름 지정 가능)
    + django.db.models.Model 상속
    + DB 테이블의 컬럼과 연결되는 변수인 Field를 class 변수로 선언
      ```python
      변수명 = Field객체(옵션) # 데이터 타입에 맞춰 선언
      # 주요 필드 옵션
        # primary_key : default False, True : primary key 컬럼, 생략하면 1씩 자동 증가하는 값을 가지는 id 컬럼 생성
        # max_length : 문자 타입의 최대 길이(글자수) 설정
        # null : default False, DB 필드에 NULL 허용 여부
        # unique : default False, 유일성 여부
        # DateField.auto_now_add : True면 처음 생성시 현재 시간으로 자동 저장 ex. create 
        # DateField.auto_now : True면 저장할 때마다 그 시점을 현재 시간으로 자동 저장 ex. update
        
        # blank : default false, 입력값 유효성 검사 시 empty 허용 여부
        # validators : 입력값 유효성 검사를 수행할 함수 지정
        # verbose_name : 입력 폼으로 사용될 때 label로 사용될 이름, 지정되지 않으면 필드명 사용
        
      ```
  + Model 클래스 작성 후 DB에 테이블 생성 또는 Model 클래스 수정 후 변경사항 갱신
    ```cmd
    python manage.py makemigrations # 변경 사항 DB에 넣기 위한 내역을 가진 migration 파일 생성
    python manage.py migrate # DB에 적용
    ```
  + admin 페이지에서 Model 관리하고 싶다면 admin.py에 Model 클래스 등록
    ```python
    # app이름/admin.py
    from .models import Model_클래스
    
    admin.site.register(Model_클래스)
    ```
+ Model 클래스를 이용한 데이터 CRUD
  + Model Manager : Model 클래스와 연결된 DB table에 SQL 실행할 수 있는 인터페이스
  + 각 Model 클래스의 class 변수 objects 이용하여 사용 : Model_클래스.objects.메소드

  + SELECT
    + 전체 조회
      + all()
    + 조건 조회
      + filter(**kwargs) : 조건 파라미터를 만족하는 조회 결과를 QuerySet으로 반환
      + exclude(**kwargs) : 조건 파라미터를 만족하지 않는 조회 결과를 QuerySet으로 반환, 조회결과 없으면 빈 QuerySet 반환
        + filter와 exclude는 SELECT의 WHERE 조건 지정
          + AND 조건
            ```python
            # ,로 이어서 작성
            Item.objects.filter(name='TV', price=4000)
            ```
          + OR 조건
            ```python
            # Q() 함수 사용
            Item.objects.filter(Q(name='TV' | ~Q(price=4000)) # Q() 앞에 ~ 붙이면 not
            # SELECT * FROM Item WHERE (name='TV' OR price!=4000)
            ```
      + get(**kwargs) : 조건 파라미터를 만족하는 조회결과를 Model 객체로 반환, 조회결과 1개일 때 사용
        + 조회결과 없으면 DoesNotExist 예외 발생, 1개 이상일 경우 MultipleObjectsReturned 예외 발생
       
  + INSERT / UPDATE
    + Model객체.save() : Model 객체의 pk값과 동일한 값이 DB에 없으면 insert, 있으면 update

  + DELETE
    + Model객체.delete() : Model 객체의 pk값의 레코드를 DB에서 삭제
## V
View
+ Client 요청을 받아 Client에게 응답할 때까지의 처리를 담당 -> Client가 요청한 작업을 처리하는 흐름(Work flow) 담당
+ 1개의 HTTP 요청에 대하여 1개의 View가 실행되어 요청된 작업 처리
  + urls.py에 사용자가 View를 요청할 URL을 mapping
+ 구현 방식은 함수기반 방식(FBV)와 클래스기반 방식(CBV)
  + app 내의 views.py에 작성
  + 함수 기반 View
    ```python
    # app이름/urls.py
    from django.urls import path
    from . import views
    
    urlpatterns = [path('요청 url 또는 요청 url/<타입"이름>', views.View함수, name='설정이름')]
    ```
    
    ```python
    # app이름/views.py
    from django.shortcuts import render
    
    def View함수(request [, path parameter 변수]): # HttpRequest 객체를 받는 request 변수 반드시 선언
      #
      # 사용자 요청 처리
      #
      return render(request, template 파일, template 파일로 전달할 값(dict))
    ```
  + 클래스 기반 View
    ```python
    # app명/urls.py
    # 1. class 기반 view 구현시 class 변수를 이용하여 속성만 설정한 경우
    from . import views
    
    # 2. class를 직접 구현하지 않고 url 매핑 시 GenericView.as_view()의 매개변수 설정
    from django.views.generic import ListView, DetailView, ...
    from .models import Model클래스
    
    urlpatterns = [
    # as_view() : View 객체 생성 및 dispatch() 메소드(HTTP 요청 방식에 맞는 처리 메소드 찾아 호출) 호출

    # 1. class 기반 view 구현시 class 변수를 이용하여 속성만 설정한 경우
    path('요청 url', views.View함수.as_view(), name='설정이름'),

    # 2. class를 직접 구현하지 않고 url 매핑 시 GenericView.as_view()의 매개변수 설정
    path('요청 url', ListView.as_view(model = Model클래스, template='template 파일'), name='설정이름'),
    ]
    ```
    
    ```python
    # app이름/views.py
    from django.views.generic import ListView, DetailView, ...
    
    # 1. class 기반 view 구현시 class 변수를 이용하여 속성만 설정한 경우
    class View함수(상속받을 Generic View):
      model = Model클래스 # 조회할 모델(테이블)
      template_name = 'template 파일' # 조회 결과를 template에 전달하며 호출
      # 조회 결과를 template에 전달 
    ```
+ template 호출 : 처리 결과 응답
  + render(request, template 파일, template 파일로 전달할 값(dict)) # request 반드시 선언해야 하는 매개변수
  + View에서 DB의 값을 변경한 후 template을 render()로 호출하면 처리결과를 전달할 수 있기에 새로 고침하면 다시 적용되는 문제 발생
    + #### redirect()를 통해 DB의 처리(update/insert)와 응답하는 것의 처리 분리해야 함 ####
    ```python
    from django.shortcuts import redirect
    from django.urls import reverse # View에서 urls.py의 path 설정 이름을 이용해 url을 조회.

    # 함수 또는 클래스 내부 함수
    # redirect() 함수를 이용해 응답처리 View를 웹브라우저가 다시 요청하도록 설정.

    url_str = reverse("app이름:응답처리_설정이름", args=[path parameter값]) #args: path parameter값 등록
    return redirect(url_str)
    ```
# T
Template
+ Client에게 보여지는 부분(응답화면)의 처리를 담당
+ HTML로 변환
+ templates/app이름 폴더 생성한 후 template 파일 생성
+ template 파일 : HTML(정적 작업) + django template 문법(동적 작업) 사용하여 만든 html
+ template 문법
  ```python
  # 주석(comment)
  {# 한 줄 주석 #}
  
  {% comment %}
    #주석 내용
  {% endcomment %}
  
  <!-- html주석, 소스 코드에 노출됨 --> 
  
  # template 변수
  {{변수}}
  
  # template 필터
  {{변수 | 필터}}
  
  # template 태그
  {% 태그 %}
  
  # 반복문
  {% for 변수 in iterable %}
    # 반복 구문
  {% empty%} # 반복 조회할 iterable 원소가 없을 때
  {% endfor %}
  
  # 조건문
  {% if 조건%}
  {% elif 조건 %}
  {% else %}
  {% endif %}
  
  # URL
  # urls.py에 등록된 URL을 가져와 URL 생성
  {%url 'app이름:url설정이름' 전달값 %}
  
  # extends
  # 상속 받을 부모 template 지정(기존 template을 상속 받아 재사용)
  {% extends '상속받을 template 경로' %}
  
  # block
  # 재정의할 구역 지정
 
  # 부모 template
  {% block block이름 %}
    # 자식 template에서 재정의할 영역 지정해줌
  {% endblock block이름%}
  
  # 자식 template
  {% block block이름 %}
    # 재정의
  {% endblock block이름%}  
  
  ```
  
    
  
  
