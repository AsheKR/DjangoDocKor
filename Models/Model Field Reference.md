# Model field reference

## Field options

모든 필드 유형에서 사용할 수 있다. 모두 선택 사항이다.

### null

#### Field.null

**True**면 Django는 데이터베이스 안에빈 값인 NULL을 저장한다.
Default는 **False**

**CharField**나 **TextField** 같은 문자열 기반의 필드인 경우 null 사용을 피해야한다. 만약 문자열 기반의 필드가 null=True를 가지게된다면 그 의미는 두가지 값을 가지게 될 수 있다. **데이터가 없는 "NULL"**과 **빈 문자열**의 의미를 가지게 된다. 대부분의 경우 "데이터 없음"에 두가지 의미를 가질 필요가 없다. Django에서 협약은 not NULL은 빈 문자열을 사용한다. 한가지 예외로 **CharField**가 **unique=True**와 **blank=True**를 가질 경우 **null=True**는 여러개의 빈 값을 저장할 때 unique 제약 조건 위반을 피하기 위해 필요하다.

문자열 기반이나 문자열이 아닌 기반의 필드 둘 다  form에서 빈 값을 허용하고 싶다면**blank=True** 를 필요로 한다. **null** 파라미터는 오직 데이터베이스에만 영향을 끼치기 때문이다.

---

### blank

#### Field.blank

**True**면 이 필드에 blank를 허용한다.
Default는 **False**

**null**과는 다르다! **null**은 순수하게 데이터베이스와 연관된 것이고, **blank**는 값 체크와 연관된 것이다. field가 **blank=True**를 가질경우 form에서 빈 값을 허용하게 되고, **blank=False**인 경우 그 필드는 반드시 적어주어야 한다.

---

### choices

#### Field.choices

필드의 선택항목으로 사용될 두 아이템이 순환가능한 객체안에 구성되어있는 것을 말한다. ( 예 ) `[(A,B), (C,D) ...]` )
choices가 주어진다면 모델 유효성 검사를 통해 기본 양식 위젯이 표준 텍스트 필드 대신 선택항목이 포함된 선택 상자로 나타내준다.

튜플의 첫 요소는 모델에 설정할 실제 값이고, 두번째 요소는 사람이 읽을 수 있는 이름이다.

```python
YEAR_IN_SCHOOL_CHOICES = (
	('FR', 'Freshman'),
	('SO', 'Sophomore'),
	('JR', 'Junior'),
	('SR', 'Senior'),
)
```

일반적으로 모델 클래스 안에서 choices를 적절히 정의하고, 각 값에 대해 적절히 명명된 상수를 정의하는 것이 가장 좋다.

```python
from django.db import models

class Student(models.Model):
    FRESHMAN = 'FR'
    SOPHOMORE = 'SO'
    JUNIOR = 'JR'
    SENIOR = 'SR'
    YEAR_IN_SCHOOL_CHOICES = (
        (FRESHMAN, 'Freshman'),
        (SOPHOMORE, 'Sophomore'),
        (JUNIOR, 'Junior'),
        (SENIOR, 'Senior'),
    )
    year_in_school = models.CharField(
        max_length=2,
        choices=YEAR_IN_SCHOOL_CHOICES,
        default=FRESHMAN,
    )

    def is_upperclass(self):
        return self.year_in_school in (self.JUNIOR, self.SENIOR)
```

모델클래스 외부에서 choices 목록을 정의한 후 참조할 수 있지만 모델 클래스의 각 choices 항목에 대한 선택사항과 이름을 정의하면 해당 정보를 사용하는 클래스와 해당 정보가 모두 유지된다.그리고 만들어놓은 choices를 쉽게 참조할 수 있다. ( 예 ) `Student.SOPHOMORE`)

또한 choices를 그룹으로 분류하여 나타나게 할 수 있다.

```python
MEDIA_CHOICES = (
    ('Audio', (
            ('vinyl', 'Vinyl'),
            ('cd', 'CD'),
        )
    ),
    ('Video', (
            ('vhs', 'VHS Tape'),
            ('dvd', 'DVD'),
        )
    ),
    ('unknown', 'Unknown'),
)
```

Form 화면에 나타나는 정보는 다음과 같다.

