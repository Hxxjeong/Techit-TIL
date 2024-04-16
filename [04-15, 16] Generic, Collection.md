## [04/15, 16] Generic, Collection



### 제네릭 (Generic)

type을 파라미터로 사용할 수 있게 하는 기능

- 파라미터 종류

  - 멀티 타입 파라미터

    `class Pair<K, V>`: 두 타입 파라미터를 갖는 Pair 클래스

    이를 사용할 때는 Pair<Integer, String> myPair = new Pair<>(); 처럼 각 타입 파라미터에 대해 구체적인 타입 지정

  - 바운디드 타입 파라미터

    타입 파라미터에 특정 상위 타입 제한 두는 경우

    `class NumericBox<T extends Number>`: Number 클래스의 서브 클래스만을 타입 파라미터로 가질 수 있는 NumericBox 클래스

    NumericBox는 Number 또는 그 서브클래스의 인스턴스만 저장

- 제네릭 메소드

  ```java
  public <T> void genericMethod(T data) {
      // 메소드 본문
  }
  ```

- Auto Boxing & Auto Unboxing

  **Auto boxing**: 기본 타입의 값을 해당하는 래퍼 클래스의 객체로 자동으로 변환하는 것 (ex. int to Integer)

  **Auto Unboxing**: 래퍼 클래스의 객체를 해당하는 기본 타입의 값으로 자동으로 변환하는 것 (ex. Integer to int)



### 컬렉션 프레임워크

데이터 집합을 효율적으로 처리할 수 있도록 설계된 표준화된 방법 제공

#### 실습문제

```java
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class CollectionExam02 {

    static class BoundException extends RuntimeException {
        public BoundException(String msg) {
            super(msg);
        }
    }

    // 입력 처리
    public static void inputNum(Scanner sc, List<Integer> list) {
        while (true) {
            int num = sc.nextInt();
            if(num == 0)
                break;
            else {
                // 점수 유효성 검사
                if(num<0 || num>100)
                    throw new BoundException("숫자는 0 이상 100 이하여야 합니다.");
                else
                    list.add(num);
            }
        }
    }

    // 점수 출력
    public static void printNum(List<Integer> list) {
        for(Integer i: list) {
            System.out.println(i);
        }
    }

    // 합과 평균 계산
    public static void sumAndAvg(List<Integer> list) {
        int sum = 0;
        for(Integer i: list)
            sum += i;
        double avg = (double)sum/list.size();

        System.out.println("합계: " + sum + ", 평균: " + avg);
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        System.out.println("숫자를 입력하세요. 0을 입력하면 종료합니다.");

        List<Integer> list = new ArrayList<>();
        inputNum(sc, list);
        printNum(list);
        sumAndAvg(list);
    }
}
import java.util.ArrayList;
import java.util.List;
import java.util.Scanner;

public class WordCollectionExam {

    public static void inputWords(Scanner sc, List<String> list) {
        while (true) {
            String str = sc.nextLine();

            if(str.equals("quit"))
                break;
            else
                list.add(str);
        }
    }

    public static void printWords(List<String> list) {
        for(String s: list)
            System.out.println(s);
    }

    public static void allWords(List<String> list) {
        int allSum = 0;
        for(String s: list)
            allSum += s.length();
        System.out.println("모든 단어의 개수: " + list.size() + ", 글자수의 합: " + allSum);
    }

    public static void findLongWord(List<String> list) {
        int max = list.get(0).length();
        String longStr = list.get(0);
        for(int i=0; i<list.size(); i++) {
            if(list.get(i).length() > max) {
                longStr = list.get(i);
                max = list.get(i).length();
            }
        }
        System.out.println("가장 긴 단어: " + longStr + ", 그 길이: " + max);
    }

    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        List<String> list = new ArrayList<>();

        System.out.println("단어를 입력하세요. quit을 입력하면 종료합니다.");
        inputWords(sc, list);

        // 입력한 단어 모두 출력
        printWords(list);

        // 모든 단어의 개수와 글자 수의 합
        allWords(list);

        // 단어 중 가장 긴 단어
        findLongWord(list);
    }
}
```



#### Iterator

컬렉션 프레임워크에서 컬렉션에 저장된 요소를 읽거나 제거하는 방법을 제공하는 인터페이스

Set의 경우 인덱스를 통해 접근할 수 없지만 Iterator를 통해 데이터를 처리할 수 있음 (for-each문으로도 가능)

- 주요 기능

  순차적 접근, 안전한 요소 제거, 단방향 이동

```java
import java.util.*;

public class IteratorExam {
    public static void main(String[] args) {
        // set은 순서가 없기 때문에 인덱스로 접근 X
        Set<String> set = new HashSet<>();
        set.add("a");
        set.add("b");
        set.add("c");
        Iterator<String> setIter = set.iterator();
        while (setIter.hasNext()) {
            System.out.println(setIter.next());
        }
    }
}
```



#### Set

- 주요 클래스

  - HashSet

    가장 널리 사용되는 Set 구현체

    내부적으로 HashMap을 사용하여 요소를 저장

    요소의 순서를 보장하지 않으며 빠른 검색 속도를 제공

  - LinkedHashSet

    HashSet과 유사하지만 요소들을 삽입된 순서대로 유지

    LinkedList의 형태로 데이터를 저장하여 순서 보존

  - TreeSet

    레드-블랙 트리 데이터 구조를 기반으로 하는 Set 구현체

    요소들을 정렬된 순서대로 저장하고 검색, 삭제, 삽입 작업을 효율적으로 수행



#### Map

Key와 Value 쌍으로 데이터를 저장하는 자료구조

Map의 Key는 중복될 수 없음

```java
import java.util.HashMap;
import java.util.Map;

public class PhoneBookExam {
    public static void main(String[] args) {
        Map<String, String> phoneBook = new HashMap<>();

        phoneBook.put("김철수", "010-1234-5678");
        phoneBook.put("박영희", "010-9876-5432");
        phoneBook.put("홍길동", "010-1472-5836");

        String gildongNum = phoneBook.get("홍길동");
        System.out.println("홍길동의 전화번호: " + gildongNum);

        System.out.println("전쳬 전화번호 목록");
        for(Map.Entry<String, String> entry: phoneBook.entrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }

        phoneBook.remove("홍길동");    // 삭제

        // 특정 값 존재 여부
        if(phoneBook.containsKey("김철수"))
            System.out.println("Contain!");
        if(phoneBook.containsValue("010-3365-9752"))
            System.out.println("Contain!");
        else
            System.out.println("NO :(");
    }
}
```



### Comparable vs Comparator

#### **Comparable**

객체 자체가 정렬 기준을 가지고 있는 경우에 사용

`compareTo()` 메서드를 오버라이드하여 정렬 기준을 지정

- `compareTo()` 메소드 규칙

  현재 객체(this)가 매개변수로 전달된 객체보다 작으면 음수, 같으면 0, 크면 양수 반환

  ex. `String` 클래스 - `Comparable` 인터페이스를 구현하여 문자열 정렬



#### **Comparator**

객체의 정렬 기준을 외부에서 제공해야 할 때 사용

`compare()` 메서드를 오버라이드하여 두 객체를 비교

- `compare()`: 두 객체를 매개변수로 받아들이고, 비교 결과에 따라 음수, 0, 양수 반환

`Collections.sort()`나 `Arrays.sort()`와 함께 사용되어 객체의 정렬 순서 제어