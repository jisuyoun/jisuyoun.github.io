---
layout: single
title: "[Backend] Docker와 Docker Compose 적용하기"
published: true
categories:
  - Python
tag:
  - Project
  - Python
---

**Docker**는 애플리케이션과 그 실행 환경을 컨테이너라는 표준화된 단위로 패키징하여 격리하고,  **Docker Compose**는 여러 컨테이너들을 묶어서 하나의 애플리케이션으로 정의하고 관리할 수 있게 해준다.

## 1. Docker 사용시 이점

- **환경 일관성:** 개발, 테스트, 배포 환경 모두 동일한 컨테이너 이미지를 사용하므로, 어디서든 동일하게 작동될 수 있다.
- **격리:** 각 구성 요소는 독립적인 컨테이너에서 실행되므로, 서로의 환경에 영향을 주지 않는다.
- **이식성:** 컨테이너 이미지는 Docker가 설치된 어떤 환경에서도 실행될 수 있다.
- **배포 간소화:** 복잡한 설치 및 설정 과정 없이 docker compose up 명령 하나로 시스템 전체를 실행할 수 있다.

## 2. Dockerfile 작성 전략: 공통 Dockerfile 활용

만약 프로젝트의 구성이 Web Application, Worker Script, Message Broker, Database, Persistent File Storage 이렇게 구성이 되어있다고 가정한다.

- **Web Application:** 사용자 인터페이스를 제공하고, API 요청을 처리하는 웹 서버
- **Worker Script:** 특정 백그라운드 작업을 수행하는 스크립트 (예: 데이터 처리, 메시지 소비)
- **Message Broker:** 서비스 간의 비동기 통신을 위한 메시지 큐 시스템 (예: Kafka, RabbitMQ)
- **Database:** 애플리케이션 데이터를 저장하는 데이터베이스
- **Persistent File Storage:** 사용자 업로드 파일이나 애플리케이션에서 생성된 파일(예: 이미지)을 저장하는 공간

Web Application과 Worker Script는 모두 동일한 언어 기반이며, 동일한 프로젝트 소스 코드를 사용하고, 많은 동일 언어 라이브러리 종속성을 공유할 것이다. 이러한 경우, 각 서비스별로 별도의 Dockerfile을 작성하는 것보다 하나의 공통 Dockerfile을 작성하여 환경 설정, 필요한 라이브러리 설치, 프로젝트 소스 코드 복사 등을 수행하고, Docker Compose 파일에서 이 이미지를 재사용하여 각 서비스별로 실행할 명령만 다르게 지정하는 것이 효율적이다.

### Dockerfile 작성

프로젝트의 루트 디렉토리에 Dockerfile 이라는 이름으로 파일을 생성한다.

```python
# 사용할 베이스 이미지 지정: Python 3.10 버전의 슬림 이미지 (더 작음)
FROM python:3.10-slim-bookworm

# 컨테이너 내부의 작업 디렉토리 설정
WORKDIR /app

# 로컬 프로젝트의 requirements.txt 파일을 컨테이너로 복사
COPY requirements.txt /app/requirements.txt

# requirements.txt에 명시된 Python 패키지 설치
# --no-cache-dir 옵션으로 캐시 사용 방지하여 이미지 크기 절약
# 필요한 경우 이미지 처리 라이브러리 등 위한 시스템 종속성 설치 (예시)
# RUN apt-get update && apt-get install -y --no-install-recommends libgl1-mesa-glx libglib2.0-0 && rm -rf /var/lib/apt/lists/*
RUN pip install --no-cache-dir -r requirements.txt

# 프로젝트의 모든 소스 코드를 로컬에서 컨테이너의 작업 디렉토리로 복사
# .dockerignore 파일에 명시된 파일/폴더는 제외하고 복사됩니다.
COPY . /app

# 컨테이너가 외부에 노출할 포트 선언 (Web Application용)
EXPOSE 8000

# 컨테이너 시작 시 실행될 기본 명령어 (docker-compose.yml에서 덮어쓸 예정)
# CMD ["python", "web_app.py"] # 예시

```

### **.dockerignore 파일 작성**

Dockerfile 옆에 .dockerignore 파일을 생성하여 Docker 이미지 빌드 시 불필요한 파일(가상 환경 폴더, 캐시 파일, 데이터 파일 등)이 이미지에 포함되지 않도록 한다.

