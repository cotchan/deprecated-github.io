---
title: Java) 중첩 클래스(Inner class)
author: cotchan
date: 2021-04-05 00:00:00 +0800
categories: [Java]
tags: [java]   
---

+ **이 포스팅은 개인 공부 목적으로 작성한 포스팅입니다**
+ **아래 출처 글을 바탕으로 작성하였습니다.**
+ 계속 내용을 업데이트 할 예정입니다.

---

## 1. 중첩 클래스란?

+ **중첩 클래스는 말 그대로 `다른 클래스의 내부에 존재하는 클래스를 의미`합니다.**
+ 중첩 클래스는 **특정 클래스가 한 곳(다른 클래스)에서만 사용될 때 논리적으로 군집화하기 위해 사용합니다.** 
+ 이로 인해 불필요한 노출을 줄이면서 캡슐화를 할 수 있고 조금 더 읽기 쉽고 유지 보수하기 좋은 코드를 작성하게 됩니다.


---

## 2. 정적, 비정적

+ 정적 내부 클래스와 비정적 내부 클래스를 나누는 기준
+ 이름에서도 알 수 있듯이 `static` 예약어가 클래스에 같이 작성되어 있는지로 판단할 수 있습니다.

+ **`정적 내부클래스 B`**

```java
class A{
    private int a;
  
    static class B{
    	    private int b;  
    }
}
```

---

+ **`비정적 내부클래스 B`**

```java
class A{
    private int a;
  
    class B{
  	      private int b;  
    }
}
```

---

## 3. 생성 방식의 차이

+ `static`의 존재 여부에 따라 해당 내부 클래스를 생성하는 방식이 달라집니다.

---

## 3-1. 정적 내부클래스 생성방법

+ **`static 예약어`가 있음으로 인해 `독립적으로 생성할 수 있습니다.`**

```java
void foo(){
    A.B b = new B();
}
```

---

## 3-2. 비정적 내부클래스 생성방법

+ 비정적 내부 클래스를 생성하는 경우에는 **`반드시 A객체를 생성한 뒤 객체를 이용해서 생성해야 합니다.`**
+ 즉, **비정적 내부 클래스는 `바깥 클래스(이 경우 A)에 대한 참조가 필요합니다.`(중요)**

```java
void foo(){
    A a = new A();
    A.B b = a.new B();
}
//or
void foo(){
    A.B b = new A().new B();
}
```

---

## 4. 메모리 누수 가능성

---

## 4-1. 정적 내부 클래스의 경우

+ 정적 내부 클래스의 경우 **바깥 클래스에 대한 참조 값을 가지고 있지 않기 때문에 `메모리 누수가 발생하지 않습니다.`**

---

## 4-2. 비정적 내부 클래스의 경우

+ 비정적 내부 클래스의 경우 **바깥 클래스에 대한 참조를 가지고 있기 때문에 `메모리 누수가 발생할 여지가 있습니다.`**
+ 바깥 클래스는 더 이상 사용되지 않지만 내부 클래스의 참조로 인해 GC가 수거하지 못해서 바깥 클래스의 메모리 해제를 하지 못하는 경우가 발생할 수 있습니다.

---

## 5. 사용시기

---

## 5-1. 정적 내부 클래스의 경우

+ 메모리 누수가 발생할 수 있는 문제점이 있기 때문에 만약 **`내부 클래스가 독립적으로 사용된다면 정적 클래스로 선언하여 사용하는 것이 좋습니다.`**

+ **바깥 클래스에 대한 참조를 가지지 않아 메모리 누수가 발생하지 않기 때문입니다.**

---

## 5-2. 비정적 내부 클래스의 경우


---

+ 출처
  + [정적, 비정적 내부 클래스 알고 사용하기](https://woowacourse.github.io/javable/post/2020-11-05-nested-class/)
