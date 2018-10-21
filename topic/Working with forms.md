# Working with forms

> About this document
>
> 이 문서는 웹 폼의 기본사항과 장고에서 다루는 방법을 소개한다.
> Form API의 특정 영역에 대한 자세한 내용은 `Forms API`, `Forms fields`, 'Form and validation' 문서를 참조하라.

아무것도 하지 않고 콘텐츠를 게시하고 방문자의 입력을 수락하지 않는 웹 사이트 및 응용 프로그램을 만들 계획이 아니라면 form을 이해하고 사용해야한다.

Django는 사이트 방문자로부터 입력을 받아 form을 작성한 후 입력을 처리하고 응답하는데 도움이되는 도구와 라이브러리를 제공한다.

## HTML forms

HTML에서 form은 방문자가 텍스트 입력, 옵션 선택, 객체 또는 컨트롤 조작 등과 같은 작업을 수행할 수 있는 `<form> ... </form>` 내의 요소이고 그 정보를 서버로 보낸다.

이러한 form 인터페이스 요소 중 일부(텍스트 입력 또는 확인란)는 매우 간단하며 HTML 자체에 내장되어 있다. 다른 것들은 훨씬 복잡한다. 날짜 선택 도구를 팝업하거나, 슬라이더를 이동하거나 컨트롤 조작할 수 있는 인터페이스는 일반적으로 HTML 양식 `<input>`요소와 함께 Javascript CSS를 사용하여 이러한 효과를 얻는다.

`<input>`요소뿐만아니라, form에 두가지를 지정해야한다.
  - where: 사용자의 입력에 해당하는 데이터가 반환되어야하는 URL
  - How: 데이터가 반환되어야하는 HTTP method

예를 들어, Django 관리자의 로그인 폼은 여러개의 <input> 요소를 포함한다. 하나는 사용자 이름의 `type="text"`, password는 `type="password"` 다른 하나는 "Log in" 버튼을 위한 `type="submit"이`이다.
또한 사용자가 볼 수 없는 몇가지 숨겨진 필드가 있고, 장고는 다음에 수행할 작업을 결정할 때 사용한다. 또한 form 데이터가 `<form>`의 `action` 속성에 `admin/`에 지정된 URL로 보내져야하며, `method` 속성에 `post` HTTP 매커니즘을 사용하여 보내야한다고 브라우저에 알린다.

`<input type="submit" value="Log in">`요소가 트리거되면 데이터는 `/admin/`으로 리턴된다.

## GET and POST

`GET`, `POST`는 폼을 다룰 때 사용할 수 있는 유일한 HTTP 메서드이다.
Django의 로그인 폼은 `POST` 메서드를 사용하여 반환된다. `POST` 메서드는 브라우저가 form 데이터를 묶어 인코딩 전송하여 서버로 보낸다음 응답을 수신한다.
반대로 `GET`은 제출된 데이터를 문자열로 묶어 URL을 작성하는데 사용한다.  URL에는 데이터를 전송해야하는 주소는 물론, 키와 데이터 값이 포함된다.

`GET`과 `POST`는 일반적으로 다른 용도로 사용된다.

시스템의 상태를 변경하는데 사용할 수 있는 요청 (예: 데이터베이스에서 변경 요청)은 `POST`를 사용해야한다. `GET`은 시스템 상태에 영향을 주지 않는 요청에마 사용해야한다.

비밀번호가 URL에 표시되고 브라우지 기록 및 서버 로그에도 일반텍스트로 표시되므로 `GET`은 비밀번호 양식에 적합하지 않다. 대량의 데이터나 이미지같은 바이너리 데이터에도 적합하지 않다. 관리자 양식에 `GET` 요청을 사용하는 웹 응용프로그램은 보안 위험이 있다. 공격자가 양식의 민감한 부분에 대한 액세스 권한은 얻으려는 요청 정보를 쉽게 모방할 수 있다.`POST`는 Django의 CSRF 보호와 같은 다른 보안 기능과 결합되어 액세스를 보다 효율적으로 제어한다.

반면 GET 요청을 나타내는 URL은 책갈피, 공유 또는 재요청할 수 있기때문에 웹 검색같은 양식에 적합하다.

---

## Django's role in forms

form 처리는 복잡한 사업이다. Django의 Admin을 생각할 때 다양한 유형의 여러 데이터 항목을 양식으로 표현하고, HTML 로 렌더링하고, 편리한 인터페이스를 사용하여 편집하고, 서버로 반환하고, 유효성을 검사하고, 추가 처리를 위해 정리한 다음 저장하거나 전달해야할 수도 있다.

