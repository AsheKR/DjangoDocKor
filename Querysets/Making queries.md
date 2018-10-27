# Making queries

데이터 모델을 생성하면 Django는 자동으로 객체를 생성, 검색, 업데이트 및 삭제할 수 있는 데이터베이스 추상화 API를 제공한다.

문서는 API를 사용하는 방법을 설명한다. 다양한 모델 조회 옵션에 대해 자세한 내뇽은 `data model reference` 문서를 참조하라.

이 가이드(및 참조)전체에서 `Weblog` 응용 프로그램을 구성하는 다음 모델을 참조할것이다.

```python
from django.db import models

class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

    def __str__(self):
        return self.name

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    def __str__(self):
        return self.name

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField()
    authors = models.ManyToManyField(Author)
    n_comments = models.IntegerField()
    n_pingbacks = models.IntegerField()
    rating = models.IntegerField()

    def __str__(self):
        return self.headline
```

## Creating objects

django는 Python 객체에서 데이터베이스 테이블 데이터를 나타내기 위해 직관적인 시스템을 사용한다. 모델 클래스는 데이터베이스 테이블을 나타내며 클래스 인스턴스는 데이터베이스 테이블의 특정 레코드를 나타낸다.

객체를 생성하려면 모델클래스에 키워드 인자를 사용하여 인스턴스화 하고, `save()`메서드를 통해 데이터베이스에 저장한다.

`app/blog/models.py`에 위 내용이 있다고 가정하고 다음 예로 인스터스 및 데이터베이스 테이블 열을 생성할 수 있다.

```python
>>> from blog.models import blog
# 인스턴스를 만드는 과정
>>> b = Blog(name='Beatles Blog', tagline='All the latest Beatles news.')
# 인스턴스의 내용을 바탕으로 데이터베이스에 새로운 열을 추가하는 과정
>>> b.save()
```
위 내용은 뒤에서 SQL의 INSERT문을 수행한다. Django는 `save()` 메서드가 호출되기 전까지 database에 접근하지 않는다.

`save()`메서드는 아무 값도 리턴하지 않는다.

## Saving changes to objects

이미 데이터베이스에 있는 객체를 바꾸려면, `save()`를 사용한다.

주어진 `Blog`인스턴스 `b5`로 이 예제는 이름을 변경하고 데이터베이스에서 해당 레코드를 업데이트한다.
```python
>>> b5.name = 'New name'
>>> b5.save()
```

위 내용은 뒤에서 SQL의 UPDATE문을 수행한다. Django는 `save()` 메서드가 호출되기 전까지 database에 접근하지 않는다.

### Saving **ForeignKey** and **ManyToManyField** fields

`ForeignKey` 필드를 업데이트하는 것은 정상 필드를 저장하는 것과 같은 방식으로 작동한다. 다른점은 `ForeignKey` 필드에 올바른 타입의 객체를 지정하면 된다. 이 예는 `Entry` 및 `Blog`의 적절한 인스턴스가 이미 데이터베이스에 저장되어 있다 가정하여 `Entry`인스턴스 항복의 블로그 속성을 업데이트한다.

```python
>>> from blog.models import Blog, Entry
>>> entry = Entry.objects.get(pk=1)
>>> cheese_blog = Blog.objects.get(name='Cheddar Talk')
>>> entry.blog = cheese_blog
>>> entry.save()
```

`ManyToManyField`를 업데이트하는 것은 약간 다르게 동작한다. 필드에 `add()`메서드를 사용하여 관계에 레코드를 추가한다. 이 예제에서는 `Author`인스턴스 `joe`를 `entry`객체에 추가한다.

```python
>>> from blog.models import Author
>>> joe = Author.objects.create(name='Joe')
>>> entry.authors.add(joe)
```

한번에 `ManyToManyField`에 여러 레코드를 추가시키려면 add()호출에 여러 인수를 포함시킨다.

```python
>>> john = Author.objects.create(name="John")
>>> paul = Author.objects.create(name="Paul")
>>> george = Author.objects.create(name="George")
>>> ringo = Author.objects.create(name="Ringo")
>>> entry.authors.add(john, paul, george, ringo)
```

Django는 잘못된 유형의 객체를 할당하거나, 추가하려고하면 오류를 발생시킨다.

## Retrieving objects

데이터베이스에서 객체를 검색하려면 Model class의 `Manager`를 통해 `QuerySet`을 생성한다.

