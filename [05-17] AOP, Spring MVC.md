## [05/17] AOP, Spring MVC



### AOP (Aspect Oriented Programming)

기존의 OOP 언어를 보완하는 확장 형태 (관점 지향 프로그래밍)

관심의 분리 (Seperation of Concerns) = 핵심 관점(업무 로직) + 횡단 관점(트랜잭션/로그/보안/인증 처리 등)

ex. AspectJ, JBossAOP, SpringAOP



### AOP 용어

#### Joinpoint

어플리케이션을 실행할 때 특정 작업이 실행되는 시점

- 표현식의 주요 구성 요소

  execution: 메소드 실행을 기준으로 매칭 / 메소드 실행 시 Joinpoint를 찾는 데 사용

  - execution 표현식 형식

    `execution(modifiers-pattern? ret-type-pattern declaring-type-pattern? name-pattern(param-pattern) throws-pattern?)`

    modifiers-pattern: 접근 제어자 (생략 가능)

    ret-type-pattern: 반환 타입 / 와일드카드 * 사용 가능

    declaring-type-pattern: 메소드를 포함하는 클래스 또는 인터페이스

    name-pattern: 메소드 이름 / 와일드카드 * 사용 가능

    param-pattern: 메소드 매개변수 / `(..)`를 사용하여 모든 매개변수 지정 가능

    throws-pattern: 메소드가 던질 수 있는 예외

  within: 특정 타입에 속한 조인 포인트를 매칭하는 데 사용

  this: Bean 참조가 주어진 타입의 인스턴스인 Joinpoint 매칭

  target: 대상 객체가 주어진 타입의 인스턴스인 Joinpoint 매칭

  args: 메소드가 주어진 타입의 인수를 갖는 Joinpoint 매칭

  @annotation: 특정 어노테이션이 적용된 메소드를 Joinpoint로 매칭

  @within: 지정된 어노테이션이 타입 수준에서 선언된 객체의 모든 Joinpoint를 매칭

  @target: 지정된 어노테이션이 대상 객체에 적용된 Joinpoint 매칭

  @args: 전달된 인수에 어노테이션이 있는 메소드를 Joinpoint로 매칭



#### Advice

Joinpoint에서 실행되어야 하는 코드

횡단 관점에 해당

- 유형

  비즈니스 사이에 실행하는 유형은 제공되지 않음

  Before: Joinpoint 전에 실행

  After Returning: Joinpoint가 정상적으로 종료한 후에 실행 (예외가 발생하면 실행 X)

  After Throwing: Joinpoint에서 예외가 발생했을 때 실행 (정상적으로 종료하면 실행 X)

  After: Joinpoint에서 처리가 완료된 후 실행

  Around: Joinpoint 전후에 실행



#### Target

실질적인 비즈니스 로직을 구현하고 있는 코드

핵심 관점에 해당



#### Pointcut

Target 클래스와 Advice가 Weaving(결합)될 때 둘 사이의 결합 규칙을 정의

ex. Advice가 실행된 Target의 특정 메소드 등을 지정

- Weaving

  Jointpoint를 Advice로 감싸는 과정



#### Aspect

Advice + Pointcut

일정한 패턴을 가지는 클래스에 Advice를 적용하도록 지원할 수 있는 것



### Spring AOP

Spring은 Aspect의 적용 대상(target)이 되는 객체에 대한 Proxy를 만들어 제공

- @Aspect

  클래스 레벨에 @Order를 명시하여 작동 순서를 정할 수 있음

  @Before, @After, @AfterReturning, @AfterThrowing, @Around 명시 가능

  ```java
  import org.aspectj.lang.JoinPoint;
  import org.aspectj.lang.ProceedingJoinPoint;
  import org.aspectj.lang.annotation.*;
  import org.springframework.stereotype.Component;
  
  // Advice + Pointcut
  @Aspect
  @Component  // Bean 등록 필요
  public class ServiceAspect {
      @Pointcut("execution(* com.example.aopexam.SimpleService.*(..))")
      public void pc(){}
  
      @Pointcut("execution(* hello())")
      public void pc2(){}
  
      @Before("execution(* com.example.aopexam.SimpleService.*(..))") // pointcut 지정
      // advice
      public void before(JoinPoint joinPoint) {
          System.out.println("Before :::: " + joinPoint.getSignature().getName());    // mehtod name
      }
  
      @After("execution(* com.example.aopexam.SimpleService.*(..))")
      public void after() {
          System.out.println("After ::::");
      }
  
      @AfterReturning(pointcut = "execution(* com.example.aopexam.*.*(..))", returning = "result")
      public void afterReturningServiceMethod(JoinPoint joinPoint, Object result) {
          System.out.println("After method: " + joinPoint.getSignature().getName() + ", return value: " + result);
      }
  
      @Before("execution(* setName(String))")
      public void beforeName(JoinPoint joinPoint) {
          System.out.println("beforeName :::: ");
          System.out.println(joinPoint.getSignature().getName());
          Object[] args = joinPoint.getArgs();
          for(Object o: args) System.out.println(":::: 인자 --> " + o);
      }
  
      @AfterThrowing(value = "pc2()", throwing = "ex")
      public void afterThrowing(Throwable ex) {
          System.out.println("AfterThrowing ::::");
          System.out.println("exception value ::" + ex);
      }
  
      @Around("pc()")
      public String around(ProceedingJoinPoint pjp) throws Throwable {
          System.out.println("Around run :::: 실제 호출된 메소드가 실행되기 전에 할 일 구현");
  
          String value = (String) pjp.proceed();// 실제 target 메소드 호출
  
          System.out.println("Around run :::: 실제 호출된 메소드가 실행된 후 할 일 구현");
          value += " aop run!";
  
          return value;
      }
  }
  ```



### Spring MVC

Model-View-Controller 패턴을 기반으로 하는 웹 어플리케이션 프레임워크



#### MVC 디자인 패턴의 흐름

![image-20250201185551705](/Users/hj/Library/Application Support/typora-user-images/image-20250201185551705.png)

Imcoming request: 웹 요청

Front Controller: 모든 요청을 적절한 Handler나 Controller로 Routing하는 중앙 집중식 Controller (일반적으로 DispatcherServlet)

Controller: 필요한 데이터를 처리하고 비즈니스 로직을 실행한 후 결과 데이터 Model 객체에 저장

Model: Controller가 생성한 데이터를 포함

Delegate request of responese: Model 데이터를 바탕으로 응답을 렌더링할 View를 찾기 위해 View template로 요청 위임

View template: Model 데이터를 사용하여 최종 사용자에게 보여줄 HTML 페이지 등의 응답 생성



#### Spring MVC 프레임워크 요청 처리

![image-20250201185616195](/Users/hj/Library/Application Support/typora-user-images/image-20250201185616195.png)
