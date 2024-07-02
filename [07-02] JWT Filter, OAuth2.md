## [07/02] JWT Filter, OAuth2



### JWT 인증/인가 Filter

Filter는 클라이언트가 보낸 HTTP 요청에서 JWT 토큰을 추출하고 이를 사용하여 `Authentication` 객체 생성

1. HTTP 요청에서 JWT 토큰 추출하기

   요청 헤더 Authorization에서 accessToken 쿠키를 찾아 토큰 추출

2. 추출된 JWT 토큰 검증 및 처리

   토큰의 유효성을 검사

3. `Authentication` 객체 생성

   토큰이 유효한 경우 추출한 claims 정보를 사용하여 사용자의 정보를 얻음

   얻은 정보를 바탕으로 Spring Security의 CustomUserDetails 생성

   권한(authorities) 정보를 설정하여 `JwtAuthenticationToken` 생성

   생성된 인증 객체를 Spring Security의 `SecurityContextholder`에 설정하여 현재 인증된 사용자로 설정

4. 처리된 요청의 흐름 제어

   인증 과정에 문제가 없는 경우 원래 요청을 처리하기 위해 Filter 또는 Controller로 요청 전달

   예외가 발생한 경우 클라이언트에게 적절한 HTTP 상태 코드와 함께 예외 메시지 반환

```java
public class JwtAuthenticationToken extends AbstractAuthenticationToken {
    private String token;
    private Object principal;   // 로그인한 사용자 id, email
    private Object credentials;

    // security로 인증 이용
    public JwtAuthenticationToken(Collection<? extends GrantedAuthority> authorities,
                                  Object principal,
                                  Object credentials) {
        super(authorities);
        this.principal = principal;
        this.credentials = credentials;
        this.setAuthenticated(true);
    }

    // token으로 인증 이용
    public JwtAuthenticationToken(String token) {
        super(null);
        this.token = token;
        this.setAuthenticated(false);
    }

    @Override
    public Object getCredentials() {
        return this.credentials;
    }

    @Override
    public Object getPrincipal() {
        return this.principal;
    }
}
public class CustomUserDetails implements UserDetails {
    private final String username;
    private final String password;
    private final String name;
    private final List<GrantedAuthority> authorities;

    public CustomUserDetails(String username, String password, String name, List<String> roles) {
        this.username = username;
        this.password = password;
        this.name = name;
        this.authorities = roles.stream()
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());
    }

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return authorities;
    }

    @Override
    public String getPassword() {
        return password;
    }

    @Override
    public String getUsername() {
        return username;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    private final JwtTokenizer jwtTokenizer;
    private final CustomAuthenticationEntryPoint customAuthenticationEntryPoint;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(authorize -> authorize
                        .requestMatchers("/userregform", "/userreg", "/", "/login", "/refreshToken", "/loginform").permitAll()
                        .anyRequest()
                        .authenticated()
                )
                // filter 생성
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenizer), UsernamePasswordAuthenticationFilter.class)
                .formLogin(form -> form.disable())  // form 로그인 사용 X
                .sessionManagement(sessionManagement -> sessionManagement
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 세션 로그인 사용 X
                )
                .httpBasic(httpBasic -> httpBasic.disable())
                .cors(cors -> cors
                        .configurationSource(configurationSource())
                )
                // 예외 처리
                .exceptionHandling(exception -> exception
                        .authenticationEntryPoint(customAuthenticationEntryPoint)
                );

        return http.build();
    }

    // CORS 설정
    public CorsConfigurationSource configurationSource() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();

        config.addAllowedOrigin("*");
        config.addAllowedHeader("*");
        config.addAllowedMethod("*");
        config.setAllowedMethods(List.of("GET", "POST", "DELETE"));

        source.registerCorsConfiguration("/**", config);    // 모든 요청에 설정 적용

        return source;
    }
}
```



#### Refresh Token으로 Access Token 발급

