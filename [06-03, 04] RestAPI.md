## [06/03, 04] RestAPI



### `@Contoller` vs `@RestController`

- `@Controller`

  웹 어플리케이션에서 HTTP 요청을 처리하는 역할

  view 템플릿 반환

- `@RestController`

  `@Controller` + `@ResponseBoby`

  RESTful 웹 서비스를 쉽게 만들 수 있도록 함

  - RESTful 서비스의 주요 원칙

    자원(Resource) 기반: 모든 콘텐츠는 자원으로 표현되며, 자원은 URI로 식별

    무상태(Stateless): 각 요청은 독립적이며 서버는 클라이언트의 상태 정보 저장 X

    연결성(Connectivity): 자원은 하이퍼링크를 통해 서로 연결될 수 있음

    표준 메소드 사용: HTTP 표준 메소드를 사용하여 리소스에 대한 CRUD 작성 수행

  클래스가 HTTP 요청을 처리하고 데이터를 JSON이나 XML 형태로 클라이언트에게 직접 반환

  - JSON 변환 과정
    1. 클라이언트는 URL로 HTTP 요청을 보냄
    2. DispatcherServlet은 요청을 받아 HandlerMapping을 사용하여 요청 URL에 해당하는 핸들러를 찾음
    3. HandlerMapping은 요청을 처리할 Controller의 메소드에 대한 정보를 DispatcherServlet에 반환
    4. DispatcherServlet은 HandlerAdapter를 통해 실제 메소드 호출
    5. Controller의 메소드는 요청을 처리하고 반환
    6. HandlerAdapter는 반환된 데이터를 DispatcherServlet에 반환
    7. DispatcherServlet은 HttpMessageConverter를 사용하여 반환된 데이터를 JSON 형식으로 변환
    8. 변환된 JSON 데이터는 DispatcherServlet을 토애 클라이언트에게 반환



### curl

https://chocolatey.org/install

- powershell에서 다음 명령어 실행

  ```powershell
  Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('<https://community.chocolatey.org/install.ps1>'))
  ```

  choco install curl



### 실습 코드

- Service

  ```java
  import com.example.restexam.domain.Todo;
  import com.example.restexam.repository.TodoRepository;
  import lombok.RequiredArgsConstructor;
  import org.springframework.data.domain.Sort;
  import org.springframework.stereotype.Service;
  import org.springframework.transaction.annotation.Transactional;
  
  import java.util.List;
  import java.util.NoSuchElementException;
  
  @Service
  @RequiredArgsConstructor
  @Transactional(readOnly = true)
  public class TodoService {
      private final TodoRepository todoRepository;
  
      // 전체 리스트
      public List<Todo> getTodos() {
          return todoRepository.findAll();
          // 정렬하는 경우
  //        return todoRepository.findAll(Sort.by("id").descending());
      }
  
      // todo 1개
      public Todo getTodo(Long id) {
          Todo todo = todoRepository.findById(id)
                  .orElseThrow(() -> new NoSuchElementException("해당하는 id가 없습니다."));
          return todo;
      }
  
      // create
      @Transactional
      public Todo createTodo(String todo) {
          Todo newTodo = todoRepository.save(new Todo(todo));
          return newTodo;
      }
  
      // update (id로 받기)
      // id에 해당하는 done toggle
      @Transactional
      public Todo updateTodo(Long id) {
          Todo todo = todoRepository.findById(id)
                  .orElseThrow(() -> new NoSuchElementException("해당하는 id가 없습니다."));
          todo.setDone(!todo.isDone());
          return todo;    // findById로 찾았기 때문에 영속성 컨텍스트가 관리하게 되여 save를 사용하지 않아도 ok
      }
  
      // update (todo로 받기)
      // id에 해당하는 todo 정보 수정
      @Transactional
      public Todo updateTodo(Todo todo) {
          Todo todo1 = null;
          try {
              todo1 = todoRepository.save(todo);
          }
          catch (Exception e) {
              e.printStackTrace();
          }
          return todo1;
      }
  
      // delete
      @Transactional
      public void deleteTodo(Long id) {
          Todo todo = todoRepository.findById(id)
                  .orElseThrow(() -> new NoSuchElementException("해당하는 id가 없습니다."));
          todoRepository.delete(todo);
      }
  }
  ```

- Controller

  ```java
  import com.example.restexam.domain.Todo;
  import com.example.restexam.service.TodoService;
  import lombok.RequiredArgsConstructor;
  import org.springframework.http.HttpStatus;
  import org.springframework.http.HttpStatusCode;
  import org.springframework.http.ResponseEntity;
  import org.springframework.web.bind.annotation.*;
  
  import java.util.List;
  
  @RestController
  @RequestMapping("/api/todos")
  @RequiredArgsConstructor
  public class TodoController {
      private final TodoService todoService;
  
      @GetMapping
      public List<Todo> getTodos() {
          return todoService.getTodos();
      }
  
      @GetMapping("/{id}")
      public Todo getTodo(@PathVariable("id") Long id) {
          return todoService.getTodo(id);
      }
  
      @PostMapping
      public Todo createTodo(@RequestBody Todo todo) {
          return todoService.createTodo(todo.getTodo());
      }
  
      @PatchMapping("/{id}")
      public Todo updateById(@PathVariable("id") Long id) {
          return todoService.updateTodo(id);
      }
  
      @PatchMapping
      public Todo updateTodo(@RequestBody Todo todo) {
          return todoService.updateTodo(todo);
      }
  
      @DeleteMapping("/{id}")
      public ResponseEntity<?> deleteTodo(@PathVariable("id") Long id) {
          todoService.deleteTodo(id);
          return new ResponseEntity<>(HttpStatus.NO_CONTENT);
      }
  }
  ```



