## [05/27-29] Logging, JPA



### 로깅 (Logging)

어플리케이션의 동작 상태를 이해하고 문제가 발생했을 때 원인을 분석하기 위해 필수적인 부분

SLF4J & Logback을 기본으로 사용하여 처리

- 라이브러리

  SLF4J, Logback, Log4j2

- 로깅 설정 방법

  1. appliaction.properties (application.yml)

     ```properties
     # 로깅 레벨 설정
     logging.level.root=WARN
     logging.level.org.springframework.web=INFO
     logging.level.com.yourcompany=DEBUG
     # 로그 파일 설정
     logging.file.name=app.log
     ```

  2. logback-spring.xml

     ```xml
     <!-- logback-spring.xml -->
     <?xml version="1.0" encoding="UTF-8"?>
     
     <configuration>
     
         <!-- Console Appender -->
         <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
             <encoder>
                 <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
             </encoder>
         </appender>
     
         <!-- A 클래스에 대한 로그 설정 -->
         <logger name="a" level="INFO">
             <appender-ref ref="CONSOLE" />
         </logger>
     
         <!-- Root Logger -->
         <root level="INFO">
             <appender-ref ref="FILE"/>
         </root>
     
     </configuration>
     ```



### JPA (Java Persistence API)

ORM(Object-Relational Mapping) 표준을 제공하는 API

객체 지향 모델을 관계형 데이터베이스 테이블에 매핑

![image-20250201190512622](/Users/hj/Library/Application Support/typora-user-images/image-20250201190512622.png)

- 구성 요소

  - EntityManager

    엔티티의 생명주기 관리

    트랜잭션 하나 당 메소드 하나

    EntityManagerFactory 인터페이스를 통해 생성

    - 메소드

      `persist(Obejct)`: 데이터베이스에 새 엔티티 저장

      `merge(Object)`: 변경된 엔티티를 데이터베이스와 동기화

      `remove(Object)`: 엔티티 삭제

      `find(Class, Object)`: 주어진 id로 데이터베이스에서 엔티티를 찾아 반환

  - Entity

    JPA 어노테이션을 사용하여 데이터베이스 테이블과 매핑

  - 영속성 컨텍스트 (Persistence Context)

    엔티티 인스턴스의 생명주기를 관리하는 환경

    트랜잭션 범위 내에서 엔티티를 저장하고 관리

  - EntityTransaction

    데이터베이스 트랜잭션 관리

    `begin()`, `commit()`, `rollback()`

- Hibernate

  인터페이스의 구현체

  - Session

    SessionFactory를 통해 생성

    단일 작업 단위를 수행하는 주체

    JPA의 EntityManager

    `save(Object)`, `update(Object)`, `delete(Object)`, `createQuery(String)`

- persistence.xml

  `<persistence>`: JPA 설정의 root element / 모든 persistence-unit 설정 포함

  `<persistence-unit>`: 데이터베이스 연결과 엔티티 관리를 위한 설정 그룹 정의

  persistence-unit(영속성 단위): 일련의 엔티티 클래스와 이들을 관리하기 위한 설정을 그룹화한 단위

  - ddl.auto 옵션

    create: 어플리케이션 실행 시 데이터베이스 스키마를 새로 생성

    update: 기존 스키마를 유지하면서 변경된 엔티티 구조에 따라 데이터베이스 스키마 업데이트

    create-drop: 어플리케이션 시작 시 스키마 생성, 종료 시 스키마 삭제

    validate: 어플리케이션 시작 시 엔티티와 데이터베이스 테이블의 스키마가 일치하는지 검증



### JPA 기본 개념

- CRUD 실습

  ```java
  import jakarta.persistence.EntityManager;
  import jakarta.persistence.EntityManagerFactory;
  import jakarta.persistence.Persistence;
  
  public class UserDAO {
      private EntityManagerFactory entityManagerFactory;
  
      // 유저 생성
      public void createUser(User user) {
          EntityManager entityManager = JPAUtil.getEmf().createEntityManager();
          try {
              entityManager.getTransaction().begin();
  
              entityManager.persist(user);
  
              entityManager.getTransaction().commit();
          } finally {
              entityManager.close();
          }
      }
  
      // 유저 조회
      public User findUser(Long id) {
          EntityManager entityManager = JPAUtil.getEmf().createEntityManager();
          try{
              User user = entityManager.find(User.class, id);
              entityManager.close();
              return user;
          } finally {
              entityManager.close();
          }
      }
  
      // 유저 수정
      public void updateUser(User user) { // parameter의 user는 영속성 컨텍스트 X
          EntityManager entityManager = JPAUtil.getEmf().createEntityManager();
          try {
              entityManager.getTransaction().begin();
  
              // parameter의 user를 존재한 영속성 컨텍스트와 merge
              entityManager.merge(user);
  
              entityManager.getTransaction().commit();
          } finally {
              entityManager.close();
          }
      }
  
      // 유저 삭제
      public void deleteUser(User user) {
          EntityManager entityManager = JPAUtil.getEmf().createEntityManager();
          try {
              entityManager.getTransaction().begin();
  
              // 영속성 컨텍스트에 연결되어 있는 항목 삭제
              entityManager.remove(entityManager.contains(user) ? user : entityManager.merge(user));
  
              entityManager.getTransaction().commit();
          } finally {
              entityManager.close();
          }
      }
  }
  ```



