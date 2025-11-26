Russian-Roulette-Project

한성대학교 3학년 2학기 네트워크프로그래밍 프로젝트
팀원 <br>
이경석 - 2171073 <br>
김준현 - 2171062

🎯 Russian Roulette -- 2인 네트워크 게임 (Phase-1 확장본)
🧱 프로젝트 개요

프로젝트명: 러시안 룰렛 2인 네트워크 게임

개발환경: Java 17~21 / Eclipse IDE

사용 기술: Swing GUI, TCP Socket 통신 (ServerSocket / Socket), UTF-8 스트림

네트워크 구조: 서버 1대 + 클라이언트 2대 (LAN 환경 접속)

같은 Wi-Fi 환경에서 서버 IP와 포트를 입력하면 2명이 접속해 1:1 대전을 진행합니다.

🧩 프로젝트 목표
✅ 현재 구현 상태 (Phase-1 + 아이템 확장)

"카드 시스템 없이, 아이템 기반의 러시안 룰렛 전투 구현"

2인 매칭 / 대기방 / READY 상태 동기화

6칸 탄창 랜덤 생성 (실탄·공탄)

조준(AIM) – SELF / ENEMY

발사(FIRE) – 규칙에 따른 HP 감소 및 턴 교대

HP(라이프 이미지), 잔탄 수, 현재 턴, 탄창 진행 상태 표시

실탄/공탄 모두 소진 시 자동 재장전 + 아이템 재지급

아이템 시스템 3종 구현 (카드 → 아이템으로 변경)

H = Heal (HP 회복)

S = Search (다음 탄환이 실탄/공탄인지 확인)

B = Bomb (다음 실탄 피해 2배)

이후 단계 예고 (계획)
Phase	내용
Phase-2	GUI 디테일/애니메이션 보완, UX 개선
Phase-3	아이템 밸런싱·종류 확장, 로그/리플레이 기능
Phase-4	예외 처리 / 에러 복구 / 재접속 처리
Phase-5	멀티룸 지원, 옵션 메뉴 등 확장

사운드 효과는 이번 버전에서는 적용하지 않기로 결정
기존 "카드 시스템" 계획은 아이템(Heal/Search/Bomb) 시스템으로 완전히 대체되었습니다.

⚙️ 게임 규칙
기본 규칙
조준 대상	탄 종류	결과	턴 변화
SELF	BLANK	아무 일 없음	유지
SELF	BULLET	자기 HP − 1 (또는 2)	교대
ENEMY	BLANK	아무 일 없음	교대
ENEMY	BULLET	상대 HP − 1 (또는 2)	교대

HP: 각 플레이어 5에서 시작, 0 이하가 되면 패배

폭탄(Bomb) 효과가 활성화된 상태에서 맞으면 2 피해

탄창: 6칸, 실탄/공탄 랜덤 구성

탄창 6발을 모두 사용하면:

서버가 자동으로 새로운 6칸 탄창을 랜덤 생성

두 플레이어 모두에게 아이템 일부를 다시 지급

RELOAD 메시지로 남은 실탄/공탄 수를 클라이언트에 통보

아이템 규칙 (H / S / B)
코드	이름	효과
H	Heal	자신의 HP를 1 회복 (최대 5). HP가 이미 5이면 사용 불가. 내 턴에서만 사용 가능
S	Search	현재 총구 위치의 탄이 실탄인지/공탄인지 서버가 확인 후, 나에게만 알려줌.
B	Bomb	다음에 내가 쏘는 실탄이 명중할 경우 피해 2배 (HP 2 감소). 한 번 사용하면 효과 소진.

아이템은 최대 6칸 가방(슬롯 1~6)에 저장됩니다.

아이템 사용 시:

클라이언트: USE_ITEM SLOT=n 전송

서버: 효과 처리 후,

ITEM_UPDATE로 양쪽 가방 상태를 다시 브로드캐스트

필요 시 FIRE_RESOLVE RESULT=HEAL ... 또는 PEEK_RESULT TYPE=... 전송

