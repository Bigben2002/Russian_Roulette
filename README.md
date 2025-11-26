🔫 러시안 룰렛 2인 네트워크 게임 (Russian Roulette Network Game)

"운과 전략이 교차하는 긴장감 넘치는 1:1 데스매치"

📋 목차

프로젝트 개요

목표와 현황

게임 규칙

시스템 구조

시스템 상세 로직 및 데이터 흐름

실행방법

조작키

기술 요약

최종본 요약 설명

1. 프로젝트 개요

프로젝트명: 러시안 룰렛 2인 네트워크 게임

개발환경: Java 17~21 / Eclipse IDE or IntelliJ IDEA

사용 기술: Swing GUI, TCP Socket 통신 (ServerSocket/Socket), Multi-threading

네트워크 구조: Central Server 1대 + Client 2대 (1:1 매칭 시스템)

2. 목표와 현황

"아이템 전략 요소 및 GUI 애니메이션 완성"

✅ 매칭 시스템: 대기실(Lobby) 및 Ready 상태 동기화 구현

✅ 코어 로직: 6칸 탄창 랜덤 생성 (실탄/공탄), 턴 기반 격발 시스템

✅ 전략 요소: 조준(AIM) 변경 (나에게 쏘기 vs 상대에게 쏘기)

✅ 아이템 구현: Heal(체력 회복), Search(탄 확인), Bomb(데미지 2배)

✅ GUI/UX: 리볼버 회전 애니메이션, 직관적인 HUD(HP, 잔탄, 아이템 슬롯)

3. 게임 규칙

1) 기본 사격 규칙

조준 대상 (AIM)

탄 종류 (Result)

결과 (Effect)

턴 변화 (Turn)

전략적 의미

SELF (나)

BLANK (공탄)

생존

유지 (내 턴)

리스크를 감수하고 연속 공격 기회 획득

SELF (나)

BULLET (실탄)

HP -1

교대 (상대 턴)

도박 실패, 자폭 데미지

ENEMY (상대)

BLANK (공탄)

효과 없음

교대 (상대 턴)

안전한 턴 넘기기

ENEMY (상대)

BULLET (실탄)

상대 HP -1

교대 (상대 턴)

공격 성공

2) 아이템 시스템 (전략 요소)

아이콘

아이템 명

효과 설명

H

Heal

자신의 HP를 1 회복합니다. (최대 체력 5 초과 불가)

S

Search

다음에 발사될 탄환이 실탄인지 공탄인지 몰래 확인합니다.

B

Bomb

다음 발사하는 실탄의 데미지가 2배가 됩니다. (공탄일 경우 소모됨)

4. 시스템 구조

src/
 ├─ server/
 │   ├─ ServerGuiMain.java      // 서버 프로그램 진입점 (GUI 실행)
 │   ├─ ServerFrame.java        // 서버 관리자 UI (포트 설정, 로그 모니터링)
 │   ├─ ServerCore.java         // 소켓 Accept 및 클라이언트 관리자 생성
 │   ├─ ClientHandler.java      // 클라이언트별 통신 스레드 (입력 수신)
 │   ├─ Room.java               // ★ 핵심 게임 로직 (탄창 관리, 아이템 처리, 판정)
 │   └─ Protocol.java           // 통신 프로토콜 상수 정의
 └─ client/
     ├─ ClientMain.java         // 클라이언트 프로그램 진입점
     ├─ StartFrame.java         // 접속 설정창 (IP, Port, 닉네임)
     ├─ RoomFrame.java          // 대기실 (Ready 상태 동기화)
     ├─ GameRoomFrame.java      // ★ 인게임 메인 UI, 키 입력, 렌더링(Canvas)
     ├─ NetworkClient.java      // 서버 송수신 처리 스레드
     └─ ImageLoader.java        // 이미지 리소스 로드 유틸리티



5. 시스템 상세 로직 및 데이터 흐름 (Data Flow)

본 게임은 Event-Driven 아키텍처를 기반으로 하며, 클라이언트의 요청(Command)을 서버가 수신하여 처리(Logic)하고, 결과를 전체 클라이언트에 전파(Broadcast)하는 방식으로 동작합니다.

🌊 1. 접속 및 매칭 (Handshake Phase)

서버 시작 (ServerCore):

start() 메서드 실행 시 ServerSocket이 열리고 acceptLoop() 스레드가 클라이언트 접속을 대기합니다.

클라이언트 접속 (StartFrame → ServerCore):

클라이언트 접속 시 accept()가 소켓을 반환합니다.

handshakeAndReadName()을 통해 HELLO 프로토콜을 주고받으며 닉네임을 확인합니다.

두 명의 클라이언트(p1, p2)가 접속하면 Room 객체를 생성하고 핸들러 스레드를 시작합니다.

대기실 입장 (Protocol → RoomFrame):

서버가 ENTER_ROOM 메시지를 전송하면, 클라이언트는 StartFrame을 닫고 RoomFrame(대기실)을 엽니다.

🌊 2. 게임 준비 및 시작 (Ready Phase)

Ready 요청 (RoomFrame):

사용자가 READY 버튼 클릭 시 net.send(Protocol.READY)를 호출합니다.

상태 동기화 (ClientHandler → Room):

서버 스레드(run())가 메시지를 수신하고 room.onReady()를 호출합니다.

