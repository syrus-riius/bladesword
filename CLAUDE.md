# 일기토 (Ikkiuchi)

A roguelite of single combat. 일대일 검투 로그라이트 — 매 합 두 검객이 거리와 기예를 동시에 결정해 부딪힌다.

## 답변 언어

이 저장소에서의 대화·커밋 메시지·코드 주석은 **한국어**가 기본. 변수명·함수명·git branch 이름은 영문.

## 기술 스택 — 의도적으로 단순함

- **Vanilla HTML/CSS/JS** — 단일 `index.html` 셸 + ES 모듈 import 없이 한 `<script>` 안에서 전부 동작
- **빌드 툴 없음** — npm, webpack, vite, TypeScript 모두 미사용. 이 결정은 의도적이다 — 핸드폰에서 GitHub Pages로 5초만에 반영되는 사이클이 가장 중요하기 때문
- **데이터는 JSON 분리** — 게임 로직은 HTML 안, 튜닝 가능한 모든 값은 `data/*.json`
- **테스트 프레임워크 없음** — 매뉴얼 테스트 (브라우저로 직접 플레이). Node.js로 JS 문법 파싱 검증은 가능 (`node --check`)
- **폰트**: Noto Serif KR, Noto Sans KR, EB Garamond (Google Fonts CDN)

## 파일 구조

```
ikkiuchi-modular/
├── index.html          ← 진입점. 모든 게임 로직 + admin console + 스타일
├── data/               ← 균형 튜닝 가능 영역 (자주 만지는 곳)
│   ├── commands.json     기예 (베기, 찌르기, 회피 등)
│   ├── classes.json      플레이 가능 클래스 3종
│   ├── enemies.json      적 5종 (산적 → 검호)
│   ├── weapons.json      무기 3종 (카타나, 환도, 단봉)
│   ├── map.json          5스테이지 진행 + 상점/대장간 풀
│   ├── balance.json      핵심 상수 (시작 자원, 비용, 가격, AI 가중치)
│   └── README.md         튜닝 가이드 — 어떤 값 바꾸면 뭐가 변하는지
└── README.md           셋업 + 워크플로우 가이드
```

## 데이터 주도 설계 — 핵심 규칙

새 기능을 만들 때 **하드코딩된 숫자/문자열은 거의 없어야** 한다. 새 적, 새 기예, 새 이벤트는 거의 전부 해당 JSON 파일에 행 추가만으로 가능하도록 설계됐다.

- 적 추가 → `data/enemies.json`에 새 객체 추가, `data/map.json`의 풀에 키 추가
- 기예 추가 → `data/commands.json`에 추가, 효과는 `index.html`의 `executeCommand` switch에 처리 분기 추가
- 새 상수 → `data/balance.json`에 추가, 코드는 `BALANCE.foo`로 참조
- JSON 파일에는 `_comment` 키로 주석 작성 가능 — 클라이언트에서 `stripComments`가 자동 제거함

새 코드를 짜기 전에 항상 자문할 것: **"이 값을 사용자가 admin console에서 바꾸고 싶어할까?"** 답이 yes면 JSON으로.

## 로컬 개발