🧩 시스템 구조
src/
 ├─ server/
 │   ├─ ServerGuiMain.java   // 서버 프로그램 진입점 (GUI 실행)
 │   ├─ ServerFrame.java     // 포트 입력 + Start/Stop + 서버 로그 출력
 │   ├─ ServerCore.java      // ServerSocket 관리, 클라이언트 2명 수락, Room 생성
 │   ├─ ClientHandler.java   // 클라이언트별 스레드, 수신 문자열 → Room으로 전달
 │   ├─ Room.java            // 게임 로직 (턴, HP, 탄창, 아이템, 승패 판정)
 │   └─ Protocol.java        // 통신 프로토콜 문자열 상수 정의
 └─ client/
     ├─ ClientMain.java      // 클라이언트 진입점 (StartFrame 띄움)
     ├─ StartFrame.java      // Host / Port / Name 입력 화면
     ├─ RoomFrame.java       // 대기방(로비): P1/P2 이름, READY 상태, GAME_START까지 대기
     ├─ GameRoomFrame.java   // 실제 게임 화면 (이미지, HP/탄창/아이템, 채팅, 키 입력)
     ├─ NetworkClient.java   // 클라이언트 소켓 연결 및 수신 전용 스레드 관리
     └─ ImageLoader.java     // /resources/images/ 하위 이미지 로더
resources/
 └─ images/
     ├─ room_bg.png
     ├─ player1.png
     ├─ player2.png
     ├─ gun.png
     ├─ life.png
     ├─ Heal.png
     ├─ Search.png
     └─ bomb.png

🔌 실행 방법
🖥️ 서버 실행 (Server GUI)

Eclipse에서 server.ServerGuiMain을 Java Application으로 실행

뜨는 ServerFrame 창에서:

Port: 사용할 포트 번호 입력 (예: 7777)

Start 버튼 클릭

아래 로그 영역에 예시처럼 출력되면 준비 완료:

[Server] Listening on 7777


Stop 버튼을 누르면 ServerCore.stop()이 호출되어 ServerSocket이 닫히고,
새로운 접속은 막힙니다.

💻 클라이언트 실행

Eclipse에서 client.ClientMain을 2번 실행 (각각 다른 프로세스로 실행)

각 클라이언트에서 StartFrame 창에 다음 정보 입력:

Host: 서버가 실행 중인 PC의 IP

같은 PC에서 테스트할 경우 127.0.0.1

Port: 서버와 동일한 포트 (예: 7777)

Name: 각 플레이어 닉네임 입력

Connect 버튼 클릭

연결 성공 시 RoomFrame(대기방)으로 화면 전환

서버 로그에는 클라이언트 접속 내역과 방 생성 로그가 남음

🕹 대기방(RoomFrame) 동작

서버에 두 명이 접속하면 Room이 생성됩니다.

클라이언트는 다음 메시지들을 받으며 상태를 갱신합니다.

ROOM_STATUS WAITING 1/2

첫 번째 클라이언트가 들어왔을 때 표시

ROOM_STATUS WAITING 2/2

두 번째 클라이언트까지 입장 완료

ROOM_CREATED P1=... P2=...

누가 P1 / P2인지 이름을 알려줌

READY_STATE P1_READY=true P2_READY=false

각 플레이어의 READY 버튼 클릭 상태를 계속 브로드캐스트

플레이 흐름

두 클라이언트 모두 RoomFrame에서 READY 버튼 클릭

서버(Room)가 onReady()에서 두 플레이어가 모두 준비되면:

랜덤 탄창 생성, 초기 HP/아이템/턴 상태 설정

GAME_START P1=... P2=... B=... K=... P1_ITEMS=... P2_ITEMS=... 송신

RELOAD, TURN, AIM_UPDATE까지 연속으로 브로드캐스트

클라이언트(RoomFrame)는 GAME_START를 받으면:

GameRoomFrame을 생성하면서

P1/P2 이름

초기 실탄 수(B)

초기 공탄 수(K)

초기 아이템 목록(P1_ITEMS, P2_ITEMS)
를 전달

네트워크 수신 콜백을 GameRoomFrame으로 넘겨준 뒤 자기 자신(RoomFrame)을 dispose()

🎮 게임 화면 (GameRoomFrame) 조작 키
키	기능
↑ / ↓	조준 대상 변경 (내 화면 기준으로 나 / 상대 방향)
SPACE	발사(FIRE) – 내 턴일 때만 유효
1 ~ 6	해당 슬롯 아이템 사용 (H/S/B)
Chat 버튼	채팅창 열기 (대화 내용은 양쪽 화면에 공유)
창 닫기(X)	EXIT_ROOM 전송 후 소켓 종료, 게임 종료

