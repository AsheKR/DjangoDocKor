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

**null** 과 **blank** 는 다르다.
**null** 은 데이터베이스에 NULL을 허용하는 것이고, **blank** 는 사용자 입력 폼에서 빈 값을 허용하지만, 데이터베이스에서는 NULL을 허용하지 않는다.
**blank** 는 `form validation`으로 검사하여 폼에서 공백인가를 검증하고, **blank=False** 인경우 폼에서 해당 필드는 반드시 채워져야한다.

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

### One-to-one relationships

one-to-one 관계를 정의하려면, `OneToOneField`를 이용하면 된다. 다른 관계를 필드와 마찬가지로 모델 클래스의 어트리뷰트로 선언하면 된다.

일대일 관계는 다른 모델을 확장하여 새로운 모델을 만드는 경우 유용하게 사용할 수 있다.

예를 들어, 가게(Places)정보가 담긴 데이터베이스를 구축한다고 한다. 아마 데이터베이스에 주소, 전화번호 등의 정보가 들어간다. 그런데 맛집 데이터베이스를 추가적으로 구축할 경우, 새로 Restaurant모델을 만들 수도 있지만, 반복을 피하기 위해 Restaurant모델 Place모델만 `OneToOneField`로 선언한다.

`ForeignKeyField`와 마찬가지로 `recursive`나 아직 선언되지 않은 모델에 대해서도 관계를 가질 수 있다.

`OneToOneField`는 `parent_link`라는 옵션을 제공한다.
이 옵션은 구체적인 모델에서 상속한 모델에서 사용되는 경우 이 필드는 서브 클래싱하여 일반적으로 암시적으로 생성되는 OneToOneField가 아니라 부모 클래스에 대한 링크로 다시 사용해야 함을 나타낸다.

`OneToOneField`는 자동으로 모델의 기본키가 되었지만, 지금은 아니다.(`primary_key` 인수를 통해 직접 지정해줄수는 있다.) 그러므로 단일 모델에 여러 `OneToOneField` 필드를 가질 수 있다.

```python
class Place(models.Model)
  name = models.CharField(max_length=50)
  address = models.CharField(max_length=80)

class Restaurant(models.Model):
  place = models.OneToOneField(
    Place,
    on_delete=models.CASCADE,
    primary_key=True,
  )

  serves_hot_dogs = models.BooleanField(default=False)
  serves_pizza = models.BooleanField(default=False)
```

`OneToOneField`의 역참조시 `_set`이 아닌 클래스명의 소문자를 사용한다.
또한 참조, 역참조시 `Queryset`이 아닌 하나의 객체만이 나타난다.

`OneToOne` 필드 참조시 해당 객체가 없으면 `ObjectDoesNotEixst`오류가 발생한다.

```python
try:
  Place.objects.first().restaurant
except:
  print('no object')
```

예외 처리하지않고 하기 위해서 `hasattr`메서드를 사용한다.

해당 Place 인스턴스에 `restaurant`속성이 있는지 확인하기.
```python
hasattr(Place.objects.first(), 'restaurant')
```

### Models across files

다른 앱에 선언된 모델과 관계를 가질 수 있다. 그렇게 하려면, 다른 앱의 모델을 import해서 아래와 같이 관계 필드를 선언하면 된다.

```python
from django.db import models
from .models import ZipCode

class Restaurant(models.Model):
  zip_code = models.ForeignKey(
    ZipCode,
    on_delete=models.SET_NULL,
    blank=True,
    null=True,
  )
````

### Field name restrictions

장고 모델 필드명에는 2가지 제약을 두고있다.
1. 파이썬 예약어는 필드명으로 사용할 수 없다.
2. 필드 이름에 밑줄 두개를 연속으로 사용할 수 없다. 이는 장고에서 특별한 문법으로 사용되기때문이다.

데이터베이스 컬럼명에 밑줄을 두개 넣어야하는 상황이면, db_column 옵션을 사용해 제약을 우회할 수 있다.

### Custom field types

장고에서 제공하는 필드 타입 중 적절한 것이 없거나, 특정 데이터베이스에서 제공하는 특별한 타입을 사용하고 싶으면, 필드를 직접 만들어 사용할 수 있다.

### Meta options

아래와 같이 모델 클래스 내부에 Meta 라는 이름의 클래스를 선언해서 모델에 메타데이터를 추가할 수 있다.

```python
from django.db import models

