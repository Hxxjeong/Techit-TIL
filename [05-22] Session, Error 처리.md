## [05/22] Session, Error 처리



### Session

#### HttpSession 이용

```java
import jakarta.servlet.http.HttpSession;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class SessionController {
    // HttpSession 직접 이용
    @GetMapping("/visit2")
    public String trackVisit(HttpSession session) {
        // 세션으로부터 방문 횟수 얻기
        Integer visitCount = (Integer) session.getAttribute("visitCount");

        if(visitCount == null) {    // 한번도 방문하지 않은 경우
            visitCount = 0; // 초기값 설정
        }
        visitCount++;
        session.setAttribute("visitCount", visitCount);

        return "visit2";
    }

    // 특정 세션 삭제
    @GetMapping("resetVisit")
    public String resetVisit(HttpSession session) {
        // session 객체에서 해당 속성만 초기화
        session.removeAttribute("visitCount2");
        session.setAttribute("visitCount", 0);
        session.invalidate(); // 세션 전체 정리
        return "redirect:/visit2";
    }
}
```



#### `@SessionAttribute` 어노테이션 이용

Session 속성을 Spring이 자동으로 관리하므로 SessionStatus 이용하여 Session 속성 제거

Controller 메소드가 호출될 때마다 Session에서 속성을 Model로 복원하기 때문

```java
import jakarta.servlet.http.HttpSession;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.SessionAttributes;
import org.springframework.web.bind.support.SessionStatus;

@Controller
@SessionAttributes("visitCount2")   // annotation 이용
public class SessionController {
    // 방문 횟수 초기화
    @ModelAttribute("visitCount2")
    public Integer initVisitCount() {
        return 0;
    }

    // 어노테이션 이용
    @GetMapping("/visit2")
    public String trackVisit(@ModelAttribute("visitCount2") Integer visitCount2, Model model) {
        visitCount2++;
        model.addAttribute("visitCount2", visitCount2);
        return "visit2";
    }

    // 세션 삭제
    @GetMapping("resetVisit")
    public String resetVisit(SessionStatus status) {
        status.setComplete();   // 사용자 세션 모두 초기화
        return "redirect:/visit2";
    }
}
```

- 볼만한 자료

  https://stackoverflow.com/questions/27191798/spring-sessionattributes-vs-httpsession

  https://velog.io/@woply/spring-서블릿이-제공하는-HttpSession-스프링이-제공하는-SessionAttribute



### 요청 처리

`@Requestmapping`: 요청 URL을 Controller의 특정 메소드에 매핑하는 데 사용 / HTTP 요청을 특정 메소드에 연결하여 해당 메소드가 요청 처리

#### 요청 데이터 처리

`@RequestParam`: URL 쿼리 파라미터나 form 데이터를 메소드 파라미터로 매핑

`@PathVariable`: URI의 일부를 변수로 캡처

`@RequestBody`: 요청 본문을 객체로 매핑 / 주로 JSON 또는 XML 데이터 처리에 사용

```java
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;

@Controller
public class RequestMappingController {
    // RequestMapping의 method는 GetMapping 형태로 쓸 수 있음
    @RequestMapping(value = "/ex", method = RequestMethod.GET)
    @ResponseBody
    public String getExample() {
        return "Get Test";
    }

    // url: localhost:8080/search?query={query} 형태
    @GetMapping("/search")
    @ResponseBody
    public ResponseEntity<String> Search(@RequestParam String query) {
        ResponseEntity<String> response = new ResponseEntity<>(query, HttpStatus.OK);
        return response;
    }

    @GetMapping("/guest/{name}")
    public String guest(@PathVariable String name) {
        System.out.println(name);
        return "redirect:/result";
    }
}
```



### View Resolver

컨트롤러에서 반환된 view 이름을 기반으로 실제 view를 찾아내고 렌더링을 위한 view 객체를 반환하는 역할

- Custom View

  ```java
  import jakarta.servlet.http.HttpServletRequest;
  import jakarta.servlet.http.HttpServletResponse;
  import org.springframework.web.servlet.View;
  
  import java.io.PrintWriter;
  import java.util.Map;
  
  public class MyCustomView implements View { // View
      @Override
      public String getContentType() {
          return "text/html";
      }
  
      @Override
      public void render(Map<String, ?> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
          response.setContentType(getContentType());
          PrintWriter writer = response.getWriter();
          writer.print("<html><body>");
          writer.print("<h2>Custom Page</h2>");
          writer.print("<p>This is my custom view.</p>");
          writer.print("</body></html>");
  
          writer.close();
      }
  }
  ```

- Custom View Resolver

  ```java
  import com.example.springmvc.view.MyCustomView;
  import lombok.Setter;
  import org.springframework.core.Ordered;
  import org.springframework.web.servlet.View;
  import org.springframework.web.servlet.ViewResolver;
  
  import java.util.Locale;
  
  @Setter
  public class MyCustomViewResolver implements ViewResolver, Ordered {
      private int order;
  
      @Override
      public int getOrder() {
          return this.order;
      }
  
      @Override
      public View resolveViewName(String viewName, Locale locale) throws Exception {
          if(viewName.startsWith("my-prefix")) {
              return new MyCustomView();
          }
          return null;    // 다음 view resolver가 처리
      }
  }
  ```

- Config

  ```java
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.web.servlet.config.annotation.EnableWebMvc;
  
  @Configuration
  @EnableWebMvc
  public class WebConfig {
      @Bean
      public MyCustomViewResolver myCustomViewResolver() {
          MyCustomViewResolver resolver = new MyCustomViewResolver();
          resolver.setOrder(0);   // 우선순위 설정 (가장 높은 순위 부여)
          return resolver;
      }
  }
  ```

- Controller

  ```java
  package com.example.springmvc.controller;
  
  import org.springframework.stereotype.Controller;
  import org.springframework.web.bind.annotation.GetMapping;
  
  @Controller
  public class MyViewController {
      @GetMapping("/custom")
      public String customView() {
          return "my-prefix-custom";
      }
  }
  ```



### 에러 페이지 처리

기존 WhiteLabel Error Page가 아닌 오류 페이지 설정

`@ControllerAdvice` 어노테이션을 사용하여 전역 예외 처리기 설정 / 모든 Controller에서 발생할 수 있는 예외를 처리하고 적절한 응답 생성

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;

@ControllerAdvice
public class GlobalExceptionHandler {
    // 로거 설정
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(Exception.class)
    public String handler(Exception e, Model model) {
        logger.error("error: ", e);
        model.addAttribute("errorMessage", e.getMessage());
        return "error";
    }
}
```