턴이 아닌 플레이어가 SPACE 또는 아이템 키를 누를 경우,
클라이언트에서 tryFire() / tryUseItem() 단계에서 바로 무시합니다.

게임 종료 후에는 GAME_OVER WIN=P1|P2|DRAW가 도착하고,
화면 상단에 승리/패배 배너가 표시됩니다.

🧠 기술 요약
구분	내용
통신 방식	TCP 소켓 (ServerSocket / Socket, UTF-8 텍스트 스트림)
멀티스레드	서버: 클라이언트당 1 스레드(ClientHandler) / 접속 대기 스레드 1개
GUI	Swing 기반 (JFrame / JPanel / 커스텀 페인팅 및 이미지 렌더링)
데이터 교환	사람이 읽을 수 있는 텍스트 프로토콜 (키=값 형태)
로직 구조	서버(Room)가 단일 진실 소스 — HP, 탄창, 턴, 아이템은 모두 서버 기준
포커스 처리	턴/화면 전환마다 requestFocusInWindow()로 키 입력 포커스 유지
📡 소켓 & 데이터 흐름 정리

아래는 코드 단위의 실제 흐름을 기준으로,
서버/클라이언트/클라이언트 간 데이터가 어떻게 움직이는지 정리한 것입니다.

1️⃣ 접속 & 핸드셰이크

클라이언트 연결 시도

NetworkClient.connect(host, port, name)

내부에서 new Socket(host, port)로 TCP 연결 생성

HELLO / 이름 교환

서버 쪽 ServerCore.acceptLoop()에서 handshakeAndReadName() 호출

서버 → 클라: HELLO 한 줄 전송

클라 → 서버: 입력한 name 문자열 한 줄 전송

서버는 이 이름을 ClientHandler 생성자 인자로 넘겨 닉네임으로 사용

Room 생성

두 번째 클라이언트까지 접속하면

ClientHandler p1, ClientHandler p2 생성

Room room = new Room(p1, p2, n1, n2)

두 Handler 스레드를 각각 시작 (new Thread(p1).start(), new Thread(p2).start())

room.announceCreatedAndReady()로

ROOM_STATUS, ROOM_CREATED 등을 두 클라이언트에게 브로드캐스트

2️⃣ READY → GAME_START까지
클라이언트 측

RoomFrame의 READY 버튼 클릭:

net.send(Protocol.READY);

NetworkClient.send() 내부에서

PrintWriter.println("READY") → 서버로 전송

서버 측

ClientHandler.run()에서 수신 루프:

while ((line = in.readLine()) != null) {
    String trimmedLine = line.trim();

    if (trimmedLine.equals(Protocol.READY)) {
        if (room != null) room.onReady(this);
        continue;
    }
}


Room.onReady(ClientHandler who)

p1Ready, p2Ready 플래그 갱신

둘 다 true가 되면 startGame() 호출

startGame() 내부 동작 (서버 기준)

현재 탄창, 아이템, 턴 상태를 기반으로:

broadcast(Protocol.GAME_START
        + " P1=" + n1
        + " P2=" + n2
        + " B=" + bulletsLeft
        + " K=" + blanksLeft
        + " P1_ITEMS=" + p1ItemsStr
        + " P2_ITEMS=" + p2ItemsStr);

broadcast(Protocol.RELOAD + " " + idx + "/6 B=" + bulletsLeft + " K=" + blanksLeft);
broadcast(Protocol.TURN   + " P" + turn);
broadcast(Protocol.AIM_UPDATE + " WHO=P1 TARGET=ENEMY");
broadcast(Protocol.AIM_UPDATE + " WHO=P2 TARGET=ENEMY");


이 모든 문자열은 Room.broadcast() → ClientHandler.send() → out.println(...) 형식으로
두 클라이언트 모두에게 동시에 전송됩니다.

클라이언트 게임 시작

RoomFrame.handleServerLine() 에서 GAME_START 처리:

if (line.startsWith(Protocol.GAME_START + " ")) {
    String initialBullets = parseKV(line, "B");
    String initialBlanks  = parseKV(line, "K");
    String p1InitialItems = parseKV(line, "P1_ITEMS");
    String p2InitialItems = parseKV(line, "P2_ITEMS");

    GameRoomFrame gf = new GameRoomFrame(p1Name, p2Name, myName, net,
                                         parseIntSafe(initialBullets, 0),
                                         parseIntSafe(initialBlanks, 0),
                                         p1InitialItems, p2InitialItems);
    net.setOnLine(gf.getLineConsumer());
    gf.setVisible(true);
    dispose();  // 로비 창 닫기
}

