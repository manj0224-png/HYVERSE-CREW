# HYVERSE Running Crew Tracker — Claude Code 인계 문서 v2.0

> 이 문서는 Claude Code 터미널 세션에서 그대로 붙여넣어 사용하기 위한 인계 문서입니다.
> 전체 소스코드(`hyverse.html`)와 함께 첨부하면 바로 작업을 이어받을 수 있습니다.

---

## 1. 프로젝트 개요

- **프로젝트명**: HYVERSE Running Crew Tracker
- **버전**: v2.0 (2026-04 기준)
- **목적**: 러닝크루 HYVERSE 멤버들의 런 기록 · 페이스 분석 · 대회/일정 관리 통합 트래커
- **현재 형태**: 단일 HTML 파일 (Claude Artifacts 위젯)
- **데이터 저장**: `localStorage` (5개 키)
- **외부 의존성 (CDN)**:
  - Chart.js 4.4.1 — 그래프
  - html2canvas 1.4.1 — 리포트 이미지 저장
  - Google Fonts: Bebas Neue, DM Sans, JetBrains Mono

---

## 2. 구현된 기능 (9개 탭)

| # | 탭 | 핵심 기능 |
|---|-----|----------|
| 01 | 대시보드 | 크루 통계 요약, 이달 목표 달성률, 최근 런 리스트 (수정/삭제 버튼 포함) |
| 02 | 기록 추가 | 런 기록 입력 폼 + 심박존 가이드 |
| 03 | 페이스 분석 | 페이스/심박/날씨별 차트 (Chart.js, 런 종류별 필터) |
| 04 | 크루 랭킹 | 최고 페이스 / 총 거리 / 참여 횟수 정렬 |
| 05 | 목표 관리 | 크루원별 월간 목표 거리·페이스 설정 + 달성률 |
| 06 | 정기런 스케줄러 | D-day 표시, 크루원 참석 토글 |
| 07 | 대회 캘린더 | D-day 카운트다운, 종목별 배지 |
| 08 | **크루원 관리** ⭐신규 | 멤버 추가/삭제, 연쇄 정리 (기록·목표·참석) |
| 09 | **월별 리포트** ⭐신규 | PNG 이미지 저장 + 인쇄 (html2canvas) |

⭐ = v2.0에서 추가된 기능

---

## 3. 데이터 스키마

모든 데이터는 `localStorage`에 JSON 배열로 저장됩니다.

### 3.1 크루원 (`hv-members`) ⭐신규
```js
{
  id: Date.now(),           // 고유 id
  name: "민준",             // 크루원 이름 (PK 역할, unique)
  addedAt: "2025-09-01"     // 가입일 (YYYY-MM-DD)
}
```

### 3.2 런 기록 (`hv-runs`)
```js
{
  id: Date.now(),
  name: "민준",             // FK → members.name
  date: "2026-04-25",       // YYYY-MM-DD
  type: "tempo",            // easy | tempo | race | long | interval
  dist: 10,                 // km (number)
  pace: 5.1,                // 분 단위 소수 (정렬·계산용)
  paceStr: "5:06",          // 표시용 문자열
  hr: 158,                  // 심박수 bpm (number | null)
  weather: "맑음",          // 맑음|흐림|비|더움|추움|바람 | null
  memo: "한강 코스"         // 자유 메모
}
```

### 3.3 목표 (`hv-goals`)
```js
{
  id: Date.now(),
  name: "민준",             // FK → members.name
  month: "2026-04",         // YYYY-MM
  dist: 100,                // 목표 거리 km (number | null)
  pace: "4:40"              // 목표 페이스 (string | null)
}
```
- 같은 `name + month` 조합은 **덮어쓰기**

### 3.4 정기런 일정 (`hv-sched`)
```js
{
  id: Date.now(),
  name: "한강 일요일런",
  date: "2026-04-27",
  time: "07:00",
  loc: "여의도 한강공원",
  dist: "15km",             // 자유 입력 문자열
  attendees: ["민준", "서연"]   // 참석 확정 크루원 배열
}
```