Room 클래스 내부의 p1Ready, p2Ready 변수가 갱신되고, READY_STATE를 브로드캐스트하여 GUI의 버튼 상태를 변경합니다.

게임 인스턴스 생성 (Room → GameRoomFrame):

두 플레이어 모두 Ready 상태가 되면 room.startGame()이 트리거됩니다.

randomizeCylinder(): 6칸 배열(cyl[])에 실탄/공탄을 무작위 배정합니다.

giveItem(): 양측 아이템 배열을 초기화합니다.

GAME_START 프로토콜을 전송하면 클라이언트는 GameRoomFrame을 생성하여 게임 화면으로 전환합니다.

🌊 3. 인게임 데이터 흐름 (Game Loop)

모든 게임 로직은 Server Authoritative(서버 권한) 방식으로 Room 클래스에서 판정됩니다.

A. 조준 (Aiming)

Input: GameRoomFrame에서 방향키(↑/↓) 감지 (KeyBinding).

Request: net.send(Protocol.AIM + " ENEMY").

Logic: ClientHandler가 room.onAim() 호출 → Room 객체 내 aimP1 또는 aimP2 변수 업데이트.

Response: broadcast(AIM_UPDATE) → 모든 클라이언트의 canvas.repaint() 트리거 (총구 회전 애니메이션).

B. 아이템 사용 (Item Usage)

Input: 숫자키(1~6) 감지.

Request: net.send(Protocol.USE_ITEM + " SLOT=1").

Logic (Room.onUseItem):

Heal: hp 변수 증가 로직 수행 → FIRE_RESOLVE 메시지로 HP 갱신 전파.

Search: 현재 탄(cyl[idx]) 확인 → 요청자에게만 PEEK_RESULT 전송 (Secret Info).

Bomb: 해당 플레이어에게 HasBombEffect 플래그를 true로 설정.

Sync: 사용된 아이템 슬롯을 비우고 ITEM_UPDATE를 브로드캐스트.

C. 발사 및 판정 (Firing & Resolution)

Input: Spacebar 감지.

Request: net.send(Protocol.FIRE).

Logic (Room.onFire):

턴 검증: 요청자가 현재 턴(turn)인지 확인.

격발 판정: 현재 탄창 인덱스(idx)의 값 확인 (1=실탄, 0=공탄).

데미지 연산: 실탄일 경우 HasBombEffect 여부에 따라 데미지(1 or 2) 결정 및 hp 차감.

승패 확인: hp <= 0일 경우 GAME_OVER 상태로 전환.

턴 교체 규칙: 공탄을 자신에게 쏜 경우만 턴 유지, 그 외에는 turn 변수 교체.

재장전: idx >= 6일 경우 randomizeCylinder() 재실행.

Response: FIRE_RESOLVE (결과, 남은 체력, 잔탄 수) 및 TURN 메시지를 순차적으로 전송하여 클라이언트 GUI 동기화.

6. 실행방법

1단계: 서버 실행

server.ServerGuiMain 클래스를 실행합니다.

Server GUI 창이 뜨면 Port 번호(기본 7777)를 확인합니다.

Game Start 버튼을 눌러 서버를 엽니다. (로그창에 "Listening on 7777" 확인)

2단계: 클라이언트 1 접속 (Player 1)

client.ClientMain 클래스를 실행합니다.

StartFrame에서 IP(127.0.0.1), Port(7777), Name(User1)을 입력합니다.

Connect 버튼을 클릭합니다.

대기실(Lobby)로 입장하여 STATUS: WAITING...을 확인합니다.

3단계: 클라이언트 2 접속 (Player 2)

client.ClientMain 클래스를 한 번 더 실행하여 새 창을 엽니다.

Name을 다르게(User2) 입력하고 접속합니다.

두 클라이언트 모두 대기실 화면에서 Ready 버튼이 활성화됩니다.

4단계: 게임 시작

두 클라이언트가 각각 READY 버튼을 누릅니다.

양쪽 모두 Ready 상태가 되면 자동으로 게임 화면(GameRoomFrame)으로 전환됩니다.

P1부터 턴이 시작됩니다.

7. 조작키

키 (Key)

기능 (Function)

비고

↑ (Up)

상대방(ENEMY) 조준

기본 상태

↓ (Down)

자신(SELF) 조준

턴 유지 전략 시 사용

SPACE

격발 (Fire)

현재 조준된 대상으로 발사

1 ~ 6

아이템 사용

해당 슬롯의 아이템 즉시 사용

8. 기술 요약

Socket Networking: BufferedReader / PrintWriter를 이용한 문자열 기반 프로토콜 통신.

Protocol Design: Key-Value 형태의 명령어 (CMD KEY=VALUE)로 확장성 확보.

Swing Graphics: Graphics2D를 활용한 리볼버 회전 로직 및 투명 PNG 리소스 합성.

Thread Safety: 서버의 Room 클래스 메서드에 synchronized를 적용하여 동시성 문제 해결.

9. 최종본 요약 설명

이 프로젝트는 단순한 운 게임을 넘어, 네트워크 동기화와 **전략적 요소(아이템, 타겟 변경)**를 결합한 완성도 높은 2인용 게임입니다. 서버가 게임의 모든 상태(탄창, 체력, 턴)를 권한을 가지고 관리(Authoritative Server)하며, 클라이언트는 이를 시각화하는 구조로 설계되어 보안성과 안정성을 갖추었습니다.
