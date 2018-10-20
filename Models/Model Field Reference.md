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

## Field types

### AutoField

#### class AutoField(\*\*options)

자동으로 증가하는 IntegerField이다. 보통 직접 사용하지 않고 별도로 지정하지 않으면 **Primary_key** 필드가 자동으로 모델에 추가된다.

### BigAutoField

#### class BigAutoField(\*\*options)

64bit Integer필드의 크기를 가지고, AutoField와 매우 유사하다.
1 ~ 9223372036854775807 까지 가능

### BigIntegerField

#### class BigIntegerField(\*\*options)
64bit Integer 필드를 가지고 이 필드의 기본 양식 위젯은 TextInput이다.
-9223372036854775808 ~ 9223372036854775807 까지 가능

### BinaryField

#### class BinaryField(\*\*options)

원시 이진 데이터를 저장하는 필드.
**Byte**, **ByteArray** 또는 **Memoryview**를 할당할 수 있다.

기본적으로 **BinaryField**는 **editable**을 False로 설정한다.
이 경우 ModelForm에는 포함될 수 없다.

**BinaryField**는 추가적인 옵션을 가진다.

**BinrayField.max_length**
필드의 최대 길이를 지정할 수 있다. **MaxLengthValidator**를 사용하여 Django의 유효성 검사에서 최대 길이가 적용된다.

### BooleanField

#### class BooleanField(\*\*options)

True 혹은 False 값을 가지는 필드

이 필드의 기본 양식 위젯은 **CheckboxInput** 이거나 **null=True** 인경우 **NullBooleanSelect** 이다.

**Field.default** 가 정의되어 있지 않으면 **BooleanField**의 기본값은 **None** 이다.

### CharField

#### class CharField(max_length=None, **options)

문자열 필드, 작은것부터 큰 사이즈의 문자열을 포함한다.
큰 양의 문자열일 경우 **TextField**를 사용한다.
기본 위젯은 **TextInput**이다.

**CharField** 에는 필수 인자가 하나 있다.

**CharField.max_length**
필드의 최대 길이를 정한다. max_length는 데이터베이스 레벨과 Django가 MaxLengthValidator를 사용하여 유효성을 검사할 때 적용된다.

### DateField

#### class DateField(auto_now=False, auto_now_add=False, \*\*options)

date는 Python의 **datetime.date** 인스턴스를 사용하여 나타낸다. 몇가지 추가 옵션 인수가 존재한다.

**DateField.auto_now**

개체가 저장될때마다 필드의 값을 지금으로 자동 설정한다.
"마지막으로 수정 된" 시간을 저장할 때 유용한다.
재정의 할 수 있는 기본값이 아니다.

이 필드는 **Model.save()** 를 호출할 때 자동으로 업데이트된다. **Query.update()** 와 같은 다른 방법으로 다른 필드를 업데이트 할 때 업데이트 되지 않지만 이와 같은 업데이트에서 필드의 사용자 지정 값을 지정할 수는 있다.

**DateField.auto_now_add**

객체가 처음 생성될 때 필드를 자동으로 현재날짜로 사용된다. 재정의 할 수 있는 기본값이 아니다. 따라서 이 필드의 기본 값을 설정하더라도 무시된다. 이 내용은 **auto_now_add=False** 를 사용한 후 다음과 같이 정의할 수 있다.

- default=date.today - *from datetime.date.today()*
- default=timezone.now - *from django.utils.timezone.now()*

이 필드의 기본 양식 위젯은 **TextInput** 이다. 관리자는 Javascript 캘린더와, 오늘을 추가할 수 있는 바로가기 위젯을 생성해준다. **invalid_date** 오류 메세지 키가 추가로 포함된다.


> 위의 **auto_now, auto_now_add** 는 상호 배타적이므로, 두가지 옵션을 함께 사용하면 오류가 발생한다.
> 위의 두 옵션 중 하나를 사용하게 되면 editable=False, blank=True가 설정된다.

### DateTimeField

#### class DateTimeField(auto_now=False, auto_now_add=False, \*\*options)

Python에서 **datetime.datetime** 인스턴스로 표현되는 날짜아 시간. **DateTime** 과 동일한 추가 인수를 사용한다.

### DecimalField

#### class DecimalField(max_digits=None, decimal_places=None, \*\*options)

고정 소수점 십진수로 파이썬에서 **Decimal** 인스턴스로 나타낸다. **DecimalValidator** 를 사용하여 입력의 유효성을 검사한다.

두가지 필수 인자를 가진다.

**DecimalField.max_digits**
숫자에 허용되는 최대 자릿수이다. 이 수는 **decimal_places**보다 크거나 같아야한다.

**DecimalField.decimal_places**
숫자와 함께 저장할 소수 자릿수이다.

예를 들어, 최대 999까지 소수점 이하 2 자리로 저장하려면 다음을 사용한다.
```python
models.DecimalField(..., max_digits=5, decimal_places=2)
```

이 필드의 기본 양식 위젯은 **localize** 가 False 일 때 **NumberInput** 이고 그렇지 않으면 **TextInput** 이다.

### DurationField

#### class DurationField(\*\*options)

기간을 저장하는 필드 - Python의 **timedelta** 를 모델로함.
PostgreSQL을 사용할 때 사용되는 데이터 타입은 **Interval** 이고 Oracle 에서 데이터 타입은 INTERVAL DAY(9) TO SECOND(6)이다. 그렇지 않으면 마이크로초의 **BIGINT** 가 사용된다.

> DurationField를 사용한 산술은 대부분의 경우 작동한다. 그러나 PostgreSQL 이외의 모든 데이터베이스에서 Duration field의 값을 DateTimeField의 산술 인스턴스와 비교하는 것은 작동하지 않는다.

### EmailField

#### class EmailField(max_length=254, \*\*options)

