---
title: sb2) 21. [스프링 시큐리티] 구글 로그인 연동 - 2(SecurityConfig, CustomOAuth2UserService, OAuthAttributes, SessionUser 클래스 생성) 
author: cotchan 
date: 2021-03-20 19:10:21 +0800 
categories: [Spring-Boot2, Spring-Boot2_DEV]
tags: [spring-boot2] 
---

+ **이 포스팅은 개인 공부 목적으로 작성한 포스팅입니다**

---

## 1. config.auth 패키지 생성

+ **앞으로 시큐리티 관련 클래스는 모두 이 곳에 담습니다.**

![Desktop View](/assets/img/post/spring-boot2/2021-03-20-spring-security-config-auth.png)

---

## 2. SecurityConfig 클래스 생성

+ `config.auth`

```java
package com.cotchan.randomproblem_local.config.auth;

import com.cotchan.randomproblem_local.domain.user.Role;
import lombok.RequiredArgsConstructor;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;

@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    private final CustomOAuth2UserService customOAuth2UserService;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
                .csrf().disable()
                .headers().frameOptions().disable()
                .and()
                    .authorizeRequests()
                    .antMatchers("/", "/css./**", "/images/**", "/js/**", "/h2-console/**").permitAll()
                    .antMatchers("/api/v1/**").hasRole(Role.USER.name())
                    .anyRequest().authenticated()
                .and()
                    .logout()
                        .logoutSuccessUrl("/")
                .and()
                    .oauth2Login()
                        .userInfoEndpoint()
                            .userService(customOAuth2UserService);
    }
}
```

---

+ **`@EnableWebSecurity`**
  + Spring Security 설정들을 활성화시켜 줍니다.

+ **`csrf().disable().headers().frameOptions().disable()`**
  + h2-console 화면을 사용하기 위해 해당 옵션들을 disable 합니다.

+ **`authorizeRequests`**
  + URL별 권한 관리를 설정하는 옵션의 시작점입니다.
  + authorizeRequests가 선언되어야만 antMatchers 옵션을 사용할 수 있습니다.

+ **`antMatchers`**
  + 권한 관리 대상을 지정하는 옵션입니다.
  + URL, HTTP 메소드 별로 관리가 가능합니다.
  + **"/" 등 지정된 URL들은 permitAll() 옵션을 통해 전체 열람 권한을 주었습니다.**
  + **"/api/v1/**" 주소를 가진 API는 USER 권한을 가진 사람만 가능하도록 했습니다.**

+ **`anyRequest`**
  + 설정된 값들 이외의 나머지 URL들을 나타냅니다.
  + 여기서는 authenticated()을 추가하여 나머지 URL들은 모두 인증된 사용자들에게만 허용하게 합니다.
  + 인증된 사용자는 로그인한 사용자들을 의미합니다.

+ **`logout().logoutSuccessUrl("/")`**
  + 로그아웃 기능에 대한 여러 설정의 진입점입니다.
  + 로그아웃 성공 시 / 주소로 이동합니다.

+ **`oauth2Login`**
  + OAuth 2 로그인 기능에 대한 여러 설정의 진입점입니다.

+ **`userInfoEndpoint`**
  + OAuth 2 로그인 성공 이후 사용자 정보를 가져올 때의 설정들을 담당합니다.

+ **`userService`**
  + 소셜 로그인 성공 시 후속 조치를 진행할 UserService 인터페이스의 구현체를 등록합니다.
  + 리소스 서버(즉, 소셜 서비스들)에서 사용자 정보를 가져온 상태에서 추가로 진행하고자 하는 기능을 명시할 수 있습니다.

---

## 3. CustomOAuth2UserService 클래스 생성

+ `config.auth`
+ **이 클래스에서는 구글 로그인 이후 가져온 사용자의 정보(email, name, picture 등)들을 기반으로 가입 및 정보수정, 세션 저장 등의 기능을 지원합니다.**

```java
package com.cotchan.randomproblem_local.config.auth;

import com.cotchan.randomproblem_local.config.auth.dto.OAuthAttributes;
import com.cotchan.randomproblem_local.config.auth.dto.SessionUser;
import com.cotchan.randomproblem_local.domain.user.User;
import com.cotchan.randomproblem_local.domain.user.UserRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.oauth2.client.userinfo.DefaultOAuth2UserService;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserRequest;
import org.springframework.security.oauth2.client.userinfo.OAuth2UserService;
import org.springframework.security.oauth2.core.OAuth2AuthenticationException;
import org.springframework.security.oauth2.core.user.DefaultOAuth2User;
import org.springframework.security.oauth2.core.user.OAuth2User;
import org.springframework.stereotype.Service;

import javax.servlet.http.HttpSession;
import java.util.Collections;

@RequiredArgsConstructor
@Service
public class CustomOAuth2UserService implements OAuth2UserService<OAuth2UserRequest, OAuth2User> {

    private final UserRepository userRepository;
    private final HttpSession httpSession;

    @Override
    public OAuth2User loadUser(OAuth2UserRequest userRequest) throws OAuth2AuthenticationException {
        OAuth2UserService<OAuth2UserRequest, OAuth2User> delegate = new DefaultOAuth2UserService();
        OAuth2User oAuth2User = delegate.loadUser(userRequest);

        String registrationId = userRequest.getClientRegistration().getRegistrationId();
        String userNameAttributeName = userRequest.getClientRegistration().getProviderDetails()
                .getUserInfoEndpoint().getUserNameAttributeName();

        OAuthAttributes attributes = OAuthAttributes.of(registrationId,
                                                        userNameAttributeName,
                                                        oAuth2User.getAttributes());

        User user = saveOrUpdate(attributes);

        httpSession.setAttribute("user", new SessionUser(user));

        return new DefaultOAuth2User(
                Collections.singleton(new SimpleGrantedAuthority(user.getRoleKey())),
                attributes.getAttributes(),
                attributes.getNameAttributeKey()
        );
    }

    //사용자 정보가 업데이트 되었을 때를 대비하여 update 기능도 구현
    //사용자의 이름이나 프로필 사진이 변경되면 User 엔티티에도 반영됩니다.
    private User saveOrUpdate(OAuthAttributes authAttributes) {
        User user = userRepository.findByEmail(authAttributes.getEmail())
                .map(entity -> entity.update(authAttributes.getName(), authAttributes.getPicture()))
                .orElse(authAttributes.toEntity());

        return userRepository.save(user);
    }
}
```

