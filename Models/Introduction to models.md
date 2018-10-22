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

### Many-To-Many Relationships

many-to-many 관계 정의를 위해서는, `ManyToManyField`를 사용한다.
`ManyToManyField`는 관계를 정의할 모델 클래스를 인수로 가져야한다.

예를 들어 `Pizza`모델은 여러개의 `Topping`객체를 가질 수 있다. `Topping`은 여러개의 `Pizza` 위에 올라갈 수 있으며, `Pizza` 역시 여러개의 `Topping`을 가질 수 있다.

```python
from django.db import models


class Topping(models.Model):
  name = models.CharField(max_length=10)

  def __str__(self):
    return self.name


class Pizza(models.Model):
  name = models.CharField(max_length=10)
  toppings = models.ManyToManyField(Topping)

  def __str__(self):
    return self.name
```

`ManyToManyField`는 어느쪽의 모델에 있어도 상관없지만 단 하나의 모델에만 존재해야한다. 그래서 의미상으로 맞는 쪽으로 필드를 두어야한다. 피자가 많은 토핑을 갖는지, 토핑이 많은 피자를 갖는지. 이 필드는 데이터베이스의 새 테이블을 만들어 관계를 정의한다.

새 테이블의 이름은 `<appname>_<classname__lowercase>_<ManyToManyField>` 이름으로 만들어지게 된다.
즉, `<appname>_pizza_toppings`로 만들어지게 된다. 해당 테이블의 row에는 중간 테이블의 PK인 `id`, pizza 테이블의 ForeignKey인 `pizza_id`, topping 테이블의 ForeignKey인 `topping_id`를 기본적으로 가지게 된다.

여기서도 `recursive relationships`와 `relationships to models not yet defined`를 사용할 수 있다.
일반적으로 필드 명은 관계된 모델 객체의 복수형을 추천하지만, 필수는 아니다.

```python
>>> 치즈피자, 불고기피자 = [Pizza.objects.create(name=name) for name in '치즈 불고기'.split()]
>>> 치즈, 불고기, 피망 = [Topping.objects.create(name=name) for name in '치즈 불고기 피망'.split()]
# ManyToMany 에서는 save를 호출하지 않고 add 만하더라도 자동으로 데이터베이스에 추가가된다.
# 아래 내용은 하나의 열의 데이터를 생성한다.
>>> 치즈피자.toppings.add(치즈)
# 아래 내용은 세개의 열의 데이터를 생성한다.
>>> 불고기피자.toppings.add(치즈, 불고기, 피망)
# 정방향 참조시에는 필드 이름을 사용한다.
>>> 치즈피자.toppings.all()
<QuerySet [<Topping: 치즈>]>
>>> 불고기피자.toppings.all()
<QuerySet [<Topping: 치즈>, <Topping: 불고기>, <Topping: 피망>]>
# 역방향 참조시 기본적으로 `<classname__lowercase>_set`으로 참조하게 된다.
>>> 치즈.pizza_set.all()
<QuerySet [<Topping: 치즈피자>, <Topping: 불고기피자>]>
```

---

### Extra fields on many-to-many Relationships

이외에도, 피자 토핑 연결 대한 추가적인 정보(토핑이 추가된 시기, 해당 피자에 얼마나 토핑이 올라가는가)를 해결하기 위해서는 `intermediate`(중간)모델을 사용한다.

#### Intermediate

```python
from django.db import models

class Person(models.Model):
  name = models.CharField(max_length=120)

  def __str__(self):
    return self.name


class Group(models.Model):
  name = models.CharField(max_length=120)
  members = models.ManyToManyField(Person, through='Membership')

  def __str__(self):
    return self.name


class Membership(models.Model):
  person = models.ForeignKey(Person, on_delete=models.CASCADE)
  group = models.ForeignKey(Group, on_delete=models.CASCADE)
  date_joined = models.DateField()
  invite_reason = models.CharField(max_length=60)
```

```python
>>> ringo = Person.objects.create(name="Ringo Starr")
>>> paul = Person.objects.create(name="Paul McCartney")
>>> beatles = Group.objects.create(name="The Beatles")
>>> m1 = Membership(person=ringo, group=beatles, date_joined(1962, 8, 16), invite_reason="Needed a new drummer")
>>> m2 = Membership(person=paul, group=beatles, date_joined(1960, 8, 1), invite_reason="Wanted to form a band.")
>>> beatles.members.all()
<QuerySet [<Person: Ringo Starr>, <Person: Paul McCartney>]>
>>> john = Person.obejcts.create(name='john')
>>> beatles.members.add(john)
! ERROR 발생. # 중간모델을 사용하는 경우 추가 필드가 존재하므로 add 사용이 불가능하다. (추가 필드가 없더라도 add 사용이 불가하다!) 이외에도 일반적 many-to-many 필드와는 다르게 add, create, set 등의 직접 할당 명령어를 사용할 수 없다.
# 그러므로 중간모델을 작성하였으면 위와같이 중간모델을 직접 생성하는 수 밖에 없다.
```