**EmailValidator** 를 사용하여 이메일 유효성을 검사하는 **CharField** 이다.

### FileField

#### class FileField(upload_to=None, max_length=100, \*\*options)

파일 업로드 필드이다.

> primary_key 인자는 지원되지 않고, 사용할 경우 오류를 발생한다.

두가지 선택적 옵션이 존재한다.

**FileField.upload_to**

이 속성은 업로드 디렉토리와 파일 이름을 설정하는 방법을 제공하며, 두 가지 방법으로 설정할 수 있다. 두 경우 모두 값은 **Storage.save()** 메서드에 전달된다.

**strftime()** 형식을 사용하는 문자열을 명시하면 날짜/시간 형식의 문자열로 대체된다.

```python
class MyModel(models.Model):
	#파일은 MEDIA_ROOT/uploads 에 업로드 될 것이다.
	upload = models.FileField(upload_to='uploads/')
	# 파일이 /MEDIA_ROOT/uploads/년/월/일 형식으로 업로드 될 것이다.
	upload= models.FileField(upload_to='uploads/%Y/%m/%d/')
```

기본 **FileSystemStorage** 를 사용하는 경우 문자열 값이 **MEDIA_ROOT** 경로에 추가되어 업로드 된 파일이 저장될 로컬 파일 시스템의 위치가 형성된다.

**upload_to** 는 함수와 같은 호출 가능 함수를 사용할 수 있다. 이것은 파일 이름을 포함하여 업로드 경로를 얻기 위해 사용된다. 이 호출 가능 객체는 두개의 인수를 받아 들여 저장소 시스템에 전달되는 유닉스 스타일 경로(슬래시 포함)을 반환해야한다. 두 가지 인수는 다음과 같다.

Argument | Description
---------|------------
instance|FileField가 정의 된 모델의 인스턴스. 보다 구체적으로 현재 파일이 첨부되는 특정 인스턴스이다. 대부분의 경우 이 개체는 아직 데이터베이스에 저장되지 않았으므로 **AutoField** 를 사용하는 경우 기본 키 필드 값이 아직 없을 수 도 있다.
filename|원래 파일에 주어진 이름. 최종 목적지 경로를 결정될 때 고려할 수 있고 고려하지 않을 수 있다.

```python
def user_driectory_path(instance, filename):
	# 파일이 MEDIA_ROOT/user_<id>/<filename> 에 업로드 될것이다.
	return 'user_{0}/{1}'.format(instance.user.id, filename)

class MyModel(models.Model):
	upload = models.FileField(upload_to=useuser_driectory_path)
```

**FileField.storage**
파일의 저장 및 검색을 처리하는 저장 storage 객체이다.

이 필드의 기본 양식 위젯은 **ClearableFileInput** 이다.

모델에서 **FileField** 또는 **ImageField** 를 사용하면 몇 단계가 수행된다.

1. 설정 파일에서 Django가 업로드 된 파일을 저장할 디렉토리의 전체 경로로 **MEDIA_ROOT** 를  정의해야한다. 성능향상을 위해 이러한 파일은 데이터베이스에 저장되지 않는다. **MEDIA_URL** 을 해당 디렉터리의 기본 공개 URL로 정의한다. 이 디렉터리가 웹 서버의 사용자 계정이 사용가능한지 확인한다.
2. **FileField** 또는 **ImageField** 를 추가하고 **upload_to** 옵션을 정의하여 업로드 된 파일에 사용할 **MEDIA_ROOT** 의 하위 디렉터리를 정의한다.
3. 데이터베이스에 저장되는 것은 모두 파일의 **MEDIA_ROOT** 로부터 상대경로이다. Django에서 제공하는 url 속성을 사용할 수도있다. 예를 들어 **ImageField** 의 이름이 **mug_shot** 인 경우 **{{ object.mug_shot.url }}** 로 템플릿에서 이미지의 절대 경로를 가져올 수 있다.

업로드 된 파일의 디스크상 파일이름이나 파일의 크기를 가져오려면 각각 **name** 및 **size** 속성을 사용할 수 있다.

업로드된 파일의 상대 URL 경로를 얻고싶으 url 속성을 사용하여 얻을 수 있다. 내부적으로 **Storage** 클래스의 **url()** 메서드를 호출한다.

업로드 된 파일을 다룰 때마다 업로드 할 파일과 파일의 유형에 주의를 기울여 보안 허점을 피해야한다. 업로드 된 모든 파일의 유효성을 검사하여 파일이 관리자가 허용한 것임을 확신하게 해야한다. 예를들어 누군가가 검증없이 파일을 웹 서버의 문서 루트에 있는 디렉터리에 업로드하도록 허용한다면 누군가가 CGI 또는 PHP 스크립트를 업로드하고 사이트의 URL에 방문하여 해당 스크립트를 실행할 수 있다.

또한 업로드 된 HTML 파일은 브라우저가 실행할 수 있기 때문에 XSS 또는 CSRF 공격과 동일한 보안위협이 될 수 있다.

**FileField** 인스턴스는 기본 최대 길이가 100자인 varchar 열로 데이터베이스에 만들어진다. 다른 필드와 마찬가지로 **max_length** 인수를 사용하여 최대 길이를 변경할 수 있다.

### FileField 와 FieldFile

#### class FieldFile

모델에서 **FileField** 에 접근하면 **FieldFile** 인스턴스가 기본 파일에 액세스하기 위한 프록시로 제공된다.

**FieldFile** 의 API는 **FILE** API를 미러링한다. 하나 주요 차이점이 있는데, 클래스에 의해 래핑된 객체는 반드시 파이썬 내장 파일 객체를 감싸는 래퍼일 필요는 없다. 대신 **Storage.open()** 메서드의 결과를 둘러싼 래퍼이다. 이 메서드는 **File** 객체일 수 있고 **FILE** API의 사용자 정의 저장소 구현 일 수도 있다.