### REST API 데이터 처리

`implementation 'com.fasterxml.jackson.dataformat:jackson-dataformat-xml'` 추가

- JSON

  Jackson 라이브러리를 사용하여 JSON 데이터를 자바 객체로 Serialize하거나 Deserialize

  `@RestController`나 `@ResponseBody` 어노테이션이 붙은 메소드에서 자동으로 처리

- XML

  JAXB 라이브러리를 사용하여 자바 객체와 XML 사이 매핑

  `@XmlRootElement`, `@XmlElement` 등의 어노테이션을 사용하여 클래스와 필드를 XML 요소에 매핑

```java
@RestController
public class MyRestController {
    // JSON
    @GetMapping(value = "/product/json/{id}", produces = "application/json")
    public Product getProductAsJson(@PathVariable("id") Long id) {
        return new Product(id, "pen", 10.0);
    }

    // XML
    @GetMapping(value = "/product/xml/{id}", produces = "application/xml")
    public Product getProductAsXml(@PathVariable("id") Long id) {
        return new Product(id, "apple", 6.44);
    }
}
```

Content Negotiation: Accpet 헤더를 기반으로 클라이언트가 요청한 데이터 형식에 따라 응답 형식 결정 (`@RequestMapping`의 produces 속성 이용)



### ResponseEntity

Spring MVC에서 HTTP 요청에 대한 응답을 구성할 때 사용

HTTP 상태 코드, 헤더, 응답 본문을 포함하여 HTTP 응답을 전체적으로 제어할 수 있게 해줌

```java
@RestController
@RequestMapping("/api/memos")
public class MemoRestController {
    private final Map<Long, String> memo = new HashMap<>();
    private final AtomicLong counter = new AtomicLong();    // id 자동 증가

    @PostMapping
    public ResponseEntity<Long> createMemo(@RequestBody String content) {
        Long id = counter.incrementAndGet();
        memo.put(id, content);  // service logic
        return ResponseEntity.ok(id);
    }

    @GetMapping("/{id}")
    public ResponseEntity<String> getMemo(@PathVariable("id") Long id) {
        String m = memo.get(id);
        if(m == null) return ResponseEntity.notFound().build();
        return ResponseEntity.ok(m);
    }

    @PutMapping("/{id}")
    public ResponseEntity<String> updateMemo(@PathVariable("id") Long id,
                                             @RequestBody String content) {
        if(!memo.containsKey(id)) return ResponseEntity.notFound().build();
        memo.put(id, content);
        return ResponseEntity.ok(content);
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<String> deleteMemo(@PathVariable("id") Long id) {
        String removeMemo = memo.remove(id);
        if(removeMemo == null) return ResponseEntity.notFound().build();
        return ResponseEntity.ok(removeMemo);
    }
}
```



### 파일 업로드 / 다운로드

`MultipartFile` 인터페이스를 통해 구현

```java
@RestController
public class FileController {
    // 파일 업로드
    @PostMapping("/upload")
    public ResponseEntity<String> fileUploadHandler(@RequestParam("file")MultipartFile file,
            @RequestPart("info")UploadInfo uploadInfo) {
        //
        System.out.println(file.getOriginalFilename()+"================");
        System.out.println(uploadInfo.getDescription()+"==========");
        System.out.println(uploadInfo.getTag()+"===============");

        String message = "";
        try {
            InputStream inputStream = file.getInputStream();
            StreamUtils.copy(inputStream, new FileOutputStream("C:/Techit/tmp/file/" + file.getOriginalFilename()));    // 서버의 저장 위치
            return ResponseEntity.ok().body(message);
        }
        catch (IOException e) {
            message = "FAIL UPLOAD!!: " + file.getOriginalFilename();
            return ResponseEntity.badRequest().body(message);
        }
    }

    // 파일 다운로드
    @GetMapping("/download")
    public void downloadFile(HttpServletResponse response) {
        Path filePath = Paths.get("c:/Techit/tmp/file/cat.jpg");
        response.setContentType("image/jpeg");

        try {
            InputStream inputStream = Files.newInputStream(filePath);
            StreamUtils.copy(inputStream, response.getOutputStream());  // 입력 스트림에서 출력 스트림으로 데이터 복사
            response.flushBuffer();
        }
        catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

------

https://www.youtube.com/watch?v=RP_f5dMoHFc