### 3.5 대회 (`hv-races`)
```js
{
  id: Date.now(),
  name: "서울국제마라톤 2026",
  date: "2026-05-03",
  cat: "풀마라톤",          // 풀마라톤 | 하프 | 10K | 5K | 기타
  loc: "광화문",
  goal: "3:30:00"           // 목표 기록 문자열
}
```

### 3.6 데이터 무결성 규칙
- **크루원 삭제 시**: 해당 이름의 `runs`, `goals` 전부 삭제 + `schedules.attendees` 배열에서 제거 (연쇄 정리)
- **크루원 이름 unique**: 동명이인 등록 불가 (이니셜·숫자 등으로 구분 권장)
- **마이그레이션 로직**: `hv-members`가 비어있고 `hv-runs`에 데이터가 있으면, 런 기록에서 이름을 자동 추출해 멤버 테이블 생성

---

## 4. 디자인 시스템

### 4.1 폰트
| 용도 | 폰트 |
|------|------|
| 디스플레이 (숫자, 타이틀) | **Bebas Neue** |
| 본문 | **DM Sans** (400/500/600/700) |
| 모노스페이스 (라벨, 코드) | **JetBrains Mono** (400/500) |

### 4.2 컬러 토큰 (CSS 변수)
```css
--brand:        #00C4A0   /* 틸 그린 (브랜드) */
--brand-dark:   #009f82
--brand-soft:   rgba(0,196,160,0.12)
--bg / --bg-elev / --bg-muted
--text / --text-muted / --text-soft
--border / --border-strong
--danger:       #e53935
--warn:         #f59e0b
--ok:           #22c55e
```

라이트/다크 테마는 `@media (prefers-color-scheme: dark)`로 자동 전환.

### 4.3 컴포넌트 스타일 (재사용 패턴)
- `.card` — 기본 카드 (border + shadow + 14px radius)
- `.btn` / `.btn-primary` / `.btn-ghost` / `.btn-danger` / `.btn-sm` / `.btn-icon`
- `.chip` (+ `.easy` `.tempo` `.race` `.long` `.interval`) — 런 종류별 색상
- `.stat-label` + `.stat-value` — 대시보드 카운터 패턴
- `.run-row` — 그리드 4-col (date | main | stats | actions)
- `.dday` (+ `.soon` `.far` `.done`) — D-day 뱃지
- `.modal-backdrop` + `.modal` — 확인/편집 모달
- `.toast` — 하단 중앙 알림
- `.report-view` — 라이트 테마 고정 인쇄용 리포트 (PDF/PNG 출력)

---

## 5. 코드 구조 (단일 HTML 내부)

```
hyverse.html (≈1360 lines)
├── <style>          : CSS 변수 + 컴포넌트 스타일 + 인쇄 미디어 쿼리
├── <body>
│   ├── header       : 브랜드 마크 + 라이브 인디케이터
│   ├── tabs nav     : 9개 탭 버튼
│   ├── 9× <section.panel data-panel="...">
│   ├── modal        : 공용 모달 컨테이너
│   └── toast        : 공용 토스트
└── <script>         : 아래 15개 섹션
    1.  STORAGE LAYER       — load/save/KEYS
    2.  SEED DATA           — 초기 샘플 데이터 + 마이그레이션
    3.  HELPERS             — fmtDate, paceToMin, minToPace, daysDiff, avatarColor 등
    4.  TAB NAV             — 탭 전환 + per-tab render 호출
    5.  MEMBER SELECT SYNC  — 드롭다운 자동 동기화
    6.  DASHBOARD           — renderDashboard()
    7.  RUN CRUD            — 추가/수정/삭제 + 모달
    8.  ANALYSIS CHARTS     — Chart.js 3종 (pace/hr/weather)
    9.  RANK                — renderRank() + 정렬 필터
    10. GOALS               — 추가/덮어쓰기/렌더
    11. SCHEDULE            — 추가/참석 토글
    12. RACES               — 추가/렌더
    13. MEMBERS             — 추가/삭제 (연쇄 정리)
    14. MONTHLY REPORT      — 생성 + PNG 저장 + 인쇄
    15. INIT                — seedIfEmpty + 첫 렌더
```

