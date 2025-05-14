---
layout: single
title: "[Python] SQLite3"
published: true
categories:
  - Python
tag:
  - Project
  - Python
---

SQLite는 별도의 서버 구축 없이 파일 하나로 데이터베이스를 사용할 수 있다는 장점을 갖는다.

특히, Python은 표준 라이브러리로 sqlite3 모듈을 제공하여 SQLite 데이터베이스를 아주 쉽게 다룰 수 있도록 지원한다.

## 1. SQLite 데이터베이스 연결 (또는 생성)

`sqlite3.connect()`함수를 사용하여 인자로 데이터베이스 파일 경로를 지정한다. 만약 파일이 없을 경우 새로운 데이터베이스 파일을 생성한다.

```python
import sqlite3

# 데이터베이스 파일 경로 지정 (원하는 이름으로 변경 가능)
# :memory: 를 사용하면 메모리 상에서만 데이터베이스를 사용하고, 연결이 끊기면 사라집니다.
db_path = 'my_database.db'

try:
    conn = sqlite3.connect(db_path)
    print(f"데이터베이스에 성공적으로 연결되었습니다: {db_path}")
except sqlite3.Error as e:
    print(f"데이터베이스 연결 오류: {e}")

```

## 2. 커서(Cursor) 객체 생성

SQL 명령을 실행하기 위해서는 커서 객체가 필요하다.

연결 객체의 `cursor()` 메소드를 호출하여 커서 객체를 얻을 수 있다.

```python
cursor = conn.cursor()
```

## 3. 테이블 생성

SQL의 CREATE TABLE 문을 사용하며, 커서 객체의 `execute()` 메소드를 통해 실행한다.

```python
try:
    # 예시: 'users' 테이블 생성
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS users (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            age INTEGER
        )
    ''')
    print("테이블 'users'가 성공적으로 생성되었거나 이미 존재합니다.")
except sqlite3.Error as e:
    print(f"테이블 생성 오류: {e}")
```

여기서 `IF NOT EXISTS`는 테이블이 이미 존재하더라도 오류가 발생하지 않도록 방지해준다.

## 4. 데이터 삽입

INSERT INTO SQL 문을 사용하여 테이블에 데이터를 삽입한다.

### 단일 데이터 삽입

```python
try:
    user_name = 'Alice'
    user_age = 30
    cursor.execute("INSERT INTO users (name, age) VALUES (?, ?)", (user_name, user_age))
    print(f"'{user_name}' 데이터 삽입 성공")
except sqlite3.Error as e:
    print(f"데이터 삽입 오류: {e}")

```

### 다중 데이터 삽입

여러 개의 데이터를 삽입할 때는 `executemany()` 메소드를 사용하는 것이 효율적이다.

```python
try:
    users_data = [
        ('Bob', 25),
        ('Charlie', 35),
        ('David', 28)
    ]
    cursor.executemany("INSERT INTO users (name, age) VALUES (?, ?)", users_data)
    print("여러 데이터 삽입 성공")
except sqlite3.Error as e:
    print(f"데이터 삽입 오류: {e}")

```

## 5. 데이터 조회

SELECT SQL 문을 사용하여 테이블의 데이터를 조회한다.

`fetchone()`, `fetchall()`, `fetchmany()` 메소드를 사용하여 결과를 가져온다.

- fetchone(): 하나의 결과 행을 튜플로 반환한다. 더 이상 결과가 없으면 None을 반환한다.
- fetchall(): 모든 결과 행을 튜플의 리스트로 반환한다.
- fetchmany(size): 지정된 개수만큼의 결과 행을 튜플의 리스트로 반환한다.

```python
try:
    # 모든 데이터 조회
    cursor.execute("SELECT * FROM users")
    all_users = cursor.fetchall()
    print("\n--- 모든 사용자 데이터 ---")
    for user in all_users:
        print(user)

    # 특정 조건의 데이터 조회
    cursor.execute("SELECT name, age FROM users WHERE age > ?", (29,))
    older_users = cursor.fetchall()
    print("\n--- 29세 초과 사용자 데이터 ---")
    for user in older_users:
        print(user)

    # 하나의 데이터만 조회
    cursor.execute("SELECT * FROM users WHERE name = ?", ('Alice',))
    alice = cursor.fetchone()
    print("\n--- Alice 데이터 ---")
    print(alice)

except sqlite3.Error as e:
    print(f"데이터 조회 오류: {e}")

```

## 6. 데이터 수정

UPDATE SQL문을 사용하여 데이터를 수정한다.

```python
try:
    cursor.execute("UPDATE users SET age = ? WHERE name = ?", (31, 'Alice'))
    print("\nAlice의 나이 수정 성공")
except sqlite3.Error as e:
    print(f"데이터 수정 오류: {e}")

```

## 7. 데이터 삭제

DELETE FROM SQL문을 사용하여 데이터를 삭제한다.

```python
try:
    cursor.execute("DELETE FROM users WHERE name = ?", ('Bob',))
    print("\nBob 데이터 삭제 성공")
except sqlite3.Error as e:
    print(f"데이터 삭제 오류: {e}")

```

## 8. 변경사항 커밋

```python
conn.commit()
```

## 9. 데이터베이스 연결 종료

작업을 마친 후에는 반드시 연결을 종료해야한다.

```python
conn.close()
```

## 10. Context Manager 사용

sqlite3 연결 객체는 컨텍스트 관리자를 지원한다. with 문을 사용하면 코드 블록이 끝날 때 자동으로 commit() 또는 오류 발생 시 rollback()을 수행하고 연결을 닫아주므로 편리해진다.

```python
import sqlite3

db_path = 'my_database.db'

try:
    with sqlite3.connect(db_path) as conn: # <- 여기가 context manager
        cursor = conn.cursor()

        # 테이블 생성, 데이터 삽입, 조회, 수정, 삭제 등의 작업 수행
        cursor.execute('''
            CREATE TABLE IF NOT EXISTS products (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                price REAL
            )
        ''')
        print("테이블 'products'가 성공적으로 생성되었거나 이미 존재합니다.")

        # 데이터 삽입 예시
        products_data = [
            ('Laptop', 1200.00),
            ('Mouse', 25.50),
            ('Keyboard', 75.00)
        ]
        cursor.executemany("INSERT INTO products (name, price) VALUES (?, ?)", products_data)
        print("상품 데이터 삽입 성공")

        # 데이터 조회 예시
        cursor.execute("SELECT * FROM products")
        all_products = cursor.fetchall()
        print("\n--- 모든 상품 데이터 ---")
        for product in all_products:
            print(product)

        # 'with' 블록을 벗어나면 자동으로 commit() 또는 rollback()이 호출됩니다.
        # 예외가 발생하지 않으면 commit()이 수행됩니다.

    print("\n데이터베이스 작업 완료 및 연결 자동 종료")

except sqlite3.Error as e:
    print(f"데이터베이스 작업 중 오류 발생: {e}")
    # 'with' 블록 내에서 오류 발생 시 자동으로 rollback()이 수행됩니다.
```