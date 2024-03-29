---
title: sb3) 인증(Authentication) 과정 정리 
author: cotchan 
date: 2021-04-25 22:00:21 +0800 
categories: [Spring, Spring_Security]
tags: [spring-security]
---

+ **이 포스팅은 개인 공부 목적으로 작성한 포스팅입니다**
+ **아래 출처를 참고하여 작성하였습니다.**

---

## 인증이란?

+ **인증이란 `현재 사용자가 누구인지 식별하는 것`을 의미합니다.**
  + 사용자의 신원을 식별하는 것
  + 예시) 아이디/패스워드 로그인
+ **인증 과정에서 사용하는 주요 컴포넌트는 `AuthenticationManager`, `AuthenticationProvider` 입니다.**

---

## 인증 처리(과정) 요약

+ **인증처리를 위한 핵심 컴포넌트**

+ **`UsernamePasswordAuthenticationFilter`** (디폴트 인증필터)
  + **사용자 인증 요청을 Authentication 인터페이스로 추상화하고, AuthenticationManager를 호출합니다.**
    + 1) 사용자로부터 받은 요청에서(@RequestBody) '로그인 안된 인증 주체'(Authentication)을 만듭니다.
      + getPrincipal 사용
      + getCredential 사용
    + 2) 만든 '로그인 안된 인증주체'에 대한 처리를 AuthenticationManager에게 위임합니다.

+ **`AuthenticationManager`**
  + **사용자 아이디/비밀번호를 인증하기 위해 적절한 AuthenticationProvider를 찾아 처리를 위임합니다.**
    + 1) AuthenticationManager는 AuthenticationProvider를 List로 가지고 있습니다.
    + 2) AuthenticationManager는 AuthenticationProvider를 하나 선택해서 실제 로그인 요청을 처리합니다.

+ **`AuthenticationProvider`**
  + **실질적으로 사용자 인증을 처리하고, 인증 결과를 Authentication 인터페이스로 반환합니다.**
    + 1) AuthenticationProvider는 요청으로 들어온 '로그인 안 된 인증주체'를 `타입캐스팅`합니다.
    + 2) 그리고 UserService를 이용해서 로그인 처리를 해봅니다.
    + 3) 로그인이 정상처리 되어 UserService로부터 UserEntity를 받으면 UserEntity로부터 `'로그인된 인증주체'`를 만듭니다.
    + 4) '로그인 된 인증주체'는 principal 타입이 String에서 JwtAuthentication으로 바뀌고 jwt를 발급받습니다. 
    + 5) '로그인 된 인증주체'를 리턴해줍니다.

+ 기타
  + AuthenticationManager에게 '로그인된 인증주체'를 넘겨받은 컨트롤러는 '로그인된 인증주체'에서 아래의 정보만 꺼내서 클라이언트에게 돌려줄 응답에 포함시킵니다.
    + jwt
    + userInformation

---

## 1. JwtAuthenticationTokenFilter

+ **`웹 요청이 컨트롤러에 가기 전에` 먼저 수행됩니다.**
+ JwtAuthenticationTokenFilter에서
  + `SecurityContextHolder를 뒤져보고` http 요청에 JwtToken이 있는지 확인해봅니다.
  + 만약에 `Jwt가 있다면` Jwt 값을 검증하고 인증 정보를 생성해서 SecurityContextHolder에 추가

+ **SecurityContextHolder는 현재 로그인된 사용자 정보를 담고있습니다.**

---

## 2. AuthenticationRestController

```java
//AuthenticationRestController.java

@RestController
@RequestMapping("api/auth")
public class AuthenticationRestController {

  private final AuthenticationManager authenticationManager;

  //... 생성자

  @PostMapping
  public ApiResult<AuthenticationResultDto> authentication(@RequestBody AuthenticationRequest authRequest) throws UnauthorizedException {
    try {
      
      //인증 주체를 만듭니다.
      JwtAuthenticationToken authToken = new JwtAuthenticationToken(authRequest.getPrincipal(), authRequest.getCredentials());

      //만든 인증 주체를 AuthenticationManager에게 위임합니다.
      Authentication authentication = authenticationManager.authenticate(authToken);

      SecurityContextHolder.getContext().setAuthentication(authentication);
  
      //AuthenticationManager로부터 받은 로그인된 인증 주체를 사용하여 응답을 내려줍니다. 
      return OK(
        new AuthenticationResultDto((AuthenticationResult) authentication.getDetails())
      );
    } catch (AuthenticationException e) {
      throw new UnauthorizedException(e.getMessage());
    }
  }
}
```

---

## 2-1. 로그인 안 된 인증주체 만들기

+ **컨트롤러는 사용자로부터 받은 요청에서(`@RequestBody`) 로그인 안 된 인증 주체(`Authentication`)를 만듭니다.**
  + `getPrincipal` 사용
  + `getCredential` 사용

---

## 2-2. 만든 인증주체에 대한 처리를 AuthenticationManager에게 위임

+ 컨트롤러는 자신이 만든 `'로그인 안 된 인증주체'에 대한 처리`를 `AuthenticationManager에게 위임`

+ **지금까지 설명한 부분이 아래 이미지의 `빨간색 네모 박스에 해당하는 내용`입니다.**

<img width="2281" alt="스크린샷 2021-04-25 오후 10 20 37" src="https://user-images.githubusercontent.com/75410527/115995058-adc72b80-a614-11eb-9dc0-732c15357a88.png">

