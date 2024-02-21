# Spring Boot Logging 설정

## 처음 프로젝트를 시작하면서 프레임워크를 구성할 때..

- 가장 먼저 해야하는 설정, 로깅 설정!
- 스프링 부트에선 기본적으로 **Logback**이 설정되어 있다. 다음과 같이 **spring-boot-starter-logging** 라이브러리에 기본적으로 설치되어 있어서 SLF4J의 3가지 모듈이 Logback과 연결된다.
- 기본적인 로깅 방법

![스크린샷 2024-02-20 오후 11.56.07.png](%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202024-02-20%20%EC%98%A4%ED%9B%84%2011.56.07.png)

![스크린샷 2024-02-20 오후 11.56.55.png](%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202024-02-20%20%EC%98%A4%ED%9B%84%2011.56.55.png)

- 그러나 모두 찍히지는 않고 info, warn, error 레벨의 로그만 콘솔창에서 확인할 수 있다. 그 이유는 로그 레벨은 기본적으로 **[trace > debug > info > warn > error]** 순인데 디폴트 설정이 info로 되어 있기 때문에 그보다 하위 단계인 trace와 debug는 기록되지 않은 것이다.

## logback-spring.xml으로 콘솔에 로그 출력하기

- 로깅을 하면 개발시 운영 중 발생하는 문제점을 모니터링하거나 추적하는데 용이하고 데이터를 분석해 통계를 낼 수도 있다.
- 따라서 로그 레벨을 조절하여 콘솔에 출력하고 이를 기록하는 데에 목적이 있고 개발 환경과 배포 환경에 따라 다른 로그 설정값을 줄 수 있다.
- xml 설정에서 주요 컴포넌트로는 appender와 logger 두 가지로 나눌 수 있다.
    - **appender**: logger를 어디에 출력할지 설정한다. 콘솔, 파일, DB 등 지정할 수 있다.
    - **logger**: 아래 예제는 매우 단순한 형태로 실제 업무에선 FILE 뿐만 아니라 SockerAppender나 LogStash 등도 함께 설정해서 사용할 수 있다.
- Log Pattern
    - **`%d{yyyy-MM-dd HH:mm:ss.SSS}`**: 날짜 및 시간
    - **`%thread`**: 현재 스레드의 이름
    - **`[%X{traceId}]`**: MDC (Mapped Diagnostic Context)에서 "traceId"라는 키로 설정된 값의 출력
    - **`%-5level`**: 로그 레벨을 나타냅니다. 여기서는 왼쪽 정렬된 5자리의 공간을 할당하고 레벨을 표시
    - **`%logger`**: 로거의 이름
    - **`%m`**: 실제 로그 메시지
    - **`%n`**: 새로운 줄을 표시
![스크린샷 2024-02-21 오전 2.02.10.png](%EC%8A%A4%ED%81%AC%EB%A6%B0%EC%83%B7%202024-02-21%20%EC%98%A4%EC%A0%84%202.02.10.png)

- 가독성을 위해 로그 색상을 지정한다.
    - %clr(pattern){color}: 로그 pattern을 %clr ~ {color}로 감싸주면 된다.
    - springboot에서 제공하는 콘솔 로그 패턴 색상

      [[org.springframework.boot.logging.logback 패키지> deafults.xml 파일 참고](https://github.com/spring-projects/spring-boot/blob/main/spring-boot-project/spring-boot/src/main/resources/org/springframework/boot/logging/logback/defaults.xml)]


```xml
<!--    콘솔 로그에 색상 입히기       -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />

    <!-- 콘솔 로깅 설정 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %clr(%5level) %cyan(%logger) - %msg%n</pattern>
        </encoder>
    </appender>
```

## RollingFileAppender 파일 로깅

- RollingFileAppender는 FileAppender를 상속하여 로그 파일을 rollover한다.
- rollover는 다음 파일로 이동하는 행위로 한 로그 파일에 무한정 기록할 수 없으니 특정 기준에 따라 기록하는 파일 대상을 바꿔주는 것이다.
- 해당 appender는 주요 2가지 설정을 해줘야 한다.
    - **RollingPolicy**: rollover에 필요한 action을 설정한다. (ex. TimeBasedPolicy, SizeAndTimeBasedRollingPolicy 등)
    - **TriggeringPolicy**: rollover가 발생하는 기준(정책)을 설정한다.

```xml
<!-- 로깅을 파일로 저장 -->
    <appender name="LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>./${LOGS_ABSOLUTE_PATH}/coupon.log</file>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <charset>utf8</charset>
            <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %thread [%X{traceId}] %-5level %logger - %m%n</Pattern>
        </encoder>
        <!--로깅 파일이 특정 조건을 넘어가면 다른 파일로 만들어 준다.-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>./${LOGS_ABSOLUTE_PATH}/coupon.%d{yyyy-MM-dd}.%i.gz</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>180</maxHistory>
        </rollingPolicy>
    </appender>
```

## 개발 환경에 따른 로그 출력

- dev 환경에서는 모든 로그를 debug 수준에서 출력하고 파일에는 기록하지 않는다.
- prod 환경에서는 모든 로그를 info 수준에서 출력하고 파일에 기록한다.

```xml
  	<springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="STDOUT"/>
        </root>
    </springProfile>

    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="STDOUT"/>
        </root>
    </springProfile>

    <!--  쿼리문 로그에 출력되어 있는 파라미터에 바인딩 되는 값을 알 수 있음  -->
    <logger name="org.hibernate.type.descriptor.sql" additivity="false">
        <level value = "TRACE" />
        <appender-ref ref="STDOUT"/>
    </logger>
```

## 최종 logback-spring.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!--    파일을 저장할 위치 지정       -->
    <property name="LOGS_ABSOLUTE_PATH" value="./logs"/>
    <!--    콘솔 로그에 색상 입히기       -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter" />

    <!-- 콘솔 로깅 설정 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %clr(%5level) %cyan(%logger) - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 로깅을 파일로 저장 -->
    <appender name="LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>./${LOGS_ABSOLUTE_PATH}/coupon.log</file>
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <charset>utf8</charset>
            <Pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %thread [%X{traceId}] %-5level %logger - %m%n</Pattern>
        </encoder>
        <!--로깅 파일이 특정 조건을 넘어가면 다른 파일로 만들어 준다.-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>./${LOGS_ABSOLUTE_PATH}/coupon.%d{yyyy-MM-dd}.%i.gz</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>100MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>180</maxHistory>
        </rollingPolicy>
    </appender>

    <!--    logging level 지정 TRACE, DEBUG, INFO, WARN, ERROR, FATAL/OFF-->

		<!--   배포 환경    -->
    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="STDOUT"/>
            <appender-ref ref="LOG"/>
        </root>
    </springProfile>

		<!--   개발 환경    -->
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="STDOUT"/>
        </root>
        <!--  쿼리문 로그에 출력되어 있는 파라미터에 바인딩 되는 값을 알 수 있음  -->
        <logger name="org.hibernate.type.descriptor.sql" additivity="false">
            <level value = "TRACE" />
            <appender-ref ref="STDOUT"/>
        </logger>
    </springProfile>


</configuration>
```