class Ox(models.Model)
  horn_length = models.IntegerField()

  class Meta:
    ordering = ["horn_length"]
    verbose_name_plural = "oxen"
```

모델 메타데이터는 필드의 옵션과 달리 모델 단위의 옵션을 가진다. 예를 들어 정렬옵션(ordering), 데이터베이스 테이블 이름(db_table), 또는 읽기 좋은 이름(verbose_name)이나 복수(verbos_name_plural)이름을 지정해 줄 수 있다.
모델 클래스에 Meta클래스를 반드시 선언해야하는 것은 아니며, 또한 모든 옵션을 설정해야 하는것도 아니다.

### Model attributes

#### objects

모델 클래스에 가장 중요한 속성은 `Manage`이다 `Manager`객체는 모델 클래스를 기반으로 데이터베이스에 대한 쿼리 인터페이스를 제공하며, 데이터베이스 레코드를 모델 객체로 인스턴스화 하는데 사용된다. 특별히 `Manager`를 할당하지 않으면 장고는 기본 `Manager`를 클래스 속성으로 자동 할당한다. 이 때, 속성 이름이 **objects** 이다.

`Manager` 모델 클래스를 통해 접근할 수 있으며, 모델 인스턴스(객체)를 통해서 접근할 수는 없다.

### Model methods

모델 객체(row) 단위의 기능을 구현하려면 모델 클래스에 메서드를 구현하면 된다. 테이블 단위의 기능은 Manager에 구현한다.

이러한 규칙은 비즈니스 로직 모델에서 관리하는데 있어 중요한 테크닉이다.

예를 들어, 다음과 같이 커스텀 메서드를 추가할 수 있다.

```python
from django.db import models


class Person(models.Model):
  first_name = models.CharField(max_length=50)
  last_name = models.CharField(max_length=50)
  birth_date = models.DateField()

  def baby_boomer_status(self):
    "Returns the person's baby-bommer status"
    import datetime
    if self.bith_date < datetime.date(1945, 8, 1):
      return "Pre-bommer"
    elif self.bith_date < datetime.date(1965, 1, 1):
      return "Baby bommer"
    else:
      return "Post-bommer"

  def _get_full_name(self):
    "Returns the person's full name"
    return "%s  %s" %(self.first_name, self.last_name)

  full_name = property(_get_full_name)
```

이 예제의 마지막 메서드는 `Property`이다.
> 프로퍼티는 메서드를 속성처럼 접근할 수 있게 해준다.
> "\_"로 시작하는 비공개 메서드를 정의한 후, property라는 함수로 비공개 메서드를 호출하면, 그 결과가 반환되어 full_name 란에 들어가게된다.
> 이를 줄여쓰는것이 Property getter, setter 데코레이터이다.

### Overriding predefined model methods

`model instance reference`에는 모델 클래스에 자동적으로 주어지는 메서드들이 나와있다. 이러한 메서드를 오버라이드해서 사용할수도 있다. 특히 `save()` 및 `delete()`의 작업방식을 바꾸는 경우가 많다.

```python
from django.db import models

class Blog(models.Model):
  name = models.CharField(max_length=100)
  tagline = models.TextField()

  def save(self, **args, **kwargs):
    # 수행할 내용1 ...
    super().save(*args, **kwargs) # 기존의 save() 메서드를 호출한다.
    # 수행할 내용2 ...
```

저장을 막을 수도 있다.

```python
from django.db import models

