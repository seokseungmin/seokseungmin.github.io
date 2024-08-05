---
title: "스프링 핵심가이드 8주차"
excerpt: "[제로베이스] 스프링 핵심가이드"

categories:
  - Spring
tags:
  - [Spring]

permalink: /categories/Spring8Week/ # url

toc: true
toc_sticky: true

date: 2024-08-05
last_modified_at: 2024-08-05
---

[스프링부트 핵심가이드 프로젝트 따라하면서 짠 코드](https://github.com/seokseungmin/actuator)<br>

본 게시글은 '스프링 부트 핵심 가이드' 책의 내용을 정리한 것입니다.<br>
저자 : 장정우<br>
출판사 : 위키북스<br>

# 13장 서비스의 인증과 권한 부여

애플리케이션을 개발하다 보면 인증과 인가 등의 보안 기능을 추가해야 할 때가 있습니다.
이번 장에서는 보안과 관련된 용어와 개념을 알아보고 스프링에 보안을 적용할 때 사용하는 스프링 시큐리티(Spring Security)에 대해 알아보겠습니다.
지금까지 실습한 애플리케이션은 화면이 없는 무상태 REST 애플리케이션이기 때문에 이번 장에서는 로그인을 통한 일반적인
인증과 인가 방식이 아닌 매 요청마다 토근값을 활용하는 보안 기법을 알아보겠습니다.

# 보안 용어 이해
스프링 시큐리티를 활용하려면 먼저 보안과 관련된 용어를 알아두는 것이 중요합니다.
그러므로 스프링 시큐리티를 배우기에 앞서 보안과 관련된 용어를 간단하게 설명하겠습니다.

### 인증
인증(authentication)은 사용자가 누구인지 확인하는 단계를 의미합니다.
인증의 대표적인 예로 '로그인'이 있습니다. 로그인은 데이터베이스에 등록된 아이디와 패스워드를 사용자가 입력한 아이디와
비밀번호와 비교해서 일치 여부를 확인하는 과정입니다.
로그인에 성공하면 애플리케이션 서버는 응답으로 사용자에게 토큰(token)을 전달합니다.
로그인에 실패한 사용자는 토큰을 전달받지 못해 원하는 리소스에 접근 할 수 없게 됩니다.

### 인가
인가(authorization)는앞에서 설명한 인증을 통해 검증된 사용자가 애플리케이션 내부의 리소스에 접근할 때
사용자가 해당 리소스에 접근할 권리가 있는지를 확인하는 과정을 의미합니다.
예를 들어, 로그인한 사용자가 특정 게시판에 접근해서 글을 보려고 하는 경우 게시판 접근 등급을 확인해 접근을 허가하거나 거부하는 것이
대표적인 인가의 사례입니다.

일반적으로 사용자가 인증 단계에서 발급받은 토근은 인가 내용을 포함하고 있으며, 사용자가 리소스에 접근하면서
토근을 함께 전달하면 애플리케이션 서버는 토근을 통해 권한 유무등을 확인해 인가를 수행합니다.

# 접근 주체
접근 주체(principal)는 말 그대로 애플리케이션의 기능을 사용하는 주체를 의미합니다.
접근 주체는 사용자가 될 수도 있고, 디바이스, 시스템 등이 될 수도 있습니다.
애플리케이션은 앞서 소개한 인증 과정을 통해 접근 주체가 신뢰할 수 있는지 확인하고, 인가 과정을 통해 접근 주체에게
부여된 권한을 확인하는 과정 등을 거칩니다.


# 스프링 시큐리티의 동작 구조
스프링 시큐리티는 서블릿 필터(Servlet Filter)를 기반으로 동작하며
DispatcherServlet 앞에 필터가 배치돼 있습니다.

![new repo](/assets/images/posts_img/Spring/ServletFilter.png)

필터체인(FilterChain)은 서블릿 컨테이너에서 관리하는 ApplicationFilterChain을 의미합니다.
클라이언트에서 애플리케이션으로 요청을 보내면 서블릿 컨테이너는 URI를 확인해서 필터와 서블릿을 매핑합니다.
스프링 시큐리티는 사용하고자 하는 필터체인을 서블릿 컨테이너의 필터 사이에서 동작시키기 위해 DelegatingFilterProxy를 사용합니다.

![new repo](/assets/images/posts_img/Spring/FilterChainProxy.png)

DelegatingFilterProxy는 서블릿 컨테이너의 생명주기와 스프링 애플리케이션 컨텍스트(Application Context) 사이에서
다리 역할을 수행하는 필터 구현체입니다.
표준 서블릿 필터를 구현하고 있으며, 역할을 위임할 필터체인 프록시(FilterChainProxy)를 내부에 가지고 있습니다.
필터체인 프록시는 스프링 부트의 자동 설정에 의해 자동 생성됩니다.

필터체인 프록시는 스프링 시큐리티에서 제공하는 필터로서 보안 필터체인(SecurityFilterChain)을 통해 많은
보안 필터(Security Filter)를 사용할 수 있습니다. 필터체인 프록시에서 사용할 수 있는 보안 필터체인은
List 형식으로 담을 수 있게 설정돼 있어 URI 패턴에 따라 특정 보안필터 체인을 선택해서 사용하게 됩니다.

보안필터 체인에서 사용하는 필터는 여러 종류가 있으며, 각 필터마다 실행되는 순서가 다릅니다.
공식 문서에서 소개하는 필터의 실행 순서는 다음과 같습니다.

```
ChannelProcessingFilter
WebAsyncManagerIntegrationFilter
SecurityContextPersistenceFilter
HeaderWriterFilter
CorsFilter
CsrfFilter
LogoutFilter
OAuth2AuthorizationRequestRedirectFilter
Saml2WebSsoAuthenticationRequestFilter
X509AuthenticationFilter
AbstractPreAuthenticatedProcessingFilter
CasAuthenticationFilter
OAuth2LoginAuthenticationFilter
Saml2WebSsoAuthenticationFilter
UsernamePasswordAuthenticationFilter
DefaultLoginPageGeneratingFilter
DefaultLogoutPageGeneratingFilter
ConcurrentSessionFilter
DigestAuthenticationFilter
BearerTokenAuthenticationFilter
BasicAuthenticationFilter
RequestCacheAwareFilter
SecurityContextHolderAwareRequestFilter
JaasApiIntegrationFilter
RememberMeAuthenticationFilter
AnonymousAuthenticationFilter
OAuth2AuthorizationCodeGrantFilter
SessionManagementFilter
ExceptionTranslationFilter
AuthorizationFilter
SwitchUserFilter
```

보안 필터체인은 WebSecurityConfigurerAdapter 클래스를 상속받아 설정할 수 있습니다.
앞에서 이야기한 것처럼 필터체인 프록시는 여러 보안 필터체인을 가질 수 있는데,
여러 보안 필터체인을 만들기 위해서는 WebSecurityConfigurerAdapter 클래스를 상속받는 클래스를 여러 개 생성하면 됩니다.
이때 WebSecurityConfigurerAdapter 클래스에는 #Order 어노테이션을 통해 우선순위가 지정돼 있는데,
2개 이상의 클래스를 생성했을 때 똑같은 설정으로 우선순위가 100이 설정돼 있으면 예외가 발생하기 때문에 상속받은 클래스에서
@Order 어노테이션을 지정해 순서를 정의하는 것이 중요합니다.
별도의 설정이 없다면 스프링 시큐리티에서는 다음과 같이 SecurityFilterChain에서 사용하는 필터중 UsernamePasswordAuthenticationFilter를 통해 인증을 처리합니다.

![new repo](/assets/images/posts_img/Spring/UsernamePasswordAuthenticationFilter.png)

위 그림의 인증 수행 과정을 설명하면 다음과 같습니다.
클라이언트로부터 요청을 받으면 서블릿 필터에서 SecurityFilterChain에서으로 작업이 위임되고
그중 UsernamePasswordAuthenticationFilter(위 그림에서 AuthenticationFilter에 해당)에서 인증을 처리합니다.
AuthenticationFilter는 요청 객체(HttpServletRequest)에서 username과 password를 추출해서 토큰을 생성합니다.
그러고 나서 AuthenticationManager에게 토큰을 전달합니다. AuthenticationManager는 인터페이스이며, 일반적으로 사용되는 구현체는 ProviderManager입니다.
ProviderManager는 인증을 위해 AuthenticationProvider로 토큰을 전달합니다.
AuthenticationProvider는 토근의 정보를 UserDetailService에 전달합니다.
UserDetailService는 전달받은 정보를 통해 데이터베이스에서 일치하는 사용자를 찾아 UserDetails 객체를 생성합니다.
생성된 UserDetails 객체는 AuthenticationProvider로 전달되며, 해당 Provider에서 인증을 수행하고 성공하게 되면
ProviderManager로 권한을 담은 토큰을 전달합니다.
ProviderManager는 검증된 토근을 AuthenticationFilter로 전달합니다.
AuthenticationFilter는 검증된 토큰을 SecurityContextHolder에 있는 SecurityContext에 저장합니다.
위 과정에서 사용된 UsernamePasswordAuthenticationFilter는 접근 권한을 확인하고 인증이 실패할 경우 로그인 폼이라는
화면을 보내는 역할을 수행합니다.
이 책에서 실습 중인 프로젝트는 화면이 없는 RESTful 애플리케이션이기 때문에 다른 필터에서 인증 및 인가 처리를 수행해야 합니다.
이 책에서는 JWT 토큰을 사용해 인증을 수행할 예정이라 JWT와 관련된 필터를 생성하고 UsernamePasswordAuthenticationFilter 앞에 배치해서 먼저 인증을 수행할수 있게 설정하겠습니다.
 
# JWT
JWT(JSON Web TOken)는 당사자 간에 정보를 JSON 형태로 안전하게 전송하기 위한 토큰입니다.
JWT는 URL로 이용할 수 있는 문자열로만 구성돼 있으며, 디지털 서명이 적용돼 있어 신회할 수 있습니다.
JWT는 주로 서버와의 통신에서 권한 인가를 위해 사용됩니다.
URL에서 사용할 수 있는 문자열로만 구성돼 있기 때문에 HTTP 구성요소 어디든 위치할 수 있습니다.

### JWT의 구조
JWT는 점('.')으로 구분된 아래의 세 부분으로 구성됩니다.

- 헤더(Header)
- 내용(Payload)
- 서명(Signature)

따라서 JWT는 일반적으로 다음과 같은 형식을 띠고 있습니다.

xxxxxx.yyyyy.zzzzz
헤더.내용.서명

### 헤더
JWT의 헤더는 검증과 관련된 내용을 담고 있습니다.
헤더는 다음과 같이 두 가지 정보를 포함하고 있는데, 바로 alg와 typ 속성입니다.

```
{
  "alg": "HS256"
  "typ": "JWT"
}
```

alg 속성에서는 해싱 알고리즘을 지정합니다.
해싱 알고리즘은 보통 SHA256 또는 RSA를 사용하며, 토큰을 검증할 때 사용되는 서명 부분에서 사용됩니다.
위 예제에 작성돼 있는 HS256은  'HMACSHA256' 알고리즘을 사용한다는 의미입니다.
그리고 typ 속성에는 토큰의 타입을 지정합니다.
이렇게 완성된 헤더는 Base64Url 형식으로 인코딩돼 사용됩니다.

# 내용
JWT의 내용에는 토큰에 담는 정보를 포함합니다. 이곳에 포함된 속성들은 클레임(Claim)이라 하며, 크게 세 가지로 분류됩니다.

- 등록된 클레임(Registered Claims)
- 공개 클레임(Public Claims)
- 비공개 클레임(Private Claims)
 
등록된 클레임은 필수는 아니지만 토큰에 대한 정보를 담기 위해 이미 이름이 정해져 있는 클레임을 뜻합니다.
등록된 클레임은 다음과 같이 정의돼 있습니다.

- iss : JWT의 발급자(Issuer) 주체를 나타냅니다. iss의 값은 문자열이나 URL를 포함하는 대소문자를 구분하는 문자열입니다.
- sub: JWT의 제목(Subject)입니다.
- aud: JWT의 수신인(Audience)입니다. JWT를 처리하려는 각 주체는 해당 값으로 자신을 식별해야 합니다.
요청을 처리하는 주체가 'aud' 값으로 자신을 식별하기 않으면 JWT는 거부됩니다.
- exp: JWT의 만료시간(Expiration)입니다. 시간은  NumericDate 형식으로 지정해야 합니다.
- nbf: 'Not Before'를 의미합니다.
- lat: JWT가 발급된 시간(Issued at)입니다.
- jti: JWT의 식별자(JWT ID)입니다. 주로 정복 처리를 방지하기 위해 사용됩니다.

공개 클레임은 키 값을 마음대로 정의할 수 있습니다. 다만 충돌이 발생하지 않을 이름으로 설정해야 합니다.
비공개 클레임은 통신 간에 상호 합의되고 등록된 클레임과 공개된 클레임이 아닌 클레임을 의미합니다.
내용의 예는 다음과 같습니다.

JWT 사용예시

```
{
  "sub" : "wikibooks payload"
  "exp" : "1602076408"
  "userId" : "wikibooks"
  "username" : "flature"
}
```

이렇게 완성된 내용은 Base64Url 형식으로 인코딩되어 사용됩니다.

# 서명
JWT의 서명 부분은 인코딩된 헤더, 인코딩된 내용, 비밀키, 헤더의 알고리즘 속성값을 가져와 생성됩니다.
예를 들어, HMAC SHA256 알고리즘을 사용해서 서명을 생성한다면 다음과 같은 방식으로 생성됩니다.

### 서명 생성 방식

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
)
```

서명은 토큰의 값들을 포함해서 암호화하기 때문에 메시지가 도중에 변경되지 않았는지 확인 할 때 사용됩니다.

# JWT 디버거 사용하기
JWT 공식 사이트에서는 더욱 쉽게 JWT를 생성해볼 수 있습니다.
웹 브라우저에서 다음 URL로 접속

https://jwt.io/#debugger-io

접속해서 화면을 보면 Encoded와 Decoded로 나눠져 있으며, 양측의 내용이 일치하는지 사이트에서 확인할 수도 있고
Decoded의 내용을 변경하면 Encoded의 콘텐츠가 자동으로 반영됩니다.

# 스프링 시큐리티와 JWT 적용
이제 애플리케이션에 스프링 시큐리티와 JWT를 적용해보겠습니다.
이전 장에서 사용했던 기본 프로젝트 틀을 가져가기 위해 7장에서 사용한 프로젝트 코드를 그대로 가져와 사용합니다.
인증과 인가 코드를 작성하기 위해 의존성을 추가합니다.

```
    // 시큐리티
    implementation 'org.springframework.boot:spring-boot-starter-security'

    // JWT      ------------------------------------------------------------
    implementation 'io.jsonwebtoken:jjwt-api:0.11.2'
    implementation 'io.jsonwebtoken:jjwt-impl:0.11.2'
    implementation 'io.jsonwebtoken:jjwt-gson:0.11.2'
