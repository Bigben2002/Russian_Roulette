# 📘 Russian-Roulette-Project
한성대학교 3학년 2학기 네트워크프로그래밍 프로젝트

팀원  
- **이경석 – 2171073** (Server Architecture & Core Logic)  
- **김준현 – 2171062** (Client GUI & Item Features)

---

# 📍 1. 프로젝트 개요

본 프로젝트는 **Java(Swing GUI)** 와 **TCP 소켓 통신(ServerSocket/Socket)** 기반으로 제작한  
**2인 네트워크 러시안 룰렛 게임**입니다.

LAN/Wi-Fi 환경에서 두 명의 플레이어가 서버에 접속하여  
턴 기반 러시안 룰렛 전투를 실시간으로 진행하도록 구현되었습니다.

### 🔧 개발환경
- Java 17 ~ Java 21  
- Eclipse IDE / IntelliJ IDEA  
- Windows / macOS 지원

### 🧪 사용 기술
- Swing GUI 기반 HUD 인터페이스
- TCP Socket + UTF-8 스트림
- 멀티스레드 서버 구조(ClientHandler)
- 텍스트 기반 커스텀 통신 프로토콜
- 이미지 렌더링 기반 HP/Life/Item 표시

### 🌐 네트워크 구조
- 서버 1대 + 클라이언트 2대  
- 동일 LAN 환경에서 IP·포트 입력 후 접속  
- **Server Authoritative** 구조로 모든 판정을 서버가 담당

---

# 📍 2. 목표와 현황  
🎯 Final Phase 목표: **전투 시스템 + 아이템 전략 + GUI 연동 완성**

### ✔ 구현 완료 내역
- 2인 매칭 및 닉네임 교환
- READY 동기화
- 코인토스 기반 **초기 턴 배정**
- 6칸 탄창(BULLET/BLANK 랜덤)
- 조준(AIM) SELF / ENEMY
- 발사(FIRE) → HP 감소 + 턴 교대
- SELF + BLANK → 턴 유지
- HP UI / 라이프 이미지 표시
- 총 이미지 회전 애니메이션(상·하·좌·우)
- **아이템 3종 (Heal, Search, Bomb) 구현**
- 채팅 기능
- EXIT_ROOM 처리

### ✖ 제외된 기능
- 사운드 효과  
- 카드 시스템 → 아이템 시스템으로 대체

---

# 📍 3. 게임 규칙

## 3-1. 기본 사격 규칙

| 조준 대상 | 탄 종류 | 결과 | 턴 변화 |
|----------|---------|-------|---------|
| SELF     | BLANK   | 생존(Risk Success) | 유지 |
| SELF     | BULLET  | HP -1 | 교대 |
| ENEMY    | BLANK   | 아무 일 없음 | 교대 |
| ENEMY    | BULLET  | 상대 HP -1 | 교대 |

- HP 초기값: **5**
- HP가 0 이하 → 즉시 패배
- 탄창 6칸 모두 소모 → **서버가 자동으로 재장전**

---

## 3-2. 아이템 시스템

| 코드 | 아이템명 | 효과 |
|------|----------|--------|
| H | Heal | HP +1 회복 (최대 5) |
| S | Search | 다음 탄환이 BULLET/BLANK인지 확인 |
| B | Bomb | 다음 실탄 명중 시 피해 2배 |

### 아이템 특징
- HUD 슬롯 최대 6개  
- 사용 키: **1~6번**  
- Search 결과는 **본인에게만 비공개 메시지로 전달**  
- Bomb 효과는 **다음 발사(FIRE) 1회에만 적용됨**

---
# 📍 4. 시스템 구조

## 디렉터리 구조 (트리 깨짐 방지 버전)

### src/server
- `ServerGuiMain.java` — 서버 GUI 실행  
- `ServerFrame.java` — 서버 로그 및 제어 UI  
- `ServerCore.java` — ServerSocket 생성 및 Accept  
- `Room.java` — ★ 게임 핵심 로직 (턴·HP·탄창·아이템·승패 판정)  
- `ClientHandler.java` — 클라이언트별 메시지 수신 스레드  
- `Protocol.java` — 통신 명령어 정의

### src/client
- `ClientMain.java` — 클라이언트 실행 진입점  
- `StartFrame.java` — Host/Port/Name 입력 창  
- `RoomFrame.java` — 대기실 READY 화면  
- `GameRoomFrame.java` — ★ 인게임 HUD 표시 + 키 입력  
- `NetworkClient.java` — 서버 수신 스레드  
- `ImageLoader.java` — 이미지 로딩 유틸리티