---

## 3. AuthenticationManager

+ AuthenticationManager는 인터페이스고, 실제 로직을 처리하는 구현체는 `ProviderManager` 입니다.

+ **지금부터 설명하는 부분은 아래 이미지의 `빨간색 네모 박스에 해당하는 내용`입니다.**

<img width="2276" alt="스크린샷 2021-04-25 오후 10 21 13" src="https://user-images.githubusercontent.com/75410527/115995082-cd5e5400-a614-11eb-9f46-8708b7799da3.png">


---

## 3-1. Manager는 Provider로 인증 처리

+ AuthenticationManager는 AuthenticationProvider를 List로 가지고 있습니다.
+ **그리고 AuthenticationManager는 이 `Provider List를 사용하여 실제 인증 처리`를 합니다.**

---

## 3-2. Provider.supports == true ?

+ **AuthenticationManager는 `AuthenticationProvider를 하나 선택`해서 `실제 로그인 요청을 처리`**
+ AuthenticationProvider를 선택하는 방법은 
  + **Provider의 `supports 메서드가 true를 리턴`하면 이 provider가 로그인 요청을 처리합니다.**

---

## 3-3. Provider.authenticate 호출

+ supports 메서드에서 true를 리턴한 provider가 `인증처리에 대한 메서드 authenticate를 호출`합니다.
  + authenticate 메서드가 실질적으로 로그인을 처리

---

## 3-4. authenticate 메서드에서는

---

## 3-5. 인증주체 타입 캐스팅

+ 우선, 자신이 정한 인증 주체로 authentication을 타입캐스팅합니다.
  + 타입 캐스팅이 가능한 이유는 parameter로 넘어온 authentication을 처리할 수 있다고 했기 때문
  + `Provider.supports == true` 이므로 

---

## 3-6. UserService 이용해서 로그인 처리

+ 그리고 나서 타입 캐스팅한 인증 주체를 실제 Service를 사용하여 로그인 처리를 합니다. 

```java
//3-5, 3-6에 해당하는 코드
//JwtAuthenticationProvider.java


//null이 아닌 값을 반환하면 인증 처리가 완료됩니다.
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {

  /**
   * 이 인증 주체에 대한 처리를 할 수 있다고 했기 때문에 이 인증 주체 타입(JwtAuthenticationToken)으로 타입 캐스팅을 합니다.
   * 이 인증 주체는 '로그인 전'을 의미하는 인증주체
   */
  JwtAuthenticationToken authenticationToken = (JwtAuthenticationToken) authentication;

  //그리고 나서 실제 UserSerivce를 이용해서 로그인 처리를 합니다.
  return processUserAuthentication(authenticationToken.authenticationRequest());
}
```

---

## 3-7. '로그인 후 인증 주체' 만들기

+ UserService를 이용하여 로그인을 처리하기 때문에
+ 로그인이 정상처리 되어 UserService로부터 User Entity를 받으면 
+ User Entity로부터 '로그인 후 인증 주체'를 만듭니다.
+ **`로그인 후 인증주체`는**
  + **`principal 타입`이 String에서 `JwtAuthentication`으로 바뀌고 `jwt를 발급`받습니다.**

```java
//JwtAuthenticationProvider.java

private Authentication processUserAuthentication(AuthenticationRequest request) {
  try {
    //여기에 principal 부분은 email, credentials 부분은 비밀번호입니다.
    User user = userService.login(new Email(request.getPrincipal()), request.getCredentials());

    /**
     * 인증 주체를 새로 만들고 있습니다.
     * 이 인증 주체는 '로그인 후'를 의미하는 인증주체
     */
    JwtAuthenticationToken authenticated =
      new JwtAuthenticationToken(new JwtAuthentication(user.getSeq(), user.getEmail()), null, createAuthorityList(Role.USER.value()));

      String apiToken = user.newApiToken(jwt, new String[]{Role.USER.value()});

    authenticated.setDetails(new AuthenticationResult(apiToken, user));
    return authenticated;
  } catch (Exception e) {
  }
}
```

---

## 3-8. '로그인 된 인증주체' return

+ **결국 AuthenticationManager.authenticate 메서드에서 `'로그인 된 인증주체'를 return 해줍니다.`**

---

## 4. Manager -> 다시 컨트롤러로 

+ AuthenticationManager에게 `'로그인된 인증주체'`를 넘겨받은 컨트롤러는
+ `'로그인 된 인증주체'`에서 아래의 정보만 꺼내서 클라이언트에게 돌려줄 응답에 포함시킵니다.
  + `jwt`
  + `userInformation`

---

## 4-1. 컨트롤러가 Response 응답 전달

+ 바로 위의 `4` 과정에서 조립한 응답을 클라이언트에게 전달합니다.

---

## 5. 원본 필기 내용

+ 아래 필기 내용을 바탕으로 작성한 포스팅입니다.

![img001](https://user-images.githubusercontent.com/75410527/115998762-c854d100-a623-11eb-8ae3-66df165f2a04.jpg)

---

![img002](https://user-images.githubusercontent.com/75410527/115998767-cab72b00-a623-11eb-8926-866ee87752e7.jpg)

---

+ 출처
  + [[스터디/9기] 단순 CRUD는 그만! 웹 백엔드 시스템 구현(Spring Boot)](https://programmers.co.kr/learn/courses/11694) 