### 5.1 주요 헬퍼 함수
```js
$(s), $$(s)              // querySelector / All 단축
load(key), save(key, v)  // localStorage JSON 래퍼
paceToMin("4:30")        // → 4.5
minToPace(4.5)           // → "4:30"
daysDiff("2026-05-03")   // → 오늘 기준 일수
avatarColor(name)        // 이름 해시 → 10색 팔레트 중 1색
toast("메시지")          // 1.8초 토스트
confirmModal({title, body, danger}) // Promise<boolean>
```

---

## 6. 다음 개발 로드맵

### 단기 (현재 환경에서 바로)
- [ ] 신발 마일리지 트래커 — `runs.shoe` 필드 + 누적 km 대시보드 + 교체 알림 (보통 700-800km)
- [ ] 구간 스플릿 — `runs.splits: [{km:1,sec:280},...]` 추가, 페이스 변화 그래프
- [ ] CSV 내보내기/가져오기 — localStorage 한계 대응 + 백업
- [ ] 크루 단체 런 — 같은 날·같은 런 그룹핑 뷰
- [ ] 주간 챌린지 — 크루원끼리 거리/횟수 경쟁 보드

### 중기 (Claude Code 프로젝트화)
- [ ] **React + Vite 마이그레이션** (아래 §7 참조)
- [ ] SQLite 또는 Supabase 연동 → 데이터 영속화
- [ ] 멤버별 로그인 (Supabase Auth)
- [ ] 월별 리포트 PDF 직접 생성 (jsPDF + autoTable)

### 장기 (서비스화)
- [ ] Strava / Garmin Connect API 연동 → 자동 import
- [ ] 카카오톡 알림 연동 (정기런 D-1 리마인더)
- [ ] 모바일 PWA 또는 React Native 앱

---

## 7. React + Vite 마이그레이션 권장 구조

Claude Code에서 새 프로젝트로 옮길 때 추천하는 디렉토리 구조입니다.

```
hyverse-tracker/
├── package.json
├── vite.config.ts
├── tsconfig.json
├── index.html
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── styles/
    │   ├── tokens.css        # CSS 변수 (현재 디자인 토큰 그대로 이식)
    │   └── globals.css
    ├── lib/
    │   ├── storage.ts        # localStorage 래퍼 (현재 §5.1 그대로)
    │   ├── helpers.ts        # paceToMin, minToPace, daysDiff 등
    │   └── seed.ts           # 초기 시드 데이터
    ├── types/
    │   └── index.ts          # Member, Run, Goal, Schedule, Race 타입
    ├── store/
    │   └── useTracker.ts     # zustand 또는 Context로 상태 관리
    ├── components/
    │   ├── ui/               # Card, Button, Chip, Modal, Toast 등 공용
    │   ├── layout/           # Header, Tabs
    │   ├── RunRow.tsx
    │   ├── ProgressRow.tsx
    │   ├── DDayBadge.tsx
    │   └── Avatar.tsx
    ├── tabs/
    │   ├── Dashboard.tsx
    │   ├── AddRun.tsx
    │   ├── Analysis.tsx
    │   ├── Rank.tsx
    │   ├── Goals.tsx
    │   ├── Schedule.tsx
    │   ├── Races.tsx
    │   ├── Members.tsx
    │   └── Report.tsx
    └── charts/
        ├── PaceChart.tsx     # Chart.js → recharts 추천
        ├── HRChart.tsx
        └── WeatherChart.tsx
```

### 권장 라이브러리
| 영역 | 추천 |
|------|------|
| 상태관리 | zustand (가벼움) 또는 Context API |
| 차트 | **recharts** (React 친화) 또는 Chart.js 유지 |
| 폼 | react-hook-form + zod |
| UI 프리미티브 | shadcn/ui (radix 기반) |
| 날짜 | date-fns |
| PDF/이미지 | html2canvas + jsPDF |
| 데이터 영속화 | localStorage → Supabase (단계적 전환) |

---

## 8. Claude Code 작업 의뢰 템플릿

아래 템플릿을 채워서 Claude Code에 첫 프롬프트로 넣으세요.

