📘 Russian-Roulette-Project

한성대학교 3학년 2학기 네트워크프로그래밍 프로젝트

팀원

이경석 – 2171073 (Server Architecture & Core Logic)

김준현 – 2171062 (Client GUI & Item Features)

1. 프로젝트 개요

본 프로젝트는 Java(Swing GUI) + TCP 소켓 통신(ServerSocket/Socket) 을 이용하여
2인의 플레이어가 LAN 환경에서 실시간으로 러시안 룰렛 게임을 진행할 수 있도록 구현한 네트워크 게임입니다.

🔧 개발환경

Java 17 ~ Java 21

Eclipse IDE / IntelliJ IDEA

Windows / macOS 지원

🧰 사용 기술

Swing GUI (2D UI 구성)

TCP 소켓 통신 (UTF-8 기반 스트림)

멀티스레드 기반 서버 구조 (ClientHandler)

직접 설계한 텍스트 기반 프로토콜 (Protocol)

이미지 렌더링 기반 HUD (Display UI)

🌐 네트워크 구조

서버 1대 + 클라이언트 2대

동일 LAN/Wi-Fi 환경에서 IP와 포트 입력 후 접속

서버 기준으로 모든 게임 로직이 처리되는 서버 권위(Server Authoritative) 구조

2. 목표와 현황
🎯 Final Phase 목표

“전투 시스템 + 아이템 전략 + GUI 연동 완성”

✅ 구현 완료 내역

✔ 2인 매칭 및 READY 동기화

✔ 코인토스 기반 초깃턴 지정

✔ 6칸 탄창(실탄·공탄 랜덤)

✔ 조준(AIM) SELF / ENEMY

✔ 발사(FIRE), HP 감소, 턴 교대

✔ SELF + BLANK → 턴 유지

✔ HP UI, 탄창 인덱스, 라이프 이미지 구성

✔ 아이템 시스템 3종 구현 (Heal, Search, Bomb)

✔ 총 이미지 회전 애니메이션(좌/우/상/하)

✔ GameRoomFrame 에서 실시간 HUD 표시

✔ 채팅 기능

✔ 퇴장(EXIT_ROOM) 처리

❌ 제외된 기능

✖ 사운드 기능 (범위 조정으로 제외)

✖ 카드 시스템 → 아이템 시스템으로 완전히 대체

3. 게임 규칙
3-1. 기본 사격 규칙
조준 대상	탄 종류	결과	턴 변화
SELF	BLANK	생존 (Risk Success)	유지
SELF	BULLET	HP -1	교대
ENEMY	BLANK	아무 일 없음	교대
ENEMY	BULLET	상대 HP -1	교대

HP 초기값: 5

HP가 0 이하가 되면 즉시 패배

6칸 탄창을 모두 소모하면 서버가 자동 재장전

재장전 시 아이템도 일정 규칙에 따라 재지급됨

3-2. 아이템 시스템 (전략 요소)
코드	아이템명	효과 설명
H	Heal	HP +1 회복 (최대 5)
S	Search	다음 탄환이 BULLET/BLANK인지 확인
B	Bomb	다음 명중 시 데미지가 2배로 적용됨

아이템 특징

인게임 HUD에 아이콘 또는 문자 형태로 표시

사용 키는 숫자 1 ~ 6

Search의 결과는 해당 플레이어에게만 개인 메시지로 전달

Bomb은 다음 FIRE 명령에만 1회 적용

4. 시스템 구조
📁 전체 디렉터리 구조

src/server

ServerGuiMain.java — 서버 GUI 실행 진입점

ServerFrame.java — 서버 로그 출력 및 Start/Stop 제어 화면

ServerCore.java — ServerSocket 생성, 클라이언트 Accept, Room 생성

Room.java — ★ 게임 핵심 로직 (턴, HP, 탄창, 아이템, 승패 판정, 브로드캐스트)

ClientHandler.java — 클라이언트별 메시지 수신 스레드, 수신 데이터를 Room으로 전달

Protocol.java — 모든 통신 명령어 상수 정의

src/client

ClientMain.java — 클라이언트 실행 진입점

StartFrame.java — 서버 IP / Port / Name 입력 화면

RoomFrame.java — 대기실 화면 (READY 상태 표시 및 GAME_START 대기)

GameRoomFrame.java — 인게임 HUD, 키보드 입력(AIM/FIRE/ITEM) 처리

NetworkClient.java — 서버와의 소켓 연결 및 수신 스레드 관리

ImageLoader.java — /resources/images 폴더에서 이미지 로딩 유틸리티

resources/images

배경 이미지

캐릭터(플레이어) 이미지

총/리볼버 이미지

라이프(HP) 아이콘

아이템 아이콘 (Heal, Search, Bomb 등)

5. 시스템 상세 로직 및 데이터 흐름 (Data Flow)
5-1. 접속 및 핸드셰이크 과정

사용자가 ClientMain 실행 → StartFrame 표시

Host / Port / Name 입력 후 서버 접속

서버로부터 HELLO 메시지 수신

클라이언트는 자신의 닉네임 문자열 전송