```
Media choice: 선택박스

선택박스 내용
Audio (선택 불가하고 그룹 이름으로만 쓰임)
	Vinyl
	CD
Video
	VHS Tape
	DVD
unknown
	unknown
```

각 튜플의 첫 요소는 그룹에 적용할 이름이다. 두번째 요소는 튜플 내부에 2개로 이루어진 튜플이 여러개 존재하고, 각 2개로 이루어진 튜플은 옵션에 대한 value와 human-readable한 이름을 포함한다.
그룹 옵션은 단일 목록 내 그룹화되지 않은 옵션과 결합될 수 있다. ( unknown 항목 )

chioces가 있는 모델 필드는 현재 값에 대한 사람이 읽을 수 있는 이름을 검색하는 메서드를 추가한다. (get_F00_display())

choices는 반드시 리스트나 튜플일 필요 없이 아무 반복 가능한 객체이면 된다. 이를 통해 choices를 동적으로 구성 가능하다. choices를 동적으로 구현하는 것 보다 ForeignKey를 사용하는 것이 더 낫다. choices는 더이상 변화하지 않는 정적인 데이터를 의미하기 때문.

**default=?**와 함께 필드에 **blank=False** 가 설정되어 있지 않으면 "----------"가 포함 된 레이블이 선택 상자와 함께 표시된다. 이 동작을 무시하려면 **None** 을 포함하는 선택 항목에 튜플을 추가한다. ( 예 ) `(None, 'Your String For Display')`) 또는 **CharField**처럼 **None** 대신 빈 문자열을 사용할 수 있다.

### db_column

#### Field.db_column

이 필드에 사용할 데이터베이스 열의 이름, 이것이 주어지지 않으면 장고는 필드 이름을 사용한다.

데이터베이스 열 이름이 SQL 예약어이거나 Python 변수 이름에 허용되지 않는 문자 (특히 하이픈)을 포함하여도 괜찮다. 장고는 열과 테이블 이름을 인용한다. (?)

### db_index

#### Field.db_index

**True**면 현재 필드에 index가 생성된다.

### db_tablespace

#### Field.db_tablespace

현재 필드에 **index**가 사용되었을 경 **database tablespace** 이름을 할당할 수 있다. 기본값은 설정되어 있을 경우 프로젝트 내 **DEFAULT_INDEX_TABLESPACE** 설정을 사용하고 또는 모델의 **db_tablespace**가 설정된 경우 사용한다. 백엔드가 인덱스의 tablespace를 지원하지 않으면 옵션은 무시된다.

### default

#### Field.default

필드의 기본값을 나타낸다. 값 또는 호출 가능한 객체이고 호출 가능하면 새로운 객체가 생성될 때 마다 호출된다. 기본값은 변경 가능한 객체(모델 인스턴스, 리스트, 셋)일 수 없다. 이는 해당 오브젝트의 동일한 인스턴스에 대해 참조가 모든 새 모델 인스턴스에서 기본값으로 사용되기 때문이다. 대신 원하는 기본 값을 호출가능한 코드를 사용하라. 예를 들어 **JSONField**의 기본 **dict**를 지정하려면 다음과 같이 작성한다.

```Python
def contact_default():
    return {"email": "to1@example.com"}

contact_info = JSONField("ContactInfo", default=contact_default)
```

**lambda**는 default에서 사용할 수 없다. 왜냐하면 마이그레이션으로 직렬화 할 수 없기 때문이다. (serialized by migrations)

모델 인스턴스에 매핑되는 **ForeignKey** 필드의 경우 기본값은 모델 인스턴스 대신 참조하는 필드 값 (**to_field**가 설정되어 있지 않은 경우 **pk**)이어야 한다.

**default** 값은 새 모델 인스턴스가 만들어지고 값이 필드에 제공되지 않을 때 사용한다. 필드가 기본 키인 경우 필드가 **None**으로 설정된 경우 **default** 값을 사용한다.

### editable

#### Field.editable

**False**일 경우 해당 필드는 관리자 또는 다른 ModelForm에서 표시되지 않는다. 또한 모델 유효성 검사를 건너 뛴다.
기본값은 **True**

### error_messages

#### Field.error_messages

error_messages 인수를 사용하면 해당 필드에서 발생시키는 기본 메세지를 대체할 수 있다. 덮어쓰려 키와 매칭되는 에러메세지가 포함된 **dict** 를 전달한다.

