---
title: Flutter) 플러터 용어사전
author: cotchan
date: 2021-03-17 15:42:00 +0800
categories: [Flutter, Flutter_main]
tags: [flutter]   
---

+ **이 포스팅은 개인 공부 목적으로 작성한 포스팅입니다**
+ **아래 출처 글을 바탕으로 작성하였습니다.**

---

## 1. Widget 클래스

+ **`화면을 그리는 모든 디자인 요소`를 위젯이라고 합니다.**

---

## 2. StatelessWidget 클래스

+ `상태가 없는 위젯`을 정의할 때는 StatelessWidget 사용
  + **상태를 가지지 않는다는 것은 한 번 그려진 후 다시 그리지 않는다는 뜻입니다.**
  + 이러한 클래스는 프로퍼티로 `변수를 가지지 않습니다.` (상수는 가능)

---

## 3. StatefulWidget 클래스

+ `상태가 있는 위젯`을 정의할 때는 StatefulWidget 클래스 사용
+ StatefulWidget의 서브클래스는 상태를 가질 수 있습니다.
  + **그리고 그 상태는 `State 클래스의 서브 클래스로 정의`합니다.**

---

## 4. State 클래스

+ State 클래스를 상속받은 클래스를 `상태 클래스`라고 부릅니다.
+ `상태 클래스`는 **변경 가능한 상태**를 `프로퍼티 변수`로 표현합니다.
  + 나중에 이 변수의 값을 변경하면서 화면을 다시 그리게 됩니다.

+ **State 클래스에서는 주로 `상태를 저장할 변수`들과 `그 변수를 조작할 메서드`를 작성합니다.**

---

## 5. build()

+ **화면에 그려질 부분을 정의합니다.**
+ **build() 메서드는 위젯을 생성할 때 호출됩니다.**
+ **실제로 화면에 그릴 위젯을 작성해 반환합니다.**

---

## 6. createState()

+ 이 메서드는 `StatefulWidget`이 생성될 때 한 번만 실행되는 메서드

---

## 7. setState()

+ **State클래스가 제공하는 메서드입니다.**

+ **전달된 익명 함수를 실행한 후 화면을 다시 그리는 역할을 합니다.**
+ 화면은 build() 메서드가 실행되면서 그려집니다.
  + 즉, setState()는 익명 함수를 실행한 후 build() 메서드가 다시 실행되게 하는 역할

---

## 8. runApp()

+ **`runApp()` 함수는 인수로 넘긴 위젯을 화면에 표시합니다.**

```dart
void main() {
    runApp(MyApp());
}
``` 

---

## 9. =>

+ **`=>` 기호는 함수의 내용이 한 줄 일 때 중괄호({})를 대체하는 역할을 합니다.**

---

+ 출처
  + [[flutter]프로젝트 구조](https://devlopsquare.tistory.com/81)