```java
// refresh token 생성
    @PostMapping("/refreshToken")
    public ResponseEntity createRefreshToken(HttpServletRequest request,
                                             HttpServletResponse response) {
        String refreshToken = null;
        Cookie[] cookies = request.getCookies();

        // 쿠키로부터 refresh token 얻어오기
        if(cookies != null) {
            for(Cookie cookie: cookies) {
                if("refreshToken".equals(cookie.getName())) {
                    refreshToken = cookie.getValue();
                    break;
                }
            }
        }

        // 쿠키가 없는 경우 예외 처리
        if(refreshToken == null) return new ResponseEntity<>(HttpStatus.BAD_REQUEST);

        // 토큰으로부터 정보 얻기
        Claims claims = jwtTokenizer.parseRefreshToken(refreshToken);
        Long userId = Long.valueOf((Integer) claims.get("userId"));

        User user = userService.findById(userId)
                .orElseThrow(() -> new IllegalArgumentException("사용자를 찾지 못하였습니다."));

        // access token 생성
        List roles = (List) claims.get("roles");

        String accessToken = jwtTokenizer.createAccessToken(
                userId,
                user.getEmail(),
                user.getName(),
                user.getUsername(),
                roles
        );

        // 쿠키에 토큰 저장
        Cookie accessTokenCookie = new Cookie("accessToken", accessToken);
        accessTokenCookie.setHttpOnly(true);
        accessTokenCookie.setPath("/");
        accessTokenCookie.setMaxAge(Math.toIntExact(JwtTokenizer.ACCESS_TOKEN_EXPIRE_COUNT / 1000));

        response.addCookie(accessTokenCookie);

        // 응답 객체
        UserLoginRspDto rspDto = UserLoginRspDto.builder()
                .accessToken(accessToken)
                .refreshToken(refreshToken)
                .userId(userId)
                .name(user.getName())
                .build();

        return new ResponseEntity(rspDto, HttpStatus.OK);
    }
```



### OAuth2

사용자가 신뢰할 수 있는 외부 인증 제공자를 통해 어플리케이션에 로그인할 수 있도록 하는 인증 메커니즘

- 관련 용어

  - Resource Owner

    자신의 리소스에 접근할 권한을 가지고 있는 주체

  - Client

    사용자가 접근하려는 어플리케이션

    OAuth2 프로토콜에서 리소스 소유자의 권한을 요청하는 주체

    리소스 소유자의 동의를 얻어 OAuth2 제공자로부터 액세스 토큰을 받고 이를 사용해 리소스 서버에 요청을 보냄

  - Authorization Server (인증 서버)

    OAuth2 제공자로 사용자를 인증하고 권한을 부여하며 액세스 토큰을 발급하는 서버

  - Resource Server

    보호된 리소스를 호스팅하는 서버

    액세스 토큰을 사용해 보호된 리소스에 대한 요청 처리

  - Access Token

    인증 서버가 클라이언트에게 발급하는 토큰으로 리소스 서버에 접근할 때 사용

    리소스 서버에 보호된 리소스를 요청할 때 클라이언트가 사용하는 자격 증명

  - Refresh Token

    액세스 토큰이 만료되었을 때 새로운 액세스 토큰을 얻기 위해 사용하는 토큰

    사용자가 다시 인증하지 않고 새로운 액세스 토큰을 얻을 수 있도록 함

  - Authorization Grant (권한 부여)

    클라이언트가 액세스 토큰을 얻기 위해 인증 서버에 제출하는 자격 증명



#### 로그인 흐름

1. 사용자 인증 요청

   클라이언트는 사용자를 인증하기 위해 인증 서버로 리다이렉션

2. 사용자 인증

   사용자는 인증 서버에서 로그인하고, 인증 서버는 권한 코드를 클라이언트에게 리다이렉션

3. 권한 코드 교환

   클라이언트는 권한 코드를 액세스 토큰으로 교환하기 위해 인증 서버에 요청을 보냄

4. 토큰 수신

   인증 서버는 클라이언트에게 액세스 토큰 (또는 리프레시 토큰) 반환