#### Entity

데이터베이스 테이블의 row에 대응되는 객체지향 모델

PK(`@Id` 이용)와 기본 생성자 필요

equals: id 기반 비교

영속성 컨텍스트가 Entity의 동일성(Identity, == 비교) 관리

- `@GenerationsType` strategy

  AUTO: JPA 구현체가 자동으로 적합한 전략 선택 / SEQUENCE, TABLE, IDENTITY 중 선택

  IDENTITY: DB의 ID 자동 생성 기능 사용 / auto increment

  SEQUENCE: 시퀀스를 사용하여 PK 생성

  TABLE: 키 생성을 위한 별도의 데이터베이스 테이블 사용

- 관계 매핑

  `@OneToMany`: 1:N 관계 매핑의 1 / mappedBy 속성으로 소유가 아닌 쪽 지정 (+ fetch, cascade)

  `@ManyToOne`: 1:N 관계 매핑의 N / `@JoinColumn`으로 어떤 컬럼을 통해 매핑되는지 명시 (+ fetch, 기본적으로 Eager Loading)



#### EntityManager

JPA에서 데이터베이스 작업을 관리하는 주요 인터페이스



#### Persistence Context

Entity를 영구 저장하는 환경

- 주요 기능

- 1차 캐시

  한 트랜잭션 내에서 반복된 데이터베이스 호출을 최소화

- Entity 생명주기 관리

  Entity의 상태(비영속, 영속, 삭제, 분리) 관리

  - Entity의 생명주기

    New, Managed, Detached, Removed

    메소드: `persist(Entity)`, `merge(Entity)`, `remove(Entity)`, `detach(Entity)`

    ![image-20250201190609779](/Users/hj/Library/Application Support/typora-user-images/image-20250201190609779.png)

- Dirty Checking (변경 감지)

  트랜잭션이 commit될 때 영속성 컨텍스트에 있는 Entity를 검사하여 변경된 Entity를 자동으로 데이터베이스에 반영

- Lazy Loading

  Entity와 연관된 객체들을 필요한 시점까지 로딩을 지연시켜 성능 최적화



### JPA 관계 매핑

#### `@ManyToMany`

두 Entity가 양방향으로 여러 요소와 관계를 맺을 수 있을 때 사용

```java
@Entity
@Getter
@Setter
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false)
    private String name;

    @ManyToMany
    @JoinTable(
            name = "employees_projects",
            joinColumns = @JoinColumn(name = "employee_id"),
            inverseJoinColumns = @JoinColumn(name = "project_id")
    )
    private Set<Project> projects = new HashSet<>();
}
```



##### `@JoinTable`

joinColumns: 현재 entity에서 조인 테이블로의 외래키 지정

inverseJoinColumns: 반대쪽 entity에서 조인 테이블로의 외래키 지정

```java
@Entity
@Getter
@Setter
@Table(name = "projects")
public class Project {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(nullable = false)
    private String name;

    @ManyToMany(mappedBy = "projects")
    private Set<Employee> employees = new HashSet<>();
}
```



#### `@OneToOne`

한 entity가 다른 entity와 하나의 관계만을 가질 때 사용

`@ManyToOne`, `@OneToMany`와 비슷하게 사용함 (관계가 1:N / 1:1의 차이)

외래키를 가진 쪽에서 항목 삭제 시 null로 처리해주고 삭제해야 함

```java
import jakarta.persistence.EntityManager;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class PersonExamMain {
    public static void delete(Long id) {
        EntityManager entityManager = JPAUtil.getEmf().createEntityManager();
        entityManager.getTransaction().begin();
        try {
            // passport 삭제
            Passport passport = entityManager.find(Passport.class, id);
            if(passport != null)
                passport.getPerson().setPassport(null); // person이 가지고 있는 passport를 null로 만들어야 함
            entityManager.remove(passport);

            entityManager.getTransaction().commit();
        } finally {
            entityManager.close();
        }
    }

    public static void main(String[] args) {
        delete(2L);
    }
}
```