Django의 폼 기능은 작업의 상당 부분을 단순화하고 자동화할 수 있으며 대부분의 프로그래머가 작성한 코드에서 수행할 수 있는것보다 더 안전히 수행할 수 있다.

Django는 폼과 관련된 세가지 부분을 처리한다.
  - 데이터 준비 및 재구성하여 렌더링 준비
  - 데이터에 대한 HTML 양식 작성
  - 클라이언트로부터 제출된 양식 및 데이터 수신 처리

이 모든 작업을 수동으로 수행하는 코드를 작성하는 것은 가능하지만, Django가 모든 작업을 처리할 수 있다.

---

## Forms in Django

HTML 양식을 간단히 설명했지만, HTML `<form>`은 기계화의 일부일 뿐이다.

웹 응용 프로그램의 컨텍스트에서 `form`은 HTML `<form>` 생성된 Django `Form` 또는 제출 될 때 반환된 구조화 된 데이터나 end-to-end 작업 부분의 콜렉션을 참조한다.

## Django의 `Form` 클래스

이 컴포넌트 시스템의 핵심은 `Django`의 `Form`클래스이다. Django 모델이 객체의 논리적 구조, 동작 및 파트가 우리에게 표시되는 방식을 설명하는 것과 같은 방식으로 `Form` 클래스는 form을 설명하고, form이 작동하고 나타나는 방식을 결정한다.

모델 클래스의 필드가 데이터베이스의 필드에 매핑되는 것과 비슷한 방식으로 form 클래스의 필드는 HTML form `<input>` 요소에 매핑된다. (ModelForm은 Form을 통해 모델 클래스의 필드를 HTML 폼의 `<input>`요소로 매핑한다. 이것이 장고 관리자 기반이다.)

form의 필드 자체는 클래스이다. 양식이 제출되면 양식 데이터를 관리하고 유효성 검사를 수행한다. `DateField`와 `FileField`는 매우 다른 종류의 데이터를 처리하고, 그 데이터와 다른 작업을 수행해야한다.

`form` 필드는 브라우저의 사용자에게 HTML 위젯(사용자 인터페이스)로 표시된다. 각 필드 유형에는 적절한 기본 위젯 클래스가 있지만 피룡에 따라 재정의할 수 있다.

----

## Instantiating, processing, and rendering forms

Django가 객체를 렌더링할때, 일반적으로 다음을 수행한다.
  - view에서 가져온다.(데이터베이스에서 가져온다.)
  - template context로 전달한다.
  - 템플릿 변수를 사용하여 HTML 마크 업으로 확장된다.

템플릿의 폼 렌더링은 다른 종류의 오브젝트를 렌더링하는 것과 거의 동일한 작업을 포함하지만 몇가지 중요한 차이가 있다.

데이터가 포함되지 않은 모델 인스턴스의 경우, (?)
반면에 사용자가 채워지지않은 폼을 채우길 원할 때 사용한다.

따라서 뷰에서 모델 인스턴스를 처리할 때 일반적으로 데이터베이스에서 모델 인스턴스를 검색한다. form을 다룰 때 일반적으로 view에서 인스턴스화한다.

form을 인스턴스화할 때 양식을 비워두거나 미리 채울 수 있다.
  - 저장된 모델 인스턴스의 데이터(관리자 폼의 편집양식처럼)
  - 다른 소스에서 수집한 데이터
  - 이전 HTML 양식 제출에서 받은 데이터

  마지막의 경우가 가장 흥미롭다. 사용자가 웹 사이트를 읽을 수 있게할뿐만아니라 정보를 보낼 수 있게하기 때문이다.

---

## building a form

### 끝내야할 작업

사용자 이름을 얻기위한 웹사이트에서 간단한 양식을 만들고 싶다 가정할때, 템플리셍서 다음과 같은 form이 필요하다.

```html
<form action="/your-name/" method="post">
    <label for="your_name">Your name: </label>
    <input id="your_name" type="text" name="your_name" value="{{ current_name }}">
    <input type="submit" value="OK">
</form>
```

`POST` 메서드를 사용하여 form 데이터를 URL `/your-name/`로 리턴하도록 브라우저에 지시한다. `Your name:` 이라는 텍스트 필드와 "OK"라고 표시된 버튼이 표시된다. 템플릿 컨텍스트에 current_name 이라는 변수ㅜ가 있으면 your_name 필드를 미리 채우는데 사용된다.

