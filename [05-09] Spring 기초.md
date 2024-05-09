## [05/09] Spring 기초



### 프레임워크

일종의 반제품

- 구성 요소

  프레임워크 코어 (Cold Spot): 변경되지 않고 반복적으로 재사용되는 부분 / 프레임워크에서 제공하는 라이브러리

  확장 포인트 (Hook Point): 어플리케이션을 구축할 때 사용할 확장 포인트 제공 (추상클래스나 인터페이스 형태)

  확장 모듈 (Hot Spot): 각 어플리케이션이 확장 포인트를 상속해서 어플리케이션만의 비즈니스를 구현하는 것

  메타 데이터: 환경설정 파일 (ex. XML, Java Config Class)



### Servlet

웹 애플리케이션에서 동적인 내용을 생성하고 제공하는 자바 프로그래밍 기술

정적인 컨텐츠는 클라이언트에게서 요청이 오면 서버에서 그 요청에 따라 파일이나 이미지를 보내주면 되지만, 동적인 컨텐츠의 경우 Servlet 필요

웹 서버에서 동적인 컨텐츠를 생성하고 이를 클라이언트에게 전달하는 역할

자바 클래스로 작성되어 있으며 클라이언트 요청에 따라 응답 생성



### WAS (Web Application Server)

서블릿 컨테이너를 포함하고 있음



### EJB (Enterprise JavaBeans)

Java 기반 엔터프라이즈 애플리케이션 개발을 위한 서버 측 컴포넌트 모델 및 기술

Java EE(Java Platform, Enterprise Edition) 스펙의 일부로 제공되며, 기업 환경에서 대규모 애플리케이션을 개발하기 위한 강력한 도구

- 유형

  1. **Session Bean**

     클라이언트의 요청을 처리하고 비즈니스 로직을 실행하는데 사용

     클라이언트의 상태를 유지할 수 있는 상태(Session)를 가질 수 있음

  2. **Entity Beans**

     영속적인 데이터

     데이터베이스와 상호 작용하고 영속성을 관리하는데 사용

  3. **Message-Driven Beans**

     비동기 메시지 처리를 위한 Bean

     JMS(Java Message Service)와 같은 메시징 시스템을 통해 메시지를 수신하고 처리하는 데 사용

분산 컴포넌트 기술과 트랜잭션 관리, 보안 등의 기능을 제공하여 엔터프라이즈 환경에서 안정적이고 확장 가능한 애플리케이션을 개발할 수 있도록 지원

비즈니스 로직과 데이터 액세스 레이어를 분리하고, 여러 클라이언트가 공유하는 중요한 로직을 중앙 집중화하여 개발 및 유지보수 단순화 가능



### POJO (Plain Old Java Object)

자바 개발에서 특정한 제약이나 프레임워크에 의존하지 않고 간단하게 작성된 객체

기존의 자바 객체를 POJO로 정의할 때, 특정한 인터페이스를 구현하거나 특정한 클래스를 상속받을 필요 X



### Spring Boot

- 실행 클래스

  ```java
  @SpringBootApplication
  public class DemoApplication {
      public static void main(String[] args) {
          SpringApplication.run(DemoApplication.class, args);
      }
  }
  ```

  - @SpringBootApplication 어노테이션

    스프링부트 어플리케이션의 기본적인 설정 자동 구성



### 스프링 코어 (DI, AOP)

- 컨테이너 (Container)

  인스턴스의 생명주기 관리

  생성된 인스턴스에게 추가적인 기능 제공

  ex. Tomcat



#### IoC (Inversion of Control, 제어의 역전)

Object(Bean)의 생성과 의존 관계 설정, 사용, 제거 등의 작업을 스프링 컨테이너가 담당

코드 대신 컨테이너가 제어권을 가지고 있기 때문



#### DI (의존 관계 주입, Dependency Injection)

각자의 클래스가 서로 의존하지 않고 독립적으로 동작할 수 있도록 의존성 주입자가 의존성을 각각 주입해주는 것

클래스 사이의 의존 관계를 Bean 설정 정보를 바탕으로 컨테이너가 자동으로 연결해주는 것

- 종류

  필드 주입: @Autowired 사용

  Setter 주입

  생성자 주입

- 장점

  인스턴스의 스코프, 생명주기 제어 가능, AOP 방식

- 사용되는 용어

  의존 객체(Dependency): 객체 A가 객체 B를 사용하는 경우, 객체 A가 객체 B에 의존

  의존 주입(Dependency Injection): 외부에서 의존 객체를 생성자, setter, 필드 등을 통해 주입

  Bean: DI를 사용하기 위해 생성되는 객체

  BeanFactory: Bean을 생성하고 관리하는 컨테이너

  Component: Bean을 생성하기 위한 어노테이션 중 하나 / 해당 클래스를 Bean으로 등록

