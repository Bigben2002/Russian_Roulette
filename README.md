<div align="center">🔫 러시안 룰렛 : 2인 네트워크 게임</div>

<div align="center">

"운과 전략이 교차하는 긴장감 넘치는 1:1 데스매치"

</div>

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

1. 🧐 프로젝트 개요

구분

내용

프로젝트명

러시안 룰렛 2인 네트워크 게임 (Russian Roulette Network Game)

개발환경

Java 17~21 / Eclipse IDE or IntelliJ IDEA

사용 기술

Swing GUI, TCP Socket 통신 (ServerSocket/Socket), Multi-threading

네트워크

Central Server 1대 + Client 2대 (1:1 매칭 시스템)

2. 🎯 목표와 현황

"아이템 전략 요소 및 GUI 애니메이션 완성"

✅ 매칭 시스템: 대기실(Lobby) 및 Ready 상태 동기화 구현

✅ 코어 로직: 6칸 탄창 랜덤 생성 (실탄/공탄), 턴 기반 격발 시스템

✅ 전략 요소: 조준(AIM) 변경 (나에게 쏘기 vs 상대에게 쏘기)

✅ 아이템 구현: Heal(체력 회복), Search(탄 확인), Bomb(데미지 2배)

✅ GUI/UX: 리볼버 회전 애니메이션, 직관적인 HUD(HP, 잔탄, 아이템 슬롯)

3. 🔫 게임 규칙

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

4. 📂 디렉토리 구조

src/
 ├─ 📂 server/
 │   ├─ 📜 ServerGuiMain.java      // 서버 프로그램 진입점 (GUI 실행)
 │   ├─ 📜 ServerFrame.java        // 서버 관리자 UI (포트 설정, 로그 모니터링)
 │   ├─ 📜 ServerCore.java         // 소켓 Accept 및 클라이언트 관리자 생성
 │   ├─ 📜 ClientHandler.java      // 클라이언트별 통신 스레드 (입력 수신)
 │   ├─ 📜 Room.java               // ★ 핵심 게임 로직 (탄창 관리, 아이템 처리, 판정)
 │   └─ 📜 Protocol.java           // 통신 프로토콜 상수 정의
 └─ 📂 client/
     ├─ 📜 ClientMain.java         // 클라이언트 프로그램 진입점
     ├─ 📜 StartFrame.java         // 접속 설정창 (IP, Port, 닉네임)
     ├─ 📜 RoomFrame.java          // 대기실 (Ready 상태 동기화)
     ├─ 📜 GameRoomFrame.java      // ★ 인게임 메인 UI, 키 입력, 렌더링(Canvas)
     ├─ 📜 NetworkClient.java      // 서버 송수신 처리 스레드
     └─ 📜 ImageLoader.java        // 이미지 리소스 로드 유틸리티


5. 🔄 시스템 상세 로직 및 데이터 흐름 (Data Flow)

본 게임은 Event-Driven 아키텍처를 기반으로 하며, 클라이언트의 요청(Command)을 서버가 수신하여 처리(Logic)하고, 결과를 전체 클라이언트에 전파(Broadcast)하는 방식으로 동작합니다.

🌊 1. 접속 및 매칭 (Handshake Phase)

서버 시작 (ServerCore): start() 실행 시 ServerSocket 오픈, acceptLoop() 대기.

클라이언트 접속 (StartFrame): 소켓 연결 → HELLO 프로토콜 교환 → 닉네임 등록.

대기실 입장: 두 명 접속 시 서버가 Room 생성 후 ENTER_ROOM 전송.

🌊 2. 게임 준비 및 시작 (Ready Phase)

Ready 요청: RoomFrame에서 버튼 클릭 → Protocol.READY 전송.

상태 동기화: 서버가 양쪽 Ready 확인 시 startGame() 트리거.

게임 생성: 탄창 랜덤 배정(randomizeCylinder) 및 아이템 지급 후 GAME_START 전송.

🌊 3. 인게임 데이터 흐름 (Game Loop)

핵심 원칙: Server Authoritative (서버 권한 검증)

A. 조준 (Aiming)

Input: ↑/↓ 키 입력

Flow: Client → Protocol.AIM → Server(Room.onAim) → AIM_UPDATE → All Clients

Result: 화면의 권총 회전 애니메이션 동기화

B. 아이템 사용 (Item Usage)

Input: 1~6 숫자키 입력

Flow: Client → Protocol.USE_ITEM → Server(Room.onUseItem) → 결과 처리

Logic:

Heal: HP 증가 후 방송.

Search: 요청자에게만 PEEK_RESULT 전송 (비공개 정보).

Bomb: 다음 실탄 데미지 2배 플래그 설정.

C. 발사 및 판정 (Firing)

Input: Spacebar 입력

Flow: Client → Protocol.FIRE → Server(Room.onFire)

Logic:

턴 검증 및 탄창 확인(실탄/공탄).

데미지 계산 및 HP 차감.

승패 판정 (hp <= 0 ? Game Over).

룰에 따른 턴 교체 및 재장전 처리.

6. ⚙️ 실행방법

1단계: 서버 실행

server.ServerGuiMain 실행.

Port(기본 7777) 확인 후 Game Start 클릭.

2단계: 클라이언트 접속

client.ClientMain 실행 (2개 창 띄우기).

각각 다른 이름(예: User1, User2)으로 접속.

IP: 127.0.0.1 / Port: 7777.

3단계: 게임 시작

두 클라이언트 모두 READY 버튼 클릭.

게임 화면 진입 후 P1부터 시작.

7. 🎮 조작키

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

8. 🛠 기술 요약

Socket Networking: BufferedReader / PrintWriter를 이용한 문자열 기반 프로토콜 통신.

Protocol Design: Key-Value 형태의 명령어 (CMD KEY=VALUE)로 확장성 확보.

Swing Graphics: Graphics2D를 활용한 리볼버 회전 로직 및 투명 PNG 리소스 합성.

Thread Safety: 서버의 Room 클래스 메서드에 synchronized를 적용하여 동시성 문제 해결.

9. 📝 최종본 요약 설명

이 프로젝트는 단순한 운 게임을 넘어, 네트워크 동기화와 **전략적 요소(아이템, 타겟 변경)**를 결합한 완성도 높은 2인용 게임입니다. 서버가 게임의 모든 상태(탄창, 체력, 턴)를 권한을 가지고 관리(Authoritative Server)하며, 클라이언트는 이를 시각화하는 구조로 설계되어 보안성과 안정성을 갖추었습니다.
