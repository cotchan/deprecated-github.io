---
title: sb3) 도메인 이벤트와 메시징 시스템(feat. MSA)
author: cotchan 
date: 2021-05-22 12:21:21 +0800 
categories: [Spring-Boot3]
tags: [spring-boot3] 
---

+ **이 포스팅은 개인 공부 목적으로 작성한 포스팅입니다**

---

## 1. 도메인 이벤트란?

<img width="645" alt="스크린샷 2021-05-22 오후 8 28 33" src="https://user-images.githubusercontent.com/75410527/119225033-67de8400-bb3c-11eb-860c-2528173b97ba.png">

---

## 2. 도메인 이벤트 처리시 장점

<img width="646" alt="스크린샷 2021-05-22 오후 8 28 43" src="https://user-images.githubusercontent.com/75410527/119225036-6b720b00-bb3c-11eb-98cc-f616db140e9e.png">

---

## 3. MS에서의 이벤트 전파

+ **마이크로 서비스는 `하나의 도메인을 담당하는 시스템`입니다.**
  + 하나의 도메인은 여러 개의 인스턴스로 이뤄질 수 있습니다.

+ 하나의 마이크로 서비스가 동작하는 원리는 아래와 같습니다. 
  1. **`하나의 command를 받습니다.`**
    + 예시
      + api 콜
      + event를 만들게 하는 행위
      + ex. price가 업데이트 되었다.
  2. **`event를 발생시킵니다.`**
  3. **그리고 `event를 이벤트 버스에 쏩니다.`**
    + **MSA에서 말하는 '이벤트 버스'는 `메시지 브로커를 의미`합니다.**
    + 메시지 브로커라는 또 다른 Software가 돌고 있는 것입니다.(`kafka`)
    + 이벤트 버스가 실제 메시지 전달을 담당합니다.
    + 이벤트 버스 또한 한 대 이상의 서버일 수 있습니다. 
 
+ 즉 메시지를 다른 서비스에게 비동기로 전달하는 형태로 시스템을 구성하게 됩니다.
+ 여러 서비스들끼리 Message Queue로 연결할 때는 `TCP 기반으로 통신`합니다.
+ 소켓이 연결되어 있고, 내부적으로 연결되어 있는 형태(HTTP X)

---

<img width="646" alt="스크린샷 2021-05-22 오후 8 29 00" src="https://user-images.githubusercontent.com/75410527/119225043-70cf5580-bb3c-11eb-98c8-2a8449aeb59e.png">

<img width="646" alt="스크린샷 2021-05-22 오후 8 29 07" src="https://user-images.githubusercontent.com/75410527/119225045-73ca4600-bb3c-11eb-825c-c40b823d5c76.png">

---

+ 출처
    + [[스터디/9기] 단순 CRUD는 그만! 웹 백엔드 시스템 구현(Spring Boot)](https://programmers.co.kr/learn/courses/11694) 