```python
# Python 관련 무시 파일
.venv/
__pycache__/
*.pyc
*.sqlite3

# 데이터 파일 및 폴더 (Volume으로 관리할 대상)
*.db
data/
files/ # 예시: 파일 스토리지 폴더

# Docker 관련 파일 자체
Dockerfile
docker-compose.yml

# Git 관련 파일
.git/
.gitignore

```

### Docker Compose로 여러 서비스 오케스트레이션

하나의 Dockerfile로 이미지를 빌드하고, 이 이미지를 사용하여 Web Application, Worker Script 컨테이너를 실행하며, 외부 서비스인 Message Broker 컨테이너를 추가하고, 영구 데이터를 위한 Volume을 관리하려면 Docker Compose 파일(docker-compose.yml)이 필요하다.

프로젝트 루트 디렉토리에 docker-compose.yml 파일을 생성한다.

```python
version: '3.8' # Docker Compose 파일 형식 버전

services:
  # 1. Message Broker 서비스 정의 (Kafka 예시)
  # 외부에서 제공되는 이미지를 사용합니다.
  messenger:
    image: bitnami/kafka:3.6.1 # Bitnami Kafka 이미지 사용
    ports:
      - "9092:9092" # 호스트 포트:컨테이너 포트 (외부 접근용)
    volumes:
      - messenger_data:/bitnami/kafka # Kafka 데이터 영속화를 위한 볼륨
    environment:
      # Kafka Kraft 모드 설정 (설정은 이미지에 따라 다름)
      - KAFKA_CFG_NODE_ID=0
      - KAFKA_CFG_PROCESS_KIND=both
      - KAFKA_CFG_CONTROLLER_QUORUM_VOTERS=0@messenger:9093 # 서비스 이름으로 통신
      # 리스너 설정
      - KAFKA_CFG_LISTENERS=PLAINTEXT://:9092,CONTROLLER://:9093
      - KAFKA_CFG_ADVERTISED_LISTENERS=PLAINTEXT://localhost:9092 # 외부 접근 주소
      - ALLOW_PLAINTEXT_LISTENER=yes # 테스트용 설정

  # 2. Web Application 서비스 정의
  web:
    build: . # <-- 현재 디렉토리의 Dockerfile을 사용하여 이미지 빌드
    # 컨테이너 시작 시 실행할 명령어 (Django 개발 서버 예시)
    command: python manage.py runserver 0.0.0.0:8000
    ports:
      - "8000:8000" # 호스트 포트 8000을 컨테이너 포트 8000에 매핑
    volumes:
      # 데이터베이스 파일 및 파일 스토리지 폴더 마운트 (데이터 영속화)
      - db_data:/app/data # data 폴더 전체를 볼륨 마운트
      - file_storage_data:/app/files # files 폴더 전체를 볼륨 마운트 (설정 경로에 맞게 조정)
      # 개발 중 소스 코드 변경 즉시 반영을 원하면 바인드 마운트 사용 가능:
      # - .:/app
    environment:
      # 애플리케이션 코드에서 사용할 환경 변수 설정
      - DATABASE_URL=sqlite:///app/data/app.db # DB 경로 (컨테이너 내부 경로)
      - MESSAGE_BROKER_HOST=messenger:9092 # 메시지 브로커 주소 (서비스 이름으로 통신)
    depends_on:
      - messenger # Web Application은 Message Broker가 시작된 후 실행되어야 함

  # 3. Worker Script 서비스 정의
  worker:
    build: . # <-- 현재 디렉토리의 Dockerfile을 사용하여 이미지 빌드 (Web과 동일 이미지)
    # 컨테이너 시작 시 실행할 명령어 (Worker 스크립트 예시)
    command: python worker_script.py
    volumes:
      # 데이터베이스 파일 및 파일 스토리지 폴더 마운트 (Worker가 데이터를 읽고 쓸 수 있도록)
      - db_data:/app/data # Web과 동일한 db_data 볼륨 마운트
      - file_storage_data:/app/files # Web과 동일한 file_storage_data 볼륨 마운트
    environment:
      # Worker 스크립트에서 사용할 환경 변수 설정
      - DATABASE_URL=sqlite:///app/data/app.db # DB 경로 (컨테이너 내부 경로)
      - MESSAGE_BROKER_HOST=messenger:9092 # 메시지 브로커 주소 (서비스 이름으로 통신)
    depends_on:
      - messenger # Worker는 Message Broker가 시작된 후 실행되어야 함
      # - db_init: # DB 초기화 Worker가 있다면 의존성 추가

# 영구 데이터 저장을 위한 Volume 정의
volumes:
  messenger_data: # Message Broker 데이터 볼륨
  db_data:        # 데이터베이스 파일 및 관련 데이터 폴더 볼륨
  file_storage_data: # 사용자 파일 또는 생성 파일 저장 볼륨
  # db_init_data: # DB 초기화 Worker용 볼륨 (선택 사항)
```

