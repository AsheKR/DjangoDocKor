# Models

  모델은 데이터에 대한 정보를 나타내는 최종 소스이다. 데이터의 필수 필드와, 함수를 포함한다.
  일반적으로 각각의 모델은 데이터베이스의 테이블에 매핑된다.

  기본사항 :
    - 각각의 모델은 `django.db.models.Model`의 서브클래스이다.
    - 모델의 각 속성은 데이터베이스의 필드를 나타낸다.
    - 장고는 데이터베이스 액세스 API를 제공한다.

## Quick Example

이 샘플 모델은 `first_name` 과 `last_name`을 가진 `Person`을 정의한다.

```python
from django.db import models

class Person(models.Model):
  first_name = models.CharField(max_length=30)
  last_name = models.CharField(max_length=30)
```  

`first_name`과 `last_name`은 모델의 필드이다. 각각의 필드는 클래스의 속성을 나타내며, 데이터베이스의 컬럼에 매핑된다.

위의 `Person` 모델은 아래와 같은 데이터 테이블을 만든다.

```sql
CRAETE TABLE myapp_person(
  'id' serial NOT NULL PRIMARY KEY,
  'first_name' varchar(30) NOT NULL,
  'last_name' varchar(30) NOT NULL
);
```

몇가지 기술적 정보:
  - 테이블 이름은 기본적으로 `<appname>_<classname__lowercase>`로 생성되고, 재정의 가능하다.
  - `id` 필드는 자동으로 추가되며, 오버라이드 할 수 있다.
  - 이 예제의 `CREATE TABLE` SQL은 `PostgreSQL` 문법을 사용하지만, 장고는 지정된 데이터베이스에 맞는 SQL을 사용한다.

## Using models

모델을 정의하면 장고에게 이 모델을 사용할 것을 알려야한다. `settings.py`에 `INSTALLED_APPS` 설정에 `models.py`를 포함하고 있는 APP 이름(모듈의 이름)을 추가 한다.
```python
INSTALLED_APPS = [
  #...
  'myapp',
  #...
]
```
`INSTALLED_APPS`에 모듈(APP)을 추가했다면 그 하위에 있는 `models.py`의 내용을 가져오게 된다.
새 애플리케이션을 `INSTALLED_APPS`에 추가했다면, `manage.py migrate` 명령을 실행한다.
`manage.py migrate` 명령어는 현재 `INSTALLED_APPS`에 존재하는 `migrations`들을 찾아 데이터베이스에 적용시켜주는 역할을 한다.
선택적으로 `manage.py makemigrations`를 사용해 먼저 마이그레이션을 만들어 줄 수 있다.
`INSTALLED_APPS`안에 존재하는 애플리케이션 중 모델 내부에 변경사항이 있다면 `manage.py makemigrations`를 해줘 선행적으로 마이그레이션 파일을 생성해야 `manage.py migrate` 명령으로 데이터베이스에 적용시킬 수 있다.

## Fields

모델에서 가장 중요한 부분이고, 반드시 필요한 부분이다. 필드는 데이터베이스의 컬럼을 정의한다. 또한 필드는 클래스 속성으로 사용된다.
clean, save, dete 같은 `models API` 와 중복되지 않도록 해야한다.

```python
from django.db import models

class Musician(models.Model):
  first_name = models.CharField(max_length=50)
  last_name = models.CharField(max_length=50)
  instrument = models.CharField(max_length=100)

class Album(models.Model):
  artist = models.ForeignKey(Musician, on_delete=models.CASCADE)
  name = models.CharField(max_length=100)
  release_date = models.DateField()
  num_stars = models.IntegerField()
```

### Field types

각각의 필드는 적절한 필드 클래스의 인스턴스여야 한다.
필드 클래스는 아래의 몇가지 사항을 정의한다.

  - 데이터베이스 컬럼의 데이터 형
  - form field를 렌더링 할 때 기본 HTML 위젯
  - Django admin에서 자동으로 만들어지는 form의 검증 형태

장고는 매우 다양한 내장 필드 타입을 제공한다. 또한 장고에서 제공하지 않는 자신만의 필드를 쉽게 생성할 수 있다.

---

### Field options

각 필드는 고유 인수를 가진다. 예를 들어, `CharField`는 `max_length` 인수를 반드시 가져야하며, 데이터베이스에서 VARCHAR 필드의 사이즈를 지정한다.

