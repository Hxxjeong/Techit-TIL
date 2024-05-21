## [05/21] Thymeleaf



### Thymeleaf 문법

- 특정 요소 출력

  th:each의 status 변수를 사용하여 index 범위 조정

  ```html
  <!DOCTYPE html>
  <html lang="en" xmlns:th="<http://www.thymeleaf.org/>">
  <head>
      <meta charset="UTF-8">
      <title>List</title>
  </head>
  <body>
  <h2>List Example</h2>
  <ul>
      <!-- status 변수 사용하여 인덱스 조건 설정 -->
      <li th:each="item, stat:${items}"
      th:if="${stat.index>=2 and stat.index<=3}"
      th:text="${item}"></li>
  </ul>
  </body>
  </html>
  ```

- 데이터 변환

  `#temporals` 내장 객체 사용

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
      <meta charset="UTF-8">
      <title>Datetime</title>
  </head>
  <body>
  <h2>Date and Time</h2>
  <p>
      Current Date:
      <span th:text="${#temporals.format(currentDate, 'yyyy-MM-dd')}"></span>
  </p>
  <p>
      Current DateTime:
      <span th:text="${#temporals.format(currentDateTime, 'yyyy-MM-dd HH:mm:ss')}"></span>
  </p>
  <p>
      Current Time:
      <span th:text="${#temporals.format(currentTime, 'HH:mm:ss')}"></span>
  </p>
  <p>
      Current Zoned DateTime:
      <span th:text="${#temporals.format(currentZonedDateTime, 'yyyy-MM-dd HH:mm:ss zzzz')}"></span>
  </p>
  </body>
  </html>
  ```



### form 테이터

- form 생성:`<form>` 태그 사용

  `th:action`: form 데이터를 제출할 서버의 URL 지정

  `th:object`: form 데이터와 연결된 Model 객체 지정

  `th:field`: form 객체의 각 필드와 입력 필드를 자동으로 바인딩

- 데이터 처리

  `@ModelAttribute` 어노테이션을 사용하여 form 데이터를 자바 객체에 매핑하고 처리

  어노테이션을 사용하면 form 데이터가 자동으로 해당 객체의 필드에 바인딩됨

  `@ModelAttribute` 어노테이션은 메소드 파라미터, 메소드 레벨에서 모두 사용 가능

- 유효성 검증

  `@Valid` 어노테이션을 사용하여 객체가 바인딩될 때 자동으로 검증 수행

  유효성 검사를 통과하지 못한 데이터는 `BindingResult` 객체에 오류로 등록됨

  - Validation Annotation

    `@NotEmpty`: 값이 입력되었는지 검사

    `@NotBlank:` 공백인지 검사

    `@NotNull`: null이 아닌지 검사

    `@Size`: 입력된 문자열의 최소/최대 크기 검사

    `@Pattern(regexp="정규식")`: 정규식에 일치하는지 검사

    `@Email`: 이메일 주소 문자열인지 검사

    .properties 파일에 메시지를 따로 설정 가능

    ```
    jakarta.validation.constraints.Size.message=크기가 {min} 이상 {max} 이하이어야 합니다.
    jakarta.validation.constraints.Email.message=이메일 주소가 바르지 않습니다.
    jakarta.validation.constraints.NotEmpty.message=필수 입력항목입니다.
    jakarta.validation.constraints.Max.message={value} 이하이어야 합니다.
    jakarta.validation.constraints.Min.message={value} 이상이어야 합니다.
    jakarta.validation.constraints.Pattern.message={regexp} 패턴과 일치해야 합니다.
    ```

```java
import com.example.springmvc.domain.User2;
import com.example.springmvc.domain.UserForm;
import jakarta.validation.Valid;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.validation.BindingResult;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;

@Controller
public class FormController {
    @GetMapping("/form")
    public String form(Model model) {
        model.addAttribute("userForm", new UserForm());
        return "form";
    }

    // 넘길 값이 많을 때 @ModelAttribute 유용
    // 전송된 form 데이터를 userForm 클래스 인스턴스에 바인딩
    @PostMapping("/submitForm")
    public String submit(@Valid @ModelAttribute("userForm") UserForm userForm,
                         BindingResult bindingResult) {
        if(bindingResult.hasErrors()) { // 유효성 검사에서 에러일 때
            return "form";
        }
        return "result";
    }
}
```



### forwarding vs. redirect

- Forward (서버 내 포워딩)

  클라이언트로부터 받은 요청을 서버 내의 다른 자원(Servlet, JSP)으로 전달

  요청의 URL이 변경되지 않음 (클라이언트는 forwarding이 일어났는지 알 수 없음)

  요청과 응답 객체가 보존되어 전달

- Redirect

  서버가 클라이언트에게 새로운 위치로 요청을 재지시하는 방법

  브라우저의 URL이 변경됨

  요청과 응답 객체가 새로 생성됨

```java
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;

@Controller
public class RoutingController {
    @GetMapping("/start")
    public String start(Model model) {
        model.addAttribute("forwardTest", "hello!!");
        return "forward:/forward";  // forwarding 명시
    }

    @GetMapping("/forward")
    public String forward() {
        return "forwardPage";
    }

    @GetMapping("/redirect")
    public String redirect(Model model) {
        // redirect 되면 return 된 페이지에서 model 객체를 찾음 -> 페이지에 출력 X
        model.addAttribute("redirectTest", "hello~~");
        return "redirect:/finalDestination";
    }

    @GetMapping("/finalDestination")
    public String finalDestination() {
        return "finalPage";
    }
}
```



### Cookie & Session

요청이 바뀌어도 유지할 정보 저장 (Cookie: 브라우저에 저장 / Session: 서버에 저장)

Session은 Cookie가 있어야 동작 가능

- Cookie

  서버가 사용자의 웹 브라우저에 저장하는 작은 데이터 조각

  클라이언트가 서버에 요청할 때마다 데이터가 요청과 함께 서버로 전송됨

  사용자의 여러 정보를 저장하므로 보안에 취약할 수 있음

- Session

  서버 측에서 사용자와 관련된 데이터 저장

  세션 ID를 통해 각 클라이언트 식별

  서버 메모리에 데이터를 저장하기 때문에 쿠키보다 보안이 좋음