HTML 양식을 포함하는 템플릿을 렌더링하고 current_name 필드를 적절히 제공할 수 있는 view가 필요하다.

form이 제출되면 서버에 전송된 `POST`요청에는 `form` 데이터가 포함된다.

이제 해당 `/your-name/` URL에 해당하는 view가 필요하며 요청에서 적덜한 키 / 값 쌍을 찾은 다음 처리한다.

이 예제는 매우 간단한 form 형태이다. 실제로 form에는 수십, 수백개의 필드가 포함될 수 있으며 그 중 많은 필드가 미리 채워져야할수도 있다. 사용자는 작업을 완료하기 전 편집 - 제출 주기를 여러번 검토해야할것이다.

양식을 제출 전 브라우저에서 일부 유효성 검사를 수행한다. 우리는 훨씬 복잡한 필드를 사용하여 사용자가 달력에서 날짜를 선택하는 등의 작업을 수행할 수 있다.

---

## building a form in Django

### The `Form` class

우리는 HTML 양식이 어떻게 보이길 원하는지 알고 있다. 장고에서 출발점은 다음과 같다.

```python
# forms.py
from django import forms

class NameForm(forms.Form):
  your_name = forms.CharField(label='Your name', max_length=100)
```

이 단일 필드(your_name)이 있는 Form 클래스를 정의한다. 사람이 잘 읽을 수 있게 레이블 필드를 적용했다. 이 `label`은 렌더링 될 때 `<label>`로 나타나게된다.

필드의 최대 허용길이는 max_length에 의해 정의된다. 이것은 두가지 작업을 한다.
  - html `<input>`에 `max_length="100"`을 넣는다.
  - Django가 브라우저에서 form 데이터를 받을 때 데이터 길이를 확인한다.

Form  인스턴스에는 모든 필드에 대한 유효성 검사 루틴을 실행하는 `is_valid()`메서드가 있다. 이 메서드를 호출하면 모든 필드에 유효한 데이터가 들어있다면 `True`를 반환하고, 폼의 데이터가 `cleaned_data` 속성에 배치된다.

위의 `NameForm` 모델이 렌더링되면 다음과 같다.

```html
<label for="your_name">Your name: </label>
<input id="your_name" type="text" name="your_name" maxlength="100" required>
```

여기에는 `<form>` 태그 또는 제출버튼이 포함되지 않는다. 템플릿에 직접 제공해야한다.

### The view

Django 웹 사이트로 보내진 form 데이터는 일반적으로 form을 렌더링한 동일한 view에 의해 처리된다. 이것은 우리가 같은 로직을 재사용할 수 있게 해준다.
양식을 처리하려면 URL에 대한 view에서 양식을 인스턴스화해야한다.

```python
# views.py
from django.http import HttpResponseRedirect
from django.shortcuts import render

from .forms import NameForm

def get_name(request):
  # POST가 오면 form data를 처리한다.
  if request.method == 'POST':
    # form 데이터를 받아 form 인스턴스를 생성한다.
    form = NameForm(request.POST)
    # form data가 모두 유효한지 검사한다.
    if form.is_valid():
      # 필요에 따라 form.cleaned_data에서 데이터를 처리한ㄷ나.
      # ...
      # 처리 후 URL redirect
      return HttpResponseRedirect('/thanks/')
  else:
    # GET 메서드 혹은 다른 방식으로 받았을 때 빈 양식을 생성한다.
    form = NameForm()

  return render(request, 'name.html', {'form': form})
```

이 view에 `GET` 요청이 들어오면 빈 form 인스턴스를 만들어 렌더링할 템플릿 컨텍스트에 배치한다. 이는 URL을 처음 방문할 때 발생한다.

`POST` 요청시 양식을 제출하면 view에서 form 인스턴스를 다시 작성하고 요청 데이터로 채운다. `form=NameForm(request.POST)` `binding data to the form`이라고 한다. (채우면 bound form이 된다.)

form의 `is_valid()`메서드를 호출한다. `True`이면 양식이 있는 템플릿으로 돌아간다. `False`면 이전에 제출된 내용을 다시 채우고 양식을 재작성하는 페이지로 다시 되돌아간다.


