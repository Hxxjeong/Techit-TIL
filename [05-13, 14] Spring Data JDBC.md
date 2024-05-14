## [05/13, 14] Spring Data JDBC



### JDBC vs. Spring Data JDBC

- JDBC (Java Database Connectivity): Java에서 DB에 접근할 수 있게 해주는 API로 SQL 쿼리 실행, DB 연결 관리 등의 기능 제공 / 보일러플레이트 코드 필요

- Spring Data JDBC: Spring 프레임워크의 일부로 JDBC의 복잡성을 추상화하고 간소화 함 / 템플릿 디자인 패턴을 사용하여 코드 중복을 줄임 / 예외를 일관된 방식으로 처리

  - DataSource

    DB 연결 관리

    .properties, .yml 파일을 통해 설정

  - JdbcTemplate

    SQL 쿼리 실행을 단순화하는 주요 클래스

    CRUD의 CUD - `update()`, R - `query()/queryForObject()` 메소드 사용

  - NamedParameterJdbcTemplate

    쿼리에서 이름 기반의 파라미터 사용 (JdbcTemplate의 확장)



### Spring JDBC 과정

1. DataSource Bean 생성

   application 설정 파일에서 정의된 데이터베이스 연결 정보 바탕

2. JdbcTemplate Bean 생성

   생성자 또는 성정을 통해 DataSource 객체를 주입받아 초기화

3. TransactionManager Bean 설정 (선택)

4. 쿼리 실행

   `SpringApplication.run()` 메소드 호출로 어플리케이션 시작

   결과 처리: RowMapper롤 통해 SQL 쿼리 결과를 자바 객체로 매핑



#### DAO (Data Access Object)

데이터베이스 또는 다른 저장소와의 상호작용을 캡슐화하는 디자인 패턴

- 구성 요소

  인터페이스, 클래스, DTO

```java
import java.util.List;

public interface UserDao {
    void createUser(User user);
    List<User> findAllUsers();
    void updateUserEmail(String name, String email);    // email update
    void deleteUser(String name);   // 이름을 기준으로 삭제
}
import lombok.RequiredArgsConstructor;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;

import java.sql.ResultSet;
import java.sql.SQLException;
import java.util.List;

@Repository
@RequiredArgsConstructor    // final 필드에 대해 생성자 추가
public class UserDaoImpl implements UserDao {   // DAO 인터페이스 구현
    // JdbcTemplate 객체 이용
    // 생성자 기반
    private final JdbcTemplate jdbcTemplate;

    @Override
    public void createUser(User user) {
        String sql = "insert into users(name, email) values (?, ?)";
        jdbcTemplate.update(sql, user.getName(), user.getEmail());
    }

    @Override
    public List<User> findAllUsers() {
        RowMapper<User> users = (ResultSet rs, int rowNum) -> new User(
                rs.getLong("id"),
                rs.getString("name"),
                rs.getString("email")
        );

        List<User> userList = jdbcTemplate.query("select * from users", users);

        return userList;
    }

    @Override
    public void updateUserEmail(String name, String email) {
        String sql = "update users set email = ? where name = ?";
        jdbcTemplate.update(sql, email, name);
    }

    @Override
    public void deleteUser(String name) {
        String sql = "delete from users where name = ?";
        jdbcTemplate.update(sql, name);
    }
}
```

- 데이터 접근 예외 처리

  DataAccessException 이용



#### Repository Interface

데이터 접근 계층(Data Access Layer) 단순화하는 패턴

`CrudRepository,` `JPARepository` 등을 구현하면 기본 제공 메소드가 있기 때문에 인터페이스 작성이 간편해짐

- 실습 코드 (+ 페이징 처리)

  Entity 생략

  - Repository

    ```java
    import org.springframework.data.repository.CrudRepository;
    
    import java.util.List;
    
    public interface UserRepository extends CrudRepository<User, Long>, CustomUserRepository {
        List<User> findByName(String name);
    }
    ```

    ```java
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    
    public interface CustomUserRepository {
        Page<User> findAllUsersWithPagination(Pageable pageable);
    }
    ```

    ```java
    import lombok.RequiredArgsConstructor;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.Pageable;
    import org.springframework.data.support.PageableExecutionUtils;
    import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
    
    import java.util.HashMap;
    import java.util.List;
    import java.util.Map;
    
    @RequiredArgsConstructor
    public class CustomUserRepositoryImpl implements CustomUserRepository {
        private final NamedParameterJdbcTemplate jdbcTemplate;
    
        @Override
        public Page<User> findAllUsersWithPagination(Pageable pageable) {
            String countQuery = "SELECT COUNT(*) FROM users";
            String query = "SELECT id, name, email FROM users LIMIT :limit OFFSET :offset";
    
            Map<String, Object> params = new HashMap<>();
            params.put("limit", pageable.getPageSize());
            params.put("offset", pageable.getOffset());
    
            List<User> users = jdbcTemplate.query(query, params, (rs, rowNum) ->
                    new User(rs.getString("name"), rs.getString("email")));
    
            // 페이징 처리: count를 구하는 쿼리와 결과를 가지고 오는 쿼리 모두 필요
            return PageableExecutionUtils.getPage(users, pageable,
                    () -> jdbcTemplate.queryForObject(countQuery, params, Long.class));
        }
    }
    ```

    ```java
    import org.springframework.boot.CommandLineRunner;
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import org.springframework.context.annotation.Bean;
    import org.springframework.data.domain.Page;
    import org.springframework.data.domain.PageRequest;
    
    import java.util.List;
    
    @SpringBootApplication
    public class MainApplication {
        public static void main(String[] args) {
            SpringApplication.run(MainApplication.class, args);
        }
    
        @Bean
        public CommandLineRunner demo(UserRepository userRepository) {
            return args -> {
                userRepository.save(new User("choi", "choi@email.com"));
    
                User user = userRepository.findById(8L).get();
                System.out.println(user.getName());
                List<User> users = userRepository.findByName("kim");
                users.forEach(u -> System.out.println(u.getName() + ": " + u.getEmail()));
    
                // 2건씩 페이징 처리하여 0페이지 조회 (페이지 0부터 시작)
                PageRequest pageRequest = PageRequest.of(0, 2);
                Page<User> allUser = userRepository.findAllUsersWithPagination(pageRequest);
    
                allUser.forEach(user -> System.out.println(user.getName() + ": " + user.getEmail()));
            };
        }
    }
    ```
