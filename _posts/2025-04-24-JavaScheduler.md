---
layout: single
title: "[Java] Java Scheduler"
published: true
categories:
  - Java
tag:
  - Study
  - Java
---

Java Scheduler란, 스프링에서 일정 주기마다 메소드를 실행시켜야 할 경우 사용하는 방법이다.

## [사용방법]

### 1. @Scheduled 사용

1. application.java에 @EnableScheduling을 추가한다.

```java
@SpringBootApplication
@MapperScan("com.example.demo")
@EnableScheduling
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
  }
}
```

1. 스케쥴러를 등록할 메소드에 @Scheduled를 추가

```java
@Scheduled(cron = "0 0 0 * * *")
public List<Map<String, Object>> execute() {
	System.out.println("테스트");
}
```

### 2. 스케쥴러의 주기를 별도로 관리하기

1. application.yml에 cron 표현식으로 작성

```java
scheduler:
	scrap:
		test: "0 0 0 * * *"
```

1. SpEL을 통해 @Scheduled에 속성값을 넣어주기

```java
@Scheduled(cron = "${scheduler.scrap.test}")
public List<Map<String, Object>> execute() {
	System.out.println("테스트");
}
```

## [주기 설정]

주기를 설정하는 방법으로는 3가지가 있다.

| fixedDelay | 작업 완료시점을 기준으로 일정시간 지연 후 반복 |
| --- | --- |
| fixedRate | 작업 시작지점을 기준으로 일정시간 지연 후 반복 |
| cron | 정규식 표현 |

### 1. cron 표현식

| **필드명** | **값의 허용 범위** | **허용된 특수문자** |
| --- | --- | --- |
| 초 | 0 ~ 59 | , - * / |
| 분 | 0 ~ 59 | , - * / |
| 시 | 0 ~ 23 | , - * / |
| 일 | 1 ~ 31 | , - * / L W |
| 월 | 1 ~ 12 또는 JAN ~ DEC | , - * / |
| 요일 | 0 ~ 6 또는 SUN ~ SAT | , - * ? / L # |
| 연도 | empty 또는 1970 ~ 2099 | , - * / |

### 2. 특수문자 뜻

| **특수문자** | **설명** |
| --- | --- |
| * | 모든 값을 뜻함 |
| ? | 특정한 값이 없음 |
| - | 범위를 뜻함 ex) 월요일부터 수요일 ⇒ MON-WED 로 표현 |
| , | 값을 여러 개 넣을 수 있음 ex) 월, 수, 금 |
| / | 시작시간 / 단위 ex) 0분부터 매 5분 ⇒ 0/5 로 표현 |
| L | 일에서 사용하면 마지막 일, 요일에서 사용하면 마지막 요일을 뜻함
* 마지막 요일 = 토요일 |
| W | 가장 가까운 평일 ex) 15W는 15일에서 가장 가까운 평일 |
| # | 몇 주의 무슨 요일을 표현 ex) 2번째주 수요일 ⇒ 3#2 로 표현 |

### 3. 예시

| **cron 표현식** | **빈도** |
| --- | --- |
| 0 0/5 * * * ? | 5분마다 |
| 0 0 12 * * ? | 매일 낮 12시에 |
| 0 0 0 * * * | 매일 00시에 |
| 0 0 2 ? 7 MON-WEB | 매년 7월 월~수 2시 00분 00초에 |
| 0/10 * * * * | 10초마다 |
| 0 0 14 * * * | 매일 오후 2시 |