5. 리소스 요청

   클라이언트는 액세스 토큰을 사용해 리소스 서버에 보호된 리소스를 요청

6. 리소스 접근 허용

   리소스 서버는 액세스 토큰을 검증하고 유효한 경우 보호된 리소스를 클라이언트에게 반환



#### 실습

- SecurityConfig

  ```java
  @Configuration
  @EnableWebSecurity
  @RequiredArgsConstructor
  public class SecurityConfig {
      private final SocialUserService socialUserService;
      private final CustomOAuth2AuthenticationSuccessHandler customOAuth2AuthenticationSuccessHandler;
  
      @Bean
      public PasswordEncoder passwordEncoder() {
          return new BCryptPasswordEncoder();
      }
  
      @Bean
      public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
          http
                  .csrf(csrf -> csrf.disable())   // csrf 토큰 해제
                  .authorizeHttpRequests(authorize -> authorize
                          // form 로그인
                          .requestMatchers("/userregform", "/userreg", "/", "/loginform").permitAll()
                          // social 로그인
                          .requestMatchers("/oauth2/**", "/login/oauth2/code/github","/registerSocialUser", "/saveSocialUser").permitAll()
                          .anyRequest()
                          .authenticated()
                  )
                  .formLogin(form -> form.disable())
                  .cors(cors -> cors
                          .configurationSource(configurationSource()))
                  .httpBasic(httpBasic -> httpBasic.disable())
                  // OAuth2 로그인 설정
                  .oauth2Login(oauth2 -> oauth2
                          .loginPage(("/loginform"))
                          .failureUrl("/loginFailure")
                          .userInfoEndpoint(userInfo -> userInfo
                                  .userService(this.oauth2UserService())
                          )
                          .successHandler(customOAuth2AuthenticationSuccessHandler)
                  );
  
          return http.build();
      }
  
      // OAuth2 설정
      @Bean
      public OAuth2UserService<OAuth2UserRequest, OAuth2User> oauth2UserService() {
          DefaultOAuth2UserService delegate = new DefaultOAuth2UserService();
          return oauth2UserRequest -> {
              OAuth2User oauth2User = delegate.loadUser(oauth2UserRequest);
              // 여기에 Github 유저 정보를 처리하는 로직 추가
              // 예: DB에 사용자 정보 저장, 권한 부여 등
  
              String token = oauth2UserRequest.getAccessToken().getTokenValue();
  
              // Save or update the user in the database
              String provider = oauth2UserRequest.getClientRegistration().getRegistrationId();
              String socialId = String.valueOf(oauth2User.getAttributes().get("id"));
              String username = (String) oauth2User.getAttributes().get("login");
              String email = (String) oauth2User.getAttributes().get("email");
              String avatarUrl = (String) oauth2User.getAttributes().get("avatar_url");
  
              socialUserService.createOrUpdate(socialId, provider, username, email, avatarUrl);
  
              return oauth2User;
          };
      }
  
      // CORS 설정
      public CorsConfigurationSource configurationSource() {
          UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
          CorsConfiguration config = new CorsConfiguration();
  
          config.addAllowedOrigin("*");
          config.addAllowedHeader("*");
          config.addAllowedMethod("*");
          config.setAllowedMethods(List.of("GET", "POST", "DELETE"));
  
          source.registerCorsConfiguration("/**", config);    // 모든 요청에 설정 적용
  
          return source;
      }
  }
  ```

