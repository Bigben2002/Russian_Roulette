📘 Russian-Roulette-Project

한성대학교 3학년 2학기 네트워크프로그래밍 프로젝트

팀원

이경석 – 2171073 (Server Architecture & Core Logic)

김준현 – 2171062 (Client GUI & Item Features)

1. 프로젝트 개요

본 프로젝트는 Java(Swing GUI) + TCP 소켓 통신(ServerSocket/Socket) 을 이용하여
2인의 플레이어가 LAN 환경에서 실시간으로 러시안 룰렛 게임을 진행할 수 있도록 구현한 네트워크 게임입니다.

개발환경

Java 17 ~ Java 21

Eclipse IDE / IntelliJ IDEA

Windows / macOS 지원

사용 기술

Swing GUI (2D UI 구성)

TCP 소켓 통신 (UTF-8 기반 스트림)

멀티스레드 기반 서버 구조 (ClientHandler)

직접 설계한 텍스트 기반 프로토콜

이미지 렌더링 기반 HUD(Display UI)

네트워크 구조

서버 1대 + 클라이언트 2대

동일 LAN/Wi-Fi 환경에서 IP와 포트 입력 후 접속

서버 기준으로 모든 게임 로직이 처리되는 서버 권위(Server Authoritative) 구조

2. 목표와 현황
🎯 Final Phase 목표: “전투 시스템 + 아이템 전략 + GUI 연동 완성”
구현 완료 내역

✔ 2인 매칭 및 READY 동기화

✔ 코인토스 기반 초깃턴 지정

✔ 6칸 탄창(실탄·공탄 랜덤)

✔ 조준(AIM) SELF / ENEMY

✔ 발사(FIRE), HP 감소, 턴 교대

✔ SELF + BLANK → 턴 유지

✔ HP UI, 탄창 인덱스, 라이프 이미지 구성

✔ 아이템 시스템 3종 구현 (Heal, Search, Bomb)

✔ 총 이미지 회전 애니메이션(좌/우/상/하)

✔ GameRoomFrame에서 실시간 HUD 표시

✔ 채팅 기능

✔ 퇴장(EXIT_ROOM) 처리

✖ (제외) 사운드 기능

✖ (제외) 카드 시스템 → 아이템 시스템으로 완전 대체

3. 게임 규칙
3-1. 기본 사격 규칙
조준 대상	탄 종류	결과	턴 변화
SELF	BLANK	생존(Risk Success)	유지
SELF	BULLET	HP -1	교대
ENEMY	BLANK	아무 일 없음	교대
ENEMY	BULLET	상대 HP -1	교대

HP 초기값: 5

HP가 0 이하가 되면 즉시 패배

6칸 탄창을 모두 소모하면 서버가 자동 재장전

재장전 시 아이템도 일부 재지급됨

3-2. 아이템 시스템 (전략 요소)
코드	아이템명	효과 설명
H	Heal	HP +1 회복 (최대 5)
S	Search	다음 탄환이 BULLET/BLANK인지 확인
B	Bomb	다음 명중 시 데미지가 2배

아이템 특징

인게임 HUD에 아이콘으로 표시

사용 키는 숫자 1~6

Search의 결과는 해당 플레이어에게만 개인 메시지로 전달됨

Bomb은 다음 FIRE 명령에만 적용됨

4. 시스템 구조
전체 디렉터리 구조 (트리 깨짐 방지 버전)

src/server

ServerGuiMain.java — 서버 GUI 실행

ServerFrame.java — 서버 로그 및 제어 화면

ServerCore.java — ServerSocket 생성, Accept 담당

Room.java — ★ 게임 핵심 로직 (턴·HP·탄창·아이템·판정·브로드캐스트)

ClientHandler.java — 클라이언트별 메시지 수신 스레드

Protocol.java — 모든 통신 명령어 목록 상수화

src/client

ClientMain.java — 클라이언트 실행 진입점

StartFrame.java — IP/Port/Name 입력 화면

RoomFrame.java — 대기실 화면 (READY 대기)

