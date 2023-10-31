---
layout: post
title: 웹소켓(WebSocket)
date: 2023-10-31
categories: [네트워크, 프로토콜]
tags: [웹소켓]
---

# 웹 소켓(WebSocket)이란?

HTML5 표준 기술로, HTTP 환경에서 클라이언트와 서버사이에 하나의 TCP 연결을 통해 실시간으로 `전이중 통신`을 가능하게 하는 `프로토콜`이다. 여기서 전이중 통신이란, 일방적인 송신 또는 수신만이 가능한 단방향 통신과 달리 가정에서의 전화와 같이 양방향으로 송신과 수신이 가능한 것을 말한다. 양방향 통신이 아닌 단방향 통신의 예로는 텔레비전 방송, 라디오를 들을 수 있는데, 데이터를 수신만 할 수 있고, TV나 라이오를 통해 데이터를 보낼 수 없다.

## 소켓 프로토콜이란?

소켓 프로토콜은 컴퓨터 네트워크에서 프로세스 간 통신을 가능하게 하는 통신 규약의 일종이다. 네트워크를 통해 데이터를 주고 받기 위해서는 서로 간에 통신을 위한 규칙이 필요한데, 이러한 규칙들이 프로토콜로 정의된다.

소켓은 컴퓨터에세 프로세스 간 통신을 수행하는 데 사용되는 인터페이스로, 네트워크 소켓은 프로세스가 네트워크를 통해 데이터를 주고 받을 수 있도록 돕느다. 소켓 프로토콜은 네트워크 소켓을 이용하여 데이터를 주고 받기 위한 규칙들을 정의한 것이다.

### 네트워크 소켓

네트워크 소켓은 컴퓨터 네트워크에서 프로세스 간 통신을 수행하는 인터페이스이다. 네트워크 소켓을 이용하여 서로 다른 프로세스들 간에 데이터르 주고 받을 수 있다.

웹 브라우저를 실행하는 클라이언트가 웹 서버에 데이터를 요청하거나, 이메일 클라이언트가 메일 서버로부터 이메일을 받아오는 등의 통신은 네트워크 소켓을 이용하여 이루어진다.

### 주요한 소켓 프로토콜

### TCP(Transmission Control Protocol)

- 연결 지향적 프로토콜로, 신뢰성이 높은 데이터 전송을 제공한다.
- 데이터 전송에 있어서 순서를 보장하고, 손실된 데이터를 다시 전송하며, 중복된 데이터를 제거한다.
- 신뢰성을 보장하기 위해 흐름제어, 혼잡제어 등의 기능을 제공한다.
- 대표적으로 웹 브라우저를 이용한 HTTP 통신에 사용된다.

### UDP(User Datagram Protocol)

- 비연결 지향적 프로토콜로, 신뢰성이 낮은 데이터 전송을 제공한다.
- 데이터를 보내기만 하고 확인을 받지 않기 때문에 TCP보다 빠른 속도를 가진다.
- 신뢰성 낮기 때문에 데이터 손실이 발생할 수 있으며, 순서가 바뀔 수 있다.
- 실시간 스트리밍, 음성 및 영상 통화 등에 주로 사용된다.

---

# 웹소켓 이전의 기술

웹 소켓 기술이 없을 때는 `Polling`이나 `Long Polling` 등의 방식으로 실시간은 아니지만 그에 준하게끔 구현하여 해결했다.

## Polling

Polling은 사전적 의미로 충돌 회피 또는 동기화 처리 등을 목적으로 상태를 주기적으로 검사하여 조건을 만족할 때 자료처리를 하는 방식이다. 실시간처럼 보이는 웹사이트들은 클라이언트가 서버에게 일정한 주기를 가지고 응답을 주고 받는 방식이다. 이는 Ajax Polling이라고도 불린다. Ajax 를 사용하기 때문이다.

### 문제점

- 폴링의 주기가 짥으면 서버의 성능에 부담이 간다.
- 주기가 길면 실시간성이 떨어진다.
- 설시간으로 주는 건 불가능하다. 실시간 효과를 내려면 간격을 줄여야 하지만 서버와 클라이언트 모두에게 부담이다.
- 보낼 데이터가 없어도 계속해서 데이터를 줘야 하므로 서버의 리소스를 낭비하게 된다.

