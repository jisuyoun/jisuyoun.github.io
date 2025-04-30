---
layout: single
title: "[Python] VSCode에 Django 세팅하기"
published: true
categories:
  - Python
tag:
  - Project
  - Python
---

## 확장 설치

1. 작업할 새 폴더를 만든다.
2. Vscode 실행 후 1번에서 만든 폴더를 연다.
3. `Ctrl + Shift + X`를 눌러 확장으로 들어간다.
4. Django를 검색해서 설치
    
    ![django.png](django.png)
    
5. python을 검색해서 설치
    
    ![python.png](python.png)
    

## 가상환경 설치

1. Ctrl + `을 눌러 터미널 창을 열어준다.
2. 터미널 창에서 `python -m venv venv` 입력한다.
    1. 2번째 venv 부분은 가상환경의 이름을 지어주는 곳이다.
3. 명령어를 입력하여 가상환경이 제대로 설치되었다면 아래와 같이 나온다.
    
    ![venv.png](venv.png)
    

## 가상환경 실행

1. 터미널에 아래 코드를 입력
    1. **Windows 기준:** `venv/Scripts/activate` 또는 `venv\Scripts\activate`
        1. powershell에 입력했더니 오류가 난다면 터미널을 cmd로 변경하여 입력해준다.
        2. powershell에서 스크립트를 실행시킬 수 없다는 오류가 날 경우 실행 정책 문제이므로, `Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned` 입력 → 이 방법을 추천하지는 않음
    2. **Mac 기준:** `venv/bin/activate`
2. 가상환경을 나가고자 할 경우에는 `deactivate`를 입력해주면 된다.

## 패키지 설치

1. 가상환경을 실핸한 후 터미널에서 `pip install django`를 입력한다.
    
    ![image.png](image.png)
    

1. DRF를 설치하기 위해 `pip install djangorestframework`를 입력한다.
    
    ![image.png](image%201.png)
    

💡DRF란 Django REST Framework의 줄임말로, Django 프레임워크를 기반으로 RESTful API를 구축하는 데 사용되는 강력하고 유연한 도구 세트를 말한다.
{: .notice--primary}

## gitignore 설정

만약 git을 사용할 경우 venv가 업로드 되지 않게 설정해준다.

1. [https://www.toptal.com/developers/gitignore/](https://www.toptal.com/developers/gitignore/) 접속
2. 자기에게 맞는 운영체제, 개발 환경, 프로그래밍 언어 입력 후 생성 클릭
    
    ![io.png](io.png)
    

1. 생성 버튼을 클릭한 후 나온 화면 내용을 전부 복사한다.
    
    ![nore.png](nore.png)
    
2. venv 폴더 안에 있는 .gitignore에 복사한 내용을 붙여준다.
    1. 만약 .gitignore가 없을 경우 생성
3. 12번째 줄에 있는 media 아래 venv 입력한 후 저장한다.
4. `pip freeze > requirements.txt`를 입력하여 requirements.txt 파일을 생성한다.
    1. 다음에 이 패키지를 다운 받을 때 클론을 받은 후 `pip install -r requirements.txt` 명령어를 통해 설치했던 패키지들을 한번에 다운 받을 수 있다.

## Django 프로젝트 생성

1. `django-admin startproject {프로젝트이름}`을 입력해준다.
    
    ![list.png](list.png)
    
- **__init__.py:** 현재 폴더가 파이썬 패키지임을 나타내는 파일
- **asgi.py:** 웹 서버와 프레임워크 간의 인터페이스를 정의하는 파일
- **settings.py:** Django 전체의 setting을 설정/관리 하는 곳
    - INSTALLED_APPS: Django에 설치된 앱
    - MIDDLEWARE: 사용자 요청/응답 사이에 작동하는 시스템
    - TEMPLATES: 나의 html파일을 자동으로 인식하도록 함
    - DATABASES: 내가 사용할 데이터베이스 연동 설정
    - AUTH_PASSWORD_VALIDATORES: 패스워드 보안 수준 검증
    - LANGUAGE_CODE: 화면에 어떤 언어를 보여줄 것인지 설정
    - TIME_ZONE: 나타내고자 하는 시간
- **urls.py:** Django의 API의 주소를 관리하는 곳
- **wsgi.py:** Django의 서버를 다루게 해주는 python 파일
- **venv:** 프로젝트의 패키지들을 관리하는 가상환경

1. 만들어준 패키지 폴더 중 가장 안쪽 폴더 내에 templates 폴더 생성
2. 그  후, `python manage.py runserver` 명령어를 통해 manage.py 실행
3. [http://127.0.0.1:8000](http://127.0.0.1:8000)으로 들어가서 Django 페이지가 뜨면 세팅 완료
    
    ![image.png](image%202.png)