`is_valid()`가 `True`이면 `cleaned_data` 속성에서 모든 유효성이 검사된 양식 데이터를 찾을 수 있다. 이 데이터를 사용하여 HTTP리다이렉션을 보내기 전 데이터베이스를 업데이트하거나 브라우저로 다른 처리를 수행하여 다음에 어디로 가야하는지 알려준다.

### The template


`name.html` 템플릿에서 많은 작업을 수행할 필요가 없다.

```html
# name.html
<form action="/your-name/" method="post">
    {% csrf_token %}
    {{ form }}
    <input type="submit" value="Submit">
</form>
```

{{ form }}의 필드와 속성은 장고 템플릿 언어에 의해 HTML 언어로 풀리게된다.

---

## More about Django Form classes

모든 form 클래스는 `django.forms.Form` 또는 `django.forms.ModelForm`의 하위 클래스로 작성된다. `ModelForm`은 `Form`의 하위 클래스로 생각할 수 있다. `form`과 `ModelForm`은 실제로 `BaseForm` 클래스에서 일반적 기능을 상속받지만, 구현 세부사항은 중요하지 않다.

### Bound and Unbound form instances

`Bound and unbound forms`의 차이는 중요하다.
  - 언바운드 형식에는 연결된 데이터가 없다. 사용자에게 렌더링되면 비어있거나 기본값을 포함한다.
  - 바인드 된 양식을 제출했으면 해당 데이터가 유효한지 여부를 판별하는데 사용할 수 있다. 잘못된 바운드 양식이 렌더링되면 사용자에게 수정해야할 데이터를 알려주는 인라인 오류 메세지가 포함된다.
form의 `is_bound` 속성은 form 의 데이터가 바인드 되었는지 여부를 알려준다.

### More on fields

위의 예제보다 더 유용한 개인 웹 사이트에서 "Contact me" 기능을 구현하는데 사용할 수 있다.

```python
# forms.py
from django import forms

class ContactForm(forms.Form):
  subject = forms.CharField(max_length=100)
  message = forms.CharField(widget=forms.Textarea)
  sender = forms.EmailField()
  cc_myself = forms.BooleanField(required=False)
```

우리의 이전 양식은 하나의 필드인 `your_name` `CharField`를 사용했다. 현재 양식에는 `subject. message. sender, cc_myself`라는 네개의 필드가 있다. `CharField, EmailField, BooleanField`는 사용가능한 필드 유형 중 세가지 이다. 전체 목록은 `Form Fields`에서 찾을 수 있다.

### Widgets

각 form 양식에는 해당 위젯 클래스가 있으며, 이 클래스는 `<input tpye="text">`와 같은 HTML 양식 위젯에 해당한다. 대부분의 경우 이 필드는 유용한 기본 위젯이 있다. 예를 들어 기본적으로 `CharField`에는 HTML에서 `<input type="text">`를 생성하는 `TextInput` 위젯이 있다. `<textarea>`가 필요하면 , `message` 필드에 했던것처럼 양식 필드를 정의할 때 적절한 위젯을 지정할 수 있다.

### Field data

form과 함께 제출된 데이터가 무엇이던간에 `is_valid()`를 호출하여 유효성을 검사하고 `is_valid()`가 `True`를 반환하면 유효성이 검사된 양식 데이터는 `form.cleaned_data` dict에 저장된다. 이 데이터는 파이썬 유형으로 변환된다.

> 검증되지 않은 데이터 `request.POST`로 접근할 수 있지만 검증된 데이터로 접근하는 것이 더 좋다.

위의 `ContactForm`에서 `cc_myself`는 불리언 값을 나타낸다. 마찬가지로 `IntegerField`, `FloatField`와 같은 필드는 값을 각각 파이썬 int 및 float로 변환한다.

여기서 양식을 처리하는 법을 나타낸다.

```python
#view.py

from django.core.mail import send_mail

if form.is_valid():
  subject = form.cleaned_data['subject']
  message = form.cleaned_data['message']
  sender = form.cleaned_data['sender']
  cc_myself = form.cleaned_data['cc_myself']

  recipients = ['info@example.com']
  if cc_myself:
    recipients.append(sender)

  send_mail(subject, message, sender, recipients)
  return HttpResponseRedirect('/thanks/')
```

일부 필드 유형은 추가 처리가 필요하다. 예를 들어 양식을 사용하여 업로드된 파일은 다르게 처리해야한다.(`request.POST`가 아닌 `request.FILES`에서 값을 가져온다.) form으로 파일 업로드하는 것은 `Binding uploaded files to a form`을 참조한다.