class Blog(models.Model):
  name = models.CharField(max_length=100)
  tagline = models.TextField()

  def save(self, *args, **kwargs):
    if self.name == "Yoko Ono's blog":
      return # 해당 블로그는 저장하지 않겠다!
    else:
      super().save(*args, **kwargs)
```

슈퍼 클래스 메서드 호출하는 것을 기억하는것이 중요하다.

> super().save(\*args, \*\*kwargs)

이는 객체가 데이터베이스에 저장되도록한다.
수퍼 클래스 메서드를 호출하는 것을 잊어버리면 기본 동작이 실행되지 않으며 데이터베이스에 저장하지 않는다.

또한 모델 메서드에 전달할 수 있는 인수를 전달하는 것이 중요하다. 이는 `*args`, `**kwargs` 두 변수로 전달된다.
Django는 수시로 내장 모델 메서드의 기능을 확장하여 새로운 인수를 추가한다. 메서드 정의에서 `*args`, `**kwargs`를 사용하면 코드가 추가될 때 해당 인수를 자동으로 지원받는다는 보장을 받는다.

> 오버라이드된 모델 메서드는 `bulk operations`에서는 동작하지 않는다.
> QuerySet을 사용하여 대량으로 객체를 삭제할 때 또는 계단식 삭제의 결과로 객체의 delete()메서드가 반드시 호출되지 않을 수 있다. 사용자 정의 삭제 논리를 실행하려면 `pre_delete`와 `post_delete` 시그널을 사용할 수 있다.
> 불행히 `bulk operations`에서는 `save()`메서드와 `pre_save` 및 `post_save` 시그널이 호출되지 않기 때문에 객체를 대량으로 만들거나, 업데이터 할 때 해결 방법이 없다.

### Executing custom SQL

또 다른 공통 패턴은 모델 메서드 및 모듈 수준 메서드에 사용자 지정 SQL문을 작성하는 것이다. `using raw SQL`문서를 참조하라.

## Model inheritance
`Django`의 모델 상속은 파이썬에서 일반적인 클래스 상속이 작동하는 방식과 거의 동일하게 작동하지만 반드시 따라야하는 기본 사항이 있다. 기본 클래스가 `django.db.models.Model`을 상속받아야한다.

부모 모델이 자체 데이터베이스 테이블을 가지는 모델이 될지 (Concrete Model),
또는 부모가 자식 모델에게 전달할 정보만을 가지고 있는지 여부만 결정한다 (Abstract Model).

`Django`에는 세가지 스타일의 상속을 제공한다.
1. 부모 클래스를 사용하여 각 하위 모델에 대해 일일이 입력하지 않으려는 정보를 제공하는 경우, 이 클래스는 따로 분리하여 사용하지 않으므로, 즉 데이터베이스 테이블이 생성되지 않아 추가 기본 클래스(Abstract base classes)를 사용한다.
(부모 테이블 X, 자식 테이블 O)

2. 기존 모델을 하위 클래스화(다른 애플리케이션의 모델이어도 무관)하고, 각 모델이 자체 데이터베이스 테이블을 가지기 원한다면 다중 테이블 상속(Multi table Inheritance)가 필요하다.
(부모 테이블 O, 자식 테이블 O)

3. 마지막으로 모델 필드를 변경하지 않고 모델의 파이썬 동작만 수정하려는 경우 `Proxy`모델을 사용할 수 있다.
(부모 테이블 O, 자식 테이블 X)

### Abstract base classes

추상 기본 클래스는 몇 가지 공통된 정보를 여러 다른 모델에 넣으려 할 때 유용하다. 기본 클래스를 작성하고 `Meta`에 `abstract=True`를 넣는다. 이 모델은 데이터베이스 테이블을 만드는데 사용되지 않는다. 대신 다른 모델의 기반 클래스로 사용될 때 해당 필드는 자식 클래스의 필드에 추가된다. 자식의 이름과 같은 이름 (상속받은 클래스의 이름과 같은 이름의 필드)를 가진 추상 기본 클래스의 필드를 갖는 것은 오류이며, `Django`는 이에 대해 오류를 발생시킨다.

```python
class CommonInfo(models.Model):
  name = models.CharField(max_length=100)
  age = models.PositiveIntegerField()

  class Meta:
    abstract = True

