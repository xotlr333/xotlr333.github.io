---
layout: post
title: Spring REST Docs
date: 2023-10-01
categories: [Spring, Docs]
tags: [API 명세서]
---

spring 환경에서 개발할 경우 API문서 자동화 도구로 가장 많이 사용되는 것으로 Swagger와 Spring REST Docs가 있다.

# Swagger VS Spring REST Docs

## Swagger

Swagger을 사용하기 위해서는 컨트롤러 메소드에 어노테이션을 추가해야한다.

<img width="637" alt="swagger 예시" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/351c61e2-fe33-43af-ac12-ec2730116dad">

어노테이션을 통해 자동으로 API문서를 생성해준다. 그리고 API문서에서 API를 직접 실행해볼 수도 있다. 하지만 프로덕션 코드에 문서화에 대한 코드를 포함시켜야하는 단점과 API 스팩을 변경하더라고 어노테이션을 변경하지 않으면 API 문서는 수정되느 않는다는 단점을 가지고 있다.

스웨거는 작은 프로젝트에서는 효과적이나 규모가 큰 프로젝트에서는 유지보수성이 떨어지고 변경점이 늘어나서 관리가 힘들다.

[[Spring Boot] Swagger를 활용한 API 문서 자동화](https://gaga-kim.tistory.com/entry/Spring-Boot-Swagger를-활용한-API-문서-자동화)

[[API] - Swagger 사용법](https://velog.io/@cho876/API-Swagger-사용법)

## Spring REST Docs

Spring REST Docs 는 스프링 프레임워크에서 제공하는 API 문서 자동화 도구이다. 스웨거와 다르게 프로덕션 코드에 문서화 관련 코드를 추가하지 않아도 된다.

그 대신 테스트 코드를 필수로 작성해야한다. 미리 정의한 테스트가 실행되고 해당 테스트가 성공하면, 그 테스트에 대한 asciidoc 스니펫이 생성된다.

- asciidoc

  - Spring REST Docs 통해 새성되는 텍스트 기반 문서 포맷
  - 기술 문서 작성을 위해 설계된 가벼운 마크럽 언어
  - Asciidoc사용법 - [https://narusas.github.io/2018/03/21/Asciidoc-basic.html](https://narusas.github.io/2018/03/21/Asciidoc-basic.html)

- snippets(스니펫)
  - 재사용 가능한 소스 코드, 기계어, 텍스트의 작은 부분을 일컫는 프로그래밍 용어
  - 테스트 케이스에 API 스펙 정보를 추가하여 생성한 문서 일부의 조각

이 스니펫과 손으로 직접 작성한 문서를 결합하여 최종적인 API 문서가 완성된다.

성공한 테스트에 대해서만 문서가 작성된다. 그렇기 때문에 API 스펙을 수정하여도 테스트 코드를 수행한다면 API 문서와 API 스팩은 항상 일치하다.

테스트 코드만 작성하면 자동으로 API문서가 완성된다.

---

# Spring REST Docs 사용법

## 작업 환경

- Spring Boot
- Gradle
- MockMVC

## build.gradle

```java
plugins {
	id 'java'
	id 'org.springframework.boot' version '2.7.12'
	id 'io.spring.dependency-management' version '1.0.15.RELEASE'

	// .adoc 파일 생성을 위한 플러그인
	id "org.asciidoctor.jvm.convert" version "3.3.2"
}

// asciidoctorExt : Asciidoctor의 확장 기능을 사용하기 위한 설정
configurations {
	asciidoctorExt
}

// snippetsDir : 테스트 실행시 생성되는 응답을 저장할 디렉토리 지정
ext {
	snippetsDir = file('build/generated-snippets')
}

group = 'com.apply'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '11'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-validation'
	implementation 'org.springframework.boot:spring-boot-starter-web'

	// lombok 설치
	compileOnly 'org.projectlombok:lombok:1.18.20'
	annotationProcessor 'org.projectlombok:lombok:1.18.20'

	implementation 'com.google.code.gson:gson:2.8.7'

	runtimeOnly 'com.mysql:mysql-connector-j'

  // JWT를 위한 의존성
	implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
	runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
	runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'

	testImplementation 'org.springframework.boot:spring-boot-starter-test'

	// Spring REST Docs 의존성
	testImplementation("org.springframework.restdocs:spring-restdocs-mockmvc")
	asciidoctorExt("org.springframework.restdocs:spring-restdocs-asciidoctor")
}

// test 할 때 snippetsDir에 생성된 응답을 asciidoctor로 변환하여 .adoc 파일 생성
test {
	useJUnitPlatform()
	outputs.dir snippetsDir
}

// asciidoctor 실행 전에 실행
// 기존에 있던 API 문서를 삭제
asciidoctor.doFirst {
	delete file('src/main/resources/static')
}

// asciidoctor를 사용하기 위해서 asciidoctor task에 asciidoctorExt 설정
asciidoctor {
	inputs.dir snippetsDir
	configurations 'asciidoctorExt'
	dependsOn test
}

// asciidoctor task 실행시 생성된 html 파일을 src/main/resources/static 디렉토리에 복사
tasks.register('copyDocument', Copy) {
	dependsOn asciidoctor
	from file("build/docs/asciidoc")
	into file("src/main/resources/static")
}

// jar 파일로 만들어질 때 build/docs/asciidoc 파일을 src/main/resources/static 로 복사
bootJar {
	dependsOn copyDocument
	from ("build/docs/asciidoc") {
		into 'src/main/resources/static'
	}
}

// 빌드 전에 copyDocument task 실행
build {
	dependsOn(copyDocument)
}
```

대략적인 실행순서

1. 테스트 코드를 실행하여 snippetsDir에 .adoc 파일을 생성한다.
2. 생성된 .adoc 파일을 이용해 src/docs/asciidoc/index.adoc 파일을 작성한다.
3. 작성한 index.adoc 파일을 html 파일로 변환한다. 변환된 파일은 build/docs/asciidoc 에 생성된다.
4. 변화된 index.html 파일을 src/resource/static 디렉토리로 복사한다.
5. 서버 실행 후, [localhost:8080/index.html](http://localhost:8080/index.html) 로 API 문서를 확인한다.

## 테스트 코드 작성

테스크 코드 작성시, andDo를 이용하여 REST Docs의 스니펫을 만들어준다.

```java
@Test
    public void testGetMemberinfo() throws Exception {
        // Given
        String jwtToken = "jwtToken";
        JwtUserInfoResponse userInfo = JwtUserInfoResponse.builder()
                                            .memberId(1L)
                                            .role("USER")
                                            .build();

        Member mockMember = Member.builder()
                                .id(1L)
                                .email("xotlr333@naver.com")
                                .role(RoleType.USER)
                                .build();

        when(jwtTokenProvider.resolveToken(any(HttpServletRequest.class))).thenReturn(jwtToken);
        when(jwtTokenProvider.getUserInfo(jwtToken)).thenReturn(userInfo);
        when(memberService.findById(userInfo.getMemberId())).thenReturn(mockMember);

        // When
        MvcResult result = mockMvc.perform(get("/api/members/login-member"))
                .andExpect(status().isOk())
                .andDo(document("get-login-member",
                        responseFields(
                                fieldWithPath("email").description("이메일"),
                                fieldWithPath("roleType").description("권한"),
                                fieldWithPath("profileImg").description("프로필 이미지")
                        )))
                .andReturn();

        // Then
        String responseContent = result.getResponse().getContentAsString();
        MemberResponse memberResponse = new ObjectMapper().readValue(responseContent, MemberResponse.class);
        assertEquals(mockMember.getEmail(), memberResponse.getEmail());

    }
```

위의 코드를 보면 andDo를 이용해서 문서에 필요한 내용을 작성한 것을 확인할 수 있다. 이러한 정보가 테스트 실행하면 .adoc 파일로 생성된다.

## 생성된 .adoc 파일

![adoc파일](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/6ff2428b-b24b-45a1-9012-254ab72dde6d)

이 파일들을 이용해 index.adoc 파일을 작성한다.

## index.adoc 파일 작성

실제로 보여줄 문서를 작성하는 단계이다. 문서는 Asciidoc 문법을 이용해서 작성한다.

먼저 src/docs/asciidoc 디렉토리를 생성 후, index.adoc를 생성한다.

![index.adoc](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/2a0edb96-796b-4828-93fd-513cdf30094c)

생성 후, index.adoc 파일을 Asciidoc 문법으로 작성한다.

```java
= REST API 문서

== 회원 API
로그인한 회원의 정보를 조회한다.

=== 요청
include::{snippets}/get-login-member/http-request.adoc[]

=== 응답

include::{snippets}/get-login-member/http-response.adoc[]

include::{snippets}/get-login-member/response-fields.adoc[]
```

위의 문서는 실제로 아래처럼 보여진다.

![rest docs 예시](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/191e67c4-6218-49e7-ad19-ea4ba2007963)

[Asciidoc 기본 사용법](https://narusas.github.io/2018/03/21/Asciidoc-basic.html#목록list)

## index.html 파일 생성

위의 단계에서 작성한 index.adoc 파일을 index.html 파일로 변환하기 위해 build.gradle 파일에서 asciidoctor 를 실행시킨다.

![asciidoctor 예시](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/99008019-80e9-4d67-87d7-a832e1a729ae)

실행하면 build/docs/asciidoc 디렉토리에 index.html 파일이 생성된 것을 확인할 수 있다.

![index.html 생성 예시](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/9d208595-01a4-4aba-9000-df6aa55e454b)

## index.html 파일 복사

build/docs/asciidoc에 있는 index.html를 실제로 볼 수 있게 src/resource/static 디렉토리로 복사해야한다.

복사하기 위해 build.gradle 파일에서 build를 실행시킨다.

![build](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/3aac8352-fb99-4308-88b9-6940d67d62e7)

build를 실행시키면 copyDocument task가 실행되어 파일이 복사된다.

![복사된 파일](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/216bc02b-6a1d-4a8e-a82a-2119deae5eef)

그러면 모든 준비는 끝났다.

이제 서버를 실행시켜 [localhost:8080/index.html](http://localhost:8080/index.html) 로 이동하여 문서를 확인한다.

<img width="1440" alt="완성된 예시" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/11ad7cb8-1fe5-4403-9ca0-e72d38d079f6">

참고)

[https://hudi.blog/spring-rest-docs/](https://hudi.blog/spring-rest-docs/)

[https://techblog.woowahan.com/2597/](https://techblog.woowahan.com/2597/)

[https://helloworld.kurly.com/blog/spring-rest-docs-guide/](https://helloworld.kurly.com/blog/spring-rest-docs-guide/)

[https://tecoble.techcourse.co.kr/post/2020-08-18-spring-rest-docs/](https://tecoble.techcourse.co.kr/post/2020-08-18-spring-rest-docs/)

[https://velog.io/@gudcks0305/Spring-Spring-Rest-Docs-사용법](https://velog.io/@gudcks0305/Spring-Spring-Rest-Docs-%EC%82%AC%EC%9A%A9%EB%B2%95)

[https://velog.io/@cho876/API-Swagger-사용법](https://velog.io/@cho876/API-Swagger-%EC%82%AC%EC%9A%A9%EB%B2%95)

[Spring REST Docs 사용하기 - 테스트코드 작성](https://browngoo.tistory.com/40)

[Spring REST Docs 적용 및 최적화 하기](https://backtony.github.io/spring/2021-10-15-spring-test-3/)

[[Spring] Spring Rest Docs 사용법](https://velog.io/@gudcks0305/Spring-Spring-Rest-Docs-사용법)