3️⃣ 턴 내 조준(AIM)과 발사(FIRE) 흐름
(1) 조준 변경

플레이어가 ↑ / ↓ 키를 누르면:

// GameRoomFrame.setupKeyBindings()
if ("P2".equals(myRole)) {
    // P2는 화면 상/하 감각을 뒤집어 매핑
} else {
    // P1 기준 ↑: ENEMY, ↓: SELF
}
net.send("AIM SELF" or "AIM ENEMY");


서버 ClientHandler:

if (trimmedLine.startsWith(Protocol.AIM + " ")) {
    String target = trimmedLine.substring(Protocol.AIM.length() + 1).trim();
    if (room != null) room.onAim(this, target);
    continue;
}


Room.onAim():

if (who == p1) { aimP1 = t; playerRole = "P1"; }
else if (who == p2) { aimP2 = t; playerRole = "P2"; }

broadcast("AIM_UPDATE WHO=P1 TARGET=SELF/ENEMY");


GameRoomFrame.handleServerLine()에서 AIM_UPDATE를 받아
각 클라이언트가 자신의 화면에서 총 이미지 회전 각도를 갱신합니다.

(2) 발사(FIRE)

클라이언트 입력

내 턴 + SPACE → tryFire():

private void tryFire() {
    if (!myRole.equals(currentTurn)) return; // 턴 아니면 무시
    net.send(Protocol.FIRE);
}


서버 처리

ClientHandler에서 FIRE 감지 후:

if (trimmedLine.equals(Protocol.FIRE)) {
    if (room != null) room.onFire(this);
    continue;
}


Room.onFire(ClientHandler who) 핵심 단계

현재 턴인지 확인 (if (shooter != turn) return;)

현재 조준 대상 (SELF / ENEMY) 확인

탄창 배열 cyl[idx]에서 현재 칸의 탄 종류(1=실탄, 0=공탄) 확인

Bomb 효과 여부에 따라 damage = 1 또는 2

맞은 쪽 HP 감소, bulletsLeft / blanksLeft 감소

FIRE_RESOLVE 브로드캐스트:

String r = (result == 1) ? "BULLET" : "BLANK";
String targetLabel = hitSelf ? "SELF" : "ENEMY";

broadcast(Protocol.FIRE_RESOLVE
        + " RESULT=" + r
        + " TARGET=" + targetLabel
        + " HP1=" + hp1 
        + " HP2=" + hp2
        + " B_LEFT=" + bulletsLeft
        + " K_LEFT=" + blanksLeft
        + " SHOT=" + idx + "/6"
        + " DMG=" + damage);


HP가 0 이하이면 GAME_OVER WIN=P1|P2|DRAW 전송

탄창을 다 쓰면(6발 소모) randomizeCylinder()로 새 탄창 생성,
giveItem()으로 양쪽 아이템 지급, ITEM_UPDATE + RELOAD 브로드캐스트

턴 교대 규칙:

실탄이면 무조건 턴 교대

공탄 + SELF 조준이면 턴 유지

공탄 + ENEMY 조준이면 턴 교대
→ 계산 후 TURN P1|P2 + AIM_UPDATE 다음 턴 조준 상태까지 동기화

클라이언트에서 결과 반영

GameRoomFrame.handleServerLine()에서 FIRE_RESOLVE 수신 →
parseFireResolve()로 HP, 남은 탄, 샷 인덱스 등을 반영하고,
화면 상단에 결과(실탄/공탄, 맞은 대상, 피해량)를 표시합니다.

4️⃣ 아이템 사용 흐름 (H/S/B)

클라이언트에서 1~6 키 입력

setupKeyBindings()에서 1~6 키를 "USE_ITEM_n" 액션에 매핑

tryUseItem(slot) 호출

private void tryUseItem(int slot) {
    boolean isMyTurn = myRole.equals(currentTurn);
    String[] myItemArray = ("P1".equals(myRole) ? p1Items : p2Items);
    String item = myItemArray[slot - 1];

    // 빈 슬롯, 잘못된 인덱스, 턴 아님, HP 가득인 Heal 등은 사용 무시
    ...
    myItemArray[slot - 1] = "-";
    if ("B".equals(item)) bombStatus = "강화!!! (다음 발사 실탄 데미지 2배)";
    canvas.repaint();
    net.send(Protocol.USE_ITEM + " SLOT=" + slot);
}