- **version:** Docker Compose 파일 형식을 지정한다.
- **services:** 실행할 각 컨테이너(서비스)를 정의한다.
    - build: 현재 디렉토리의 Dockerfile을 사용하여 이미지를 빌드하고 해당 이미지를 사용한다.
    - image: [이미지 이름]: Docker Hub 등에서 제공되는 기존 Docker 이미지를 사용한다.
    - command: 컨테이너가 시작될 때 실행할 명령어를 지정한다. 각 서비스의 주요 프로세스(웹 서버, 스크립트)를 실행한다.
    - ports: 호스트 머신의 포트와 커넽이너 내부의 포트를 연결하여 외부에서 서비스에 접근할 수 있도록 한다. (Web Application에 필요)
    - volumes: 영구 데이터를 위해 컨테이너 내부 경로와 Docker Volume을 연결한다. 여러 서비스가 동일한 Volume을 마운트하여 데이터를 공유할 수 있다.
    - environment: 컨테이너 내부에서 사용할 환경 변수를 설정한다. 애플리케이션 코드에서 이 환경 변수 값을 읽어와 설정으로 사용한다. 서비스 이름(messenger)을 사용하여 다른 컨테이너에 접근할 수 있다(Docker Compose 내부 DNS).
    - depends_on: 서비스 간의 실행 순서 의존성을 설정한다. (예: 웹이나 Script는 Message Broker가 시작된 후에 시작되어야 함)
- **volumes:** services에서 사용될 Docker Volume들을 정의한다. docker compose up 실행 시 정의된 Volume이 없으면 자동으로 생성된다.

### 애플리케이션 코드 수정 (환경 변수 활용)

Docker Compose 파일에서 환경 변수(environment 항목)를 통해 설정 값을 전달하므로, Web Application 및 Worker Script 코드는 설정 파일 등에서 이 환경 변수 값을 읽어오도록 수정해야 한다.

```python
# 예시: settings.py (Django) 또는 config.py (Worker)

import os

# 환경 변수에서 데이터베이스 URL 읽어오기, 없으면 기본값 사용
DATABASE_URL = os.environ.get('DATABASE_URL', 'sqlite:///path/to/default/app.db')

# 환경 변수에서 메시지 브로커 주소 읽어오기, 없으면 기본값 사용
MESSAGE_BROKER_HOST = os.environ.get('MESSAGE_BROKER_HOST', 'localhost:9092')

# 데이터베이스 URL 파싱하여 Django 설정에 맞게 조정 (SQLite 예시)
if DATABASE_URL.startswith('sqlite:///'):
    DB_PATH = DATABASE_URL.replace('sqlite:///', '')
    DATABASES = {
        'default': {
            'ENGINE': 'django.db.backends.sqlite3',
            'NAME': DB_PATH,
        }
    }
# 다른 DB 타입 (postgresql, mysql)에 대해서도 파싱 로직 추가
```

## 3. 시스템 실행하기

Dockerfile과 docker-compose.yml 작성을 마친 후, 프로젝트 루트 디렉토리에서 터미널을 열고 다음 명령어를 실행한다.

### 이미지 빌드 및 컨테이너 실행

```python
docker compose up --build
```

`--build`옵션은 Dockerfile을 사용하여 이미지를 빌드한 후 컨테이너를 실행한다. 처음 실행하거나 Dockerfile 내용이 변경되었을 때 필요하다. 이후에는 `docker compose up`만으로 실행할 수 있다.

### 컨테이너 백그라운드 실행

```python
docker compose up -d --build
```

`-d` 옵션은 컨테이너를 백그라운드에서 실행시킨다.

### 컨테이너 로그 확인

```python
docker compose logs [서비스 이름] # 예: docker compose logs web
docker compose logs -f # 모든 서비스의 로그를 실시간으로 확인
```

### 컨테이너 중지

```python
docker compose down
```

실행 중인 모든 컨테이너를 중지하고 제거한다. `-v` 옵션을 추가하면 볼륨도 함께 제거되므로 데이터가 삭제된다.