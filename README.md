title: "러시안 룰렛 2인 네트워크 게임 - 서버/클라이언트 구조 & 실행 흐름 README"
description: >
  Eclipse + JDK 21 환경에서 Swing과 Socket을 이용해 구현한
  2인 네트워크 러시안 룰렛 게임의 현재 구조, 실행 흐름, 주요 클래스 역할을
  한 번에 볼 수 있도록 정리한 README YAML입니다.
  (Phase-1: 채팅 + 로비/게임방 입장까지 구현 상태 기준)

sections:
  - id: "1-project-structure"
    title: "1. 프로젝트 전체 구조 한 번에 정리"
    content: |-
      ### 📁 전체 구조 개요

      폴더 구조를 “역할” 기준으로 정리하면 이렇게야:

      ---

      #### 🖥 server/

      - **ServerGuiMain**
        - 서버 프로그램의 main 함수.
        - “서버 GUI 창(ServerFrame)을 띄워라” 한 줄만 하는 시작점.

      - **ServerFrame**
        - 포트 번호 입력 + “방 만들기(서버 시작)” 버튼이 있는 GUI.
        - 여기서 실제로 서버를 켜고 끄는 역할.

      - **ServerCore**
        - 진짜 서버 핵심.
        - ServerSocket을 열고, 클라이언트 2명을 받아서 Room을 만들고,
          각각에 ClientHandler를 붙임.

      - **Room**
        - 한 방(2인 게임방)의 상태를 관리하는 클래스.
        - 지금은 주로  
          “방이 만들어졌다는 것, READY, ENTER_ROOM, CHAT 브로드캐스트”까지 담당.

      - **ClientHandler**
        - 클라이언트 1명당 1개씩 돌아가는 스레드.
        - 클라이언트가 보낸 문자열을 읽어서 Room에게 넘겨주고,
          Room이 다시 둘에게 방송.

      - **Protocol**
        - 문자열 프로토콜(“ROOM_STATUS”, “CHAT” 같은 키워드들)을 상수로 모아 둔 클래스.

      - **(추후) GunChamber, Inventory**
        - 탄창/카드 시스템 로직이 들어갈 자리.

      ---

      #### 💻 client/

      - **ClientMain**
        - 클라이언트 프로그램 시작점.
        - StartFrame을 띄움.

      - **StartFrame**
        - Host/IP, Port, Name(플레이어 이름)을 입력하고
          “Connect” 버튼 누르는 창.

      - **RoomFrame**
        - 서버에 연결된 후,
          “지금 몇 명 접속했는지 / P1, P2 이름 / READY 상태”를 보여주는 대기방 화면.

      - **NetworkClient**
        - 서버와의 소켓 연결 + 송수신을 담당하는 통신 전담 클래스.

      - **GameRoomFrame**
        - 실제 게임방 화면.
        - 지금은 **배경 + 플레이어 이미지 + 채팅 UI**까지 구현되어 있음.

      - **ImageLoader**
        - 리소스 폴더에서 PNG 이미지 읽어오는 유틸.

      - **(추후) HudPanel, TablePanel 등**
        - HP, 탄창 표시, 총 이미지/조준/발사 UI를 분리해서 넣을 예정.

      ---

      이제 이걸 토대로, 실제 흐름이 어떻게 이어지는지부터 설명할게.

  - id: "2-execution-flow"
    title: "2. 실행 흐름 한 번에 보는 시나리오"
    content: |-
      ### 🖥 서버 쪽 흐름

      1. **ServerGuiMain.main() 실행**
         - `new ServerFrame().setVisible(true)` → 서버 GUI 창 뜸

      2. **서버 GUI에서 포트 입력 → “방 만들기(서버 시작)” 버튼 클릭**
         - ServerFrame이 `ServerCore.start(port)` 호출
         - ServerSocket 열고, `acceptLoop()` 스레드가 돌아가면서
           **클라이언트 두 명을 기다림**.

      3. **1번 클라이언트 접속**
         - HELLO 핸드셰이크
         - 이름 받음
         - `ROOM_STATUS WAITING 1/2` 전송

      4. **2번 클라이언트 접속**
         - HELLO 핸드셰이크
         - 이름 받음
         - `ROOM_STATUS WAITING 2/2` 전송

      5. **Room 객체 생성 (P1, P2와 이름)**
         - Room 생성 후, 두 클라에게 순서대로 방송:
           - `ROOM_CREATED`
           - `ROOM_STATUS READY 2/2`
           - `ENTER_ROOM`  
             → 두 클라이언트에게 모두 전송

      6. **이후 채팅 처리**
         - 각 클라이언트에서 오는 `CHAT ...` 메시지를
           - ClientHandler가 읽고,
           - `Room.broadcastChat()`으로 다시 둘에게 뿌림.

      ---

      ### 💻 클라이언트 쪽 흐름

      1. **ClientMain.main() 실행 → StartFrame 띄움**

      2. **Host/IP, Port, Name 입력 후 “Start / Connect” 버튼 클릭**
         - RoomFrame 생성되면서 내부에서  
           `NetworkClient.connect(host, port, name)` 호출

      3. **NetworkClient가 서버와 연결하고:**

         - 서버의 `HELLO` 한 줄을 먼저 읽고 확인
         - 내 이름을 한 줄로 서버에 보내서 “나는 누구다” 등록
         - 서버에서 오는 내용을 계속 읽는 reader 스레드 시작

      4. **서버가 보내는 메시지에 따라 UI 동작**

         - `ROOM_STATUS WAITING 1/2 / WAITING 2/2`  
           → RoomFrame의 상태 라벨 갱신

         - `ROOM_CREATED P1=... P2=...`  
           → P1/P2 이름 라벨 설정

         - `ENTER_ROOM P1=... P2=...`  
           → 이제 GameRoomFrame을 띄우고,
             NetworkClient의 수신 리스너를 GameRoomFrame 쪽으로 교체.

      5. **게임방 입장 이후**

         - 이제부터 `GameRoomFrame.getLineConsumer()`가
           서버에서 오는 메시지를 받음.
         - 지금은 `CHAT`만 처리해서 채팅창에 보여줌.

      ---

      이제 이걸 코드 단위로 쪼개서 자세히 볼게.

  - id: "3-server-code-details"
    title: "3. 서버 코드 상세 설명"
    content: |-
      ## 3-1. ServerGuiMain – 서버 진입점

      ```java
      public class ServerGuiMain {
          public static void main(String[] args) {
              javax.swing.SwingUtilities.invokeLater(() -> new ServerFrame().setVisible(true));
          }
      }
      ```

      - main 함수에서 하는 일은 딱 1개:
        - Swing UI는 **이벤트 디스패치 스레드(EDT)**에서 실행해야 해서
          `SwingUtilities.invokeLater(...)` 안에서 `ServerFrame`을 띄움.

      - “람다식” 문법(`() -> ...`)은 사실 아래랑 완전히 같은 뜻이야 (발표용 설명):

      ```java
      SwingUtilities.invokeLater(new Runnable() {
          @Override
          public void run() {
              new ServerFrame().setVisible(true);
          }
      });
      ```

      → **요약:**  
      “서버 프로그램을 실행하면, 서버 설정/로그 창인 ServerFrame이 열린다.”

      ---

      ## 3-2. ServerFrame – 포트 입력 + 서버 시작/정지 + 로그

      ### 주요 필드

      ```java
      private final JTextField portField = new JTextField("7777", 8);
      private final JButton startBtn = new JButton("방 만들기(서버 시작)");
      private final JButton stopBtn  = new JButton("서버 중지");
      private final JTextArea logArea = new JTextArea();

      private final ServerCore core;
      ```

      - **portField** : 포트 번호(기본 7777)
      - **startBtn / stopBtn** : 서버 시작 / 중지 버튼
      - **logArea** : 서버 상태 메시지를 찍는 콘솔 같은 텍스트 영역
      - **core** : 실제 서버 동작을 담당하는 `ServerCore` 인스턴스

      ### 생성자에서 하는 UI 구성

      - `setLayout(new BorderLayout(8,8));`
      - **위쪽(NORTH)** : 포트 입력 + 버튼들이 있는 `JPanel top`
      - **가운데(CENTER)** : 스크롤 가능한 `logArea`

      ### 버튼 이벤트

      ```java
      startBtn.addActionListener(this::onStart);
      stopBtn.addActionListener(e -> onStop());
      ```

      - `onStart`, `onStop` 메서드가 아래에 정의됨.

      ### onStart – 서버 켜기

      ```java
      private void onStart(ActionEvent e) {
          final int port;
          try {
              port = Integer.parseInt(portField.getText().trim());
              if (port < 1024 || port > 65535) throw new NumberFormatException();
          } catch (NumberFormatException ex) {
              JOptionPane.showMessageDialog(this, "포트는 1024~65535 사이의 숫자여야 합니다.");
              return;
          }
          try {
              core.start(port);
              appendLog("[UI] 방 만들기 완료. 접속을 기다립니다... (2명 모이면 READY → ENTER_ROOM)");
              startBtn.setEnabled(false);
          } catch (Exception ex) {
              JOptionPane.showMessageDialog(this, "서버 시작 실패: " + ex.getMessage());
          }
      }
      ```

      - 포트 문자열을 읽어서 정수로 변환.
      - 1024~65535 범위를 벗어나면 에러 팝업.
      - 정상이면 `core.start(port)` 호출 → 실제 서버 가동.
      - 이후:
        - 로그에 “방 만들기 완료…” 메시지 출력
        - 동일 포트로 중복 실행 못 하도록 `startBtn` 비활성화.

      ### onStop – 서버 끄기

      ```java
      private void onStop() {
          core.stop();
          startBtn.setEnabled(true);
      }
      ```

      - `ServerCore.stop()` 호출해서 서버 소켓을 닫고,
      - 다시 시작할 수 있게 버튼 활성화.

      ### 로그 출력 함수

      ```java
      private void appendLog(String s) {
          logArea.append(s + "\n");
          logArea.setCaretPosition(logArea.getDocument().getLength());
      }
      ```

      - 새 로그를 추가하고,
      - 커서를 제일 아래로 내려서 **항상 최신 로그가 보이게** 함.

      → **요약(발표용)**  
      “ServerFrame은 포트를 입력해서 서버를 켜고 끌 수 있는 창이고,  
       실제 서버 동작은 ServerCore에게 맡긴다.  
       서버가 하는 일, 오류 메시지는 logArea에 계속 기록된다.”

      ---

      ## 3-3. ServerCore – 서버의 핵심 로직

      ### 주요 필드

      ```java
      private final Consumer<String> log;
      private ServerSocket server;
      private volatile boolean running = false;
      private Thread acceptThread;
      ```

      - `log` :
        - 문자열을 받아서 화면에 찍어주는 콜백.
        - 실제 구현은 `ServerFrame.appendLog`가 들어옴.
      - `server` :
        - 클라이언트 접속을 받는 리스닝 소켓 (`ServerSocket`).
      - `running` :
        - 서버가 돌고 있는지 여부.
      - `acceptThread` :
        - 클라이언트 접속을 기다리는 스레드.

      ### start(port) – 서버 시작

      ```java
      public synchronized void start(int port) throws IOException {
          if (running) { log.accept("[Server] already running"); return; }
          server = new ServerSocket(port);
          running = true;
          log.accept("[Server] Listening on " + port);

          acceptThread = new Thread(this::acceptLoop, "AcceptLoop");
          acceptThread.start();
      }
      ```

      - 이미 `running`이면:
        - “이미 켜져 있다”고 로그만 찍고 리턴.
      - 아니라면:
        - 지정된 포트로 `ServerSocket` 생성
        - `running = true`
        - `acceptLoop`를 별도 스레드에서 실행 → 이 안에서 `accept()`를 반복 호출.

      ### stop() – 서버 중지

      ```java
      public synchronized void stop() {
          running = false;
          try {
              if (server != null && !server.isClosed()) server.close();
          } catch (IOException ignored) {}
          log.accept("[Server] Stopped.");
      }
      ```

      - `running` 플래그를 내려서 `acceptLoop`가 더 이상 돌아가지 않게 하고,
      - `ServerSocket`을 닫아서 `accept()`에서 빠져나오게 함.

      ### acceptLoop() – 2명의 클라이언트를 받아 방 만들기

      ```java
      private void acceptLoop() {
          try {
              while (running) {
                  // 첫 번째 클라이언트
                  Socket s1 = server.accept();
                  String n1 = handshakeAndReadName(s1);
                  log.accept("[Server] P1 connected: " + n1 + " from " + s1.getRemoteSocketAddress());
                  sendLine(s1, Protocol.ROOM_STATUS + " WAITING 1/2");

                  // 두 번째 클라이언트
                  Socket s2 = server.accept();
                  String n2 = handshakeAndReadName(s2);
                  log.accept("[Server] P2 connected: " + n2 + " from " + s2.getRemoteSocketAddress());
                  sendLine(s2, Protocol.ROOM_STATUS + " WAITING 2/2");

                  // 핸들러 만들고 → 룸 만들고 → 핸들러에 룸 주입
                  ClientHandler p1 = new ClientHandler(s1, null, n1);
                  ClientHandler p2 = new ClientHandler(s2, null, n2);
                  Room room = new Room(p1, p2, n1, n2);
                  p1.setRoom(room);
                  p2.setRoom(room);

                  // 핸들러 스레드 시작
                  new Thread(p1, "P1-Handler").start();
                  new Thread(p2, "P2-Handler").start();

                  // 룸 생성 & READY & ENTER_ROOM 알림
                  room.announceCreatedAndReady();
                  log.accept("[Server] Room READY: " + n1 + " vs " + n2);
              }
          } catch (IOException e) {
              if (running) log.accept("[Server] accept error: " + e.getMessage());
          } finally {
              stop();
          }
      }
      ```

      #### 순서 정리

      1. `while (running)` :
         - 서버가 켜져 있는 동안 계속 반복.

      2. `server.accept()` :
         - 첫 번째 접속한 클라이언트의 `Socket s1` 확보.
         - `handshakeAndReadName(s1)` :
           - HELLO 핸드셰이크 + 클라이언트 이름 한 줄 읽기.
         - P1 연결 로그 찍고,  
           `ROOM_STATUS WAITING 1/2` 전송.

      3. 다시 `server.accept()` :
         - 두 번째 클라이언트 `Socket s2`.
         - 같은 방식으로 P2 이름 받고,  
           `ROOM_STATUS WAITING 2/2` 전송.

      4. 두 명이 모였으니:

         ```java
         ClientHandler p1 = new ClientHandler(s1, null, n1);
         ClientHandler p2 = new ClientHandler(s2, null, n2);
         Room room = new Room(p1, p2, n1, n2);
         p1.setRoom(room);
         p2.setRoom(room);
         ```

         - 각 ClientHandler를 스레드로 돌림.

      5. `room.announceCreatedAndReady();` 호출
         - `ROOM_CREATED ...`
         - `ROOM_STATUS READY 2/2`
         - `ENTER_ROOM ...`  
           을 두 명에게 동시에 보냄.

      → 이렇게 해서  
      **“두 명이 접속되면 바로 한 방이 만들어지고,  
      둘에게 동시에 ENTER_ROOM 신호를 보내서 게임 화면으로 들어가게 하는 구조”**가 됨.

      ### handshakeAndReadName()

      ```java
      private static String handshakeAndReadName(Socket s) throws IOException {
          BufferedReader in = new BufferedReader(new InputStreamReader(s.getInputStream(), "UTF-8"));
          BufferedWriter out = new BufferedWriter(new OutputStreamWriter(s.getOutputStream(), "UTF-8"));
          out.write(Protocol.HELLO + "\n"); out.flush();
          String name = in.readLine();
          return (name == null || name.isBlank()) ? "Player" : name.trim();
      }
      ```

      - 서버가 먼저 `"HELLO\n"` 을 보내고,
      - 클라이언트가 이름을 한 줄 보내면 그걸 읽어서 반환.
      - 이름이 비어있으면 기본 이름 `"Player"`로 대체.

      ### sendLine()

      ```java
      private static void sendLine(Socket s, String line) throws IOException {
          BufferedWriter out = new BufferedWriter(new OutputStreamWriter(s.getOutputStream(), "UTF-8"));
          out.write(line);
          out.write("\n");
          out.flush();
      }
      ```

      - 간단한 유틸: 소켓으로 **문자열 한 줄 전송**.

      ---

      ## 3-4. Protocol – 통신 메시지 키워드 정의

      ```java
      public final class Protocol {
          private Protocol() {}

          public static final String HELLO = "HELLO";
          public static final String ROOM_STATUS  = "ROOM_STATUS";
          public static final String ROOM_CREATED = "ROOM_CREATED";
          public static final String ENTER_ROOM   = "ENTER_ROOM";
          public static final String CHAT         = "CHAT";

          public static String kv(String k, Object v) { return k + "=" + v; }
      }
      ```

      - 문자열을 상수로 모아 둔 클래스.
      - **오타 방지 + 의미 통일** 목적.
      - `kv("P1", name1)` 을 쓰면 `"P1=name1"` 형식의 문자열을 만들어줌.

      **발표용 한 줄 요약:**

      - “서버와 클라이언트가 같은 말을 쓰기 위해,  
         프로토콜 문자열을 한 곳에 상수로 정의해 놓은 클래스입니다.”

      ---

      ## 3-5. Room – 한 게임방(2명)을 대표하는 클래스

      ```java
      public class Room {
          private final ClientHandler p1;
          private final ClientHandler p2;
          private final String name1;
          private final String name2;
      ```

      - `p1`, `p2` :
        - 각각의 클라이언트와 통신을 담당하는 ClientHandler.
      - `name1`, `name2` :
        - P1, P2의 이름.

      ### broadcast()

      ```java
      public void broadcast(String line) throws IOException {
          p1.send(line);
          p2.send(line);
      }
      ```

      - 두 클라이언트에게 **같은 문자열**을 한 번에 보내고 싶을 때 사용.

      ### announceCreatedAndReady()

      ```java
      public void announceCreatedAndReady() throws IOException {
          broadcast(Protocol.ROOM_CREATED + " "
              + Protocol.kv("P1", name1) + " "
              + Protocol.kv("P2", name2));
          broadcast(Protocol.ROOM_STATUS + " READY 2/2");
          broadcast(Protocol.ENTER_ROOM + " "
              + Protocol.kv("P1", name1) + " "
              + Protocol.kv("P2", name2));
      }
      ```

      - 2명이 입장 완료된 시점에 순서대로 방송:
        1. `ROOM_CREATED P1=... P2=...`
        2. `ROOM_STATUS READY 2/2`
        3. `ENTER_ROOM P1=... P2=...`

      - 클라이언트는 이 메시지를 보고:
        - 대기 화면 라벨을 채우고
        - 게임방 화면으로 진입.

      ### broadcastChat()

      ```java
      public void broadcastChat(String sender, String message) throws IOException {
          broadcast(Protocol.CHAT + " " + sender + ": " + message);
      }
      ```

      - “채팅 메세지를 두 클라이언트에게 브로드캐스트할 때 쓰는 함수”
      - 포맷 예: `CHAT 홍길동: 안녕`

      ---

      ## 3-6. ClientHandler – 클라이언트 한 명의 입력을 처리하는 스레드

      ```java
      public class ClientHandler implements Runnable {
          private final Socket socket;
          private Room room; // setRoom으로 설정
          private final String name;
          private final BufferedReader in;
          private final BufferedWriter out;
      ```

      - `socket` :
        - 이 플레이어와 연결된 소켓.
      - `room` :
        - 어떤 방에 속해 있는지 (2명 방).
      - `name` :
        - 플레이어 이름.
      - `in`, `out` :
        - 문자열을 읽고 쓰기 위한 스트림.

      ### 생성자

      ```java
      public ClientHandler(Socket socket, Room room, String name) throws IOException {
          this.socket = socket;
          this.room = room;
          this.name = name;
          this.in  = new BufferedReader(new InputStreamReader(socket.getInputStream(), "UTF-8"));
          this.out = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream(), "UTF-8"));
      }
      ```

      - 소켓에서 입력/출력 스트림을 만들어서, **텍스트 기반**으로 통신.

      ### send()

      ```java
      public void send(String line) throws IOException {
          out.write(line);
          out.write("\n");
          out.flush();
      }
      ```

      - 해당 플레이어에게 **문자열 한 줄 보내기**.

      ### run() – 클라이언트 입력 루프

      ```java
      @Override
      public void run() {
          try {
              String line;
              while ((line = in.readLine()) != null) {
                  System.out.println("[Server] from " + name + ": " + line);
                  if (line.startsWith("CHAT ")) {
                      String msg = line.substring(5).trim();
                      if (room != null && !msg.isEmpty()) {
                          System.out.println("[Server] BROADCAST CHAT by " + name + " -> " + msg);
                          room.broadcastChat(name, msg);
                      }
                  }
              }
          } catch (IOException ignored) {
          } finally {
              try { socket.close(); } catch (IOException ignored) {}
          }
      }
      ```

      - `while (in.readLine() != null)` :
        - 클라이언트가 연결을 유지하는 동안 **계속해서 문자열을 기다림**.
      - 현재는 명령이 `"CHAT "` 으로 시작할 때만 처리:
        - `CHAT` 뒤의 텍스트(`msg`)를 잘라서 가져옴.
        - `msg`가 비어있지 않고, `room`이 설정되어 있다면:
          - `room.broadcastChat(name, msg)` 호출  
            → 두 명에게 `CHAT <name>: <msg>` 전송.
      - 예외나 연결 끊김이 나면 finally에서 소켓 정리.

      → 지금 단계에서는

      - 서버 스레드는 **“채팅”만 처리**한다.
      - 게임 로직(턴, HP, 탄창)은 앞으로 `Room` / `GunChamber` 쪽으로 확장할 예정.

  - id: "4-client-code-details"
    title: "4. 클라이언트 코드 상세 설명"
    content: |-
      ## 4-1. ClientMain – 클라이언트 진입점

      ```java
      public class ClientMain {
          public static void main(String[] args) {
              javax.swing.SwingUtilities.invokeLater(() -> new StartFrame().setVisible(true));
          }
      }
      ```

      - 서버 쪽과 동일한 패턴.
      - 클라이언트 프로그램을 실행하면,
        제일 먼저 **접속 정보를 입력하는 StartFrame**이 뜸.

      ---

      ## 4-2. StartFrame – 서버 접속 정보 입력 창

      ### 주요 필드

      ```java
      private final JTextField hostField = new JTextField("127.0.0.1", 14);
      private final JTextField portField = new JTextField("7777", 6);
      private final JTextField nameField = new JTextField("Player", 10);
      private final JButton connectBtn = new JButton("Start / Connect");
      ```

      - Host/IP, Port, Name을 입력하는 UI 요소들.
      - 기본값:
        - host: `127.0.0.1` (같은 PC)
        - port: `7777`
        - name: `"Player"`

      ### 생성자에서 UI 구성

      - `GridLayout(4, 2)`를 사용해서 **라벨-입력칸 쌍**으로 배치.
      - 마지막 줄에 `connectBtn` 배치.
      - 버튼 이벤트:

      ```java
      connectBtn.addActionListener(e -> connect());
      ```

      ### connect() – 실제 연결 시도

      - 사용자 입력값 읽기
      - 유효성 검사 (비었는지, 포트가 숫자인지)
      - 문제가 없으면:

        ```java
        RoomFrame rf = new RoomFrame(host, port, name);
        rf.setVisible(true);
        dispose();
        ```

        - `RoomFrame`을 만들어서 띄우고,
        - 지금 `StartFrame`은 닫음.

      - 연결 실패 시 예외 종류에 따라 다른 안내 메시지:
        - `ConnectException` : 서버가 안 켜졌거나 포트가 다름
        - `UnknownHostException` : Host 이름을 해석 못함
        - `SocketTimeoutException` : 연결 시간 초과
        - 기타 : 상세 예외 이름과 메시지 보여줌.

      → **요약:**  
      “StartFrame은 접속 정보를 입력받고, 서버에 접속해서 RoomFrame으로 넘기는 역할.”

      ---

      ## 4-3. NetworkClient – 클라이언트의 통신 전담

      ### 주요 필드

      ```java
      private Socket socket;
      private BufferedReader in;
      private BufferedWriter out;

      private Consumer<String> onLine;
      ```

      - 서버와의 소켓 + 스트림.
      - `onLine` :
        - 서버에서 한 줄 읽을 때마다 호출되는 콜백.
        - 처음엔 `RoomFrame`의 메서드, 나중에는 `GameRoomFrame` 쪽으로 교체됨.

      ### 생성자 & 리스너 교체

      ```java
      public NetworkClient(Consumer<String> onLine) { this.onLine = onLine; }

      public void setOnLine(Consumer<String> newListener) { this.onLine = newListener; }
      ```

      - RoomFrame에서 만들 때:
        - `new NetworkClient(this::onServerLine)`
      - GameRoomFrame으로 넘어갈 때:
        - `net.setOnLine(gf.getLineConsumer())`

      ### connect(host, port, name)

      - `Socket` 생성 후, `connect` 호출 (타임아웃 3초)
      - `setSoTimeout(0)` → `read` 대기를 **무제한**으로 설정
      - `in`, `out` 스트림 준비

      #### HELLO 핸드셰이크

      ```java
      String hello = in.readLine();
      if (hello == null || !hello.startsWith("HELLO")) {
          throw new IOException("Handshake failed: HELLO not received");
      }
      ```

      - 헬로가 정상적으로 오면, 내 이름을 서버로 전송:

      ```java
      out.write(name);
      out.write("\n");
      out.flush();
      ```

      #### reader 스레드 시작

      ```java
      Thread reader = new Thread(() -> {
          try {
              String line;
              while ((line = in.readLine()) != null) {
                  final String msg = line;
                  System.out.println("[Client] RECV: " + msg);
                  SwingUtilities.invokeLater(() -> {
                      if (onLine != null) onLine.accept(msg);
                  });
              }
          } catch (IOException ignored) {
          } finally {
              try { socket.close(); } catch (IOException ignored) {}
          }
      }, "Client-Reader");
      reader.setDaemon(true);
      reader.start();
      ```

      - `in.readLine()`으로 서버에서 오는 메시지를 계속 기다림.
      - 한 줄 읽을 때마다:
        - 로그 출력
        - `onLine.accept(msg)` 호출  
          (UI 스레드에서 실행되도록 `invokeLater` 사용).
      - 연결이 끊어지면 finally에서 소켓 정리.

      ### send(line)

      ```java
      public void send(String line) {
          try {
              System.out.println("[Client] SEND: " + line);
              out.write(line);
              out.write("\n");
              out.flush();
          } catch (IOException ignored) {}
      }
      ```

      - 서버로 **문자열 한 줄** 보내는 함수.
      - 예: 채팅 입력 시 `net.send("CHAT 안녕하세요")`

      ---

      ## 4-4. RoomFrame – 두 명이 모일 때까지 상태 표시하는 대기방

      ### 필드

      ```java
      private final JLabel statusLabel = new JLabel("STATUS: -");
      private final JLabel p1Label = new JLabel("P1: -");
      private final JLabel p2Label = new JLabel("P2: -");
      private final NetworkClient net;

      private String p1Name = null;
      private String p2Name = null;
      private final String myName;
      ```

      - `STATUS` :
        - “WAITING 1/2”, “READY 2/2” 같은 상태를 보여줌.
      - `P1`, `P2` :
        - 각 플레이어 이름을 보여줌.
      - `net` : `NetworkClient` 인스턴스.
      - `myName` : 내가 입력한 이름.

      ### 생성자에서 하는 일

      - UI 배치:
        - 상태/플레이어 이름 라벨 3개를 세로로 배치.
      - `NetworkClient` 초기화:

      ```java
      net = new NetworkClient((Consumer<String>) this::onServerLine);
      net.connect(host, port, name);
      ```

      - `onServerLine` 메서드가 서버 메시지를 처리.

      ### onServerLine – 서버에서 오는 메시지 처리

      ```java
      private void onServerLine(String line) {
          if (line.startsWith("ROOM_STATUS")) {
              statusLabel.setText("STATUS: " + line.substring("ROOM_STATUS ".length()).trim());
          } else if (line.startsWith("ROOM_CREATED")) {
              p1Name = parseKV(line, "P1");
              p2Name = parseKV(line, "P2");
              p1Label.setText("P1: " + (p1Name == null ? "-" : p1Name));
              p2Label.setText("P2: " + (p2Name == null ? "-" : p2Name));
          } else if (line.startsWith("ENTER_ROOM")) {
              if (p1Name == null) p1Name = parseKV(line, "P1");
              if (p2Name == null) p2Name = parseKV(line, "P2");
              try {
                  GameRoomFrame gf = new GameRoomFrame(p1Name, p2Name, myName, net);
                  net.setOnLine(gf.getLineConsumer()); // ★ 중요: 수신기 교체
                  gf.setVisible(true);
                  dispose();
              } catch (Exception ex) {
                  JOptionPane.showMessageDialog(this, "게임방 열기 실패: " + ex.getMessage());
              }
          }
      }
      ```

      - `ROOM_STATUS ...`
        - → `STATUS:` 라벨 갱신.
      - `ROOM_CREATED P1=... P2=...`
        - → 이름을 파싱해서 각각 라벨에 표시.
      - `ENTER_ROOM ...`
        - → `GameRoomFrame` 생성, 보이기.
        - → 가장 중요한 부분:

          ```java
          net.setOnLine(gf.getLineConsumer());
          ```

        - 이제부터 서버에서 오는 문자열은
          - 더 이상 `RoomFrame`이 아니라
          - **`GameRoomFrame`이 처리**하게 됨.
        - 마지막으로 부모 창(`RoomFrame`)은 `dispose()`로 닫음.

      ---

      ## 4-5. GameRoomFrame – 게임방 화면 + 채팅 기능

      ### 주요 필드

      ```java
      private final String p1Name;
      private final String p2Name;
      private final String myName;
      private final NetworkClient net;

      private final RoomCanvas canvas;

      // 내 역할, 채팅창(1개), 누적 줄 제한
      private final String myRole;
      private ChatDialog chatDialog;
      private static final int MAX_CHAT_LINES = 500; // 성능 위해 최근 500줄만 유지
      ```

      - `p1Name`, `p2Name` : 두 플레이어 이름.
      - `myName` : 내 이름.
      - `myRole` : 내가 P1인지 P2인지  
        (`myName.equals(p1Name) ? "P1" : "P2" ...`).
      - `canvas` : 실제 배경 + 캐릭터를 그리는 `JPanel`.
      - `chatDialog` :
        - 채팅창(최대 한 개). 필요할 때만 생성.
      - `MAX_CHAT_LINES` :
        - 채팅이 너무 많이 쌓일 때 성능 문제 방지용  
          (500줄만 유지).

      ### 생성자 – 화면 구성

      - 타이틀: `"Game Room"`
      - 레이아웃: `BorderLayout`
      - 중앙 패널을 `OverlayLayout`으로 구성:

        - **아래 레이어** : `canvas` (`RoomCanvas`)
        - **위 레이어** : 투명한 overlay 패널  
          (상단 우측에 조작키/Chat 버튼)

      ```java
      canvas = new RoomCanvas(p1Name, p2Name);
      JPanel center = new JPanel();
      center.setLayout(new OverlayLayout(center));
      ```

      ### 상단 우측 바

      ```java
      JPanel overlay = new JPanel(new BorderLayout());
      overlay.setOpaque(false);
      JPanel topBar = new JPanel(new FlowLayout(FlowLayout.RIGHT, 8, 8));
      topBar.setOpaque(false);

      JLabel youLabel = new JLabel("You: " + myRole + " (" + myName + ")");
      JButton helpBtn = new JButton("조작키");
      JButton chatBtn = new JButton("Chat");
      ```

      - `helpBtn` :
        - 조작 설명 팝업 (`openHelpDialog()` 호출).
      - `chatBtn` :
        - 채팅창 열기 (`ensureChatDialog()` 후 `setVisible(true)`).

      → **발표용 설명**  
      “게임방 화면의 중앙에는 실제 플레이어 그림이 있고,  
       오른쪽 위에는 내가 P1인지 P2인지, 그리고 조작키 안내 버튼과  
       채팅 버튼이 있어요.”

      ### 서버 수신 처리: getLineConsumer()

      ```java
      public Consumer<String> getLineConsumer() {
          return line -> {
              if (line.startsWith("CHAT ")) {
                  String payload = line.substring(5).trim();
                  String sender = payload;
                  String msg = "";
                  int idx = payload.indexOf(':');
                  if (idx >= 0) {
                      sender = payload.substring(0, idx).trim();
                      msg = payload.substring(idx + 1).trim();
                  }
                  String role = sender.equals(p1Name) ? "P1" : (sender.equals(p2Name) ? "P2" : "?");
                  boolean isMe = sender.equals(myName);
                  String display = (role.equals("?") ? "" : "[" + role + "] ")
                                 + (isMe ? "[ME] " : "")
                                 + sender + ": " + msg;

                  // 대화창이 없으면 만들고, 안 떠 있으면 한 번만 띄움
                  ensureChatDialog();
                  chatDialog.appendLine(display, true); // 자동 스크롤 + 줄 제한
              }
          };
      }
      ```

      - 서버에서 `CHAT 홍길동: 안녕` 이런 줄을 받으면:
        - `payload`에서 `sender`(홍길동), `msg`(안녕) 분리.
        - 그 `sender`가 P1인지 P2인지 판단 (`role`).
        - 내가 보낸 메시지라면 `[ME]` 태그도 붙임.
        - 최종 문장 예:
          - `[P1] [ME] 홍길동: 안녕`
      - 채팅창이 아직 없으면 `ensureChatDialog()`로 만들고,
        `appendLine()`으로 한 줄 추가.

      ### RoomCanvas – 배경 + 상/하 플레이어 이미지 그리는 패널

      ```java
      static class RoomCanvas extends JPanel {
          private final String p1Name, p2Name;
          private final BufferedImage bg, p1img, p2img;

          public RoomCanvas(String p1Name, String p2Name) {
              this.p1Name = p1Name;
              this.p2Name = p2Name;
              setPreferredSize(new Dimension(1000, 560));
              bg    = ImageLoader.load("/images/room_bg.png");
              p1img = ImageLoader.load("/images/player1.png");
              p2img = ImageLoader.load("/images/player2.png");
          }
      ```

      - 생성 시, 배경 이미지와 P1/P2 이미지를 리소스에서 읽어옴.

      ```java
          @Override protected void paintComponent(Graphics g) {
              super.paintComponent(g);
              int w = getWidth(), h = getHeight();

              if (bg != null) g.drawImage(bg, 0, 0, w, h, null);
              else { g.setColor(Color.DARK_GRAY); g.fillRect(0,0,w,h); }

              int cx = w / 2;
              drawAvatar(g, p1img, p1Name == null ? "P1" : p1Name, cx, (int)(h*0.20));
              drawAvatar(g, p2img, p2Name == null ? "P2" : p2Name, cx, (int)(h*0.80));
          }
      ```

      - 상단 20% 위치에 P1,
      - 하단 80% 위치에 P2를 그림.

      `drawAvatar` 내부에서:

      - 이미지를 박스 사이즈에 맞게 스케일해서 중심에 그림.
      - 아래쪽에 이름 문자열을 **중앙 정렬**해서 출력.

      → 화면 구조:

      - 위쪽: P1 이미지 + 이름 라벨
      - 아래쪽: P2 이미지 + 이름 라벨

      이 구현 덕분에,

      - 각자의 화면에서 누가 위/아래에 있는지는 고정되어 있고,
      - 나중에 “내가 항상 아래”로 하기 위해 추가 조정도 가능.

      ### ChatDialog – 채팅창 구현 (JDialog)

      - `JDialog`로 분리된 윈도우.
      - `JTextArea area` : 채팅 내용 표시.
      - `JTextField input` : 사용자 입력.
      - `JButton sendBtn` : 전송 버튼.

      `input`의 Enter, `sendBtn` 클릭 시 `doSend()` 호출:

      ```java
      private void doSend() {
          String txt = input.getText();
          if (txt != null && !txt.trim().isEmpty()) {
              String mine = "[ME] " + myName + ": " + txt.trim();
              appendLine(mine, true);   // 내 화면에 먼저 출력
              net.send("CHAT " + txt.trim()); // 서버로 전송
              input.setText("");
          }
      }
      ```

      #### 특징

      - 내가 보낸 메시지는
        - 서버를 기다리지 않고 **먼저 내 채팅창에 표시**해서 지연을 줄임.
      - 하지만 서버는 `CHAT` 메시지를 받아서
        - `홍길동: ...` 형태로 브로드캐스트하기 때문에,
        - 상대방은 정확한 포맷으로 보게 됨.

      ### 줄 제한 – 오래된 채팅 삭제

      ```java
      private void trimTopLines(JTextArea ta, int maxLines) {
          Document doc = ta.getDocument();
          int lines = ta.getLineCount();
          if (lines <= maxLines) return;
          int over = lines - maxLines;
          int endOfCut = ta.getLineEndOffset(over - 1);
          doc.remove(0, endOfCut);
      }
      ```

      - 라인이 `MAX_CHAT_LINES(500)`를 넘으면,
      - 위쪽부터 잘라서 오래된 대화는 지우고  
        **최근 500줄만 유지**.

      ---

      ## 4-6. ImageLoader – 리소스에서 이미지 읽기

      ```java
      public class ImageLoader {
          public static BufferedImage load(String pathInResources) {
              try (InputStream in = ImageLoader.class.getResourceAsStream(pathInResources)) {
                  if (in == null) return null;
                  return ImageIO.read(in);
              } catch (Exception e) {
                  return null;
              }
          }
      }
      ```

      - `"/images/room_bg.png"`처럼,
        리소스 경로를 주면 `BufferedImage`를 반환.
      - 읽기에 실패하거나 파일이 없으면 `null` 반환.
      - `RoomCanvas`에서 `null`을 체크해서,
        - 배경이 없으면 회색으로 채우는 로직이 있음.

  - id: "5-chat-sequence-example"
    title: "5. 실제 “채팅 한 번 주고받는 흐름” 예시"
    content: |-
      ### 상황 설정

      - P1, P2가 서버에 접속해서 `ENTER_ROOM`까지 받고,
      - 둘 다 `GameRoomFrame` 화면 상태라고 가정.

      ---

      ### 1) P1이 Chat 창에서 “안녕?” 입력 후 Enter

      #### 💻 P1 클라이언트 쪽

      - `ChatDialog.doSend()` 실행:
        - `"[ME] P1이름: 안녕?"` 을 **자기 채팅창에 먼저 append**.
      - `net.send("CHAT 안녕?")` 호출 → 서버로 전송.

      ---

      ### 2) 서버에서 처리

      - P1의 `ClientHandler.run()`에서 `"CHAT 안녕?"` 한 줄 읽음.
      - `room.broadcastChat(name, "안녕?")` 호출.
      - `Room.broadcastChat`이
        - `CHAT P1이름: 안녕?` 을 **P1, P2 둘 다에게 전송**.

      ---

      ### 3) 두 클라이언트의 NetworkClient 리더 스레드

      - `onLine.accept("CHAT P1이름: 안녕?")` 호출.
      - 현재 리스너는 `GameRoomFrame.getLineConsumer()` 이므로:

        - `sender = "P1이름"`, `msg = "안녕?"`
        - `role = "P1"`로 판별.
        - P1에게는 `isMe == true`, P2에게는 `isMe == false`.

      - 최종 표시 문자열:
        - P1 화면: `[P1] [ME] P1이름: 안녕?`
        - P2 화면: `[P1] P1이름: 안녕?`

      이런 흐름을 이해하면,  
      나중에 `FIRE`, `AIM`, `TURN` 같은 게임 명령어가 추가되어도

      - “문자열 한 줄 보내고 받는다”라는 전체 구조는 그대로라는 걸
        설명할 수 있어.

  - id: "6-class-one-line-summary"
    title: "6. 발표용으로 각 클래스 한 줄씩 요약 (외워두면 좋음)"
    content: |-
      - **ServerGuiMain**  
        → “서버 프로그램의 시작점으로, 서버 GUI 창인 ServerFrame을 띄워줍니다.”

      - **ServerFrame**  
        → “포트 번호를 입력해서 서버를 켜고 끄며, 서버 로그를 화면에 보여주는 역할을 합니다.”

      - **ServerCore**  
        → “실제로 소켓을 열고, 두 명의 클라이언트를 받아서 한 개의 Room을 만들고, 각 클라이언트에 ClientHandler를 붙여줍니다.”

      - **Room**  
        → “한 게임방(2명)을 나타내며, READY/ENTER_ROOM/CHAT 같은 메시지를 두 플레이어에게 동시에 방송하는 역할을 합니다. 나중에는 턴, HP, 탄창 로직도 이 안에 들어갈 예정입니다.”

      - **ClientHandler**  
        → “클라이언트 한 명당 하나씩 돌아가는 서버 스레드로, 클라이언트가 보낸 문자열을 읽어서 Room에 전달하고, 그 결과를 다시 클라이언트들에게 보내줍니다.”

      - **Protocol**  
        → “서버와 클라이언트가 동일한 문자열 명령어를 쓰도록 상수로 정의해둔 클래스입니다.”

      - **ClientMain**  
        → “클라이언트 프로그램의 시작점으로, 접속 정보를 입력하는 StartFrame을 띄워줍니다.”

      - **StartFrame**  
        → “Host/IP, Port, Name을 입력받아서 서버에 연결하고, 성공하면 RoomFrame으로 넘깁니다.”

      - **NetworkClient**  
        → “클라이언트에서 서버와의 소켓 연결과 모든 송수신을 담당하는 통신 전담 클래스입니다.”

      - **RoomFrame**  
        → “두 명의 플레이어가 방에 모일 때까지 상태와 이름을 보여주고, 서버에서 ENTER_ROOM 메시지가 오면 실제 게임 화면(GameRoomFrame)으로 넘어가는 대기방입니다.”

      - **GameRoomFrame**  
        → “게임방 화면으로, 배경과 두 플레이어 이미지를 그리고, 상단에 내 역할(P1/P2)을 보여주며, 채팅 UI를 포함하고 있습니다. 현재는 채팅이 구현되어 있고, 이후 총/턴/HP UI가 추가됩니다.”

      - **ImageLoader**  
        → “리소스 폴더에서 PNG 이미지를 읽어와 BufferedImage로 돌려주는 유틸 클래스입니다.”