![Polling](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/f37a6580-efc4-4f87-877c-f2b94c211567)

## Long Polling

Long Polling은 Polling의 방식에서 개선된 방식으로, 서버가 대기하고 있다가 이벤트가 발생 시 응답하는 방식이다. 서버는 즉시 응답을 주지 않는다.

특정 이벤트가 발생하거나 타임아웃이 발생하면 응답을 전달한다. 클라이언트는 응답받은 후 다시 서버에게 데이터를 요청한다. 그리고 클리이언트는 정보를 받자마자 바로 다시 서버에게 요청을 보낸다. 이 결과 연결은 무한히 지속되게 된다. 클라이언트는 마치 실시간으로 데이터를 받는 느낌을 받을 수 있다.

### 문제점

- Polling 방식보다는 서버의 부담이 줄어들지만 클라이언트에게 동시에 많은 양의 메세지가 올 경우 Polling과 별 차이가 없게되며, 다수의 클라이언트에게 동시에 이벤트가 발생될 경우에는 곧바로 다수의 클라이언트가 서버로 접속을 시도하게 되면서 서버의 부담이 급증하게된다.

![Long Polling](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/5915b9d4-4154-43a8-8272-93e90f79e775)

---

# 웹소켓 작동 원리

기존의 HTTP 프로토콜은 클라이언트가 서버에 요청을 보내고 서버가 응답하는 단방향 통신이었지만, 웹소켓은 연결을 유지하며 양방향을 데이터를 주고 받을 수 있어서 실시간성이 필요한 응용 프로그램에 적합하다.

## 작동 원리