#### Recursive

Many-To-Many 필드에서 한 모델 인스턴스가 다른 여러개의 모델 인스턴스를 가리키고, 다른 여러개의 인스턴스가 나의 인스턴스를 가리킬 때가 있다.예를 들어 팔로우/팔로워 관계가 그러하다.

```python
from django.db import models


class FacebookUser(models.Model):
  name = models.CharField(max_length=50)
  # 재귀적 모델은 'self' 라는 이름을 사용해서 자기 자신을 가리킨다.
  friends = models.ManyToManyField('self')

  def __str__(self):
    return self.name
```

테이블 생성 규칙은 위와 같다. `<appname>_<classname__lowercase>_<ManyToManyField>` 즉, `<appname>_facebookuser_friends` 로 테이블이 생성된다. 다른점은 안의 열의 이름이 다른데, 어떤 모델이 어떤 모델을 가리킨다를 표현하기 위해 위에서 보았던 `<classname__lowercase>_id`가 아닌 `from_facebookuser_id`, `to_facebookuser_id` 라는 열을 생성한다.


```python
>>> u1,u2,u3 = [FacebookUser.objects.create(name=name) for name in '박보영 아이유 수지'.split()]
>>> u1.friends.add(u3)
# 기본적으로 재귀관계에서는 대칭적으로 관계가 생성된다.
# 두개의 데이터 열을 생성한다. 하나는 u1에서 u3로 friends를 걸고, 나머지 하나는 u3에서 u1으로 friends를 거는 관계를 생성한다.
>> u1.friends.add(u2)
# 마찬가지로 2개의 데이터 열을 생성한다.
```

#### Recursive and Intermediate

#### Recursive and Symmetrical_False

팔로워, 팔로우 같은 기능의 경우 상대방이 팔로우를 한다고해서 내가 상대방을 팔로우하지는 않는다. 대칭적 관계 생성을 막기 위한 옵션이 `symmetrical=False`이다.

```python
from django.db import models


class InstagramUser(models.Model):
  name = models.CharField(max_length=50)
  # 내가 팔로우하는 유저 목록을 following이라 하고
  # 나를 팔로우하는(역참조) 유저 목록을 followers 라고 하기때문에 따로 생성했다.
  # 이렇게 비 대칭적인 구조를 가질 때 related_name을 지정해줘서 의미를 명확히 해주는것이 좋다.
  following = models.ManyToManyField(
    'self',
    symmetrical=False,
    related_name = 'followers',
  )
```

```python
>>> u1,u2,u3,u4 = [FacebookUser.objects.create(name=name) for name in '박보영 아이유 수지 민아'.split()]
# u1 <- u2,u3,u4 팔로잉 관계를 만들어본다.
>>> u2.following.add(u1)
>>> u3.following.add(u1)
>>> u4.following.add(u1)
# 내가 팔로잉한 사람을 출력
>>> u2.following.all()
<QuerySet [<InstagramUser: 박보영>]>
# 나를 팔로우하고 있는 사람들을 출력
>>> u1.followers.all()
<QuerySet [<InstagramUser: 아이유>, <InstagramUser: 수지>, <InstagramUser: 민아>]>
```

#### Recursive and Symmetrical_False and Intermediate

이번엔 더 자세한 관계 기록을 하기 위해 중간 모델을 생성해본다.

