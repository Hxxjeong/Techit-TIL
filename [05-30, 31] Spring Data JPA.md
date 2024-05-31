## [05/30, 31] Spring Data JPA



### Spring Data JPA

데이터 액세스 계층을 쉽게 구현하고 관리할 수 있게 해주며, 복잡한 쿼리를 간단하게 처리하고 데이터베이스 작업을 자동화하는 모듈

- JDBC vs JPA

  JDBC: 도메인 모델을 데이터베이스에 1:1로 매핑

  JPA: Java Persistence API를 사용하여 도메인 모델과 데이터베이스 간의 복잡한 매핑과 데이터 관리 처리



##### 쿼리 메소드

- 메소드 이름을 사용하여 쿼리를 자동으로 생성 (Query Creation)

  | **Keyword**                        | **Sample**                                                   |
  | ---------------------------------- | ------------------------------------------------------------ |
  | And                                | findByLastNameAndFirstName(String lastName, String firstName) |
  | Or                                 | findByLastNameOrFirstName(String lastName, String firstName) |
  | Between                            | findByStartDateBetween(Date date1, Date date2)               |
  | LessThan                           | findByAgeLessThan(int age)                                   |
  | GreaterThan                        | findByAgeGreaterThan(int age)                                |
  | LessThanEqual                      | findByAgeLessThanEqual(int age)                              |
  | GreaterThanEqual                   | findByAgeGreaterThanEqual(int age)                           |
  | After                              | findByStartDateAfter(Date date1)                             |
  | Before                             | findByStartDateBefore(Date date1)                            |
  | IsNull                             | findByAgeIsNull()                                            |
  | IsNotNull, NotNull                 | findByAgeNotNull(), findByAgeIsNotNull()                     |
  | Like                               | findByFirstNameLike(String pattern)                          |
  | pattern값의 예: "김%" "%홍%","%진" |                                                              |
  | NotLike                            | findByFirstNameNotLike(String pattern)                       |
  | StartingWith                       | findByFirstNameStartingWith(String name)                     |
  | name 값의 예: "김"                 |                                                              |
  | EndingWith                         | findByFirstNameEndingWith(String name)                       |
  | name 값의 예: "진"                 |                                                              |
  | Containing                         | findByFirstNameContaining(String name)                       |
  | name 값의 예: "홍"                 |                                                              |
  | OrderBy                            | findByAgeOrderByLastNameDesc(int age)                        |
  | Not                                | findByLastNameNot(String name)                               |
  | In                                 | findByAgeIn(Collection<Integer> ages)                        |
  | NotIn                              | findByAgeNotIn(Collection<Integer> age)                      |
  | True                               | findByActiveTrue()                                           |
  | False                              | findByActiveFalse()                                          |
  | Top                                | findTop10ByOrderByAge()                                      |
  | 조회 결과의 선두 10개만 리턴       |                                                              |

- `@Query` 어노테이션 사용

  Repository 메소드에 JPQL(Java Persistence Query Language) 또는 SQL 쿼리 직접 정의

  `@Modifying` 어노테이션과 함께 수정/삭제 기능 구현 가능

  메소드의 파라미터를 쿼리에 바인딩할 때 `@Param` 어노테이션 사용 가능 (위치 기반 / 이름 기반)

  - JPQL vs SQL

    JPQL: entity 객체를 대상으로 하는 쿼리 언어 / entity 클래스와 필드를 기반으로 쿼리 작성

    SQL: 데이터베이스에 직접 작성하는 네이티브 쿼리 언어



#### JPA

- 네이티브 쿼리

  JPQL이나 Criteria API 외에 특정 데이터베이스 구문을 직접 사용해야 할 경우 사용

  `@Query` 어노테이션에서 nativeQuery를 true로 설정

  ```java
  @Query(value = "select name, email from jpa_user where name like %:name%", nativeQuery = true)
      List<Object[]> findUserByNameNative(@Param("name") String name);
  ```

- Entity 생성

  `@Temporal(TemporalType.DATE)`: Date 타입에서 사용 (LocalDate에서는 어노테이션 필요 X)

  `@ManyToOne` / `@OneToMany`: ERD 보고 1:N 관계 파악 (주로 FK가 있는 쪽이 N)

- Fetch

  entity 간의 관계를 로드하는 방법을 제어

  - Lazy Fetching (지연 로딩)

    관계된 entity를 실제 필요할 때까지 로드 X

    장점: 성능 최적화에 유리, 메모리 사용량 감소 가능

    단점: 관련 entity에 접근할 때마다 추가 쿼리 발생 가능, 트랜잭션 범위 밖에서 지연 로딩된 entity에 접근할 때 예외가 발생할 수 있음

    관계 매핑: `@OneToMany`, `@ManyToMany`

  - Eager Fetching (즉시 로딩)

    entity가 로드될 때 관계된 entity도 함께 로드됨

    장점: 다수의 쿼리 실행을 피할 수 있음, 지연 로딩 관련 예외가 발생하지 않음

    단점: 메모리 사용량이 많아질 수 있으며, 불필요한 데이터를 가져오는 데 시간이 소요될 수 있음

    관계 매핑: `@ManyToOne`, `@OneToOne`

  FETCH JOIN: 여러 entity를 한번에 가져올 때 사용 → N+1 문제 해결

  ```java
      // JOIN FETCH 사용 (Employee와 Department 사이 관계가 즉시 로드(Eager Fetching))
      @Query("SELECT e FROM Employee e JOIN FETCH e.department d WHERE d.departmentId IN :departmentIds AND e.salary BETWEEN :minSalary AND :maxSalary")
      List<Employee> findByDepartmentIdInAndSalaryBetween(@Param("departmentIds") List<Integer> departmentIds, @Param("minSalary") Double minSalary, @Param("maxSalary")Double maxSalary);
  ```

  - N+1 문제?

    SQL 결과가 N개 있을 때, 쿼리가 추가적으로 실행되는 경우

    > **EAGER 전략으로 데이터를 조회하는 경우**
    >
    > 1. JPQL에서 만든 SQL을 통해 데이터를 조회 (모든 부서 로드)
    > 2. 해당 데이터의 연관 관계인 하위 엔티티들을 추가 조회 (각 부서마다 해당 부서에 속한 직원 찾기)
    > 3. 2번 과정을 통해 N+1문제 발생

    > **LAZY 전략으로 데이터를 가져온 이후에 연관 관계인 하위 엔티티를 다시 조회하는 경우**
    >
    > 1. JPQL에서 만든 SQL을 통해 데이터를 조회 → 지연 로딩이기 때문에 추가 조회는 하지 않음
    > 2. 하위 엔티티를 가지고 작업하게 되면 추가 조회가 발생하기 때문에 결국 N + 1 문제 발생

    - 해결 방법

      Fetch Join 사용

      Batch Size 설정: 한번에 N개의 entity 미리 로드

      Entity Graph 사용

