## [06/25] Filter를 이용한 로그인



### Spring Security

Spring Boot Web Application은 하나의 요청을 하나의 스레드가 처리 (내장된 서블릿 컨테이너 사용)

내장 Tomcat의 스레드 수를 설정하려면 application 파일에서 설정

```yaml
server:
    tomcat:
        max-threads: 250    # 최대 스레드 수
        min-spare-threads: 20   # 최소 여유 스레드 수
```

스레드를 너무 많이 설정하면 메모리가 부족하거나 성능이 나빠질 수 있음



#### ThreadLocal

특정 스레드에 바인딩된 변수를 저장하고 읽을 수 있는 메커니즘 제공

각 스레드가 고유한 값을 가질 수 있도록 해줌

```java
public class UserContext {
    // 초기화
    private static final ThreadLocal<User> USER_THREAD_LOCAL = ThreadLocal.withInitial(() -> null);

    // 현재 스레드에 사용자 설정
    public static void setUser(User user) {
        USER_THREAD_LOCAL.set(user);
    }

    // 현재 스레드에 저장된 사용자 정보 반환
    public static User getUser() {
        return USER_THREAD_LOCAL.get();
    }

    // 현재 스레드에서 ThreadLocal 변수 제거
    public static void clear() {
        USER_THREAD_LOCAL.remove();
    }
}
```

- 필터 사용

  ```java
  import jakarta.servlet.*;
  
  import java.io.IOException;
  
  public class UserFilter implements Filter {
      @Override
      public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
          // 사용자가 요청하면서 보낸 값이 있다면 추출해서 UserContext에 저장
          try {
              // 사용자 정보 설정
              User user = new User();
              user.setUsername("kim");
              user.setPassword("1234");
  
              UserContext.setUser(user);
  
              chain.doFilter(request, response);
          }
          finally {
              UserContext.clear();    // 기존에 사용했던 스레드를 또 사용할 수 있기 때문에 값을 삭제하고 재사용
          }
      }
  }
  ```

  ```java
  @Configuration
  public class FilterConfig {
      @Bean
      public FilterRegistrationBean<UserFilter> UserFilter() {
          FilterRegistrationBean<UserFilter> registrationBean = new FilterRegistrationBean<>();
          UserFilter UserFilter = new UserFilter();
  
          registrationBean.setFilter(userFilter);
          registrationBean.addUrlPatterns("/*");
          registrationBean.setOrder(1);
  
          return registrationBean;
      }
  }
  ```

각 스레드가 독립적으로 `ThreadLocal` 변수를 갖기 때문에 동시성 문제를 피할 수 있음

요청-응답 주기 동안 상태를 쉽게 유지할 수 있음
