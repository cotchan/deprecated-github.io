---
title: sb3) GeneralExceptionHandler (@ControllerAdvice)
author: cotchan 
date: 2021-05-09 18:20:21 +0800 
categories: [Spring-Boot3]
tags: [spring-module] 
---

+ **이 포스팅은 개인 공부 목적으로 작성한 포스팅입니다**

---

## 1. @ControllerAdvice

+ **@ControllerAdvice는 당신의 스프링 어플리케이션에게 이 클래스가 당신의 어플리케이션의 예외 처리를 맡을거라고 알려주게 됩니다.**

<img width="1311" alt="스크린샷 2021-05-09 오후 7 14 05" src="https://user-images.githubusercontent.com/75410527/117568389-e548d400-b0fa-11eb-97cc-fb7e7364f7d9.png">

---

+ **`@ControllerAdvice 사용 시 주의사항`** 
  + **@ControllerAdvice에서 예외를 잡는 건 `Controller 코드를 실행한 쓰레드에서 예외가 throw 됐을 때 입니다.`**
  + **만약 Controller에서 비동기 처리를 위해 다른 스레드에게 작업을 위임했다면 `거기서 발생한 예외는 Controller에서 발생한 예외가 아닙니다.`**
    + 그러므로 비동기 처리한 코드는 `.exceptionally` 등을 사용해서 별도의 예외 핸들링이 필요합니다.

---

## 2. @ExceptionHandler

+ **이 어노테이션을 사용하면 예외를 처리할 클래스를 정의합니다.**

---

## 3. GeneralExceptionHandler

+ **요구사항**

<img width="1219" alt="스크린샷 2021-05-09 오후 7 13 08" src="https://user-images.githubusercontent.com/75410527/117568395-f1cd2c80-b0fa-11eb-9442-7255606cfc33.png">

---

+ `controller 패키지`에 선언합니다.

```java
//GeneralExceptionHandler.java

package com.github.prgrms.social.controller;

@ControllerAdvice
public class GeneralExceptionHandler {

  private final Logger log = LoggerFactory.getLogger(getClass());

  private ResponseEntity<ApiResult<?>> newResponse(Throwable throwable, HttpStatus status) {
    HttpHeaders headers = new HttpHeaders();
    headers.add("Content-Type", "application/json");
    return new ResponseEntity<>(ERROR(throwable, status), headers, status);
  }

  @ExceptionHandler(NoHandlerFoundException.class)
  public ResponseEntity<?> handleNotFoundException(Exception e) {
    return newResponse(e, HttpStatus.NOT_FOUND);
  }

  @ExceptionHandler({
    IllegalStateException.class, IllegalArgumentException.class,
    TypeMismatchException.class, HttpMessageNotReadableException.class,
    MissingServletRequestParameterException.class, MultipartException.class,
  })
  public ResponseEntity<?> handleBadRequestException(Exception e) {
    log.debug("Bad request exception occurred: {}", e.getMessage(), e);
    return newResponse(e, HttpStatus.BAD_REQUEST);
  }

  @ExceptionHandler(HttpMediaTypeException.class)
  public ResponseEntity<?> handleHttpMediaTypeException(Exception e) {
    return newResponse(e, HttpStatus.UNSUPPORTED_MEDIA_TYPE);
  }

  @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
  public ResponseEntity<?> handleMethodNotAllowedException(Exception e) {
    return newResponse(e, HttpStatus.METHOD_NOT_ALLOWED);
  }

  @ExceptionHandler(ServiceRuntimeException.class)
  public ResponseEntity<?> handleServiceRuntimeException(ServiceRuntimeException e) {
    if (e instanceof NotFoundException)
      return newResponse(e, HttpStatus.NOT_FOUND);
    if (e instanceof UnauthorizedException)
      return newResponse(e, HttpStatus.UNAUTHORIZED);

    log.warn("Unexpected service exception occurred: {}", e.getMessage(), e);
    return newResponse(e, HttpStatus.INTERNAL_SERVER_ERROR);
  }

  @ExceptionHandler({Exception.class, RuntimeException.class})
  public ResponseEntity<?> handleException(Exception e) {
    log.error("Unexpected exception occurred: {}", e.getMessage(), e);
    return newResponse(e, HttpStatus.INTERNAL_SERVER_ERROR);
  }

}
```

---

+ **코드 설명**
    + **`handleBadRequestException` method**
      + 위에 @ExceptionHandler으로 선언한 IllegalStateException부터, MultipartException까지 예외를 잡습니다.
    + **`handleException` method**
      + Exception 클래스의 자식들의 모든 예외를 처리합니다.

---

## 3-1. ResponseEntity

+ **`오류에 대한 응답 포맷을 맞추기 위해` ResponseEntity를 사용합니다.**
  + GeneralExceptionHandler는 HTTP Header 또한 조작하고 있기 때문에 ApiResult 포맷만으로는 부족합니다.
+ **`Response Body와 Response Header를 모두 조작할 때는 ResponseEntity를 사용해야 합니다.`**

---

## 3-2. ServiceRuntimeException

