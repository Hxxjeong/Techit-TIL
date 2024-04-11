## [04/11] final, 예외 처리



## Final

### Final 클래스

다른 클래스가 상속받을 수 없게 하는 클래스

- 필요성

  불변성 보장, 상속 방지, 불변 클래스 생성, 성능 최적화

- 특성

  확장 불가능, 메소드 오버라이딩 방지



### Final 필드

한번 초기화되면 변경할 수 없는 필드

Setter 사용 불가, 생성자에 파라미터로 필요

- 필요성

  상수 정의, 불변 객체 생성, 스레드 안전성, 메모리 가시성 보장



### Final 메소드

한번 선언되면 하위 클래스에서 오버라이딩 불가

- 필요성

  메소드 무결정 보장, 오버라이딩 방지, 보안 강화



### 예시 코드

```java
public class Person {
  private final int id;   // Setter 사용 불가, 생성자에 파라미터로 필요
  private String name;

  public Person(int id, String name) {
      this.id = id;
      this.name = name;
  }

  public final void print() { // 오버라이딩 불가
      System.out.println("Parent");
  }
}
```



## 예외 처리 (Exception Handling)

프로그램 실행 중에 발생할 수 있는 예외에 대비하여 프로그램의 정상적인 흐름을 유지하고 예외 상황을 안전하게 처리하는 프로그래밍 기법

try-catch-finally 블록을 사용하여 구현

try-with-resource: try 구문에서 리소스 구현

```java
public class TryWithResourceExam {
  public static void main(String[] args) {
      // try 블록에서 미리 선언
      try(FileInputStream fis = new FileInputStream("abc")) {
          // 처리할 로직
      }
      catch (Exception e) {
          e.printStackTrace();
      }
  }
}
```

catch 문이 실행되면 try 문은 실행되지 않음

여러 개의 catch 문 작성 가능

- `printStackTrace()` (+ 예외 처리 메커니즘)

  주로 예외가 발생한 위치와 호출 스택 정보를 확인하기 위해 사용

  예외가 발생하면 해당 예외는 호출 스택(call stack)을 따라 상위 호출자로 전파됨 → 호출 스택 정보를 이용하여 예외가 발생한 위치와 호출 스택 정보를 출력 →  호출 스택의 맨 아래부터 시작하여 이전 호출자들의 호출 스택 정보를 순차적으로 출력

  - 호출 스택 (call stack)

    메소드 호출의 순서와 위치를 나타내는 정보

    예외가 발생한 메소드가 호출 스택의 맨 아래에 위치하고, 예외를 처리하지 않은 경우엔 예외는 이전 호출자로 전파



### 예외 종류

해당 클래스는 마지막 catch 문으로 작성해야 함

- 실행 시 체크하는 예외 (RuntimeException)

  메소드 시그니처에 예외를 명시적으로 선언하지 않아도 자동으로 호출 스택을 통해 전파

  ex. NullPointerException, ArrayIndexOutOfBoundsException, ArithmeticException

- 컴파일 시 체크하는 예외 (CheckedException)

  예외를 처리하지 않으면 컴파일 오류가 발생

  ex. IOException, SQLException, FileNotFoundException

![image](/Users/hj/Desktop/image.png)



### 예외 떠넘기기 (예외 전파, Exception Propagation)

throws 키워드 사용

```java
public class ThrowsExam {
  public static void main(String[] args) throws FileNotFoundException {
          FileInputStream file = new FileInputStream("abc");
  }
}
```