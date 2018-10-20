# Model index refernce

인덱스 클래스를 사용하면 데이터베이스의 인덱스를 쉽게 생성할 수 있다. `Meta.indexes` 옵션을 사용하여 추가할 수도 있다. 이 문서는 `index` 옵션을 포함하는 `INDEX` API reference를 설명한다.

> Referncing built-in indexes
> Indexes는 `django.db.models.indexes`에 정의되어있지만 편의를 위해 `django.db.models`에 포함되어있다. 표준 규칙은 `from django.db import models` 를 사용하고 `models.<IndexClass>` 의 indexes를 참조하여 사용한다.

## Index options

class Index(fields=(), name=None, db_tablespace=None)
  데이터베이스에 B-Tree 인덱스를 생성한다.

## Fields

### Index.fields

index가 필요한 필드의 이름을 list 또는 tuple로 작성

기본적으로 인덱스는 각 열에 대해 오름차순으로 생성된다. 열에 대해 내림차순으로 인덱스를 정의하려면 필드 이름 앞 하이픈을 추가한다.

예를들어 인덱스 `(fields=['headline', '-pub_date'])`는 `(headline, pub_date DESC)`와 함께 SQL을 생성한다.

---

## name

### Index.name

index의 이름, 이름이 제공되지 않으면 Django는 이름을 자동생성한다. 다른 데이터베이스와의 호환성을 위해 인덱스 이름은 30자를 초과할 수 없으며 숫자 (0-9) 또는 (\_)로 시작하면 안된다.

---

## db_tablespace

### Index.db_tablespace

이 index에 사용할 `database tablespace`의 이름. 단일 필드 index인 경우 `db_tablespace`가 제공되지 않으면 index는 필드의 `db_tablespace`에 작성된다.

`Field.db_tablespace`가 지정되지 않은경우 혹은 인덱스가 여러 필드를 사용하는 경우, 인덱스는 모델 클래스의 Meta의 `db_tablespace` 옵션에 테이블스페이스에 작성된다. `tablespace`가 설정되어있지 않은 경우, 인덱스는 테이블과 동일한 `tablespace`에 작성된다.