class Student(CommonInfo):
  home_group = models.CharField(max_length=5)
```

`Student` 모델에는 `name`, `age` 및 `home_group`의 세가지 필드가 있다. `CommonInfo` 모델은 `abstract base class`이기 때문에 일반 `Django`모델로 사용할 수 없다. 이 모델은 데이터베이스 테이블을 생성하지 않으며, `Manager`를 가지지 않으므로 직접 인스턴스화하거나 데이터베이스에 저장할 수 없다.

### Meta inheritance

추상 기본 클래스가 생성되면 Django는 기본 클래스에서 선언한 `Meta` 내부 클래스를 속성으로 사용할 수 있게한다. 자식 클래스가 자신의 메타 클래스를 선언하지 않으면 부모 클래스의 메타를 상속받는다. 자식이 부모의 `Meta`클래스를 확장하려고하면 해당 클래스를 서브클래스로 사용할 수 있다.

```python
from django.db import models


class CommonInfo(models.Model):
  #...
  class Meta:
    abstract = True
    ordering = ['name']


class Student(CommonInfo):
  #...
  class Meta(CommonInfo.Meta):
    db_table = 'student_info'
```

Django는 추상 기본 클래스의 `Meta`클래스를 조정한다. `Meta`속성을 적용하기 전 `abstract`속성의 값을 `False`로 설정한다. 즉, 추상 기본 클래스의 자식은 자동으로 추상 클래스가 되지 않는다. 물론 다른 추상 클래스에서 상속받은 추상 클래스를 만들수도 있다. 매번 `abstract=True`를 명시적으로 설정하는것을 기억하면 된다.

일부 속성은 추상 기본 클래스의 `Meta`클래스에 포함하는 것이 옳지 않다.  예를들어 `db_talbe`의 경우 모든 자식 클래스(자신의 메타를 지정하지 않은 클래스)가 동일한 데이터베이스 테이블을 사용한다는 것을 의미하기때문에, 이는 원하지 않는 동작을 초래한다.

### Be careful with related_name and related_query_name
`ForignKey` 또는 `ManyToManyField`에서 `related_name`과 `related_query_name`을 사용하는 경우 고유한 reverse name과 query name을 항상 지정해야한다. 이 필드들(ManyToManyField, ForignKey)를 가진 추상 기본 클래스를 상속받은 경우, 매번 해당 속성(related_name 또는 related_query_name)에 대해 정확히 동일한 값이 사용되므로 일반적으로 문제가 발생한다.

이 문제를 해결하기위해 값의 일부에 `%(app_label)s` 또는 `%(class)s`를 지원한다.
- `%(class)s`는 필드가 사용되는 하위 클래스 이름의 lower_cased 이름으로 대체된다.
- `%(app_label)s`는 하위 클래스가 포함된 어플리케이션 이름의 lower_cased 이름으로 대체된다.

설치된 각 응용 프로그램 이름은 고유해야하며 각 응용 프로그램 내 모델 클래스의 이름도 고유해야하므로 결과 이름이 달라지게된다.

```python
# common.models
from django.db import models


class Base(models.Model):
  m2m = models.ManyToManyField(
    OtherModel,
    related_name="%(app_name)s_%(class)s_related"
    related_query_name="%(app_name)s_%(class)ss"
  )
  class Meta:
    abstract = True

class ChildA(Base):
  pass

class ChildB(Base):
  pass
```

```python
# rare.models
from common.models import Base


class ChildB(Base):
  pass