모든 필드에 사용가능한 공통 인수도 있다. 이들은 모두 선택사항이다.

**null**
`True`일 경우, 데이터베이스에 NULL을 허용한다. 기본값은 `False`
**blank**
`True`일 경우, 필드에서 빈 값을 허용한다. 기본값은 `False`

**null**과 **blank**는 다르다.
**null**은 데이터베이스에 NULL을 허용하는 것이고, **blank**는 사용자 입력 폼에서 빈 값을 허용하지만, 데이터베이스에서는 NULL을 허용하지 않는다.
**blank**는 `form validation`으로 검사하여 폼에서 공백인가를 검증하고, **blank=False**인경우 폼에서 해당 필드는 반드시 채워져야한다.

**choices**
반복 가능한(리스트 또는 튜플) 튜플의 묶음을 선택 목록으로 사용한다. 이 인수가 주어지면, 기본 폼 위젯은 SELECT BOX로 대체되어 선택 값을 제한한다.

```python
YEAR_IN_SCHOOL_CHOICES = (
  ('FR', 'Freshman'),
  ('SO', 'sophomore'),
  ('JR', 'Junior'),
  ('SR', 'Senior'),
  ('GR', 'Graduate'),
)
```

각 튜플의 첫 요소는 데이터베이스에 저장되는 값이고, 두 번째 요소는 위젯에 표시되는 값이다.

```python
from django.db import models

class Person(models.Model):
  SHIRT_SIZES = (
    ('S', 'Small'),
    ('M', 'Medium'),
    ('L', 'Large'),
  )
  name = models.CharField(max_length=60)
  shirt_size = models.CharField(max_length=1, choices=SHIRT_SIZES)
```

모델 인스턴스에서는 표시되는 값을 액세스 하기위해 `get_FOO_display()` 함수를 사용한다. 사용법은 다음과 같다.

```python
>>> from fields.models import Person
>>> Person.objects.all()
<QuerySet [<Person: Person object (1)>]>
>>> person = Person.objects.get(pk=1)
>>> person.shirt_size
'M'
>>> person.get_shirt_size_display()
'Medium'
```

**default**
필드에 기본값으로 설정된다.

**help_text**
폼 위젯에서 추가적으로 보여줄 도움말 텍스트이다. 폼을 사용하지 않아도 문서화에 많은 도움이 된다.

**primary_key**
`True` 일 경우, 해당 필드는 모델의 `primary_key`로 사용된다.

어떤 필드에도 `primary_key=True`로 설정하지 않는다면 장고는 자동으로 `IntegerField`를 생성해 `primary_key`로 사용한다.
그러므로 반드시 `primary_key=True`를 어떤 필드에 추가할 필요는 없다.

`primary_key`는 읽기전용 필드다. 기존 개체의 `primary_key` 값을 변경한 후 저장하면, 이전 객체와는 별개의 새로운 객체가 생성된다.

**unique**
`True`일 경우, 이 필드의 값을 테이블 전체에서 고유해야한다.

---

### Automatic primary key fields
기본적으로 장고는 각 모델에 다음 필드를 제공한다.
```python
id = models.AutoField(primary_key=True)
```
`auto-increment primary key`이다.
임의의 `primary_key`를 지정하고싶다면, 필드 중 하나의 `primary_key=True`를 지정하면 된다.
장고는 `primary_key=True` 필드가 있을 경우 `id` 컬럼을 추가하지 않는다.

각 모델은 정확히 하나 혹은 0개의 `primary_key=True` 필드를 가져야한다.

---

### Verbos field names

`ForeignKey`, `ManyToManyField`, `OneToOneField`를 제외한 모든 필드에서, Verbos field name은 첫 인수이다.
만약 Verbose name이 주어지지 않은 경우, 장고는 자동으로 해당 필드의 이름을 이용하여 Verbos name을 만들어 사용한다.

아래의 예에서, verbos name은 `person's first name`이다.
```python
first_name = models.CharField("person's first name", max_length=30)
```

