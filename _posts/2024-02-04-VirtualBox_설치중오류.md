---
layout: single
title:  "[Linux] 리눅스 설치 중 발생한 문제 정리"
summary: "리눅스 설치를 위한 VirtualBox 설치 중 발생했던 문제를 정리하였다."
date: 2024-02-04 20::00 +0900
categories:
  - Etc
tags:
  - Record
  - Linux
toc: true
toc_sticky: true
---   

<br />

[리눅스 입문-개념으로 탄탄히!!](https://www.inflearn.com/course/%EB%A6%AC%EB%88%85%EC%8A%A4-%EC%9E%85%EB%AC%B8/dashboard) 강의를 듣기 위해 VirtualBox를 설치하는 중에 생겼던 문제를 정리하였다.   

<br /><br />

# 1. VirtualBox 및 우분투 다운   
[VirtualBox 관리자](https://www.virtualbox.org/)를 먼저 설치하였다. 아래 링크에서 가장 최신버전을 다운받았다.   

[우분투는 20.04 버전](https://ubuntu.com/download/desktop)을 다운받았다.   

처음에는 우분투도 최신 버전으로 다운 받았었는데, 20.04가 아닌 최신 버전을 받을 경우 VirtualBox에 우분투를 설치하려고 할 시 아래와 같은 화면이 뜨면서 설치가 진행되지 않았다.   

![image](https://github.com/jisuyoun/jisuyoun.github.io/assets/122525676/7bc00d5f-131b-4873-8a1f-d6f4a9097ee4)

이 문제는 우분투를 20.04로 다운받으면 나타나지 않는다!   

<br /><br />

# 2. 버튼이 보이지 않는 문제   
새로 만들어진 가상머신을 실행시키면 우분투 설치가 진행된다.   

설치 중 문제가 발생했는데, 처음 화면에서 한국어로 설정한 후 설치할 때는 더블클릭으로 해결이 됐으나,   

진행중 아래 화면에 도달하니 다음으로 갈 수 있는 버튼이 보이지 않아 진행이 불가능하였다.   

![image](https://github.com/jisuyoun/jisuyoun.github.io/assets/122525676/5adc508b-d659-47fb-8a70-c35ab8b88b75){: width="70%" height="70%"}     

이럴 경우에는 <span style="background:#ffc2b3">**Alt + F7**</span>를 누르면 마우스 포인터가 주먹 쥔 손으로 바뀌게 되는데,

이 때 마우스를 조금 움직여보면 설치 창이 움직이는 것을 확인할 수 있다.

다음으로 갈 수 있는 버튼이 보이는 위치로 창 위치를 조정하면 진행이 가능하다.   

<br /><br />   

# 3. 너무 느린 설치속도   
다운을 진행하다보니 너무 느린 것은 물론이고, 이름과 비밀번호가 입력되지 않아 진행이 막히게 되었다.   

이를 해결하기 위해 나의 경우에는 가상머신을 우선 종료한 후 아래 화면에서 가상 머신을 선택한 후 설정 버튼을 눌렀다.   

![image](https://github.com/jisuyoun/jisuyoun.github.io/assets/122525676/3bc513e6-2756-4876-ba87-ea4f9e113255){: width="70%" height="70%"}     
   
## 3.1. 기본 메모리 늘리기
설정에서 시스템 > 마더보드에서 <span style="background:#ffc2b3">**기본 메모리**</span>를 늘려준다.   
<span style="color:gray;font-size:0.75em">기존에는 1024 MB로 지정하였었다.</span>   

![image](https://github.com/jisuyoun/jisuyoun.github.io/assets/122525676/37c88d42-1d78-48b0-ab20-b5e1f948225e){: width="70%" height="70%"}      
   
## 3.2. CPU 늘리기
시스템 > 프로세서에서 <span style="background:#ffc2b3">**CPU의 갯수를 2개**</span>로늘려준다.    
<span style="color:gray;font-size:0.75em">기존에는 CPU는 1개였다.</span>   

![image](https://github.com/jisuyoun/jisuyoun.github.io/assets/122525676/5e77d3a6-4d78-4afd-adeb-2a22ddb32b8d){: width="70%" height="70%"}    

## 3.3. 반가상화 인터페이스 변경   
시스템 > 가속 에서 반가상화 인터페이스를 <span style="background:#ffc2b3">**Hyper-V**</span> 로 변경하였다.   

🔔 리눅스면 KVM, 윈도우라면 Hyper-V 로 하면 된다고 한다.
{: .notice--primary}   

![image](https://github.com/jisuyoun/jisuyoun.github.io/assets/122525676/0fffcb8e-343c-4ae3-acfd-ffb5c2668baa){: width="70%" height="70%"}     

## 3.4. 디스플레이 변경   
디스플레이 > 화면 에서 <span style="background:#ffc2b3">**비디오 메모리를 늘려주고,**</span> 확장된 기능에서 <span style="background:#ffc2b3">**'3차원 가속 활성화'도 체크**</span>해주었다.   

![image](https://github.com/jisuyoun/jisuyoun.github.io/assets/122525676/5b351eaa-80bb-4f39-a9ac-db19d532bea5){: width="70%" height="70%"}     

이렇게 설정하고나니 속도가 느린 것이 좀 개선되었고,   

사용자 이름과 비밀번호 설정도 가능하게 되었다.   

설치가 된 후에는 우분투의 바탕화면? 이 뜨게 되는데, 

여기서 장치 > 게스트 확장 CD 이미지 삽입... 을 눌러 추가적인 설치 파일들을 다운받는 것이 좋다한다.   
![image](https://github.com/jisuyoun/jisuyoun.github.io/assets/122525676/3a461c2b-6f4c-4c22-8b7c-f9232bea7daf){: width="70%" height="70%"}     

