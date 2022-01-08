# Form
  + HTML의 입력form 생성
  + 입력form 값들에 대한 유효성 검증 (Validation)
  + 검증을 통과한 값들을 dictionary 형태로 제공
## Django Form
+ 하나의 URL로 입력form 제공과 처리를 같이 한다.       
  + GET 방식 요청
    + 입력form 출력
  + POST 방식 요청
    + 입력form을 받아 유효성 검증
      + 유효성 검증 성공 : 해당 데이터를 Model로 저장하고 SUCCESS_URL로 redirect 방식으로 이동
      + 유효성 검증 실패 : 오류 메세지와 함꼐 입력form 다시 출력 (render()을 통해 이동)
+ Form 클래스
  + app이름/forms.py 파일 생성
  + Form Field 클래스
    + Model의 Field와 유사
      + Model Field : DB 컬럼과 연결
      + Form Field : HTML 입력form과 연결
    + Form Field마다 기본 유효성 검증(validation) 처리가 구현되어 있음
  ```python
  # app이름/forms.py
  from django import forms
  
  # Example
  class PostForm(forms.Form): # Form 상속
    title = forms.CharField()
    content = forms.CharField(widget=forms.Textarea) # widget : 입력 양식
  ```
## ModelForm
+ 설정한 Model을 이용해 Form Field 구성
  + Form의 하위 클래스
  + ModelForm은 Model과 연동되어 save(저장) 기능 제공
+ 지정된 Model로부터 Field를 읽어 Form Field 생성
  + Form의 Field는 따로 만드는 것이 아닌 Model의 것을 가져와 생성
  + 일반적으로 Form의 Field와 Model Field은 서로 연결
  + 입력Form에서 받은 요청 파라미터의 처리는 Form과 동일
  ```python
  # app이름/forms.py
  from django import forms

  # Example
  class PostForm(forms.ModelForm): # Form 상속
    class Meta:
      model = Post        # Form을 생성할 때 사용할 Model클래스 지정
      fields = '__all__'  # Form Field를 생성할 때 사용할 Model Field 변수들 지정 
      
      # '__all__' : 모든 필드를 사용
      # ['필드명', '필드명', ...] : 사용할 필드들 지정
      # field들의 widget(입력 양식)을 변경하고자 할 때 widgets 속성에 딕셔너리 형태로 등록
      # widgets = {
      #     "name":forms.Textarea
      # }
  ```
## Form을 이용하여 입력Form template 생성
+ View에서 전달된 Form/ModelForm 객체를 이용하여 입력Form 생성
+ form은 iterable로 반복 시 각각의 Field를 반환
  ```python
  # {% load bootstrap4 %} # 부트스트랩 적용
  
  <form method='post'>
    {% csrf_token %} # POST 요청 방식일 경우 반드시 작성
    {% for form in form_list %}
      {{form}} # 입력 폼 
    # {% bootstrap_form form %} # 부트스트랩 적용
    {% endfor%}
  </form>
  ```