`index.html`을 더블클릭하면 **작동 안 한다** (브라우저가 file://에서 fetch 차단). 반드시 HTTP 서버로:

```bash
cd ikkiuchi-modular
python3 -m http.server 8000
# → http://localhost:8000/
```

검증 워크플로우:
1. 코드 변경 후 `node --check index.html`은 안 됨 (HTML이라). 대신 인라인 JS만 추출해 검증하거나 브라우저에서 콘솔 에러 확인
2. JSON 변경 후 `python3 -m json.tool data/balance.json`로 유효성 체크
3. 브라우저 새로고침 → 콘솔 에러 없는지 확인 → 게임 한 합 진행해서 화면 표시 확인

## 배포 — GitHub Pages

`main` 브랜치에 push하면 1~2분 후 자동 배포. `https://[user].github.io/ikkiuchi/`.

핸드폰에서 직접 데이터 튜닝하는 경로 두 가지:
1. **Admin console** (런타임 편집 + 자동 commit) — 페이지 안의 ⚙ 버튼
2. **GitHub 웹 에디터** (직접 JSON 수정) — repo에서 파일 클릭 → 연필

## Admin Console — 알고 있어야 할 것

`index.html` 안에 데이터 편집 화면이 내장돼 있다 (클래스 선택 화면 우상단 ⚙). GitHub Personal Access Token을 등록하면 모바일에서 값 변경 → 즉시 commit.

PAT 권한 요구사항:
- **Fine-grained token**, repo만 선택, **Contents: Read and write** + Metadata: Read

저장 흐름은 두 단계:
1. localStorage에 즉시 저장 (같은 기기 즉시 반영)
2. GitHub API PUT → commit → Pages 재배포 (다른 기기 1~2분 후 반영)

코드는 `index.html` 내부의 `ADMIN` 객체. 수정 시 주의 — 행 추가/복제/삭제 로직은 데이터 파일 구조에 의존한다.

## 게임 디자인 원칙 — 뺄 수 없는 것들

이 게임의 핵심 메커닉은 **거리와 기예의 동시결정**. 이걸 깨는 변경은 거부한다.

- **합(turn) = 양측이 동시에 의도+기예 선택 → 동시 공개 → 거리 충돌 해결 → 기예 효과**
- 5스테이지 짧은 런 (모바일 한 손 플레이 가정 — 한 런 5~10분)
- 균형 깨짐(`balance` 자원)이 KO 게이지 역할 — HP만으로 결판 안 남
- 기력(`stamina`)은 매 턴 회복되는 행동 점수, HP는 누적 자원

## 코드 컨벤션

- ES2020+ vanilla JS, 화살표 함수 OK, async/await OK
- 이벤트 핸들러는 `id` 기반 `$('id').onclick = ...` 패턴 (이미 정착됨)
- DOM 빌드는 textContent + appendChild, innerHTML은 신뢰 가능한 정적 문자열에만
- 상태는 module-level `state`/`game` 객체에 모임 (단일 진실의 원천), React 같은 프레임워크 안 씀
- CSS는 변수 적극 사용 (`--enemy`, `--gold`, `--surface` 등 색상 토큰 정의됨)
- 모바일 우선 — viewport 380px 기준으로 레이아웃 검증

## 절대 하지 말 것

- **빌드 시스템 도입 안 함** — npm, bundler, TS 추가 제안 거부. 단일 HTML + JSON의 단순함이 핵심 가치
- **`index.html`을 여러 .js 파일로 쪼개지 않음** — 이전에 시도했다 롤백한 결정. 데이터 분리만 함, 코드는 한 파일
- **종속성 추가 금지** — Google Fonts 외 외부 CDN, npm 패키지, jQuery 등 모두 X
- **BALANCE 값을 코드에 하드코딩 금지** — 항상 `data/balance.json` 경유
- **localStorage 키 이름 변경 금지** (`ikkiuchi-data-*`, `ikkiuchi-github-*`) — 기존 사용자 데이터 손실
- **종이 프로토 검증 없는 메커닉 추가 금지** — 새 시스템은 종이로 5번 굴려본 후에만 코드화

## 자주 하는 작업

| 작업 | 어디 만지나 |
|---|---|
| 적 데미지 조정 | `data/enemies.json` → `deck` 배열의 기예 |
| 시작 자원 | `data/balance.json` → `startingHp`, `startingGold` 등 |
| 새 기예 추가 | `data/commands.json` + `index.html`의 `executeCommand` |
| 상점 가격 | `data/balance.json` → `shop` 섹션 |
| AI 행동 | `data/balance.json` → `ai` 섹션 가중치 |
| 새 화면 추가 | `index.html`의 `<section class="screen">` 추가 + flow.js의 라우터 분기 |

## 커밋 메시지 컨벤션

한국어 OK. 짧게. 형식:
- `data: 베기 데미지 3-5 → 4-6`
- `fix: admin 저장 시 sha 충돌 해결`
- `feat: 자객 적 추가`

## 새 세션에서 작업 시작할 때

1. 먼저 `git status`로 작업 중인 변경사항 있는지 확인
2. `data/README.md`와 이 파일을 컨텍스트로 갖고 시작
3. 변경 후엔 항상 로컬 서버 띄우고 (`python3 -m http.server 8000`) 브라우저로 한 합 진행해보고 commit