![작동 원리](https://github.com/xotlr333/xotlr333.github.io/assets/81614820/02448e9e-8951-401f-9d34-76f34f3c0ac0)

### 1. Opening Handshake

웹소켓 클라이언트에서 핸드쉐이크 요청을 전송하고 이에 대한 응답으로 핸드쉐이크 응답을 받는데, 이때 응답 코드는 101이다. 101은 `프로토콜 전환` 을 서버가 승인했음을 알리는 코드이다.

```java
// 요청 헤더
GET /ws/chat HTTP/1.1
Host: localhost:8080
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Version: 13
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
```

- Upgrade : 프로토콜을 전환하기 위해 사용하는 헤더. 웹소켓 요청 시에는 반드시 websocket이라는 값을 가지며, 이 값이 없거나 다른 값이면 cross-protocol attack이라고 간주하여 웹소켓 접속이 중지된다.
- Connection : 현재의 전송이 완료된 후 네트워크 접속을 유지할 것인가에 대한 정보. 웹소켓 요청 시에는 반드시 Upgrade라는 값을 가지며, Upgrade와 마찬가지로 이 값이 없거나 다른 값이면 웹소켓 접속을 중시시킨다.
- Sec-WebSocket-Key : 유효한 요청인지 확인하기 위해 사용하는 키 값.
- Sec-WebSocket-Version : 클라이언트가 사용하고자 하는 웹소켓 프로토콜 버전.
- Sec-WebSocket-Extensions : 클라이언트는 자신이 제공하는 확장 기능에 대해 기술.

```java
// 응답 헤더
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: HSmrc0sMlYUkAGmm5OPpG2HaGWk=
Sec-WebSocket-Extensions: permessage-deflate;client_max_window_bits=15
```

- Upgrade와 Connection은 요청과 동일하다.
- Sec-WebSocket-Accept : 요청 헤더의 Sec-WebSocket-Key에 유니크 아이디를 더해서 SHA-1로 해싱한 후, base64로 인코딩한 결과. 웹소켓 연결이 개시되었음을 알린다.
  클라이언트의 보안 키를 기반으로 서버가 생성한 응답 키로, 클라이언트가 보낸 "Sec-WebSocket-Key" 값을 기반으로 웹소켓 연결을 수락한 증거로 사용된다.

### 2. Data Transfer

핸드쉐이크를 통해 웹소켓 연결이 수립되면, 데이터 전송 파트가 시작된다. 여기에서는 클라이언트와 서버가 ‘메시지’라는 개념으로 데이터를 주고 받는데, 여기서 메시지는 한 개 이상의 ‘프레임’으로 구성되어 있다.

핸드쉐이크가 끝난 시점부터 서버와 클라이언트는 서로가 살아있는지 확인하기 위해 heartbeat 패킷을 보내며, 주기적으로 ping을 보내 체크한다. 이는 서버와 클라이언트 양측에서 설정이 가능하다.

### 3. Close Handshake

클라이언트와 서버 모두 커넥션을 종료하기 위한 컨트롤 프레임을 전송할 수 있다. 이 컨트롤 프레임은 Closing Handshake를 시작하라는 특정한 컨트롤 시퀀스를 포함한 데이터를 가지고 있다. 위 그림에서는 서버가 커넥션을 종료한다는 프레임을 보내고, 클라이언트가 이에 대한 응답으로 Close 프레임을 전송한다. 그러면 웹소켓 연결이 종료된다. 연결 종료 이후에 수신되는 모든 추가적인 데이터는 버려진다.

핸드셰이크(Handshake)

- 클라이언트가 서버에 웹소켓 연결을 요청한다. 이를 웹소켓 핸드셰이크라고 한다.
- 클라이언트는 HTTP Request 메시지를 보내고, 서버는 이를 확인하여 Upgrade 요청을 받아들이고 웹소켓 연결을 설정한다.

---

# 동작 예시

스프링과 크롬을 이용하면 쉽게 웹소켓을 구현할 수 있다.

### build.gradle

```java
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-websocket'
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

dependencies에 Websocket 를 추가한다.

### WebSocket Handler

socket 통신은 서버와 클라이언트가 1:N으로 관계를 맺는다. 따라서 한 서버에 여러 클라이언트가 접속할 수 있으며, 서버에는 여러 클라이언트가 발송한 메시지를 받아 처리해줄 Handler가 필요하다.

TextWebSocketHandler를 상속받아 Handler를 작성한다. TextWebSocketHandler는 텍스트 메시지만 처리하는 WebSocketHandler 구현을 위해 편리하게 사용하는 기본 클래스이다.

```java
@Slf4j
@Component
public class WebSocketHandler extends TextWebSocketHandler {

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String payload = message.getPayload();
        log.info("payload : " + payload);
        TextMessage textMessage = new TextMessage(payload);
        session.sendMessage(textMessage);
    }
}
```

클라이언트로부터 메시지를 받으면 로그로 출력하고 클라이언트에게 받은 메시지를 그대로 보내주는 기능을 구현했다.

### WebSocket Config

handler를 이용하여 WebSocket을 활성화하기 위해 Config 파일이 필요하다.

@EnableWebSocket을 선언하여 WebSocket을 활성화한다.

WebSocket에 접속하기 위한 Endpoint 는 /ws/chat으로 설정하고 도메인이 다른 서버에서도 접속 가능하도록 setAllowedOrigins(”\*”)를 설정한다. 그럼 클라이언트가 ws://localhost:8080/ws/chat으로 연결하고 메시지 통신을 할 수 있는 기본적인 준비가 끝난다.

```java
@RequiredArgsConstructor
@Configuration
@EnableWebSocket // websocket 활성화
public class WebSocketConfig implements WebSocketConfigurer {

