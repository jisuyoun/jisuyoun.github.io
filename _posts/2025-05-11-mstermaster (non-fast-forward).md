---
layout: single
title: "[Git] mster → master (non-fast-forward)"
published: true
categories:
  - Git
tag:
  - Git
---    

### mster → master (non-fast-forward)

원격 저장소에 있는 main 브랜치와 로컬 브랜치와 다르기 때문에 발생하는 문제다.

만약 main 브랜치를 master로 변경하여 사용하는 방법으로 해결할 경우에는 아래 방법을

1. **원격 브랜치 확인**
    
    git branch -r
    
    ```java
      origin/HEAD -> origin/master
      origin/master
    ```
    
2. **로컬 브랜치 이름 변경**
    
    git branch -m main master
    
3. **원격 저장소에 푸시**
    
    git push -u origin master
    

다른 방법으로는 git pull —rebase 입력 후 git push

다른 방법으로는 git push -f origin master 

다른 방법으로는, 로컬 main 브랜치에서 원격 main 브랜치와 병합하는 방법 - 추천 안 함

1. **로컬 main 브랜치에서 원격 main 브랜치와 병합**
    
    git pull origin main --allow-unrelated-histories
    
2. **충돌 해결**
    
    git add .
    
    git commit -m “메시지”
    
3. **원격 main 브랜치에 푸시**
    
    git push -u origin main