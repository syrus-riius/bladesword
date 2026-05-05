# 일기토 (Ikkiuchi) — 게임 디자인 프로토타입

일대일 검투 로그라이트. 매 합 두 검객이 거리와 기예를 동시에 결정해 부딪힌다.

## 구조

```
ikkiuchi-modular/
├── index.html        ← 진입점 (브라우저로 이걸 연다)
├── data/             ← 게임 데이터 (이 안만 만져도 균형 80% 조정)
│   ├── commands.json   기예 (베기, 찌르기 등)
│   ├── classes.json    클래스 (떠돌이 검객, 도부수, 행상)
│   ├── enemies.json    적 (산적, 표창쟁이, 도끼병, 자객, 검호)
│   ├── weapons.json    무기 (카타나, 환도, 단봉)
│   ├── map.json        스테이지 진행 + 상점 풀
│   ├── balance.json    핵심 상수 (시작 자원, 비용, 가격)
│   └── README.md       튜닝 가이드 (어떤 값을 바꾸면 뭐가 변하나)
└── README.md         ← 이 파일
```

## 첫 실행 — 로컬에서 띄우기

`index.html`을 더블클릭하면 **작동 안 한다**. 브라우저가 보안상 같은 폴더의 JSON을 못 읽기 때문.
간단한 로컬 웹서버가 필요하다.

### 옵션 1 — Python (어느 OS든 가장 쉬움)

터미널 / 명령 프롬프트에서 `ikkiuchi-modular` 폴더로 이동 후:

```bash
python3 -m http.server 8000
```

그리고 브라우저로 `http://localhost:8000/` 접속.

### 옵션 2 — Node.js

```bash
npx serve
```

### 옵션 3 — VS Code의 Live Server

1. VS Code에 "Live Server" 익스텐션 설치
2. `index.html` 열고 우클릭 → "Open with Live Server"

## 핸드폰에서 테스트하기 — GitHub Pages (추천)

가장 빠르고 영구적인 방법. 한 번 설정하면 계속 쓴다.

1. **GitHub 가입** (이메일만 있으면 됨)
2. **새 repository 만들기** — 이름은 자유 (예: `ikkiuchi`), Public으로
3. **파일 업로드**: 웹에서 "Add file" → "Upload files" → 이 폴더 전체(`index.html`, `data/` 폴더 등) 드래그
4. **Pages 활성화**: Settings → Pages → Source: "Deploy from branch" → main → Save
5. **1~2분 기다림** → URL이 생긴다: `https://[본인아이디].github.io/[repo이름]/`
6. **핸드폰 브라우저로 접속** → 홈 화면에 추가하면 앱처럼 사용

### 수정 워크플로우

1. 데스크톱에서 `data/balance.json` 같은 파일을 텍스트 에디터로 수정
2. 변경 사항 저장
3. GitHub 웹에서 그 파일 클릭 → 연필 아이콘 → 변경 내용 붙여넣기 → "Commit changes"
   - 또는 git 명령으로 push
4. **1~2분 후 핸드폰에서 새로고침** → 변경 반영

## 개발 워크플로우 — 데이터만 만지기

### "이 기예 데미지 좀 키우고 싶어"

`data/commands.json`에서 그 기예 찾기:
```json
"slash": { ..., "hp": [3, 5], ... }
```
`hp`의 [최소, 최대] 굴림 범위 수정. JSON은 큰따옴표만 쓴다.

### "이 적이 너무 어려워"

`data/enemies.json`에서 그 적의 `hp` 줄이기, 또는 `deck`에서 강한 기예 빼기.

### "기력 시스템 너무 빡빡 / 헐렁"

`data/balance.json`에서:
- `staminaPerTurn`: 매 합 자동 회복량
- `intent.costs`: 이동 비용
- 각 기예의 `cost`는 commands.json

자세한 가이드는 `data/README.md` 참조.

## 데이터 편집 — Admin Console (v1.1)

게임 안에 데이터 편집 화면이 있다. 클래스 선택 화면 우상단의 ⚙ 버튼.

### 기능

- **탭 별 편집**: 기예 / 클래스 / 적 / 무기 / 스테이지 / 균형
- **행별 펼침**: 객체를 카드로 표시, 탭하면 펼쳐서 모든 필드 편집
- **타입 자동 추론**: 숫자는 number input, 배열은 JSON input, 중첩 객체는 textarea
- **행 액션**: 복제 / 키 변경 / 삭제
- **새 행 추가**: 첫 항목 템플릿 복제

### 저장 — 두 단계 안전망

**1. 즉시 저장** — 변경사항은 항상 localStorage에 저장됨. 같은 기기·브라우저에서 새로고침해도 유지. GitHub 연결 안 돼도 동작.

**2. GitHub 자동 commit** — 우상단 🔑 버튼으로 GitHub 토큰 등록하면 "💾 저장" 누를 때마다 자동 commit. 1~2분 후 GitHub Pages 재배포 → 다른 기기에서도 반영.

### GitHub 토큰 만들기

1. github.com → 우상단 프로필 → Settings
2. 좌하단 Developer settings
3. Personal access tokens → **Fine-grained tokens** → Generate new token
4. 설정:
   - Token name: `ikkiuchi-admin` (자유)
   - Expiration: 1년 (또는 짧게)
   - Repository access: Only select repositories → 해당 repo 선택
   - Permissions → Repository permissions → **Contents: Read and write**