### JPA 상속 매핑

객체 지향 모델에서의 상속 구조 관계형 데이터베이스 스키마에 매핑



#### SINGLE_TABLE (단일 테이블 전략)

상속 계층의 모든 클래스를 하나의 테이블에 매핑

```java
@Entity
@Getter
@Setter
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)   // 상속 관계
@DiscriminatorColumn(name = "type", discriminatorType = DiscriminatorType.STRING)   // column 이름과 type
public class Vehicle {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String manufacturer;
}

@Entity
@Getter
@Setter
@DiscriminatorValue("CAR") // column 값
class Car extends Vehicle {
    private int seatCount;
}

@Entity
@Getter
@Setter
@DiscriminatorValue("TRUCK")
class Truck extends Vehicle {
    private double payloadCapacity;
}
```

각 컬럼에서 Car의 해당하는 컬럼은 payloadCapacity가, TRUCK에 해당하는 컬럼은 seatCount가 null이 됨

- 테이블 구조

  ![image-20250201190656066](/Users/hj/Library/Application Support/typora-user-images/image-20250201190656066.png)



#### JOINED (조인 테이블 전략)

각 클래스를 별도의 테이블로 매핑하고 상속 관계에 있는 클래스 간 조인 사용

```java
@Entity
@Getter
@Setter
@Inheritance(strategy = InheritanceType.JOINED)   // 상속 관계
public class Vehicle2 {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String manufacturer;
}

@Entity
@Getter
@Setter
class Car2 extends Vehicle2 {
    private int seatCount;
}

@Entity
@Getter
@Setter
class Truck2 extends Vehicle2 {
    private double payloadCapacity;
}
```

부모의 field는 부모 테이블에, 자식의 field는 자식 테이블에 생성되면 부모의 PK를 FK로 가지고 있음

- 테이블 구조

  - Vehicle

    ![image-20250201190731834](/Users/hj/Library/Application Support/typora-user-images/image-20250201190731834.png)

  - Car

    ![image-20250201190742594](/Users/hj/Library/Application Support/typora-user-images/image-20250201190742594.png)

  - Truck

    ![image-20250201190751500](/Users/hj/Library/Application Support/typora-user-images/image-20250201190751500.png)



#### TABLE_PER_CLASS (테이블 당 구체 클래스 전략)

각 구체 클래스를 자신의 테이블로 매핑

부모의 PK에 `@GeneratedValue`에서 TABLE이나 SEQUENCE 전략 필요

```java
@Entity
@Getter
@Setter
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)   // 상속 관계
public class Vehicle3 {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE)
    private Long id;
    private String manufacturer;
}

@Entity
@Getter
@Setter
class Car3 extends Vehicle3 {
    private int seatCount;
}

@Entity
@Getter
@Setter
class Truck3 extends Vehicle3 {
```

- 테이블 구조

  Vehicle에는 속성만 가지고 데이터가 없음 (하위 entity에서 저장되기 때문)

  - Car

    ![image-20250201190827780](/Users/hj/Library/Application Support/typora-user-images/image-20250201190827780.png)

  - Truck

    ![image-20250201190833998](/Users/hj/Library/Application Support/typora-user-images/image-20250201190833998.png)

  - Hibernate_sequences

    해당 테이블을 통해 Car와 Truck의 id 지정 (Vehicle의 속성을 가지기 때문에 id가 중복되지 않아야 함)

    ![image-20250201190840431](/Users/hj/Library/Application Support/typora-user-images/image-20250201190840431.png)



### 임베디드 타입 (Embedded Type)

JPA에서 entity의 일부로 정의된 재사용 가능한 도메인 모델의 일부를 표현하는 데 사용

중복 없이 여러 entity 간 공통 구조 공유 가능

```java
@Embeddable // 공통적으로 사용하는 속성
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
    private String country;
}
@Entity
@Table(name = "companies")
@Getter
@Setter
@NoArgsConstructor
public class Company {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @Embedded   // 공통 속성 사용
    private Address address;

    public Company(String name, Address address) {
        this.name = name;
        this.address = address;
    }
}
```

- 테이블 구조

  ![image-20250201190857483](/Users/hj/Library/Application Support/typora-user-images/image-20250201190857483.png)