`QuerySet`은 데이터베이스의 객체 컬렉션을 나타낸다. 0개, 하나 또는 여러개의 필터를 가질 수 있다. 필터는 주어진 매개변수를 기반으로 쿼리 결과의 범위를 좁힌다. SQL용어에서 `QuerySet`은 `SELECT`문과 같으며 필터는 `WHERE` 또는 `LIMIT`와 같은 조건절이다.

모델의 `Manager`를 사용하여 `Queryset`을 얻는다. 각 모델에는 하나 이상의 `Manager`가 있으며 기본적으로 `objects`라고 한다. 모델 클래스를 통해 직접 액세스 할 수 있다.

```python
>>> Blog.objects
<django.db.models.manager.Manager object at ...>
>>> b = Blog(name="Foo", tagline='Bar')
>>> b.objects
Trackbak:
  ...
AttributeError: "Manager isn't accessible via Blog instances."
```

> `Manager`는 모델 인스턴스가 아닌 모델 클래스를 통해서만 액세스 할 수 있으므로 "테이블 수준"의 작업과 "레코드 수준"의 작업을 구분할 수 있다.

`Manager`는 모델에 대한 `QuerySets`의 주요 소스이다. 예를 들어 `Blog.objects.all()`은 데이터베이스의 모든 `Blog`객체를 포함하는 `QuerySet`을 반환한다.

### Retrieving all objects

테이블 개체를 검색하는 가장 간단한 방법은 모든 개체를 가져오는 것이다.  `Manager`의 `all` 메서드를 사용하면 된다.

```python
>>> all_entries  Entry.objects.all()
```
`all()`메서드는 해당 데이터베이스 테이블의 모든 개체를 포함한 `QuerySet`을 반환한다.

### Retrieving specific objects with filter

`all()`에 의해 반환 된 `Queryset`은 데이터베이스 테이블의 모든 개체이다. 그러나 보통 전체중 일부를 사용할필요가 있다.

subset을 작성하려면 필터 조건을 추가하여 초기 `QuerySet`을 구체화할 수 있다. `QuerySet`을 구체화하는 가장 일반적인 두가지 방법은 다음과 같다.

**filter(\*\*kwargs)**
지정된 검색 매개변수와 일치하는 객체가 포함된 새 `Queryset`을 반환한다.

**exclude(\*\*kwargs)**
지정된 검색 매개변수가 포함되지 않는 새 `QuerySet`을 반환한다.

위의 함수 정의에서 조회 매개변수(\*\*kwargs)는 `Field lookups`에 설명된 형식이여야한다.

예를 들어, 2006년부터 블로그 항목의 `QuerySet`을 얻으려면 `filter()`를 다음과 같이 한다.

```python
Entry.objects.filter(pub_date__year=2006)
```

#### Chaining filters

`QuerySet`을 정제한 결과는 `Queryset` 그 자체이므로 상세 검색을 연결할 수 있다.

```python
>>> Entry.objects.filter(
...   headline__startswith='What'
... ).exclude(
...   pub_date__gte=datetime.date.today()
... ).filter(
...   pub_date__gte=datetime.date(2005, 1, 30)
... )
```

데이터베이스에 있는 모든 항목의 초기 `QuerySet`을 가져와 필터하고, 제외하고, 다른 필터를 거친다.
최종 결과는 2005년 1월 30일과 현재 날짜 사이에 게시된 "What"으로 시작하는 제목이 있는 모든 항목을 포함하는 `QuerySet`이다.

#### Filtered QuerySets are unique

`QuerySet`을 다듬을 때마다 이전 `QuerySet`에 바인딩된 새로운 `QuerySet`을 얻게된다. 각각의 상세 검색은 저장되고 사용되며, 재사용될 수 있는 별개의 고유 `QuerySet`을 생성한다.

```python
>>> q1 = Entry.objects.filter(headline__startswith="What")
>>> q2 = q1.exclude(pub_date__gte=datetime.date.today())
>>> q3 = q1.filter(pub_date__gte=datetime.date.today())
```

이 세 `QuerySet`은 분리되어있다.
첫번째는 "What"으로 시작하는 제목을 포함한 `QuerySet`
두번째는 첫번째 항목의 하위 집합이며 `pub_date`가 오늘 또는 미래에 있는 레코드를 제외하는 추가 조건이 있다.
세번째는 첫번째 레코드의 하위 집합이며 `pub_date`가 현재 또는 미래의 레코드만 선택하는 추가 조건이 있다. 초기 `q1`은 다른 새 필터에 영향을 받지 않는다.

#### QuerySets are lazy

