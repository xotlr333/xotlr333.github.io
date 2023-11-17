---
layout: post
title: Filter VS Interceptor VS AOP
date: 2023-11-06
categories: [Spring, 구조]
tags: [AOP, Filter, Interceptor]
---

## Spring 요청/응답 흐름

![스프링 구조](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/ca38e4d3-1d3d-4a0d-a0e6-cf209191f428)

- Interceptor 와 Filter는 Servlet 단위에서 실행된다.
- AOP는 메소드 앞의 Proxy 패턴의 형태로 실행된다.
- 요청 실행 순서 : Filter → Interceptor → AOP → Controller
- 응답 실행 순서 : Controller → AOP → Interceptor → Filter

---

## Filter(필터)

필터는 말 그대로 요청과 응답을 거른 뒤 정제하는 역할을 한다.

Dispatcher Servlet에 요청이 전달되기 전/후에 URL 패턴에 맞는 모든 요청에 대해 부가 작업을 처리할 수 있는 기능을 제공한다.

즉, 스프링 컨테이너가 아닌 톰캣과 같은 웹컨테이너에 의해 관리가 되는 것이고, 스프링 범위 밖에서 처리되는 것이다. (스프링 빈으로는 등록 가능)

### Filter의 메소드 종류

필터를 사용하기 위해서는 javax.servlet의 Filter 인터페이스를 구현해야 하며, 다음과 같은 메소드를 가진다.

```java
public interface Filter {

    public default void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    public default void destroy() {}
```

#### 1. init()

필터 객체를 초기화하고 서비스에 추가하기 위한 메소드이다.

웹 컨테이너가 1회 init()을 호출하여 필터 객체를 초기화하면 이루 요청들은 doFilter()를 통해 처리된다.

참고)

