---
title: JPA) 준영속 상태 
author: cotchan 
date: 2021-01-19 04:50:21 +0800 
categories: [JPA, JPA_Basic]
tags: [jpa] 
---

+ **이 포스팅은 개인 공부 목적으로 작성한 포스팅입니다**

---

## 1. 영속 상태가 되는 경우

1. 내가 `em.persist`해서 집어넣을때 영속 상태가 됩니다.
2. `em.find`로 JPA를 통해서 조회를 했을 때도, 영속성 컨텍스트에 없으면 조회 후에 그 엔티티는 영속 상태가 됩니다.

---

## 2. 준영속 상태란

+ **영속 상태의 엔티티가 영속성 컨텍스트에서 분리되는 것입니다.**
  + JPA가 더 이상 해당 엔티티를 관리하지 않습니다.
+ 준영속 상태가 되면 영속성 컨텍스트가 제공하는 기능을 사용하지 못합니다.

---

## 3. 준영속 상태로 만드는 방법

+ `em.detach(entity)`
  + 특정 엔티티만 준영속 상태로 전환합니다.
+ `em.clear()`
  + 영속성 컨텍스트를 완전히 초기화할 때 사용합니다.
+ `em.close()`
  + 영속성 컨텍스트를 종료합니다.


---

+ 출처
    + [자바 ORM 표준 JPA 프로그래밍 - 기본편](https://www.inflearn.com/course/ORM-JPA-Basic)