```

`common.ChildA.m2m` 필드의 `reverse_name`은 `common_childa_related`이고 `reverse_query_name`은 `common_childas`이다.
`common.ChildB.m2m` 필드의 `reverse_name`은 `common_childb_related`이고 `reverse_query_name`은 `common_childbs`이다.
`rare.ChildB.m2m` 필드의 `reverse_name`은 `rare_childb_related`이고 `reverse_query_name`은 `rare_childbs`이다.

추상 기본 클래스의 필드에 `related_name`속성을 지정하지않으면 상속 받은 자식 클래스의 기본 `reverse_name`은 필드를 직접 선언한 경우와 마찬가지로 `<classname__lowercase>_set`이 된다.

### Multi-table inheritance

Django가 지원하는 모델 상속의 두번째 유형은 계층 구조의 각 모델이 모두 각각 자신을 나타내는 모델일 때이다. 각 모델은 자체 데이터베이스 테이블에 해당하며 개별적으로 쿼리하고 생성할 수 있다. 상속 관계는 자동으로 생성된 `OneToOneField`를 통해 자식 모델과 부모간의 링크를 만든다.

```python
from django.db import models


class Place(models.Model):
  name = models.CharField(max_length=50)
  address = models.CharField(max_length=50)


class Restaurant(Place):
  serves_hot_dogs = models.BooleanField(default=False)
  serves_pizza = models.BooleanField(default=False)
```

`Place`의 모든 필드는 `Restaurant`에서 사용할 수 있지만 데이터는 다른 데이터베이스 테이블에 있다. 그래서 아래 두 명령 모두 가능하다.

```python
>>> Place.objects.filter(name="Bob's cafe")
>>> Restaurant.objects.filter(name="Bob's cafe")

# Restaurant이면서 Place가 있는 경우, 모델 이름의 소문자 버전을 사용하여 Place 객체에서 Restaurant객체를 가져올 수 있다.
Place.objects.first().restaurant
# 없는 경우에는 Restaurant.DoesNotEixst 오류가 발생한다.
```

Restaurant에서 자동으로 생성된 OneToOneField는 다음과 같은 형태를 가진다.
```python
place_ptr = models.OneToOneField(
  Place,
  on_delete=models.CASCADE,
  parent_link=True,
)
```
`Restaurant`에서 자신의 `OneToOneField`에 `parent_link=True`를 사용하여 해당 필드를 재정의할 수 있다.

### Meta and multi-table inheritance

다중 테이블 상속 상황에서 자식 클래스가 부모의 Meta클래스에서 상속받는 것은 의미가 없다. 모든 메타 옵션은 이미 상위 클래스에 적용되었고 다시 적용하면 모순된 행동만 발생한다. (기본 클래스가 자체적으로 존재하지 않는 추상 기본 클래스의 경우와 대조적이다.)

따라서 자식 모델은 부모 메타 클래스에 액세스할 수 없다. 그러나 자식이 부모로부터 동작을 상속하는 몇 가지 사항이 있다. 자식이 `ordering`특성이나 `get_latest_by`특성을 지정하지 않으면 해당 특성을 부모로부터 상속한다.

부모가 ordering 되어있고 이를 해제하려면 명시적으로 사용을 중지할 수 있다.
```python
class ChildModel(ParentModel):
  # ...
  class Meta:
    # Remove parent's ordering effect
    ordering = []
```

#### Inheritance and reverse relations

다중 테이블 상속은 암시적으로 `OneToOneField`를 사용하여 부모와 자식을 연결하기 때문에 위의 예와 같이 상위에서 하위로 이동할 수 있다. 그러나 이 경우 `related_query_name`의 값으로 `ForignKey`및`ManyToManyField`관계에 대한 기본 값을 사용한다. 이러한 관계 유형들을 부모 모델의 하위 클래스에 배치하는 경우 해당 필드 각각에 반드시 `related_query_name`속성을 지정해야한다. 이를 잊으면 Django는 유효성 검사 오류를 발생시킨다.

```python
class Supplier(Place):
  customers = models.ManyToManyField(Place)
```

PK가 OneToOneField이기 때문에 `related_name`의 중복은 발생하지 않고 `related_query_name`중복 오류가 발생하게 된다.
s
```python
Reverse query name for 'Supplier.customers' clashes with reverse query
name for 'Supplier.place_ptr'.