`QuerySet`을 생성하는 행위는 어떤 데이터베이스 활동도 포함하지 않는다. 많은 필터를 쌓을 수 있고, 장고는 `QuerySet`이 _evaluated_ 될 때까지 실제로 쿼리를 실행하지 않는다.

```python
>>> q = Entry.objects.filter(headline__startswith="What")
>>> q = q.filter(pub_date__lte=datetime.date.today())
>>> q = q.exclude(body_text__icontains="food")
>>> print(q)
```

3번의 데이터베이스를 조회하는것같지만, 실제로는 마지막행(`print(q)`)에서만 데이터베이슿에 한번 조회를 요청한다. 일반적으로 `QuerySet`의 결과는 사용자가 "요청"할때까지 데이터베이스에서 가져오지 않는다. 그렇게하면 `QuerySet`은 데이터베이스에 액세스하여 _evaluated_ 된다. _evaluated_ 되는 시기에 대한 자세한 내용은 `When QuerySets are evaluated.`를 참조하라

### Retrieving a single object with get()

`filter()`는 하나의 객체만이 쿼리와 일치하는 경우에도 항상 `QuerySet`을 제공한다. 이 경우 단일 요소를 포함하는 `QuerySet`이 제공된다.

쿼리와 일치하는 객체가 하나뿐인경우 객체를 집접 반환하는 `Manager`에서 `get()`메서드를 사용할 수 있다.

```python
>>> one_entry = Entry.objects.get(pk=1)
```

`filter()`와 마찬가지로 `get()`과 함께 모든 쿼리 표현식을 사용할 수 있다. `Field lookups`를 참조하라.


`get()`을 사용하는것과 `filter()`의 결과를 `[0]`으로 슬라이스하는 것의 차이점에 유의하라. `get()`은 쿼리와 일치하는 것이 없을경우 `DoesNotExist` 예외를 발생시킨다. 이 예외는 쿼리가 수행되고 있는 모델 클래스의 속성이다. 위 코드에서 기본키가 1인 `Entry`객체가 없으면 Django는 `Entry.DoesNotExist`를 발생시킨다.

비슷하게 장고는 `get()`의 결과에 하나 이상의 항목이 올경우 에러를 발생시킨다. 이 경우 `ModelObject` 자체 특성인 `MultiObjectsReturned`가 발생한다.

### Other QuerySet methods

대부분 데이터베이스에서 객체를 검색해야 할 때 `all()`, `get()`, `filter()`, `exclude()`를 사용한다. 그러나 다른 메서드들은 검색과는 거리가 멀다. 모든 다양한 `QuerySet`메서드 전체 목록은 `QuerySet API Reference`를 참조하라

### Limiting QuerySets

Python의 배열 슬라이스 구문의 subset를 사용하여 `QuerySet`을 특정 수의 결과로 제한할 수 있다. 이는 SQL의 `LIMIT` 및 `OFFSET`절과 동일하다.

예를들어, 첫번째 나오는 5개의 객체를 반환하는 것이다.(`LIMIT 5`)
```python
>>> Entry.objects.all()[:5]
```

6번째에서 10번째가지의 객체를 반환한다.(OFFSET 5 LIMIT 5)

```python
>>> Entry.objects.all()[5:10]
```

Negative indexing(`Entry.objects.all([-1])`)은 지원하지 않는다.

일반적으로 `QuerySet`을 슬라이스하면 새 `QuerySet`을 반환한다. 쿼리는 _evaluate_ 되지 않는다.
예외는 파이썬 슬라이스 구문의 "step" 매개변수를 사용하는 경우이다. 예를 들어 처음 10개의 2씩 증가하는 객체 목록을 반환하기 위해 실제로 쿼리를 실행한다.

```python
>>> Entry.objects.all()[:10:2]
```

필터링 또는 정렬된 쿼리셋을 슬라이스한경우 어떤 결과를 초래할지 애매모호하기때문에 금지되어있다.

목록이 아닌 단일 객체를 검색하려면(예: SELECT foo FROM bar LIMIT 1) 슬라이스 대신 간단한 index를 사용한다. 예를 들어 항목을 제목순으로 정렬한 후 데이터베이스의 첫 항목을 반환한다.

```python
>>> Entry.objects.order_by('headline')[0]
```

이는 대략 다음과 같다.

```python
>>> Entry.objects.order_by('headline')[0:1].get()
```

그러나 이들 중 첫 항목은 `IndexError`를 발생시키고 두번째 항목은 지정된 조건과 일치하는 개체가 없는 경우 `DoesNotExist`를 발생시킨다. 자세한 애용은 `get()`항목을 참조하라.

