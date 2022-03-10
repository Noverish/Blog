---
layout: post
title: Spring Boot - 커스텀 Tomcat 에러 페이지 설정하기
date: 2022-03-10 09:37:00 +0900
cover: /covers/spring.png
disqusId: 46755ed445f2379a2c42988729cb333735c5bd47
toc: true
category: Spring
tags:
- spring
- spring-boot
- tomcat
---

Spring에서 커스텀 Tomcat 에러 페이지를 설정 하는 법을 알아보겠습니다.

<!-- more -->

# 1. Tomcat 에러 페이지


![Tomcat 에러 페이지](./tomcat-error-page.png)

Spring Boot 프로젝트에서 `http://localhost:8080/[]` 로 접근하면 위와 같은 페이지를 볼 수 있습니다.
이 페이지는 우리가 늘 보던 Whitelabel 에러 페이지와는 다른 모습입니다.

![Spring의 HTTP 요청 처리 흐름 (<a href="https://taes-k.github.io/2020/02/16/servlet-container-spring-container/">그림 출처</a>)](./spring-tomcat.png)

[HTTP/1.1 명세](https://tools.ietf.org/rfc/rfc7230.txt)에 따르면 URI 경로에 포함할 수 없는 특수 문자들이 정의되어 있는데,
이러한 특수 문자를 포함하여 HTTP 요청을 보내면 스프링 컨테이너로 전달되기 전 Tomcat 단에서 에러 페이지를 응답하기 때문에 그렇습니다.

따라서 Spring 에러 페이지 설정과는 다른 방법의 설정이 필요합니다.

# 2. 커스텀 Tomcat 에러 페이지 정의

아래와 같이 `ErrorReportValve` 를 상속하여 커스텀 Tomcat 에러 페이지를 정의합니다.

```java
import java.io.IOException;
import java.io.Writer;
import org.apache.catalina.connector.Request;
import org.apache.catalina.connector.Response;
import org.apache.catalina.valves.ErrorReportValve;

public class CustomTomcatErrorValve extends ErrorReportValve {
    protected void report(Request request, Response response, Throwable throwable) {
        if (!response.setErrorReported())
            return;

        try {
            Writer writer = response.getReporter();
            writer.write("<h1>This is custom tomcat error page!</h1>");
            response.finishResponse();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

- line 9: 이 메서드에서 에러 페이지를 응답했다는 것을 기록하기 위한 용도 입니다. 이미 응답이 된 경우에는 바로 return 합니다.
- line 14: 응답할 에러 페이지를 정의합니다.

# 3. 커스텀 Tomcat 에러 페이지 등록

아래와 같이 `TomcatWebSocketServletWebServerCustomizer`를 상속하여 위에서 정의한 커스텀 Tomcat 에러 페이지를 등록하는 Bean을 생성합니다.

```java
import org.apache.catalina.Container;
import org.apache.catalina.core.StandardHost;
import org.springframework.boot.autoconfigure.websocket.servlet.TomcatWebSocketServletWebServerCustomizer;
import org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory;
import org.springframework.stereotype.Component;

@Component
public class TomcatErrorPageCustomizer extends TomcatWebSocketServletWebServerCustomizer {
    @Override
    public void customize(TomcatServletWebServerFactory factory) {
        factory.addContextCustomizers((context) -> {
            Container parent = context.getParent();
            if (parent instanceof StandardHost) {
                ((StandardHost)parent).setErrorReportValveClass("kim.hyunsub.demo.CustomTomcatErrorValve");
            }
        });
    }

    @Override
    public int getOrder() {
        return 100;
    }
}
```

- line 14: 위에서 정의한 `CustomTomcatErrorValve`의 패키지 경로를 입력합니다.
- line 21: Spring에서 정의하는 `TomcatWebServerFactoryCustomizer`가 먼저 설정되어야 하므로 0을 초과하는 값으로 우선순위를 정해줍니다.
  [Github 코드](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/embedded/TomcatWebServerFactoryCustomizer.java#L76)

자세한 정보는 [여기](https://github.com/spring-projects/spring-boot/issues/21257#issuecomment-745565376)를 확인해주세요.

# 4. 결과

위와 같이 설정하고 다시 `http://localhost:8080/[]`에 접속하면 아래와 같이 우리가 설정한 커스텀 Tomcat 에러 페이지를 볼 수 있습니다.

![커스텀 Tomcat 에러 페이지](./custom-tomcat-error-page.png)

# 부록. URI 경로에 특수 문자 허용

[HTTP/1.1 명세](https://tools.ietf.org/rfc/rfc7230.txt)에 따르면 URI 경로에 포함할 수 없는 특수 문자들이 정의되어 있습니다.
이러한 특수 문자를 Spring Configuration Property로 허용해 줄 수 있습니다.

```
server.tomcat.relaxed-path-chars=",<,>,[,\,],^,`,{,|,}
server.tomcat.relaxed-query-chars=",<,>,[,\,],^,`,{,|,}
```

자세한 정보는 [Spring 공식 문서](https://docs.spring.io/spring-boot/docs/current/reference/html/application-properties.html#application-properties.server.server.tomcat.relaxed-path-chars)와
[Tomcat 공식 문서](https://tomcat.apache.org/tomcat-9.0-doc/config/http.html)를 확인해주세요