---

## Working with form templates

form을 템플릿으로 가져오려면 form 인스턴스를 템플릿 컨텍스트에 배치하기만하면 된다. 따라서 form이 컨텍스트에서 form이라고 하면 {{ form }}은 `<label>`, `<input>`요소를 적절히 렌더링한다.

### Form rendering options

`<label>/<input>` 쌍에는 몇가지 출력 옵션이 있다.
  - {{ form.as_table }} 폼의 요소들을 테이블화시킨다.
  - {{ form.as_p }} 폼의 요소 각각을 p 태그로 감싼다.
  - {{ form.as_ul }} 폼의 요소를 <li> 태그로 감싼다.

table 또는 ul 로 만들려면 직접 제공해야한다. `ContactForm` 인스턴스의 `{{ form.as_p }}`옵션 출력은 다음과 같다.

```html
<p><label for="id_subject">Subject:</label>
    <input id="id_subject" type="text" name="subject" maxlength="100" required></p>
<p><label for="id_message">Message:</label>
    <textarea name="message" id="id_message" required></textarea></p>
<p><label for="id_sender">Sender:</label>
    <input type="email" name="sender" id="id_sender" required></p>
<p><label for="id_cc_myself">Cc myself:</label>
    <input type="checkbox" name="cc_myself" id="id_cc_myself"></p>
```

각 필드에는 `id_<field-name>` 으로 설정된 ID속성이 있으며, 이는 함께 제공되는 레이블 태그에 의해 참조된다. 이는 form이 screen reader software같은 기술과 연관될 수 있도록 하는데 중요하다. 또한 레이블 ID가 생성되는 방식을 사용자 정의할 수 있다.

### Rendering fields manually

Django가 form의 필드를 unpack할 필요는 없다. 우리가 원한다면 수동으로 처리할 수 있다. (예를들면 필드 정렬) 각 필드는 {{ form.name_of_field }}를 사용하는 폼의 속성으로 사용할 수 있으며, Django 템플릿에서 적절히 렌더링된다.

```django
{{ form.non_field_errors }}
<div class="fieldWrapper">
    {{ form.subject.errors }}
    <label for="{{ form.subject.id_for_label }}">Email subject:</label>
    {{ form.subject }}
</div>
<div class="fieldWrapper">
    {{ form.message.errors }}
    <label for="{{ form.message.id_for_label }}">Your message:</label>
    {{ form.message }}
</div>
<div class="fieldWrapper">
    {{ form.sender.errors }}
    <label for="{{ form.sender.id_for_label }}">Your email address:</label>
    {{ form.sender }}
</div>
<div class="fieldWrapper">
    {{ form.cc_myself.errors }}
    <label for="{{ form.cc_myself.id_for_label }}">CC yourself?</label>
    {{ form.cc_myself }}
</div>
```

완전한 `<label>`요소는 `label_tag()`를 사용하여 생성할 수 있다.

```django
<div class="fieldWrapper">
    {{ form.subject.errors }}
    {{ form.subject.label_tag }}
    {{ form.subject }}
</div>
```

### Rendering form error messages

물론, 이런 유연성은 더 많은 작업을 요구한다. 지금까지 우리는 어떻게 폼 오류를 표시할지에 대해 걱정할 필요가 없었다.이 예에서 각 필드에 대한 오류 form 전체에 대한 오류를 반드시 해결해야 한다. `{{ form.non_field_errors }}`는 form의 위에 작성되고 템플릿의 각 필드의 에러를 찾는다.

`{{ form.name_field_errors }}`는 form 오류 목록이 표시되고 ul 태그의 리스트로 렌더링 된다.

```html
<ul class="errorlist">
    <li>Sender is required.</li>
</ul>
```

`errorlist`라는 CSS 클래스가 있어 스타일을 지정할 수 있다. 오류 표시를 추가로 사용자지정하려면 에러를 순회한다.

```django
{% if form.subject.errors %}
  <ol>
    {% for error in form.subject.errors %}
      <li><strong>{{ error|escape }}</strong></li>
    {% endfor %}
  </ol>
{% endif %}
```

Non-field 오류들(form.as_p() 와 같은 헬퍼를 사용할 때 폼 맨 위에 렌더링되는 숨겨진 필드 오류 및 / 비공개 필드 오류)는 필드별 오류와 구별할 수 있도록 추가 비 클래스 필드로 렌더링된다. 예를들어 `{{ form.non_field_errors }}`는 다음과 같다.

