---
layout: single
title: "[Python] Django Model"
published: true
categories:
  - Python
tag:
  - Project
  - Python
---

파이썬을 이용한 웹 애플리케이션을 만들 때 데이터는 필수적인 요소로, **Model**은 사용자 정보, 게시글 내용, 상품 목록 등 다양한 데이터를 효율적으로 다루기 위한 개념이다.

## 1. Django Model이란?

Django Model은 **ORM(Object-Relational Mapping)**의 한 형태다. ORM은 객체 지향 프로그래밍 언어의 객체와 관계형 데이터베이스의 테이블을 자동으로 매핑해주는 기술을 말한다.

간단히 말해, Django Model 클래스는 데이터베이스의 특정 테이블을 나타내고, 이 클래스의 인스턴스는 그 테이블의 특정 행(record)을 나타낸다. Model 클래스 안에 정의된 속성들은 테이블의 컬럼(field)이 된다.

## 2. Model 작성 시 포함되는 내용

Model은 주로 models.Model 클래스를 상속받아 파이썬 클래스로 작성된다. 하나의 Model 파일(models.py)에 여러 개의 Model 클래스를 정의할 수 있다.

- **필드 정의:** 데이터베이스 테이블의 각 컬럼을 나타낸다. Django는 다양한 필드 타입을 제공하며, 각 필드는 특정 데이터 타입을 가진다.
- **Meta 클래스:** Model에 대한 메타데이터, 즉 “Model 자체”에 대한 정보를 설정하는 내부 클래스다. 데이터베이스 테이블 이름, 정렬 순서 등을 지정할 때 사용된다.
- **__str __메소드:** 객체를 문자열로 표현할 때 어떻게 보일지 정의한다. Django 관리자 화면이나 대화형 셀에서 객체를 더 쉽게 식별할 수 있게 해준다.
- **사용자 정의 메소드:** 특정 Model 인스턴스와 관련된 비즈니스 로직이나 유틸리티 기능을 추가할 수 있다.

## 3. 주요 Model Field 타입

Django는 데이터베이스에서 사용되는 다양한 데이터 타입을 표현하기 위한 여러 필드 타입을 제공한다.

`null=True, blank=True, unique=True` 등 옵션을 갖을 수 있다.

| **필드 타입** | **설명** | **예시** |
| --- | --- | --- |
| CharField | 짧은 문자열. max_length 필수. | title = models.CharField(max_length=100) |
| TextField | 긴 문자열 (텍스트 블록). | content = models.TextField() |
| IntegerField | 정수. | count = models.IntegerField() |
| BooleanField | True/False 값. | is_published = models.BooleanField() |
| DateField | 날짜 (YYYY-MM-DD). | pub_date = models.DateField() |
| DateTimeField | 날짜와 시간 (YYYY-MM-DD HH:MM[:ss[.uuuuuu]][+HH:MM[:ss[.uuuuuu]]]). | created_at = models.DateTimeField() |
| ForeignKey | 다른 Model과의 다대일(Many-to-One) 관계 설정. | author = models.ForeignKey(User, on_delete=models.CASCADE) |
| ManyToManyField | 다른 Model과의 다대다(Many-to-Many) 관계 설정. | tags = models.ManyToManyField(Tag) |
| OneToOneField | 다른 Model과의 일대일(One-to-One) 관계 설정. | profile = models.OneToOneField(Profile, on_delete=models.CASCADE) |
| DecimalField | 정확한 소수점 값. max_digits와 decimal_places 필수. | price = models.DecimalField(max_digits=5, decimal_places=2) |
| EmailField | 이메일 주소. CharField(max_length=254)와 유사하나 유효성 검사 포함. | email = models.EmailField() |
| URLField | URL. CharField와 유사하나 유효성 검사 포함. | website = models.URLField() |

## 4. Models Options (Meta 클래스 활용)

### 주요 Meta 옵션

**db_table**

데이터베이스에서 사용할 테이블 이름을 명시적으로 지정한다. 지정하지 않으면 `앱이름_모델이름` 형식으로 자동 생성된다.

```python
class Article(models.Model):
    class Meta:
        db_table = 'blog_articles' # 기본값은 'blog_article'
```

**ordering**

객체 목록 가져올 때 기본 정렬 순서를 지정한다. 필드 이름 앞에 -를 붙이면 내림차순으로 정렬된다.

```python
class Article(models.Model):
    class Meta:
        ordering = ['-pub_date', 'title'] # 발행일 내림차순, 제목 오름차순
```

**verbose_name**

단일 객체를 나타내는 이름이다.

```python
class Article(models.Model):
    class Meta:
        verbose_name = '게시글'
```

**verbose_name_plural**

체 목록을 나타내는 이름이다.

```python
class Article(models.Model):
    class Meta:
        verbose_name_plural = '게시글 목록'
```

**unique_together**

주어진 필드들의 조합이 데이터베이스 테이블 전체에서 고유해야 함을 강제한다.

```python
class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    class Meta:
        unique_together = [['order', 'product']] # 한 주문에 같은 상품 중복 불가
```

**constraints**

데이터베이스 레벨에서 데이터의 유효성을 강제하는 제약 조건을 정의한다.

`UniqueConstraint`, `CheckConstraint`, `ExclusionConstraint` 등이 있습니다.

```python
from django.db.models import UniqueConstraint, CheckConstraint, Q, F

class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    discount_price = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)

    class Meta:
        constraints = [
            UniqueConstraint(fields=['name'], name='unique_product_name'),
            CheckConstraint(check=Q(price__gt=0), name='price_must_be_positive'),
            CheckConstraint(check=Q(discount_price__isnull=True) | Q(discount_price__lt=F('price')), name='discount_price_lt_price'),
        ]
```