    private final WebSocketHandler webSocketHandler;

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(webSocketHandler, "/ws/chat").setAllowedOrigins("*");
    }
}
```

### Simple Web Socket Client

크롬 확장자에서 간단하게 소켓 통신을 확인할 수 있는 확장자가 있다.

<img width="1248" alt="simple websocket client" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/09a6f085-873d-4fd4-ad64-1feeae5c73ef">

<img width="497" alt="simple websocket client" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/2d41be7d-0a4d-482a-be9e-e16a579e395c">

1.  Server Location : 핸드쉐이크 요청 URL를 입력 후, Open 버튼을 클릭하여 웹소켓을 연결한다.
    개발자 도구에서 네트워크 탭으로 이동하여 헤더 데이터를 확인한다.
    <img width="687" alt="헤더 데이터" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/9ad226bc-d450-4b9f-8a6b-414ca989c3ff">
2.  Request : 웹소켓이 연결되면 메시지를 서버로 보내본다.
3.  Message Log : 보낸 메시지와 응답 메시지가 잘 나오는지 확인한다.
    개발자 도구의 네트워크 탭에서 메시지를 확인할 수 있다.
    <img width="233" alt="메시지 확인" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/982c33db-ad4d-4d98-a68d-8064258c982f">

    <img width="423" alt="요청 응답 확인" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/c0ffe37a-bb4d-4d62-847d-d026d39ad5d4">

---

# 웹소켓 단점

### 1. Cross Browser 문제

<img width="1108" alt="브라우저 호환성" src="https://github.com/xotlr333/xotlr333.github.io/assets/81614820/8daff4c6-56aa-4a2e-bffb-04c5a8be05d6">

브라우저 중 버전에 따라 지원을 안하는 경우도 있다.

### 2. Stateful

WebSocket은 Stateless를 지향하는 일반 HTTP 와는 다르게 상태를 유지하는 Stateful 한 성격을 가지고 있어 서버와 클라이언트가 연결을 항상 유지해야한다.

이는 연결을 유지해야하기 때문에 부하가 발생할 수 있고, 비정상적으로 연결을 끊어졌을 때 적절한 대응이 필요하여 프로그램 구현에 보다 많은 복잡성을 유발한다.

# 해결방안

### SockJS

SockJS는 WebSocket Emulation을 지원하는 Javascript 라이브러리이다.

WebSocket Emulation 이란 WebSocket을 지원하지 않거나 연결 실패로 연결이 되지 않는 경우 Long Polling  등의 기존 HTTP 기반의 다른 기술로 전환해 연결을 시도하는 방법이다.

이를 활용해 기존에 웹소켓을 지원하지 않는 브라우저의 환경에서도 실시간 통신을 가능하게 해준다.

---

참고)

[https://choseongho93.tistory.com/266](https://choseongho93.tistory.com/266)
[https://velog.io/@codingbotpark/Web-Socket-이란](https://velog.io/@codingbotpark/Web-Socket-%EC%9D%B4%EB%9E%80)
[https://tecoble.techcourse.co.kr/post/2021-08-14-web-socket/](https://tecoble.techcourse.co.kr/post/2021-08-14-web-socket/)
[https://cyber0946.tistory.com/112](https://cyber0946.tistory.com/112)
[https://softlabsblog.tistory.com/191](https://softlabsblog.tistory.com/191)
[https://velog.io/@hahan/Polling-Long-Polling-Streaming](https://velog.io/@hahan/Polling-Long-Polling-Streaming)
[https://doozi0316.tistory.com/entry/WebSocket이란-개념과-동작-과정-socketio-Polling-Streaming](https://doozi0316.tistory.com/entry/WebSocket%EC%9D%B4%EB%9E%80-%EA%B0%9C%EB%85%90%EA%B3%BC-%EB%8F%99%EC%9E%91-%EA%B3%BC%EC%A0%95-socketio-Polling-Streaming)
[https://yuricoding.tistory.com/134](https://yuricoding.tistory.com/134)
[https://bol-bbang.tistory.com/77](https://bol-bbang.tistory.com/77)
[https://www.daddyprogrammer.org/post/4077/spring-websocket-chatting/](https://www.daddyprogrammer.org/post/4077/spring-websocket-chatting/)
[https://kellis.tistory.com/65](https://kellis.tistory.com/65)
[https://tecoble.techcourse.co.kr/post/2020-09-20-websocket/](https://tecoble.techcourse.co.kr/post/2020-09-20-websocket/)
[https://velog.io/@hahm0726/WebSocket의-단점](https://velog.io/@hahm0726/WebSocket%EC%9D%98-%EB%8B%A8%EC%A0%90)
