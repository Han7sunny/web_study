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
      # exclude = ['필드명', '필드명', ...] # Form Field를 생성할 때 제외할 Model Field 변수들 지정 
      # field들의 widget(입력 양식)을 변경하고자 할 때 widgets 속성에 딕셔너리 형태로 등록
      # widgets = {
      #     "name":forms.Textarea
      # }
  ```
## Generic Edit View
+ app이름/views.py
+ CreateView
  + 저장 - insert 처리
  + GET 방식 요청: 입력양식 화면으로 이동 (render())
  + POST 방식 요청: 입력(등록) 처리. 
    + 처리성공: 성공페이지로 이동 (redirect)
    + 처리실패: 입력양식 화면으로 이동 (render())
    ```python
    from django.views.generic import CreateView
    from django.urls import reverse_lazy

    class ExampleCreateView(CreateView):
      template_name = '입력폼을 작성한 template 파일 경로'
      form_class = 입력폼에서 사용할 Form/ModelForm
      
      # 등록 성공 후 redirect 방식으로 이동할 View의 url을 문자열로 반환
      success_url = reverse_lazy('app이름:설정이름', args=[전달할 값]) # path parameter로 전달할 값들을 리스트에 순서대로 담는다.
      
      # 또는
      
      # 등록 / 수정 / 삭제 작업 후 작업한 Model 정보를 "path parameter"로 사용할 경우 get_success_url overriding
      # insert/update한 모델객체 조회: self.objects
      # overriding
      def get_success_url(self):
        return reverse_lazy('app이름:설정이름', args=[전달할 값]) 
        
        # pk로 접근 : urls.py에서 해당 url 경로의 path parameter <타입:pk> 입력
        # return reverse_lazy('app이름:설정이름', args=[self.object.pk])

    ```
+ UpdateView
  + GET 방식 요청 처리 : pk로 수정할 정보를 조회해서 template(수정 form)으로 전달(render())
    + urls.py에서 해당 url path parameter로 <int:pk> 지정           
      path('update/<int:pk>', views.ExampleUpdateView.as_view(), name='update')
  + POST 방식 요청 처리 : update 처리, redirect방식으로 View를 요청
    ```python
    from django.views.generic import UpdateView
    from django.urls import reverse_lazy

    class ExampleUpdateView(UpdateView):
      template_name = "수정 form template파일 경로"
      form_class = 수정 form에서 사용할 Form/ModelForm
      model = Post # 수정폼 template에 전달할 값을 조회하기 위한 Model 클래스 등록
      # 조회결과를 "모델클래스명 소문자" 또는 "object"(권장) 라는 이름으로 template에게 전달
      # context_object_name = 'template에서 사용할 이름 지정'
      
      # 수정처리 후 redirect 방식으로 이동할 View의 url을 문자열로 반환
      success_url = reverse_lazy('app이름:설정이름', args=[전달할 값]) # path parameter로 전달할 값들을 리스트에 순서대로 담는다.

      # 또는
      
      # 등록 / 수정 / 삭제 작업 후 작업한 Model 정보를 "path parameter"로 사용할 경우 get_success_url overriding
      def get_success_url(self):
          return reverse_lazy('app이름:설정이름', args=[전달할 값])
          # self.object : update 처리한 Model 객체
    
    # app이름/urls.py에서
    # path(url/<int:pk>, views.ExampleUpdateView.as_view(), name='설정이름')
    ```
+ DeleteView
  + 삭제 확인 화면을 거쳐서 삭제 처리한다.
    ```python
    from django.views.generic import DeleteView
    from django.urls import reverse_lazy

    class ExampleDeleteView(DeleteView):
      template_name = "삭제form template파일 경로"
      model = 삭제할 Model
      success_url = reverse_lazy('app이름:설정이름')
      
    # app이름/urls.py에서
    # path(url/<int:pk>, views.ExampleDeleteView.as_view(), name='설정이름')  
    ```
  + template 없이 pk로 삭제 
    ```python
    from django.shortcuts import redirect
    # 함수기반으로 작성. path parameter로 삭제할 글의 id(pk)를 받아서 삭제처리.

    def example_delete(request, pk):
        example = Model클래스.objects.get(pk=pk) 
        example.delete() # model의 pk 삭제
        return redirect("/") # root
          
    # app이름/urls.py에서
    # path('url/<int:pk>', views.example_delete, name='설정이름')
    ```

## Generic Display View
+ DetailView
  + pk로 조회한 결과를 template으로 보내주는 generic View
  + url 패턴 : pk를 path 파라미터로 받는다. <type:pk> 변수명을 pk로 지정해야 한다. (urls.py)
    ```python
    from django.views.generic import DetailView
    
    class ExampleDetailView(DetailView):
      template_name = '응답할 template 파일 경로'
      model = PK로 조회할 모델클래스
      # 조회결과를 "모델클래스명 소문자" 또는 "object"(권장) 라는 이름으로 template에게 전달
      # context_object_name = 'template에서 사용할 이름 지정'
      
    # app이름/urls.py에서
    # path(url,DetailView.as_view(template_name="응답할 template 파일 경로",model='PK로 조회할 모델클래스'), name='설정이름') 한 줄로 작성 가능
    ```
+ ListView
  + 지정한 Model의 전체 데이터를 조회하여 Template으로 전달
  + 조회결과를 template에 "object_list"(권장) 또는 "모델이름소문자_list" 라는 이름으로 전달
     ```python
    from django.views.generic import ListView

    class ExampleListView(ListView):
        template_name = "결과를 보여줄 template 파일 경로"    
        model = 조회할 Model클래스
    ```
  + ListView는 paging 기능 지원 
    + get_context_data(self, **kwargs) : 추가적으로 전달할 context data(Paging ..)을 위하여 overriding
    + context data : view가 template에게 전달하는 값(dictionary)을 의미
    ```python
    from django.views.generic import ListView

    class ExampleListView(ListView):
        template_name = "결과를 보여줄 template 파일 경로"    
        model = 조회할 Model클래스
        
        paginate_by = n  # 한 페이지의 데이터 개수 n , paginator 생성됨
        # 동일 : Paginator(조회할 Model클래스.objects.all(), n) -> context_data에 담김
        
        # 페이징관련 값들을 context data에 추가
        # get_context_data(): Generic View를 구현할 때 template에게 추가적으로 전달해야하는 context data가 있을때 overriding
        def get_context_data(self, **kwargs): # overriding
          # 부모객체의 get_context_data()를 호출해서 generic view가 자동으로 생성한 Context data를 받아온다.
          context = super().get_context_data(**kwargs) # 반드시 실행! 부모가 작성한 것 쓰지 않는다고 해도 {조회결과, Paginator 등}
          
          # ListView에서 paginate_by 속성을 설정하면 context data에 Paginator객체가 등록된다.
          paginator = context['paginator']
          page_group_count = n # 페이지그룹에 속한 페이지 개수 n
          current_page = int(self.request.GET.get('page', 1)) # 사용자가 요청한 페이지 번호, 넘어온 값 없으면 1(처음)
          # CBV에서 HttpRequest는 self.request로 사용할 수 있다.

          # 페이지 그룹의 페이지 범위 조회
          start_idx = int((current_page-1)/page_group_count)*page_group_count # # current_page - 1 : index 매칭
          end_idx = start_idx + page_group_count
          page_range = paginator.page_range[start_idx : end_idx] # range(start,end)
          
          # 그룹의 시작 페이지가 이전페이지가 있는지, 그룹의 마지막 페이지가 다음페이지가 있는지 여부 + 페이지 번호
          start_page = paginator.page(page_range[0]) #시작 페이지의 Page객체 (생성)
          end_page = paginator.page(page_range[-1]) # 마지막 페이지의 Page객체 (생성)

          has_previous = start_page.has_previous() #시작의 이전페이지가 있는지 여부
          has_next = end_page.has_next() # 마지막 페이지의 다음 페이지가 있는지 여부

          context['page_range'] = page_range # range객체 생성(시작 페이지 번호, 마지막 페이지 번호)
          if has_previous: # 이전 페이지 그룹 존재
              context['has_previous'] = has_previous
              context['previous_page_no'] = start_page.previous_page_number  # 시작페이지의 이전 페이지 번호

          if has_next: # 다음 페이지 그룹 존재
              context['has_next'] = has_next
              context['next_page_no'] = end_page.next_page_number # 마지막 페이지의 다음 페이지 번호

          return context # dictionary -> 처음에 부모로부터 받아옴  + 'page_range' + ... 추가됨
          # 최종적으로 render()의 딕셔너리 값으로 넘어감
    ```
  
## Form을 이용하여 입력Form template 생성
+ View에서 전달된 Form/ModelForm 객체를 이용하여 입력Form 생성
+ form은 iterable로 반복 시 각각의 Field를 반환
  ```python
  # template 파일
  # {% load bootstrap4 %} # 부트스트랩 적용
  
  <form method='post'>
    {% csrf_token %} # POST 요청 방식일 경우 반드시 작성
    {% for form in form_list %}
      {{form}} # 입력 폼 
    # {% bootstrap_form form %} # 부트스트랩 적용
    {% endfor%}
  </form>
  ```