### Field lookups

필드 조회는 `SQL WHERE`절의 항목을 지정하는 방법이다. `QuerySet` 메서드 `filter()`, `exclude()`, `get()`에 대한 키워드 인수로 지정된다.

기본 검색 키워드 인수는 `field__lookuptype=value`형식을 취한다.

```python
>>> Entry.objects.filter(pub_date__lte='2006-01-01')
```

이는 대략적으로 다음 SQL로 변환한다.

```sql
SELECT * FROM blog_entry WHERE pub_date <= '2006-01-01';
```

lookup에 지정된 필드는 모델 필드의 이름이여야한다. 그러나 한가지 예외가 있다. `ForeignKey`의 경우 `_id`접미사로 필드 이름을 지정해야한다. 이 경우, value 매개변수에는 외부 모델의 기본 키의 값이 포함되어야한다.


```python
>>> Entry.objects.filter(blog_id=4)
```

잘못된 키워드 인수를 전달하면 lookup 함수가 `TypeError`를 발생시킨다.

데이터베이스 API는 약 20개의 조회 유형을 지원한다. `field lookup reference`에서 완전한 참조를 찾을 수 있다. 사용할 수 있는 것을 제공하기위해 다음과 같은 일반적이 몇가지 조회가 사용된다.


`exact`
```python
>>> Entry.objects.get(headline__exact='Cat bites dog')
```
이는 다음의 SQL문을 생성한다.
```sql
SELECT ... WHERE head_line='Cat bites dog'
```

조회 유형을 제공하지 않으면(즉 키워드 인수에 밑줄이 두개 포함되지 않은 경우) 그 조회 타입은 `exact`로 가정한다.

```python
>>> Blog.objects.get(id__exact=14)
>>> Blog.objects.get(id=14)
```

이는 정확한 조회가 일반적인 경우이기 때문에 편의상 사용한다.

`iexact`
대소문자 구분없이 **일치** 하는것을 찾는다.
```python
>>> Blog.objects.get(name__iexcact="beatles blog")
```

위는 `Beatles Blog`, `beatles blog`, `BeAtlES blOG`와 일치한다.

`contains`
대소문자를 구별하고, **포함** 된 내용을 찾는다.

```python
>>> Entry.objects.get(headline__contains='Lennon')
```
다음과 같은 SQL문을 생성한다.

```sql
SELECT ... WHERE headline LIKE '%Lennon%';
```

이는 `Today Lennon honred`는 일치하지만 `today lennon honored`는 일치하지 않는다.

대소문자를 구분하지 않는 `icontains` 버전도 있다.

`startswith, endswith`
~으로 시작하는 것, ~으로 끝나는 것을 검색한다. 대소문자를 구분하지 않는 `istartswith`, `iendswith`도 존재한다.

더 많은 정보는 `field lookup reference`에서!

### Lookups that span relationships

Django는 백그라운드에서 자동으로  SQL JOIN을 처리하면서 조회시 관계를 따르는 가장 강력하고 직관적인 방법을 제공한다. 관계를 확장하려면 원하는 필드에 도달 할 때까지 모델에서 관련 필드의 필드 이름을 이중 밑줄로 구분하여 사용한다.

이는 `Blog`이름이 `Beatles Blog`인 모든 `Entry`객체를 검색한다.

```python
>>> Entry.objects.filter(blog__name="Beatles Blog")
```

이는 원하는많큼 깊을 수 있다. 거꾸로도 동작한다. "역"관계를 나타내기위해 모델의 소문자 이름을 사용한다.

이는 `Entry`에서 `headline`이 `Lennon`을 포함하는 `Blog`객체를 검색한다.

```python
>>> Blog.objects.filter(entry__headline__contains='Lennon')
```

여러 관계에 걸쳐 필터링을 수행하고 중간 모델 중 하나에 필터 조건을 충족시키는 값이 없는 경 Django는 비어있는(모든 값이 NULL인)것처럼 취급한다. 이는 오류를 제기하지 않는다는 말이된다.

```python
>>> Blog.objects.filter(entry__authors__name='Lennon')
```

`Entry`과 관련된 `author`가 없는 경우 누락된 작성자로인해 `name`이 첨부되지 않은 것처럼 처리된다. 이는 올바르게 예상한대로 동작한것이고. 다음과 같은 상황에서 혼란스러울 수 있다.

```python
>>> Blog.objects.filter(entry__authors__name__isnull=True)
```

