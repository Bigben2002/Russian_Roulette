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
