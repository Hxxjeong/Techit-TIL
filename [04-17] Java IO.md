## Java IO

### InputStream & OutputStream: 바이트 단위 입출력

- File, ByteArray, Buffered, Data

  ```java
  import java.io.FileInputStream;
  import java.io.FileOutputStream;
  
  public class ReadFile {
      public static void main(String[] args) throws Exception {
          System.out.println(System.getProperty("user.dir"));
          FileInputStream fis = new FileInputStream("a.txt");
          FileOutputStream fos = new FileOutputStream("b.txt");
  
          byte[] bytes = new byte[1024];  // buffer의 크기
          int n;  // 읽은 글자
  
  //        while ((n = fis.read(bytes)) != -1) { // 파일의 끝
  //            String str = new String(bytes);
  //            System.out.println(str);
  //            fos.write(bytes);
  //        }
  
          while ((n = fis.read()) != -1) {
              System.out.println((char) n);
          }
  
          fos.close();
          fis.close();
      }
  }
  ```



### Reader & Writer: 문자 단위 입출력

특징: 문자 단위 처리, 인코딩 호환성, 텍스트 중심

- File, String, Buffered, InputStream

  - InputStreamReader

    byte 스트림을 문자 스트림으로 변환하는 데 사용 (바이트 기반 → 문자 기반)

  ```java
  import java.io.FileReader;
  import java.io.FileWriter;
  import java.io.IOException;
  
  public class CharStreamExam {
      public static void main(String[] args) {
          try(FileReader fr = new FileReader("a.txt");
              FileWriter fw = new FileWriter("c.txt")) {
              int c;
              while ((c = fr.read()) != -1) {
                  fw.write(c);
              }
          }
          catch (IOException e) {
              e.printStackTrace();
          }
      }
  }
  ```



##### 특수 IO (System 클래스)

in: 표준 입력 스트림, InputStream 형태로 제공

out: 표준 출력 스트림, PrintStream의 인스턴스

err: 표준 에러 스트림, PrintStream의 인스턴스



##### FileWriter vs. PrintWriter

1. 쓰기 메서드 차이
   - `FileWriter`: 문자(char) 단위로 파일에 데이터를 씀 / `write()` 메서드 사용
   - `PrintWriter`: 문자열이나 다른 데이터 유형을 출력하기 위한 `print()` 및 `println()`과 같은 다양한 메서드를 제공 / 이를 사용하여 텍스트 데이터를 파일에 쓸 수 있음
2. 편의성
   - `PrintWriter`는 `FileWriter`보다 편리하고 유연 / `PrintWriter`의 메서드를 사용하면 자동으로 문자열을 문자로 변환하고 필요한 경우에는 줄 바꿈을 추가할 수 있음
   - `FileWriter`는 문자열을 쓸 때 반드시 문자열을 문자로 변환해야 하고, 줄 바꿈을 추가하려면 추가적인 작업이 필요
3. 예외 처리
   - `FileWriter`는 `IOException` 처리 / 파일 쓰기 도중에 발생하는 예외를 명시적으로 처리해야 함
   - `PrintWriter`는 `FileWriter`를 감싸기 때문에 예외 처리가 간단 / `PrintWriter`는 내부적으로 예외를 처리하고 `IOException` throw X



### 보조 스트림

기본 스트림에 추가적인 기능 제공 (메소드 중요)

Buffered, Data, Object, Print, Character to Byte

```java
import java.io.DataInputStream;
import java.io.FileInputStream;

public class DataInExam {
    public static void main(String[] args) throws Exception {
        DataInputStream dis = new DataInputStream(new FileInputStream("data.txt"));

        System.out.println(dis.readBoolean());
        System.out.println(dis.readByte());
        System.out.println(dis.readChar());
        System.out.println(dis.readDouble());

        dis.close();
    }
}
import java.io.DataOutputStream;
import java.io.FileOutputStream;

public class DataOutExam {
    public static void main(String[] args) throws Exception {
        // Java의 데이터 타입으로 파일에 출력
        DataOutputStream dos = new DataOutputStream(new FileOutputStream(("data.txt")));

        dos.writeBoolean(true);
        dos.writeByte(Byte.MAX_VALUE);
        dos.writeChar('a');
        dos.writeDouble(2.1);

        dos.close();
    }
}
```

- 객체 직렬화 & 역직렬화

  - **객체 직렬화(Serialization)**

    객체를 바이트 스트림으로 변환하는 과정

    객체의 상태와 데이터를 저장하고 네트워크를 통해 전송하거나 파일에 저장 가능

    객체 직렬화를 하려면 `Serializable` 인터페이스를 구현해야 함

    객체를 직렬화하기 위해서는 `ObjectOutputStream` 클래스 사용

  - **객체 역직렬화(Deserialization)**

    바이트 스트림으로부터 객체를 복원하는 과정

    저장된 객체의 상태와 데이터를 복원 가능

    객체 역직렬화를 하려면 직렬화된 객체를 저장한 바이트 스트림과 같은 클래스의 클래스 파일이 필요

    객체를 역직렬화하기 위해서는 `ObjectInputStream` 클래스 사용