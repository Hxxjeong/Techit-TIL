## [07/15] Redis를 이용한 WebSocket



### Spring Security를 이용한 Web Socket 연결

- Interceptor가 필요한 이유

  인증 및 권한 부여, 세션 관리, 로그 및 모니터링, 사용자 정의 로직 적용

  ```java
  @Component
  public class CustomHandshakeInterceptor extends HttpSessionHandshakeInterceptor {
      @Override
      public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
          // 인증 정보로부터 username 얻어오기
          SecurityContext context = SecurityContextHolder.getContext();
          if(context.getAuthentication() != null) {
              attributes.put("SPRING_SECURITY_CONTEXT", context);
          }
  
          return super.beforeHandshake(request, response, wsHandler, attributes);
      }
  
      @Override
      public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception ex) {
          super.afterHandshake(request, response, wsHandler, ex);
      }
  }
  ```

  ```java
  @Configuration
  @EnableWebSocket
  @RequiredArgsConstructor
  // WebSocket 설정을 위한 구성 클래스
  public class WebSocketConfig implements WebSocketConfigurer {
      private final CustomHandshakeInterceptor customHandshakeInterceptor;
  
      @Override
      public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
          registry.addHandler(new MyWebSocketHandler(), "/ws")
                  .setAllowedOrigins("*")
                  .addInterceptors(customHandshakeInterceptor);
      }
  }
  ```

  ```java
  // WebSocket 연결을 관리하고 메시지를 처리하며, 연결이 종료될 때의 작업을 정의하는 클래스
  public class MyWebSocketHandler extends TextWebSocketHandler {
      private static final Set<WebSocketSession> sessions = Collections.synchronizedSet(new HashSet<>());
  
    // WebSocket 연결이 성립될 때 호출
      @Override
      public void afterConnectionEstablished(WebSocketSession session) throws Exception {
          System.out.println("Connection established with session: " + session.getId());
          sessions.add(session);
      }
  
    // 메시지를 수신할 때 호출
      @Override
      protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
          String payload = message.getPayload();
          System.out.println("Received message: " + payload);
  
          // 현재 인증된 사용자 정보 가져오기
          SecurityContext context = (SecurityContext) session.getAttributes().get("SPRING_SECURITY_POLICY");
          String username = "Unknown User";
  
          if(context != null
                  && context.getAuthentication() != null
                  && context.getAuthentication().getPrincipal() instanceof UserDetails) {
              UserDetails userDetails = (UserDetails) context.getAuthentication().getPrincipal();
              username = userDetails.getUsername();
          }
  
          // sessions가 동기화 되어 있기 때문에 synchronized 블록이 생략되어도 됨
          for (WebSocketSession webSocketSession : sessions) {
              if (webSocketSession.isOpen()) {    // 세션이 열려있을 때만 전송
                  webSocketSession.sendMessage(new TextMessage(username + ": " + payload));
              }
          }
      }
  
    // 연결이 종료될 때 호출
      @Override
      public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
          System.out.println("Connection closed with session: " + session.getId());
          sessions.remove(session);
      }
  }
  ```



### Redis

- Docker-compose.yml

  ```yaml
  services:
    redis:
      image: redis:latest
      container_name: redis
      command: ["redis-server", "--requirepass", "1234"]
      ports:
        - "6379:6379"
      volumes:
      - ./redis-data:/data
  
  volumes:
    redis-data:
  ```



#### Spring Data Redis

Spring 프레임워크를 사용하여 Redis와 상호작용하는 데 도움을 주는 라이브러리

- 구성 요소

  - RedisConnectionFactory

    Redis와의 연결을 생성하는 데 사용

    Redis의 연결 설정 관리 (호스트, 포트, 비밀번호 등)

  - RedisTemplate

    Redis와의 데이터 작업을 수행하는 데 사용되는 주요 클래스

    Redis에서 데이터를 직렬화하고 역직렬화하는 방식 제어

  - RedisSerializer

    Redis에 저장하거나 읽어올 때 데이터를 직렬화하고 역직렬화하는 방법 정의

    `StringRedisSerializer`: 문자열 데이터 직렬화

    `GenericJackson2JsonRedisSerializer`: JSON 형식으로 데이터를 직렬화하는 데 사용

  - `@RedisHash`

    Redis Hash로 저장할 엔티티 클래스에 적용

- 동작 방식

  1. 설정 및 초기화

     RedisConfig 클래스에서 설정 초기화

     `LettuceConnetionFactory`를 통해 Redis 서버와의 연결 설정

  2. 데이터 작업

     `RedisTemplate`는 데이터 직렬화 및 역직렬화를 관리하며 Redis 서버와의 상호작용 수행

  3. 어플리케이션 실행

```java
@Configuration
@EnableRedisRepositories
// Spring Data Redis를 설정하기 위한 구성 클래스
public class RedisConfig {
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration("localhost", 6379);
        redisStandaloneConfiguration.setPassword("1234");

      // Redis 연결 팩토리 생성
        return new LettuceConnectionFactory(redisStandaloneConfiguration);
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
        redisTemplate.setConnectionFactory(redisConnectionFactory);

        redisTemplate.setKeySerializer(new StringRedisSerializer());
        redisTemplate.setHashKeySerializer(new StringRedisSerializer());
        redisTemplate.setValueSerializer(new GenericJackson2JsonRedisSerializer());
        redisTemplate.setHashValueSerializer(new GenericJackson2JsonRedisSerializer());

        return redisTemplate;
    }
}
```



#### Redis를 이용한 Pub/Sub 개발

Pub/Sub 모델: 메시지 전송을 위한 비동기 통신 방식으로 메시지 채널을 통해 송신자와 수신자가 간접적으로 통신

- 동작 방식
  1. Publisher가 메시지 발생
  2. Subscriber가 채널 구독
  3. 메시지 전달

```java
@Configuration
public class RedisListenerConfig {
    @Bean
    public ChannelTopic channelTopic() {
        return new ChannelTopic("chatMessage");
    }

    @Bean
    public MessageListenerAdapter messageListenerAdapter(RedisMessageSubscriber redisMessageSubscriber) {
        return new MessageListenerAdapter(redisMessageSubscriber);
    }

    // 채널로부터 메시지를 수신하여 등록된 리스너에게 전달
    @Bean
    public RedisMessageListenerContainer redisMessageListenerContainer(RedisConnectionFactory redisConnectionFactory,
                                                                       MessageListenerAdapter messageListenerAdapter) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(redisConnectionFactory);
        container.addMessageListener(messageListenerAdapter, new ChannelTopic("chatMessage"));

        return container;
    }
}
@Service
@RequiredArgsConstructor
public class RedisMessagePublisher {
    private final RedisTemplate<String, Object> redisTemplate;
    private final ChannelTopic channelTopic;

    public void publish(String message) {
        redisTemplate.convertAndSend(channelTopic.getTopic(), message);
    }
}
@Component
@RequiredArgsConstructor
public class RedisMessageSubscriber implements MessageListener {
    private static final Set<WebSocketSession> sessions = Collections.synchronizedSet(new HashSet<>());

    private final RedisTemplate<String, String> redisTemplate;

    public static void addSession(WebSocketSession session) {
        sessions.add(session);
    }

    public static void removeSession(WebSocketSession session) {
        sessions.remove(session);
    }

    @Override
    public void onMessage(Message message, byte[] pattern) {
        String msg = (String) redisTemplate.getValueSerializer().deserialize(message.getBody());

        for(WebSocketSession session : sessions) {
            if(session.isOpen()) {
                try {
                    session.sendMessage(new TextMessage(msg));
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}
```