**read()** 및 **write()** 와 같이 **FILE** 에서 상속된 API 외에도 **FieldFile** 에는 기본 파일과 상호 작용하는 데 사용할 수 있는 몇가지 메서드가 포함되어 있다.

>  이 클래스의 두 가지 메소드 인 save ()와 delete ()는 기본적으로 연관된 FieldFile의 모델 객체를 데이터베이스에 저장한다.

**FieldFile.name**

**FileField**에 연결된 **Storage** 로부터의 상대경로를 포함하는 파일 이름

**FieldFile.size**

**Storage.size()** 함수의 결과값

**Fieldfile.url**

**Storage** 클래스의 **url()** 함수를 호출하여 파일의 상대 URL에 접근하는 읽기 전용 프로퍼티

**FieldFile.open(mode='rb')**

지정된 모드에서 인스턴스와 관련된 파일을 열거나 다시 연다. 표준 파이썬 **open()** 메서드와는 달리, file descriptor 를 리턴하지 않는다.

기본 파일이 액세스할 때 암시적으로 열리므로 기본 파일에 대한 포인터를 재설정하거나 모드를 변경하는 경우를 제외하고 이 메서드를 호출할 필요 없다.

**FieldFile.close()

표준 파이썬 **file.close()** 메소드와 유사하게 동작하고 인스턴스와 관련된 파일을 닫는다.

**FieldFile.save(name, content, save=True)**

파일 이름과, 파일 내용을 가져와 필드의 Storage 클래스에 전달한 다음 저장된 파일을 모델 필드와 연결한다. 모델의 **FileField** 인스턴스에 파일을 수동으로 연결하려면 **save()** 메서드를 사용하여 해당 파일의 데이터를 유지한다.

두가지 필수 인수를 취한다.
**name** 은 파일의 이름, **content** 는 파일 내용을 포함하는 객체이다.
save 인수는 선택적이며 필드와 연관된 파일이 변경된 후 모델 인스턴스가 저장되는지 여부를 제어한다. 기본 값은 **True**

**content** 인수는 Python의 내장 파일 객체가 아닌 **django.core.files.File** 의 인스턴스야 한다. 다음과 같이 기존의 Python 객체에서 **FILE** 을 생성할 수 있다.

```Python
from django.core.files import File
# 파이썬의 내장함수로 파일을 가져옴
f = open('/path/to/hello.world')
myfile = File(f)
```

혹은 다음과 같이 파이썬 문자열로 만들 수 있다.

```Python
from django.core.files.base import ContentFile
myfile = ContentFile("Hello world")
```

**FieldFile.delete(save=True)**

이 인스턴스와 관련된 파일을 삭제하고 필드의 모든 특성을 지운다. 이 메서드는 호출될 때 파일이 열리면 닫는다.

**save** 인수는 선택적이며, 필드와 연관된 파일이 삭제된 후 모델 인스턴스가 저장되는지 여부를 제어한다.
기본 값은 **True**

모델을 삭제하면 관련 파일이 삭제되지 않는다. 해당 인스턴스와 관련된 파일을 정리해야하는 경우 직접 처리해야한다.

### FilePathField

#### class FilePathField(path=None, match=None, recursive=False, max_length=100, \*\*options)

파일시스템의 특정 디렉터리 내 파일 이름으로 제한된 CharField
세가지 특수한 인자를 가지며, 첫 인자는 필수이다.

**FilePathField.path**

필수.
`FilePathField`가 선택해야하는 디렉터리에 대한 파일 시스템의 절대 경로. 예 ) `"home/images"`

**FilePathField.match**

선택사항
문자로된 정규표현식,`FilePathField`가 파일이름을 필터할 때 사용한다. 정규표현식은 전체 경로가 아닌, 파일 이름에 적용된다.

**FilePathField.recursive**

선택사항
Boolean 속성, 기본값은 `False`
`path` 의 하위 디렉터리가 모두 포함될지 여부를 지정한다.

**FilePathField.allow_files**

선택사항
Boolean 속성, 기본값은 `True`
지정된 위치의 파일을 포함할지 여부를 지정한다. 이 속성이나 `allow_folders` 둘중 하나는 `True`여야한다.

**FilePathField.allow_folders**

선택사항
Boolean 속성, 기본값은 `False`
지정된 위치의 폴더를 포함할지 여부를 지정한다. 이 속성이나 `allow_files` 둘중 하나는 `True`이여야한다.

물론 이러한 함수들을 함께 사용할 수 있다.

하나의 잠재적 문제는 전체 경로가 아닌 기본 파일 이름에 `match`가 적용된다는 것이다.

```Python
FilePathField(path='/home/images', match='foo.*', recursive=True)
```

`match`가 기본 파일 이름에 적용되기 때문에
`/home/images/foo.png` 는 맞지만, `/home/images/foo/bar.png`는 맞지 않다.

`FilePathField` 인스턴스는 100자인 `varchar` 열로 데이터베이스에 만들어진다. 다른 필드와 마찬가지로 `max_length` 인수를 사용하여 최대 길이를 변경 가능하다.

### FloatField

#### class FloatField(\*\*options)

파이썬의 `float` 인스턴스로 표현된 부동 소수점 숫자

기본 form 위젯은 `localize`가 `False`이면 `NumberInput`, 아니면 `TextInput`

> DecimalField와 FloatField는 컴퓨터 내부에서 수를 다르게 나타낸다.

### ImageField

#### class ImageField(upload_to=None, height_field=None, width_field=None, max_length=100, \*\*options)

`FileField`의 모든 속성과 메서드를 상속하지만, 업로드된 객체가 유효한 이미지인지 검사한다.

게다가 `FileField`에 사용할 수 있는 속성외에 `height`, `width` 속성이 존재한다.

