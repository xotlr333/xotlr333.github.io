---
layout: post
title: Srping Security
date: 2023-11-24
categories: [Spring, Security]
tags: [Spring Security]
---

Spring 기반의 애플리케이션의 보안(인증과 권한, 인가 등)을 담당하는 스프링 하위 프레임워크이다.

## 인증 순서

spring security는 세션-쿠키방식으로 인증한다.

1. 유저가 로그인을 시도 (http request)
2. AuthenticationFilter 에서 부터 userDB까지 타고 들어간다
3. DB에 있는 유저라면 UserDetails로 꺼내서 유저의 session 생성
4. spring security의 인메모리 세션저장소인 SecurityContextHolder에 저장
5. 유저에게 sesstion ID와 함께 응답을 내려줌
6. 이후 요청에서는 요청 쿠키에서 JSESSIONID를 까봐서 검증 후 유효하면 Authentication를 쥐어준다

![spring security](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/77e1ea13-b90a-4b11-bd2a-e951d249524c)

## 주요 Fliter들

### SecurityContextPersistenceFilter

SecurityContextRepository에서 SecurityContext를 가져오거나 저장하는 역할을 한다. (SecurityContext는 밑에)

### LogoutFilter

설정된 로그아웃 URL로 오는 요청을 감시하며, 해당 유저를 로그아웃 처리

### (UsernamePassword)AuthenticationFilter

(아이디와 비밀번호를 사용하는 from 기반 인증) 설정된 로그인 URL로 오는 요청을 감시하며, 유저 인증 처리

1. AuthenticationManager를 통한 인증 실행
2. 인증 성공 시, 얻은 Authentication 객체를 SecurityContext에 저장 후 AuthenticationSuccessHandler 실행
3. 인증 실패 시, AuthenticationFailureHandler 실행

### DefaultLoginPageGeneratingFilter

인증을 위한 로그인폼 URL을 감시한다.

### BasicAuthenticationFilter

HTTP 기본 인증 헤더를 감시하여 처리한다.

### RequestCacheAwareFilter

로그인 성공 후, 원래 요청 정보를 재구성하기 위해 사용된다.

### SecurityContextHolderAwareRequestFilter

HttpServletRequestWrapper를 상속한 SecurityContextHolderAwareRequestWrapper 클래스로 HttpServletRequest 정보를 감싼다. SecurityContextHolderAwareRequestWrapper 클래스는 필터 체인상의 다음 필터들에게 부가정보를 제공한다.

### AnonymousAuthenticationFilter

이 필터가 호출되는 시점까지 사용자 정보가 인증되지 않았다면 인증토큰에 사용자가 익명 사용자로 나타난다.

### SessionManagementFilter

이 필터는 인증된 사용자와 관련된 모든 세션을 추척한다.

### ExceptionTranslationFilter

이 필터는 보호된 요청을 처리하는 중에 발생할 수 있는 예외를 위임하거나 전달하는 역할을 한다.

### FilterSecurityInterceptor

이 필터는 AccessDecisionManager로 권한부여 처리를 위임함으로써 접근 제어 결정을 쉽게 해준다.

## 주요 모듈

### SecurityContextHolder

보안 주체의 세부 정보를 포함하여 응용프로그램의 현재 보안 컨텍스트에 대한 세부 정보가 저장된다.

### SecurityContext

Authentication을 보관하는 역할을 하며, SecurityContext를 통해 Authentication 객체를 꺼내올 수 있다.

### Authentication

Authentication는 현재 접근하는 주체의 정보와 권한을 담는 인터페이스이다. Authentication 객체는 SecurityContext에 저장되며, SecurityContextHolder를 통해 SecurityContext에 접근하고, SecurityContext를 통해 Authentication에 접근할 수 있다.

### UsernamePasswordAuthenticationToken

Authentication을 implements한 AbstractAuthenticationToken의 하위 클래스로, User의 ID가 Principal 역할을 하고, Password가 Credential의 역할을 한다. UsernamePasswordAuthenticationToken의 첫 번째 생성자는 인증 전 객체를 생성하고, 두번째 생성자는 인증이 완료된 객체를 생성한다.

### AuthenticationProvider

실제 인증에 대한 부분을 처리하는데, 인증 전의 Authentication 객체를 받아서 인증이 완료된 객체를 반환하는 역할을 한다.’

### Authentication Manager

인증에 대한 부분은 SpringSecurity의 AuthenticationManager를 통해서 처리하게 되는데, 실질적으로는 AuthenticationManager에 등록된 AuthenticationProvider에 의해 처리된다.

### UserDetails

인증에 성공하여 생성된 UserDetails 객체는 Authentication 객체를 구현한 UsernamePasswordAuthenticationToken을 생성하기 위해 사용된다.

### UserDetailsService

UserDetailsService 인터페이스는 UserDetails 객체를 반환하는 단 하나의 메소드를 가지고 있는데, 일반적으로 이를 구현한 클래스의 내부에 UserRepository를 주입받아 DB와 연결하여 처리한다.

### PasswordEncoding

AuthenticationManagerBuilder.userDetailsService().passwordEncoder()를 통해 패스워드 암호화에 사용될 PasswordEncoder 구현체를 지정할 수 있다.

### GrantedAuthority

현재 사용자(principal)가 가지고 있는 권한을 의미한다.

---

참고)

[https://www.boostcourse.org/web326/lecture/58997?isDesc=false](https://www.boostcourse.org/web326/lecture/58997?isDesc=false)

[https://sjh836.tistory.com/165](https://sjh836.tistory.com/165)

[https://mangkyu.tistory.com/76](https://mangkyu.tistory.com/76)