GameRoomFrame.java — ★ 인게임 HUD + 키 입력 처리

NetworkClient.java — 서버 통신 스레드

ImageLoader.java — 이미지 로딩 유틸리티

resources/images

배경 이미지

캐릭터 이미지

총 이미지

라이프(HP) 아이콘

아이템 아이콘

5. 시스템 상세 로직 및 데이터 흐름 (Data Flow)
5-1. 접속 및 핸드셰이크 과정

ClientMain 실행 → StartFrame에서 Host/Port/Name 입력

서버 접속 후 즉시 HELLO 수신

닉네임 전송

ServerCore에서 두 명의 연결이 확인되면 Room 생성

게임 대기실(RoomFrame)로 진입

5-2. READY → GAME_START 흐름

Client → Server
READY 전송

Server → All
READY_STATE 브로드캐스트

두 명 모두 READY되면 서버는 아래 명령어를 순서대로 전송

GAME_START

RELOAD (새 탄창 지급)

AIM_UPDATE (기본 SELF 조준)

TURN (초기 턴 배정)

ITEM_UPDATE (아이템 지급)

5-3. AIM 데이터 흐름

Client → Server
AIM SELF 또는 AIM ENEMY

Server → All
AIM_UPDATE WHO=P1 TARGET=SELF

클라이언트는 해당 상태에 따라 총 이미지 회전

5-4. FIRE 데이터 흐름

Client → Server
FIRE

서버(Room)가 처리하는 순서

턴 검사

현재 탄 인덱스 확인 (BULLET / BLANK)

데미지 계산 (Bomb 여부 포함)

HP 업데이트

탄창 인덱스 증가

턴 교대

승패 판정

Server → All
FIRE_RESOLVE RESULT=... HP1=x HP2=y

5-5. 아이템 사용 흐름

Client → Server
USE_ITEM SLOT=n

Server 처리

아이템 효과 적용

Bomb/Lens/Heal 효과 수행

HUD 업데이트

Server → All
ITEM_UPDATE WHO=P2 ITEMS=H---B-

Search 결과는 서버가 개인에게만 전송:
PEEK_RESULT TYPE=BULLET

5-6. 퇴장 흐름

Client → Server
EXIT_ROOM

Server → 다른 플레이어
EXIT_ROOM

GameRoomFrame에서 팝업 또는 HUD 표시 후 종료

6. 실행방법
서버 실행 방법

ServerGuiMain을 실행

포트 입력 (예: 7777)

Start 버튼 클릭

로그에 “[Server] Listening on 7777” 출력

클라이언트 실행 방법

ClientMain을 두 번 실행

Host / Port / Name 입력

Connect 버튼 클릭

RoomFrame 대기

두 명 모두 READY → 게임 자동 시작

7. 조작 키
키	설명
↑ / ↓	조준 방향 변경 (SELF / ENEMY)
SPACE	발사 (FIRE)
1~6	아이템 사용
Chat 버튼	채팅
창 닫기(X)	EXIT_ROOM 전송 후 종료
8. 기술 요약

통신 방식: TCP 기반, UTF-8 스트림, 텍스트 프로토콜

멀티스레드: 서버는 ClientHandler 2개를 각각 스레드로 운영

프로토콜 구조: AIM, FIRE, USE_ITEM, READY, EXIT_ROOM, ITEM_UPDATE 등

렌더링: Swing 기반 2D HUD

데이터 관리: 서버(Room)가 HP, 턴, 탄창, 아이템을 단일 상태로 유지

9. 최종본 요약 설명

본 프로젝트는 실시간 네트워크 통신 + GUI 게임 로직 + 전략 시스템을
모두 통합한 완성도 높은 학습 프로젝트입니다.

서버는 모든 상태를 결정하는 핵심 엔진

클라이언트는 입력 및 HUD 표현에 집중

단순한 룰렛 게임을 넘어 전략 아이템, 회전 애니메이션, 실시간 HUD가 결합된 구조

확장성: 멀티룸, 관전자 모드, 전투 로그, 스킨 시스템까지 확장 가능
