## [05/31] TDD, Criteria API



### TDD

- JUnit

  자바에서 단위 테스트를 작성하고 실행하기 위한 프레임워크

  주로 AssertJ의 assertThat 이용

  - 사용되는 어노테이션

    `@BeforeAll`: 테스트 클래스의 모든 테스트 메소드가 실행되기 전 한번 실행 / 메소드가 static이어야 함

    `@BeforeEach`: 각 테스트 메소드가 실행되기 전 실행

    `@AfterEach`: 각 테스트 메소드가 실행된 후 실행

    `@AfterAll`: 테스트 클래스의 모든 테스트 메소드가 실행된 후 한번 실행 / 메소드가 static이어야 함

- Mocking

  테스트 중에 의존성을 가짜 객체로 대체하여 테스트를 수행하는 기법

  ```java
  import static org.mockito.Mockito.*;
  import static org.junit.jupiter.api.Assertions.assertEquals;
  import org.junit.jupiter.api.Test;
  import org.mockito.Mockito;
  
  public class UserServiceTest {
  
      @Test
      public void testGetUserName() {
          // Mock 객체 생성
          UserRepository mockRepo = Mockito.mock(UserRepository.class);
  
          // Mock 객체의 동작 정의
          when(mockRepo.findUserById(1)).thenReturn(new User(1, "John Doe"));
  
          // UserService에 mock 객체 주입
          UserService userService = new UserService(mockRepo);
  
          // 테스트 수행
          String userName = userService.getUserName(1);
          assertEquals("John Doe", userName);
  
          // mock 객체와의 상호작용 검증
          verify(mockRepo).findUserById(1);
      }
  }
  ```



### Criteria API

JPA의 일부로서 프로그래밍 방식으로 type-safe한 쿼리를 구성할 수 있는 API

복잡한 검색 기능을 구현할 때 SQL이나 JPQL 문자열을 직접 작성하지 않고도 동적으로 쿼리를 생성하고 실행

동적 쿼리(Dynamic Query): 실행 시점에 쿼리의 구조가 결정되는 SQL 문장

- 구성 요소

  CriteriaBuilder: 쿼리를 생성하고 수정하는 데 사용되는 Factory Class

  CriteriaQuery: 생성할 쿼리 정의

  Root: 쿼리의 FROM 절에 해당하는 주 entity 지정

- Criteria API 사용

  ```java
  @Repository
  public class UserRepositoryCustomImpl implements UserRepositoryCustom {
  
      @Autowired
      private EntityManager entityManager;
  
      @Override
      public List<User> findUsersByCriteria(String name, String email) {
          CriteriaBuilder cb = entityManager.getCriteriaBuilder();
          CriteriaQuery<User> query = cb.createQuery(User.class);
          Root<User> user = query.from(User.class);
  
          List<Predicate> predicates = new ArrayList<>();
  
          if (name != null) {
              predicates.add(cb.equal(user.get("name"), name));
          }
          if (email != null) {
              predicates.add(cb.equal(user.get("email"), email));
          }
  
          query.where(cb.and(predicates.toArray(new Predicate[0])));
  
          return entityManager.createQuery(query).getResultList();
      }
  }
  ```

- 같은 코드를 JPQL로 작성

  ```java
  public interface UserRepository extends JpaRepository<User, Long> {
  
      @Query("SELECT u FROM User u WHERE (:name IS NULL OR u.name = :name) AND (:email IS NULL OR u.email = :email)")
      List<User> findUsersByNameAndemail(@Param("name") String name, @Param("email") String email);
  }
  ```

- Join

  Inner Join: 두 테이블의 교집합만 결과로 반환

  Outer Join: 한 테이블의 값이 다른 테이블에 일치하는 값이 없어도 해당 데이터를 포함하여 결과 반환 (mysql: full outer join이 없기 때문에 union 사용)

  Cross Join: 두 테이블의 카테시안 곱 반환

  Natural Join: 두 테이블 간 이름이 같은 모든 컬럼에 대해 암묵적인 inner join 수행

- Service

  각 메소드에서 `@Transactional`을 사용하는 것을 권장

- OSiV (Open Session in View)

  웹 어플리케이션에서 hibernate 같은 ORM 프레임워크를 사용할 때 DB 세션(or EntityManager)을 뷰 렌더링까지 유지하는 패턴

  뷰 레이어에서 Lazy Loading을 사용할 수 있게 됨