5. Generate token → 표시되는 `github_pat_...` 복사 (한 번만 보임)

게임의 ⚙ → 🔑 누르고:
- Repository: `yourname/ikkiuchi`
- Branch: `main`
- Token: 방금 복사한 값

저장 누르면 연결 검증 후 사용 가능.

### 워크플로우

핸드폰에서:
1. ⚙ 눌러 admin 진입
2. 균형 탭 → `staminaPerTurn` 2 → 3
3. 💾 저장 → "✓ commit 됨"
4. ← 게임으로 돌아가 즉시 시험 (localStorage)
5. 1~2분 후 PC에서 같은 URL 열어도 반영됨

### 다른 옵션

- **⤓ JSON 내보내기**: 현재 탭의 JSON 파일 다운로드. 직접 git commit 하고 싶을 때.
- **↺ 원본 리셋**: 현재 탭의 변경사항 버리고 GitHub 원본 다시 로드.

### 보안 메모

토큰은 localStorage에 평문 저장됨. 본인 기기에서만 쓰는 도구라면 OK. 공용 PC에서 사용 후 🔑 → "지우기"로 토큰 제거.

토큰 권한이 한 repo의 Contents뿐이라 유출돼도 그 repo 코드 수정만 가능 — 큰 위험 아님. 단 Public repo에서 commit 작성자가 본인 이름으로 표시되는 건 알아두기.

## 자주 쓰는 수정 패턴

| 하고 싶은 것 | 만질 파일 | 키 |
|---|---|---|
| 시작 골드 늘림 | `balance.json` | `startingGold` |
| 매 합 더 빨리 | `balance.json` | `staminaPerTurn` 키움 |
| 강타 너무 강함 | `commands.json` | `cleave.hp` 줄임 |
| 산적 너무 약함 | `enemies.json` | `bandit.hp`, `bandit.deck` |
| 회피 비싸게 | `commands.json` | `dodge.cost` |
| 광기 강화 도박성 ↓ | `balance.json` | `forge.options[2].failChance` |
| 표창쟁이 도주 더 자주 | `balance.json` | `ai.kiterRetreatChance` 키움 |
| 새 클래스 추가 | `classes.json` | 새 항목 + 그 deck의 기예가 commands.json에 있는지 확인 |
| 새 적 추가 | `enemies.json`, `map.json` | 적 정의 + map.stages에 등록 |

## JSON 편집 시 주의

이 게임이 안 켜지는 가장 흔한 이유는 JSON 문법 실수. 다음 규칙을 꼭 지킨다:

- **큰따옴표만** — `'a'` 안 됨, `"a"` OK
- **마지막 항목 뒤 쉼표 X** — `{ "a": 1, }` 안 됨
- **숫자 따옴표 X** — `"hp": 30` (○), `"hp": "30"` (X)
- **`true`/`false`/`null`** — 따옴표 없이

수정 후 의심스러우면 `https://jsonlint.com`에 붙여넣어 검증하면 안전.

## 향후 작업 — 코드 모듈화

현재 `index.html` 안에 모든 JS 코드가 들어있다. 이건 의도적 — 디버깅과 배포가 쉽고, JSON 분리만으로도 90%의 튜닝 작업이 가능하기 때문.

데이터 분리를 한 후 게임 디자인이 안정되면, 그 다음 단계는:
1. JS를 `src/*.js` ES 모듈로 분리 (combat.js, ai.js, ui.js 등)
2. 그 후 진짜 게임 엔진(Godot 추천)으로 옮김

지금은 종이 프로토 → HTML 프로토 → 데이터 분리 단계까지. 모듈화는 다음 마일스톤.

## 변경 이력

- **v1.2**: 우선순위(speed) 시스템 + 무기 내구도 + 검 놓침 추가
  - 모든 기예에 speed 0~4 등급. 다른 등급이면 빠른 쪽 적중·느린 쪽 무효
  - 밀치기/균형깨기/일격필살/호흡 = speed 4 (가장 느림) — 찌르기·표창 등 빠른 공격에 무효됨
  - 급습/발도술 = speed 0 (즉발) — 모든 공격을 무효화
  - 무기 내구도: 같은 등급 공격 충돌 시 닳음, 부서지면 데미지 50~90% 감소
  - 매 합 0.5% 확률로 검 놓침 — 밀치기로만 회복 (보스전 제외)
  - 대장간에 수리 옵션 추가 (응급 30g, 완전 60g)
- v1.1: Admin Console + GitHub API 자동 commit
- v1.0: JSON 데이터 분리 (`data/` 폴더)
- v0.13: 견제 모든 클래스 기본 덱
- v0.12: reachable 카드 클릭 가능 (이동+공격 동시 실행)
- v0.11: 무기 시스템, 대장간/수련장 분리, 매 라운드 무한 접근, 시작 골드 10000
- v0.10: 견제 → 기력 회복 기예
- v0.9: 5단계 의도 + 이동 비용, 가변 맵 크기
- v0.8: 조이스틱 + 기예 그리드 분할
- 이전: HTML 프로토타입 빌드 (v0.1~0.7)
- 종이 프로토 룰북 v0.8 별도 (`ikkiuchi-rulebook-v0.8.{md,docx}`)

## 참고

- 종이 프로토 룰북: `ikkiuchi-rulebook-v0.8.docx` / `ikkiuchi-rulebook-v0.8.md`
- 데이터 가이드: `data/README.md`