위는 작성자에 빈 이름이 있는 블로그 객체와 빈 작성자가 있는 블로그개체를 반환한다. 후자의 객체를 원하지 않는다면 다음과 같이 작성한다.

```python
Blog.objects.filter(entry__authors__isnull=False, entry__authors__name__isnull=True)
```

### Spanning multi-valued relationships

`ManyToManyField` 또는 역방향 `ForeignKey`를 기반으로 개체를 필터링할 때 두가지 필터 방식이 존재한다.

`
Blog1, Blog2

Entry1
      blog=Blog1
      headline='Lennon'
      pub_date=datetime(2008, 3, 3),

Entry2
      blog=Blog2
      headline='lhy'
      pub_date=datetime(2008, 3, 3),

Entry3
      blog=Blog2
      headline='Lennon'
      pub_date=datetime(2018, 10, 18),
`

위의 내용을 바탕으로 아래를 설명한다.

```python
# 여러 관계 값을 필터하는 경우 다른 쿼리문을 반환한다.
>>> Blog.objects.filter(entry__headline='Lennon', entry__pub_date__year=2008)
# Blog중에서
#   headline이 'Lennon'이면서 'pub_date'의 'year'가 2008인 Entry가 포함된 Blog를 반환
# result = <QuerySet [<Blog: Blog1>]>
>>> Blog.objects.filter(entry__headline='Lennon').filter(entry__pub_date__year=2008)
# Blog중에서
#   headline이 'Lennon'인 Entry가 포함된 Blog 중에서
#     pub_date의 year가 2008인 경우의 Entry를 포함한 Blog를 반환
# result = <QuerySet [<Blog: Blog1>, <Blog: Blog2>]>

# 아래 두개는 같은 쿼리문을 반환한다.
>>> Entry.objects.filter(headline='Lennon').filter(pub_date__year=2008)
>>> Entry.objects.filter(headline='Lennon', pub_date__year=2008)
```

> 여러값을 주어진 Exclude의 경우 Filter와는 다르게 동작한다.
> ```python
>   Blog.objects.exclude(
>     entry__headline__contains='Lennon',
>     entry__pub_date__year=2008
>   )
> ```
> 예상한 결과는 Blog2 객체만 반환해야하는데 실제로는 Blog1, Blog2 모두를 반환한다.
> 여러값을 제한하는 경우 다믕과 같이 작성해야한다.
> ```python
>   Blog.objects.exclude(
>     entry__in=Entry.objects.filter(
>        headline__contains='Lennon',
>        pub_date__year=2008,
>     )
>   )
> ```

### Filters can reference fileds on the model

지금까지는 모델 필드의 값과 상수를 비교하는 필터를 만들었다. 그러나 모델 필드의 값을 동일한 모델의 다른 필드와 비교하려면 어떻게 해야할까?

Django는 이러한 비교를 허용하는 `F 표현식`을 제공한다. `F()`의 인스턴스는 쿼리 내의 모델 필드에 대한 참조로 사용된다. 그런 다음 이 참조를 쿼리 필터에 사용하여 동일한 모델 인스턴스의 두개의 다른 필드 값을 비교할 수 있다.

이 예는 `pingbacks`보다 많은 `comments`가 있는 모든 블로그를 찾기위해 `pingbacks`를 참조하는 `F()`객체를 생성하고 쿼리에서 해당 `F()`객체를 사용한다.

```python
>>> from django.db.models import F
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks'))
```

Django는 상수 및 다른 `F()`객체와 함께 `F()`객체를 사용하여 산술연산을 지원한다. `pingbacks`보다 2배많은 `comments`가 있는 블로그 항목을 찾기위해 다음같이 작성한다.
```python
>>> Entry.objects.filter(n_comments__gt=F('n_pingbacks') * 2)
```

Entry의 `rating`이 `pingbacks`와 `comments`의 합보다 작은 Entry를 찾기위해 다음과 같이 작성한다.

```python
>>> Entry.objects.filter(rating__lt=F(n_pingbacks) + F(n_comments))
```

이중 밑줄 표기법을 사용하여 `F()`객체의 관계를 확장할 수 있다. 이중 밑줄이 있는 `F()`객체는 관련 객체에 액세스 하는데 필요한 조인을 도입한다. 예를들어 작성자의 이름과 블로그 이름이 동일한 Entry을 검색하기위해 다음과 같이 작성한다.

```python
>>> Entry.objects.filter(blog__name=F('authors__name'))
```

날짜/시간 필드의 경우 `timedelta` 객체를 더하거나 뺄 수 있다. 다음은 게시된 후 3일 이상 수정된 모든 Entry를 반환한다.

