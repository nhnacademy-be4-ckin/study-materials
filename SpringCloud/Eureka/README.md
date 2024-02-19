# Spring Cloud Netflix Eureka (+ CI/CD 무중단배포)

## 목차

1. Spring Cloud Netflix Eureka (Load Balancing)
2. 무중단 배포

## 1. Spring Cloud Netflix Eureka

지난 시간의 내용

![gateway](resources/gateway.png)

Front → **Gateway** → API

> Gateway를 통해 **API**의 흐름을 제어

### if..

1. 수많은 서비스(인스턴스)를 관리해야한다면?
2. 해당 인스턴스 IP, PORT 번호를 관리할 수 있을까?

ex. `Netflix` , `Amazon`

![msa.png](resources/msa.png)

위의 사진처럼 관리해야할 서비스가 존재한다고 생각해보면 어떻게 관리?

- 이러한 문제점을 해결하기 위해서 Netlifx가 만든 `Spring Cloud Netflix`  사용!
- _`Eureka Server`, `Eureka Client`로 서비스를 관리_

## 1-1. Service Discovery Pattern

- MSA 환경에서는 서로간의 서비스를 원격 호출하기 위해서 각 서비스의 IP, PORT 번호를 알아야 호출 가능
    - 클라우드 기반의 MSA 환경 -> 네트워크 주소가 동적으로 할당
    - 이러한 문제를 해결하기 위해서는 클라이언트가 서비스를 호출하기 위해서 서비스를 찾는 매커니즘 필요

-> `Service Discovery Pattern`

### Service Discovery 기능
- 서비스를 등록하고 등록된 서비스 목록 반환
- health check 통해 현재 서비스 상태 판별
- 서비스 부하 분산 비율 조정

### Client Side Discovery
- 서비스 클라이언트가 서비스 레지스트리에서 서비스의 위치를 찾아서 호출하는 방식
- [Spring Cloud LoadBalancer](https://spring.io/guides/gs/spring-cloud-loadbalancer/)

### Server Side Discovery
- proxy server(로드 밸런서, 스프링 클라우드 게이트웨이) 제공
- Service registry(Eureka)에 등록된 서비스 위치 기반으로 라우팅


> Netflix Eureka는 **Client Side Discovery** 사용


## 1-2. Spring Cloud Netflix Eureka 사용

### Eureka Server

_eureka server - Main (Eureka)_

```java

@EnableEurekaServer
@SpringBootApplication
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class, args);
    }

}
```

- `@EnableEurekaServer` 어노테이션을 통해 Eureka Server로 설정
- Eureka Server는 Eureka Client 정보를 관리

### Eureka Client

_eureka client - Main (Gateway)_

```java

@EnableEurekaClient
@SpringBootApplication
@ConfigurationPropertiesScan
public class GatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(GatewayApplication.class, args);
    }

}
```

- `@EnableEurekaClient` 어노테이션을 통해 Eureka Client 설정
- Eureka Client는 Eureka Server에 자신의 정보를 등록
- 각 클라이언트들의 application-prod.properties 파일에 Eureka 설정을 추가

```properties
# 서비스 이름
spring.application.name=ckin-gateway-service

# Eureka 설정
eureka.client.register-with-eureka=true
eureka.client.fetch-registry=true
eureka.client.service-url.defaultZone=http://admin:1234@133.186.213.88:8761/eureka
eureka.instance.prefer-ip-address=true
management.endpoints.web.exposure.include=health,info

# Gateway 정보 설정 (입력을 안할경우 Docker Container의 IP로 설정)
eureka.instance.instance-id=gateway
eureka.instance.ip-address=133.186.209.180
```

### Gateway - Route 설정
```java
@Configuration
@RequiredArgsConstructor
public class RouteConfig {
    private final ServerInfoProperties serverInfoProperties;
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder) {
        return routeLocatorBuilder.routes()
                .route("ckin-auth", r -> r.path("/auth/**")
                        .uri(serverInfoProperties.getAuthUri()))
                .route("ckin-coupon", r -> r.path("/coupon/**")
                        .uri(serverInfoProperties.getCouponUri()))
                .route("ckin-api", r -> r.path("/api/**")
                        .uri("lb://CKIN-API-SERVICE"))
                .build();
    }
}
```

> CKIN-API-SERVICE : Eureka Server에 등록된 API 서비스 이름
>
> ![img.png](resources/api_server.png)

- `.route(...).uri("lb://CKIN-API-SERVICE")`에 작성한 것 처럼 실제 포트번호를 작성하지 않고 Eureka Server에 등록된 서비스 이름으로 호출
- 현재는 따로 설정을 하지 않아 `Round Robin` 방식으로 호출
- [로드밸런싱 테스트](http://133.186.209.180:9010/api/eureka)



## 2. 무중단 배포

> 만약 API 서버에 업데이트를 해야되는 경우, 기존의 서비스를 중단시키지 않고 업데이트를 진행하려면?

- 현재 우리의 API 서버는 2개의 인스턴스로 구성되어 있음
- 로드밸런싱을 통해 서비스를 분산
- 이러한 경우 `무중단 배포`를 통해 서비스를 업데이트 가능

### 2-1. 무중단 배포 방법

> 설명을 위해 api-1, api-2라고 가정, CKIN 프로젝트라고 생각

- 기존 shell script
```shell
#!/bin/sh

netstat -tuln | grep ":7030"  # 실행중인 포트번호 잡기

if [ $? -eq 0 ]; then         # 실행중인 도커가 있다면

        docker stop api # 실행중인 도커 멈추기
        docker rm api     # 도커 컨테이너 삭제
        docker rmi nhnckin/api # 도커 이미지 삭제

fi

docker pull nhnckin/api    # 도커 받아오기

docker run -d -e "SPRING_PROFILES_ACTIVE=prod" --name api -p 7030:7030 nhnckin/api # 도커 실행
```

- 위의 스크립트를 그냥 실행한다면 오류가 발생!
- api-1, api-2가 동시에 중단
  - Eureka Server에 등록된 서비스가 존재하지않아 요청을 받아줄 서비스가 없음


#### 아래와 같은 순서로 진행해야 무중단 배포가 가능!

1. api-1 인스턴스에 업데이트 진행
   - CI/CD (Jenkins, Github Actions) 등을 통해 배포 
   - 서비스를 DOWN 상태로 변경 (캐싱 되어있는 시간 고려)
   - docker stop 후 docker rm, rmi
   - 새로운 docker image 받아오기
   - docker 실행 (Eureka Server에 등록되는 시간 고려)
2. api-1 인스턴스 업데이트가 완료되었을 경우 api-2 인스턴스 업데이트 진행
   - 위와 내용 동일
3. 무중단 배포 완료


## 구조

![structure](resources/structure.png)


### 👀 추가 내용
> [스케일 업, 스케일 아웃](https://tecoble.techcourse.co.kr/post/2021-10-12-scale-up-scale-out/)