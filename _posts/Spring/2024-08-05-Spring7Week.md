---
title: "스프링 핵심가이드 7주차"
excerpt: "[제로베이스] 스프링 핵심가이드"

categories:
  - Spring
tags:
  - [Spring]

permalink: /categories/Spring7Week/ # url

toc: true
toc_sticky: true

date: 2024-08-05
last_modified_at: 2024-08-05
---

[스프링부트 핵심가이드 프로젝트 따라하면서 짠 코드](https://github.com/seokseungmin/valid_exception)<br>

본 게시글은 '스프링 부트 핵심 가이드' 책의 내용을 정리한 것입니다.<br>
저자 : 장정우<br>
출판사 : 위키북스<br>

# 프로젝트에 종속성 추가 및 엔드포인트 설정

애플리케이션을 개발하는 단계를 지나, 운영 단계에 접어들면 애플리케이션이 정상적으로 동작하는지 모니터링하는 환경을 구축하는 것이 매우 중요해진다.

스프링 부트 액추에이터는 HTTP 엔드포인트나 JMX를 활용해 애플리케이션을 모니터링하고 관리할 수 있는 기능을 제공한다.

## JMX란?
Java Management Extensions(JMX)는 실행 중인 애플리케이션의 상태를 모니터링하고 설정을 변경할 수 있게 해주는 API이다. JMX를 통해 리소스 관리를 하려면 MBeans(Managed Beans)를 생성해야 한다.

### 종속성 추가
```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}
```

## Swagger2 호환성 문제
Swagger2가 `spring-boot-starter-actuator`와 호환되지 않는다는 이슈가 있었다. [Stack Overflow](https://stackoverflow.com/questions/70036953/spring-boot-2-6-0-spring-fox-3-failed-to-start-bean-documentationpluginsboo)

Swagger2를 지우고 `springdoc`을 추가하여 해결하였다. `springdoc`도 Swagger처럼 많은 설정을 할 수 있으나, 해주지 않아도 사용 가능하다!

### Swagger2 종속성 제거 후 springdoc 종속성 추가
```groovy
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.0.2'
```
기존에 작성했던 Swagger2 관련 파일을 제거한다.

### application 설정 파일의 pathmatch 설정 제거
```properties
spring.mvc.pathmatch.matching-strategy=ant_path_matcher
```

실행 후 Swagger UI 진입 경로: `localhost:8080/swagger-ui/index.html`

## 엔드포인트
액추에이터의 엔드포인트는 애플리케이션의 모니터링을 위한 경로이다. 스프링 부트에는 여러 내장 엔드포인트가 포함돼 있으며 커스텀 엔드포인트를 추가할 수도 있다.

액추에이터를 추가하면 기본적으로 엔드포인트 URL로 `/actuator`가 추가되며, 이 뒤에 경로를 추가해 상세 내역에 접근할 수 있다.

만약 `/actuator`가 아닌 다른 경로를 사용하고 싶다면 `application.properties` 또는 `application.yml` 파일에 다음과 같이 작성해준다.
```properties
management.endpoints.web.base-path=/custom-path
```

### 자주 사용되는 액추에이터 엔드포인트

| ID                | 설명                                                                                  |
|-------------------|---------------------------------------------------------------------------------------|
| `auditevents`     | `AuditEventRepository` 빈이 필요. 호출된 Audit 이벤트 정보를 표시                      |
| `beans`           | 애플리케이션에 있는 모든 스프링 빈 리스트 표시                                         |
| `caches`          | 사용 가능한 캐시 표시                                                                 |
| `conditions`      | 자동 구성 조건 내역 생성                                                              |
| `configprops`     | `@ConfigurationProperties`의 속성 리스트 표시                                         |
| `env`             | 애플리케이션에서 사용할 수 있는 환경 속성 표시                                        |
| `health`          | 애플리케이션의 상태 정보 표시                                                         |
| `httptrace`       | 가장 최근에 이뤄진 100건의 요청 기록 표시. `HttpTraceRepository` 빈 필요.             |
| `info`            | 애플리케이션의 정보 표시                                                              |
| `integrationgraph`| 스프링 통합 그래프 표시. `spring-integration-core` 모듈에 대한 의존성 추가 필요        |
| `loggers`         | 애플리케이션의 로거 구성을 표시, 수정                                                 |
| `metrics`         | 애플리케이션의 메트릭 정보 표시                                                       |
| `mappings`        | 모든 `@RequestMapping`의 매핑 정보 표시                                               |
| `quartz`          | Quartz 스케줄러 작업에 대한 정보 표시                                                 |
| `scheduledtasks`  | 애플리케이션에서 예약된 작업 표시                                                     |
| `sessions`        | 스프링 세션 저장소에서 사용자의 세션을 검색하고 삭제 할 수 있다. 스프링 세션을 사용하는 서블릿 기반 웹 애플리케이션이 필요하다. |
| `shutdown`        | 애플리케이션을 정상적으로 종료할 수 있다. 디폴트: 비활성화                             |
| `startup`         | 애플리케이션이 시작될 때 수집된 시작 단계 데이터를 표시한다. `BufferingApplicationStartup`으로 구성된 스프링 애플리케이션이 필요하다. |
| `threaddump`      | 스레드 덤프를 수행                                                                     |

만약 Spring MVC, Spring WebFlux, Jersey를 사용한다면 추가로 다음과 같은 엔드포인트를 사용할 수 있다.

| ID                | 설명                                                                                  |
|-------------------|---------------------------------------------------------------------------------------|
| `heapdump`        | 힙 덤프 파일을 반환. 핫스팟 HotSpot VM 상에서 hprof 포맷의 파일이 반환되며, OpenJ9JVM 에서는 PHD 포맷 파일을 반환한다. |
| `jolokia`         | Jolokia가 클래스패스에 있을 때 HTTP를 통해 JMX 빈을 표시한다. `jolokia-core` 모듈에 대한 의존성 추가가 필요. WebFlux에서는 사용할 수 없다. |
| `logfile`         | `logging.file.name` 또는 `logging.file.path` 속성이 설정돼 있는 경우 로그 파일의 내용 반환 |
| `prometheus`      | Prometheus 서버에서 스크랩할 수 있는 형식으로 메트릭 표시. `micrometer-registry-prometheus` 모듈의 의존성 추가 필요. |

엔드포인트는 활성화 여부와 노출 여부를 설정할 수 있다. 활성화는 기능 자체를 활성화할 것인지를 결정하는 것으로, 비활성화된 엔드포인트는 애플리케이션 컨텍스트에서 완전히 제거된다.

엔드포인트를 활성화하려면 `application` 설정 파일에 액추에이터 속성을 추가하면 된다.
```properties
management.endpoints.web.base-path=/actuator

## End point Active
management.endpoint.shutdown.enabled=true
management.endpoint.caches.enabled=false
```
이 설정은 엔드포인트 기본 경로를 `/actuator`로 변경하고, shutdown 기능은 활성화, caches 기능은 비활성화하겠다는 의미이다.

또한, 액추에이터 설정을 통해 기능 활성화/비활성화가 아니라 엔드포인트의 노출 여부만 설정하는 것도 가능하다. 노출 여부는 JMX를 통한 노출과 HTTP를 통한 노출이 있어 설정이 구분된다.

### HTTP 노출 설정
```properties
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=threaddump,heapdump
```

### JMX 노출 설정
```properties
management.endpoints.jmx.exposure.include=*
management.endpoints.jmx.exposure.exclude=threaddump,heapdump
```
위의 두 설정 모두 엔드포인트를 전체적으로 노출하며, 스레드 덤프와 힙 덤프 기능은 제외한다는 의미이다. 엔드포인트는 애플리케이션에 관한 민감한 정보를 포함하고 있기 때문에 노출 설정을 신중하게 고려해야 한다.

### 노출 설정에 대한 기본값

| ID                | JMX | WEB |
|-------------------|-----|-----|
| `auditevents`     | ⭕  | X   |
| `beans`           | ⭕  | X   |
| `caches`          | ⭕  | X   |
| `conditions`      | ⭕  | X   |
| `configprops`     | ⭕  | X   |
| `env`             | ⭕  | X   |
| `health`          | ⭕  | ⭕  |
| `heapdump`        | 해당 없음 | X   |
| `httptrace`       | ⭕  | X   |
| `info`            | ⭕  | X   |
| `integrationgraph`| ⭕  | X   |
| `jolokia`         | 해당 없음 | X   |
| `logfile`         | 해당 없음 | X   |
| `loggers`         | ⭕  | X   |
| `liquibase`       | ⭕  | X   |
| `metrics`         | ⭕  | X   |
| `mappings`        | ⭕  | X   |
| `Prometheus`      | 해당 없음 | 해당 없음 |
| `quartz`          | ⭕  | X   |
| `scheduledtasks`  | ⭕  | X   |
| `sessions`        | ⭕  | X   |
| `shutdown`        | ⭕  | X   |
| `startup`         | ⭕  | X   |
| `threaddump`      | ⭕  | X   |

# 액추에이터 기능 살펴보기

액추에이터를 활성화하고 노출 지점도 설정하고 나면 애플리케이션에서 해당 기능을 사용할 수 있다.

모든 기능을 살펴보기 위해서는 다른 의존성을 추가하거나, 설정을 추가해야 하기 때문에 이번 실습에서는 기본 제공 기능 위주로 살펴본다.

## 앱 기본 정보 `/info`

가동 중인 애플리케이션의 정보를 볼 수 있다. 스프링 부트 2.5버전까지는 `/info` 엔드포인트에서 정보를 제공했으나, 이후 버전에서는 `/info`에 대한 정보를 기본적으로 노출시키지 않는다고 공식 문서에 나와 있다. 이를 활성화하려면 다음과 같은 설정이 필요하다:

```properties
management.endpoints.web.exposure.include=info
management.info.env.enabled=true
```

참고한 Stack Overflow 답변: [Spring Boot actuator endpoints unavailable after 2.5 upgrade](https://stackoverflow.com/questions/70036953/spring-boot-2-6-0-spring-fox-3-failed-to-start-bean-documentationpluginsboo)

## 앱 상태 `/health`

`/health` 엔드포인트를 이용하면 애플리케이션의 상태를 확인할 수 있다. 별도의 설정 없이 아래 경로로 접근 가능하다.

```
http://localhost:8080/actuator/health
```

접속하면 아래와 같은 결과를 만날 수 있다:

```json
{
    "status": "UP"
}
```

나타날 수 있는 `status`의 값으로는 `UP`, `DOWN`, `UNKNOWN`, `OUT_OF_SERVICE`가 있다. 이 결과는 네트워크 계층 중 L4 Loadbalancing 레벨에서 애플리케이션의 상태를 확인하기 위해 사용된다.

만약 상세 정보를 확인하고 싶다면 아래와 같이 설정할 수 있다:

```properties
management.endpoint.health.show-details=always
```

설정 후 실행 화면:

```json
{
    "status": "UP",
    "components": {
        "diskSpace": {
            "status": "UP",
            "details": {
                "total": 1022878543872,
                "free": 869693964288,
                "threshold": 10485760,
                "path": "C:\\Users\\Royse\\OneDrive\\바탕 화면\\actuator\\.",
                "exists": true
            }
        },
        "ping": {
            "status": "UP"
        }
    }
}
```

이때, 설정값으로 입력할 수 있는 것으로는 다음과 같다:

- `never` (default): 세부사항 표시 안함
- `when-authorized`: 승인된 사용자에게만 세부 상태 표시. 확인 권한은 application 설정파일에 추가한 `management.endpoint.health.roles` 속성으로 부여할 수 있다.
- `always`: 모든 사용자에게 세부 정보 표시

그리고 `status` 정보에 대해서 확인할 것은 모든 `status`의 값이 `UP` 상태일 때만 애플리케이션의 상태가 `UP`으로 표시된다. 하나라도 `DOWN` 상태인 항목이 있다면 애플리케이션의 상태도 `DOWN`으로 표기되며 HTTP 상태 코드도 변경된다.

## 빈 정보 확인 `/beans`

스프링 컨테이너에 등록된 스프링 빈의 전체 목록을 표시할 수 있다. JSON 형식으로 빈의 정보를 반환한다. 다만, 스프링은 워낙 많은 빈이 자동으로 등록되어 운영되기 때문에 관련 정보를 출력해서 육안으로 내용을 파악하기는 어렵다.

접속 경로:

```
http://localhost:8080/actuator/beans
```

```json
{
    "contexts": {
        "actuator": {
            "beans": {
                "endpointCachingOperationInvokerAdvisor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.invoker.cache.CachingOperationInvokerAdvisor",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/EndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.EndpointAutoConfiguration",
                        "environment"
                    ]
                },
                "defaultServletHandlerMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.HandlerMapping",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "swaggerWebMvcConfigurer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.ui.SwaggerWebMvcConfigurer",
                    "resource": "class path resource [org/springdoc/webmvc/ui/SwaggerConfig.class]",
                    "dependencies": [
                        "org.springdoc.webmvc.ui.SwaggerConfig",
                        "org.springdoc.core.properties.SwaggerUiConfigParameters",
                        "indexPageTransformer"
                    ]
                },
                "applicationTaskExecutor": {
                    "aliases": [
                        "taskExecutor"
                    ],
                    "scope": "singleton",
                    "type": "org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/task/TaskExecutorConfigurations$TaskExecutorConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$TaskExecutorConfiguration",
                        "taskExecutorBuilder"
                    ]
                },
                "observationRegistryPostProcessor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.ObservationRegistryPostProcessor",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/observation/ObservationAutoConfiguration.class]",
                    "dependencies": []
                },
                "characterEncodingFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.web.servlet.filter.OrderedCharacterEncodingFilter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/HttpEncodingAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration"
                    ]
                },
                "forceAutoProxyCreatorToUseClassProxying": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.aop.AopAutoConfiguration$ClassProxyingConfiguration$$Lambda$494/0x0000022a81378a50",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/aop/AopAutoConfiguration$ClassProxyingConfiguration.class]",
                    "dependencies": []
                },
                "healthEndpointGroups": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.health.AutoConfiguredHealthEndpointGroups",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthEndpointConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e",
                        "management.endpoint.health-org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties"
                    ]
                },
                "webConversionServiceProvider": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.providers.WebConversionServiceProvider",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration$WebConversionServiceConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration$WebConversionServiceConfiguration"
                    ]
                },
                "management.endpoint.health-org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties",
                    "dependencies": []
                },
                "webEndpointDiscoverer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.web.annotation.WebEndpointDiscoverer",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/WebEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration",
                        "endpointOperationParameterMapper",
                        "endpointMediaTypes"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration",
                    "dependencies": [
                        "spring.servlet.multipart-org.springframework.boot.autoconfigure.web.servlet.MultipartProperties"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration$DispatcherServletRegistrationConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration$DispatcherServletRegistrationConfiguration",
                    "dependencies": []
                },
                "preserveErrorControllerTargetClassPostProcessor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$PreserveErrorControllerTargetClassPostProcessor",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration.class]",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonObjectMapperBuilderConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonObjectMapperBuilderConfiguration",
                    "dependencies": []
                },
                "openAPIBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.service.OpenAPIService",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "securityParser",
                        "org.springdoc.core.properties.SpringDocConfigProperties",
                        "propertyResolverUtils"
                    ]
                },
                "logbackMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.logging.LogbackMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/LogbackMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.LogbackMetricsAutoConfiguration"
                    ]
                },
                "management.info-org.springframework.boot.actuate.autoconfigure.info.InfoContributorProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.info.InfoContributorProperties",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration",
                    "dependencies": [
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e",
                        "management.endpoints.web-org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties"
                    ]
                },
                "webEndpointPathMapper": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.web.MappingWebEndpointPathMapper",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/WebEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.cache.CachesEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.cache.CachesEndpointAutoConfiguration",
                    "dependencies": []
                },
                "propertySourcesPlaceholderConfigurer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.context.support.PropertySourcesPlaceholderConfigurer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/context/PropertyPlaceholderAutoConfiguration.class]",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointAutoConfiguration",
                    "dependencies": [
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e",
                        "management.endpoints.jmx-org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointProperties",
                        "spring.jmx-org.springframework.boot.autoconfigure.jmx.JmxProperties"
                    ]
                },
                "jmxMBeanExporter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.jmx.JmxEndpointExporter",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/jmx/JmxEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointAutoConfiguration",
                        "mbeanServer",
                        "endpointObjectNameFactory",
                        "jmxAnnotationEndpointDiscoverer"
                    ]
                },
                "management.simple.metrics.export-org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleProperties",
                    "dependencies": []
                },
                "beanNameViewResolver": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.view.BeanNameViewResolver",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration"
                    ]
                },
                "endpointObjectNameFactory": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.jmx.DefaultEndpointObjectNameFactory",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/jmx/JmxEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointAutoConfiguration",
                        "mbeanServer"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.endpoint.web.servlet.WebMvcEndpointManagementContextConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.web.servlet.WebMvcEndpointManagementContextConfiguration",
                    "dependencies": []
                },
                "viewResolver": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.view.ContentNegotiatingViewResolver",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter",
                        "org.springframework.beans.factory.support.DefaultListableBeanFactory@62e6b5c8"
                    ]
                },
                "management.endpoints.web.cors-org.springframework.boot.actuate.autoconfigure.endpoint.web.CorsEndpointProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.web.CorsEndpointProperties",
                    "dependencies": []
                },
                "stringHttpMessageConverter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.http.converter.StringHttpMessageConverter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/http/HttpMessageConvertersAutoConfiguration$StringHttpMessageConverterConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration$StringHttpMessageConverterConfiguration",
                        "environment"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration$TomcatWebServerFactoryCustomizerConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration$TomcatWebServerFactoryCustomizerConfiguration",
                    "dependencies": []
                },
                "tomcatServletWebServerFactoryCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.TomcatServletWebServerFactoryCustomizer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/ServletWebServerFactoryAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration",
                        "server-org.springframework.boot.autoconfigure.web.ServerProperties"
                    ]
                },
                "org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration",
                    "dependencies": []
                },
                "server-org.springframework.boot.autoconfigure.web.ServerProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.ServerProperties",
                    "dependencies": []
                },
                "messageConverters": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.http.HttpMessageConverters",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/http/HttpMessageConvertersAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration"
                    ]
                },
                "jsonComponentModule": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.jackson.JsonComponentModule",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/jackson/JacksonAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration"
                    ]
                },
                "websocketServletWebServerCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.websocket.servlet.TomcatWebSocketServletWebServerCustomizer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/websocket/servlet/WebSocketServletAutoConfiguration$TomcatWebSocketConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration$TomcatWebSocketConfiguration"
                    ]
                },
                "jmxAnnotationEndpointDiscoverer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.jmx.annotation.JmxEndpointDiscoverer",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/jmx/JmxEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointAutoConfiguration",
                        "endpointOperationParameterMapper"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration$ServletWebConfiguration$SpringMvcConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration$ServletWebConfiguration$SpringMvcConfiguration",
                    "dependencies": []
                },
                "configurationPropertiesReportEndpointWebExtension": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.context.properties.ConfigurationPropertiesReportEndpointWebExtension",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/context/properties/ConfigurationPropertiesReportEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointAutoConfiguration",
                        "configurationPropertiesReportEndpoint",
                        "management.endpoint.configprops-org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointProperties"
                    ]
                },
                "org.springframework.boot.autoconfigure.sql.init.SqlInitializationAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.sql.init.SqlInitializationAutoConfiguration",
                    "dependencies": []
                },
                "objectMapperProvider": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.providers.ObjectMapperProvider",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "org.springdoc.core.properties.SpringDocConfigProperties"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration",
                    "dependencies": []
                },
                "mappingJackson2HttpMessageConverter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.http.converter.json.MappingJackson2HttpMessageConverter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/http/JacksonHttpMessageConvertersConfiguration$MappingJackson2HttpMessageConverterConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.http.JacksonHttpMessageConvertersConfiguration$MappingJackson2HttpMessageConverterConfiguration",
                        "jacksonObjectMapper"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.context.ShutdownEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.context.ShutdownEndpointAutoConfiguration",
                    "dependencies": []
                },
                "meterRegistryPostProcessor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.MeterRegistryPostProcessor",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/MetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e"
                    ]
                },
                "mbeanExporter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.jmx.export.annotation.AnnotationMBeanExporter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/jmx/JmxAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration",
                        "objectNamingStrategy",
                        "org.springframework.beans.factory.support.DefaultListableBeanFactory@62e6b5c8"
                    ]
                },
                "endpointMediaTypes": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.web.EndpointMediaTypes",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/WebEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration"
                    ]
                },
                "jvmCompilationMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.jvm.JvmCompilationMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/JvmMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.JvmMetricsAutoConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointWebExtensionConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointWebExtensionConfiguration",
                    "dependencies": []
                },
                "healthStatusAggregator": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.health.SimpleStatusAggregator",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthEndpointConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointConfiguration",
                        "management.endpoint.health-org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.SystemMetricsAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.SystemMetricsAutoConfiguration",
                    "dependencies": []
                },
                "management.server-org.springframework.boot.actuate.autoconfigure.web.server.ManagementServerProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.server.ManagementServerProperties",
                    "dependencies": []
                },
                "spring.ssl-org.springframework.boot.autoconfigure.ssl.SslProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.ssl.SslProperties",
                    "dependencies": []
                },
                "mbeanServer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "com.sun.jmx.mbeanserver.JmxMBeanServer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/jmx/JmxAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration"
                    ]
                },
                "servletWebServerFactoryCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryCustomizer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/ServletWebServerFactoryAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration",
                        "server-org.springframework.boot.autoconfigure.web.ServerProperties"
                    ]
                },
                "mvcUrlPathHelper": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.util.UrlPathHelper",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.startup.StartupTimeMetricsListenerAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.startup.StartupTimeMetricsListenerAutoConfiguration",
                    "dependencies": []
                },
                "servletMappingDescriptionProvider": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.web.mappings.servlet.ServletsMappingDescriptionProvider",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/web/mappings/MappingsEndpointAutoConfiguration$ServletWebConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration$ServletWebConfiguration"
                    ]
                },
                "webServerFactoryCustomizerBeanPostProcessor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.web.server.WebServerFactoryCustomizerBeanPostProcessor",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.task.TaskSchedulingConfigurations$ThreadPoolTaskSchedulerBuilderConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskSchedulingConfigurations$ThreadPoolTaskSchedulerBuilderConfiguration",
                    "dependencies": []
                },
                "metricsHttpClientUriTagFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.config.MeterFilter$9",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/observation/web/client/HttpClientObservationsAutoConfiguration$MeterFilterConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.observation.web.client.HttpClientObservationsAutoConfiguration$MeterFilterConfiguration",
                        "management.observations-org.springframework.boot.actuate.autoconfigure.observation.ObservationProperties",
                        "management.metrics-org.springframework.boot.actuate.autoconfigure.metrics.MetricsProperties"
                    ]
                },
                "org.springdoc.core.configuration.SpringDocConfiguration$WebConversionServiceConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.configuration.SpringDocConfiguration$WebConversionServiceConfiguration",
                    "dependencies": []
                },
                "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration$SpringDocWebMvcActuatorConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration$SpringDocWebMvcActuatorConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration",
                    "dependencies": []
                },
                "management.health.diskspace-org.springframework.boot.actuate.autoconfigure.system.DiskSpaceHealthIndicatorProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.system.DiskSpaceHealthIndicatorProperties",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.scheduling.ScheduledTasksObservabilityAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.scheduling.ScheduledTasksObservabilityAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration",
                    "dependencies": []
                },
                "spring.sql.init-org.springframework.boot.autoconfigure.sql.init.SqlInitializationProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.sql.init.SqlInitializationProperties",
                    "dependencies": []
                },
                "controllerEndpointHandlerMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.web.servlet.ControllerEndpointHandlerMapping",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/servlet/WebMvcEndpointManagementContextConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.servlet.WebMvcEndpointManagementContextConfiguration",
                        "controllerEndpointDiscoverer",
                        "management.endpoints.web.cors-org.springframework.boot.actuate.autoconfigure.endpoint.web.CorsEndpointProperties",
                        "management.endpoints.web-org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties"
                    ]
                },
                "management.endpoint.env-org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointProperties",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.JvmMetricsAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.JvmMetricsAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$Jackson2ObjectMapperBuilderCustomizerConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$Jackson2ObjectMapperBuilderCustomizerConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration$StringHttpMessageConverterConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration$StringHttpMessageConverterConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration",
                    "dependencies": []
                },
                "standardJacksonObjectMapperBuilderCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$Jackson2ObjectMapperBuilderCustomizerConfiguration$StandardJackson2ObjectMapperBuilderCustomizer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/jackson/JacksonAutoConfiguration$Jackson2ObjectMapperBuilderCustomizerConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$Jackson2ObjectMapperBuilderCustomizerConfiguration",
                        "spring.jackson-org.springframework.boot.autoconfigure.jackson.JacksonProperties"
                    ]
                },
                "observabilitySchedulingConfigurer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.scheduling.ScheduledTasksObservabilityAutoConfiguration$ObservabilitySchedulingConfigurer",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/scheduling/ScheduledTasksObservabilityAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.scheduling.ScheduledTasksObservabilityAutoConfiguration",
                        "observationRegistry"
                    ]
                },
                "taskSchedulerBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.task.TaskSchedulerBuilder",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/task/TaskSchedulingConfigurations$TaskSchedulerBuilderConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.task.TaskSchedulingConfigurations$TaskSchedulerBuilderConfiguration",
                        "spring.task.scheduling-org.springframework.boot.autoconfigure.task.TaskSchedulingProperties"
                    ]
                },
                "metricsEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.metrics.MetricsEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/MetricsEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.MetricsEndpointAutoConfiguration",
                        "simpleMeterRegistry"
                    ]
                },
                "management.observations-org.springframework.boot.actuate.autoconfigure.observation.ObservationProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.ObservationProperties",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.aop.AopAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.aop.AopAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.info.InfoContributorAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.info.InfoContributorAutoConfiguration",
                    "dependencies": []
                },
                "metricsObservationHandlerGrouping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.ObservationHandlerGrouping",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/observation/ObservationAutoConfiguration$OnlyMetricsConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration$OnlyMetricsConfiguration"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration",
                    "dependencies": []
                },
                "simpleMeterRegistry": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.simple.SimpleMeterRegistry",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/export/simple/SimpleMetricsExportAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleMetricsExportAutoConfiguration",
                        "simpleConfig",
                        "micrometerClock"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration",
                    "dependencies": []
                },
                "environmentEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.env.EnvironmentEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/env/EnvironmentEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointAutoConfiguration",
                        "environment",
                        "management.endpoint.env-org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointProperties"
                    ]
                },
                "conventionErrorViewResolver": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.error.DefaultErrorViewResolver",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration$DefaultErrorViewResolverConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$DefaultErrorViewResolverConfiguration"
                    ]
                },
                "startupTimeMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.metrics.startup.StartupTimeMetricsListener",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/startup/StartupTimeMetricsListenerAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.startup.StartupTimeMetricsListenerAutoConfiguration",
                        "simpleMeterRegistry"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                    "dependencies": [
                        "spring.mvc-org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties",
                        "spring.web-org.springframework.boot.autoconfigure.web.WebProperties",
                        "org.springframework.beans.factory.support.DefaultListableBeanFactory@62e6b5c8",
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter",
                        "swaggerWebMvcConfigurer",
                        "endpointObjectMapperWebMvcConfigurer"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.web.tomcat.TomcatMetricsAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.web.tomcat.TomcatMetricsAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.observation.web.servlet.WebMvcObservationAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.web.servlet.WebMvcObservationAutoConfiguration",
                    "dependencies": []
                },
                "org.springdoc.core.configuration.SpringDocConfiguration$OpenApiResourceAdvice": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.configuration.SpringDocConfiguration$OpenApiResourceAdvice",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.LogbackMetricsAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.LogbackMetricsAutoConfiguration",
                    "dependencies": []
                },
                "spring.mvc-org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties",
                    "dependencies": []
                },
                "org.springdoc.core.configuration.SpringDocConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.configuration.SpringDocConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.observation.web.client.HttpClientObservationsAutoConfiguration$MeterFilterConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.web.client.HttpClientObservationsAutoConfiguration$MeterFilterConfiguration",
                    "dependencies": []
                },
                "localeCharsetMappingsCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration$LocaleCharsetMappingsCustomizer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/HttpEncodingAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.logging.LoggersEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.logging.LoggersEndpointAutoConfiguration",
                    "dependencies": []
                },
                "configurationPropertiesReportEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.context.properties.ConfigurationPropertiesReportEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/context/properties/ConfigurationPropertiesReportEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointAutoConfiguration",
                        "management.endpoint.configprops-org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointProperties"
                    ]
                },
                "formContentFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.web.servlet.filter.OrderedFormContentFilter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration"
                    ]
                },
                "jsonMixinModuleEntries": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.jackson.JsonMixinModuleEntries",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/jackson/JacksonAutoConfiguration$JacksonMixinConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e"
                    ]
                },
                "multipartConfigElement": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "jakarta.servlet.MultipartConfigElement",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/MultipartAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration"
                    ]
                },
                "requestContextFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.web.servlet.filter.OrderedRequestContextFilter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter.class]",
                    "dependencies": []
                },
                "defaultViewResolver": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.view.InternalResourceViewResolver",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter"
                    ]
                },
                "operationBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.service.OperationService",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "parameterBuilder",
                        "requestBodyBuilder",
                        "securityParser",
                        "propertyResolverUtils"
                    ]
                },
                "routerFunctionMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.function.support.RouterFunctionMapping",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "mvcConversionService",
                        "mvcResourceUrlProvider"
                    ]
                },
                "jacksonObjectMapperBuilder": {
                    "aliases": [],
                    "scope": "prototype",
                    "type": "org.springframework.http.converter.json.Jackson2ObjectMapperBuilder",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/jackson/JacksonAutoConfiguration$JacksonObjectMapperBuilderConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonObjectMapperBuilderConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e",
                        "standardJacksonObjectMapperBuilderCustomizer"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration$ServletWebConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration$ServletWebConfiguration",
                    "dependencies": []
                },
                "beansEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.beans.BeansEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/beans/BeansEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.beans.BeansEndpointAutoConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e"
                    ]
                },
                "management.endpoints.jmx-org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointProperties",
                    "dependencies": []
                },
                "actuatorApplication": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "com.springboot.actuator.ActuatorApplication$$SpringCGLIB$$0",
                    "dependencies": []
                },
                "spring.task.scheduling-org.springframework.boot.autoconfigure.task.TaskSchedulingProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskSchedulingProperties",
                    "dependencies": []
                },
                "healthEndpointWebExtension": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.health.HealthEndpointWebExtension",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthEndpointWebExtensionConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointWebExtensionConfiguration",
                        "healthContributorRegistry",
                        "healthEndpointGroups",
                        "management.endpoint.health-org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties"
                    ]
                },
                "multipartResolver": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.multipart.support.StandardServletMultipartResolver",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/MultipartAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration"
                    ]
                },
                "localeResolver": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.i18n.AcceptHeaderLocaleResolver",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "handlerFunctionAdapter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.function.support.HandlerFunctionAdapter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration$WebEndpointServletConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration$WebEndpointServletConfiguration",
                    "dependencies": []
                },
                "requestMappingHandlerMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "mvcContentNegotiationManager",
                        "mvcConversionService",
                        "mvcResourceUrlProvider"
                    ]
                },
                "webExposeExcludePropertyEndpointFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.expose.IncludeExcludeEndpointFilter",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/WebEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration"
                    ]
                },
                "localSpringDocParameterNameDiscoverer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.core.StandardReflectionParameterNameDiscoverer",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration"
                    ]
                },
                "schemaPropertyDeprecatingConverter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.converters.SchemaPropertyDeprecatingConverter",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration"
                    ]
                },
                "lifecycleProcessor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.context.support.DefaultLifecycleProcessor",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/context/LifecycleAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration",
                        "spring.lifecycle-org.springframework.boot.autoconfigure.context.LifecycleProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration$SameManagementContextConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration$SameManagementContextConfiguration",
                    "dependencies": [
                        "environment"
                    ]
                },
                "requestMappingHandlerAdapter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "mvcContentNegotiationManager",
                        "mvcConversionService",
                        "mvcValidator"
                    ]
                },
                "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonMixinConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonMixinConfiguration",
                    "dependencies": []
                },
                "tomcatMetricsBinder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.metrics.web.tomcat.TomcatMetricsBinder",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/web/tomcat/TomcatMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.web.tomcat.TomcatMetricsAutoConfiguration",
                        "simpleMeterRegistry"
                    ]
                },
                "noteEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "com.springboot.actuator.config.actuator.NoteEndpoint",
                    "resource": "file [C:\\Users\\Royse\\OneDrive\\바탕 화면\\actuator\\build\\classes\\java\\main\\com\\springboot\\actuator\\config\\actuator\\NoteEndpoint.class]",
                    "dependencies": []
                },
                "org.springdoc.core.configuration.SpringDocConfiguration$SpringDocSpringDataWebPropertiesProvider": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.configuration.SpringDocConfiguration$SpringDocSpringDataWebPropertiesProvider",
                    "dependencies": []
                },
                "observationRestClientCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.metrics.web.client.ObservationRestClientCustomizer",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/observation/web/client/RestClientObservationConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.observation.web.client.RestClientObservationConfiguration",
                        "observationRegistry",
                        "management.observations-org.springframework.boot.actuate.autoconfigure.observation.ObservationProperties"
                    ]
                },
                "jvmHeapPressureMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.jvm.JvmHeapPressureMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/JvmMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.JvmMetricsAutoConfiguration"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration",
                    "dependencies": [
                        "server-org.springframework.boot.autoconfigure.web.ServerProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.endpoint.web.ServletEndpointManagementContextConfiguration$WebMvcServletEndpointManagementContextConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.web.ServletEndpointManagementContextConfiguration$WebMvcServletEndpointManagementContextConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.logging.LogFileWebEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.logging.LogFileWebEndpointAutoConfiguration",
                    "dependencies": []
                },
                "springApplicationAdminRegistrar": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrar",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/admin/SpringApplicationAdminJmxAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration",
                        "environment"
                    ]
                },
                "welcomePageNotAcceptableHandlerMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.WelcomePageNotAcceptableHandlerMapping",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e",
                        "mvcConversionService",
                        "mvcResourceUrlProvider"
                    ]
                },
                "spring.info-org.springframework.boot.autoconfigure.info.ProjectInfoProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.info.ProjectInfoProperties",
                    "dependencies": []
                },
                "resourceHandlerRegistrationCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$ResourceChainResourceHandlerRegistrationCustomizer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$ResourceChainCustomizerConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$ResourceChainCustomizerConfiguration",
                        "spring.web-org.springframework.boot.autoconfigure.web.WebProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.MetricsEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.MetricsEndpointAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.observation.web.client.HttpClientObservationsAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.web.client.HttpClientObservationsAutoConfiguration",
                    "dependencies": []
                },
                "endpointOperationParameterMapper": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.invoke.convert.ConversionServiceParameterValueMapper",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/EndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.EndpointAutoConfiguration"
                    ]
                },
                "infoEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.info.InfoEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/info/InfoEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.info.InfoEndpointAutoConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.observation.web.servlet.WebMvcObservationAutoConfiguration$MeterFilterConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.web.servlet.WebMvcObservationAutoConfiguration$MeterFilterConfiguration",
                    "dependencies": []
                },
                "eagerlyInitializeJmxEndpointExporter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.LazyInitializationExcludeFilter$$Lambda$927/0x0000022a814d64d8",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/jmx/JmxEndpointAutoConfiguration.class]",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.health.HealthContributorAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.health.HealthContributorAutoConfiguration",
                    "dependencies": []
                },
                "sslBundleRegistry": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.ssl.DefaultSslBundleRegistry",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/ssl/SslAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.ssl.SslAutoConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.info.InfoEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.info.InfoEndpointAutoConfiguration",
                    "dependencies": []
                },
                "classLoaderMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.jvm.ClassLoaderMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/JvmMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.JvmMetricsAutoConfiguration"
                    ]
                },
                "observationRestTemplateCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.metrics.web.client.ObservationRestTemplateCustomizer",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/observation/web/client/RestTemplateObservationConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.observation.web.client.RestTemplateObservationConfiguration",
                        "observationRegistry",
                        "management.observations-org.springframework.boot.actuate.autoconfigure.observation.ObservationProperties"
                    ]
                },
                "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$ThreadPoolTaskExecutorBuilderConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$ThreadPoolTaskExecutorBuilderConfiguration",
                    "dependencies": []
                },
                "servletWebChildContextFactory": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.ManagementContextFactory",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/web/servlet/ServletManagementContextAutoConfiguration.class]",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.ssl.SslAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.ssl.SslAutoConfiguration",
                    "dependencies": [
                        "spring.ssl-org.springframework.boot.autoconfigure.ssl.SslProperties"
                    ]
                },
                "org.springframework.boot.autoconfigure.internalCachingMetadataReaderFactory": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.core.type.classreading.CachingMetadataReaderFactory",
                    "dependencies": []
                },
                "additionalModelsConverter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.converters.AdditionalModelsConverter",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "objectMapperProvider"
                    ]
                },
                "fileWatcher": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.ssl.FileWatcher",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/ssl/SslAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.ssl.SslAutoConfiguration"
                    ]
                },
                "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$ParameterNamesModuleConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$ParameterNamesModuleConfiguration",
                    "dependencies": []
                },
                "meterRegistryCloser": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.MetricsAutoConfiguration$MeterRegistryCloser",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/MetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.MetricsAutoConfiguration"
                    ]
                },
                "mvcContentNegotiationManager": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.accept.ContentNegotiationManager",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "objectNamingStrategy": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jmx.ParentAwareNamingStrategy",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/jmx/JmxAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$ResourceChainCustomizerConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$ResourceChainCustomizerConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.CompositeMeterRegistryAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.CompositeMeterRegistryAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.audit.AuditEventsEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.audit.AuditEventsEndpointAutoConfiguration",
                    "dependencies": []
                },
                "errorAttributes": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.web.servlet.error.DefaultErrorAttributes",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration"
                    ]
                },
                "httpRequestHandlerAdapter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.mvc.HttpRequestHandlerAdapter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration",
                    "dependencies": []
                },
                "beanNameHandlerMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "mvcConversionService",
                        "mvcResourceUrlProvider"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.observation.web.client.RestTemplateObservationConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.web.client.RestTemplateObservationConfiguration",
                    "dependencies": []
                },
                "spring.servlet.multipart-org.springframework.boot.autoconfigure.web.servlet.MultipartProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.MultipartProperties",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration",
                    "dependencies": [
                        "spring.info-org.springframework.boot.autoconfigure.info.ProjectInfoProperties"
                    ]
                },
                "sslPropertiesSslBundleRegistrar": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.ssl.SslPropertiesBundleRegistrar",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/ssl/SslAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.ssl.SslAutoConfiguration",
                        "fileWatcher"
                    ]
                },
                "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration$SpringDocWebMvcRouterConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration$SpringDocWebMvcRouterConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryConfiguration$EmbeddedTomcat": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryConfiguration$EmbeddedTomcat",
                    "dependencies": []
                },
                "defaultMeterObservationHandler": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.observation.DefaultMeterObservationHandler",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/observation/ObservationAutoConfiguration$MeterObservationHandlerConfiguration$OnlyMetricsMeterObservationHandlerConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration$MeterObservationHandlerConfiguration$OnlyMetricsMeterObservationHandlerConfiguration",
                        "simpleMeterRegistry",
                        "management.observations-org.springframework.boot.actuate.autoconfigure.observation.ObservationProperties"
                    ]
                },
                "springDataWebPropertiesProvider": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.providers.SpringDataWebPropertiesProvider",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration$SpringDocSpringDataWebPropertiesProvider.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration$SpringDocSpringDataWebPropertiesProvider"
                    ]
                },
                "resourceHandlerMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.handler.SimpleUrlHandlerMapping",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "mvcContentNegotiationManager",
                        "mvcConversionService",
                        "mvcResourceUrlProvider"
                    ]
                },
                "simpleControllerHandlerAdapter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "responseBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.service.GenericResponseService",
                    "resource": "class path resource [org/springdoc/webmvc/core/configuration/SpringDocWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration",
                        "operationBuilder",
                        "genericReturnTypeParser",
                        "org.springdoc.core.properties.SpringDocConfigProperties",
                        "propertyResolverUtils"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleMetricsExportAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleMetricsExportAutoConfiguration",
                    "dependencies": []
                },
                "propertyResolverUtils": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.utils.PropertyResolverUtils",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "org.springframework.beans.factory.support.DefaultListableBeanFactory@62e6b5c8",
                        "messageSource",
                        "org.springdoc.core.properties.SpringDocConfigProperties"
                    ]
                },
                "org.springdoc.core.properties.SwaggerUiConfigProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.properties.SwaggerUiConfigProperties",
                    "dependencies": []
                },
                "cachesEndpointWebExtension": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.cache.CachesEndpointWebExtension",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/cache/CachesEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.cache.CachesEndpointAutoConfiguration",
                        "cachesEndpoint"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointAutoConfiguration",
                    "dependencies": []
                },
                "simpleAsyncTaskSchedulerBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.task.SimpleAsyncTaskSchedulerBuilder",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/task/TaskSchedulingConfigurations$SimpleAsyncTaskSchedulerBuilderConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.task.TaskSchedulingConfigurations$SimpleAsyncTaskSchedulerBuilderConfiguration"
                    ]
                },
                "spring.lifecycle-org.springframework.boot.autoconfigure.context.LifecycleProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.context.LifecycleProperties",
                    "dependencies": []
                },
                "management.metrics-org.springframework.boot.actuate.autoconfigure.metrics.MetricsProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.MetricsProperties",
                    "dependencies": []
                },
                "swaggerWelcome": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.ui.SwaggerWelcomeWebMvc",
                    "resource": "class path resource [org/springdoc/webmvc/ui/SwaggerConfig.class]",
                    "dependencies": [
                        "org.springdoc.webmvc.ui.SwaggerConfig",
                        "org.springdoc.core.properties.SwaggerUiConfigProperties",
                        "org.springdoc.core.properties.SpringDocConfigProperties",
                        "org.springdoc.core.properties.SwaggerUiConfigParameters",
                        "springWebProvider"
                    ]
                },
                "flashMapManager": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.support.SessionFlashMapManager",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "healthContributorRegistry": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.health.AutoConfiguredHealthContributorRegistry",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthEndpointConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e",
                        "healthEndpointGroups",
                        "diskSpaceHealthIndicator",
                        "pingHealthContributor"
                    ]
                },
                "org.springframework.boot.autoconfigure.task.TaskSchedulingConfigurations$TaskSchedulerBuilderConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskSchedulingConfigurations$TaskSchedulerBuilderConfiguration",
                    "dependencies": []
                },
                "management.endpoint.configprops-org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.context.properties.ConfigurationPropertiesReportEndpointProperties",
                    "dependencies": []
                },
                "parameterNamesModule": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "com.fasterxml.jackson.module.paramnames.ParameterNamesModule",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/jackson/JacksonAutoConfiguration$ParameterNamesModuleConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$ParameterNamesModuleConfiguration"
                    ]
                },
                "propertiesObservationFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.PropertiesObservationFilterPredicate",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/observation/ObservationAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration",
                        "management.observations-org.springframework.boot.actuate.autoconfigure.observation.ObservationProperties"
                    ]
                },
                "shutdownEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.context.ShutdownEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/context/ShutdownEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.context.ShutdownEndpointAutoConfiguration"
                    ]
                },
                "micrometerClock": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.Clock$1",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/MetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.MetricsAutoConfiguration"
                    ]
                },
                "requestBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.core.service.RequestService",
                    "resource": "class path resource [org/springdoc/webmvc/core/configuration/SpringDocWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration",
                        "parameterBuilder",
                        "requestBodyBuilder",
                        "operationBuilder",
                        "localSpringDocParameterNameDiscoverer"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.beans.BeansEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.beans.BeansEndpointAutoConfiguration",
                    "dependencies": []
                },
                "responseSupportConverter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.converters.ResponseSupportConverter",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "objectMapperProvider"
                    ]
                },
                "propertiesMeterFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.PropertiesMeterFilter",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/MetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.MetricsAutoConfiguration",
                        "management.metrics-org.springframework.boot.actuate.autoconfigure.metrics.MetricsProperties"
                    ]
                },
                "healthHttpCodeStatusMapper": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.health.SimpleHttpCodeStatusMapper",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthEndpointConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointConfiguration",
                        "management.endpoint.health-org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration",
                    "dependencies": []
                },
                "spring.jmx-org.springframework.boot.autoconfigure.jmx.JmxProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jmx.JmxProperties",
                    "dependencies": []
                },
                "uptimeMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.system.UptimeMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/SystemMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.SystemMetricsAutoConfiguration"
                    ]
                },
                "controllerExposeExcludePropertyEndpointFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.expose.IncludeExcludeEndpointFilter",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/WebEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration"
                    ]
                },
                "pathMappedEndpoints": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.web.PathMappedEndpoints",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/WebEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration",
                        "jmxAnnotationEndpointDiscoverer",
                        "servletEndpointDiscoverer",
                        "webEndpointDiscoverer",
                        "controllerEndpointDiscoverer"
                    ]
                },
                "jvmThreadMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.jvm.JvmThreadMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/JvmMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.JvmMetricsAutoConfiguration"
                    ]
                },
                "scheduledTasksEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.scheduling.ScheduledTasksEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/scheduling/ScheduledTasksEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.scheduling.ScheduledTasksEndpointAutoConfiguration"
                    ]
                },
                "diskSpaceMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.metrics.system.DiskSpaceMetricsBinder",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/SystemMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.SystemMetricsAutoConfiguration",
                        "management.metrics-org.springframework.boot.actuate.autoconfigure.metrics.MetricsProperties"
                    ]
                },
                "securityParser": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.service.SecurityService",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "propertyResolverUtils"
                    ]
                },
                "indexPageTransformer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.ui.SwaggerIndexPageTransformer",
                    "resource": "class path resource [org/springdoc/webmvc/ui/SwaggerConfig.class]",
                    "dependencies": [
                        "org.springdoc.webmvc.ui.SwaggerConfig",
                        "org.springdoc.core.properties.SwaggerUiConfigProperties",
                        "org.springdoc.core.properties.SwaggerUiOAuthProperties",
                        "org.springdoc.core.properties.SwaggerUiConfigParameters",
                        "swaggerWelcome",
                        "objectMapperProvider"
                    ]
                },
                "managementServletContext": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.servlet.ServletManagementContextAutoConfiguration$$Lambda$915/0x0000022a814c9880",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/web/servlet/ServletManagementContextAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.web.servlet.ServletManagementContextAutoConfiguration",
                        "management.endpoints.web-org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties"
                    ]
                },
                "fileDescriptorMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.system.FileDescriptorMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/SystemMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.SystemMetricsAutoConfiguration"
                    ]
                },
                "org.springdoc.core.properties.SpringDocConfigProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.properties.SpringDocConfigProperties",
                    "dependencies": []
                },
                "org.springdoc.core.properties.SwaggerUiConfigParameters": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.properties.SwaggerUiConfigParameters",
                    "dependencies": [
                        "org.springdoc.core.properties.SwaggerUiConfigProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration$MeterObservationHandlerConfiguration$OnlyMetricsMeterObservationHandlerConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration$MeterObservationHandlerConfiguration$OnlyMetricsMeterObservationHandlerConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration",
                    "dependencies": [
                        "server-org.springframework.boot.autoconfigure.web.ServerProperties"
                    ]
                },
                "servletEndpointDiscoverer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.web.annotation.ServletEndpointDiscoverer",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/WebEndpointAutoConfiguration$WebEndpointServletConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration$WebEndpointServletConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e"
                    ]
                },
                "environmentEndpointWebExtension": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.env.EnvironmentEndpointWebExtension",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/env/EnvironmentEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointAutoConfiguration",
                        "environmentEndpoint",
                        "management.endpoint.env-org.springframework.boot.actuate.autoconfigure.env.EnvironmentEndpointProperties"
                    ]
                },
                "metricsHttpServerUriTagFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.config.MeterFilter$9",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/observation/web/servlet/WebMvcObservationAutoConfiguration$MeterFilterConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.observation.web.servlet.WebMvcObservationAutoConfiguration$MeterFilterConfiguration",
                        "management.observations-org.springframework.boot.actuate.autoconfigure.observation.ObservationProperties",
                        "management.metrics-org.springframework.boot.actuate.autoconfigure.metrics.MetricsProperties"
                    ]
                },
                "restClientBuilderConfigurer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.client.RestClientBuilderConfigurer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/client/RestClientAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.client.RestClientAutoConfiguration"
                    ]
                },
                "eagerTaskExecutorMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.LazyInitializationExcludeFilter$$Lambda$927/0x0000022a814d64d8",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/task/TaskExecutorMetricsAutoConfiguration.class]",
                    "dependencies": []
                },
                "mvcValidator": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.validation.ValidatorAdapter",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration$TomcatWebSocketConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration$TomcatWebSocketConfiguration",
                    "dependencies": []
                },
                "conditionsReportEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.condition.ConditionsReportEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/condition/ConditionsReportEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.condition.ConditionsReportEndpointAutoConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e"
                    ]
                },
                "applicationAvailability": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.availability.ApplicationAvailabilityBean",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/availability/ApplicationAvailabilityAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration"
                    ]
                },
                "org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration",
                    "dependencies": []
                },
                "org.springdoc.webmvc.ui.SwaggerConfig": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.ui.SwaggerConfig",
                    "dependencies": []
                },
                "mvcResourceUrlProvider": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.resource.ResourceUrlProvider",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "healthEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.health.HealthEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthEndpointConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointConfiguration",
                        "healthContributorRegistry",
                        "healthEndpointGroups",
                        "management.endpoint.health-org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration$SameManagementContextConfiguration$EnableSameManagementContextConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration$SameManagementContextConfiguration$EnableSameManagementContextConfiguration",
                    "dependencies": []
                },
                "spring.task.execution-org.springframework.boot.autoconfigure.task.TaskExecutionProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskExecutionProperties",
                    "dependencies": []
                },
                "viewControllerHandlerMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.HandlerMapping",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "mvcConversionService",
                        "mvcResourceUrlProvider"
                    ]
                },
                "openApiResource": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.api.OpenApiWebMvcResource",
                    "resource": "class path resource [org/springdoc/webmvc/core/configuration/SpringDocWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration",
                        "requestBuilder",
                        "responseBuilder",
                        "operationBuilder",
                        "org.springdoc.core.properties.SpringDocConfigProperties",
                        "springDocProviders"
                    ]
                },
                "themeResolver": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.theme.FixedThemeResolver",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.availability.AvailabilityHealthContributorAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.availability.AvailabilityHealthContributorAutoConfiguration",
                    "dependencies": []
                },
                "mvcPatternParser": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.util.pattern.PathPatternParser",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$SimpleAsyncTaskExecutorBuilderConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$SimpleAsyncTaskExecutorBuilderConfiguration",
                    "dependencies": [
                        "spring.task.execution-org.springframework.boot.autoconfigure.task.TaskExecutionProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointWebExtensionConfiguration$MvcAdditionalHealthEndpointPathsConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointWebExtensionConfiguration$MvcAdditionalHealthEndpointPathsConfiguration",
                    "dependencies": []
                },
                "servletEndpointRegistrar": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.web.ServletEndpointRegistrar",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/ServletEndpointManagementContextConfiguration$WebMvcServletEndpointManagementContextConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.ServletEndpointManagementContextConfiguration$WebMvcServletEndpointManagementContextConfiguration",
                        "management.endpoints.web-org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties",
                        "servletEndpointDiscoverer",
                        "dispatcherServletRegistration"
                    ]
                },
                "org.springframework.boot.sql.init.dependency.DatabaseInitializationDependencyConfigurer$DependsOnDatabaseInitializationPostProcessor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.sql.init.dependency.DatabaseInitializationDependencyConfigurer$DependsOnDatabaseInitializationPostProcessor",
                    "dependencies": []
                },
                "dispatcherServlet": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.DispatcherServlet",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/DispatcherServletAutoConfiguration$DispatcherServletConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration$DispatcherServletConfiguration",
                        "spring.mvc-org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.task.TaskExecutorMetricsAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.task.TaskExecutorMetricsAutoConfiguration",
                    "dependencies": [
                        "applicationTaskExecutor",
                        "simpleMeterRegistry"
                    ]
                },
                "restClientBuilder": {
                    "aliases": [],
                    "scope": "prototype",
                    "type": "org.springframework.web.client.RestClient$Builder",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/client/RestClientAutoConfiguration.class]",
                    "dependencies": []
                },
                "springWebProvider": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.core.providers.SpringWebMvcProvider",
                    "resource": "class path resource [org/springdoc/webmvc/core/configuration/SpringDocWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration"
                    ]
                },
                "jvmInfoMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.jvm.JvmInfoMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/JvmMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.JvmMetricsAutoConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.endpoint.web.ServletEndpointManagementContextConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.web.ServletEndpointManagementContextConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration",
                    "dependencies": []
                },
                "parameterBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.service.GenericParameterService",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "propertyResolverUtils",
                        "objectMapperProvider"
                    ]
                },
                "webEndpointServletHandlerMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/servlet/WebMvcEndpointManagementContextConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.servlet.WebMvcEndpointManagementContextConfiguration",
                        "webEndpointDiscoverer",
                        "servletEndpointDiscoverer",
                        "controllerEndpointDiscoverer",
                        "endpointMediaTypes",
                        "management.endpoints.web.cors-org.springframework.boot.actuate.autoconfigure.endpoint.web.CorsEndpointProperties",
                        "management.endpoints.web-org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties",
                        "environment"
                    ]
                },
                "threadPoolTaskExecutorBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.task.ThreadPoolTaskExecutorBuilder",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/task/TaskExecutorConfigurations$ThreadPoolTaskExecutorBuilderConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$ThreadPoolTaskExecutorBuilderConfiguration",
                        "spring.task.execution-org.springframework.boot.autoconfigure.task.TaskExecutionProperties"
                    ]
                },
                "processorMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.system.ProcessorMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/SystemMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.SystemMetricsAutoConfiguration"
                    ]
                },
                "org.springframework.boot.autoconfigure.task.TaskSchedulingConfigurations$SimpleAsyncTaskSchedulerBuilderConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskSchedulingConfigurations$SimpleAsyncTaskSchedulerBuilderConfiguration",
                    "dependencies": [
                        "spring.task.scheduling-org.springframework.boot.autoconfigure.task.TaskSchedulingProperties"
                    ]
                },
                "dispatcherServletMappingDescriptionProvider": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.web.mappings.servlet.DispatcherServletsMappingDescriptionProvider",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/web/mappings/MappingsEndpointAutoConfiguration$ServletWebConfiguration$SpringMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration$ServletWebConfiguration$SpringMvcConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.scheduling.ScheduledTasksEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.scheduling.ScheduledTasksEndpointAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$TaskExecutorBuilderConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$TaskExecutorBuilderConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.integration.IntegrationMetricsAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.integration.IntegrationMetricsAutoConfiguration",
                    "dependencies": []
                },
                "customInfoContributor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "com.springboot.actuator.config.actuator.CustomInfoContributor",
                    "resource": "file [C:\\Users\\Royse\\OneDrive\\바탕 화면\\actuator\\build\\classes\\java\\main\\com\\springboot\\actuator\\config\\actuator\\CustomInfoContributor.class]",
                    "dependencies": []
                },
                "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration$MeterObservationHandlerConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration$MeterObservationHandlerConfiguration",
                    "dependencies": []
                },
                "management.endpoint.sbom-org.springframework.boot.actuate.sbom.SbomProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.sbom.SbomProperties",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$WebMvcAutoConfigurationAdapter",
                    "dependencies": [
                        "spring.web-org.springframework.boot.autoconfigure.web.WebProperties",
                        "spring.mvc-org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties",
                        "org.springframework.beans.factory.support.DefaultListableBeanFactory@62e6b5c8"
                    ]
                },
                "springDocProviders": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.providers.SpringDocProviders",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "objectMapperProvider"
                    ]
                },
                "errorPageRegistrarBeanPostProcessor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.web.server.ErrorPageRegistrarBeanPostProcessor",
                    "dependencies": []
                },
                "errorPageCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$ErrorPageCustomizer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration",
                        "dispatcherServletRegistration"
                    ]
                },
                "mvcConversionService": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.format.WebConversionService",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "loggersEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.logging.LoggersEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/logging/LoggersEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.logging.LoggersEndpointAutoConfiguration",
                        "springBootLoggingSystem"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$DefaultErrorViewResolverConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$DefaultErrorViewResolverConfiguration",
                    "dependencies": [
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e",
                        "spring.web-org.springframework.boot.autoconfigure.web.WebProperties"
                    ]
                },
                "jmxIncludeExcludePropertyEndpointFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.expose.IncludeExcludeEndpointFilter",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/jmx/JmxEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.jmx.JmxEndpointAutoConfiguration"
                    ]
                },
                "controllerEndpointDiscoverer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.web.annotation.ControllerEndpointDiscoverer",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/WebEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointAutoConfiguration"
                    ]
                },
                "diskSpaceHealthIndicator": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.system.DiskSpaceHealthIndicator",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/system/DiskSpaceHealthContributorAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.system.DiskSpaceHealthContributorAutoConfiguration",
                        "management.health.diskspace-org.springframework.boot.actuate.autoconfigure.system.DiskSpaceHealthIndicatorProperties"
                    ]
                },
                "tomcatWebServerFactoryCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.embedded.TomcatWebServerFactoryCustomizer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/embedded/EmbeddedWebServerFactoryCustomizerAutoConfiguration$TomcatWebServerFactoryCustomizerConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration$TomcatWebServerFactoryCustomizerConfiguration",
                        "environment",
                        "server-org.springframework.boot.autoconfigure.web.ServerProperties"
                    ]
                },
                "jvmMemoryMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.jvm.JvmMemoryMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/JvmMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.JvmMetricsAutoConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration$OnlyMetricsConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration$OnlyMetricsConfiguration",
                    "dependencies": []
                },
                "modelConverterRegistrar": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.converters.ModelConverterRegistrar",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.endpoint.EndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.EndpointAutoConfiguration",
                    "dependencies": []
                },
                "restClientSsl": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.client.AutoConfiguredRestClientSsl",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/client/RestClientAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.client.RestClientAutoConfiguration",
                        "sslBundleRegistry"
                    ]
                },
                "threadPoolTaskSchedulerBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.task.ThreadPoolTaskSchedulerBuilder",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/task/TaskSchedulingConfigurations$ThreadPoolTaskSchedulerBuilderConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.task.TaskSchedulingConfigurations$ThreadPoolTaskSchedulerBuilderConfiguration",
                        "spring.task.scheduling-org.springframework.boot.autoconfigure.task.TaskSchedulingProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.condition.ConditionsReportEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.condition.ConditionsReportEndpointAutoConfiguration",
                    "dependencies": []
                },
                "viewNameTranslator": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.view.DefaultRequestToViewNameTranslator",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "org.springframework.boot.autoconfigure.http.JacksonHttpMessageConvertersConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.http.JacksonHttpMessageConvertersConfiguration",
                    "dependencies": []
                },
                "polymorphicModelConverter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.converters.PolymorphicModelConverter",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "objectMapperProvider"
                    ]
                },
                "mvcPathMatcher": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.util.AntPathMatcher",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration"
                    ]
                },
                "handlerExceptionResolver": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.handler.HandlerExceptionResolverComposite",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "mvcContentNegotiationManager"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.system.DiskSpaceHealthContributorAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.system.DiskSpaceHealthContributorAutoConfiguration",
                    "dependencies": []
                },
                "fileSupportConverter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.converters.FileSupportConverter",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "objectMapperProvider"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.metrics.MetricsAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.MetricsAutoConfiguration",
                    "dependencies": []
                },
                "routerFunctionProvider": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.core.providers.RouterFunctionWebMvcProvider",
                    "resource": "class path resource [org/springdoc/webmvc/core/configuration/SpringDocWebMvcConfiguration$SpringDocWebMvcRouterConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.webmvc.core.configuration.SpringDocWebMvcConfiguration$SpringDocWebMvcRouterConfiguration"
                    ]
                },
                "basicErrorController": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration",
                        "errorAttributes"
                    ]
                },
                "cachesEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.cache.CachesEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/cache/CachesEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.cache.CachesEndpointAutoConfiguration"
                    ]
                },
                "pingHealthContributor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.health.PingHealthIndicator",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthContributorAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.health.HealthContributorAutoConfiguration"
                    ]
                },
                "dispatcherServletRegistration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletRegistrationBean",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/DispatcherServletAutoConfiguration$DispatcherServletRegistrationConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration$DispatcherServletRegistrationConfiguration",
                        "dispatcherServlet",
                        "spring.mvc-org.springframework.boot.autoconfigure.web.servlet.WebMvcProperties"
                    ]
                },
                "mappingsEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.web.mappings.MappingsEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/web/mappings/MappingsEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e"
                    ]
                },
                "healthEndpointGroupMembershipValidator": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointConfiguration$HealthEndpointGroupMembershipValidator",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthEndpointConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointConfiguration",
                        "management.endpoint.health-org.springframework.boot.actuate.autoconfigure.health.HealthEndpointProperties",
                        "healthContributorRegistry"
                    ]
                },
                "healthEndpointGroupsBeanPostProcessor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointConfiguration$HealthEndpointGroupsBeanPostProcessor",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthEndpointConfiguration.class]",
                    "dependencies": []
                },
                "management.endpoints.web-org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$TaskExecutorConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$TaskExecutorConfiguration",
                    "dependencies": []
                },
                "tomcatServletWebServerFactory": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/ServletWebServerFactoryConfiguration$EmbeddedTomcat.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryConfiguration$EmbeddedTomcat"
                    ]
                },
                "org.springframework.boot.autoconfigure.http.JacksonHttpMessageConvertersConfiguration$MappingJackson2HttpMessageConverterConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.http.JacksonHttpMessageConvertersConfiguration$MappingJackson2HttpMessageConverterConfiguration",
                    "dependencies": []
                },
                "spring.jackson-org.springframework.boot.autoconfigure.jackson.JacksonProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jackson.JacksonProperties",
                    "dependencies": []
                },
                "error": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$StaticView",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration"
                    ]
                },
                "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonObjectMapperConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonObjectMapperConfiguration",
                    "dependencies": []
                },
                "swaggerConfigResource": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.webmvc.ui.SwaggerConfigResource",
                    "resource": "class path resource [org/springdoc/webmvc/ui/SwaggerConfig.class]",
                    "dependencies": [
                        "org.springdoc.webmvc.ui.SwaggerConfig",
                        "swaggerWelcome"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointAutoConfiguration",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$WhitelabelErrorViewConfiguration",
                    "dependencies": []
                },
                "simpleAsyncTaskExecutorBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.task.SimpleAsyncTaskExecutorBuilder",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/task/TaskExecutorConfigurations$SimpleAsyncTaskExecutorBuilderConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$SimpleAsyncTaskExecutorBuilderConfiguration"
                    ]
                },
                "jsonMixinModule": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.jackson.JsonMixinModule",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/jackson/JacksonAutoConfiguration$JacksonMixinConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonMixinConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e",
                        "jsonMixinModuleEntries"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration$DispatcherServletConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration$DispatcherServletConfiguration",
                    "dependencies": []
                },
                "sbomEndpoint": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.sbom.SbomEndpoint",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/sbom/SbomEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.sbom.SbomEndpointAutoConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e"
                    ]
                },
                "spring.web-org.springframework.boot.autoconfigure.web.WebProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.WebProperties",
                    "dependencies": []
                },
                "org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration",
                    "dependencies": [
                        "spring.jmx-org.springframework.boot.autoconfigure.jmx.JmxProperties"
                    ]
                },
                "genericReturnTypeParser": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.configuration.SpringDocConfiguration$1",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration"
                    ]
                },
                "jvmGcMetrics": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.core.instrument.binder.jvm.JvmGcMetrics",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/JvmMetricsAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.JvmMetricsAutoConfiguration"
                    ]
                },
                "org.springframework.boot.autoconfigure.web.client.RestClientAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.client.RestClientAutoConfiguration",
                    "dependencies": []
                },
                "healthEndpointWebMvcHandlerMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.endpoint.web.servlet.AdditionalHealthEndpointPathsWebMvcHandlerMapping",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/health/HealthEndpointWebExtensionConfiguration$MvcAdditionalHealthEndpointPathsConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.health.HealthEndpointWebExtensionConfiguration$MvcAdditionalHealthEndpointPathsConfiguration",
                        "webEndpointDiscoverer",
                        "healthEndpointGroups"
                    ]
                },
                "mvcViewResolver": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.servlet.view.ViewResolverComposite",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "mvcContentNegotiationManager"
                    ]
                },
                "simpleConfig": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimplePropertiesConfigAdapter",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/metrics/export/simple/SimpleMetricsExportAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleMetricsExportAutoConfiguration",
                        "management.simple.metrics.export-org.springframework.boot.actuate.autoconfigure.metrics.export.simple.SimpleProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.observation.web.client.RestClientObservationConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.observation.web.client.RestClientObservationConfiguration",
                    "dependencies": []
                },
                "welcomePageHandlerMapping": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.servlet.WelcomePageHandlerMapping",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext@1a20270e",
                        "mvcConversionService",
                        "mvcResourceUrlProvider"
                    ]
                },
                "servletExposeExcludePropertyEndpointFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.expose.IncludeExcludeEndpointFilter",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/web/ServletEndpointManagementContextConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.web.ServletEndpointManagementContextConfiguration",
                        "management.endpoints.web-org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.sbom.SbomEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.sbom.SbomEndpointAutoConfiguration",
                    "dependencies": [
                        "management.endpoint.sbom-org.springframework.boot.actuate.sbom.SbomProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.web.servlet.ServletManagementContextAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.servlet.ServletManagementContextAutoConfiguration",
                    "dependencies": []
                },
                "observationRegistry": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "io.micrometer.observation.SimpleObservationRegistry",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/observation/ObservationAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.observation.ObservationAutoConfiguration"
                    ]
                },
                "mvcUriComponentsContributor": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.web.method.support.CompositeUriComponentsContributor",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration$EnableWebMvcConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration$EnableWebMvcConfiguration",
                        "mvcConversionService",
                        "requestMappingHandlerAdapter"
                    ]
                },
                "filterMappingDescriptionProvider": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.web.mappings.servlet.FiltersMappingDescriptionProvider",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/web/mappings/MappingsEndpointAutoConfiguration$ServletWebConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.web.mappings.MappingsEndpointAutoConfiguration$ServletWebConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration",
                    "dependencies": []
                },
                "endpointObjectMapper": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.jackson.JacksonEndpointAutoConfiguration$$Lambda$769/0x0000022a814470a8",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/endpoint/jackson/JacksonEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.endpoint.jackson.JacksonEndpointAutoConfiguration"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.web.exchanges.HttpExchangesEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.web.exchanges.HttpExchangesEndpointAutoConfiguration",
                    "dependencies": []
                },
                "jacksonObjectMapper": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "com.fasterxml.jackson.databind.ObjectMapper",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/jackson/JacksonAutoConfiguration$JacksonObjectMapperConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration$JacksonObjectMapperConfiguration",
                        "jacksonObjectMapperBuilder"
                    ]
                },
                "webMvcObservationFilter": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.web.servlet.FilterRegistrationBean",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/observation/web/servlet/WebMvcObservationAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.observation.web.servlet.WebMvcObservationAutoConfiguration",
                        "observationRegistry",
                        "management.observations-org.springframework.boot.actuate.autoconfigure.observation.ObservationProperties"
                    ]
                },
                "sbomEndpointWebExtension": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.sbom.SbomEndpointWebExtension",
                    "resource": "class path resource [org/springframework/boot/actuate/autoconfigure/sbom/SbomEndpointAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.actuate.autoconfigure.sbom.SbomEndpointAutoConfiguration",
                        "sbomEndpoint"
                    ]
                },
                "org.springframework.boot.autoconfigure.aop.AopAutoConfiguration$ClassProxyingConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.aop.AopAutoConfiguration$ClassProxyingConfiguration",
                    "dependencies": []
                },
                "org.springdoc.core.properties.SwaggerUiOAuthProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.properties.SwaggerUiOAuthProperties",
                    "dependencies": []
                },
                "httpMessageConvertersRestClientCustomizer": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.autoconfigure.web.client.HttpMessageConvertersRestClientCustomizer",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/web/client/RestClientAutoConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.web.client.RestClientAutoConfiguration"
                    ]
                },
                "management.endpoint.logfile-org.springframework.boot.actuate.autoconfigure.logging.LogFileWebEndpointProperties": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.logging.LogFileWebEndpointProperties",
                    "dependencies": []
                },
                "taskExecutorBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.task.TaskExecutorBuilder",
                    "resource": "class path resource [org/springframework/boot/autoconfigure/task/TaskExecutorConfigurations$TaskExecutorBuilderConfiguration.class]",
                    "dependencies": [
                        "org.springframework.boot.autoconfigure.task.TaskExecutorConfigurations$TaskExecutorBuilderConfiguration",
                        "spring.task.execution-org.springframework.boot.autoconfigure.task.TaskExecutionProperties"
                    ]
                },
                "org.springframework.boot.actuate.autoconfigure.endpoint.jackson.JacksonEndpointAutoConfiguration": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springframework.boot.actuate.autoconfigure.endpoint.jackson.JacksonEndpointAutoConfiguration",
                    "dependencies": []
                },
                "requestBodyBuilder": {
                    "aliases": [],
                    "scope": "singleton",
                    "type": "org.springdoc.core.service.RequestBodyService",
                    "resource": "class path resource [org/springdoc/core/configuration/SpringDocConfiguration.class]",
                    "dependencies": [
                        "org.springdoc.core.configuration.SpringDocConfiguration",
                        "parameterBuilder"
                    ]
                }
            }
        }
    }
}
```

## 스프링 부트 자동 설정 내역 확인 `/conditions`

스프링 부트의 자동 설정 조건 내역을 확인할 수 있다.

접속 경로:

```
http://localhost:8080/act/conditions
```

```json
{
    "contexts": {
        "actuator": {
            "positiveMatches": {
                "SpringDocConfiguration": [
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "@ConditionalOnWebApplication (required) found 'session' scope"
                    },
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.api-docs.enabled) matched"
                    }
                ],
                "SpringDocConfiguration#fileSupportConverter": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.converters.FileSupportConverter; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration#objectMapperProvider": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.providers.ObjectMapperProvider; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration#openAPIBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.service.OpenAPIService; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration#operationBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.service.OperationService; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration#parameterBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.service.GenericParameterService; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration#polymorphicModelConverter": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.model-converters.polymorphic-converter.enabled) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.converters.PolymorphicModelConverter; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration#requestBodyBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.service.RequestBodyService; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration#responseSupportConverter": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.converters.ResponseSupportConverter; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration#schemaPropertyDeprecatingConverter": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.model-converters.deprecating-converter.enabled) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.converters.SchemaPropertyDeprecatingConverter; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration#securityParser": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.service.SecurityService; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration#springDocProviders": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.providers.SpringDocProviders; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration.SpringDocSpringDataWebPropertiesProvider": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.boot.autoconfigure.data.web.SpringDataWebProperties'"
                    }
                ],
                "SpringDocConfiguration.SpringDocSpringDataWebPropertiesProvider#springDataWebPropertiesProvider": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.providers.SpringDataWebPropertiesProvider; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocConfiguration.WebConversionServiceConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.boot.autoconfigure.web.format.WebConversionService'"
                    }
                ],
                "SpringDocConfigProperties": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.api-docs.enabled) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springdoc.core.configuration.SpringDocConfiguration; SearchStrategy: all) found bean 'org.springdoc.core.configuration.SpringDocConfiguration'"
                    }
                ],
                "SwaggerUiConfigParameters": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.swagger-ui.enabled) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springdoc.core.configuration.SpringDocConfiguration; SearchStrategy: all) found bean 'org.springdoc.core.configuration.SpringDocConfiguration'"
                    }
                ],
                "SwaggerUiConfigProperties": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.swagger-ui.enabled) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springdoc.core.configuration.SpringDocConfiguration; SearchStrategy: all) found bean 'org.springdoc.core.configuration.SpringDocConfiguration'"
                    }
                ],
                "SwaggerUiOAuthProperties": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.swagger-ui.enabled) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springdoc.core.configuration.SpringDocConfiguration; SearchStrategy: all) found bean 'org.springdoc.core.configuration.SpringDocConfiguration'"
                    }
                ],
                "SpringDocWebMvcConfiguration": [
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    },
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.api-docs.enabled) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springdoc.core.configuration.SpringDocConfiguration; SearchStrategy: all) found bean 'org.springdoc.core.configuration.SpringDocConfiguration'"
                    }
                ],
                "SpringDocWebMvcConfiguration#openApiResource": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.use-management-port=false) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.webmvc.api.OpenApiWebMvcResource; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocWebMvcConfiguration#requestBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.webmvc.core.service.RequestService; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocWebMvcConfiguration#responseBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.service.GenericResponseService; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocWebMvcConfiguration#springWebProvider": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.core.providers.SpringWebProvider; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SpringDocWebMvcConfiguration.SpringDocWebMvcActuatorConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping'"
                    }
                ],
                "SpringDocWebMvcConfiguration.SpringDocWebMvcRouterConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.web.servlet.function.RouterFunction'"
                    }
                ],
                "SpringDocWebMvcConfiguration.SpringDocWebMvcRouterConfiguration#routerFunctionProvider": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.webmvc.core.providers.RouterFunctionWebMvcProvider; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SwaggerConfig": [
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    },
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.swagger-ui.enabled) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springdoc.core.configuration.SpringDocConfiguration; SearchStrategy: all) found bean 'org.springdoc.core.configuration.SpringDocConfiguration'"
                    }
                ],
                "SwaggerConfig#indexPageTransformer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.webmvc.ui.SwaggerIndexTransformer; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SwaggerConfig#swaggerConfigResource": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.use-management-port=false) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.webmvc.ui.SwaggerConfigResource; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SwaggerConfig#swaggerWebMvcConfigurer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.webmvc.ui.SwaggerWebMvcConfigurer; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SwaggerConfig#swaggerWelcome": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (springdoc.use-management-port=false) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springdoc.webmvc.ui.SwaggerWelcomeWebMvc; SearchStrategy: all) did not find any beans"
                    }
                ],
                "AuditEventsEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "BeansEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "BeansEndpointAutoConfiguration#beansEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.beans.BeansEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "CachesEndpointAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.cache.CacheManager'"
                    },
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "CachesEndpointAutoConfiguration#cachesEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.cache.CachesEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "CachesEndpointAutoConfiguration#cachesEndpointWebExtension": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.cache.CachesEndpoint; SearchStrategy: all) found bean 'cachesEndpoint'; @ConditionalOnMissingBean (types: org.springframework.boot.actuate.cache.CachesEndpointWebExtension; SearchStrategy: all) did not find any beans"
                    },
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "ConditionsReportEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "ConditionsReportEndpointAutoConfiguration#conditionsReportEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.autoconfigure.condition.ConditionsReportEndpoint; SearchStrategy: current) did not find any beans"
                    }
                ],
                "ShutdownEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "ShutdownEndpointAutoConfiguration#shutdownEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.context.ShutdownEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ConfigurationPropertiesReportEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "ConfigurationPropertiesReportEndpointAutoConfiguration#configurationPropertiesReportEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.context.properties.ConfigurationPropertiesReportEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ConfigurationPropertiesReportEndpointAutoConfiguration#configurationPropertiesReportEndpointWebExtension": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.context.properties.ConfigurationPropertiesReportEndpoint; SearchStrategy: all) found bean 'configurationPropertiesReportEndpoint'; @ConditionalOnMissingBean (types: org.springframework.boot.actuate.context.properties.ConfigurationPropertiesReportEndpointWebExtension; SearchStrategy: all) did not find any beans"
                    },
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "EndpointAutoConfiguration#endpointCachingOperationInvokerAdvisor": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.invoker.cache.CachingOperationInvokerAdvisor; SearchStrategy: all) did not find any beans"
                    }
                ],
                "EndpointAutoConfiguration#endpointOperationParameterMapper": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.invoke.ParameterValueMapper; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JacksonEndpointAutoConfiguration#endpointObjectMapper": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'com.fasterxml.jackson.databind.ObjectMapper', 'org.springframework.http.converter.json.Jackson2ObjectMapperBuilder'"
                    },
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (management.endpoints.jackson.isolated-object-mapper) matched"
                    }
                ],
                "JmxEndpointAutoConfiguration": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (spring.jmx.enabled=true) matched"
                    }
                ],
                "JmxEndpointAutoConfiguration#endpointObjectNameFactory": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.jmx.EndpointObjectNameFactory; SearchStrategy: current) did not find any beans"
                    }
                ],
                "JmxEndpointAutoConfiguration#jmxAnnotationEndpointDiscoverer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.jmx.JmxEndpointsSupplier; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JmxEndpointAutoConfiguration#jmxMBeanExporter": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnSingleCandidate (types: javax.management.MBeanServer; SearchStrategy: all) found a single bean 'mbeanServer'"
                    }
                ],
                "ServletEndpointManagementContextConfiguration": [
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    }
                ],
                "ServletEndpointManagementContextConfiguration.WebMvcServletEndpointManagementContextConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'"
                    }
                ],
                "WebEndpointAutoConfiguration": [
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "@ConditionalOnWebApplication (required) found 'session' scope"
                    }
                ],
                "WebEndpointAutoConfiguration#controllerEndpointDiscoverer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.web.annotation.ControllerEndpointsSupplier; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebEndpointAutoConfiguration#endpointMediaTypes": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.web.EndpointMediaTypes; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebEndpointAutoConfiguration#pathMappedEndpoints": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.web.PathMappedEndpoints; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebEndpointAutoConfiguration#webEndpointDiscoverer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.web.WebEndpointsSupplier; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebEndpointAutoConfiguration.WebEndpointServletConfiguration": [
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    }
                ],
                "WebEndpointAutoConfiguration.WebEndpointServletConfiguration#servletEndpointDiscoverer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.web.annotation.ServletEndpointsSupplier; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebMvcEndpointManagementContextConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.web.servlet.DispatcherServlet,org.springframework.boot.actuate.endpoint.web.WebEndpointsSupplier; SearchStrategy: all) found beans 'webEndpointDiscoverer', 'dispatcherServlet'"
                    }
                ],
                "WebMvcEndpointManagementContextConfiguration#controllerEndpointHandlerMapping": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.web.servlet.ControllerEndpointHandlerMapping; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebMvcEndpointManagementContextConfiguration#endpointObjectMapperWebMvcConfigurer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.endpoint.jackson.EndpointObjectMapper; SearchStrategy: all) found bean 'endpointObjectMapper'"
                    }
                ],
                "WebMvcEndpointManagementContextConfiguration#webEndpointServletHandlerMapping": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping; SearchStrategy: all) did not find any beans"
                    }
                ],
                "EnvironmentEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "EnvironmentEndpointAutoConfiguration#environmentEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.env.EnvironmentEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "EnvironmentEndpointAutoConfiguration#environmentEndpointWebExtension": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.env.EnvironmentEndpoint; SearchStrategy: all) found bean 'environmentEndpoint'; @ConditionalOnMissingBean (types: org.springframework.boot.actuate.env.EnvironmentEndpointWebExtension; SearchStrategy: all) did not find any beans"
                    },
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "HealthContributorAutoConfiguration#pingHealthContributor": [
                    {
                        "condition": "OnEnabledHealthIndicatorCondition",
                        "message": "@ConditionalOnEnabledHealthIndicator management.health.defaults.enabled is considered true"
                    }
                ],
                "HealthEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "HealthEndpointConfiguration#healthContributorRegistry": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.health.HealthContributorRegistry; SearchStrategy: all) did not find any beans"
                    }
                ],
                "HealthEndpointConfiguration#healthEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.health.HealthEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "HealthEndpointConfiguration#healthEndpointGroupMembershipValidator": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (management.endpoint.health.validate-group-membership=true) matched"
                    }
                ],
                "HealthEndpointConfiguration#healthEndpointGroups": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.health.HealthEndpointGroups; SearchStrategy: all) did not find any beans"
                    }
                ],
                "HealthEndpointConfiguration#healthHttpCodeStatusMapper": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.health.HttpCodeStatusMapper; SearchStrategy: all) did not find any beans"
                    }
                ],
                "HealthEndpointConfiguration#healthStatusAggregator": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.health.StatusAggregator; SearchStrategy: all) did not find any beans"
                    }
                ],
                "HealthEndpointWebExtensionConfiguration": [
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    },
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.health.HealthEndpoint; SearchStrategy: all) found bean 'healthEndpoint'"
                    }
                ],
                "HealthEndpointWebExtensionConfiguration#healthEndpointWebExtension": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.health.HealthEndpointWebExtension; SearchStrategy: all) did not find any beans"
                    }
                ],
                "HealthEndpointWebExtensionConfiguration.MvcAdditionalHealthEndpointPathsConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.web.servlet.DispatcherServlet; SearchStrategy: all) found bean 'dispatcherServlet'"
                    }
                ],
                "InfoEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "InfoEndpointAutoConfiguration#infoEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.info.InfoEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "LogFileWebEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "LoggersEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "LoggersEndpointAutoConfiguration#loggersEndpoint": [
                    {
                        "condition": "LoggersEndpointAutoConfiguration.OnEnabledLoggingSystemCondition",
                        "message": "Logging System enabled"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.boot.logging.LoggingSystem; SearchStrategy: all) found bean 'springBootLoggingSystem'; @ConditionalOnMissingBean (types: org.springframework.boot.actuate.logging.LoggersEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "CompositeMeterRegistryAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.instrument.composite.CompositeMeterRegistry'"
                    }
                ],
                "JvmMetricsAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.instrument.MeterRegistry'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found bean 'simpleMeterRegistry'"
                    }
                ],
                "JvmMetricsAutoConfiguration#classLoaderMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.jvm.ClassLoaderMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JvmMetricsAutoConfiguration#jvmCompilationMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.jvm.JvmCompilationMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JvmMetricsAutoConfiguration#jvmGcMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.jvm.JvmGcMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JvmMetricsAutoConfiguration#jvmHeapPressureMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.jvm.JvmHeapPressureMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JvmMetricsAutoConfiguration#jvmInfoMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.jvm.JvmInfoMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JvmMetricsAutoConfiguration#jvmMemoryMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.jvm.JvmMemoryMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JvmMetricsAutoConfiguration#jvmThreadMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.jvm.JvmThreadMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "LogbackMetricsAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'io.micrometer.core.instrument.MeterRegistry', 'ch.qos.logback.classic.LoggerContext', 'org.slf4j.LoggerFactory'"
                    },
                    {
                        "condition": "LogbackMetricsAutoConfiguration.LogbackLoggingCondition",
                        "message": "LogbackLoggingCondition ILoggerFactory is a Logback LoggerContext"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found bean 'simpleMeterRegistry'"
                    }
                ],
                "LogbackMetricsAutoConfiguration#logbackMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.logging.LogbackMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "MetricsAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.annotation.Timed'"
                    }
                ],
                "MetricsAutoConfiguration#micrometerClock": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.Clock; SearchStrategy: all) did not find any beans"
                    }
                ],
                "MetricsEndpointAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.annotation.Timed'"
                    },
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "MetricsEndpointAutoConfiguration#metricsEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found bean 'simpleMeterRegistry'; @ConditionalOnMissingBean (types: org.springframework.boot.actuate.metrics.MetricsEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SystemMetricsAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.instrument.MeterRegistry'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found bean 'simpleMeterRegistry'"
                    }
                ],
                "SystemMetricsAutoConfiguration#diskSpaceMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.metrics.system.DiskSpaceMetricsBinder; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SystemMetricsAutoConfiguration#fileDescriptorMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.system.FileDescriptorMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SystemMetricsAutoConfiguration#processorMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.system.ProcessorMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SystemMetricsAutoConfiguration#uptimeMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.system.UptimeMetrics; SearchStrategy: all) did not find any beans"
                    }
                ],
                "CacheMeterBinderProvidersConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.instrument.binder.MeterBinder'"
                    }
                ],
                "SimpleMetricsExportAutoConfiguration": [
                    {
                        "condition": "OnMetricsExportEnabledCondition",
                        "message": "@ConditionalOnEnabledMetricsExport management.defaults.metrics.export.enabled is considered true"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.core.instrument.Clock; SearchStrategy: all) found bean 'micrometerClock'; @ConditionalOnMissingBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SimpleMetricsExportAutoConfiguration#simpleConfig": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.simple.SimpleConfig; SearchStrategy: all) did not find any beans"
                    }
                ],
                "StartupTimeMetricsListenerAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.instrument.MeterRegistry'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found bean 'simpleMeterRegistry'"
                    }
                ],
                "StartupTimeMetricsListenerAutoConfiguration#startupTimeMetrics": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.metrics.startup.StartupTimeMetricsListener; SearchStrategy: all) did not find any beans"
                    }
                ],
                "TaskExecutorMetricsAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.instrument.binder.jvm.ExecutorServiceMetrics'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: java.util.concurrent.Executor,io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found beans 'applicationTaskExecutor', 'simpleMeterRegistry'"
                    }
                ],
                "TomcatMetricsAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'io.micrometer.core.instrument.binder.tomcat.TomcatMetrics', 'org.apache.catalina.Manager'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "@ConditionalOnWebApplication (required) found 'session' scope"
                    }
                ],
                "TomcatMetricsAutoConfiguration#tomcatMetricsBinder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found bean 'simpleMeterRegistry'; @ConditionalOnMissingBean (types: io.micrometer.core.instrument.binder.tomcat.TomcatMetrics,org.springframework.boot.actuate.metrics.web.tomcat.TomcatMetricsBinder; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ObservationAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.observation.ObservationRegistry'"
                    }
                ],
                "ObservationAutoConfiguration#observationRegistry": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.observation.ObservationRegistry; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ObservationAutoConfiguration.MeterObservationHandlerConfiguration": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found bean 'simpleMeterRegistry'; @ConditionalOnMissingBean (types: io.micrometer.core.instrument.observation.MeterObservationHandler; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ObservationAutoConfiguration.MeterObservationHandlerConfiguration.OnlyMetricsMeterObservationHandlerConfiguration": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: io.micrometer.tracing.Tracer; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ObservationAutoConfiguration.OnlyMetricsConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.instrument.MeterRegistry'; @ConditionalOnMissingClass did not find unwanted class 'io.micrometer.tracing.Tracer'"
                    }
                ],
                "HttpClientObservationsAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.observation.Observation'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.observation.ObservationRegistry; SearchStrategy: all) found bean 'observationRegistry'"
                    }
                ],
                "HttpClientObservationsAutoConfiguration.MeterFilterConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.instrument.MeterRegistry'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found bean 'simpleMeterRegistry'"
                    }
                ],
                "RestClientObservationConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.web.client.RestClient'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.web.client.RestClient$Builder; SearchStrategy: all) found bean 'restClientBuilder'"
                    }
                ],
                "RestTemplateObservationConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.web.client.RestTemplate'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.boot.web.client.RestTemplateBuilder; SearchStrategy: all) found bean 'restTemplateBuilder'"
                    }
                ],
                "WebMvcObservationAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'org.springframework.web.servlet.DispatcherServlet', 'io.micrometer.observation.Observation'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.observation.ObservationRegistry; SearchStrategy: all) found bean 'observationRegistry'"
                    }
                ],
                "WebMvcObservationAutoConfiguration#webMvcObservationFilter": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.web.filter.ServerHttpObservationFilter; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebMvcObservationAutoConfiguration.MeterFilterConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'io.micrometer.core.instrument.MeterRegistry'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found bean 'simpleMeterRegistry'"
                    }
                ],
                "SbomEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "SbomEndpointAutoConfiguration#sbomEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.sbom.SbomEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "SbomEndpointAutoConfiguration#sbomEndpointWebExtension": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.sbom.SbomEndpoint; SearchStrategy: all) found bean 'sbomEndpoint'; @ConditionalOnMissingBean (types: org.springframework.boot.actuate.sbom.SbomEndpointWebExtension; SearchStrategy: all) did not find any beans"
                    },
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "ScheduledTasksEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "ScheduledTasksEndpointAutoConfiguration#scheduledTasksEndpoint": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.actuate.scheduling.ScheduledTasksEndpoint; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ScheduledTasksObservabilityAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: io.micrometer.observation.ObservationRegistry; SearchStrategy: all) found bean 'observationRegistry'"
                    }
                ],
                "DiskSpaceHealthContributorAutoConfiguration": [
                    {
                        "condition": "OnEnabledHealthIndicatorCondition",
                        "message": "@ConditionalOnEnabledHealthIndicator management.health.defaults.enabled is considered true"
                    }
                ],
                "DiskSpaceHealthContributorAutoConfiguration#diskSpaceHealthIndicator": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (names: diskSpaceHealthIndicator; SearchStrategy: all) did not find any beans"
                    }
                ],
                "HttpExchangesEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "MappingsEndpointAutoConfiguration": [
                    {
                        "condition": "OnAvailableEndpointCondition",
                        "message": "@ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.web.exposure' property"
                    }
                ],
                "MappingsEndpointAutoConfiguration.ServletWebConfiguration": [
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    }
                ],
                "MappingsEndpointAutoConfiguration.ServletWebConfiguration.SpringMvcConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.web.servlet.DispatcherServlet; SearchStrategy: all) found bean 'dispatcherServlet'"
                    }
                ],
                "ManagementContextAutoConfiguration.SameManagementContextConfiguration": [
                    {
                        "condition": "OnManagementPortCondition",
                        "message": "Management Port actual port type (SAME) matched required type"
                    }
                ],
                "ServletManagementContextAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'jakarta.servlet.Servlet'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    }
                ],
                "SpringApplicationAdminJmxAutoConfiguration": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (spring.application.admin.enabled=true) matched"
                    }
                ],
                "SpringApplicationAdminJmxAutoConfiguration#springApplicationAdminRegistrar": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrar; SearchStrategy: all) did not find any beans"
                    }
                ],
                "AopAutoConfiguration": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (spring.aop.auto=true) matched"
                    }
                ],
                "AopAutoConfiguration.ClassProxyingConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnMissingClass did not find unwanted class 'org.aspectj.weaver.Advice'"
                    },
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (spring.aop.proxy-target-class=true) matched"
                    }
                ],
                "ApplicationAvailabilityAutoConfiguration#applicationAvailability": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.availability.ApplicationAvailability; SearchStrategy: all) did not find any beans"
                    }
                ],
                "GenericCacheConfiguration": [
                    {
                        "condition": "CacheCondition",
                        "message": "Cache org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration automatic cache type"
                    }
                ],
                "NoOpCacheConfiguration": [
                    {
                        "condition": "CacheCondition",
                        "message": "Cache org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration automatic cache type"
                    }
                ],
                "SimpleCacheConfiguration": [
                    {
                        "condition": "CacheCondition",
                        "message": "Cache org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration automatic cache type"
                    }
                ],
                "LifecycleAutoConfiguration#defaultLifecycleProcessor": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (names: lifecycleProcessor; SearchStrategy: current) did not find any beans"
                    }
                ],
                "PropertyPlaceholderAutoConfiguration#propertySourcesPlaceholderConfigurer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.context.support.PropertySourcesPlaceholderConfigurer; SearchStrategy: current) did not find any beans"
                    }
                ],
                "HttpMessageConvertersAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.http.converter.HttpMessageConverter'"
                    },
                    {
                        "condition": "HttpMessageConvertersAutoConfiguration.NotReactiveWebApplicationCondition",
                        "message": "NoneNestedConditions 0 matched 1 did not; NestedCondition on HttpMessageConvertersAutoConfiguration.NotReactiveWebApplicationCondition.ReactiveWebApplication did not find reactive web application classes"
                    }
                ],
                "HttpMessageConvertersAutoConfiguration#messageConverters": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.autoconfigure.http.HttpMessageConverters; SearchStrategy: all) did not find any beans"
                    }
                ],
                "HttpMessageConvertersAutoConfiguration.StringHttpMessageConverterConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.http.converter.StringHttpMessageConverter'"
                    }
                ],
                "HttpMessageConvertersAutoConfiguration.StringHttpMessageConverterConfiguration#stringHttpMessageConverter": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.http.converter.StringHttpMessageConverter; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JacksonHttpMessageConvertersConfiguration.MappingJackson2HttpMessageConverterConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'com.fasterxml.jackson.databind.ObjectMapper'"
                    },
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (spring.mvc.converters.preferred-json-mapper=jackson) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: com.fasterxml.jackson.databind.ObjectMapper; SearchStrategy: all) found bean 'jacksonObjectMapper'"
                    }
                ],
                "JacksonHttpMessageConvertersConfiguration.MappingJackson2HttpMessageConverterConfiguration#mappingJackson2HttpMessageConverter": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.http.converter.json.MappingJackson2HttpMessageConverter ignored: org.springframework.hateoas.server.mvc.TypeConstrainedMappingJackson2HttpMessageConverter,org.springframework.data.rest.webmvc.alps.AlpsJsonHttpMessageConverter; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JacksonAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'com.fasterxml.jackson.databind.ObjectMapper'"
                    }
                ],
                "JacksonAutoConfiguration.Jackson2ObjectMapperBuilderCustomizerConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.http.converter.json.Jackson2ObjectMapperBuilder'"
                    }
                ],
                "JacksonAutoConfiguration.JacksonObjectMapperBuilderConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.http.converter.json.Jackson2ObjectMapperBuilder'"
                    }
                ],
                "JacksonAutoConfiguration.JacksonObjectMapperBuilderConfiguration#jacksonObjectMapperBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.http.converter.json.Jackson2ObjectMapperBuilder; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JacksonAutoConfiguration.JacksonObjectMapperConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.http.converter.json.Jackson2ObjectMapperBuilder'"
                    }
                ],
                "JacksonAutoConfiguration.JacksonObjectMapperConfiguration#jacksonObjectMapper": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: com.fasterxml.jackson.databind.ObjectMapper; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JacksonAutoConfiguration.ParameterNamesModuleConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'com.fasterxml.jackson.module.paramnames.ParameterNamesModule'"
                    }
                ],
                "JacksonAutoConfiguration.ParameterNamesModuleConfiguration#parameterNamesModule": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: com.fasterxml.jackson.module.paramnames.ParameterNamesModule; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JmxAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.jmx.export.MBeanExporter'"
                    },
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (spring.jmx.enabled=true) matched"
                    }
                ],
                "JmxAutoConfiguration#mbeanExporter": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.jmx.export.MBeanExporter; SearchStrategy: current) did not find any beans"
                    }
                ],
                "JmxAutoConfiguration#mbeanServer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: javax.management.MBeanServer; SearchStrategy: all) did not find any beans"
                    }
                ],
                "JmxAutoConfiguration#objectNamingStrategy": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.jmx.export.naming.ObjectNamingStrategy; SearchStrategy: current) did not find any beans"
                    }
                ],
                "SqlInitializationAutoConfiguration": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (spring.sql.init.enabled) matched"
                    },
                    {
                        "condition": "SqlInitializationAutoConfiguration.SqlInitializationModeCondition",
                        "message": "NoneNestedConditions 0 matched 1 did not; NestedCondition on SqlInitializationAutoConfiguration.SqlInitializationModeCondition.ModeIsNever @ConditionalOnProperty (spring.sql.init.mode=never) did not find property 'mode'"
                    }
                ],
                "SslAutoConfiguration#sslBundleRegistry": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.ssl.SslBundleRegistry,org.springframework.boot.ssl.SslBundles; SearchStrategy: all) did not find any beans"
                    }
                ],
                "TaskExecutionAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor'"
                    }
                ],
                "TaskExecutorConfigurations.SimpleAsyncTaskExecutorBuilderConfiguration#simpleAsyncTaskExecutorBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.task.SimpleAsyncTaskExecutorBuilder; SearchStrategy: all) did not find any beans"
                    },
                    {
                        "condition": "OnThreadingCondition",
                        "message": "@ConditionalOnThreading found PLATFORM"
                    }
                ],
                "TaskExecutorConfigurations.TaskExecutorBuilderConfiguration#taskExecutorBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.task.TaskExecutorBuilder; SearchStrategy: all) did not find any beans"
                    }
                ],
                "TaskExecutorConfigurations.TaskExecutorConfiguration": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: java.util.concurrent.Executor; SearchStrategy: all) did not find any beans"
                    }
                ],
                "TaskExecutorConfigurations.TaskExecutorConfiguration#applicationTaskExecutor": [
                    {
                        "condition": "OnThreadingCondition",
                        "message": "@ConditionalOnThreading found PLATFORM"
                    }
                ],
                "TaskExecutorConfigurations.ThreadPoolTaskExecutorBuilderConfiguration#threadPoolTaskExecutorBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.task.TaskExecutorBuilder,org.springframework.boot.task.ThreadPoolTaskExecutorBuilder; SearchStrategy: all) did not find any beans"
                    }
                ],
                "TaskSchedulingAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.scheduling.concurrent.ThreadPoolTaskScheduler'"
                    }
                ],
                "TaskSchedulingConfigurations.SimpleAsyncTaskSchedulerBuilderConfiguration#simpleAsyncTaskSchedulerBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.task.SimpleAsyncTaskSchedulerBuilder; SearchStrategy: all) did not find any beans"
                    },
                    {
                        "condition": "OnThreadingCondition",
                        "message": "@ConditionalOnThreading found PLATFORM"
                    }
                ],
                "TaskSchedulingConfigurations.TaskSchedulerBuilderConfiguration#taskSchedulerBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.task.TaskSchedulerBuilder; SearchStrategy: all) did not find any beans"
                    }
                ],
                "TaskSchedulingConfigurations.ThreadPoolTaskSchedulerBuilderConfiguration#threadPoolTaskSchedulerBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.task.TaskSchedulerBuilder,org.springframework.boot.task.ThreadPoolTaskSchedulerBuilder; SearchStrategy: all) did not find any beans"
                    }
                ],
                "RestClientAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.web.client.RestClient'"
                    },
                    {
                        "condition": "NotReactiveWebApplicationCondition",
                        "message": "NoneNestedConditions 0 matched 1 did not; NestedCondition on NotReactiveWebApplicationCondition.ReactiveWebApplication did not find reactive web application classes"
                    }
                ],
                "RestClientAutoConfiguration#httpMessageConvertersRestClientCustomizer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.autoconfigure.web.client.HttpMessageConvertersRestClientCustomizer; SearchStrategy: all) did not find any beans"
                    }
                ],
                "RestClientAutoConfiguration#restClientBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.web.client.RestClient$Builder; SearchStrategy: all) did not find any beans"
                    }
                ],
                "RestClientAutoConfiguration#restClientBuilderConfigurer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.autoconfigure.web.client.RestClientBuilderConfigurer; SearchStrategy: all) did not find any beans"
                    }
                ],
                "RestClientAutoConfiguration#restClientSsl": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.boot.ssl.SslBundles; SearchStrategy: all) found bean 'sslBundleRegistry'; @ConditionalOnMissingBean (types: org.springframework.boot.autoconfigure.web.client.RestClientSsl; SearchStrategy: all) did not find any beans"
                    }
                ],
                "RestTemplateAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.web.client.RestTemplate'"
                    },
                    {
                        "condition": "NotReactiveWebApplicationCondition",
                        "message": "NoneNestedConditions 0 matched 1 did not; NestedCondition on NotReactiveWebApplicationCondition.ReactiveWebApplication did not find reactive web application classes"
                    }
                ],
                "RestTemplateAutoConfiguration#restTemplateBuilder": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.web.client.RestTemplateBuilder; SearchStrategy: all) did not find any beans"
                    }
                ],
                "EmbeddedWebServerFactoryCustomizerAutoConfiguration": [
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "@ConditionalOnWebApplication (required) found 'session' scope"
                    },
                    {
                        "condition": "OnWarDeploymentCondition",
                        "message": "@ConditionalOnWarDeployment the application is not deployed as a WAR file."
                    }
                ],
                "EmbeddedWebServerFactoryCustomizerAutoConfiguration.TomcatWebServerFactoryCustomizerConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'org.apache.catalina.startup.Tomcat', 'org.apache.coyote.UpgradeProtocol'"
                    }
                ],
                "DispatcherServletAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    }
                ],
                "DispatcherServletAutoConfiguration.DispatcherServletConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'jakarta.servlet.ServletRegistration'"
                    },
                    {
                        "condition": "DispatcherServletAutoConfiguration.DefaultDispatcherServletCondition",
                        "message": "Default DispatcherServlet did not find dispatcher servlet beans"
                    }
                ],
                "DispatcherServletAutoConfiguration.DispatcherServletRegistrationConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'jakarta.servlet.ServletRegistration'"
                    },
                    {
                        "condition": "DispatcherServletAutoConfiguration.DispatcherServletRegistrationCondition",
                        "message": "DispatcherServlet Registration did not find servlet registration bean"
                    }
                ],
                "DispatcherServletAutoConfiguration.DispatcherServletRegistrationConfiguration#dispatcherServletRegistration": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (names: dispatcherServlet types: org.springframework.web.servlet.DispatcherServlet; SearchStrategy: all) found bean 'dispatcherServlet'"
                    }
                ],
                "HttpEncodingAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.springframework.web.filter.CharacterEncodingFilter'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    },
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (server.servlet.encoding.enabled) matched"
                    }
                ],
                "HttpEncodingAutoConfiguration#characterEncodingFilter": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.web.filter.CharacterEncodingFilter; SearchStrategy: all) did not find any beans"
                    }
                ],
                "MultipartAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'jakarta.servlet.Servlet', 'org.springframework.web.multipart.support.StandardServletMultipartResolver', 'jakarta.servlet.MultipartConfigElement'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    },
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (spring.servlet.multipart.enabled) matched"
                    }
                ],
                "MultipartAutoConfiguration#multipartConfigElement": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: jakarta.servlet.MultipartConfigElement; SearchStrategy: all) did not find any beans"
                    }
                ],
                "MultipartAutoConfiguration#multipartResolver": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.web.multipart.MultipartResolver; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ServletWebServerFactoryAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'jakarta.servlet.ServletRequest'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    }
                ],
                "ServletWebServerFactoryAutoConfiguration#tomcatServletWebServerFactoryCustomizer": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required class 'org.apache.catalina.startup.Tomcat'"
                    }
                ],
                "ServletWebServerFactoryConfiguration.EmbeddedTomcat": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'jakarta.servlet.Servlet', 'org.apache.catalina.startup.Tomcat', 'org.apache.coyote.UpgradeProtocol'"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.web.servlet.server.ServletWebServerFactory; SearchStrategy: current) did not find any beans"
                    }
                ],
                "WebMvcAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'jakarta.servlet.Servlet', 'org.springframework.web.servlet.DispatcherServlet', 'org.springframework.web.servlet.config.annotation.WebMvcConfigurer'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.web.servlet.config.annotation.WebMvcConfigurationSupport; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebMvcAutoConfiguration#formContentFilter": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (spring.mvc.formcontent.filter.enabled) matched"
                    },
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.web.filter.FormContentFilter; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebMvcAutoConfiguration.EnableWebMvcConfiguration#flashMapManager": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (names: flashMapManager; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebMvcAutoConfiguration.EnableWebMvcConfiguration#localeResolver": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (names: localeResolver; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebMvcAutoConfiguration.EnableWebMvcConfiguration#themeResolver": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (names: themeResolver; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebMvcAutoConfiguration.ResourceChainCustomizerConfiguration": [
                    {
                        "condition": "OnEnabledResourceChainCondition",
                        "message": "@ConditionalOnEnabledResourceChain found class org.webjars.WebJarAssetLocator"
                    }
                ],
                "WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter#defaultViewResolver": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.web.servlet.view.InternalResourceViewResolver; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter#requestContextFilter": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.web.context.request.RequestContextListener,org.springframework.web.filter.RequestContextFilter; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter#viewResolver": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.web.servlet.ViewResolver; SearchStrategy: all) found beans 'defaultViewResolver', 'beanNameViewResolver', 'mvcViewResolver'; @ConditionalOnMissingBean (names: viewResolver types: org.springframework.web.servlet.view.ContentNegotiatingViewResolver; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ErrorMvcAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'jakarta.servlet.Servlet', 'org.springframework.web.servlet.DispatcherServlet'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    }
                ],
                "ErrorMvcAutoConfiguration#basicErrorController": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.web.servlet.error.ErrorController; SearchStrategy: current) did not find any beans"
                    }
                ],
                "ErrorMvcAutoConfiguration#errorAttributes": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.boot.web.servlet.error.ErrorAttributes; SearchStrategy: current) did not find any beans"
                    }
                ],
                "ErrorMvcAutoConfiguration.DefaultErrorViewResolverConfiguration#conventionErrorViewResolver": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnBean (types: org.springframework.web.servlet.DispatcherServlet; SearchStrategy: all) found bean 'dispatcherServlet'; @ConditionalOnMissingBean (types: org.springframework.boot.autoconfigure.web.servlet.error.ErrorViewResolver; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ErrorMvcAutoConfiguration.WhitelabelErrorViewConfiguration": [
                    {
                        "condition": "OnPropertyCondition",
                        "message": "@ConditionalOnProperty (server.error.whitelabel.enabled) matched"
                    },
                    {
                        "condition": "ErrorMvcAutoConfiguration.ErrorTemplateMissingCondition",
                        "message": "ErrorTemplate Missing did not find error template view"
                    }
                ],
                "ErrorMvcAutoConfiguration.WhitelabelErrorViewConfiguration#beanNameViewResolver": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (types: org.springframework.web.servlet.view.BeanNameViewResolver; SearchStrategy: all) did not find any beans"
                    }
                ],
                "ErrorMvcAutoConfiguration.WhitelabelErrorViewConfiguration#defaultErrorView": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (names: error; SearchStrategy: all) did not find any beans"
                    }
                ],
                "WebSocketServletAutoConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'jakarta.servlet.Servlet', 'jakarta.websocket.server.ServerContainer'"
                    },
                    {
                        "condition": "OnWebApplicationCondition",
                        "message": "found 'session' scope"
                    }
                ],
                "WebSocketServletAutoConfiguration.TomcatWebSocketConfiguration": [
                    {
                        "condition": "OnClassCondition",
                        "message": "@ConditionalOnClass found required classes 'org.apache.catalina.startup.Tomcat', 'org.apache.tomcat.websocket.server.WsSci'"
                    }
                ],
                "WebSocketServletAutoConfiguration.TomcatWebSocketConfiguration#websocketServletWebServerCustomizer": [
                    {
                        "condition": "OnBeanCondition",
                        "message": "@ConditionalOnMissingBean (names: websocketServletWebServerCustomizer; SearchStrategy: all) did not find any beans"
                    }
                ]
            },
            "negativeMatches": {
                "SpringDocConfiguration#propertiesResolverForSchema": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (springdoc.api-docs.resolve-schema-properties) did not find property 'springdoc.api-docs.resolve-schema-properties'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocConfiguration#propertyCustomizingConverter": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (types: org.springdoc.core.customizers.PropertyCustomizer; SearchStrategy: all) did not find any beans of type org.springdoc.core.customizers.PropertyCustomizer"
                        }
                    ],
                    "matched": []
                },
                "SpringDocConfiguration#springdocBeanFactoryPostProcessor": {
                    "notMatched": [
                        {
                            "condition": "CacheOrGroupedOpenApiCondition",
                            "message": "AnyNestedCondition 0 matched 2 did not; NestedCondition on CacheOrGroupedOpenApiCondition.OnCacheDisabled found non-matching nested conditions @ConditionalOnProperty (springdoc.cache.disabled) did not find property 'springdoc.cache.disabled'; NestedCondition on CacheOrGroupedOpenApiCondition.OnMultipleOpenApiSupportCondition AnyNestedCondition 0 matched 2 did not; NestedCondition on MultipleOpenApiSupportCondition.OnActuatorDifferentPort found non-matching nested conditions Management Port actual port type (SAME) did not match required type (DIFFERENT), @ConditionalOnProperty (springdoc.show-actuator) did not find property 'springdoc.show-actuator'; NestedCondition on MultipleOpenApiSupportCondition.OnMultipleOpenApiSupportCondition AnyNestedCondition 0 matched 2 did not; NestedCondition on MultipleOpenApiGroupsCondition.OnGroupConfigProperty @ConditionalOnProperty (springdoc.group-configs[0].group) did not find property 'springdoc.group-configs[0].group'; NestedCondition on MultipleOpenApiGroupsCondition.OnGroupedOpenApiBean @ConditionalOnBean (types: org.springdoc.core.models.GroupedOpenApi; SearchStrategy: all) did not find any beans of type org.springdoc.core.models.GroupedOpenApi"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass found required class 'org.springframework.boot.context.properties.bind.BindResult'"
                        }
                    ]
                },
                "SpringDocConfiguration#springdocBeanFactoryPostProcessor2": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnMissingClass found unwanted class 'org.springframework.boot.context.properties.bind.BindResult'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocConfiguration.SpringDocActuatorConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (springdoc.show-actuator) did not find property 'springdoc.show-actuator'"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass found required class 'org.springframework.boot.actuate.autoconfigure.endpoint.web.WebEndpointProperties'"
                        }
                    ]
                },
                "SpringDocConfiguration.SpringDocRepositoryRestConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.rest.core.config.RepositoryRestConfiguration'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocConfiguration.SpringDocWebFluxSupportConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Flux'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocDataRestConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.rest.core.config.RepositoryRestConfiguration'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocFunctionCatalogConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.cloud.function.web.function.FunctionEndpointInitializer'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocGroovyConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'groovy.lang.MetaClass'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocHateoasConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.hateoas.server.LinkRelationProvider'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocJavadocConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.github.therapi.runtimejavadoc.CommentFormatter'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocKotlinConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'kotlin.coroutines.Continuation'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocKotlinxConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'kotlinx.coroutines.flow.Flow'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocNativeConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnExpressionCondition",
                            "message": "@ConditionalOnExpression (#{${springdoc.api-docs.enabled:true} and ${springdoc.enable-native-support:false}}) resulted in false"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "@ConditionalOnWebApplication (required) found 'session' scope"
                        }
                    ]
                },
                "SpringDocPageableConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.domain.Pageable'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocSecurityConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.core.Authentication'"
                        }
                    ],
                    "matched": []
                },
                "SpringDocSortConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.domain.Sort'"
                        }
                    ],
                    "matched": []
                },
                "MultipleOpenApiSupportConfiguration": {
                    "notMatched": [
                        {
                            "condition": "MultipleOpenApiSupportCondition",
                            "message": "AnyNestedCondition 0 matched 2 did not; NestedCondition on MultipleOpenApiSupportCondition.OnActuatorDifferentPort found non-matching nested conditions Management Port actual port type (SAME) did not match required type (DIFFERENT), @ConditionalOnProperty (springdoc.show-actuator) did not find property 'springdoc.show-actuator'; NestedCondition on MultipleOpenApiSupportCondition.OnMultipleOpenApiSupportCondition AnyNestedCondition 0 matched 2 did not; NestedCondition on MultipleOpenApiGroupsCondition.OnGroupConfigProperty @ConditionalOnProperty (springdoc.group-configs[0].group) did not find property 'springdoc.group-configs[0].group'; NestedCondition on MultipleOpenApiGroupsCondition.OnGroupedOpenApiBean @ConditionalOnBean (types: org.springdoc.core.models.GroupedOpenApi; SearchStrategy: all) did not find any beans of type org.springdoc.core.models.GroupedOpenApi"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "found 'session' scope"
                        },
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (springdoc.api-docs.enabled) matched"
                        }
                    ]
                },
                "MultipleOpenApiSupportConfiguration.SpringDocWebMvcActuatorDifferentConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnManagementPortCondition",
                            "message": "Management Port actual port type (SAME) did not match required type (DIFFERENT)"
                        },
                        {
                            "condition": "ConditionEvaluationReport.AncestorsMatchedCondition",
                            "message": "Ancestor org.springdoc.webmvc.core.configuration.MultipleOpenApiSupportConfiguration did not match"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass found required class 'org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping'"
                        }
                    ]
                },
                "SpringDocWebMvcConfiguration.SpringDocWebMvcActuatorConfiguration#actuatorProvider": {
                    "notMatched": [
                        {
                            "condition": "OnExpressionCondition",
                            "message": "@ConditionalOnExpression (#{${springdoc.show-actuator:false} or ${springdoc.use-management-port:false}}) resulted in false"
                        }
                    ],
                    "matched": []
                },
                "SpringDocWebMvcConfiguration.SpringDocWebMvcActuatorConfiguration#openApiActuatorResource": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (springdoc.use-management-port) did not find property 'springdoc.use-management-port'"
                        }
                    ],
                    "matched": []
                },
                "SwaggerConfig#springWebProvider": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnMissingBean (types: org.springdoc.core.providers.SpringWebProvider; SearchStrategy: all) found beans of type 'org.springdoc.core.providers.SpringWebProvider' springWebProvider"
                        }
                    ],
                    "matched": []
                },
                "SwaggerConfig#swaggerUiConfigParameters": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnMissingBean (types: org.springdoc.core.properties.SwaggerUiConfigParameters; SearchStrategy: all) found beans of type 'org.springdoc.core.properties.SwaggerUiConfigParameters' org.springdoc.core.properties.SwaggerUiConfigParameters"
                        }
                    ],
                    "matched": []
                },
                "SwaggerConfig#swaggerUiHome": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (springdoc.swagger-ui.use-root-path=true) did not find property 'springdoc.swagger-ui.use-root-path'"
                        }
                    ],
                    "matched": []
                },
                "SwaggerConfig.SwaggerActuatorWelcomeConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (springdoc.use-management-port) did not find property 'springdoc.use-management-port'"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass found required class 'org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping'"
                        }
                    ]
                },
                "RabbitHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.amqp.rabbit.core.RabbitTemplate'"
                        }
                    ],
                    "matched": []
                },
                "AuditAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.audit.AuditEventRepository; SearchStrategy: all) did not find any beans of type org.springframework.boot.actuate.audit.AuditEventRepository"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (management.auditevents.enabled) matched"
                        }
                    ]
                },
                "AuditEventsEndpointAutoConfiguration#auditEventsEndpoint": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.audit.AuditEventRepository; SearchStrategy: all) did not find any beans of type org.springframework.boot.actuate.audit.AuditEventRepository"
                        }
                    ],
                    "matched": []
                },
                "AvailabilityHealthContributorAutoConfiguration#livenessStateHealthIndicator": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (management.health.livenessstate.enabled=true) did not find property 'enabled'"
                        }
                    ],
                    "matched": []
                },
                "AvailabilityHealthContributorAutoConfiguration#readinessStateHealthIndicator": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (management.health.readinessstate.enabled=true) did not find property 'enabled'"
                        }
                    ],
                    "matched": []
                },
                "AvailabilityProbesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "AvailabilityProbesAutoConfiguration.ProbesCondition",
                            "message": "Probes availability not running on a supported cloud platform"
                        }
                    ],
                    "matched": []
                },
                "CassandraHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.datastax.oss.driver.api.core.CqlSession'"
                        }
                    ],
                    "matched": []
                },
                "CassandraReactiveHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.datastax.oss.driver.api.core.CqlSession'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveCloudFoundryActuatorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "@ConditionalOnWebApplication did not find reactive web application classes"
                        }
                    ],
                    "matched": []
                },
                "CloudFoundryActuatorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnCloudPlatformCondition",
                            "message": "@ConditionalOnCloudPlatform did not find CLOUD_FOUNDRY"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'"
                        },
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "found 'session' scope"
                        },
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (management.cloudfoundry.enabled) matched"
                        }
                    ]
                },
                "CouchbaseHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.couchbase.client.java.Cluster'"
                        }
                    ],
                    "matched": []
                },
                "CouchbaseReactiveHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.couchbase.client.java.Cluster'"
                        }
                    ],
                    "matched": []
                },
                "ElasticsearchReactiveHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Flux'"
                        }
                    ],
                    "matched": []
                },
                "MongoHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.mongodb.core.MongoTemplate'"
                        }
                    ],
                    "matched": []
                },
                "MongoReactiveHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Flux'"
                        }
                    ],
                    "matched": []
                },
                "RedisHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.redis.connection.RedisConnectionFactory'"
                        }
                    ],
                    "matched": []
                },
                "RedisReactiveHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Flux'"
                        }
                    ],
                    "matched": []
                },
                "ElasticsearchRestHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.elasticsearch.client.RestClient'"
                        }
                    ],
                    "matched": []
                },
                "ServletEndpointManagementContextConfiguration.JerseyServletEndpointManagementContextConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.glassfish.jersey.server.ResourceConfig'"
                        }
                    ],
                    "matched": []
                },
                "JerseyWebEndpointManagementContextConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.glassfish.jersey.server.ResourceConfig'"
                        }
                    ],
                    "matched": []
                },
                "WebFluxEndpointManagementContextConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.reactive.DispatcherHandler'"
                        }
                    ],
                    "matched": []
                },
                "WebMvcEndpointManagementContextConfiguration#managementHealthEndpointWebMvcHandlerMapping": {
                    "notMatched": [
                        {
                            "condition": "OnManagementPortCondition",
                            "message": "Management Port actual port type (SAME) did not match required type (DIFFERENT)"
                        }
                    ],
                    "matched": []
                },
                "FlywayEndpointAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.flywaydb.core.Flyway'"
                        }
                    ],
                    "matched": []
                },
                "HazelcastHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.hazelcast.core.HazelcastInstance'"
                        }
                    ],
                    "matched": []
                },
                "HealthEndpointReactiveWebExtensionConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "did not find reactive web application classes"
                        }
                    ],
                    "matched": []
                },
                "HealthEndpointWebExtensionConfiguration.JerseyAdditionalHealthEndpointPathsConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.glassfish.jersey.server.ResourceConfig'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveHealthEndpointConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Flux'"
                        }
                    ],
                    "matched": []
                },
                "InfluxDbHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.influxdb.InfluxDB'"
                        }
                    ],
                    "matched": []
                },
                "InfoContributorAutoConfiguration#buildInfoContributor": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnSingleCandidate (types: org.springframework.boot.info.BuildProperties; SearchStrategy: all) did not find any beans"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnEnabledInfoContributorCondition",
                            "message": "@ConditionalOnEnabledInfoContributor management.info.defaults.enabled is considered true"
                        }
                    ]
                },
                "InfoContributorAutoConfiguration#envInfoContributor": {
                    "notMatched": [
                        {
                            "condition": "OnEnabledInfoContributorCondition",
                            "message": "@ConditionalOnEnabledInfoContributor management.info.env.enabled is not true"
                        }
                    ],
                    "matched": []
                },
                "InfoContributorAutoConfiguration#gitInfoContributor": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnSingleCandidate (types: org.springframework.boot.info.GitProperties; SearchStrategy: all) did not find any beans"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnEnabledInfoContributorCondition",
                            "message": "@ConditionalOnEnabledInfoContributor management.info.defaults.enabled is considered true"
                        }
                    ]
                },
                "InfoContributorAutoConfiguration#javaInfoContributor": {
                    "notMatched": [
                        {
                            "condition": "OnEnabledInfoContributorCondition",
                            "message": "@ConditionalOnEnabledInfoContributor management.info.java.enabled is not true"
                        }
                    ],
                    "matched": []
                },
                "InfoContributorAutoConfiguration#osInfoContributor": {
                    "notMatched": [
                        {
                            "condition": "OnEnabledInfoContributorCondition",
                            "message": "@ConditionalOnEnabledInfoContributor management.info.os.enabled is not true"
                        }
                    ],
                    "matched": []
                },
                "InfoContributorAutoConfiguration#processInfoContributor": {
                    "notMatched": [
                        {
                            "condition": "OnEnabledInfoContributorCondition",
                            "message": "@ConditionalOnEnabledInfoContributor management.info.process.enabled is not true"
                        }
                    ],
                    "matched": []
                },
                "IntegrationGraphEndpointAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.integration.graph.IntegrationGraphServer'"
                        }
                    ],
                    "matched": []
                },
                "DataSourceHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.jdbc.core.JdbcTemplate'"
                        }
                    ],
                    "matched": []
                },
                "JmsHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.jms.ConnectionFactory'"
                        }
                    ],
                    "matched": []
                },
                "LdapHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.ldap.core.LdapOperations'"
                        }
                    ],
                    "matched": []
                },
                "LiquibaseEndpointAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'liquibase.integration.spring.SpringLiquibase'"
                        }
                    ],
                    "matched": []
                },
                "LogFileWebEndpointAutoConfiguration#logFileWebEndpoint": {
                    "notMatched": [
                        {
                            "condition": "LogFileWebEndpointAutoConfiguration.LogFileCondition",
                            "message": "Log File did not find logging file"
                        }
                    ],
                    "matched": []
                },
                "MailHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.mail.javamail.JavaMailSenderImpl'"
                        }
                    ],
                    "matched": []
                },
                "HeapDumpWebEndpointAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnAvailableEndpointCondition",
                            "message": "@ConditionalOnAvailableEndpoint no 'management.endpoints' property marked it as exposed"
                        }
                    ],
                    "matched": []
                },
                "ThreadDumpEndpointAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnAvailableEndpointCondition",
                            "message": "@ConditionalOnAvailableEndpoint no 'management.endpoints' property marked it as exposed"
                        }
                    ],
                    "matched": []
                },
                "CompositeMeterRegistryConfiguration": {
                    "notMatched": [
                        {
                            "condition": "CompositeMeterRegistryConfiguration.MultipleNonPrimaryMeterRegistriesCondition",
                            "message": "NoneNestedConditions 1 matched 1 did not; NestedCondition on CompositeMeterRegistryConfiguration.MultipleNonPrimaryMeterRegistriesCondition.SingleInjectableMeterRegistry @ConditionalOnSingleCandidate (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found a single bean 'simpleMeterRegistry'; NestedCondition on CompositeMeterRegistryConfiguration.MultipleNonPrimaryMeterRegistriesCondition.NoMeterRegistryCondition @ConditionalOnMissingBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found beans of type 'io.micrometer.core.instrument.MeterRegistry' simpleMeterRegistry"
                        }
                    ],
                    "matched": []
                },
                "KafkaMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.kafka.core.ProducerFactory'"
                        }
                    ],
                    "matched": []
                },
                "Log4J2MetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.apache.logging.log4j.core.LoggerContext'"
                        }
                    ],
                    "matched": []
                },
                "MetricsAspectsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.aspectj.weaver.Advice'"
                        }
                    ],
                    "matched": []
                },
                "NoOpMeterRegistryConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnMissingBean (types: io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) found beans of type 'io.micrometer.core.instrument.MeterRegistry' simpleMeterRegistry"
                        }
                    ],
                    "matched": []
                },
                "RabbitMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.rabbitmq.client.ConnectionFactory'"
                        }
                    ],
                    "matched": []
                },
                "CacheMeterBinderProvidersConfiguration.Cache2kCacheMeterBinderProviderConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required classes 'org.cache2k.Cache2kBuilder', 'org.cache2k.extra.spring.SpringCache2kCache', 'org.cache2k.extra.micrometer.Cache2kCacheMetrics'"
                        }
                    ],
                    "matched": []
                },
                "CacheMeterBinderProvidersConfiguration.CaffeineCacheMeterBinderProviderConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required classes 'org.springframework.cache.caffeine.CaffeineCache', 'com.github.benmanes.caffeine.cache.Cache'"
                        }
                    ],
                    "matched": []
                },
                "CacheMeterBinderProvidersConfiguration.HazelcastCacheMeterBinderProviderConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required classes 'com.hazelcast.spring.cache.HazelcastCache', 'com.hazelcast.core.Hazelcast'"
                        }
                    ],
                    "matched": []
                },
                "CacheMeterBinderProvidersConfiguration.JCacheCacheMeterBinderProviderConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required classes 'org.springframework.cache.jcache.JCacheCache', 'javax.cache.CacheManager'"
                        }
                    ],
                    "matched": []
                },
                "CacheMeterBinderProvidersConfiguration.RedisCacheMeterBinderProviderConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.redis.cache.RedisCache'"
                        }
                    ],
                    "matched": []
                },
                "CacheMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (types: org.springframework.cache.CacheManager; SearchStrategy: all) did not find any beans of type org.springframework.cache.CacheManager"
                        }
                    ],
                    "matched": []
                },
                "RepositoryMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.repository.Repository'"
                        }
                    ],
                    "matched": []
                },
                "AppOpticsMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.appoptics.AppOpticsMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "AtlasMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.atlas.AtlasMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "DatadogMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.datadog.DatadogMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "DynatraceMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.dynatrace.DynatraceMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "ElasticMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.elastic.ElasticMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "GangliaMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.ganglia.GangliaMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "GraphiteMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.graphite.GraphiteMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "HumioMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.humio.HumioMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "InfluxMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.influx.InfluxMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "JmxMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.jmx.JmxMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "KairosMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.kairos.KairosMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "NewRelicMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.newrelic.NewRelicMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "OtlpMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.registry.otlp.OtlpMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "PrometheusMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.prometheusmetrics.PrometheusMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "PrometheusSimpleclientMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.prometheus.PrometheusMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "SignalFxMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.signalfx.SignalFxMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "StackdriverMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.stackdriver.StackdriverMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "StatsdMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.statsd.StatsdMeterRegistry'"
                        }
                    ],
                    "matched": []
                },
                "WavefrontMetricsExportAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.wavefront.sdk.common.WavefrontSender'"
                        }
                    ],
                    "matched": []
                },
                "DataSourcePoolMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (types: javax.sql.DataSource,io.micrometer.core.instrument.MeterRegistry; SearchStrategy: all) did not find any beans of type javax.sql.DataSource"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass found required classes 'javax.sql.DataSource', 'io.micrometer.core.instrument.MeterRegistry'"
                        }
                    ]
                },
                "DataSourcePoolMetricsAutoConfiguration.HikariDataSourceMetricsConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.zaxxer.hikari.HikariDataSource'"
                        },
                        {
                            "condition": "ConditionEvaluationReport.AncestorsMatchedCondition",
                            "message": "Ancestor org.springframework.boot.actuate.autoconfigure.metrics.jdbc.DataSourcePoolMetricsAutoConfiguration did not match"
                        }
                    ],
                    "matched": []
                },
                "JerseyServerMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.glassfish.jersey.micrometer.server.ObservationApplicationEventListener'"
                        }
                    ],
                    "matched": []
                },
                "MongoMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.mongodb.MongoClientSettings'"
                        }
                    ],
                    "matched": []
                },
                "HibernateMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.persistence.EntityManagerFactory'"
                        }
                    ],
                    "matched": []
                },
                "ConnectionPoolMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.r2dbc.pool.ConnectionPool'"
                        }
                    ],
                    "matched": []
                },
                "LettuceMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.lettuce.core.metrics.MicrometerCommandLatencyRecorder'"
                        }
                    ],
                    "matched": []
                },
                "JettyMetricsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.eclipse.jetty.server.Server'"
                        }
                    ],
                    "matched": []
                },
                "Neo4jHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.neo4j.driver.Driver'"
                        }
                    ],
                    "matched": []
                },
                "ObservationAutoConfiguration.MeterObservationHandlerConfiguration.TracingAndMetricsObservationHandlerConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean did not find required type 'io.micrometer.tracing.Tracer'"
                        },
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (types: io.micrometer.tracing.Tracer; SearchStrategy: all) did not find any beans of type io.micrometer.tracing.Tracer"
                        }
                    ],
                    "matched": []
                },
                "ObservationAutoConfiguration.MetricsWithTracingConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.tracing.Tracer'"
                        }
                    ],
                    "matched": []
                },
                "ObservationAutoConfiguration.ObservedAspectConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.aspectj.weaver.Advice'"
                        }
                    ],
                    "matched": []
                },
                "ObservationAutoConfiguration.OnlyTracingConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.tracing.Tracer'"
                        }
                    ],
                    "matched": []
                },
                "BatchObservationAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.batch.core.configuration.annotation.BatchObservabilityBeanPostProcessor'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlObservationAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'graphql.GraphQL'"
                        }
                    ],
                    "matched": []
                },
                "WebClientObservationConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.reactive.function.client.WebClient'"
                        }
                    ],
                    "matched": []
                },
                "WebFluxObservationAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "@ConditionalOnWebApplication did not find reactive web application classes"
                        }
                    ],
                    "matched": []
                },
                "OpenTelemetryAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.tracing.otel.bridge.OtelTracer'"
                        }
                    ],
                    "matched": []
                },
                "QuartzEndpointAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.quartz.Scheduler'"
                        }
                    ],
                    "matched": []
                },
                "ConnectionFactoryHealthContributorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.r2dbc.spi.ConnectionFactory'"
                        }
                    ],
                    "matched": []
                },
                "R2dbcObservationAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.r2dbc.proxy.ProxyConnectionFactory'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveManagementWebSecurityAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity'"
                        }
                    ],
                    "matched": []
                },
                "ManagementWebSecurityAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "DefaultWebSecurityCondition",
                            "message": "AllNestedConditions 1 matched 1 did not; NestedCondition on DefaultWebSecurityCondition.Beans @ConditionalOnMissingBean (types: org.springframework.security.web.SecurityFilterChain; SearchStrategy: all) did not find any beans; NestedCondition on DefaultWebSecurityCondition.Classes @ConditionalOnClass did not find required classes 'org.springframework.security.web.SecurityFilterChain', 'org.springframework.security.config.annotation.web.builders.HttpSecurity'"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "found 'session' scope"
                        }
                    ]
                },
                "SecurityRequestMatchersManagementContextConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.web.util.matcher.RequestMatcher'"
                        }
                    ],
                    "matched": []
                },
                "SessionsEndpointAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.session.Session'"
                        }
                    ],
                    "matched": []
                },
                "StartupEndpointAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "StartupEndpointAutoConfiguration.ApplicationStartupCondition",
                            "message": "ApplicationStartup configured applicationStartup is of type class org.springframework.core.metrics.DefaultApplicationStartup, expected BufferingApplicationStartup."
                        }
                    ],
                    "matched": []
                },
                "BraveAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'brave.Tracer'"
                        }
                    ],
                    "matched": []
                },
                "MicrometerTracingAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.tracing.Tracer'"
                        }
                    ],
                    "matched": []
                },
                "NoopTracerAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.tracing.Tracer'"
                        }
                    ],
                    "matched": []
                },
                "OtlpAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.tracing.otel.bridge.OtelTracer'"
                        }
                    ],
                    "matched": []
                },
                "PrometheusExemplarsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.tracing.Tracer'"
                        }
                    ],
                    "matched": []
                },
                "PrometheusSimpleclientExemplarsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.micrometer.tracing.Tracer'"
                        }
                    ],
                    "matched": []
                },
                "WavefrontTracingAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.wavefront.sdk.common.WavefrontSender'"
                        }
                    ],
                    "matched": []
                },
                "ZipkinAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'zipkin2.reporter.Encoding'"
                        }
                    ],
                    "matched": []
                },
                "WavefrontAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.wavefront.sdk.common.application.ApplicationTags'"
                        }
                    ],
                    "matched": []
                },
                "HttpExchangesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.web.exchanges.HttpExchangeRepository; SearchStrategy: all) did not find any beans of type org.springframework.boot.actuate.web.exchanges.HttpExchangeRepository"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "@ConditionalOnWebApplication (required) found 'session' scope"
                        },
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (management.httpexchanges.recording.enabled) matched"
                        }
                    ]
                },
                "HttpExchangesAutoConfiguration.ReactiveHttpExchangesConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "did not find reactive web application classes"
                        },
                        {
                            "condition": "ConditionEvaluationReport.AncestorsMatchedCondition",
                            "message": "Ancestor org.springframework.boot.actuate.autoconfigure.web.exchanges.HttpExchangesAutoConfiguration did not match"
                        }
                    ],
                    "matched": []
                },
                "HttpExchangesAutoConfiguration.ServletHttpExchangesConfiguration": {
                    "notMatched": [
                        {
                            "condition": "ConditionEvaluationReport.AncestorsMatchedCondition",
                            "message": "Ancestor org.springframework.boot.actuate.autoconfigure.web.exchanges.HttpExchangesAutoConfiguration did not match"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "found 'session' scope"
                        }
                    ]
                },
                "HttpExchangesEndpointAutoConfiguration#httpExchangesEndpoint": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (types: org.springframework.boot.actuate.web.exchanges.HttpExchangeRepository; SearchStrategy: all) did not find any beans of type org.springframework.boot.actuate.web.exchanges.HttpExchangeRepository"
                        }
                    ],
                    "matched": []
                },
                "JerseySameManagementContextConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.glassfish.jersey.server.ResourceConfig'"
                        }
                    ],
                    "matched": []
                },
                "MappingsEndpointAutoConfiguration.ReactiveWebConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.reactive.DispatcherHandler'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveManagementContextAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Flux'"
                        }
                    ],
                    "matched": []
                },
                "ManagementContextAutoConfiguration.DifferentManagementContextConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnManagementPortCondition",
                            "message": "Management Port actual port type (SAME) did not match required type (DIFFERENT)"
                        }
                    ],
                    "matched": []
                },
                "ServletManagementContextAutoConfiguration.ApplicationContextFilterConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (management.server.add-application-context-header=true) did not find property 'add-application-context-header'"
                        }
                    ],
                    "matched": []
                },
                "RabbitAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.rabbitmq.client.Channel'"
                        }
                    ],
                    "matched": []
                },
                "AopAutoConfiguration.AspectJAutoProxyingConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.aspectj.weaver.Advice'"
                        }
                    ],
                    "matched": []
                },
                "BatchAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.batch.core.launch.JobLauncher'"
                        }
                    ],
                    "matched": []
                },
                "Cache2kCacheConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.cache2k.Cache2kBuilder'"
                        }
                    ],
                    "matched": []
                },
                "CacheAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (types: org.springframework.cache.interceptor.CacheAspectSupport; SearchStrategy: all) did not find any beans of type org.springframework.cache.interceptor.CacheAspectSupport"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass found required class 'org.springframework.cache.CacheManager'"
                        }
                    ]
                },
                "CacheAutoConfiguration.CacheManagerEntityManagerFactoryDependsOnPostProcessor": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean'"
                        },
                        {
                            "condition": "ConditionEvaluationReport.AncestorsMatchedCondition",
                            "message": "Ancestor org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration did not match"
                        }
                    ],
                    "matched": []
                },
                "CaffeineCacheConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.github.benmanes.caffeine.cache.Caffeine'"
                        }
                    ],
                    "matched": []
                },
                "CouchbaseCacheConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.couchbase.client.java.Cluster'"
                        }
                    ],
                    "matched": []
                },
                "HazelcastCacheConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.hazelcast.core.HazelcastInstance'"
                        }
                    ],
                    "matched": []
                },
                "InfinispanCacheConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.infinispan.spring.embedded.provider.SpringEmbeddedCacheManager'"
                        }
                    ],
                    "matched": []
                },
                "JCacheCacheConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'javax.cache.Caching'"
                        }
                    ],
                    "matched": []
                },
                "RedisCacheConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.redis.connection.RedisConnectionFactory'"
                        }
                    ],
                    "matched": []
                },
                "CassandraAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.datastax.oss.driver.api.core.CqlSession'"
                        }
                    ],
                    "matched": []
                },
                "MessageSourceAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "MessageSourceAutoConfiguration.ResourceBundleCondition",
                            "message": "ResourceBundle did not find bundle with basename messages"
                        }
                    ],
                    "matched": []
                },
                "CouchbaseAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.couchbase.client.java.Cluster'"
                        }
                    ],
                    "matched": []
                },
                "PersistenceExceptionTranslationAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.dao.annotation.PersistenceExceptionTranslationPostProcessor'"
                        }
                    ],
                    "matched": []
                },
                "CassandraDataAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.datastax.oss.driver.api.core.CqlSession'"
                        }
                    ],
                    "matched": []
                },
                "CassandraReactiveDataAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.datastax.oss.driver.api.core.CqlSession'"
                        }
                    ],
                    "matched": []
                },
                "CassandraReactiveRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.cassandra.ReactiveSession'"
                        }
                    ],
                    "matched": []
                },
                "CassandraRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.datastax.oss.driver.api.core.CqlSession'"
                        }
                    ],
                    "matched": []
                },
                "CouchbaseDataAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.couchbase.client.java.Bucket'"
                        }
                    ],
                    "matched": []
                },
                "CouchbaseReactiveDataAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.couchbase.client.java.Cluster'"
                        }
                    ],
                    "matched": []
                },
                "CouchbaseReactiveRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.couchbase.client.java.Cluster'"
                        }
                    ],
                    "matched": []
                },
                "CouchbaseRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.couchbase.client.java.Bucket'"
                        }
                    ],
                    "matched": []
                },
                "ElasticsearchDataAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.elasticsearch.client.elc.ElasticsearchTemplate'"
                        }
                    ],
                    "matched": []
                },
                "ElasticsearchRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.elasticsearch.repository.ElasticsearchRepository'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveElasticsearchRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.elasticsearch.client.elc.ReactiveElasticsearchClient'"
                        }
                    ],
                    "matched": []
                },
                "JdbcRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.jdbc.repository.config.AbstractJdbcConfiguration'"
                        }
                    ],
                    "matched": []
                },
                "JpaRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.jpa.repository.JpaRepository'"
                        }
                    ],
                    "matched": []
                },
                "LdapRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.ldap.repository.LdapRepository'"
                        }
                    ],
                    "matched": []
                },
                "MongoDataAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.mongodb.client.MongoClient'"
                        }
                    ],
                    "matched": []
                },
                "MongoReactiveDataAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.mongodb.reactivestreams.client.MongoClient'"
                        }
                    ],
                    "matched": []
                },
                "MongoReactiveRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.mongodb.reactivestreams.client.MongoClient'"
                        }
                    ],
                    "matched": []
                },
                "MongoRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.mongodb.client.MongoClient'"
                        }
                    ],
                    "matched": []
                },
                "Neo4jDataAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.neo4j.driver.Driver'"
                        }
                    ],
                    "matched": []
                },
                "Neo4jReactiveDataAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.neo4j.driver.Driver'"
                        }
                    ],
                    "matched": []
                },
                "Neo4jReactiveRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.neo4j.driver.Driver'"
                        }
                    ],
                    "matched": []
                },
                "Neo4jRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.neo4j.driver.Driver'"
                        }
                    ],
                    "matched": []
                },
                "R2dbcDataAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.r2dbc.core.R2dbcEntityTemplate'"
                        }
                    ],
                    "matched": []
                },
                "R2dbcRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.r2dbc.spi.ConnectionFactory'"
                        }
                    ],
                    "matched": []
                },
                "RedisAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.redis.core.RedisOperations'"
                        }
                    ],
                    "matched": []
                },
                "RedisReactiveAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Flux'"
                        }
                    ],
                    "matched": []
                },
                "RedisRepositoriesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.redis.repository.configuration.EnableRedisRepositories'"
                        }
                    ],
                    "matched": []
                },
                "RepositoryRestMvcAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.rest.webmvc.config.RepositoryRestMvcConfiguration'"
                        }
                    ],
                    "matched": []
                },
                "SpringDataWebAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.data.web.PageableHandlerMethodArgumentResolver'"
                        }
                    ],
                    "matched": []
                },
                "ElasticsearchClientAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'co.elastic.clients.elasticsearch.ElasticsearchClient'"
                        }
                    ],
                    "matched": []
                },
                "ElasticsearchRestClientAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.elasticsearch.client.RestClientBuilder'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveElasticsearchClientAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'co.elastic.clients.transport.ElasticsearchTransport'"
                        }
                    ],
                    "matched": []
                },
                "FlywayAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.flywaydb.core.Flyway'"
                        }
                    ],
                    "matched": []
                },
                "FreeMarkerAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'freemarker.template.Configuration'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'graphql.GraphQL'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlQueryByExampleAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'graphql.GraphQL'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlQuerydslAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.querydsl.core.Query'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlReactiveQueryByExampleAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'graphql.GraphQL'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlReactiveQuerydslAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.querydsl.core.Query'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlWebFluxAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'graphql.GraphQL'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlRSocketAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'graphql.GraphQL'"
                        }
                    ],
                    "matched": []
                },
                "RSocketGraphQlClientAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'graphql.GraphQL'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlWebFluxSecurityAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'graphql.GraphQL'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlWebMvcSecurityAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'graphql.GraphQL'"
                        }
                    ],
                    "matched": []
                },
                "GraphQlWebMvcAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'graphql.GraphQL'"
                        }
                    ],
                    "matched": []
                },
                "GroovyTemplateAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'groovy.text.markup.MarkupTemplateEngine'"
                        }
                    ],
                    "matched": []
                },
                "GsonAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.google.gson.Gson'"
                        }
                    ],
                    "matched": []
                },
                "H2ConsoleAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.h2.server.web.JakartaWebServlet'"
                        }
                    ],
                    "matched": []
                },
                "HypermediaAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.hateoas.EntityModel'"
                        }
                    ],
                    "matched": []
                },
                "HazelcastAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.hazelcast.core.HazelcastInstance'"
                        }
                    ],
                    "matched": []
                },
                "HazelcastJpaDependencyAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.hazelcast.core.HazelcastInstance'"
                        }
                    ],
                    "matched": []
                },
                "GsonHttpMessageConvertersConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.google.gson.Gson'"
                        }
                    ],
                    "matched": []
                },
                "JacksonHttpMessageConvertersConfiguration.MappingJackson2XmlHttpMessageConverterConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.fasterxml.jackson.dataformat.xml.XmlMapper'"
                        }
                    ],
                    "matched": []
                },
                "JsonbHttpMessageConvertersConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.json.bind.Jsonb'"
                        }
                    ],
                    "matched": []
                },
                "CodecsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.reactive.function.client.WebClient'"
                        }
                    ],
                    "matched": []
                },
                "InfluxDbAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.influxdb.InfluxDB'"
                        }
                    ],
                    "matched": []
                },
                "ProjectInfoAutoConfiguration#buildProperties": {
                    "notMatched": [
                        {
                            "condition": "OnResourceCondition",
                            "message": "@ConditionalOnResource did not find resource '${spring.info.build.location:classpath:META-INF/build-info.properties}'"
                        }
                    ],
                    "matched": []
                },
                "ProjectInfoAutoConfiguration#gitProperties": {
                    "notMatched": [
                        {
                            "condition": "ProjectInfoAutoConfiguration.GitResourceAvailableCondition",
                            "message": "GitResource did not find git info at classpath:git.properties"
                        }
                    ],
                    "matched": []
                },
                "IntegrationAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.integration.config.EnableIntegration'"
                        }
                    ],
                    "matched": []
                },
                "DataSourceAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType'"
                        }
                    ],
                    "matched": []
                },
                "DataSourceTransactionManagerAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.jdbc.core.JdbcTemplate'"
                        }
                    ],
                    "matched": []
                },
                "JdbcClientAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnSingleCandidate did not find required type 'org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate'"
                        }
                    ],
                    "matched": []
                },
                "JdbcTemplateAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.jdbc.core.JdbcTemplate'"
                        }
                    ],
                    "matched": []
                },
                "JndiDataSourceAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType'"
                        }
                    ],
                    "matched": []
                },
                "XADataSourceAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.transaction.TransactionManager'"
                        }
                    ],
                    "matched": []
                },
                "JerseyAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.glassfish.jersey.server.spring.SpringComponentProvider'"
                        }
                    ],
                    "matched": []
                },
                "JmsAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.jms.Message'"
                        }
                    ],
                    "matched": []
                },
                "JndiConnectionFactoryAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.jms.core.JmsTemplate'"
                        }
                    ],
                    "matched": []
                },
                "ActiveMQAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.jms.ConnectionFactory'"
                        }
                    ],
                    "matched": []
                },
                "ArtemisAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.jms.ConnectionFactory'"
                        }
                    ],
                    "matched": []
                },
                "JooqAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.jooq.DSLContext'"
                        }
                    ],
                    "matched": []
                },
                "JsonbAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.json.bind.Jsonb'"
                        }
                    ],
                    "matched": []
                },
                "KafkaAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.kafka.core.KafkaTemplate'"
                        }
                    ],
                    "matched": []
                },
                "LdapAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.ldap.core.ContextSource'"
                        }
                    ],
                    "matched": []
                },
                "EmbeddedLdapAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.unboundid.ldap.listener.InMemoryDirectoryServer'"
                        }
                    ],
                    "matched": []
                },
                "LiquibaseAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'liquibase.change.DatabaseChange'"
                        }
                    ],
                    "matched": []
                },
                "MailSenderAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.mail.internet.MimeMessage'"
                        }
                    ],
                    "matched": []
                },
                "MailSenderValidatorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnSingleCandidate did not find required type 'org.springframework.mail.javamail.JavaMailSenderImpl'"
                        }
                    ],
                    "matched": []
                },
                "MongoAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.mongodb.client.MongoClient'"
                        }
                    ],
                    "matched": []
                },
                "MongoReactiveAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.mongodb.reactivestreams.client.MongoClient'"
                        }
                    ],
                    "matched": []
                },
                "MustacheAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.samskivert.mustache.Mustache'"
                        }
                    ],
                    "matched": []
                },
                "Neo4jAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.neo4j.driver.Driver'"
                        }
                    ],
                    "matched": []
                },
                "NettyAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.netty.util.NettyRuntime'"
                        }
                    ],
                    "matched": []
                },
                "HibernateJpaAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.persistence.EntityManager'"
                        }
                    ],
                    "matched": []
                },
                "PulsarAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.apache.pulsar.client.api.PulsarClient'"
                        }
                    ],
                    "matched": []
                },
                "PulsarReactiveAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.apache.pulsar.client.api.PulsarClient'"
                        }
                    ],
                    "matched": []
                },
                "QuartzAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.quartz.Scheduler'"
                        }
                    ],
                    "matched": []
                },
                "R2dbcAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.r2dbc.spi.ConnectionFactory'"
                        }
                    ],
                    "matched": []
                },
                "R2dbcTransactionManagerAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.r2dbc.connection.R2dbcTransactionManager'"
                        }
                    ],
                    "matched": []
                },
                "ReactorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Hooks'"
                        }
                    ],
                    "matched": []
                },
                "RSocketMessagingAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.rsocket.RSocket'"
                        }
                    ],
                    "matched": []
                },
                "RSocketRequesterAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.rsocket.RSocket'"
                        }
                    ],
                    "matched": []
                },
                "RSocketServerAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.rsocket.core.RSocketServer'"
                        }
                    ],
                    "matched": []
                },
                "RSocketStrategiesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.netty.buffer.PooledByteBufAllocator'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveOAuth2ClientAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Flux'"
                        }
                    ],
                    "matched": []
                },
                "OAuth2ClientAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.config.annotation.web.configuration.EnableWebSecurity'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveOAuth2ResourceServerAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.config.annotation.web.reactive.EnableWebFluxSecurity'"
                        }
                    ],
                    "matched": []
                },
                "OAuth2ResourceServerAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.oauth2.server.resource.authentication.BearerTokenAuthenticationToken'"
                        }
                    ],
                    "matched": []
                },
                "OAuth2AuthorizationServerAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.oauth2.server.authorization.OAuth2Authorization'"
                        }
                    ],
                    "matched": []
                },
                "OAuth2AuthorizationServerJwtAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.oauth2.server.authorization.OAuth2Authorization'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveSecurityAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Flux'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveUserDetailsServiceAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.authentication.ReactiveAuthenticationManager'"
                        }
                    ],
                    "matched": []
                },
                "RSocketSecurityAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.rsocket.core.SecuritySocketAcceptorInterceptor'"
                        }
                    ],
                    "matched": []
                },
                "Saml2RelyingPartyAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.saml2.provider.service.registration.RelyingPartyRegistrationRepository'"
                        }
                    ],
                    "matched": []
                },
                "SecurityAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.authentication.DefaultAuthenticationEventPublisher'"
                        }
                    ],
                    "matched": []
                },
                "SecurityFilterAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.config.http.SessionCreationPolicy'"
                        }
                    ],
                    "matched": []
                },
                "UserDetailsServiceAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.security.authentication.AuthenticationManager'"
                        }
                    ],
                    "matched": []
                },
                "SendGridAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'com.sendgrid.SendGrid'"
                        }
                    ],
                    "matched": []
                },
                "SessionAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.session.Session'"
                        }
                    ],
                    "matched": []
                },
                "DataSourceInitializationConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.jdbc.datasource.init.DatabasePopulator'"
                        }
                    ],
                    "matched": []
                },
                "R2dbcInitializationConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required classes 'io.r2dbc.spi.ConnectionFactory', 'org.springframework.r2dbc.connection.init.DatabasePopulator'"
                        }
                    ],
                    "matched": []
                },
                "TaskExecutorConfigurations.SimpleAsyncTaskExecutorBuilderConfiguration#simpleAsyncTaskExecutorBuilderVirtualThreads": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnMissingBean (types: org.springframework.boot.task.SimpleAsyncTaskExecutorBuilder; SearchStrategy: all) found beans of type 'org.springframework.boot.task.SimpleAsyncTaskExecutorBuilder' simpleAsyncTaskExecutorBuilder"
                        }
                    ],
                    "matched": []
                },
                "TaskExecutorConfigurations.TaskExecutorConfiguration#applicationTaskExecutorVirtualThreads": {
                    "notMatched": [
                        {
                            "condition": "OnThreadingCondition",
                            "message": "@ConditionalOnThreading did not find VIRTUAL"
                        }
                    ],
                    "matched": []
                },
                "TaskSchedulingAutoConfiguration#scheduledBeanLazyInitializationExcludeFilter": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (names: org.springframework.context.annotation.internalScheduledAnnotationProcessor; SearchStrategy: all) did not find any beans named org.springframework.context.annotation.internalScheduledAnnotationProcessor"
                        }
                    ],
                    "matched": []
                },
                "TaskSchedulingConfigurations.SimpleAsyncTaskSchedulerBuilderConfiguration#simpleAsyncTaskSchedulerBuilderVirtualThreads": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnMissingBean (types: org.springframework.boot.task.SimpleAsyncTaskSchedulerBuilder; SearchStrategy: all) found beans of type 'org.springframework.boot.task.SimpleAsyncTaskSchedulerBuilder' simpleAsyncTaskSchedulerBuilder"
                        }
                    ],
                    "matched": []
                },
                "TaskSchedulingConfigurations.TaskSchedulerConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (names: org.springframework.context.annotation.internalScheduledAnnotationProcessor; SearchStrategy: all) did not find any beans named org.springframework.context.annotation.internalScheduledAnnotationProcessor"
                        }
                    ],
                    "matched": []
                },
                "ThymeleafAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.thymeleaf.spring6.SpringTemplateEngine'"
                        }
                    ],
                    "matched": []
                },
                "TransactionAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.transaction.PlatformTransactionManager'"
                        }
                    ],
                    "matched": []
                },
                "TransactionManagerCustomizationAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.transaction.PlatformTransactionManager'"
                        }
                    ],
                    "matched": []
                },
                "JtaAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'jakarta.transaction.Transaction'"
                        }
                    ],
                    "matched": []
                },
                "ValidationAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnResourceCondition",
                            "message": "@ConditionalOnResource did not find resource 'classpath:META-INF/services/jakarta.validation.spi.ValidationProvider'"
                        }
                    ],
                    "matched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass found required class 'jakarta.validation.executable.ExecutableValidator'"
                        }
                    ]
                },
                "EmbeddedWebServerFactoryCustomizerAutoConfiguration.JettyWebServerFactoryCustomizerConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required classes 'org.eclipse.jetty.server.Server', 'org.eclipse.jetty.util.Loader', 'org.eclipse.jetty.ee10.webapp.WebAppContext'"
                        }
                    ],
                    "matched": []
                },
                "EmbeddedWebServerFactoryCustomizerAutoConfiguration.NettyWebServerFactoryCustomizerConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.netty.http.server.HttpServer'"
                        }
                    ],
                    "matched": []
                },
                "EmbeddedWebServerFactoryCustomizerAutoConfiguration.TomcatWebServerFactoryCustomizerConfiguration#tomcatVirtualThreadsProtocolHandlerCustomizer": {
                    "notMatched": [
                        {
                            "condition": "OnThreadingCondition",
                            "message": "@ConditionalOnThreading did not find VIRTUAL"
                        }
                    ],
                    "matched": []
                },
                "EmbeddedWebServerFactoryCustomizerAutoConfiguration.UndertowWebServerFactoryCustomizerConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required classes 'io.undertow.Undertow', 'org.xnio.SslClientAuthMode'"
                        }
                    ],
                    "matched": []
                },
                "HttpHandlerAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.reactive.DispatcherHandler'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveMultipartAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.reactive.config.WebFluxConfigurer'"
                        }
                    ],
                    "matched": []
                },
                "ReactiveWebServerFactoryAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "@ConditionalOnWebApplication did not find reactive web application classes"
                        }
                    ],
                    "matched": []
                },
                "WebFluxAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.reactive.config.WebFluxConfigurer'"
                        }
                    ],
                    "matched": []
                },
                "WebSessionIdResolverAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'reactor.core.publisher.Mono'"
                        }
                    ],
                    "matched": []
                },
                "ErrorWebFluxAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.reactive.config.WebFluxConfigurer'"
                        }
                    ],
                    "matched": []
                },
                "ClientHttpConnectorAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.reactive.function.client.WebClient'"
                        }
                    ],
                    "matched": []
                },
                "WebClientAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.reactive.function.client.WebClient'"
                        }
                    ],
                    "matched": []
                },
                "DispatcherServletAutoConfiguration.DispatcherServletConfiguration#multipartResolver": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnBean (types: org.springframework.web.multipart.MultipartResolver; SearchStrategy: all) did not find any beans of type org.springframework.web.multipart.MultipartResolver"
                        }
                    ],
                    "matched": []
                },
                "ServletWebServerFactoryAutoConfiguration.ForwardedHeaderFilterConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (server.forward-headers-strategy=framework) did not find property 'server.forward-headers-strategy'"
                        }
                    ],
                    "matched": []
                },
                "ServletWebServerFactoryConfiguration.EmbeddedJetty": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required classes 'org.eclipse.jetty.server.Server', 'org.eclipse.jetty.util.Loader', 'org.eclipse.jetty.ee10.webapp.WebAppContext'"
                        }
                    ],
                    "matched": []
                },
                "ServletWebServerFactoryConfiguration.EmbeddedUndertow": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required classes 'io.undertow.Undertow', 'org.xnio.SslClientAuthMode'"
                        }
                    ],
                    "matched": []
                },
                "WebMvcAutoConfiguration#hiddenHttpMethodFilter": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (spring.mvc.hiddenmethod.filter.enabled) did not find property 'enabled'"
                        }
                    ],
                    "matched": []
                },
                "WebMvcAutoConfiguration.ProblemDetailsErrorHandlingConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnPropertyCondition",
                            "message": "@ConditionalOnProperty (spring.mvc.problemdetails.enabled=true) did not find property 'enabled'"
                        }
                    ],
                    "matched": []
                },
                "WebMvcAutoConfiguration.WebMvcAutoConfigurationAdapter#beanNameViewResolver": {
                    "notMatched": [
                        {
                            "condition": "OnBeanCondition",
                            "message": "@ConditionalOnMissingBean (types: org.springframework.web.servlet.view.BeanNameViewResolver; SearchStrategy: all) found beans of type 'org.springframework.web.servlet.view.BeanNameViewResolver' beanNameViewResolver"
                        }
                    ],
                    "matched": []
                },
                "WebServicesAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.ws.transport.http.MessageDispatcherServlet'"
                        }
                    ],
                    "matched": []
                },
                "WebServiceTemplateAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.oxm.Marshaller'"
                        }
                    ],
                    "matched": []
                },
                "WebSocketReactiveAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnWebApplicationCondition",
                            "message": "@ConditionalOnWebApplication did not find reactive web application classes"
                        }
                    ],
                    "matched": []
                },
                "WebSocketMessagingAutoConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.springframework.web.socket.config.annotation.WebSocketMessageBrokerConfigurer'"
                        }
                    ],
                    "matched": []
                },
                "WebSocketServletAutoConfiguration.JettyWebSocketConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'org.eclipse.jetty.ee10.websocket.jakarta.server.config.JakartaWebSocketServletContainerInitializer'"
                        }
                    ],
                    "matched": []
                },
                "WebSocketServletAutoConfiguration.UndertowWebSocketConfiguration": {
                    "notMatched": [
                        {
                            "condition": "OnClassCondition",
                            "message": "@ConditionalOnClass did not find required class 'io.undertow.websockets.jsr.Bootstrap'"
                        }
                    ],
                    "matched": []
                }
            },
            "unconditionalClasses": [
                "org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration",
                "org.springframework.boot.actuate.autoconfigure.availability.AvailabilityHealthContributorAutoConfiguration",
                "org.springframework.boot.autoconfigure.ssl.SslAutoConfiguration",
                "org.springframework.boot.actuate.autoconfigure.info.InfoContributorAutoConfiguration",
                "org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration",
                "org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration",
                "org.springframework.boot.actuate.autoconfigure.metrics.integration.IntegrationMetricsAutoConfiguration",
                "org.springframework.boot.actuate.autoconfigure.endpoint.EndpointAutoConfiguration",
                "org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration",
                "org.springframework.boot.actuate.autoconfigure.health.HealthContributorAutoConfiguration",
                "org.springframework.boot.actuate.autoconfigure.endpoint.jackson.JacksonEndpointAutoConfiguration",
                "org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration",
                "org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration"
            ]
        }
    }
}
```

크롬 라이브러리를 이용해 예쁘게 출력해본 뒤 축소시켜 보면, 자동설정의 `@Conditional`에 따라 평가되어 `positiveMatches`와 `negativeMatches` 속성으로 구분된다고 한다.

## 스프링 환경변수 정보 `/env`

스프링의 환경변수 정보를 확인하는 데 사용된다.

접속 경로:

```
http://localhost:8080/actuator/env
```

기본적으로 `application.properties` 파일의 변수들이 표시되며, OS, JVM의 환경변수도 함께 표시된다.

```json
{
    "activeProfiles": [],
    "defaultProfiles": [
        "default"
    ],
    "propertySources": [
        {
            "name": "server.ports",
            "properties": {
                "local.server.port": {
                    "value": "******"
                }
            }
        },
        {
            "name": "servletContextInitParams",
            "properties": {}
        },
        {
            "name": "systemProperties",
            "properties": {
                "java.specification.version": {
                    "value": "******"
                },
                "sun.cpu.isalist": {
                    "value": "******"
                },
                "sun.jnu.encoding": {
                    "value": "******"
                },
                "java.class.path": {
                    "value": "******"
                },
                "java.vm.vendor": {
                    "value": "******"
                },
                "sun.arch.data.model": {
                    "value": "******"
                },
                "user.variant": {
                    "value": "******"
                },
                "java.vendor.url": {
                    "value": "******"
                },
                "catalina.useNaming": {
                    "value": "******"
                },
                "user.timezone": {
                    "value": "******"
                },
                "java.vm.specification.version": {
                    "value": "******"
                },
                "os.name": {
                    "value": "******"
                },
                "sun.java.launcher": {
                    "value": "******"
                },
                "user.country": {
                    "value": "******"
                },
                "sun.boot.library.path": {
                    "value": "******"
                },
                "spring.application.admin.enabled": {
                    "value": "******"
                },
                "sun.java.command": {
                    "value": "******"
                },
                "com.sun.management.jmxremote": {
                    "value": "******"
                },
                "jdk.debug": {
                    "value": "******"
                },
                "spring.liveBeansView.mbeanDomain": {
                    "value": "******"
                },
                "sun.cpu.endian": {
                    "value": "******"
                },
                "user.home": {
                    "value": "******"
                },
                "user.language": {
                    "value": "******"
                },
                "java.specification.vendor": {
                    "value": "******"
                },
                "java.version.date": {
                    "value": "******"
                },
                "java.home": {
                    "value": "******"
                },
                "spring.output.ansi.enabled": {
                    "value": "******"
                },
                "file.separator": {
                    "value": "******"
                },
                "java.vm.compressedOopsMode": {
                    "value": "******"
                },
                "line.separator": {
                    "value": "******"
                },
                "java.vm.specification.vendor": {
                    "value": "******"
                },
                "java.specification.name": {
                    "value": "******"
                },
                "FILE_LOG_CHARSET": {
                    "value": "******"
                },
                "java.awt.headless": {
                    "value": "******"
                },
                "user.script": {
                    "value": "******"
                },
                "sun.management.compiler": {
                    "value": "******"
                },
                "java.runtime.version": {
                    "value": "******"
                },
                "user.name": {
                    "value": "******"
                },
                "spring.jmx.enabled": {
                    "value": "******"
                },
                "path.separator": {
                    "value": "******"
                },
                "management.endpoints.jmx.exposure.include": {
                    "value": "******"
                },
                "os.version": {
                    "value": "******"
                },
                "java.runtime.name": {
                    "value": "******"
                },
                "file.encoding": {
                    "value": "******"
                },
                "java.vm.name": {
                    "value": "******"
                },
                "java.vendor.url.bug": {
                    "value": "******"
                },
                "java.io.tmpdir": {
                    "value": "******"
                },
                "catalina.home": {
                    "value": "******"
                },
                "java.version": {
                    "value": "******"
                },
                "user.dir": {
                    "value": "******"
                },
                "os.arch": {
                    "value": "******"
                },
                "java.vm.specification.name": {
                    "value": "******"
                },
                "PID": {
                    "value": "******"
                },
                "sun.os.patch.level": {
                    "value": "******"
                },
                "CONSOLE_LOG_CHARSET": {
                    "value": "******"
                },
                "catalina.base": {
                    "value": "******"
                },
                "native.encoding": {
                    "value": "******"
                },
                "java.library.path": {
                    "value": "******"
                },
                "java.vm.info": {
                    "value": "******"
                },
                "java.vendor": {
                    "value": "******"
                },
                "java.vm.version": {
                    "value": "******"
                },
                "java.rmi.server.randomIDs": {
                    "value": "******"
                },
                "sun.io.unicode.encoding": {
                    "value": "******"
                },
                "java.class.version": {
                    "value": "******"
                },
                "LOGGED_APPLICATION_NAME": {
                    "value": "******"
                }
            }
        },
        {
            "name": "systemEnvironment",
            "properties": {
                "USERDOMAIN_ROAMINGPROFILE": {
                    "value": "******",
                    "origin": "System Environment Property \"USERDOMAIN_ROAMINGPROFILE\""
                },
                "LOCALAPPDATA": {
                    "value": "******",
                    "origin": "System Environment Property \"LOCALAPPDATA\""
                },
                "PROCESSOR_LEVEL": {
                    "value": "******",
                    "origin": "System Environment Property \"PROCESSOR_LEVEL\""
                },
                "USERDOMAIN": {
                    "value": "******",
                    "origin": "System Environment Property \"USERDOMAIN\""
                },
                "FPS_BROWSER_APP_PROFILE_STRING": {
                    "value": "******",
                    "origin": "System Environment Property \"FPS_BROWSER_APP_PROFILE_STRING\""
                },
                "LOGONSERVER": {
                    "value": "******",
                    "origin": "System Environment Property \"LOGONSERVER\""
                },
                "JAVA_HOME": {
                    "value": "******",
                    "origin": "System Environment Property \"JAVA_HOME\""
                },
                "SESSIONNAME": {
                    "value": "******",
                    "origin": "System Environment Property \"SESSIONNAME\""
                },
                "ALLUSERSPROFILE": {
                    "value": "******",
                    "origin": "System Environment Property \"ALLUSERSPROFILE\""
                },
                "PROCESSOR_ARCHITECTURE": {
                    "value": "******",
                    "origin": "System Environment Property \"PROCESSOR_ARCHITECTURE\""
                },
                "PSModulePath": {
                    "value": "******",
                    "origin": "System Environment Property \"PSModulePath\""
                },
                "SystemDrive": {
                    "value": "******",
                    "origin": "System Environment Property \"SystemDrive\""
                },
                "OneDrive": {
                    "value": "******",
                    "origin": "System Environment Property \"OneDrive\""
                },
                "APPDATA": {
                    "value": "******",
                    "origin": "System Environment Property \"APPDATA\""
                },
                "USERNAME": {
                    "value": "******",
                    "origin": "System Environment Property \"USERNAME\""
                },
                "ProgramFiles(x86)": {
                    "value": "******",
                    "origin": "System Environment Property \"ProgramFiles(x86)\""
                },
                "CommonProgramFiles": {
                    "value": "******",
                    "origin": "System Environment Property \"CommonProgramFiles\""
                },
                "Path": {
                    "value": "******",
                    "origin": "System Environment Property \"Path\""
                },
                "FPS_BROWSER_USER_PROFILE_STRING": {
                    "value": "******",
                    "origin": "System Environment Property \"FPS_BROWSER_USER_PROFILE_STRING\""
                },
                "PATHEXT": {
                    "value": "******",
                    "origin": "System Environment Property \"PATHEXT\""
                },
                "DriverData": {
                    "value": "******",
                    "origin": "System Environment Property \"DriverData\""
                },
                "OS": {
                    "value": "******",
                    "origin": "System Environment Property \"OS\""
                },
                "COMPUTERNAME": {
                    "value": "******",
                    "origin": "System Environment Property \"COMPUTERNAME\""
                },
                "PROCESSOR_REVISION": {
                    "value": "******",
                    "origin": "System Environment Property \"PROCESSOR_REVISION\""
                },
                "CLASSPATH": {
                    "value": "******",
                    "origin": "System Environment Property \"CLASSPATH\""
                },
                "CommonProgramW6432": {
                    "value": "******",
                    "origin": "System Environment Property \"CommonProgramW6432\""
                },
                "ComSpec": {
                    "value": "******",
                    "origin": "System Environment Property \"ComSpec\""
                },
                "ProgramData": {
                    "value": "******",
                    "origin": "System Environment Property \"ProgramData\""
                },
                "ProgramW6432": {
                    "value": "******",
                    "origin": "System Environment Property \"ProgramW6432\""
                },
                "EFC_29520": {
                    "value": "******",
                    "origin": "System Environment Property \"EFC_29520\""
                },
                "HOMEPATH": {
                    "value": "******",
                    "origin": "System Environment Property \"HOMEPATH\""
                },
                "SystemRoot": {
                    "value": "******",
                    "origin": "System Environment Property \"SystemRoot\""
                },
                "TEMP": {
                    "value": "******",
                    "origin": "System Environment Property \"TEMP\""
                },
                "HOMEDRIVE": {
                    "value": "******",
                    "origin": "System Environment Property \"HOMEDRIVE\""
                },
                "PROCESSOR_IDENTIFIER": {
                    "value": "******",
                    "origin": "System Environment Property \"PROCESSOR_IDENTIFIER\""
                },
                "USERPROFILE": {
                    "value": "******",
                    "origin": "System Environment Property \"USERPROFILE\""
                },
                "TMP": {
                    "value": "******",
                    "origin": "System Environment Property \"TMP\""
                },
                "CommonProgramFiles(x86)": {
                    "value": "******",
                    "origin": "System Environment Property \"CommonProgramFiles(x86)\""
                },
                "ProgramFiles": {
                    "value": "******",
                    "origin": "System Environment Property \"ProgramFiles\""
                },
                "PUBLIC": {
                    "value": "******",
                    "origin": "System Environment Property \"PUBLIC\""
                },
                "NUMBER_OF_PROCESSORS": {
                    "value": "******",
                    "origin": "System Environment Property \"NUMBER_OF_PROCESSORS\""
                },
                "windir": {
                    "value": "******",
                    "origin": "System Environment Property \"windir\""
                },
                "=::": {
                    "value": "******",
                    "origin": "System Environment Property \"=::\""
                },
                "IDEA_INITIAL_DIRECTORY": {
                    "value": "******",
                    "origin": "System Environment Property \"IDEA_INITIAL_DIRECTORY\""
                }
            }
        },
        {
            "name": "Config resource 'class path resource [application.properties]' via location 'optional:classpath:/'",
            "properties": {
                "spring.application.name": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 1:25"
                },
                "spring.mvc.pathmatch.matching-strategy": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 3:40"
                },
                "management.endpoints.web.base-path": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 4:36"
                },
                "management.endpoint.shutdown.enabled": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 7:38"
                },
                "management.endpoint.cached.enabled": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 8:36"
                },
                "management.endpoint.health.show-details": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 10:41"
                },
                "management.endpoints.web.exposure.include": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 13:43"
                },
                "management.endpoints.web.exposure.exclude": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 14:43"
                },
                "management.endpoints.jmx.exposure.include": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 17:43"
                },
                "management.endpoints.jmx.exposure.exclude": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 18:43"
                },
                "info.organization.name": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 21:24"
                },
                "info.contact.email": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 22:20"
                },
                "info.contact.phoneNumber": {
                    "value": "******",
                    "origin": "class path resource [application.properties] - 23:26"
                }
            }
        }
    ]
}
```

## 로깅 레벨 확인 `/loggers`

애플리케이션의 로깅 레벨 수준이 어떻게 되어있는지 확인할 수 있다. 출력 결과가 매우 많을 수 있다.

접속 경로:

```
http://localhost:8080/actuator/loggers
```

```json
{
    "levels": [
        "OFF",
        "ERROR",
        "WARN",
        "INFO",
        "DEBUG",
        "TRACE"
    ],
    "loggers": {
        "ROOT": {
            "configuredLevel": "INFO",
            "effectiveLevel": "INFO"
        },
        "_org": {
            "effectiveLevel": "INFO"
        },
        "_org.springframework": {
            "effectiveLevel": "INFO"
        },
        "_org.springframework.web": {
            "effectiveLevel": "INFO"
        },
        "_org.springframework.web.servlet": {
            "effectiveLevel": "INFO"
        },
        "_org.springframework.web.servlet.HandlerMapping": {
            "effectiveLevel": "INFO"
        },
        "_org.springframework.web.servlet.HandlerMapping.Mappings": {
            "effectiveLevel": "INFO"
        },
        "com": {
            "effectiveLevel": "INFO"
        },
        "com.springboot": {
            "effectiveLevel": "INFO"
        },
        "com.springboot.actuator": {
            "effectiveLevel": "INFO"
        },
        "com.springboot.actuator.ActuatorApplication": {
            "effectiveLevel": "INFO"
        },
        "io": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.common": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.common.util": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.common.util.internal": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.common.util.internal.logging": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.common.util.internal.logging.InternalLoggerFactory": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.core": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.core.instrument": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.core.instrument.AbstractTimer": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.core.instrument.binder": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.core.instrument.binder.jvm": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.core.instrument.binder.jvm.ExecutorServiceMetrics": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.core.instrument.binder.jvm.JvmGcMetrics": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.core.instrument.internal": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.core.instrument.internal.DefaultGauge": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.observation": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.observation.SimpleObservation": {
            "effectiveLevel": "INFO"
        },
        "io.micrometer.observation.SimpleObservation$SimpleScope": {
            "effectiveLevel": "INFO"
        },
        "io.swagger": {
            "effectiveLevel": "INFO"
        },
        "io.swagger.v3": {
            "effectiveLevel": "INFO"
        },
        "io.swagger.v3.core": {
            "effectiveLevel": "INFO"
        },
        "io.swagger.v3.core.converter": {
            "effectiveLevel": "INFO"
        },
        "io.swagger.v3.core.converter.ModelConverters": {
            "effectiveLevel": "INFO"
        },
        "io.swagger.v3.core.jackson": {
            "effectiveLevel": "INFO"
        },
        "io.swagger.v3.core.jackson.ModelResolver": {
            "effectiveLevel": "INFO"
        },
        "org": {
            "effectiveLevel": "INFO"
        },
        "org.apache": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina.core": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina.core.ContainerBase": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina.core.ContainerBase.[Tomcat]": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina.core.ContainerBase.[Tomcat].[localhost]": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina.core.ContainerBase.[Tomcat].[localhost].[/]": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina.core.StandardEngine": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina.core.StandardService": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina.startup": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina.startup.DigesterFactory": {
            "configuredLevel": "ERROR",
            "effectiveLevel": "ERROR"
        },
        "org.apache.catalina.util": {
            "effectiveLevel": "INFO"
        },
        "org.apache.catalina.util.LifecycleBase": {
            "configuredLevel": "ERROR",
            "effectiveLevel": "ERROR"
        },
        "org.apache.coyote": {
            "effectiveLevel": "INFO"
        },
        "org.apache.coyote.http11": {
            "effectiveLevel": "INFO"
        },
        "org.apache.coyote.http11.Http11NioProtocol": {
            "configuredLevel": "WARN",
            "effectiveLevel": "WARN"
        },
        "org.apache.sshd": {
            "effectiveLevel": "INFO"
        },
        "org.apache.sshd.common": {
            "effectiveLevel": "INFO"
        },
        "org.apache.sshd.common.util": {
            "effectiveLevel": "INFO"
        },
        "org.apache.sshd.common.util.SecurityUtils": {
            "configuredLevel": "WARN",
            "effectiveLevel": "WARN"
        },
        "org.apache.tomcat": {
            "effectiveLevel": "INFO"
        },
        "org.apache.tomcat.util": {
            "effectiveLevel": "INFO"
        },
        "org.apache.tomcat.util.net": {
            "effectiveLevel": "INFO"
        },
        "org.apache.tomcat.util.net.NioSelectorPool": {
            "configuredLevel": "WARN",
            "effectiveLevel": "WARN"
        },
        "org.eclipse": {
            "effectiveLevel": "INFO"
        },
        "org.eclipse.jetty": {
            "effectiveLevel": "INFO"
        },
        "org.eclipse.jetty.util": {
            "effectiveLevel": "INFO"
        },
        "org.eclipse.jetty.util.component": {
            "effectiveLevel": "INFO"
        },
        "org.eclipse.jetty.util.component.AbstractLifeCycle": {
            "configuredLevel": "ERROR",
            "effectiveLevel": "ERROR"
        },
        "org.hibernate": {
            "effectiveLevel": "INFO"
        },
        "org.hibernate.validator": {
            "effectiveLevel": "INFO"
        },
        "org.hibernate.validator.internal": {
            "effectiveLevel": "INFO"
        },
        "org.hibernate.validator.internal.util": {
            "effectiveLevel": "INFO"
        },
        "org.hibernate.validator.internal.util.Version": {
            "configuredLevel": "WARN",
            "effectiveLevel": "WARN"
        },
        "org.springdoc": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.api": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.api.AbstractOpenApiResource": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.conditions": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.conditions.CacheOrGroupedOpenApiCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.conditions.MultipleOpenApiGroupsCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.conditions.MultipleOpenApiSupportCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.converters": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.converters.AdditionalModelsConverter": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.converters.ModelConverterRegistrar": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.providers": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.providers.WebConversionServiceProvider": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.service": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.service.GenericParameterService": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.service.GenericResponseService": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.service.OpenAPIService": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.utils": {
            "effectiveLevel": "INFO"
        },
        "org.springdoc.core.utils.PropertyResolverUtils": {
            "effectiveLevel": "INFO"
        },
        "org.springframework": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.aop": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.aop.framework": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.aop.framework.autoproxy": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.aop.framework.autoproxy.BeanFactoryAdvisorRetrievalHelper": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.aop.framework.autoproxy.InfrastructureAdvisorAutoProxyCreator": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.AbstractNestablePropertyAccessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.CachedIntrospectionResults": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.TypeConverterDelegate": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory.annotation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory.parsing": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory.parsing.FailFastProblemReporter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory.support": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory.support.DefaultListableBeanFactory": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory.support.DisposableBeanAdapter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory.xml": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory.xml.DefaultDocumentLoader": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.beans.factory.xml.XmlBeanDefinitionReader": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.BeanDefinitionLoader": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.BeanDefinitionLoader$ClassExcludeFilter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.DefaultApplicationArguments": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.DefaultApplicationArguments$Source": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.SpringApplication": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.SpringApplicationShutdownHook": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.availability": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.availability.AvailabilityProbesAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.availability.AvailabilityProbesAutoConfiguration$ProbesCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.endpoint": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.endpoint.condition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.endpoint.condition.OnAvailableEndpointCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.health": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.health.OnEnabledHealthIndicatorCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.info": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.info.OnEnabledInfoContributorCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.logging": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.logging.LogFileWebEndpointAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.logging.LogFileWebEndpointAutoConfiguration$LogFileCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.logging.LoggersEndpointAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.logging.LoggersEndpointAutoConfiguration$OnEnabledLoggingSystemCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.metrics": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.metrics.CompositeMeterRegistryConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.metrics.CompositeMeterRegistryConfiguration$MultipleNonPrimaryMeterRegistriesCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.metrics.LogbackMetricsAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.metrics.LogbackMetricsAutoConfiguration$LogbackLoggingCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.metrics.MeterRegistryCustomizer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.metrics.MeterRegistryPostProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.metrics.OnlyOnceLoggingDenyMeterFilter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.metrics.export": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.metrics.export.OnMetricsExportEnabledCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.observation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.observation.ObservationRegistryConfigurer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.observation.ObservationRegistryCustomizer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.startup": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.startup.StartupEndpointAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.startup.StartupEndpointAutoConfiguration$ApplicationStartupCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.web": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.web.server": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration$SameManagementContextConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.web.server.ManagementContextAutoConfiguration$SameManagementContextConfiguration$1": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.autoconfigure.web.server.OnManagementPortCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.EndpointFilter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.EndpointId": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.annotation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.annotation.EndpointDiscoverer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.jmx": {
            "configuredLevel": "WARN",
            "effectiveLevel": "WARN"
        },
        "org.springframework.boot.actuate.endpoint.jmx.JmxEndpointExporter": {
            "effectiveLevel": "WARN"
        },
        "org.springframework.boot.actuate.endpoint.web": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.web.EndpointLinksResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.web.ServletEndpointRegistrar": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.web.servlet": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.web.servlet.AdditionalHealthEndpointPathsWebMvcHandlerMapping": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.web.servlet.ControllerEndpointHandlerMapping": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.endpoint.web.servlet.WebMvcEndpointHandlerMapping": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.health": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.health.HealthEndpointSupport": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.health.PingHealthIndicator": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.system": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.actuate.system.DiskSpaceHealthIndicator": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.admin": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrar": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.admin.SpringApplicationAdminMXBeanRegistrar$SpringApplicationAdmin": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.AutoConfigurationImportSelector": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.AutoConfigurationPackages": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.cache": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.cache.CacheCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.condition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.condition.OnBeanCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.condition.OnClassCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.condition.OnCloudPlatformCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.condition.OnExpressionCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.condition.OnPropertyCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.condition.OnResourceCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.condition.OnThreadingCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.condition.OnWarDeploymentCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.condition.OnWebApplicationCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.context": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration$ResourceBundleCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.http": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration$NotReactiveWebApplicationCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.info": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration$GitResourceAvailableCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.logging": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.logging.ConditionEvaluationReportLogger": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.security": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.security.DefaultWebSecurityCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.sql": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.sql.init": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.sql.init.SqlInitializationAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.sql.init.SqlInitializationAutoConfiguration$SqlInitializationModeCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.ssl": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.ssl.FileWatcher": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.OnEnabledResourceChainCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.client": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.client.NotReactiveWebApplicationCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.servlet": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration$DefaultDispatcherServletCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration$DispatcherServletRegistrationCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.servlet.WelcomePageHandlerMapping": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.servlet.WelcomePageNotAcceptableHandlerMapping": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.servlet.error": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$ErrorTemplateMissingCondition": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration$StaticView": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.availability": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.availability.ApplicationAvailabilityBean": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.cloud": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.cloud.CloudFoundryVcapEnvironmentPostProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.FileEncodingApplicationListener": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.config": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.config.ConfigDataEnvironment": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.config.ConfigDataEnvironmentContributors": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.config.ConfigDataEnvironmentPostProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.config.ConfigDataImporter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.config.ConfigDataLoaders": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.config.StandardConfigDataLocationResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.logging": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.logging.LoggingApplicationListener": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.properties": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.properties.PropertySourcesDeducer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.properties.bind": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.properties.bind.ValueObjectBinder": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.properties.source": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.properties.source.ConfigurationPropertySourcesPropertyResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.properties.source.ConfigurationPropertySourcesPropertyResolver$DefaultResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.context.properties.source.ConfigurationPropertySourcesPropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.env": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.env.OriginTrackedMapPropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.env.RandomValuePropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.env.RandomValuePropertySourceEnvironmentPostProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.env.SystemEnvironmentPropertySourceEnvironmentPostProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.env.SystemEnvironmentPropertySourceEnvironmentPostProcessor$OriginAwareSystemEnvironmentPropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.jackson": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.jackson.JsonMixinModuleEntries": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.jackson.JsonMixinModuleEntries$JsonMixinComponentScanner": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.ssl": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.ssl.DefaultSslBundleRegistry": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.embedded": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.embedded.tomcat": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.embedded.tomcat.TomcatEmbeddedWebappClassLoader": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.embedded.tomcat.TomcatProtocolHandlerCustomizer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.embedded.tomcat.TomcatServletWebServerFactory": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.embedded.tomcat.TomcatStarter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.embedded.tomcat.TomcatWebServer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.server": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.server.WebServerFactoryCustomizer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.server.WebServerFactoryCustomizerBeanPostProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet.RegistrationBean": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet.ServletContextInitializerBeans": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet.context": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet.context.AnnotationConfigServletWebServerApplicationContext": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet.context.ApplicationServletEnvironment": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet.filter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet.filter.OrderedCharacterEncodingFilter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet.filter.OrderedFormContentFilter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.boot.web.servlet.filter.OrderedRequestContextFilter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation.AnnotationBeanNameGenerator": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation.ClassPathBeanDefinitionScanner": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation.CommonAnnotationBeanPostProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation.ComponentScanAnnotationParser": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation.ComponentScanAnnotationParser$1": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation.ConfigurationClassBeanDefinitionReader": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation.ConfigurationClassEnhancer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation.ConfigurationClassParser": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation.ConfigurationClassPostProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.annotation.ConfigurationClassUtils": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.event": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.event.EventListenerMethodProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.index": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.index.CandidateComponentsIndexLoader": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.support": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.support.ApplicationListenerDetector": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.support.DefaultLifecycleProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.support.DelegatingMessageSource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.support.PostProcessorRegistrationDelegate": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.support.PostProcessorRegistrationDelegate$BeanPostProcessorChecker": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.support.PropertySourcesPlaceholderConfigurer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.context.support.PropertySourcesPlaceholderConfigurer$1": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.annotation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.annotation.AnnotationTypeMapping": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.env": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.env.MapPropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.env.PropertiesPropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.env.PropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.env.PropertySource$ComparisonPropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.env.PropertySource$StubPropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.env.PropertySourcesPropertyResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.env.StandardEnvironment": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.env.SystemEnvironmentPropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.io": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.io.support": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.io.support.PathMatchingResourcePatternResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.io.support.PropertySourceProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.io.support.ResourceArrayPropertyEditor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.io.support.SpringFactoriesLoader": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.task": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.task.SimpleAsyncTaskExecutor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.task.SimpleAsyncTaskExecutor$ConcurrencyThrottleAdapter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.type": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.type.filter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.type.filter.AnnotationTypeFilter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.core.type.filter.AssignableTypeFilter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.http": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.http.converter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.http.converter.ByteArrayHttpMessageConverter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.http.converter.ResourceHttpMessageConverter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.http.converter.ResourceRegionHttpMessageConverter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.http.converter.StringHttpMessageConverter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.http.converter.json": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.http.converter.json.MappingJackson2HttpMessageConverter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.http.converter.xml": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.http.converter.xml.Jaxb2RootElementHttpMessageConverter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jmx": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jmx.export": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jmx.export.annotation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jmx.export.annotation.AnnotationMBeanExporter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jmx.export.naming": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jmx.export.naming.KeyNamingStrategy": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jmx.support": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jmx.support.JmxUtils": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jmx.support.MBeanServerFactoryBean": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jndi": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jndi.JndiTemplate": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jndi.support": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.jndi.support.SimpleJndiBeanFactory": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.scheduling": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.scheduling.concurrent": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.ui": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.ui.context": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.ui.context.support": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.ui.context.support.ResourceBundleThemeSource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.ui.context.support.UiApplicationContextUtils": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.util": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.util.PropertyPlaceholderHelper": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.validation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.validation.DataBinder": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.validation.beanvalidation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.validation.beanvalidation.OptionalValidatorFactoryBean": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.HttpLogging": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.context": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.context.request": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.context.request.async": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.context.request.async.WebAsyncManager": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.context.support": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.context.support.ServletContextPropertySource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.context.support.ServletContextResourcePatternResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.context.support.StandardServletEnvironment": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.cors": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.cors.DefaultCorsProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.filter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.filter.ServerHttpObservationFilter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.method": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.method.HandlerMethod": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.method.annotation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.method.annotation.ModelFactory": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.DispatcherServlet": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.HandlerExecutionChain": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.PageNotFound": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.config": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.config.annotation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.config.annotation.WebMvcConfigurer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.function": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.function.support": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.function.support.HandlerFunctionAdapter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.function.support.RouterFunctionMapping": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.handler": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.handler.SimpleUrlHandlerMapping": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.annotation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.annotation.ResponseStatusExceptionResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method.annotation": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method.annotation.DisconnectedClient": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method.annotation.ExceptionHandlerExceptionResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method.annotation.HttpEntityMethodProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method.annotation.ReactiveTypeHandler": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method.annotation.RequestPartMethodArgumentResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.method.annotation.ServletModelAttributeMethodProcessor": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.support": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.mvc.support.DefaultHandlerExceptionResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.resource": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.resource.CachingResourceResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.resource.CachingResourceTransformer": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.resource.PathResourceResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.resource.ResourceHttpRequestHandler": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.resource.ResourceUrlProvider": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.resource.WebJarsResourceResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.support": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.support.SessionFlashMapManager": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.view": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.view.BeanNameViewResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.view.ContentNegotiatingViewResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.servlet.view.InternalResourceViewResolver": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.util": {
            "effectiveLevel": "INFO"
        },
        "org.springframework.web.util.UrlPathHelper": {
            "effectiveLevel": "INFO"
        }
    },
    "groups": {
        "web": {
            "members": [
                "org.springframework.core.codec",
                "org.springframework.http",
                "org.springframework.web",
                "org.springframework.boot.actuate.endpoint.web",
                "org.springframework.boot.web.servlet.ServletContextInitializerBeans"
            ]
        },
        "sql": {
            "members": [
                "org.springframework.jdbc.core",
                "org.hibernate.SQL",
                "org.jooq.tools.LoggerListener"
            ]
        }
    }
}
```

post 형식으로 호출하면
로깅 레벨을 변경하는 것도 가능하다.

# 액추에이터 커스텀 기능 만들기

기본 제공 외에 개발자의 요구사항에 맞춘 커스텀 기능 설정도 할 수 있다. 커스텀 기능을 개발하는 방식에는 크게 두 가지가 있다.

첫 번째는 기존 기능에 내용을 추가하는 방식이고, 두 번째는 새로운 엔드포인트를 개발하는 방식이다.

## 정보 제공 인터페이스 구현체 생성

`application` 설정파일에 내용을 추가하는 방법은 스프링 부트 2.6 버전부터는 먹히지 않는다. 다른 방법으로, 커스텀 기능을 설정할 때 별도의 구현체 클래스를 작성해서 내용을 추가하는 방법을 사용할 수 있다.

액추에이터에서는 `InfoContributor` 인터페이스를 제공하고 있는데, 이 인터페이스를 구현하는 클래스를 생성하면 된다.

### CustomInfoContributor.java

파일 생성 경로: `config/actuator/`

```java
@Component
public class CustomInfoContributor implements InfoContributor {

    @Override
    public void contribute(Info.Builder builder) {
        Map<String, Object> content = new HashMap<>();
        content.put("code-info", "InfoContributor 구현체에서 정의한 정보입니다.");
        builder.withDetail("custom-info-contributor", content);
    }
}
```

새로 생성한 `CustomInfoContributor` 클래스를 `InfoContributor` 인터페이스의 구현체로 설정하면 `contribute()` 메서드를 오버라이드 할 수 있다. 이 메서드에서 파라미터로 받는 `Builder` 객체는 액추에이터 패키지의 `Info` 클래스 안에 정의돼 있는 클래스로서 `Info` 엔드포인트에서 보여줄 내용을 담는 역할을 수행한다.

이제 애플리케이션을 재가동하고 `http://localhost:8080/actuator/info` 경로로 들어가보면 위에서 생성한 클래스의 정보가 출력되는 것을 확인할 수 있다.

```json
{
    "custom-info-contributor": {
        "code-info": "InfoContributor 구현체에서 정의한 정보입니다."
    }
}
```

## 커스텀 엔드포인트 생성

`@Endpoint` 어노테이션으로 빈에 추가된 객체들은 `@ReadOperation`, `@WriteOperation`, `@DeleteOperation` 어노테이션을 사용하여 JMX나 HTTP를 통해 커스텀 엔드포인트를 노출시킬 수 있다.

만약, JMX에서만 사용하거나 HTTP에서만 사용하는 것으로 제한하고 싶다면 `@JmxEndpoint`, `@WebEndpoint` 어노테이션을 사용하면 된다.

### NoteEndpoint

애플리케이션에 메모 기록을 남길 수 있는 기능을 엔드포인트로 생성하는 예시를 소개한다.

파일 생성 경로: `config/actuator/NoteEndpoint.java`

```java
@Component
@Endpoint(id = "note")
public class NoteEndpoint {
    private Map<String, Object> noteContent = new HashMap<>();

    @ReadOperation
    public Map<String, Object> getNote() {
        return noteContent;
    }

    @WriteOperation
    public Map<String, Object> writeNote(String key, Object value) {
        noteContent.put(key, value);
        return noteContent;
    }

    @DeleteOperation
    public Map<String, Object> deleteNote(String key) {
        noteContent.remove(key);
        return noteContent;
    }
}
```

`@Endpoint` 어노테이션을 붙여주면 액추에이터에 엔드포인트로 자동으로 등록된다. 추가할 수 있는 속성값으로는 `id`와 `enableByDefault`가 있는데, `id`는 해당 엔드포인트의 경로를 지정하는 속성이고, `enableByDefault`는 해당 엔드포인트의 기본 활성화 여부를 설정하는 속성이다. 기본값은 `true`이다.

엔드포인트를 설정하는 클래스에는 `@ReadOperation`, `@WriteOperation`, `@DeleteOperation` 어노테이션을 사용해 각 동작 메서드를 생성할 수 있다.

- `@ReadOperation`은 HTTP의 GET 메서드에 반응하는 어노테이션이고,
- `@WriteOperation`은 HTTP의 POST 메서드에 반응하는 어노테이션이며,
- `@DeleteOperation`은 HTTP의 DELETE 메서드에 반응하는 어노테이션이다.

### 어플리케이션 재가동 후 테스트

1. **Read 확인**

```
http://localhost:8080/actuator/note
```

2. Write 확인
POST방식으로 요청을 보내,
json 데이터 형식으로 보내며,
메서드의 파라미터로 key, value 변수를 정의했기 때문에
네이밍을 지켜준다.

```
{
    "key" : "안녕",
    "value" : "heart"
}
```

```json
{
    "안녕": "heart"
}
```

3. Read 확인
GET 요청을 보내,
ReadOperation을 확인해보면 Write로 보낸 값이 잘 나타난다.

```json
{
    "안녕": "heart"
}
```

4. Delete 확인
Query 파라미터에
key = 입력했던값 형태로 값을 넣어 호출한다.

실행 결과로
key가 삭제되어 빈 객체만 보여진다.

```json
{}
```