HINT: Add or change a related_name argument to the definition for
'Supplier.customers' or 'Supplier.place_ptr'.
```

잦은 JOIN으로 성능을 저하시킬 수 있기 때문에 `MultiTalbe Inheritance`의 경우 잘 사용하지 않는다.

### Proxy models

다중 테이블 상속을 사용하면 모델의 각 하위 클래스에 대해 새 데이터베이스 테이블이 생성된다. 서브 클래스는 기본 클래스에 없는 추가 데이터 필드를 저장할 장소가 필요하기 때문에 사용한다. 그너라 때로는 모델의 파이썬에서의 동작만을 변경하고자 할 때가 있다. 예를 들어 기본 관리자를 변경하거나 새 메서드를 추가하는 경우가 이에 해당한다.

프록시 모델 상속은 위와같은 경우를 위한 것이다. 원래 모델에 대한 proxy를 생성한다. 프록시 모델의 인스턴스 생성, 삭제 및 업데이트 할 수 있으며 원본(비 프록시) 모델을 사용하는 것처럼 모든 데이터가 저장된다.차이점은 원본을 변경하지 않고 프록시 기본 모델 순서(ordering) 또는 기본 관리자(default manager)와 같은 것을 변경할 수 있다는 것이다.

프록시는 일반 모델처러 선언하되 Meta 클래스의 `proxy` 설정을 `True`로 설정하여 Django에 프록시 모델임을 알려야한다.

```python
class Person(models.Model):
  first_name = models.CharField(max_length=30)
  last_name = models.CharField(max_length=30)


class MyPerson(Person):
  class Meta:
    proxy = True

  def do_something(self):
    #...
    pass
```

Myperson 클래스는 상위 Person 클래스와 동일한 데이터베이스 테이블에서 작동한다. 특히 Person의 새로운 인스턴스는 MyPerson을 통해 액세스 할 수 있으며 그 반대의 경우도 가능하다.

```python
>>> p = Person.objects.create(first_name='foobar')
>>> Myperson.objects.get(first_name="foobar")
<MyPerson: foobar>
```

프록시 모델을 사용하여 모델에서 기본순서를 정의할수도 있다. Person 모델을 항상 ordering 하고싶지 않지만, 프록시를 사용할 때 last_name 속성으로 규칙적으로 ordering 하고자할 때는 다음과 같다.

```python
class OrderedPerson(Person):
  class Meta:
    ordering = ['last_name']
    proxy = True
```
이렇게하면 일반적 Person 쿼리는 Ordering되지 않지만, OrderedPerson 쿼리는 last_name에 의해 ordering 된다.

---

```python

class User(models.Model):
  """
    데이터베이스에 사용할 Model Class
  """
  name = models.CharField(max_length=40)
  is_admin = models.BooleanField(default=False)

  def __str__(self):
    return self.name

  class Meta:
    db_table = 'Inheritance_Proxy_User1'


class NormalUserManager(models.Manager):
  """
    Admin이 아닌 유저들만 사용하기 위한 매니저 객체
  """
  def normal_users(self):
    # 상위 클래스에서 정의한 Queryset을 반환한다.
    # 기본 Manager가 아닌 CustomManager를 사용할수도있으니 사용하는것이 권장된다.
    return super().get_queryset().filter(is_admin=False)

class AdminUserManager(models.Manager):
  """
    Admin만 사용하기 위한 매니저 객체
  """
  def admin_users(self):
    return super().get_queryset().filter(is_admin=True)


class NormalUser(UtilMixin, User):
  """
    User 모델에서 Admin이 아닌 유저만 사용하기위한 Proxy 모델
  """
  items = NormalUserManager()

  class Meta:
    proxy = True


class Admin(UtilMixin, User, AdminExtraManager):
  """
    User 모델에서 Admin만 사용하기 위한 Proxy 모델
  """

  class Meta:
    proxy = True

  def delete_user(self, user):
    user.delete()