### resources/images
- 배경 이미지  
- 플레이어 이미지  
- 총 이미지  
- HP 아이콘  
- 아이템 아이콘  

---

# 📍 5. 시스템 상세 로직 & 데이터 흐름

## 5-1. 접속 핸드셰이크  
1. 클라이언트 → 서버 접속  
2. 서버 → HELLO  
3. 클라이언트 → 닉네임 전송  
4. 서버가 2명 접속 확인  
5. 서버에서 Room 생성  
6. RoomFrame으로 진입 후 READY 대기

---

## 5-2. READY → GAME_START

### Client → Server
READY


### Server → All


READY_STATE


READY가 두 명 모두 완료되면 서버는 아래 순서대로 전송:



GAME_START
RELOAD
AIM_UPDATE
TURN
ITEM_UPDATE


---

## 5-3. AIM 흐름

### Client → Server


AIM SELF
AIM ENEMY


### Server → All


AIM_UPDATE WHO=P1 TARGET=SELF


HUD에서 총 이미지 방향 업데이트.

---

## 5-4. FIRE 흐름

### Client → Server


FIRE


### 서버(Room)의 처리 순서
1. 턴 교대 조건 검사  
2. 현재 탄(BULLET / BLANK) 판정  
3. Bomb 여부에 따른 데미지 계산  
4. HP 업데이트  
5. 탄착 인덱스 증가  
6. 턴 교대  
7. 승패 판정  

### Server → All


FIRE_RESOLVE RESULT=BLANK TARGET=ENEMY HP1=5 HP2=4 INDEX=3


---

## 5-5. 아이템 사용 흐름

### Client → Server


USE_ITEM SLOT=3


### Server → All


ITEM_UPDATE WHO=P2 ITEMS=H---B-


Search는 본인에게만 전달:


PEEK_RESULT TYPE=BULLET


---

## 5-6. 퇴장(EXIT_ROOM)

### Client → Server


EXIT_ROOM


### Server → Other


EXIT_ROOM


클라이언트는 팝업 표시 후 종료.

---
# 📍 6. 실행 방법

## 서버 실행
1. `ServerGuiMain` 실행  
2. 포트 입력 (예: 7777)  
3. Start 클릭  
4. 로그에 아래 표시되면 준비 완료  

[Server] Listening on 7777

---

## 클라이언트 실행
1. `ClientMain` 두 번 실행  
2. Host / Port / Name 입력  
3. Connect 클릭  
4. 두 명 READY → **GAME_START**

---

# 📍 7. 조작 키

| 키 | 설명 |
|----|------|
| ↑ / ↓ | 조준 방향 변경 (SELF / ENEMY) |
| SPACE | 발사 |
| 1~6 | 아이템 사용 |
| Chat 버튼 | 채팅 열기 |
| 창 닫기(X) | EXIT_ROOM 후 종료 |

---

# 📍 8. 기술 요약

| 구성 | 설명 |
|------|-------|
| **통신 방식** | TCP 기반, UTF-8 텍스트 프로토콜 |
| **서버 구조** | ClientHandler 스레드 + Room에서 모든 로직 관리 |
| **클라이언트** | 입력 처리 / 이미지 렌더링 / HUD 담당 |
| **게임 로직** | 턴, HP, 탄창, AIM/FIRE, 아이템 모두 서버에서 판정 |
| **확장성** | 멀티룸, 로그, 관전자 모드 등 확장 가능 |

---

# 📍 9. 최종본 요약 설명

본 프로젝트는 **네트워크 통신 + 실시간 게임 로직 + GUI 렌더링 + 아이템 전략 시스템**을  
하나의 구조로 통합한 실전형 턴제 네트워크 게임입니다.

### 서버(Room)는:
- HP / 턴 / 탄창 / 아이템 등 **모든 상태를 단일하게 관리**
- 클라이언트는 그 결과만 반영
- 동기화 문제 없이 안정적

### 클라이언트는:
- 입력 처리  
- HUD 렌더링  
- 이미지 로딩  
- 애니메이션  

에 집중하도록 설계되었습니다.

**학습 목적 + 실전 구조**를 모두 갖춘 완성도 높은 네트워크 게임 프로젝트입니다.

---