이러한 속성에 대한 쿼리를 용이하게 하기 위해서 두가지 선택 인수가 존재한다.

**ImageField.height_field**
모델 인스턴스가 새로 저장될 때 마다 이미지의 높이로 자동으로 채워지는 모델 필드의 이름이다.

**ImageField.width_field**
모델 인스턴스가 새로 저장될 때 마다 이미지의 길이로 자동으로 채워지는 모델 필드의 이름이다.

`Pillow` 라이브러리가 필요하다.

`ImageField` 인스턴스는 기본 최대 길이가 100자인 varchar 열로 데이터베이스에 만들어진다. 다른 필드와 마찬가지로 `max_length` 인수를 사용하여 최대 길이를 변경할 수 있다.

기본 위젯은 `ClearableFileInput`

### IntegerField이다

#### class IntegerField(\*\*options)

\-2147483648 부터 2147483647까지 저장할 수 있는 정수 필드
`MinValueValidator`와 `MaxValueValidator`를 사용하여 데이터베이스가 지원하는 값에따라 유효성을 검사한다.

기본 위젯은 `localize`가 `False`일 때 `NumberInput`, 아니면 `TextInput`

### GenericIPAddressField

#### class GenericIPAddressField(protocol='both', unpack_ipv4=False, \*\*options)

IPv4, IPv6 를 표현하기 위한 타입.
기본 위젯은 `TextInput`

IPv6 주소 정규화는 `RFC 4291 # section-2.2` 를 따르며, IPv4는 그 섹션의 3번째 단락에서 제안된 포맷을 따른다.
IPv4 의 표현방식 `::ffff:192.0.2.0`
`2001:0::0:01`은 `2001::1`로 간소화되고
`::ffff:0a0a:0a0a`는 `::ffff:10.10.10.10`로 정규화된다.
모든 문자는 소문자화 된다.

**GenericIPAddressField.protocol**

지정된 프로토콜에대한 유효한 입력을 제한한다. 허용되는 값은 `both(기본값)`, `IPv4`, `IPv6`. 대소문자를 구분하지 않는다.

**GenericIPAddressField.unpack_ipv4**

이 옵션을 키면 위에서 `RFC_4291`의 포맷인 `::ffff:192.0.2.1`을 `192.0.2.1`로 바꿔 나타내준다. 기본은 `False`이며, 오직 `protocol`이 `both`일 때만 사용할 수 있다.


공백값을 허용하는 경우 공백 값은 NULL로 저장되므로 NULL 값을 허용해야 한다.

### NullBooleanField

#### class NullBooleanField(\*\*options)

`BooleanField`에 `null=True`와 한것과 같다.

### PositiveIntegerField

#### class PositiveIntegerField(\*\*options)

IntegerField와 같지만, 0부터 2147483647까지 나타낼 수 있게 한다. 이전 버전과의 호환성을 위해 0 값이 허용된다.

### PositiveSmallIntegerField

#### class PositiveSmallIntegerField(\*\*options)

PositiveIntegerField와 같지만 0부터 32767까지 나타낼 수 있게 한다.

### SlugField

#### class SlugField(max_length=50, \*\*options)

`Slug`는 신문의 용어이다. slug는 문자, 숫자, 밑줄 또는 하이픈만 포함하는 짧은 레이블이다. 이것들은 보통 URL에 사용된다.


`CharField`와 마찬가지로 `max_length`를 지정할 수 있다. `max_length`를 지정하지 않으면 50 의 기본길이를 가진다.

이 필드는 `Field.db_index`를 True로 설정되어 있다.

다른 값의 값을 기반으로 `SlugField`를 자동으로 미리 채우는 것이 유용하다. `prepopulated_field`를 사용하여 관리자에서 자동으로 이 작업을 수행할 수 있다.

검증을 위해 `validate_slug` 또는 `validate_unicode_slug`를 사용한다.

**SlugField.allow_unicode**

`True` ASCII 문자, Unicode 문자를 모두 허용한다. 기본은 `False`

### SmallIntegerField

#### class SmallIntegerField(\*\*options)

`IntegerField`와 같지만 \-32768 ~ 32767까지 허용한다.

### TextField

#### class TextField(\*\*options)

큰 텍스트 필드
기본 위젯은 `TextArea`

`max_length`를 지정하면 자동 생성 양식 필드의 `TextArea` 위젯에 반영된다. 그러나 모델 또는 데이터베이스 수준에는 적용되지 않는다. `max_length`를 사용하고싶으면 `CharField`를 사용하자.

### TimeField

#### class TimeField(auto_now=Flase, auto_now_add=False, \*\*options)

Python의 `datetime.time` 인스턴스로 나타내는 필드. 옵션은 `DateField` 의 옵션과 같다.

기본 위젯은 `TextInput`, admin은 Javascript Shortcut을 지원해준다.

### URLField

#### class URLField(max_length=200, \*\*options)

`CharField`로 나타내는 url `URLValidator`로 유효성을 검사한다.

기본 위젯은 `TextInput`

`CharField`의 하위 클래스와 같이 max_length 옵션을 가진다. 기본은 200

### UUIDField

#### class UUIDField(\*\*options)

python 의 `UUID` 클래스를 사용하여 해당 내용을 저장하기 위한 필드.
PostgreSQL을 사용하면 `uuid` 데이터 타입을 사용하고, 다른 SQL에서는 char(32)를 사용한다.

UUID는 `primary_key`에 대한 `AutoField` 대신 사용할 수 있는 방법이다. 데이터베이스는 UUID를 생성하지 않으므로 기본값으로 사용하는 것을 추천한다.

```python
import UUID
from django.db import models

class MyUUIDModel(models.Model):
	id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
```
이 때 UUID 인스턴스가 아니라, 호출가능 객체가 기본값으로 전달된다.

