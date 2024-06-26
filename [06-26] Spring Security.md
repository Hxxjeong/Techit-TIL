## [06/26] Spring Security



### Spring Security

자바 기반 어플리케이션에서 인증(Authentication)과 권한 부여(인가, Authorization)을 담당하는 프레임워크



#### 주요 기능

- 인증 (Authentication)

  사용자의 신원을 확인하는 과정

  Form 로그인, HTTP Basic, OAuth 등 지원

- 권한 부여 (인가, Authorization)

  인증된 사용자가 어플리케이션 내에서 어떤 자원에 접근할 수 있는지 결정

  RBAC를 포함한 다양한 접근 제어 방식 지원

- 세션 관리

  사용자 세션의 생성/유지/만료 등을 관리하여 보안성을 높임

- 보안 이벤트 로깅

  로그인 시도, 실패 등의 보안 관련 이벤트를 로깅하여 보안 감사를 도움



### Spring Security Architecture

서블릿 필터 기반



#### FilterChain

클라이언트가 어플리케이션에 요청을 보내면 컨테이너는 요청 URI 경로에 따라 HttpServletRequest를 처리해야 하는 Filter 인스턴스와 서블릿이 포함된 FilterChain 생성

DispatcherServlet 인스턴스가 서블릿 역할

여러 Filter를 사용할 수 있으나 Servlet의 요청/응답 객체는 최대 하나의 Servlet만 처리

```java
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain) throws IOException, ServletException {
        // 작업 수행

        chain.doFilter(servletRequest, servletResponse);
}
```



#### DelegatingFilterProxy

서블릿 커네팅너의 생명주기와 ApplicationContext 간의 연결을 가능하게 함

Filter를 구현하는 Spring Bean에 모든 작업 위임

```java
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
    Filter delegate = getFilterBean(someBeanName);
    delegate.doFilter(request, response);
}
```

Filter Bean 인스턴스를 Lazy Loading 가능 (컨테이너가 Filter 인스턴스를 등록해야 시작)



#### FilterChainProxy

Spring Security가 제공하는 FIlter

현재 요청에 대해 어떤 Spring Security Filter 인스턴스가 호출되어야 하는지 결정 → 첫번째로 매칭되는 SecurityFilterChain만 호출



#### SecurityFilterChain

FilterChainProxy가 현재 요청에 대해 어떤 Spring Security Filter 인스턴스를 호출할지 결정하는 데 사용 (DelegatingFilterProxy 대신 FilterChainProxy에 등록)

```java
@Configuration
@EnableWebSecurity
@Slf4j
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .authorizeRequests(authorizeRequest -> authorizeRequest
                                .anyRequest()   // 모든 요청
                                .authenticated()
                )
                .formLogin(Customizer.withDefaults());

        // 아이디 기억 설정
        http
                .rememberMe(remember -> remember
                        .rememberMeParameter("remember")
                        .tokenValiditySeconds(3000) // 토큰 유지 시간
                );

        return http.build();
    }
}
```

remember-me: 세션이 만료되고 웹브라우저가 종료된 후에도 어플리케이션이 계속 사용자를 기억하는 기능

1. Remember-Me 쿠키를 구워서 클라이언트에게 보냄
2. Remember-Me 쿠키를 확인해서 토큰 기반으로 인증
3. 토큰이 검증되면 사용자로 로그인됨

![image-20250201193109258](/Users/hj/Library/Application Support/typora-user-images/image-20250201193109258.png)



### Filter 구성 예제

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(Customizer.withDefaults())    // CSRF 공격으로부터 보호
            .authorizeHttpRequests(authorize -> authorize
                .anyRequest().authenticated()   // 요청 권한 부여 처리
            )
            .httpBasic(Customizer.withDefaults())   // HTTP 기본 인증 처리
            .formLogin(Customizer.withDefaults());  // form login 처리
            return http.build();
    }
}
```

CSRF: 웹 보안 취약점의 일종으로 공격자가 의도한 행위를 특정 웹 사이트에 요청하게 하는 공격

- Filter를 Spring Bean으로 선언할 때 주의사항

  FIlter를 `@Component`로 주석 처리하거나 구성에서 Bean으로 선언하여 Spring Bean으로 선언하면 Spring Boot가 자동으로 임베디드 컨테이너에 등록 → 컨테이너와 Spring Security에 의해 Filter가 두번 호출될 수 있음

  ```java
  @Bean
  public FilterRegistrationBean<사용자Filter> tenantFilterRegistration(사용자Filter filter) {
      FilterRegistrationBean<사용자Filter> registration = new FilterRegistrationBean<>(filter);
      registration.setEnabled(false); // 중복 호출 방지
      return registration;
  }
  ```



### 보안 예외 처리

`ExceptionTranslationFilter`는 접근 거부 예외과 인증 예외를 HTTP 응답으로 변환할 수 있게 함

인증/인가 예외가 발생하지 않으면 Filter는 아무것도 하지 않음

`FilterChainProxy`에 보안 필터 중 하나로 삽입

![image-20250201193130530](/Users/hj/Library/Application Support/typora-user-images/image-20250201193130530.png)

- 동작 과정

  1. 예외 처리 Filter가 `doFilter` 메소드를 호출하여 어플리케이션의 나머지 부분 호출

  2. 사용자가 인증되지 않았거나 `AuthenticationException`인 경우 인증 시작

     2-1. `SecurityContextHolder` 초기화

     2-2. 재요청할 수 있도록 `HttpServletRequest` 저장

     2-3. `AuthenticationEntryPoint`를 사용하여 클라이언트로부터 자격 증명 요청

  3. 2-3에서 `AccessDeniedException`인 경우 접근 거부로 처리

  4. `AccessDeniedHandler`가 접근 거부 처리



### 인증 간 요청 저장

요청에 인증이 없고, 인증이 필요한 Resource에 대한 요청이 있는 경우, 재요청하기 위해 Request를 저장해야 함

- RequestCache

  사용자가 성공적으로 인증되면 `RequestCache`를 사용하여 원래 요청 재생

  `RequestCacheAwareFilter`는 `RequestCache`를 사용하여 `HttpServletRequest` 저장

- NullRequestCache

  RequestCache와 반대로 요청이 저장되지 않도록 방지함

------

https://velog.io/@ohzzi/Spring-DIIoC-IoC-DI-그게-뭔데