[[Spring] 필터(Filter)와 인터셉터(Interceptor)의 개념 및 차이](https://dev-coco.tistory.com/173)

[[Spring] Filter, Interceptor, AOP 차이 및 정리](https://goddaehee.tistory.com/154)

#### 2. doFilter()

url-pattern에 맞는 모든 HTTP 요청이 Dispatcher Servlet으로 전달되기 전에 웹 컨테이너에 의해 실행되는 메소드이다.

doFilter의 파라미터로 FilterChain이 있는데, FilterChain의 doFilter 통해 다음 대상으로 요청을 전달할 수 있게 된다.

chain.doFilter()로 전/후에 우리가 필요한 처리 과정을 넣어줌으로써 원하는 처리를 진행할 수 있다.

#### 3. destory()

필터 객체를 제거하고 사용하는 자원을 반환하기 위한 메소드이다.

웹 컨테이너가 1회 destory()를 호출하여 필터 객체를 종료하면 이후에는 doFilter에 의해 처리되지 않는다.

---

## Interceptor(인터셉터)

쉽게 말해 요청에 대한 작업 전/후로 가로챈다고 보면 된다.

Dispatcher Servlet의 Controller를 호출하기 전/후에 인터셉터가 끼어들어 요청과 응답을 참조하거나 가공할 수 있는 기능을 제공한다.

웹 컨테이너에서 동작하는 필터와 달리 인터셉터는 스프링 컨텍스트에서 동작한다.

Dispatcher Servlet이 핸들러 매핑을 통해 컨트롤러를 찾도록 요청하는데, 그 결과로 실행 체인(HandlerExecutionChain)을 돌려준다.

여기서 1개 이상의 인터셉터가 등록되어 있다면 순차적으로 인터셉터들을 거쳐 컨트롤러가 실행되도록 하고, 인터셉터가 없다면 바로 컨트롤러를 실행한다.

실제로 인터셉터가 직접 Controller로 요청을 위임하는 것은 아니다.

### Interceptor의 메소드 종류

인터셉터를 추가하기 위해서 org.springframework.web.servlet의 HandlerInterceptor 인터페이스를 구현해야 하며, 다음과 같은 메소드를 가진다.

```java
public interface HandlerInterceptor {

	default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		return true;
	}

	default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}

	default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable Exception ex) throws Exception {
	}
```

#### 1. preHandle()

controller가 호출되기 전에 실행된다.

컨트롤러 이전에 처리해야 하는 전처리 작업이나 요청 정보를 가공하거나 추가하는 경우에 사용할 수 있다.

#### 2. postHandle()

controller가 호출된 후에 실행된다. (view 렌더링 전)

컨트롤러 이후에 처리해야 하는 후처리 작업이 있을 때 사용할 수 있다. 이 메소드는 컨트롤러가 반환하는 ModelAndView 타입의 정보가 제공되는데, 최근에는 JSON 형태로 데이터를 제공하는 RestAPI 기반의 컨트롤러를 만들면 자주 사용되지 않는다.

#### 3. afterCompletion()

모든 뷰에서 최종 결과를 생성하는 일을 포함해 모든 작업이 완료된 후에 실행된다. (view 렌더링 후)

요청 처리 중에 사용한 리소스를 반환할 때 사용할 수 있다.

---

## AOP

객체 지향의 프로그램을 했을 때 중복을 줄일 수 없는 부분을 줄이기 위해 종단면(관점)에서 바라보고 처리한다.

주로 로깅, 트랜잭션, 에러처리 등 비즈니스단의 메소드에서 조금 더 세밀하게 조정하고 싶을 때 사용한다.

인터셉터나 필터와는 달리 메소드 전/후의 지점에 자유롭게 설정이 가능하다.

인터셉터와 필터는 주소로 대상을 구분해서 걸러내야하는 반면, AOP는 주소, 파라미터, 어노테이션 등 다양한 방법으로 대상을 지정할 수 있다.

AOP의 Advice와 HandlerInterceptor의 가장 큰 차이는 파라미터의 차이다.

Advice의 경우 JoinPoint나 ProceedingJoinPoint 등을 활용해서 호출한다.

반면 HandlerInterceptor는 Filter와 유사하게 HttpServletRequest, HttpServletResponse를 파타미터로 사용한다.

### AOP의 포인트컷

1. @Before: 대상 메소드의 수행 전
2. @After: 대상 메소드의 수행 후
3. @After-returning: 대상 메소드의 정상적인 수행 후
4. @After-throwing: 예외발생 후
5. @Around: 대상 메소드의 수행 전/후

---

## Filter와 Interceptor의 차이

### 핵심 차이점 : Filter에서는 Request와 Response를 조작할 수 있다.

```java
public class MyFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
    throws IOException, ServletException {
        // 다른 request와 response를 넣어줄 수 있음
        chain.doFilter(request, response);
    }
}
```

필터가 다음 필터를 호출하기 위해서는 필터 체이닝을 해주어야한다.

이때 Request, Response 객체를 넘겨주므르 우리가 원하는 request, response 객체를 넣어줄 수 있다.

하지만 인터셉터는 처리 과정이 필터와 다르다.

```java
public class MyInterceptor implements HandlerInterceptor {

    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
    throws Exception {
        // Request, Response를 교체할 수 없고 boolean 값만 반환 가능
        return true;
    }
}
```

디스패치 서블릿이 여러 인터셉터 목록을 가지고 있고, 순차적으로 실행시킨다.

그리고 true를 반환하면 다은 인터셉터가 실행되거나 컨트롤러로 요청이 전달되며, false가 반환되면 요청이 중단된다. 그러므로 다른 request, response 객체를 넘겨줄 수 없다.

### 사용 사례

#### Filter

1. 보안 및 인증/인가 관련 작업
2. 모든 요청에 대한 로깅 또는 검사
3. 이미지/데이터 압축 및 문자열 인코딩
4. Spring과 분리되어야 하는 기능

#### Interceptor

1. 세부적인 보안 및 인증/인가 공통 작업
2. API 호출에 대한 로깅 또는 검사
3. Controller로 넘겨주는 정보의 가공