## Relationship Field

### ForeignKey

#### class ForeignKey(to, on_delete, \*\*options)

`Many-to-one` 관계를 나타내고, 두가지 필수 인자를 필요로 한다.
`recursive relationship` 관계를 나타내고싶으면 `models.ForeignKey('self', on_delete=models.CASCADE)` 를 사용한다.

아직 정의되지 않은 모델에 관계를 작성해야하는 경우 모델 오브젝트 자체가 아닌 모델 이름을 사용할 수 있다. (Object가 아닌 Str)

```python
from django.db import models

class Car(models.Model):
	manufacturer = models.ForeignKey(
		'Manufacturer',
		on_delete=models.CASCADE,
	)

class Manufacturer(models.Model):
	pass
```

`abstract models` 방식으로 정의된 관계는 모델이 구체적인 모델로 서브클래스화되고 추상 모델의 `app_label`과 관련이 없는 경우 해결된다.

```Python
from django.db import models

class AbstractCar(models.Model):
	manufacturer = models.ForeignKey(
		'Manufacturer',
		on_delete=models.CASCADE
	)

	class Meta:
		abstract = True

class Car(AbstractCar):
	pass
```

다른 응용 프로그램에 정의 된 모델을 참조하려면 전체 응용 프로그램 레이블이 있는 모델을 명시적으로 지정해야한다. 예를들어 Manufacturer 모델이 production 이라는 다른 응용 프로그램에 정의되어 있는 경우 다음을 사용해야한다.

```Python
class Car(models.Model):
	manufacturer = models.ForeignKey(
		'production.Manufacturer',
		on_delete=models.CASCADE,
	)
```

`lazy relationship` 이라고 불리는 이 종류의 참조는 두 application 간 `circular import dependencies` 종속성을 해결할 때 유용할 수 있다.

데이터베이스의 인덱스는 `ForeignKey`에 자동으로 작성된다. `db_index` `False`를 통해서 비활성화 할 수 있다. 조인이 아닌 일관성을 위해 외래 키를 작성하거나 부분 또는 다중 컬럼 index 와 같은 대체 index를 작성할 경우 index의 오버헤드를 피할 수 있다.

**Database Representation**

Django는 필드 이름에 "\_id"를 추가하여 데이터베이스 열 이름을 생성한다. 위 예제에서 `Car` 모델은 데이터베이스에 `manufacturer_id` 컬럼이 생성된다. (`db_column` 속성을 이용해 바꿀 수 있다.) 그러나 사용자지정 SQL을 작성하지 않으면 Django 코드에서 데이터베이스 열 이름을 처리할 필요가 없다. 항상 Django 모델 객체의 필드 이름을 다룰것이다.

---

**Arguments**

**ForeignKey** 는 세부정보를 정의하는 다른 인수를 허용한다.

**ForeignKey.on_delete**

`ForeignKey`가 참조하는 객체가 삭제되면 Django는 `on_delete` 인수로 지정된 SQL 제약 조건의 동작을 실행한다.

아래 내용은 참조하는 객체가 삭제되면 해당 열을 NULL로 지정하라는 문장이다.
```python
user = models.ForeignKey(
	User,
	models.SET_NULL,
	blank=True,
	null=True,
)
```

`on_delete`는 데이터베이스에 제약조건을 만들지 않는다.
해당 속성에 가능한 값은 `django.db.models`에서 찾을 수 있다.

- CASCADE
	계단식 삭제, Django는 SQL 제약조건에 ON DELETE CASCADE 를 실행하고, ForeignKey를 포함하는 객체를 삭제한다.

	`Model.delete()`는 관련 모델에서 호출되지 않지만 삭제된 모든 객체에 대해 `pre_delete` 및 `post_delete` 신호가 전송된다.

- PROTECT
	`django.db.IntegrityError`의 하위 클래스인 `ProtectedError`를 발생시켜 참조된 객체의 삭제를 방지한다.

- SET_NULL
	`null=True`가 있다면 `ForeignKey` 필드를 null로 지정한다.

- SET_DEFAULT
	기본값으로 설정된다.

- SET()
	호출가능 객체가 전달된 경우, 호출의 결과를 가져온다.

```python
from django.conf import settings
from django.contrib.auth import get_user_model
from django.db import models

def get_sentinel_user():
	return get_user_model().objects.get_or_create(username='deleted')

class MyModel(models.Model):
	user = models.ForeignKey(
		settings.AUTH_USER_MODEL,
		on_delete=models.SET(get_sentinel_user),
	)
```

- DO_NOTHING
	아무런 행동도 하지 않는다. 데이터베이스 필드에 제약조건 `ON_DELETE`를 수동으로 추가하지 않으면 `IntegrityError`가 발생한다.

**ForeignKey.limit_choices_to**
이 필드가 `ModelForm` 또는 관리자를 사용하여 렌더링될 때 이 필드에 대한 사용 가능한 선택 사항에 대한 제한을 설정할 수 있다. (기본적으로 모든 쿼리셋의 오브젝트를 선택할 수 있다.) dict, Q 객체, dict, Q 객체를 리턴하는 호출가능 객체를 사용가능하다.

```python
staff_member = models.ForeignKey(
	User,
	on_delete=models.CASCADE,
	limit_choices_to={'is_staff': True},
)
```

`ModelForm`의 해당 필드가 `is_staff`가 `True`인 사용자만 나열하도록 한다.
예를들어 Python datetime 모듈과 함께 사용하여 날짜 범위에 따른 선택을 제한하는 경우, 호출 가능 형식이 유용하다.

```python
def limit_pub_date_choice():
	return {'pub_date__lte': datetime.date.utcnow()}

limit_choices_to = limit_pub_date_choice
```