```python
>>> from datetime import timedelta
>>> Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
```

F()연산은 bit연산도 지원한다.

### The pk lookup shortcut

편의상 Django는 `primary key`를 나타내는 pk lookup shortcut을 제공한다.

PK에 접근하는 아래 세 문장은 동일한 결과를 반환한다.
```python
>>> Blog.objects.get(id__exact=14) # 정확한 형태
>>> Blog.objects.get(id=14) # __exact 생략가능
>>> Blog.objects.get(pk=14) # pk는 id__exact의 함축표현
```

pk는 `__exact`에만 제한되는것이아니라 모든 검색어와 결합하여 사용할 수 있다.

```python
# 1,4,7번 PK의 블로그를 필터
>>> Blog.objects.filter(pk__in=[1,4,7])

# 14보다는 PK 블로그를 필터
>>> Blog.objects.filter(pk__gt=14)
```


pk 조회는 조인에서도 작동한다.

```python
>>> Entry.objects.filter(blog__id__exact=3)
>>> Entry.objects.filter(blog__id=3)
>>> Entry.objects.filter(blog__pk=3)
```

### Escaping percent signs and underscores in LIKE statements

LIKE문(exact, contains, startswith, endswith)와 같은 필드 조회는 LIKE문에 사용된 특수문자(% 및 \_)를 자동으로 이스케이프처리한다. LIKE문에서 %는 여러문자 와일드카드를 나타내고 \_는 단일 문자 와일드카드를 나타낸다.

```python
>>> Entry.objects.filter(headline__contain='%')
```

Django에서 다음과 같이 처리한다.

```sql
SELECT ... WHERE headline LIKE '%\%%';
```

### Caching and QuerySets

각 `QuerySet`에는 데이터베이스 액세스를 최소화하기위한 캐시가 포함되어있다.어떻게 동작하는지 이해하면 가장 효율적인 코드를 작성할 수 있다.

새로 생성된 `QuerySet`에서는 캐시가 비어있다. 처음으로 `Queryset`이 `evaluated`되고, 즉 데이터베이스 쿼리가 발생하면 Django는 쿼리 결과를 `QuerySet`의 캐시에 저장하고 명시적으로 해당 결과를 반환한다. `Queryset`의 후속 `evaluated`는 캐시 결과를 다시 재사용한다.

```python
# 현재 QuerySet 캐시는 비워져 있다. 즉, 데이터베이스에 요청을하지 않은 상태
>>> q = Blog.objects.all()
# 아래 표현식을 사용하면 evaluated, 평가되었다고 한다. 데이터베이스에 요청을 하여 결과값을 가져온다.
>>> q
# result : <QuerySet [<Blog: ...
# 한번더 사용했을 때 위의 캐시된 값을 사용한다.
>>> q
# result : <QuerySet [<Blog: ...
# 캐시된 값을 사용하면 좋은점이 데이터베이스에 요청을 최소하할 수 있지미나
# 데이터베이스 값이 업데이트되어도 캐시된 값을 변화하지 않는다.
```

아래는 두번의 데이터베이스 요청을 발생시킨다.

```python
>>> print([e.headline for e in Entry.objects.all()])
>>> print([e.pub_date for e in Entry.objects.all()])
```

이는 두번의 요청이라는 낭비를 발생하고, 두 요청 사이 새 데이터나 삭제된 데이터가 존재할 수 있기때문에 같은 데이터베이스 데이터를 사용하지 않을 수 있다.

이 같은 문제를 피하기위해 다음과 같이 작성해야한다.

```python
>>> queryset = Entry.objects.all()
>>> print([p.headline for p in queryset]) # Evaluate the query set.
>>> print([p.pub_date for p in queryset]) # Re-use the cache from the evaluation.
```

### when QuerySets are not cached

Queryset은 항상 결과를 캐시하지 않는다. 쿼리셋의 일부만을 평가할 때 캐시가 검사되지만 채워지지 않은경우 후속 쿼리에서 반환되는 항목은 캐시되지 않는다. 이는 배열 슬라이스나 인덱스를 사용하는 쿼리 세트를 제한해도 캐시가 채워지지 않음을 의미한다.

예를들어 queryset 오브젝트에서 반복적으로 특정 인덱스를 얻으면 매번 데이터베이스를 조회하게된다.

```python
>>> queryset = Entry.objects.all()
>>> print(queryset[5]) # Queries the database
>>> print(queryset[5]) # Queries the database again
```

그러나 전체 쿼리셋이 이미 평가된 경우 캐시가 사용된다.

