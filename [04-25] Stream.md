## [04/25] Stream



### 스트림

데이터의 추상화된 연속된 흐름

데이터 소스에서 데이터를 읽어 처리하고 결과를 생성하는 메커니즘

- 생성

  컬렉션에서 생성: `컬렉션 변수.stream()`

  배열에서 생성: `Arrays.stream(배열)`

  스트림의 정적 메소드 사용: `Stream.of(...)`

  파일로부터 스트림 생성: `java.nio.file.Files` 클래스 사용

  빌더를 사용한 스트림 생성: `Stream.builder()`

중간 연산: Stream 리턴 (ex. filter, map, sorted, distinct)

최종 연산: 다른 값 반환 or void (ex. forEach, collect, reduce, match)

- Optional

  null을 사용하지 않고 부재 값을 포함한 데이터를 저장하는 클래스

  `orElse(T)`: 값이 없을 때 디폴트 값 지정



### 메소드

#### 필터링

일부 원소를 걸러내는 중간 연산

`filter(Predicate p)`: 주어진 조건에 맞는 요소 추출

`distinct()`: equals 기반 동등성 판단

`limit()`, `skip()`

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class FilteringExam {
        int[] arr = {1, 2, 3, 4, 5, 6, 7, 8};
        // 짝수만 출력
        Arrays.stream(arr)
            .filter(i -> i%2==0).forEach(System.out::println);
    }
}
```



#### 데이터 변환 (매핑)

`map(Function f)`: 스트림의 각 요소를 함수에 따라 다른 형태로 변환

`flatMap(Function f)`: 하나의 스트림으로 합침, 중첩된 구조 평탄화

`boxed()`: 기본형 자료를 참조형으로 변환하는 박싱

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class MapExam {
    public static void main(String[] args) {
        int[] arr = {3, 5, 49, 1, 22};
        // 각 요소에 3을 곱해서 출력
        Arrays.stream(arr)
                .map(i -> i * 3).forEach(System.out::println);
    }
}
import java.util.Arrays;
import java.util.Collection;
import java.util.List;
import java.util.stream.Collectors;

public class FlatMapExam {
    public static void main(String[] args) {
        List<List<String>> nestedList = Arrays.asList(
                Arrays.asList("Apple", "Banana"),
                Arrays.asList("Cherry", "Date")
        );

        List<String> list = nestedList.stream()
                .flatMap(Collection::stream)    // 리스트 평탄화
                .collect(Collectors.toList());

        System.out.println(list);
    }
}
```



##### 정렬

`sorted()/(Comparator c)`: 기준에 따라 정렬 / 파라미터가 없는 경우 오름차순 정렬

```java
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Collectors;

public class SortedExam {
    public static void main(String[] args) {
        int[] arr = {5,3,27,9,5,9,0,4,34};
        Arrays.stream(arr)
                .sorted()
                .forEach(s -> System.out.print(s + " "));

        System.out.println();
        Arrays.stream(arr)
                .boxed()    // 기본형 타입을 참조형으로 변환 (Stream<Integer>)
                .sorted(Comparator.reverseOrder())  // 제네릭 타입 기본형 X
                .forEach(s -> System.out.print(s + " "));
    }
}
```



#### 요소 순회 (루핑)

`forEach`: 최종 연산

`peek`: 중간 연산 / 원본 스트림 리턴

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class PeekExam {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        List<Integer> doubleNumbers = numbers.stream()
                .peek(n -> System.out.println("before process: " + n))
                .map(n -> n * 2)
                .peek(n -> System.out.println("after process: " + n))
                .collect(Collectors.toList());
        System.out.println(doubleNumbers);
    }
}
```



#### 매칭/검색/집계

`allMatch(Predicate p)`, `anyMatch(Predicate p)`, `noneMatch(Predicate p)`: 최종 연산 (모두/하나라도/0개 매치하는지 확인)

`findAny()`, `findFirst()`

```java
import java.util.Arrays;

public class MatchExam {
    public static void main(String[] args) {
        int[] intArr = {2, 4, 6, 8};

        boolean result;

        result = Arrays.stream(intArr)
                        .allMatch(i -> i%3 == 0);
        System.out.println("모두 3의 배수 입니까?? " + result);

        result = Arrays.stream(intArr)
                        .anyMatch(i -> i%3 == 0);
        System.out.println("하나라도 3의 배수 입니까?? " + result);

        result = Arrays.stream(intArr)
                        .noneMatch(i -> i%3 == 0);
        System.out.println("모두 3의 배수가 아닙니까?? " + result);
    }
}
```



#### 리듀싱

스트림 내의 여러 데이터를 특정 기준에 따라 하나의 결과로 결합

`reduce(초기값, 람다식)`: ex. reduce(0, (a, b) -> a+b)인 경우 a가 초기값, b가 자료구조의 원소가 됨

```java
import java.util.Arrays;
import java.util.List;

public class ReduceExam {
    public static void main(String[] args) {
        List<Integer> list = Arrays.asList(1, 2, 3, 4, 5);

        int sum = list.stream()
                .reduce(0, (a, b) -> a + b);    // 초기값, 람다식(초기값, 리스트의 원소)
        System.out.println(sum);
    }
}
```



### Collectors

스트림의 요소를 다양한 형태의 결과로 수집하는 데 사용되는 메소드 제공

- 목록/세트/맵 수집

  `toList()`, `toSet()`, `toMap(Function f, Function f)`

- 데이터 요약

  `summarizingInt/Double/Long()`: 숫자 스트림에 사용 가능

- 문자열 결합

  `joining()`

- 그룹화/분할

  `groupingBy(Function f)/(Function f, Collector c)`: map 원소 수집 / 두번째 파라미터가 없으면 list 타입

  `partitioningBy(Predicate p)/(Predicate, Collector)`

  ```java
  import java.util.Arrays;
  import java.util.List;
  import java.util.Map;
  import java.util.stream.Collectors;
  
  // members는 name, score, age, sex로 구성되어 있는 클래스
  
  public class GroupingTest {
      public static void main(String[] args) {
          List<Member> members = Arrays.asList(
                  new Member("kim", 33, Member.MALE),
                  new Member("lee", 26, Member.FEMALE),
                  new Member("park", 11, Member.FEMALE),
                  new Member("choi", 47, Member.MALE)
          );
  
          // 성별을 기준으로 그룹화
          Map<Integer, List<Member>> mapBySex = members.stream()
                  .collect(Collectors.groupingBy(Member::getSex));
          System.out.println("Male: ");
          mapBySex.get(Member.MALE).stream()
                  .forEach(m -> System.out.print(m.getName() + " "));
          System.out.println("\\nFemale: ");
          mapBySex.get(Member.FEMALE).stream()
                  .forEach(m -> System.out.print(m.getName() + " "));
      }
  }
  ```



### 컬렉션 vs 스트림

| **구분**    | **컬렉션** | **스트림**                     |
| ----------- | ---------- | ------------------------------ |
| 처리 방식   | 다운로드   | 스트리밍                       |
| 저장 공간   | 필요       | 불필요                         |
| 반복 방식   | 외부 반복  | 내부 반복                      |
| 코드 구현   | 명령형     | 선언형                         |
| 원본 데이터 | 변경       | 변경하지 않고 소비             |
| 연산 병렬화 | 어려움     | 쉬움 (`parallelStream()` 이용) |