ServerCore 가 두 명의 연결을 확인하면 Room 객체 생성

양쪽 클라이언트는 RoomFrame (대기실) 상태로 진입

5-2. READY → GAME_START 흐름

Client → Server : READY

Server → All : READY_STATE (각 플레이어의 준비 상태 갱신)

두 명 모두 READY가 되면, 서버는 다음과 같은 순서로 명령어를 전송:

GAME_START : 게임 시작 알림

RELOAD : 새 탄창 지급 (6칸 배열 생성)

AIM_UPDATE : 기본 조준 상태(SELF) 설정

TURN : 처음 턴을 가질 플레이어 지정

ITEM_UPDATE : 초기 아이템 지급 상태 전송

5-3. AIM 데이터 흐름

Client → Server : AIM SELF 또는 AIM ENEMY

Server → All : AIM_UPDATE WHO=P1 TARGET=SELF

클라이언트는 수신된 AIM_UPDATE 정보를 기반으로
GameRoomFrame 에서 총 이미지 회전 방향 및 HUD 표시를 갱신합니다.

5-4. FIRE 데이터 흐름

Client → Server : FIRE

서버(Room) 내부 처리 순서:

현재 턴이 맞는지 확인

현재 탄창 인덱스의 탄 종류 확인 (BULLET / BLANK)

Bomb 아이템 효과 여부 확인 → 데미지 계산

해당 대상의 HP 감소 처리

탄창 인덱스 증가 (다음 탄으로 이동)

승패 여부 검사 (HP ≤ 0)

턴 교대 여부 결정

Server → All : FIRE_RESOLVE RESULT=... TARGET=... HP1=x HP2=y ...

5-5. 아이템 사용 흐름

Client → Server : USE_ITEM SLOT=n

서버 처리:

슬롯에 위치한 아이템 확인 (H/S/B)

Heal: HP +1 적용 (최대 5)

Search: 현재/다음 탄 정보 확인 후 해당 플레이어에게만 결과 전송

Bomb: “다음 실탄 명중 시” 데미지 2배 플래그 설정

Server → All : ITEM_UPDATE WHO=P2 ITEMS=H---B-

Server → 특정 Client : PEEK_RESULT TYPE=BULLET (Search 결과 예시)

5-6. 퇴장 흐름

Client → Server : EXIT_ROOM

Server → 다른 플레이어 : EXIT_ROOM

상대방 클라이언트에서는

HUD 또는 팝업으로 “상대방이 퇴장했습니다.” 표시

게임 종료 및 화면 정리

6. 실행방법
🖥 서버 실행 방법

ServerGuiMain 실행

포트 번호 입력 (예: 7777)

Start 버튼 클릭

서버 로그에서 다음 메시지 확인

[Server] Listening on 7777

💻 클라이언트 실행 방법

ClientMain 을 2번 실행 (플레이어 2명)

각 클라이언트에서 Host / Port / Name 입력

Connect 버튼 클릭 → RoomFrame 으로 이동

두 명 모두 READY 버튼 클릭

서버가 GAME_START 를 보내면 GameRoomFrame 으로 전환

7. 조작 키
키	설명
↑ / ↓	조준 방향 변경 (SELF / ENEMY)
SPACE	발사 (FIRE)
1 ~ 6	해당 슬롯의 아이템 사용
Chat 버튼	채팅창 열기 / 채팅 전송
창 닫기(X)	EXIT_ROOM 전송 후 게임 종료
8. 기술 요약

통신 방식 : TCP 기반, UTF-8 텍스트 프로토콜

멀티스레드 구조 : 서버는 클라이언트 1명당 ClientHandler 스레드 운용

프로토콜 명령어 :

접속/상태 : HELLO, READY, READY_STATE, GAME_START

전투 : AIM, AIM_UPDATE, FIRE, FIRE_RESOLVE, RELOAD

아이템 : USE_ITEM, ITEM_UPDATE, PEEK_RESULT

기타 : EXIT_ROOM, CHAT 등

렌더링 : Swing 기반 2D HUD (GameRoomFrame)

데이터 관리 :

Room 이 HP / 턴 / 탄창 / 아이템 상태를 단일 진실(Single Source of Truth) 로 관리

클라이언트는 서버에서 온 명령에 따라 화면만 갱신

9. 최종본 요약 설명

이 프로젝트는 실시간 네트워크 통신, GUI 게임 로직, 전략 요소(아이템) 를 모두 포함한
학습 및 포트폴리오용 네트워크 게임입니다.

서버는 “심판 + 게임 엔진” 역할을 수행하며 모든 상태를 판단

클라이언트는 입력과 화면 표현에 집중

단순한 “운빨 게임”이 아니라,

Heal / Search / Bomb 를 활용한 전략 선택

탄창 정보와 턴 흐름을 고려한 판단

실시간 애니메이션과 HUD를 통한 시각적 피드백
이 결합된 구조입니다.

또한 현재 구조는 다음과 같이 확장 가능성을 염두에 두고 설계되었습니다.

멀티룸(여러 게임방) 관리

관전자 모드

전투 로그/리플레이 기록

스킨/테마 교체 시스템
