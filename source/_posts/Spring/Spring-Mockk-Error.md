---
layout: post
title: 알 수 없는 ConsoleAppender 에러가 발생하는 경우
date: 2022-04-17 20:01:00 +0900
cover: /covers/spring.png
disqusId: d0d2895d7faf269c643ae3786d88e09faa640365
toc: true
category: Spring
tags:
- spring
- spring-boot
---

`ch.qos.logback.core.ConsoleAppender[console] - Appender [console] failed to append. io.mockk.MockKException: No other calls allowed in stdObjectAnswer than equals/hashCode/toString` 에러가 나는 경우 원인을 알아보도록 하겠습니다.

<!-- more -->

어느날 아래와 같은 에러가 나면서 테스트 일부가 실패하는 현상이 나타났습니다.

> ERROR in ch.qos.logback.core.ConsoleAppender[console] - Appender [console] failed to append. io.mockk.MockKException: No other calls allowed in stdObjectAnswer than equals/hashCode/toString

Stacktrace 에는 제가 작성한 코드가 존재하지 않고, 구글에 검색해도 해결 방법을 전혀 찾을 수 없었습니다.
정말 긴 시간의 삽질을 한 이후에 원인을 찾을 수 있었습니다.

# 원인

알고보니 다른 테스트에서 `Calendar.getInstance()`를 mockking 한 것이 원인이었습니다.
아래와 같은 과정 때문에 해당 에러가 발생한 것으로 추정됩니다.

1. 특정 테스트에서 `Calendar.getInstance()`가 mock을 반환하도록 설정
1. 이 설정을 해제 하지 않은 상태로 테스트 종료
1. 그 후의 다른 테스트에서 Spring Context를 로드하는 과정에서 `Calendar.getInstance()`를 실행하여 Calendar 인스턴스를 생성함.
1. 하지만 그 인스턴스는 mock 객체였고 이 mock 객체를 사용하는 과정에서 위와 같은 에러가 발생

# 재현 방법

```xml logback-spring.xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <Pattern>
                %d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n
            </Pattern>
        </layout>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
    </root>
</configuration>
```

```kotlin CustomTest.kt
class CustomTest: FreeSpec({
    beforeSpec {
        mockkStatic(Calendar::class)
        val calendar = mockk<Calendar>()
        every { Calendar.getInstance() } returns calendar
    }

    "test" {
        Thread.sleep(1000)
    }
})
```

```kotlin TmpApplicationTest.kt
@SpringBootTest
class TmpApplicationTest {
    @Test
    fun contextLoads() {
        println("contextLoads")
    }
}
```

위와 같이 코드를 작성하고 CustomTest가 TmpApplicationTest 보다 먼저 실행되면 TmpApplicationTest에서 위와 같은 에러가 발생합니다.

# 결론

static 메서드의 mockking은 신중하게 다뤄야하고, 꼭 테스트가 종료될 때 모든 mock을 해제 해야 합니다.
