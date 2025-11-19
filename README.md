README: |
  # 🎮 러시안 룰렛 2인 네트워크 게임  
  ### Java Socket + Swing GUI 기반 2인 실시간 네트워크 게임 프로젝트

  ## 1. 프로젝트 개요

  - 목표  
    Java 소켓 통신(TCP)과 Swing GUI를 활용해  
    2명의 플레이어가 서로 다른 PC에서 접속해서 게임방에 입장하고,  
    채팅하며 이후 러시안 룰렛 게임을 진행할 수 있는 구조를 구현하는 프로젝트이다.

  - 구조 핵심  
    - 서버 = 게임 상태를 통제하는 "진실의 원천"  
    - 클라이언트 = 화면 UI 표현 및 입력 담당  
    - 모든 결정(턴, 탄 종류, HP 변화)은 서버가 확정 → 클라가 렌더링

  - 현재 구현된 기능  
    * 서버 GUI(포트 입력/시작/정지)  
    * 클라이언트 접속 UI(IP/Port/이름)  
    * 2명 접속 시 자동 방 생성  
    * 대기방(RoomFrame) → 게임방(GameRoomFrame) 자동 전환  
    * 채팅 기능 완전 동작  
    * 배경/아바타/이름 표시 UI 구현  

  ## 2. 디렉터리 구조

  src/
    server/
      ServerGuiMain.java      # 서버 시작 main
      ServerFrame.java        # 서버 GUI
      ServerCore.java         # 서버 핵심(acceptLoop, Room 생성)
      Room.java               # 방 관리(2인)
      ClientHandler.java      # 클라별 스레드
      Protocol.java           # 통신 명령어 상수

    client/
      ClientMain.java         # 클라이언트 시작점
      StartFrame.java         # 접속 정보 입력
      RoomFrame.java          # 대기방
      GameRoomFrame.java      # 게임방(채팅 포함)
      NetworkClient.java      # 서버와 송수신 관리
      ImageLoader.java        # 이미지 로더

  resources/
    images/ (room_bg.png, player1.png, player2.png)
    sound/ (향후 사용)

  ## 3. 실행 방법

  ### 서버 실행
  1. ServerGuiMain 실행
  2. 포트 입력 후 "방 만들기(서버 시작)" 클릭
  3. 로그에 Listening 메시지 출력 → 접속 대기

  ### 클라이언트 실행
  1. ClientMain 실행
  2. Host/Port/Name 입력
  3. Connect → RoomFrame(대기방)
  4. 두 명이 모이면 GameRoomFrame으로 자동 전환
  5. Chat 버튼 → 채팅 가능

  ## 4. 전체 데이터 흐름

  ### 서버 흐름
  - ServerFrame → ServerCore.start()  
  - ServerSocket 생성  
  - acceptLoop()에서 P1, P2 순서대로 접속  
  - 이름 수신  
  - ClientHandler 두 개 생성 후 Room에 연결  
  - ROOM_CREATED, READY, ENTER_ROOM 두 명에게 전송  
  - 이후 CHAT 등 메시지 처리

  ### 클라이언트 흐름
  - StartFrame → RoomFrame 생성  
  - NetworkClient.connect()  
    - 서버 HELLO 수신  
    - 이름 송신  
    - reader 스레드 시작  
  - ROOM_STATUS / ROOM_CREATED / ENTER_ROOM 처리  
  - ENTER_ROOM 받으면 GameRoomFrame 생성  
  - net.setOnLine(...)으로 메시지 수신처 교체  
  - GameRoomFrame에서 채팅/화면 표현

  ## 5. 주요 클래스별 역할

  ### ServerGuiMain  
  - 서버 프로그램 시작점, ServerFrame 표시

  ### ServerFrame  
  - 포트 입력  
  - 서버 시작/정지  
  - 로그 출력  

  ### ServerCore  
  - ServerSocket 생성  
  - 클라이언트 2명 접속 시 Room 생성  
  - ClientHandler 스레드 2개 실행  

  ### Room  
  - P1/P2 핸들러 보유  
  - broadcast()로 두 명에게 메시지 전송  
  - announceCreatedAndReady()로  
    ROOM_CREATED / READY / ENTER_ROOM 방송  

  ### ClientHandler  
  - 클라이언트 입력(readLine)을 계속 대기  
  - CHAT 메시지 수신 시 Room.broadcastChat() 호출  

  ### ClientMain  
  - 클라이언트 시작점(StartFrame 표시)

  ### StartFrame  
  - IP / Port / Name 입력 UI  
  - 연결 성공 시 RoomFrame으로 전환  

  ### RoomFrame  
  - ROOM_STATUS / ROOM_CREATED / ENTER_ROOM 수신  
  - ENTER_ROOM 시 GameRoomFrame 생성  
  - net.setOnLine()으로 리스너 교체  

  ### GameRoomFrame  
  - 실제 게임 화면(배경 + 아바타 + 채팅 버튼)  
  - 채팅 메시지 파싱 및 출력  
  - ChatDialog 관리  

  ### NetworkClient  
  - 서버와 송수신 담당  
  - reader 스레드로 서버 메시지 지속 수신  
  - onLine.accept(...) 호출  

  ### ImageLoader  
  - 리소스(이미지) 로딩  

  ## 6. 채팅 메시지 흐름 예시

  1) P1이 ChatDialog에서 "안녕?" 입력  
  2) 클라 즉시 `[ME] P1: 안녕?` 표시  
  3) net.send("CHAT 안녕?")  
  4) 서버 ClientHandler가 수신  
  5) Room.broadcastChat("P1", "안녕?")  
  6) 두 클라 모두:  
     - P1 화면: `[P1][ME] P1: 안녕?`  
     - P2 화면: `[P1] P1: 안녕?`  

  ## 7. 향후 확장 (러시안 룰렛 로직)

  - GunChamber 6칸 랜덤(BULLET/BLANK)  
  - TURN, AIM, FIRE, FIRE_RESOLVE 확장  
  - HP 시스템(각 5)  
  - 재장전 + 카드 드로우  
  - HUD / 사운드 추가  

  원칙:  
  **“서버는 모든 판정을 내리고, 클라이언트는 표현만 한다.”**

  ## 8. 결론

  이 README는  
  서버/클라이언트 구조, 통신 흐름, UI 구성,  
  채팅 작동 방식까지 모든 설명을 한 문서로 통합한 것이다.

  발표/문서 제출/보고서용으로 그대로 사용할 수 있다.
