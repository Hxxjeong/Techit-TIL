## [04/23] TCP, UDP



### TCP를 이용한 채팅

다수의 클라이언트가 서버에 동시에 연결되어 서로 메시지를 주고 받을 수 있어야 함

- 서버 구현
  1. ServerSocket 설정: `ServerSocket`을 이용하여 특정 포트에서 클라이언트의 연결을 기다림
  2. 클라이언트 연결 처리: 클라이언트의 연결 요청이 들어오면 `accept()` 메소드를 통해 수락하고, 각 클라이언트마다 별도의 스레드를 생성하여 독립적으로 처리
  3. 메시지 중계: 클라이언트로부터 받은 메시지를 다른 클라이언트에게 전달 (BoradCast)
- 클라이언트 구현
  1. Socket을 이용한 서버 연결: 서버의 IP 주소와 포트 번호를 사용하여 서버에 연결
  2. 메시지 송수신: 콘솔에서 입력받은 메시지를 서버에 보내고, 서버로부터 오는 메시지를 콘솔에 출력

스레드 간 동기화를 고려해야 함

- 서버

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

public class ChatServer {
    public static void main(String[] args) {
        // 1. ServerSocket 생성
        try (ServerSocket serverSocket = new ServerSocket(9999);) {
            System.out.println("Ready Server.");

            // 여러 클라이언트의 정보를 넣을 자료구조
            Map<String, PrintWriter> clients = new HashMap<>();

            // 2. accept()를 통해 클라이언트를 기다림   (여러 명의 클라이언트와 통신)
            while (true) {
                Socket clientSocket = serverSocket.accept();

                // Thread 이용
                new ChatThread(clientSocket, clients).start();
            }

        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class ChatThread extends Thread {
    // 생성자를 통해 클라이언트 소켓을 얻어옴
    private Socket socket;
    private String id;  // 클라이언트를 구분할 id
    private Map<String, PrintWriter> clients;

    private BufferedReader br;   // 생성자와 run()에서 사용하기 위함
    private PrintWriter pw;

    public ChatThread(Socket socket, Map<String, PrintWriter> clients) {
        this.socket = socket;
        this.clients = clients;

        // 클라이언트가 생성될 때 클라이언트로부터 id 얻기
        // 각 클라이언트와 통신할 수 있는 통로를 얻어옴
        try {
            pw = new PrintWriter(socket.getOutputStream(), true);
            br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            id = br.readLine(); // 클라이언트가 접속하면 id 설정

            // 모든 사용자에게 id가 입장했다는 정보 알림
            broadcast(id + "님이 입장하셨습니다.");
            System.out.println("New Member's id: " + id);

            // 동시에 입장하는 경우 고려
            synchronized (clients) {
                clients.put(this.id, pw);
            }
        }
        catch (Exception e) {
            System.out.println(e);
        }
    }

    // 연결된 클라이언트가 메시지를 전송하면 다른 클라이언트들에게 메시지 전송
    @Override
    public void run() {
        String msg;
        try {
            while ((msg = br.readLine()) != null) {
                if("/quit".equalsIgnoreCase(msg))
                    break;

                // @id 메시지 형식
                if (msg.indexOf("@") == 0)
                    toSomeone(msg);
                else
                    broadcast(id + " : " + msg);
            }
        }
        catch (IOException e) {
            System.out.println(e);
        }
        finally {
            // 동시에 퇴장하는 경우 고려
            synchronized (clients) {
                clients.remove(id);
            }
            broadcast(id + "님이 나갔습니다.");

            // 사용 후 BufferedReader와 Socket 닫기
            if(br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }

            if(socket != null) {
                try {
                    socket.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }

    // 모든 클라이언트에게 전송
    // 전체 사용자에게 알려주는 메소드 (Broadcast)
    public void broadcast(String msg) {
        // Iterator를 사용
        synchronized (clients) {
            Iterator<PrintWriter> iterator = clients.values().iterator();
            while(iterator.hasNext()) {
                PrintWriter pw = iterator.next();
                try {
                    pw.println(msg);
                }
                catch (Exception e) {
                    iterator.remove();  // Broadcast 할 수 없는 사용자 제거
                    e.printStackTrace();
                }
            }
        }
    }

    // 메시지를 특정 사용자에게 전송
    public void toSomeone(String msg) {
        int firstSpaceIndex = msg.indexOf(" "); // 첫번째 공백의 인덱스

        String targetId = msg.substring(1, firstSpaceIndex);
        String message = msg.substring(firstSpaceIndex+1);

        //to(수신자)에게 메시지 전송.
        PrintWriter pw = clients.get(targetId);
        if (pw != null) {
            pw.println(id + "님의 귓속말: " + message);
        } else {
            pw.println(targetId + " 님을 찾을 수 없습니다.");
        }
    }
}
```

- 클라이언트

```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;

public class ChatClient {
    public static void main(String[] args) {
        // 처음에 아이디가 입력되어야 함 (args[0]으로 구현)
        if(args.length != 1) {
            System.out.println("사용법: java ChatClient id");
            System.exit(1);
        }

        try (Socket socket = new Socket("localhost", 9999);
             PrintWriter pw = new PrintWriter(socket.getOutputStream(), true);
             BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
             BufferedReader keyboard = new BufferedReader(new InputStreamReader(System.in));
        ) {
            // 접속되면 서버에 id를 보냄
            pw.println(args[0]);

            // 네트워크에서 입력된 내용 담당 스레드
            new InputThread(socket, br).start();

            // 키보드로부터 입력받은 내용을 서버에 전송
            String msg;
            while((msg = keyboard.readLine()) != null) {
                pw.println(msg);
                if("/quit".equalsIgnoreCase(msg))
                    break;
            }
        }
        catch (Exception e) {
            System.out.println(e);
        }
    }
}

class InputThread extends Thread {
    private Socket socket;
    private BufferedReader br;

    public InputThread(Socket socket, BufferedReader br) {
        this.socket = socket;
        this.br = br;
    }

    @Override
    public void run() {
        try {
            String msg;
            // 서버에서 받아서 콘솔에 출력
            while((msg = br.readLine()) != null) {
                System.out.println(msg);
            }
        }
        catch (Exception e) {
            System.out.println(e);
        }
        finally {
            if(br != null) {
                try {
                    br.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }

            if(socket != null) {
                try {
                    socket.close();
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }
        }
    }
}
```



### UDP 통신

데이터그램으로 전송

#### Echo 서버 구현

1. DataframSocket 객체 생성: `DatagramSocket` 객체를 생성하여 특정 포트에서 클라이언트의 데이터그램 기다림
2. 데이터 수신 및 응답: `DatagramPacket`을 사용하여 클라이언트로부터 데이터를 받아 동일한 데이터를 클라이언트에게 다시 보냄
3. 소켓 종료: 더이상 데이터 송수신이 필요없을 때 `DatagramSocket`을 닫아 자원 해제



#### Echo 클라이언트 구현

1. DataframSocket 객체 생성: `DatagramSocket` 객체를 생성하여 데이터 전송 준비
2. 데이터 전송 및 응답 수신: `DatagramPacket`에 데이터를 담아 서버로 보내고, 동일한 데이터를 받음
3. 소켓 종료: 데이터 송수신이 끝나면 `DatagramSocket`을 닫음

```java
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

public class UDPEchoServer {
    public static void main(String[] args) {
        // 1. UDP 통신 특정 포트 번호에 DatagramSocket 생성
        try (DatagramSocket socket = new DatagramSocket(8888);
        ) {
            while(true) {
                byte[] receiveData = new byte[1024];
                byte[] sendData;
                DatagramPacket receivePacket = new DatagramPacket(receiveData, receiveData.length);
                socket.receive(receivePacket);

                String msg = new String(receivePacket.getData()).trim();

                // 클라이언트 정보
                InetAddress clientAddress = receivePacket.getAddress();
                int port = receivePacket.getPort();

                DatagramPacket sendPacket = new DatagramPacket(msg.getBytes(), msg.getBytes().length, clientAddress, port);
                socket.send(sendPacket);
            }
        }
        catch (Exception e) {
            System.out.println(e);
        }
    }
}
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

public class UDPEchoClient {
    public static void main(String[] args) {
        // 클라이언트는 따로 포트를 열 필요 없음
        try (DatagramSocket clientSocket = new DatagramSocket();
             BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
             ) {
            // 서버의 주소 설정
            InetAddress ipAddress = InetAddress.getByName("localhost");
            byte[] sendData;
            byte[] receiveData = new byte[1024];

            // 키보드로 입력 받기
            System.out.println("Input Send Message: ");
            String message = br.readLine();

            // 서버로 메시지 보내기
            DatagramPacket sendPacket = new DatagramPacket(message.getBytes(), message.getBytes().length, ipAddress, 8888);
            clientSocket.send(sendPacket);

            // 서버로부터 받기
            DatagramPacket receivePacket = new DatagramPacket(receiveData, receiveData.length);
            clientSocket.receive(receivePacket);
            String receiveMsg = new String(receivePacket.getData()).trim();

            System.out.println("Message: " + receiveMsg);
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```