- CustomOAuth2AuthenticationSuccessHandler

  ```java
  @Component
  @RequiredArgsConstructor
  public class CustomOAuth2AuthenticationSuccessHandler implements AuthenticationSuccessHandler {
      private final SocialLoginInfoService socialLoginInfoService;
      private final UserService userService;
  
      @Override
      public void onAuthenticationSuccess(HttpServletRequest request,
                                          HttpServletResponse response,
                                          Authentication authentication) throws IOException, ServletException {
          // redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
          String requestUri = request.getRequestURI();
          String provider = extractProviderFromUri(requestUri);
  
          // provider가 없는 경로 요청이 왔다는 것은 문제가 발생한 것
          if(provider == null){
              response.sendRedirect("/");
              return;
          }
  
          Authentication auth = SecurityContextHolder.getContext().getAuthentication();
          DefaultOAuth2User defaultOAuth2User = (DefaultOAuth2User) auth.getPrincipal();
  
          int socialId = (int)defaultOAuth2User.getAttributes().get("id");
          String name = (String)defaultOAuth2User.getAttributes().get("name");
  
          Optional<User> userOptional = userService.findByProviderAndSocialId(provider, String.valueOf(socialId));
          // 회원 정보가 있으면 로그인 처리
          if (userOptional.isPresent()) {
              User user = userOptional.get();
  
              // CustomUserDetails 생성
              CustomUserDetails customUserDetails = new CustomUserDetails(
                      user.getUsername(),
                      user.getPassword(),
                      user.getName(),
                      user.getRoles().stream()
                              .map(Role::getName)
                              .collect(Collectors.toList())
              );
  
              // Authentication 객체 생성 및 SecurityContext에 설정
              Authentication newAuth = new UsernamePasswordAuthenticationToken(
                      customUserDetails, null, customUserDetails.getAuthorities()
              );
              SecurityContextHolder.getContext().setAuthentication(newAuth);
  
              response.sendRedirect("/welcome");
          }
          else { // 소셜로 회원가입이 아직 안 되었을 때 (회원 정보가 없는 경우)
              // 소셜 로그인 정보 저장
              SocialLoginInfo socialLoginInfo = socialLoginInfoService.create(provider, String.valueOf(socialId));
              response.sendRedirect("/registerSocialUser?provider=" + provider + "&socialId=" + socialId + "&name=" + name + "&uuid=" + socialLoginInfo.getUuid());
          }
      }
  
      private String extractProviderFromUri(String uri) {
          if(uri == null || uri.isBlank()) {
              return null;
          }
  
          if(!uri.startsWith("/login/oauth2/code/")){
              return null;
          }
  
          // 예: /login/oauth2/code/github -> gitHub
          String[] segments = uri.split("/");
          return segments[segments.length - 1];
      }
  }
  ```

- UserController

  ```java
  // 소셜 로그인 관련
      @GetMapping("/registerSocialUser")
      public String joinSocialUser(@RequestParam("provider") String provider,
                                   @RequestParam("socialId") String socialId,
                                   @RequestParam("name") String name,
                                   @RequestParam("uuid") String uuid,
                                   Model model) {
          model.addAttribute("provider", provider);
          model.addAttribute("socialId", socialId);
          model.addAttribute("name", name);
          model.addAttribute("uuid", uuid);
  
          return "users/registerSocialUser";
      }
  
      @PostMapping("/saveSocialUser")
      public String saveSocialUser(@RequestParam("provider") String provider,
                                   @RequestParam("socialId") String socialId,
                                   @RequestParam("name") String name,
                                   @RequestParam("username") String username,
                                   @RequestParam("email") String email,
                                   @RequestParam("uuid") String uuid,
                                   Model model) {
          Optional<SocialLoginInfo> socialLoginInfoOptional = socialLoginInfoService.findByProviderAndUuidAndSocialId(provider, uuid, socialId);
  
          if (socialLoginInfoOptional.isPresent()) {
              SocialLoginInfo socialLoginInfo = socialLoginInfoOptional.get();
              LocalDateTime now = LocalDateTime.now();
              Duration duration = Duration.between(socialLoginInfo.getCreatedAt(), now);
  
              if (duration.toMinutes() > 20)
                  return "redirect:/error"; // 20분 이상 경과한 경우 에러 페이지로 리다이렉트
  
              // 유효한 경우 User 정보 저장
              userService.saveUser(username, name, email, socialId, provider,passwordEncoder);
              return "redirect:/";
          }
          else return "redirect:/error"; // 해당 정보가 없는 경우 에러 페이지로 리다이렉트
      }
  ```