```

스프링 시큐리티는 기본적으로 UsernamePasswordAuthenticationFilter를 통해 인증을 수행하도록 구성돼 있습니다.
참고로 이 필터에서는 인증이 실패하면 로그인 폼이 포함된 화면을 전달하게 되는데, 이 책의 실습 프로젝트에는 이러한 화면이 없습니다.
따라서 JWT를 사용하는 인증 필터를 구현하고 UsernamePasswordAuthenticationFilter 앞에 인증 필터를 배치해서
인증 주체를 변경하는 작업을 수행하는 방식으로 구성하겠습니다.

# UserDatails와 UserDetailsService 구현
먼저 다음과 같이 사용자 정보를 담는 엔티티를 생성합니다.

```java
package com.springboot.security.data.entity;

import com.fasterxml.jackson.annotation.JsonProperty;
import jakarta.persistence.*;
import lombok.*;
import org.springframework.security.core.GrantedAuthority;
import org.springframework.security.core.authority.SimpleGrantedAuthority;
import org.springframework.security.core.userdetails.UserDetails;

import java.util.ArrayList;
import java.util.Collection;
import java.util.List;
import java.util.stream.Collectors;

@Builder
@AllArgsConstructor
@NoArgsConstructor
@Getter
@Setter
@Entity
public class User extends BaseEntity implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String uid;

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private String name;

    @ElementCollection(fetch = FetchType.EAGER)
    @Builder.Default
    private List<String> roles = new ArrayList<>();


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        // 계정이 가지고 있는 권한 목록 반환
        return this.roles
                .stream()
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());

    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public String getUsername() {
        // 계정의 이름(아이디)을 반환
        return this.uid;
    }


    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isAccountNonExpired() {
        // 계정이 만료됐는지 반환
        return true; // 만료되지 않았음을 의미
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isAccountNonLocked() {
        // 계정이 잠겨있는지 반환
        return true; // 잠겨있지 않음을 의미
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isCredentialsNonExpired() {
        // 비밀번호가 만료됐는지 반환
        return true; // 만료되지 않았음을 의미
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isEnabled() {
        // 계정이 활성화 되어 있는지 반환
        return true; // 활성화 상태임을 의미
    }
}
```
