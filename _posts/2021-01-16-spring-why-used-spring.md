---
title: Spring) 왜 스프링을 쓰나요? - 3 (객체 지향 설계와 스프링)
author: cotchan 
date: 2021-01-16 21:46:21 +0800 
categories: [Spring, Spring_Basic]
tags: [spring] 
---

+ **이 포스팅은 개인 공부 목적으로 작성한 포스팅입니다**

---

## 1. 다시 스프링으로

+ 스프링 이야기에 왜 객체 지향 이야기를?

+ **스프링은 다음 기술들로 `다형성` + `OCP`, `DIP`를 가능하게 지원해줍니다.**
  + DI(Dependency Injection): 의존관계, 의존성 주입
  + DI 컨테이너 제공

+ **클라이언트 코드의 변경 없이 기능 확장**
+ 쉽게 부품을 교체하듯이 개발

---

## 2. 스프링이 없던 시절

+ 옛날에 어떤 개발자가 좋은 객체 지향 개발을 하려고 OCP, DIP 원칙을 지키면서 개발을 해보니, 너무 할 일이 많았습니다. 오히려 배보다 배꼽이 큰 꼴. 그래서 프레임워크로 만들어 버렸습니다.

+ 순수하게 자바로 OCP, DIP 원칙들을 지키면서 개발을 해보면, 결국 스프링 프레임워크를 만들게 됩니다.(더 정확히는 DI 컨테이너)

---

## 3. 정리

+ **모든 설계에 `역할`과 `구현`을 분리해야 합니다.**
  + 자동차, 공연의 예시를 떠올려보기

+ 애플리케이션 설계도 공연을 설계하듯이 배역만 만들어두고, 배우는 언제든지 `유연`하게 `변경`할 수 있도록 만드는 것이 좋은 객체 지향 설계입니다.

+ 이상적으로는 모든 설계에 인터페이스를 부여하는 게 바람직합니다.
  + 인터페이스를 먼저 만들어놓으면 하부 구현 기술에 대한 선택을 미룰 수 있는 장점이 있습니다.
  + ex. 할인 정책 인터페이스, 할인 정책에 대한 간단한 동작만 만들고 개발 계속 이어가기

---

## 3. 정리 - 실무 고민

+ 하지만 인터페이스를 도입하면 추상화라는 비용이 발생합니다.
  + 추상화가 되버리면 개발자 코드를 한 번 더 열아봐야 합니다.
  + ex. 런타임에 MemoryMemberRepository를 쓸 지, JdbcMemberRepository를 쓸 지 결정될 수 있습니다.
  + 코드만 열면 인터페이스만 보이지, 구현클래스가 뭔지 보이지 않습니다.
  + **그래서 항상 추상화의 장점이 단점을 넘어설 때 선택을 해야 합니다.**

+ 기능을 확장할 가능성이 없다면, 구현체 클래스를 직접 사용하고, 향후 꼭 필요할 때 리팩토링해서 인터페이스를 도입하는 것도 방법입니다.



---

+ 출처
    + [스프링 핵심 원리 - 기본편](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81-%ED%95%B5%EC%8B%AC-%EC%9B%90%EB%A6%AC-%EA%B8%B0%EB%B3%B8%ED%8E%B8/dashboard)