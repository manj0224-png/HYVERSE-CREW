# HYVERSE Running Crew Tracker

러닝크루 HYVERSE의 멤버 기록·페이스 분석·정기런/대회 일정을 한 화면에서 관리하는 PWA.

브라우저로 접속해서 그대로 쓰거나, 모바일 홈 화면에 추가하면 앱처럼 사용할 수 있다.

## 기능

- **대시보드** — 크루 통계, 다가오는 일정 카드, 이달 목표 달성률, 최근 런 (검색·필터·실시간)
- **기록 추가** — 런 입력 + 1km 단위 구간 스플릿 선택 입력 + 심박존 가이드
- **페이스 분석** — 페이스/심박/날씨 차트 + 구간별 페이스 변화 분석 (후반 강한 런 / 페이스 유지 / 후반 무너짐 통계)
- **크루 랭킹** — 최고 페이스·총 거리·참여 횟수
- **목표 관리** — 크루원별 월간 목표 거리·페이스
- **정기런 / 대회 캘린더** — D-day, 참석 토글, 종목별 배지
- **크루원 관리** — 합류·삭제(연쇄 정리)
- **월별 리포트** — 개인별 PNG 저장 + 인쇄
- **크루 리포트** — 월별 크루 종합 (Top 3, 종류 분포, 그 달의 대회)

## 사용 방법

### 모바일에서 앱처럼 쓰기 (권장)

1. 모바일 브라우저로 사이트 접속
2. **iOS Safari**: 공유 → "홈 화면에 추가"
3. **Android Chrome**: 메뉴 → "앱 설치" 또는 "홈 화면에 추가"
4. 홈 화면 아이콘 누르면 풀스크린 PWA로 실행 (오프라인 동작)

### 데스크톱

브라우저로 그냥 접속해서 쓰면 된다.

## 데이터

- 모든 데이터는 **브라우저 localStorage** 에 저장 (서버·계정 없음)
- 디바이스마다 독립 (현재 동기화 없음)
- 5개 키: `hv-runs` `hv-goals` `hv-sched` `hv-races` `hv-members`

## 기술 스택

- HTML / CSS / Vanilla JS (단일 파일 `index.html`)
- Chart.js 4.4 — 차트
- html2canvas 1.4 — 리포트 이미지 저장
- Service Worker — 오프라인 캐싱 (PWA)
- Google Fonts (Bebas Neue / DM Sans / JetBrains Mono)

## 파일 구성

```
index.html           모든 UI·로직·CSS 한 파일에
manifest.json        PWA 매니페스트
service-worker.js    오프라인 캐싱
icon.svg             벡터 favicon
icon-192.png         Android 홈 아이콘 (192x192)
icon-512.png         Android 스플래시 (512x512, maskable)
apple-touch-icon.png iOS 홈 아이콘 (180x180)
README.md            이 파일
```

## 개발 메모

- 모바일 우선 반응형 (브레이크포인트 480px / 768px)
- 다크모드 자동 (`prefers-color-scheme: dark`)
- 터치 타깃 44×44 보장 (Apple HIG)
- iOS 줌 방지를 위해 폼 input 폰트 16px 고정