서버 처리 (Room.onUseItem)

슬롯 번호 검증 (1~6)

어떤 플레이어인지에 따라 p1Items 또는 p2Items 배열 사용

아이템 종류에 따라 분기:

switch (item) {
    case "H":
        // 내 HP 1 회복, 최대 5
        break;
    case "S":
        // 현재 cyl[idx]가 BULLET인지 BLANK인지 확인 후
        // who.send("PEEK_RESULT TYPE=BULLET/BLANK");
        break;
    case "B":
        // 다음 실탄 데미지 2배 플래그 설정
        break;
}

myItems[slotNum - 1] = "-"; // 사용한 아이템 소모
broadcastItemStatus();      // ITEM_UPDATE WHO=P1/P2 ITEMS=... 두 명에게 전송


클라이언트에서 UI 갱신

ITEM_UPDATE → 각 플레이어의 아이템 가방(6칸 슬롯)에 그림 아이콘(Heal/Search/Bomb) 재배치

PEEK_RESULT → 화면 한쪽에 "다음 탄: BULLET" / "BLANK" 안내

Bomb 상태 → 폭탄 효과 텍스트를 보여주고, 다음 실탄 발동 시 서버에서 2 데미지 처리 후 플래그 초기화

5️⃣ 채팅 및 퇴장

채팅

GameRoomFrame의 Chat 버튼 → ChatDialog 열기

입력 후 Enter → Protocol.CHAT + " " + txt 전송

서버 ClientHandler:

if (trimmedLine.startsWith(Protocol.CHAT + " ")) {
    String msg = trimmedLine.substring(Protocol.CHAT.length() + 1).trim();
    if (room != null && !msg.isEmpty()) room.broadcastChat(nickname, msg);
    continue;
}


Room.broadcastChat() → CHAT 닉네임: 메시지를 양쪽에 브로드캐스트

GameRoomFrame.handleServerLine()에서 CHAT 수신 시 ChatDialog에 그대로 append

퇴장

사용자가 게임 창 X 버튼 클릭 →
EXIT_ROOM 전송 후 NetworkClient.close()로 소켓 종료

상대방 클라이언트는 EXIT_ROOM을 수신하면:

"상대방이 나갔습니다. 게임 종료." 메시지 표시

자신의 소켓도 닫고 창 종료

✅ 구현 완료 체크리스트 (현재 코드 기준)

 서버/클라이언트 간 정상 연결 (HELLO 핸드셰이크 포함)

 두 명 입장 후 Room 생성 및 닉네임 동기화

 READY 상태 및 GAME_START 동기화

 HP / 턴 / 탄창 / 남은 실탄·공탄 수 동기화

 조준(AIM) SELF / ENEMY 동기화 및 총 이미지 회전

 실탄/공탄 규칙에 따른 HP 감소 및 턴 교대 로직

 탄창 6발 소진 시 자동 재장전 + 아이템 재지급

 Heal/Search/Bomb 아이템 3종 구현 및 서버 기준 처리

 FIRE_RESOLVE / GAME_OVER / RELOAD / ITEM_UPDATE / PEEK_RESULT 처리

 채팅 기능 (서버 중계, 양쪽 클라이언트 표시)

 EXIT_ROOM 처리 및 상대 퇴장 감지

👨‍💻 개발자 코멘트

이 프로젝트는 Eclipse + JDK 21 환경에서
**Socket 네트워크 통신과 Swing GUI, 그리고 게임 로직(턴제, 아이템, 상태 동기화)**을 동시에 연습하기 위해 설계되었습니다.

서버는 항상 **단일 진실 소스(HP, 턴, 탄창, 아이템 상태)**를 유지하고,
클라이언트는 오직 서버가 브로드캐스트하는 텍스트 프로토콜에만 의존해 화면을 갱신하도록 구성되어 있습니다.

📅 개발 진행 상태

Phase-1: ✅ 완료 (기본 통신 및 러시안 룰렛 전투 규칙 구현)

아이템 확장: ✅ 완료 (Heal / Search / Bomb, 재장전 시 아이템 재지급 포함)

GUI(이미지): ✅ room_bg / player / gun / life / item 아이콘 적용

예외 처리·멀티룸: 🔜 추후 확장 예정
