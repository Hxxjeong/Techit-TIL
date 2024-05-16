## [05/16] Batch, JPA



### Batch 작업

Batch Update: 여러 개의 SQL 문을 하나의 Batch로 묶어서 실행하는 기법

`JdbcTemplate`의 `batchUpdate` 메서드 사용

```java
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.jdbc.core.BatchPreparedStatementSetter;
import org.springframework.jdbc.core.JdbcTemplate;

import java.sql.PreparedStatement;
import java.sql.SQLException;
import java.util.Arrays;
import java.util.List;

@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication.run(MainApplication.class, args);
    }

    @Bean
    public CommandLineRunner batchDemo(JdbcTemplate jdbcTemplate) {
        return args -> {
            List<User> users = Arrays.asList(
                    new User(null, "Alice", "alice@example.com"),
                    new User(null, "Bob", "bob@example.com"),
                    new User(null, "Charlie", "charlie@example.com"),
                    new User(null, "David", "david@example.com")
            );

            String sql = "insert into users (name, email) values (?, ?)";

            int[] updateCounts = jdbcTemplate.batchUpdate(sql, new BatchPreparedStatementSetter() {
                @Override
                public void setValues(PreparedStatement ps, int i) throws SQLException {
                    User user = users.get(i);
                    ps.setString(1, user.getName());
                    ps.setString(2, user.getEmail());
                }

                @Override
                public int getBatchSize() {
                    return users.size();
                }
            });

            System.out.println("Batch update completed. Number of rows affected: " + Arrays.toString(updateCounts));
        };
    }
}
```



### Transaction 관리

선언적 트랜잭션 관리: `@Transactional` 어노테이션 사용 / 에러 발생 시 자동으로 rollback

프로그래매틱 트랜잭션 관리: `TransactionTemplate` 사용 / 명시적으로 rollback 설정 필요

```java
import lombok.RequiredArgsConstructor;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Service;
import org.springframework.transaction.support.TransactionTemplate;

@Service
@RequiredArgsConstructor
public class UserService {
    private final JdbcTemplate jdbcTemplate;
    private final TransactionTemplate transactionTemplate;

    public void executeComplexBusinessProcess(String name, String email) {
        transactionTemplate.execute(status -> {
            jdbcTemplate.update("insert into users(name, email) values(?, ?)", name, email);
            if(email.contains("error")) {
                status.setRollbackOnly();   // rollback

                throw new RuntimeException("Simulated error to trigger rollback");
            }
            jdbcTemplate.update("update users set email = ? where name = ?", email, name);
            return null;
        });
    }
}
```



### RowMapper / ResultSetExtractor

- RowMapper

  SQL 쿼리의 결과를 Java 객체로 매핑하는 데 사용

  각 행을 객체로 변환

- ResultSetExtractor

  전체 ResultSet을 단일 호출에서 사용하여 복잡한 객체 구성

  여러 테이블에서 조인된 결과를 받아 하나의 통합 객체로 매핑

  특정 조건에 따라 다양한 타입의 객체를 리스트에 추가할 때 사용

- 외부 SQL 파일 관리

  SQL 쿼리를 외부 파일에 저장하여 이용

  ```java
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.core.io.ClassPathResource;
  import org.springframework.core.io.Resource;
  
  import java.io.IOException;
  import java.util.Properties;
  
  @Configuration
  public class SqlConfig {
      @Bean
      public Properties sqlQueries() throws IOException {
          Resource resource = new ClassPathResource("sql/queries.sql");
          Properties properties = new Properties();
          properties.load(resource.getInputStream());
          return properties;
      }
  }
  ```

  ```java
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
  import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
  import org.springframework.stereotype.Repository;
  
  import java.util.Properties;
  
  @Repository
  public class UserDao {
      @Autowired
      private NamedParameterJdbcTemplate jdbcTemplate;
      @Autowired
      private Properties sqlQueries;
  
      public void createUser(User user) {
          String sql = sqlQueries.getProperty("INSERT_USER");
  
          MapSqlParameterSource params = new MapSqlParameterSource();
          params.addValue("name", user.getName());    // key, 값
          params.addValue("email", user.getEmail());
  
          jdbcTemplate.update(sql, params);
      }
  }
  ```



### SimpleJdbcInsert vs. JPA

#### SimpleJdbcInsert

JdbcTemplate 기반 데이터 접근을 단순화하는 클래스

단순한 데이터 삽입과 자동 생성된 키 반환에 유용

```java
import jakarta.annotation.PostConstruct;
import lombok.RequiredArgsConstructor;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;
import org.springframework.stereotype.Repository;

import java.util.HashMap;
import java.util.Map;

@Repository
@RequiredArgsConstructor
public class UserDao {
    private final JdbcTemplate jdbcTemplate;

    // 입력이 되면 자동으로 생성된 PK 값 리턴
    private SimpleJdbcInsert simpleJdbcInsert;

    @PostConstruct
    public void init() {
        simpleJdbcInsert = new SimpleJdbcInsert(jdbcTemplate)
                .withTableName("users")
                .usingGeneratedKeyColumns("id");
    }

    public User createUser(User user) {
        Map<String, Object> params = new HashMap<>();
        params.put("name", user.getName()); // 컬럼명, 객체
        params.put("email", user.getEmail());

        Number pk = simpleJdbcInsert.executeAndReturnKey(params);
        user.setId(pk.longValue());
        return user;
    }
}
```



#### JPA

@Id 어노테이션을 이용하여 구현

auto increment인 경우 `@GeneratedValue(strategy = GenerationType.IDENTITY)` 어노테이션 사용

```java
@Entity
@Setter
@Getter
public class User {
    @Id // PK
    @GeneratedValue(strategy = GenerationType.IDENTITY) // auto increment
    private Long id;
    private String name;
    private String email;
}
// Repository 생략
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;

    public User createUser(String name, String email) {
        User user = new User();
        user.setName(name);
        user.setEmail(email);
        return userRepository.save(user);
    }
}
```