+ **ServiceRuntimeException에 대한 로직은 `하나의 메소드에서 세분화해서 처리`하고 있습니다.**
+ **`비즈니스 예외는 앞으로 계속 추가될 수 있습니다.`**
  + 별도로 처리되지 않은 에러가 발생하면 로그 수집을 통해 확인 후, if 문 추가를 통해 핸들링 할 수 있습니다.


```java
@ExceptionHandler(ServiceRuntimeException.class)
public ResponseEntity<?> handleServiceRuntimeException(ServiceRuntimeException e) {
  if (e instanceof NotFoundException)
    return newResponse(e, HttpStatus.NOT_FOUND);
  if (e instanceof UnauthorizedException)
    return newResponse(e, HttpStatus.UNAUTHORIZED);

  log.warn("Unexpected service exception occurred: {}", e.getMessage(), e);
  return newResponse(e, HttpStatus.INTERNAL_SERVER_ERROR);
}
```

---

## 4. GeneralExceptionHandlerTest

```java
package com.github.prgrms.social.controller;

import com.github.prgrms.social.controller.authentication.AuthenticationRestController;
import com.github.prgrms.social.controller.post.PostRestController;
import com.github.prgrms.social.controller.user.UserRestController;
import com.github.prgrms.social.error.NotFoundException;
import com.github.prgrms.social.error.UnauthorizedException;
import com.github.prgrms.social.security.WithMockJwtAuthentication;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.mockito.Mockito;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.MvcResult;

import java.util.Objects;

import static org.assertj.core.api.Assertions.assertThat;
import static org.mockito.ArgumentMatchers.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@SpringBootTest
@AutoConfigureMockMvc
@RunWith(SpringRunner.class)
public class GeneralExceptionHandlerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    UserRestController userRestController;

    @MockBean
    PostRestController postRestController;

    @MockBean
    AuthenticationRestController authenticationRestController;


    // ========== HTTP 400 오류 처리 ==========
    @Test
    public void IllegalArgumentException_테스트() throws Exception {

        //given
        Mockito.when(userRestController.join(any())).thenThrow(new IllegalArgumentException("입력 양식이 올바르지 않습니다."));
        String requestBody = "{\"name\":\"찬영\",\"principal\":\"zhtcks@gmail.com\",\"credentials\":\"12341234123412341234\"}";

        //when
        mockMvc.perform(post("/api/user/join").contentType(MediaType.APPLICATION_JSON).content(requestBody))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value("false"))
                .andExpect(jsonPath("$.error.status").value(HttpStatus.BAD_REQUEST.value()))
                .andExpect(result -> {
                    assertThat(getApiResultExceptionClass(result)).isEqualTo(IllegalArgumentException.class);
                });
    }
    // ========== HTTP 400 오류 처리 ==========


    // ========== HTTP 401 오류 처리 ==========
    @Test
    public void UnauthorizedException_테스트() throws Exception {

        //given
        Mockito.when(authenticationRestController.authentication(any())).thenThrow(new UnauthorizedException("권한이 없습니다."));
        String requestBody = "{\"name\":\"찬영\",\"principal\":\"zhtcks@gmail.com\",\"credentials\":\"12341234123412341234\"}";

        //when
        mockMvc.perform(post("/api/auth").contentType(MediaType.APPLICATION_JSON).content(requestBody))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value("false"))
                .andExpect(jsonPath("$.error.status").value(HttpStatus.UNAUTHORIZED.value()))
                .andExpect(result -> {
                    assertThat(getApiResultExceptionClass(result)).isEqualTo(UnauthorizedException.class);
                });
    }
    // ========== HTTP 401 오류 처리 ==========


    // ========== HTTP 404 오류 처리 ==========
    @Test
    @WithMockJwtAuthentication
    public void NotFoundException_테스트() throws Exception {

        //given
        Mockito.when(postRestController.like(any(),any(),any())).thenThrow(new NotFoundException("사용자를 찾을 수 없습니다."));
        //when
        mockMvc.perform(patch("/api/user/1/post/1/like"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.success").value("false"))
                .andExpect(jsonPath("$.error.status").value(HttpStatus.NOT_FOUND.value()))
                .andExpect(result -> {
                    assertThat(getApiResultExceptionClass(result)).isEqualTo(NotFoundException.class);
                });
    }
    // ========== HTTP 404 오류 처리 ==========

    private Class<? extends Exception> getApiResultExceptionClass(MvcResult result) {
        return Objects.requireNonNull(result.getResolvedException()).getClass();
    }
}
```




---

+ 출처
    + [[스터디/9기] 단순 CRUD는 그만! 웹 백엔드 시스템 구현(Spring Boot)](https://programmers.co.kr/learn/courses/11694) 
    + [스프링부트 : REST 어플리케이션에서 예외처리하기](https://springboot.tistory.com/33)
    + [Assert an Exception is Thrown in JUnit 4 and 5](https://www.baeldung.com/junit-assert-exception)
    + [[Junit] Spring boot 에서 @ControllerAdvice 를 테스트하는 방법](https://sanghye.tistory.com/29)