오류 메세지에는 **null, blank, invalid, invalid_choice, unique, unique_for_date** 가 포함된다. 추가적인 오류 메세지 Key는 **Field types** 섹션에 지정되어있다.

에러메세지들은 양식에 Form에 전달되지 않는 경우가 많다. 이에 대해서 **Considerations regarding model's error_messages** 를 참고하자.

### help_text

#### Field.help_text

Form 위젯에서 함께 표시되는 "도움말" 텍스트이다. 필드 양식에서 사용되지 않아도 문서화에 유용하다.

이 값은 form에서 자동으로 생성되는 HTML-escaped 처리되지 않는다. 원하는 경우 **help_text** 에 HTML을 포함시킬 수 있다.

```python
help_text="Please use the following format: <em>YYYY-MM-DD</em>"
```

또는 일반 텍스트와 django.tils.html.escape()를 사용하여 HTML 특수문자를 escape 처리할 수 있다. XSS 공격을 피하기 위해 신뢰할 수 없는 사용자 도움말 텍스트의 경우 escape 처리하는 것이 좋다.

### primary_key

#### Field.primary_key

**True**면 모델의 Primary 키 필드가 된다.

모델의 필드에 **primary_key=True** 를 명시하지 않을경우, Django는 자동적으로 primary key를 저장하기 위해 **AutoField** 를 자동으로 추가한다. 이 기본키 동작을 덮어쓰지 않으려면 **primary_key=True** 를 설정할 필요가 없다.

**primary_key**는 읽기 전용이다. 기존 객체의 기본 키 값을 변경한 다음 저장하면 새 객체가 이전 것과 함께 생성된다.(?)

### unique

#### Field.unique_for_date

**True**면 테이블 전체에서 고유해야한다.

이는 데이터베이스 레벨 및 모델 검증에 의해 시행된다. **unique**필드에 중복 값이 있는 모델을 저장하려고 하면 모델의 save() 메서드에 의해 **django.db.IntegrityError** 가 발생한다.

이 옵션은 ManyToManyField 및 OneToOneField를 제외한 모든 필드 유형에 유효하다.

unique가 True이면 인덱스 생성을 의미하므로, db_index를 지정할 필요가 없다.

### unique_for_date

#### Field.unique_for_date

DateField, DateTimeField의 이름으로 설정하여 날짜 필드의 값에 대해 고유해야한다.

예를 들어, **title** 이라는 필드에 **unique_for_date="pub_date"** 라는 속성이 주어지면, Django는 **title** 과 **pub_date** 가 같은 두개의 레코드를 입력할 수 없게된다.

만약 DateTimeField를 가리키게된다면, 필드의 날짜 부분만 고려된다. **USE_TZ=True** 일 시 개체가 저장될 때 **current time zone** 에서 검사가 수행된다.

모델 유효성 검사중에는 Model.validate_unique()에 의해 적용되지만 데이터베이스 수준에서는 적용되지 않는다. **model_validate_unique()** 는 **unique_for_date** 제약 조건이 ModelForm의 일부가 아닌 필드가 포함하는 경우 ( 예 ) 필드 중 하나가 **exclude** 에 나열되거나 **editable=False** 인경우) 해당 특정 제약조건에 대한 유효성 검사를 건너 뛴다.

### unique_for_month, unique_for_year

#### Field.unique_for_month
#### Field.unique_for_year

unique_for_date와 비슷하지만 고유해야하는 부분이 월인가 년인가에 따라 다르게된다.

### verbose_name

#### Field.verbose_name

human-readable한 필드 이름을 설정한다.
**verbose_name** 이 주어지지않으면 Django는 자동적으로 밑줄을 공백으로 바꾼 필드 속성의 이름으로 생성한다.

### validators

#### Field.validators

현재 필드에서 유효성을 여러개 검사한다.

```Python
from django.core.exceptions import ValidationError
from django.utils.translation import gettext_lazy as _

def validate_even(value):
	if value % 2 != 0:
		raise ValidationError(
			_('%(value)s is not an even number'),
			params={'value':value},
		)

from django.db import models

class Mymodel(models.Model):
	even_field = models.IntegerField(validators=[validate_even])
```
