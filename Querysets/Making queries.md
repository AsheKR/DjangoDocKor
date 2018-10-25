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