class AdminExtraManager(models.Model):
  items = AdminUserManager()

  class Meta:
    abstract=True

class UtilMixin(models.Model):
  """
    필드가 없고 메서드만 존재하는 Mixin 클래스
    Proxy는 여러개의 Mixin 클래스를 상속받을 수 있다.
  """
  class Meta:
    abstract = True

  def show_items(cls):
    # 여기서 cls는 Admin이나 NormalUser 둘중 하나가 된다.
    # _default_manager는 objects와 같다.
    print(f'- Model ({cls.__name__}) items -')
    for item in cls._default_manager.all():
      print(item)

  def find_user(self, name):
    return User.objects.filter(name__contains=name)
```

해당 모델의 모델에서 선언된 첫번째 Manager가 \_default\_manager가 된다.
다중상속에서의 규칙중, 같은 메서드나, 같은 속성이 있을 경우 상속하고있는 클래스의 왼쪽부터 찾게된다.

```python
>>> NormalUser.objects
        <User1Manager>
>>> NormalUser.items
        <NormalUserManager>
>>> NormalUser._default_manager
        <NormalUserManager>


>>> Admin.objects
        <User1Manager>
>>> Admin.items
        <AdminUserManager>
# 이러한 결과가 나오는 이유는 class(UtilMixin, User1, AdminExtraManager)
# 라는 순서로 상속받고 있고, 매니저를 처음 선언한 곳은 User1 클래스이기때문에
# 해당 매니저를 사용하게된다.
>>> Admin._default_manager
        <User1Manager>
```

---

#### QuerySets still return the model that was requested

Django에서는 `Person`객체를 쿼리할때마다 `MyPerson`객체를 반환할 방법이 없다. `Person` 객체에 대한 queryset은 해당 유형의 객체를 반환한다. 즉, 그렇게 설계했고 사용하라는 뜻이다. 프록시 객체의 요점은 `Person`을 사용하는 코드를 사용하지 않고, 사용자 코드를 포함시킨 확장기능을 사용할 수 있다는 것이다.

#### Base class restrictions

프록시 모델은 정확히 하나의 비 추상 모델 클래스를 상속해야한다. 프록시 모델은 다른 데이터베이스 테이블의 행 사이에 연결을 제공하지 않으므로 여러개의 비 추상 모델을 상속받을 수 없다. 모델의 필드가 정의되지 않은 추상 모델 클래스(메서드만 있는 추상 모델 클래스, 이런것을 Mixin이라고 부른다.)를 여러개 상속 받을 수 있다.
프록시 모델은 공통의 비 추상 부모 클래스를 공유하는 임의의 수의 프록시 모델을 상속받을 수 있다.

```python
class Person: # 비 추상 부모 클래스
  pass

class NormalUser(Person):
  class Meta:
    proxy = True

class Admin(Person):
  class Meta:
      proxy = True

class Staff(Person):
  class Meta:
    proxy = True

# 가능하다! 하나의 프록시 모델은 실제로 데이터베이스에서 사용하는 공통의 비 추상 부모 클래스(Person)를 가지는 여러개의 프록시 모델을 상속할 수 있다.
class User(NormalUser, Admin, Staff):
  class Meta:
    proxy = True
```

#### Proxy model managers

프록시 모델에 모델 관리자를 지정하지 않으면 모델 부모로부터 관리자를 상속받는다. 프록시 모델에서 관리자를 정의하면 그것이 기본값이 되지만, 부모 클래스에 정의 된 관리자는 계속 사용할 수 있다.

```python
from django.db import models

class New Manager(models.Manager):
  # ...
  pass

class MyPerson(Person):
  objects = NewManager()

  class Meta:
    proxy = True
```

#### Differences between proxy inheritance and unmanaged models

프록시 모델 상속은 모델의 Meta클래스에서 관리되는 특성을 사용하여 관리되지 않는 모델(unmanaged model)을 만드는것과 매우 비슷하다.

(?)