`ForeignKey`, `ManyToManyField`, `OneToOneField`는 첫번째 인자로 모델 클래스를 가져야하기 때문에, `verbose_name` 인수를 사용한다.
```python
poll = models.ForeignKey(
  Poll,
  on_delete=models.CASCADE,
  verbos_name = 'the related poll',
)
sites = models.ManyToManyField(Site, verbos_name="list of sites")
place = models.OneToOneField(
  Place,
  on_delete=models.CASCADE,
  verbos_name='related place',
)
```
첫 글자는 대문자로 지정하지 않는다. 장고는 자동으로 첫 번째 글자를 대문자화 한다.

## Relationships

관계형 데이터베이스의 강력함은 테이블 관계에 있다. 장고는 데이터베이스의 관계 유형중 가장 일반적인 3가지를 제공한다.
`Many-To-Many`, `Many-To-One`, `OneToOne`

### Many-to-one relationships

`Many-To-one` 관계를 정의하기위해, `django.db.models.ForeignKey`를 사용한다. 다른 필드 타입과 비슷하게, 모델 클래스 속성으로 정의한다.
`ForeignKey`는 관계를 정의할 모델 클래스를 인수로 가져야한다.

예를 들어, `Car` 모델은 `Manufacturer`를 `ForeignKey`로 가질 때, `Manufacturer`는 여러개의 `Car`를 가질 수 있지만 `Car`는 하나의 `Manufacturer`를 가진다.

```python
from django.db import models

class Manufacturer(models.Model):
  #...
  pass

class Car(models.Model):
  manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
  name = models.CharField(max_length=50)
```

또한 재귀관계( 자기 자신과 다대 일 관계)와 아직 정의되지 않은 모델(relationships to models not yet defined)과 관계를 만들 수 있다.
`ForeignKey`필드의 이름이 모델 이름의 소문자인것을 권장하지만 필수는 아니다.


```python
# Manufacturer 모델 인스턴스 생성
>>> kia = Manufacturer.objects.create(name='기아')
>>> hyundai = Manufacturer.objects.create(name='현대')

# Car 모델 인스턴스 생성
>>> Car.objects.create(
...   # ForeignKey에는 정의때 사용한 클래스 모델 인스턴스를 인수로 가져야한다.
...   manufacturer=hyundai,
...   name='아반떼'
... )
# List Comprehension으로 여러개를 한번에 생성할 수 있다.
>>> [Car.objects.create(name=name) for name in 'k3 k5 k7'.split()]
```

재귀관계의 예

```python
# models.py

# 사람이라는 모델이 있고, 이름, 강사의 필드를 갖는다.
# 강사 필드는 User 인스턴스를 가리키고, 강사일 시 이 필드는 비워두어도 되므로 blank, null을 True로 두었다.
class User(models.Model):
  name = models.CharField(max_length=30)
  instructor = models.ForeignKey(
    'self',
    blank=True,
    null=True,
    on_delete=models.SET_NULL,
  )

  def __str__(self):
    return self.name
```

```python
# User 인스턴스 생성
>>> User.objects.create(name='강사님')
>>> User.objects.create(name='배우미1')
>>> User.objects.create(name='배우미2')
>>> User.objects.create(name='배우미3')
# 이름이 '강사님'인것을 제외한 나머지 모든것의 '강사(instructor)' 필드를 '강사님'으로 채우는 것
>>> User.objects.exclude(name='강사님').update(instructor=User.objects.get(name='강사님'))

>>> user = User.objects.get(name='강사님')
>>> user
<User: 강사님>
# instructor 필드로 나를 참조하고 있는 모든 객체를 `.<class이름의 소문자화>_set` 으로 가져올 수 있다. (역참조)
# 지금 여기서 user_set은 자동으로 생성된 것이라 무엇을 뜻하는지 확실히 알 수 없는데
# 이 이름을 바꾸어줄 수 있는다. 필드 옵션으로 `related_name=<값>`을 넣어주면 그 값으로 역참조가 가능하다.
# 이 다음 아래 예제에서 설명한다.
>>> user.user_set.all()
<QuerySet [<User: 배우미1>, <User: 배우미2>, <User: 배우미3>]>
```

```python
class User(models.Model):
  #...
  instructor = models.ForeignKey(
    #...
    related_name='students',
  )

# ---------

>>> user
<User: 강사님>
>>> user.students.all()
<QuerySet [<User: 배우미1>, <User: 배우미2>, <User: 배우미3>]>
```
