## [07/12] 이벤트 프로그래밍, WebSocket



### 이벤트 프로그래밍 (Event-Driven Programming)

특정 사건이 발생할 때 실행되는 프로그램 동작을 설계하는 프로그래밍 패러다임

- 주요 개념

  이벤트: 프로그램이 감지할 수 있는 특정 사전 또는 행동

  이벤트 소스: 이벤트를 발생시키는 객체 또는 컴포넌트

  이벤트 리스너: 이벤트가 발생했을 때 실행될 동작을 정의하는 객체

- 이벤트 주도 아키텍처 (Event-Driven Architecture, EDA)

  시스템 내에서 이벤트를 중심으로 데이터 흐름과 상호작용을 구성하는 아키텍처 스타일

  - 주요 구성 요소

    이벤트 프로듀서: 이벤트를 생성하고 발행하는 컴포넌트

    이벤트 컨슈머: 이벤트를 구독하고 처리하는 컴포넌트

    이벤트 버스: 이벤트를 전달하고 분배하는 메커니즘

- Java의 이벤트 프로그래밍 요소

  Event Object: 이벤트 정보를 담고 있는 객체

  Event Listener Interface: 이벤트를 처리하기 위해 구현해야 하는 인터페이스

  Event Source Object: 이벤트를 발생시키는 객체



#### 의존성 역전 방법

1. 인터페이스 사용

   ex. 댓글을 등록하면 댓글 수를 업데이트 하는 로직

   - CommentCountUpdater

     ```java
     public interface CommentCountUpdater {
       void updateCommentCount(Long postId);
     }
     ```

   - CommentService

     ```java
     @Service
     public class CommentService {
       private final CommentCountUpdater commentCountUpdater;
       public CommentService(CommentCountUpdater commentCountUpdater) {
         this.commentCountUpdater = commentCountUpdater;
     }
       public void addComment(Long postId, String content) {
         // 댓글 저장 로직
         commentCountUpdater.updateCommentCount(postId);
       }
     }
     ```

2. 이벤트 기반 방법

   특정 이벤트가 발생했을 때 이를 감지하고 처리



#### 이벤트 시스템

- 구성 요소

  이벤스 소스: 이벤트를 발생시키는 컴포넌트로 주로 비즈니스 로직 내에서 특정 조건이 충족될 때 이벤트 생성

  이벤트 리스너: 이벤트가 발생했을 때 이를 처리하는 컴포넌트로 특정 이벤트 유형을 감지하고 필요한 작업 수행

  이벤트 퍼블리셔: 이벤트를 생성하고 리스너에게 전달하는 역할 (`ApplicationEventPublisher`)

- CustomEvent

  ```java
  // 1. 이벤트 정의
  @Getter
  public class CustomEvent extends ApplicationEvent {
      private String message;
  
      public CustomEvent(Object source, String message) {
          super(source);
          this.message = message;
      }
  }
  ```

- CustomEventListener

  ```java
  // 2. 이벤트가 발생했을 때 해야 할 일 구현
  @Component  // Bean 등록
  public class CustomEventListener {
  
      @EventListener
      public void handleCustomEvent(CustomEvent event) {
          System.out.println("event 발생! " + event.getMessage());
  
          // ex. 댓글 개수 1개 증가 (이벤트가 발생했을 때 해야 할 일)
      }
  }
  ```

- CustomEventPublisher

  ```java
  // 3. 이벤트 등록
  @Component
  @RequiredArgsConstructor
  public class CustomEventPublisher {
      // post-comment 예제일 경우 CommentService가 됨
      private final ApplicationEventPublisher applicationEventPublisher;
  
      // 댓글 등록
      public void publisherEvent(final String message) {
          // 댓글 등록 로직
  
          // 이벤트 실행
          CustomEvent customEvent = new CustomEvent(this, message);
          applicationEventPublisher.publishEvent(customEvent);
      }
  }
  ```



#### 비동기 이벤트 처리

`@EnableAsync`, `@Async` 어노테이션 이용



#### 트랜잭션 내 이벤트 게시 및 처리

```java
@Service
@RequiredArgsConstructor
public class ItemService {
    private final ItemRepository itemRepository;
    private final ApplicationEventPublisher applicationEventPublisher;

    @Transactional
    public void addItem(String name, int price) {
        //  아이템 저장
        Item item = new Item(name, price);
        itemRepository.save(item);
      TransactionSynchronizationManager.registerSynchronization(new TransactionSynchronization() {
            @Override
            public void afterCommit() { // commit이 될 때만 이벤트 발생
                ItemRegEvent itemRegEvent = new ItemRegEvent(this, name, price);
                applicationEventPublisher.publishEvent(itemRegEvent);
            }
        });
    }
}
```



### 웹 소켓 프로그래밍

웹 소켓: HTTP의 단점을 보완하여 클라이언트와 서버 간의 양방향 통신을 가능하게 하여 실시간 어플리케이션 개발에 유용

- WebSocketHandler

  ```java
  public class MyWebSocketHandler extends TextWebSocketHandler {
      @Override
      public void afterConnectionEstablished(WebSocketSession session) throws Exception {
          System.out.println("Connection established with session: " + session.getId());
      }
  
      @Override
      protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
          String payload = message.getPayload();  // 실제 메시지 (헤더나 메타 데이터 등을 제외한 메시지)
          System.out.println("Received message: " + payload);
          session.sendMessage(new TextMessage("Echo: " + payload));
      }
  
      @Override
      public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
          System.out.println("Connection closed with session: " + session.getId());
      }
  }
  ```

- WebSocketConfig

  ```java
  @Configuration
  @EnableWebSocket
  public class WebSocketConfig implements WebSocketConfigurer {
      @Override
      public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
          registry.addHandler(new MyWebSocketHandler(), "/ws").setAllowedOrigins("*");
      }
  }
  ```



#### 웹 소켓 채팅 구현

```java
public class MyWebSocketHandler extends TextWebSocketHandler {
    // 웹 소켓으로 접속한 세션을 관리하기 위한 저장소 생성
    private static final Set<WebSocketSession> sessions = Collections.synchronizedSet(new HashSet<>());

    @Override
    public void afterConnectionEstablished(WebSocketSession session) throws Exception {
        System.out.println("Connection established with session: " + session.getId());
        sessions.add(session);
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        String payload = message.getPayload();
        System.out.println("Received message: " + payload);

        synchronized (sessions) {
            for (WebSocketSession webSocketSession : sessions) {
                if (webSocketSession.isOpen()) {    // 세션이 열려있을 때만 전송
                    webSocketSession.sendMessage(new TextMessage(payload));
                }
            }
        }
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
        System.out.println("Connection closed with session: " + session.getId());
        sessions.remove(session);
    }
}
```
