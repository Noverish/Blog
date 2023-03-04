---
layout: post
title: Spring Boot - WebSocket
date: 2023-03-04 15:02:00 +0900
cover: /covers/spring.png
disqusId: 67ed5863ecb3e4c785e7f7071ec80ffdd8cfa08f
toc: true
category: Spring
tags:
- spring
- spring-boot
- websocket
---

Spring Boot 에서 WebSocket을 Controller 처럼 사용하는 법을 알아보겠습니다.

<!-- more -->

# 0. 들어가며

### STOMP

[STOMP](https://stomp.github.io/)는 HTTP처럼 Text 기반 메시징 프로토콜입니다.
HTTP 프로토콜 같다고 생각하시면 됩니다.

spring-web이 HTTP의 path를 기반으로 `@RequestMapping`에 라우팅을 해준다면    
spring-websocket은 STOMP의 destination을 기반으로 `@MessageMapping`에 라우팅 해줍니다.

### SockJS

![caniuse-websocket](/imgs/caniuse-websocket.png)

[SockJS](https://github.com/sockjs/sockjs-client)는 WebSocket을 사용할 수 없는 브라우저에서도 다른 방법을 이용해 WebSocket과 같은 동작을 수행할 수 있게 해주는 라이브러리입니다.

하지만 위 사진에서 알 수 있듯이 요즘 모든 브라우저에서는 WebSocket을 지원하기 때문에, 굳이 SockJS를 사용할 필요가 없습니다.

따라서 우리 예제에서는 SockJS를 사용하지 않습니다. 만약 필요하다면 이 글을 전부 따라하신 다음에 [별도 가이드](https://stomp-js.github.io/guide/stompjs/rx-stomp/using-stomp-with-sockjs.html)를 참고해주세요.

# 1. Dependency 추가

```gradle build.gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-websocket'
}
```

# 2. Spring Boot 설정

```java WebSocketConfiguration.java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfiguration implements WebSocketMessageBrokerConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/socket")
            .setAllowedOriginPatterns("*");
    }
}
```

- 실제 `/socket` 경로로 웹 소켓 연결을 합니다. nginx 에서 웹 소켓 설정을 할 때 이 경로로 하시면 됩니다.

```java WebSocketController.java
@Controller
public class WebSocketController {
    @MessageMapping("/ping")
    public String ping(WebSocketPayload payload) {
        return payload.toString();
    }
}
```

- 여기서 `/ping`은 HTTP 상의 경로가 아니라 STOMP 프로토콜 상의 destination 경로입니다.

# 3. Web 설정

[StompJS 문서](https://stomp-js.github.io/)

```shell
$ npm install @stomp/stompjs
```

```javascript
import * as StompJS from '@stomp/stompjs';

const stomp = new StompJS.Client({
  brokerURL: `wss://${window.location.host}/socket`,
});

// 아래와 같이 연결을 시작해야 메시지를 발송할 수 있습니다.
stomp.activate();

// 아래와 같이 메시지를 발송하시면 됩니다.
stomp.publish(({
  destination: '/ping',
  body: JSON.stringify({ message: 'Hello, World!' }),
}));

// 아래와 같이 연결을 종료 할 수 있습니다.
stomp.deactivate();
```

# 4. nginx 설정 

```conf nginx.conf
location /socket {
    proxy_pass http://localhost:8080
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_cache_bypass $http_upgrade;
}
```

# 5. Interceptor 설정

```java WebSocketInterceptor.java
public class WebSocketInterceptor implements HandshakeInterceptor {
    @Override
    public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) {
        String property = parseProperty(request);
        attributes.put("property", property);
        return true;
    }

    @Override
    public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception exception) {

    }
}
```

- 위와 같이 attributes 에 값을 넣으면 각 Controller 에서 해당 값을 꺼내볼 수 있습니다.
- `beforeHandshake`의 리턴 값을 false로 하면 연결을 거절할 수 있습니다.

```diff WebSocketConfiguration.java
@Configuration
@EnableWebSocketMessageBroker
public class WebSocketConfiguration implements WebSocketMessageBrokerConfigurer {
    @Override
    public void registerStompEndpoints(StompEndpointRegistry registry) {
        registry.addEndpoint("/socket")
+           .addInterceptors(WebSocketInterceptor())
            .setAllowedOriginPatterns("*");
    }
}
```

위와 같이 Interceptor를 추가합니다.

```java WebSocketController.java
@Controller
public class WebSocketController {
    @MessageMapping("/ping")
    public String ping(WebSocketPayload payload, SimpMessageHeaderAccessor accessor) {
        String property = (String) accessor.getSessionAttributes().get("property");
        return payload.toString();
    }
}
```

`SimpMessageHeaderAccessor`를 두 번째 파라미터에 추가한 다음에 위와 같이 Interceptor에서 추가한 값을 꺼내볼 수 있습니다.