```python
>>> queryset = Entry.objects.all()
>>> [entry for entry in queryset] # Queries the database
>>> print(queryset[5]) # Uses cache
>>> print(queryset[5]) # Uses cache
```

쿼리셋 전체를 평가하기 위해 다음을 사용할 수 있다.

```python
>>> [entry for entry in queryset]
>>> bool(queryset)
>>> entry in queryset
>>> list(queryset)
```

## Complex lookups with Q objects

`filter()`의 keyword 인수는 "AND" 연산이된다. 만약 OR이나 다른 것을 쓰는 복잡한 쿼리가 필요하다면 `Q objects`를 사용해야한다.

`Q 객체`는 키워드 인수 모음을 캡슐화하는데 사용되는 객체이다. 이러한 키워드 인수는 위의 `field lookups`처럼 지정한다.

```python
from django.db.models import Q
Q(question__startswith='What')
```

`Q objects`는 `&`와 `|`를 사용하여 결합할 수 있다. 연산자가 두개의 `Q objects`에 사용되면 새로운 `Q objects`가 생성된다.

```python
Q(question__startswtih="Who") | Q(question__startswith='What')
```

이는 다음과 같은 WHERE 조건을 생성한다.

```sql
WHERE question LIKE "Who%" OR question LIKE "What%"
```

`Q objects`는 `~`를 사용하여  부정(NOT) 쿼리 조회를 사용할 수 있다.

```python
Q(question__startswith='Who') | ~Q(pub_date__year=2005)
```

키워드 인수를 사용하는 각 조회 함수는 하나 이상의 `Q objects`를 위치인자로 전달 할 수 있다. 조회 함수에 여러개의 `Q objects` 인수를 제공하면 "AND"가 된다.

```python
Poll.objects.get(
  Q(question__startswith='Who'),
  Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
```

위의 문장은 다음의 SQL 문으로 변환된다.

```sql
SELECT * from polls WHERE question LIKE 'Who%'
    AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
```

조회 함수는 `Q objects`와 키워드 인수의 혼합 사용가능하다. 조회 함수에 제공된 모든 인수는 함께 AND된다.
파이썬 규칙에서도 그렇듯 위치 인자(`Q objects`)가 앞에 와야하고 키워드 인수가 뒤에 와야한다.

```python
Poll.objects.get(
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
    question__startswith='Who',
)
```

위의 문장은 다음의 SQL 문으로 변환된다.

```python
# INVALID QUERY
Poll.objects.get(
    question__startswith='Who',
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6))
)
```

## Comparing objects

두개의 모델 인스턴스를 비교하려면 표준 Python 비교 연산자인 == 을 사용하면 된다. 이는 두 모델의 기본 키 값을 비교하여 검사한다.

```python
>>> some_entry == other_entry
>>> some_entry.id == other_entry.id
```

모델의 기본키가 `id`가 아닌경우도 PK 값으로 비교하므로 상관없다.  `name`이 PK인 경우도 정상적으로 비교된다.

```python
>>> some_obj == other_obj
>>> some_obj.name == other_obj.name
```

## Deleting objects

delete 메소드는 편리하게 `delete()`라는 이름을 가진다. 이 메서드는 즉시 개체를 삭제하고 삭제된 개체수와 개체 유형 당 삭제 수가 있는 dictionary를 반환한다.

```python
>>> e.delete()
(1, {'weblog.Entry': 1})
```

개체를 대량으로 삭제할 수도 있다. 모든 `Queryset`에는 모든 멤버를 삭제하는 `delete()`메서드가 포함되어있다.

```python
>>> Entry.objects.filter(pub_date__year=2005).delete()
(5, {'webapp.Entry': 5})
```

(?)

Django는 객체를 삭제할 때 기본적으로 `ON DELETE CASCADE`SQL 동작을 모방한다. 즉, 삭제 될 객체를 가리키는 외래 키를 가진 객체는 함께 사라진다.

이 `CASCADE` 동작은 `ForeignKey`에 대한 `on_delete`인수를 통해 사용자 지정할 수 있다.`delete()`는 `Manager` 자체에서 노출되지않는 유일한 `Queryset`이다. 이는 실수로 `Entry.objects.delete()`를 요청하지 않고 모든 항목을 삭제하지 못하게 하는 안전 매커니즘이다. 모든 개체를 삭제하려면 명시적으로 전체 쿼리 집합을 요청해야한다.

```python
>>> Entry.objects.all().delete()
```

## Copying model instances