---

+ **`registrationId`**
  + 현재 로그인 진행 중인 서비스를 구분하는 코드입니다.
  + 네이버 로그인인지, 구글 로그인인지를 구분하기 위해 사용합니다.

+ **`userNameAttributeName`**
  + OAuth2 로그인 진행 시 키가 되는 필드값을 의미합니다. `Primary Key`와 같은 의미입니다.
  + 구글의 경우 기본적으로 코드를 지원하지만, 네이버 카카오 등은 기본 지원하지 않습니다. 
    + 구글의 기본 코드는 "sub" 입니다.
  + 네이버 로그인과 구글 로그인을 동시 지원할 때 사용됩니다.


+ **`OAuthAttributes`**
  + OAuth2UserService를 통해 가져온 OAuth2User의 attribute를 담을 클래스
  + 네이버 등 다른 소셜 로그인도 이 클래스를 사용합니다.

+ **`SessionUser`**
  + **세션에 사용자 정보를 저장하기 위한 Dto 클래스입니다.**

---

## 4. OAuthAttributes 클래스 생성

+ **리소스 서버(즉, 소셜 서비스들)에서 가져온 사용자 정보를 `CustomOAuth2UserService`에서 사용하기 위해 가공할 때 가장 먼저 사용하는 `dto 클래스`입니다.**
+ `config.auth.dto`

```java
package com.cotchan.randomproblem_local.config.auth.dto;

import com.cotchan.randomproblem_local.domain.user.Role;
import com.cotchan.randomproblem_local.domain.user.User;
import lombok.Builder;
import lombok.Getter;

import java.util.Map;

@Getter
public class OAuthAttributes {

    private Map<String, Object> attributes;
    private String nameAttributeKey;
    private String name;
    private String email;
    private String picture;

    @Builder
    public OAuthAttributes(Map<String, Object> attributes, String nameAttributeKey, String name, String email, String picture) {
        this.attributes = attributes;
        this.nameAttributeKey = nameAttributeKey;
        this.name = name;
        this.email = email;
        this.picture = picture;
    }

    public static OAuthAttributes of(String registrationId,
                                     String userNameAttributeName,
                                     Map<String, Object> attributes) {
        return ofGoogle(userNameAttributeName, attributes);
    }

    private static OAuthAttributes ofGoogle(String userNameAttributeName, Map<String,Object> attributes) {
        return OAuthAttributes.builder()
                .name((String) attributes.get("name"))
                .email((String) attributes.get("email"))
                .picture((String) attributes.get("picture"))
                .attributes(attributes)
                .nameAttributeKey(userNameAttributeName)
                .build();
    }

    public User toEntity() {
        return User.builder()
                .name(name)
                .email(email)
                .picture(picture)
                .role(Role.GUEST)
                .build();
    }
}
```

---

+ **`of()`**
  + OAuth2User에서 반환하는 사용자 정보는 Map이기 때문에 값 하나하나를 반환하기 위해 사용합니다.

+ **`toEntity()`**
  + User 엔티티를 생성합니다.
  + **OAuthAttributes에서 엔티티를 생성하는 시점은 처음 가입할 때 입니다.**
  + 가입할 때의 기본 권한을 GUEST로 주기 위해서 role 빌더값에는 Role.GUEST를 사용합니다.


---

## 5. SessionUser 클래스 생성

+ `config.auth.dto` 패키지에 SessionUser 클래스를 추가합니다.
+ **SessionUser는 직렬화 기능을 가진 세션 Dto입니다.**
+ SessionUser에는 `인증된 사용자 정보`만 필요합니다.
  + 그 외에 필요한 정보들은 없으니 name, email, picture만 필드로 선언합니다.
+ SessionUser DTO의 역할은 **`OAuthAttributes dto`로 한 번 가공한** 사용자 정보를 **`User Domain`에 저장한 뒤** 세션 생성을 위해 사용하는 dto 클래스입니다.


```java
package com.cotchan.randomproblem_local.config.auth.dto;

import com.cotchan.randomproblem_local.domain.user.User;
import lombok.Getter;

import java.io.Serializable;

@Getter
public class SessionUser implements Serializable {

    private String name;
    private String email;
    private String picture;

    public SessionUser(User user) {
        this.name = user.getName();
        this.email = user.getEmail();
        this.picture = user.getPicture();
    }
}
```

---

+ 출처
  + 이동욱, 『스프링 부트와 AWS로 혼자 구현하는 웹 서비스』, 프리렉(2019) 
