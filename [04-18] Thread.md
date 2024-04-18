## [04/18] Thread



### 멀티 스레드

하나의 프로세스 내에서 여러 개의 스레드가 동작하는 것

각 스레드는 독립적인 실행 흐름을 가지며, 자신만의 스택을 가지지만 힙 메모리와 코드 영역은 공유

프로그램의 작업을 병렬적으로 처리

- 장점

  여러 작업을 동시에 처리 가능 (ex. 채팅 기능)

  프로세스보다 스레드를 생성하고 관리하는 비용이 낮음

  응답형 향상

- 작성 문법

  Thread 클래스 상속

  ```java
  class MyThread extends Thread { // Thread를 상속받은 클래스는 Thread
      @Override
      public void run() {
          System.out.println("MyThread");
          // Thread가 해야 하는 일 구현
          for(int i=0; i<10; i++) {
              System.out.println("hi");
          }
          System.out.println("MyThread End");
      }
  }
  
  public class ThreadExam {
      public static void main(String[] args) {
          System.out.println("main");
  
          MyThread mt = new MyThread();
          mt.start(); // 새 Thread(실행 흐름)이 생성됨 = 멀티스레드로 실행
  
          for(int i=0; i<10; i++) {
              System.out.println("main hi");
          }
  
          System.out.println("end main");
      }
  }
  ```

  Runnable 인터페이스 구현

  ```java
  class MyRunnable implements Runnable {
      @Override
      public void run() {
          System.out.println("MyRunnable");
          // Thread가 해야 하는 일 구현
          for(int i=0; i<10; i++) {
              System.out.println("MyRunnable hi");
          }
          System.out.println("MyRunnable End");
      }
  }
  
  public class RunnableExam {
      public static void main(String[] args) {
          System.out.println("main");
  
          // Thread 생성
          Thread thread = new Thread(new MyRunnable());
          thread.start();
  
          for(int i=0; i<10; i++) {
              System.out.println("main hi");
          }
  
          System.out.println("end main");
      }
  }
  ```



### Thread

프로세스 내에서 실제로 작업을 수행하는 최소 단위

프로세스의 리소스를 공유

스레드가 너무 많을 경우 오버헤드가 증가할 수 있음

- 생명 주기 제어 (Object의 메소드)

  `wait()`: 스레드를 대기 상태로 만듦

  `notify()`/`notifyAll()`: 대기 중인 스레드를 깨움 (`wait()`을 깨움)

- 상태 확인 및 제어

  `interrupt()`: 스레드 중단

  `isInterrupted()`/`interrupted()`: 스레드의 인터럽트 여부 확인

- 기타 메소드

  `join()`: 한 스레드가 다른 스레드의 종료를 기다림

  `setDaemon(true)`: 데몬 스레드로 설정

  - 데몬 스레드

    백그라운드에서 실행되는 스레드

    다른 모든 일반 스레드가 종료되면 자동으로 종료되는 스레드

    주로 자동 저장, 가비지 컬렉션, 자동 업데이트 등과 같은 백그라운드 작업을 처리하는 데 사용

    일반 스레드와 구별하기 위해 JVM이 관리

    JVM은 모든 일반 스레드가 종료되면 데몬스레드를 자동으로 종료시킴



### synchronized

메소드나 블록을 동기화하여 한 시점에 하나의 스레드만이 해당 코드 영역에 접근

```java
public synchronized void synchronizedMethod() {
    // 동기화된 메소드 내용
}

public void synchronizedBlock() {
    synchronized(this) {    // this: 공유 객체
        // 동기화된 블록 내용
    }
}
```