모델 인스턴스를 복사하는 기본 제공방법은 없지만, 모든 필드의 값을 복사하여 새 인스턴스를 쉽게 생성할 수 있다. 가장 간단하게 pk를 없음으로 지정하고 `save()`메서드를 호출하면 모든 필드 값을 복사한 새 인스턴스가 생성된다.

```python
blog = Blog(name='My blog', tagline='Blogging is easy')
blog.save() # blog.pk == 1

blog.pk = None
blog.save() # blog.pk == 2
```

상속일경우 더욱 복잡해진다.

```python
class ThemeBlog(Blog):
  theme = models.CharField(max_length=200)

django_blog = ThemeBlog(name='Django', tagline='Django is easy', theme='python')
django_blog.save() # django_blog.pk == 3
```

상속 작동 방식때문에 pk와 ID 모두 없음으로 설정해야한다.
상속은 부모테이블, 자식 테이블이 생성되고, 자식테이블의 PK에 `OneToOneField`로 부모테이블과 연결되게 된다.
그러므로 자식 테이블의 id, 부모 테이블의 id(PK)를 모두 바꾸어 주어야 새 인스턴스 생성이 가능하게 된다.

```python
django_blog.pk = None
django_blog.id = None
django_blog.save() # django_blog.pk == 4
```

이 프로세스는 모델 데이터베이스 테이블에 포함되지 않은 관계를 복사하지 않는다. 예를 들어, `Entry`에는 `Author`에 대한 `ManyToManyField`가 있다. 이 `Entry`를 복제한 후 새로운 `Entry`에 대한 다대 다 관계설정을 해야한다.

```python
entry = Entry.objects.all()[0] # some previous entry
old_authors = entry.authors.all()
entry.pk = None
entry.save()
entry.authors.set(old_authors)
```

`OneToOneField`의 경우, 일대일 고유 제한 조건을 위반하지 않도록 관련 개체를 복제하고 이를 새 개체의 필드에 할당해야한다. 위의 내용과 연속되어 사용한다고 가정하자.

```python
detail = EntryDetail.objects.all()[0]
detail.pk= None
detail.entry = entry
detail.save()
```

## Updating multiple objects at once

때로는 `QuerySet`의 모든 객체에 대해 필드를 특정 값으로 설정하려고 할 때 `update()`메서드를 사용하여 작업을 수행할 수 있다.

```python
# Update all the headlines with pub_date in 2007.
Entry.objects.filter(pub_date__year=2007).update(headline='Everything is the same')
```

이 방법을 사용하여 비 관계 필드와 `ForeignKey`필드만 설정가능하다.비 관계 필드를 업데이트하려면 새 값을 상수로 제공한다. `ForeignKey` 필드를 업데이트하려면 모델 인스턴스를 제공해야한다.

```python
>>> b = Blog.objects.get(pk=1)

# Change every Entry so that it belongs to this Blog.
>>> Entry.objects.all().update(blog=b)
```

`update()` 메서드는 즉시 적용되며 쿼리와 일치하는 행 수를 반환한다. 일부 행에 이미 새 값이 있는 경우 업데이트 된 행 수와 다를 수 있다.업데이트 되는 `QuerySet`에 대한 유일한 제한은 모델의 주 테이블인 하나의 데이터베이스 테이블에만 액세스 할 수 있다는 것이다. 관계 필드를 기준으로 필터링은 가능하지만 모델의 주 테이블의 열만 업데이트 할 수 있다.

```python
>>> b = Blog.objects.get(pk=1)

# Update all the headlines belonging to this Blog.
>>> Entry.objects.select_related().filter(blog=b).update(headline='Everything is the same')
```

`update()`메서드는 SQL문으로 직접 변환된다. 직접 업데이트를 위한 대량 작업이다. (?)

```python
for item in my_queryset:
  item.save()
```

업데이트 호출은 `F expressions`를 사용하여 다른 필드의 값을 기반으로 한 필드 업데이트도 가능하다. 이는 현재 값을 기반으로 카운터를 증가시키는데 특히 유용하다.

```python
>>> Entry.objects.all().update(n_pingbacks=F('n_pingbacks') + 1)
```

그러나 필터 및 제외 절의 `F()` 개체와 달리 업데이터에서 `F()`개체를 사용할 때 조인을 도입할 수 없으며 업데이트 되는 모델의 로컬 필드만 참조가능하다. `F()`개체를 사용하여 JOIN을 도입하려하면 `FieldError`가 발생한다.

```python
# This will raise a FieldError
>>> Entry.objects.update(headline=F('blog__name'))
```