```django
<ul class="errorlist nonfield">
  <li>Generic validation error</li>
</ul>
```

### Looping over the form's fields

각 form 필드에 동일한 HTML을 적용하는 경우 `{% for %}` 루프를 사용하여 차례대로 각 필드를 반복하여 중복을 줄일 수 있다.

```django
{% for field in form %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
        {% if field.help_text %}
        <p class="help">{{ field.help_text|safe }}</p>
        {% endif %}
    </div>
{% endfor %}
```

유용한 `{{ field }}`에 포함된 속성

**{{ field.label }}**
필드의 label

**{{ field.label_tag }}**
필드의 label이 적절한 `<label>` 태그에 래핑된다. 여기에는 form 의 `label_suffix`가 포함된다. 예를 들어 기본 `label_suffix`는 콜론이다.

**{{ field.id_for_label }}**
필드에 사용할 ID, 레이블을 수동으로 구성하는 경우 `label_tag` 대신 이 레이블을 사용한다. 예를 들어 인라인 자바 스크립트가 있고 필드의 ID를 하드코딩하지 않으려는 경우에도 유용하다.

**{{ field.value }}**
필드의 값이다.

**{{ field.html_name }}**
input 요소의 name 필드에 이름이다. form prefix가 설정되어있으면 이를 고려한다.

**{{ field.help_text }}**
필드와 관련된 도움말 텍스트

**{{ field.errors }}**
이 필드에 해당하는 유효성 검사 오류가 포함된 `<ul class="errorlist">`를 출력한다. 오류에 대한 `{% for error in field.errors %}` 루프를 사용하여 오류 표시를 사용자 정의할 수 있다. 이 경우 루프의 각 객체는 오류 메세지가 포함된 문자열이다.

**{{ field.is_hidden }}**
이 특성은 form 필드가 숨겨진 필드면 `True`이고 그렇지 않으면 `False`이다. 템플릿 변수로 특히 유용하지는 않지만 다음과 같은 조건부 테스트에 유용할 수 있다.

```django
{% if field.is_hidden %}
   {# Do something special #}
{% endif %}
```

**{{ field.field }}**
이 `BoundField`가 래핑하는 form 클래스의 `Field`인스턴스이다. 이 속성을 사용하여 필드 속성에 액세스 할 수 있다. `{{ char_field.field.max_length }}`

#### Looping over hidden and visible fields

Django의 기본 form 레이아웃에 의존하는 것과는 달리 템플릿에 양식을 수동으로 배치하는 경우 `<input tpye="hidden">` 필드를 숨겨진 필드와 다르게 처리해야한다. 예를들어 숨겨진 필드에 아무것도 표시되지 않으므로 오류 메세지를 일반 필드 옆에 배치하면 사용자에게 혼동을 줄 수 있으므로 해당 필드의 오류를 다르게 처리해야한다.

Django는 `hidden_fileds()` 및 `visible_fields()`와 같이 숨김 필드와 표시 필드를 독립적으로 반복할 수 있는 두가지 메서드를 제공한다. 다음은 이 두가지 방법을 사용하는 이전 예제를 수정 한 것이다.

```django
{# Include the hidden fields #}
{% for hidden in form.hidden_fields %}
  {{ hidden }}
{% endfor %}
{# Include the visible fields #}
{% for field in form.visible_fields %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
    </div>
{% endfor %}
```

이 예제는 숨겨진 필드의 오류를 처리하지 않는다. (?), 그러나 form 오류에 대한 오류 표시도 쉽게 삽입할 수 있다.

### Reusable form templates

사이트에서 여러 위치의 form에 동일한 렌더링 논리를 사용하는 경우 양식의 루프를 독립형 템플릿에 저장하고 include 태그를 사용하여 다른 템플릿에서 재사용하여 중복을 줄일 수 있다.

```django
# In your form template:
{% include "form_snippet.html" %}

# In form_snippet.html:
{% for field in form %}
    <div class="fieldWrapper">
        {{ field.errors }}
        {{ field.label_tag }} {{ field }}
    </div>
{% endfor %}
```

템플릿에 전달 된 양식 객체가 컨텍스트 내에서 다른 이름을 갖는 경우 include 태그의 with 인수를 사용하여 별칭을 지정할 수 있다.

```django
{% include "form_snippet.html" with form=comment_form %}
```

자주 이런 일을하는 경우 `inclusion tag`를 만드는 것이 좋다.