`limit_choices_to`가 복잡한 쿼리에 유용한 `Q object`를 반환하면 모델의 `ModelAdmin` 필드가 `raw_id_field`에 나열되지 않은 경우에만 관리자가 사용할 수 있는 선택 항목에 영향을 미친다.

> callable이 `limit_choices_to`에 사용되면, 새로운 인스턴스가 생성될 때마다 호출된다. 관리자나 관리 명령에 의해 모델이 검증될 때 호출될 수 있다. 관리자는 다양한 폼 입력 검증을 위해 쿼리세트를 구성하므로 호출 가능 항목이 여러번 호출 될 수 있다.

**ForeignKey.related_name**


관련 객체에서 이 객체에 대한 관계에 사용할 이름이다. `related_query_name`의 기본값이기도 하다. (타겟 모델이 역참조할 때 사용할 이름) 추상모델에서 `related_name`을 사용할 때 특별한 구문을 사용할 수 있다. (같은 역참조가 발생하면 오류를 발생하기 때문에 현재 클래스 이름, 앱 이름을 사용하는 특별한 구문이 필요하다.)

역참조를 만들게하고싶지 않는다면 `related_name`을 `+`로 만든다.

```python
user = models.ForeignKey(
	User,
	on_delete=models.CASCADE,
	related_name='+',
)
```

**ForeignKey.related_query_name**

역방향 필터에서 사용할 이름. 설정되어 있지 않은경우 `related_name` 이름 또는 `default_related_name`을 상속받고, 그렇지 않은경우 기본값은 모델의 이름의 소문자화이다.

```python
class Tag(models.Model):
	article = models.ForeignKey(
		Article,
		on_delete=models.CASCADE,
		related_name='tags',
		related_query_name='tag',
	)
	name = models.CharField(max_length=255)

Article.objects.filter(tag__name='important')
```

`related_name`과 마찬가지로 `related_query_name`에도 특수한 구문을 사용할 수 있다.

**ForeignKey.to_field**

관계가 있는 관련 객체의 필드, 기본적으로 Django의 관련 객체의 기본키를 사용하지만, 다른 필드를 사용하는 경우 해당 필드는 `unique=True`이어야한다.

**ForeignKey.db_constraint**

이 외래키에 대해 데이터베이스에 제약조건을 만들어야하는지 여부를 제어한다.
기본값은 `True`, 거의 확실히 이것만 쓴다.
`False` 일시 데이터 무결성이 매우 나빠진다.

**ForeignKey.swappable**

이 `ForeignKey`가 swappable한 모델을 가리키는 경우 마이그레이션 프레임워크의 반응을 제어한다.

(?)


### ManyToManyField

#### class ManyToManyField(to, \*\*options)

ManyToMany 관계를 나타낸다. `recursive`, `lazy relationships` 관계를 포함하여 모델 클래스 위치 인자가 필요하다.


`Related Objects`는 `RelatedManage`를 통해 추가, 제거 또는 생성할 수 있다.

**Database Representation**

Django는 `many-to-many` 관계를 표현하기 위해 중간 조인 테이블을 생성한다. 기본적으로 이 테이블의 이름은 필드의 이름과 그 안에 들어있는 모델의 테이블 이름을 사용하여 생성하게 된다. 일부 데이터베이스는 특정 길이 이상의 테이블 이름을 지원하지 않으므로 이러한 테이블 이름은 자동으로 64자로 잘리고 고유성 해시가 사용된다. 즉, `author_book_9cd4f`와 같은 테이블이름이될 수 있다. db_table 옵션을 사용하여 조인 테이블의 이름을 수동으로 제공할 수 있다.

---

**Arguments**

**ManyToManyField** 는 관계함수 제어를 위한 인자를 제공한다.

**ManyToManyField.related_name**
**ManyToManyField.related_query_name**
**ManyToManyField.limit_choices_to**
 `through` 매개변수를 이용하여 지정한 중간 테이블이 있는 `ManyToManyField`에서 사용될 때 아무런 영향을주지 못한다.
**ManyToManyField.symmetrical**
	첫 위치인자로 'self'를 주게 되는 것.
	Django는 이 모델을 처리할 때 `ManyToManyField`가 있음으로 식별하므로 Person 클래스에 `person_set` 속성을 추가하지 않는다. 대신, `ManyToManyField`는 대칭이라고 가정한다.

	자신과 `ManyToMany` 관계에서 대칭을 원하지 않으면 `False`로 설정한다. 이렇게하면 `Django`가 역방향 관계에 대한 설명을 추가하게 되어 `ManyToMany`가 비대칭이 되게 할 수 있다.

**ManyToManyField.through**
Django는 다대다 관계를 관리 할 테이블을 자동으로 생성한다.
그러나 중개 테이블을 수동으로 지정하려면 through 옵션을 사용하여 사용할 중간 테이블을 나타내는 Django 모델을 지정할 수 있다.

이 옵션의 가장 일반적인 용도는 추가 데이터를 다대다 관계와 연관시키려는 경우이다.

만약 `through`에 모델을 명시하지 않으면, 연관을 유지하기위해 생성된 암시적 `through` 모델 클래스가 있다. 이 모델은 3개의 필드를 갖는다.

원본 및 대상 모델이 다른경우 다음 필드가 생성된다.
- id
- <containing_model>\_id
- <other_model>\_id

자신의 모델을 가리키는 경우
- id
- from\_<model>\_id
- to\_<model>\_id

이 클래스는 일반 모델과 같이 특정 모델 인스턴스에 대한 관련 레코드를 쿼리하는데 사용할 수 있다.

**ManyToManyField.through_fields**

사용자 지정 매개 모델이 지정된 경우에만 사용된다. Django는 일반적으로 다대다 관계를 자동으로 설정하기 위해 사용할 중개 모델의 필드를 결정한다.