```python
from django.db import models


class TwitterUser(models.Model):
  name = models.CharField(max_length=50)
  relation_users = models.ManyToManyField(
    'self',
    # related_name에서 +는 필요 없다라는 것이다. 역 참조는 더이상 불가능해진다.
    # twitteruser_set을 사용하였을 때 from_user에서 가져와야할지, to_user에서 가져와야할지 모르게 된다.
    related_name='+',
    symmetrical=False,
    through='Relation'
  )

class Relation(models.Model):
  CHOICES_RELATION_TYPE = (
    ('f', 'Follow'),
    ('b', 'Block'),
  )
  from_user = models.ForeignKey(
    TwitterUser,
    on_delete=models.CASCADE,
    related_name='from_user_relations',
    related_query_name='from_user_relation',
  )
  to_user = models.ForeignKey(
    TwitterUser,
    on_delete=models.CASCADE,
    related_name='to_user_relations',
    related_query_name='to_user_relation',
  )
  relation_type = models.CharField(max_length=1, choices=CHOICES_RELATION_TYPE)
  created_at = models.DateTimeField(auto_now_add=True)

  class Meta:
    # 이 옵션을 사용한 이유는 중복을 방지하기 위함이다.
    # 팔로우한 상태에서 블락이라는 중복값은 가지면 안되기때문에, 이 옵션을 사용하여 두개를 하나로 묶어 필드 중복값을 허용하지 않을 수 있다.
    unique_togeter = (
      ('from_user', 'to_user'),
    )
```

```python
>>> u1,u2,u3,u4 = [TwitterUser.objects.create(name=name) for name in '수지 민아 박보영 아이유'.split()]
# u2가 u1을 팔로우
>>> Relation.objects.create(from_user=u2, to_user=u1, relation_type='f')
# u3가 u1을 팔로우
>>> Relation.objects.create(from_user=u3, to_user=u1, relation_type='f')
# u4가 u1을 팔로우
>>> Relation.objects.create(from_user=u4, to_user=u1, relation_type='b')
# 이렇게 역참조로 가져오게될 때 Relation 매니저를 사용하여 객체를 가져오는 문제가 발생한다.
>>> u1.to_user_relations.all()
<QuerySet [<Relation: 민아 to 수지 (follow)>, <Relation: 박보영 to 수지 (follow)>, <Relation: 아이유 to 수지 (block)>]>
# 그렇기 때문에 filter에 related_query_name으로 TwitterUser를 가져올 수 있다.
>>> TwitterUser.objects.filter(to_user_relation__to_user=u1, to_user_relation__relation_type='f')
<QuerySet [<TwitterUser: 수지>, <TwitterUser: 수지>]>
>>> TwitterUser.objects.filter(from_user_relation__to_user=u1, from_user_relation__relation_type='f')
<QuerySet [<TwitterUser: 민아>, <TwitterUser: 박보영>]>
```

---

위의 filer 내용은 상당히 헷갈려서 작성한다.
두가지 방법의 SQL문, 결과를 비교해서 설명한다.

첫번째는 `to_user_relation`을 사용하는 것
```python
TwitterUser.objects.filter(to_user_relation__to_user=u1)
```
이 내용은 깔끔히 정리하면 다음 SQL로 만들어진다.
```sql
SELECT * FROM TwitterUser
INNER JOIN Relation ON (TwiiterUser.id=Relation.to_user_id)
WHERE from_user_id=u1
```
WHERE절을 제외한 결과는 다음과 같다.
id | name | id | relation_type | created_at | from_user_id | to_user_id |
---|------|----|---------------|-----------|--------------|------------|
1 |  수지 | 1 |  'f'           |          |              2 |            1 |
1 |  수지 |  2 |  'f'           |			      |            3 |            1 |
1 |  수지 |  3 |  'b'           |           |            4 |            1 |
두번째는 `from_user_relation`을 사용하는 것
```python
TwitterUser.objects.filter(from_user_relation__to_user=u1)
```
```sql
SELECT * FROM TwitterUser
INNER JOIN Relation ON (TwiiterUser.id=Relation.from_user_id)
WHERE from_user_id=u1
```
WHERE절을 제외한 결과는 다음과 같다.
id | name | id | relation_type | created_at | from_user_id | to_user_id |
---|------|----|---------------|-----------|--------------|------------|
2 |  민아 | 1 |  'f'           |          |               2 |            1 |
3 | 박보영 |  2 |  'f'           |			      |            3 |            1 |
4 | 아이유 |  3 |  'b'           |           |            4 |            1 |

SQL에서 다른점을 찾아보자면 JOIN할 때 어느 필드에 조인할것인지가 달라진다.
`TwitterUser.id=Relation.to_user_id`, `TwitterUser.id=Relation.from_user_id`

1. `to_user_relation`으로 filter를 걸면
2. 데이터베이스의 `to_user_id`필드의 값에 해당하는 TwitterUser의 정보를 가져온다.
3. 그렇기 때문에 수지수지수지 라는 결과가 나오게 되는것이다!

그리고 이 이후에 WHERE 조건이 걸리게되어 해당 값이 나오게 된다.

---