```
이 프로젝트(HYVERSE Running Crew Tracker)를 이어받아 작업해줘.

[현재 상태]
- 단일 HTML 파일 (hyverse.html, ≈1360줄, 첨부)
- 9개 탭 모두 동작, localStorage 기반
- 위 인계 문서(hyverse-handoff-v2.md, 첨부) 참고

[작업 목표]
1. ___________________________________
2. ___________________________________
3. ___________________________________

[제약사항]
- 디자인 시스템(폰트/컬러/컴포넌트 패턴) 유지
- 데이터 스키마 호환성 유지 (기존 localStorage 데이터 import 가능해야 함)
- 다크모드 자동 전환 유지

먼저 현재 코드 구조를 파악하고 마이그레이션 계획을 세워줘.
구현 시작 전에 확인이 필요한 부분이 있으면 알려줘.
```

### 자주 쓰는 작업 패턴

**A. React + Vite로 옮기기**
```
hyverse.html을 React 18 + Vite + TypeScript 프로젝트로 마이그레이션해줘.
§7의 디렉토리 구조를 따라줘. 각 탭은 별도 컴포넌트로 분리하고,
공용 UI(Card, Button, Modal, Toast)는 components/ui로 추출해줘.
첫 단계로 storage layer + types + 1번 탭(대시보드)만 먼저 만들고,
동작 확인 후 나머지 탭을 순서대로 추가하자.
```

**B. Supabase 연동**
```
현재 localStorage 5개 키(hv-members, hv-runs, hv-goals, hv-sched, hv-races)를
Supabase 테이블로 옮기는 마이그레이션 스크립트를 작성해줘.
스키마는 §3을 따르고, RLS 정책은 일단 인증 사용자 전체 read/write로 시작.
기존 localStorage 데이터를 import하는 UI도 추가해줘.
```

**C. 신발 마일리지 트래커 추가**
```
현재 hyverse.html에 신발 마일리지 트래커를 추가해줘.
- 새 localStorage 키: hv-shoes
- 스키마: { id, name, brand, model, totalKm, retiredAt, addedAt }
- 기록 추가 폼에 신발 선택 드롭다운 추가 → runs.shoe 필드로 저장
- 새 탭 또는 크루원 관리 탭 하단에 신발 리스트 + 누적 km 진행바
- 700km 초과 시 교체 알림 (chip으로 표시)
디자인 시스템은 그대로 유지하고, 기존 데이터와 호환되도록 마이그레이션 처리.
```

---

## 9. 알려진 한계 / 주의사항

- **localStorage 5MB 제한**: 일반 사용엔 충분하지만 1000+ 런 기록 누적 시 검토 필요
- **단일 디바이스**: 동기화 없음 → 디바이스 간 데이터 공유 불가
- **인증 없음**: 누구나 모든 데이터 수정 가능 (현재는 신뢰 기반)
- **크루원 이름이 PK 역할**: 이름 변경 시 관련 데이터(runs, goals 등) 일괄 업데이트 필요
- **html2canvas의 한국어 폰트**: 일부 환경에서 폰트 미적용될 수 있음 → PDF 직접 생성 시 jsPDF + 한글 폰트 임베딩 필요
- **차트 다크모드**: Chart.js는 CSS 변수를 직접 못 읽음 → `getComputedStyle`로 추출해서 옵션에 주입 중

---

## 10. 변경 이력

### v2.0 (2026-04-25)
- ⭐ 크루원 관리 탭 추가 (`hv-members` 신규 키)
- ⭐ 월별 리포트 탭 추가 (PNG 저장 + 인쇄)
- ⭐ 런 기록 수정/삭제 기능 (대시보드 인라인)
- 공용 모달/토스트 헬퍼 추가
- JetBrains Mono 폰트 추가 (모노스페이스 라벨)
- 크루원 드롭다운 자동 동기화
- 기존 데이터 마이그레이션 로직 (members 자동 생성)

### v1.0 (초기)
- 7개 탭 기본 구조 (대시보드 / 기록 추가 / 페이스 분석 / 크루 랭킹 / 목표 / 정기런 / 대회)
- 샘플 데이터 (크루원 3명: 민준, 서연, 지호)