```python
from django.db import models

class Person(models.Model):
    name = models.CharField(max_length=50)

class Group(models.Model):
    name = models.CharField(max_length=128)
    members = models.ManyToManyField(
        Person,
        through='Membership',
        through_fields=('group', 'person'),
    )

class Membership(models.Model):
    group = models.ForeignKey(Group, on_delete=models.CASCADE)
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    inviter = models.ForeignKey(
        Person,
        on_delete=models.CASCADE,
        related_name="membership_invites",
    )
    invite_reason = models.CharField(max_length=64)
```

`Membership`은 `Person`(person 과 inviter) 두개의 왜래키를 가지고 있어 관계가 모호해지고 장고는 사용할 관계를 알 수 없다. 이 경우 Django가 `through_fields`를 사용하여 외래키를 명시적으로 지정해야한다.

`through_fields`는 2-tuple`('field1', 'field2')`를 허용한다. 여기서 `field1`은 `ManyToManyField`가 정의된 모델의 외래 키 이름(이 경우 group)이고.`field2`의 경우 목표 모델의 외래 키(이 경우 person)의 이름이다.

다대다 관계에 참여하는 모델 중 하나의 중개 모델에 둘 이상의 외래 키가 있는 경우 `through_fields`를 지정해주어야한다. 중간모델이 사용되고 모델에 외래키가 두개 이상이거나 Django가 사용할 두개(?)를 명시적으로 지정하려는 경우에도 재귀관계에도 적용된다.

중간 모델을 사용하는 재귀 관계는 항상 비대칭, `symmetrical=False`로 정의되므로 이때 `Source` 와 `Target`이 존재한다. 이 때 `field1`은 `Source`로 취급되고 `Target`은 `Field2`로 취급된다.

**ManyToManyField.db_table**

MnayToMany 데이터 저장을 위한 테이블의 이름을 지정한다. 제공되지않으면, Django에서 관계를 정의하는 모델의 테이블과 필드 자체의 이름을 기반으로 기본 이름을 사용한다.

**ManyToManyField.db_constraint**
중간 테이블의 왜래 키에 대해 데이터베이스에 제약조건을 생성하는지 여부를 제어한다. 기본은 True, 대부분 이 설정을 사용한다.
`False`롤 설정하게되면 무결성이 나빠지게 된다. db_constraint와 through를 모두 사용하는 것은 오류이다.

**ManyToManyField.swappable**

(?)

### OneToOneField

#### class OneToOneField(to, ono_delete, parent_link=False, \*\*options)

일대일 관계, 개념적으로 이는 `unique=True`인 `ForeignKey`와 비슷하지만 `reverse`참조시 하나의 객체를 직접 반환한다.
다른 모델을 확장하는 모델의 기본키로 가장 유용하다. 다중 테이블 상속은 하위 모델에서 상위 모델에 대한 암시적 일대일 관계를 추가하여 구현한다.

위치인자가 하나 필요하다. 재귀, `lazy relationships` 옵션을 포함한 `ForeignKey`와 똑같이 작동한다.

`OneToOneField`에 `related_name` 인수를 지정하지 않으면 Django는 현재 모델의 소문자 이름을 기본값으로 사용한다.

```python
from django.conf import settings
from django.db import models

class MySpecialUser(models.Model):
    user = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
    )
    supervisor = models.OneToOneField(
        settings.AUTH_USER_MODEL,
        on_delete=models.CASCADE,
        related_name='supervisor_of',
    )
```

최종 사용자 모델에는 다음의 속성이 존재한다.

```python
>>> user = User.objects.get(pk=1)
>>> hasattr(user, 'myspecialuser')
True
>>> hasattr(user, 'supervisor_of')
True
```
관련 테이블 항목이 없는 경우 역방향 참조에 액세스 할 때 DoesNotExist 예외가 발생한다. `MySpecialUser`에 `supervisor`가 존재하지 않는 경우

```python
>>> user.supervisor_of
Traceback (most recent call last):
    ...
DoesNotExist: User matching query does not exist.
```

게다가 `OneToOneField`는 `ForeignKey`가 허용하는 추가인수와, 하나의 추가인수를 더 허용한다.

**OneToOneField.parent_link**

`True`이고, 다른 구체적 모델을 상속받은 모델을 사용할 때 서브클래싱을 통해 암시적으로 만들어지는 추가적 OneToOne필드가 아니라 이 필드는 부모 클래스에 대한 링크로 사용되여야 함을 나타낸다.

## Field API reference

### class Field

필드는 데이터베이스 테이블 열을 나타내는 추상 클래스이다. Django는 데이터베이스 테이블을 만들기위해 `db_type()`을 참조하고 Python 타입과 database 타입을 매핑시키기 위해 `get_prep_value()`를 사용하고, 반대 경우엔 `from_db_value()`를 사용한다.

따라서 `Field`는 Django API, 특히 models와 querysets의 기본적인 요소이다.

모델에서 필드는 클래스 속성으로 인스턴스화되고 특정 테이블의 열을 나타낸다. `null`과 `unique`같은 속성과 Django 필드 값을 데이터베이스 특정 값으로 매핑하는 메서드를 가지고 있다.

`Field`는 `RegisterLookupMixin`의 하위클래스이므로 `Transform` 및 `Lookup`을 `QuerySet`에서 사용할 수 있다. (예: `field_name__exact='foo'`). 모든 `bulit-in lookups`는 기본적으로 등록되어 있다.

모든 Django의 `CharField`처럼 buit-in 필드는 특정 필드의 구현이다. 사용자 정의 필드가 필요하면 기본 제공 필드를 서브 클래스로 만들거나 필드를 처음부터 작성할 수 있다.

**Description**

	해당 필드의 자세한 설명

	`Description` 형식은 다음과 같다.
	```python
		description = _("String (up to %(max\_length)s)")
	```

	필드의 `__dict__` 안에 해당 인자가 넣어지게 된다.

장고는 `Field`를 데이터베이스 관련 타입으로 매핑하기위해 메소드를 제공한다.

**get_internal_type()**
	백엔드에서 특수한 목적으로 사용되는 현재 필드의 이름을 나타내는 문자열을 반환한다. 기본적으로, 클래스 이름을 반환한다.

**db_type(connection)**
	제공된 인자 `Field`에 매핑되는 데이터베이스에서 사용하는 데이터 타입을 반환한다.

**rel_db_type(connection)**
	`ForeignKey`, `OneToOne`로 나타내는`Field`에 매핑되는 데이터베이스에서 사용하는 데이터 타입을 반환한다.


장고에서는 데이터베이스와 상호작용할 수 있는 세가지 상황이 존재한다.
	- 데이터베이스로 쿼리할 때 (Python Value -> database backend value)
	- 데이터베이스에서 데이터를 로드할 때 ( database backend value -> Python Value )
	- 데이터베이스로 저장할 때 (Python Value -> database backend value)

**get_prep_value(value)**
	`value`는 모델 속성의 현재 값이며, 메서드는 쿼리의 매개변수로 사용하기 위해 준비된 형식으로 데이터를 반환해야한다.

**get_db_prep_value(value, connection, prepared=False)**
	`value`를 백엔드 값으로 변환한다. 기본적으로 `prepared=True`일경우 값을 반환하고, `False`일경우 `get_prep_vlaue()`를 반환한다.

**from_db_value(value, expression, connection)**
	데이터를 로드할 때 사용한다.
	데이터베이스에서 반환한 값을 파이썬 객체로 변환한다. `get_prep_value()`의 반대이다.
	이 메서드는 데이터베이스 백엔드가 이미 올바른 파이썬 유형을 리턴하거나 백엔드 자체가 변환을 수행하므로 대부분의 built-in 필드에서는 사용되지 않는다.

**get_db_prep_save(value, connection)**

	`get_db_prep_value()`와 같지만 필드 값을 데이터베이스에 저장해야할 때 호출된다. 기본적으로 `get_db_prep_value()`를 반환한다.

**pre_save(model_instance, add)**

	저장 전 값을 미리 준비하기위해 `get_db_prep_save()`보다 먼저 호출되는 메서드. ( 예: `DateField.auto_now`)

	model_instance는 이 필드가 속하는 인스턴스이며, add는 인스턴스가 데이터베이스에 처음 저장되는지 여부이다.
	이 필드의 model_instace에서 적절한 속성 값을 반환해야한다. 속성 이름은 `self.attname`에 있다.

**to_python(value)**

	필드는 serialize 것처럼 가끔 다른 유형을 값을 받게되는데,
	값을 올바른 Python 객체로 변환한다. `value_to_string()`의 역으로 작동하며, `clean()`에서도 호출된다.

**value_from_object(obj)**

	필드는 데이터베이스에 저장하기위해 serialize 하는 법을 알아야한다.
	주어진 모델 인스턴스의 필드 값을 반환한다.
	이 메서드는 종종 `value_to_string()`에 의해 사용된다.

**value_to_string(obj)**

	`obj`를 문자열로 바꾼다. 필드 값을 직렬화하는데 사용된다.

**formfield(form_class=None, choices_form_class=None, **kwargs)**

	`ModelForm`에 대한 필드의 기본 `django.forms.Field`를 반환한다.

	기본적으로 `form_class`와 `choices_form_class`는 None이면 `CharField`를 사용한다.
	만약 필드에 `choices`필드가 존재하고, `choices_form_class`가 지정되지 않은 경우, `TypedChoiceField`를 사용한다.

**deconstruct()**

	필드를 다시 만들 수 있는 충분한 정보가 있는 4-tuple을 반환한다.

	1. model의 필드 이름
	2. field의 import path ( 예: `django.db.models.IntegerField` ). 가벼운 버전이여야하므로, 덜 구체적인 것이 좋을 수 있다.
	3. 위치인자
	4. 키워드 인자

# Field attribute reference

모든 Field 인스턴스는 해당 동작으로 검사할 수 있는 여러 속성이 포함되어 있다. 필드의 기능에 따라 코드를 작성해야하는 경우, `isinstance` 대신 이러한 속성을 사용한다. 이 속성들은 `Model._meta API`과 함께 사용하여 특정 필드 유형에 대한 검색 범위를 좁힐 수 있다.사용자 정의 필드는 이러한 플래그를 구현해야한다.

## Attribute for Fields

### Field.auto_created

	모델 상속에 사용되는 `OneToOneField`같이 자동으로 만들어졌는지 여부를 나타내는 불리언 플래그

### Field.concrete

	필드에 연결된 데이터베이스 열이 있는지 여부를 나타내는 불리언 플래그

### Field.hidden

	(?)

### Field.is_relation

	다른 모델에 대한 차모가 포함되어있는지 나타내는 불리언 플래그

### Field.model

	필드가 정의된 있는 모델을 반환한다. 필드가 모델의 수퍼 클래스에 정의된 경우 모델은 인스턴스 클래스가 아닌 수퍼클래스를 참조한다.

## Attributes for fields with relations

이러한 속성은 카디널리티 및 관계의 기타 세부사항을 쿼리할 때 사용한다. 이 속성은 모든 필드에 있다.
관계 타입이 있을 경우 `Field.is_relation=True`일것이고, 불리언 값으로 나타낸다.

### Field.many_to_many
### Field.many_to_one
### Field.one_to_many
### Field.one_to_one

### Field.related_model
	필드가 관련된 모델을 가리킨다. 예를 들어 ForeinKey의 Author `ForeignKey(Author, on_dete=models.CASCADE)`. GeneriecForeignKey의 related_model은 항상 None이다.
