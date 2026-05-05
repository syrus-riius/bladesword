# 일기토 (Ikkiuchi) - 개발 가이드

개발자를 위한 게임 수정 및 확장 가이드입니다.

## 목차

1. [프로젝트 구조](#프로젝트-구조)
2. [데이터 파일 가이드](#데이터-파일-가이드)
3. [코드 구조](#코드-구조)
4. [새 기능 추가하기](#새-기능-추가하기)
5. [Admin Console 사용법](#admin-console-사용법)
6. [배포 워크플로우](#배포-워크플로우)
7. [주의사항](#주의사항)

---

## 프로젝트 구조

```
bladesword/
├── index.html          # 게임 코드 전체 (HTML + CSS + JS)
├── styles.css          # UI 스타일 (D&D 판타지 테마)
├── data/              # 게임 데이터 (JSON)
│   ├── commands.json    # 기예 (베기, 찌르기 등)
│   ├── classes.json     # 플레이 가능 클래스
│   ├── enemies.json     # 적 5종
│   ├── weapons.json     # 무기 3종
│   ├── map.json         # 5스테이지 + 상점/대장간 풀
│   ├── balance.json     # 균형 조정 핵심 상수
│   └── README.md        # 데이터 튜닝 가이드
├── CLAUDE.md          # 프로젝트 개요 (AI 컨텍스트용)
├── GAMEPLAY.md        # 게임 시스템 설명
├── DEVELOPMENT.md     # 이 문서
└── README.md          # 프로젝트 소개
```

### 핵심 원칙

1. **빌드 시스템 없음**: npm, webpack, TypeScript 미사용
2. **데이터 주도 설계**: 하드코딩 최소화, 모든 값은 JSON에
3. **모바일 우선**: 핸드폰 한 손 플레이 가정
4. **GitHub Pages 직접 배포**: main 브랜치 push → 1~2분 후 자동 배포

---

## 데이터 파일 가이드

모든 데이터 파일은 `data/` 폴더에 있으며, `data/README.md`에 상세한 튜닝 가이드가 있습니다.

### commands.json - 기예 추가/수정

```json
{
  "slash": {
    "id": "slash",
    "name": "베기",
    "nameEn": "Slash",
    "icon": "i-attack",
    "cost": 2,
    "hp": [3, 5],
    "bal": [2, 3],
    "ranges": [0, 1],
    "speed": 1,
    "desc": "기본 베기. 0~1칸에서 사용 가능."
  }
}
```

**필드 설명**:
- `id`: 고유 ID (영문, kebab-case)
- `name` / `nameEn`: 한글/영문 이름
- `icon`: 아이콘 ID (아이콘 도감 페이지에서 확인)
- `cost`: 기력 소모 (음수 = 회복)
- `hp`: [최소, 최대] 데미지 범위
- `bal`: [최소, 최대] 균형 데미지
- `ranges`: 작동 거리 배열 (0~4) 또는 `"all"`
- `speed`: 우선순위 (0=즉발, 1=빠름, 2=느림, 3=매우느림, 4=휴식)
- `desc`: 설명 (툴팁 표시)

**새 기예 추가 절차**:
1. `commands.json`에 새 객체 추가
2. `index.html`의 `executeCommand()` 함수에 특수 효과 로직 추가 (필요 시)
3. 아이콘 도감에서 `icon` ID 확인 또는 새 아이콘 추가
4. `classes.json`에서 해당 클래스 `deck` 배열에 ID 추가

### classes.json - 클래스 추가/수정

```json
{
  "ronin": {
    "id": "ronin",
    "name": "떠돌이 검객",
    "nameEn": "Wandering Sword",
    "emblem": "e-ronin",
    "culture": "일본 에도 17C · 카타나",
    "blurb": "주군 잃은 낭인. 길 위에서 칼 한 자루로 산다.",
    "mechanic": "잔심",
    "mechanicDesc": "같은 기예를 3턴 안 쓰면 다음 사용 시 +50%",
    "hp": 30,
    "bal": 30,
    "sta": 5,
    "evade": 0.10,
    "weapon": "katana",
    "deck": ["slash", "thrust", "parry", "dodge", "sliplash", "feint"]
  }
}
```

**필드 설명**:
- `hp` / `bal` / `sta`: 시작 자원
- `evade`: 회피 확률 (0.10 = 10%)
- `weapon`: `weapons.json`의 키
- `deck`: `commands.json`의 ID 배열
- `mechanic`: 고유 능력 로직은 `index.html`의 전투 코드에 하드코딩 필요

**새 클래스 추가 절차**:
1. `classes.json`에 새 객체 추가
2. 엠블렘 SVG를 `index.html`의 `<svg><defs>` 섹션에 추가 (id="e-클래스명")
3. 고유 능력이 있다면 `index.html`의 `executeCommand()` 또는 `resolveTurn()`에 로직 추가
4. 브라우저에서 테스트

### enemies.json - 적 추가/수정

```json
{
  "bandit": {
    "id": "bandit",
    "name": "산적",
    "nameEn": "Bandit",
    "stage": 1,
    "hp": 22,
    "bal": 22,
    "sta": 5,
    "evade": 0.05,
    "gold": 25,
    "deck": ["slash", "push", "rest"],
    "aiType": "brute"
  }
}
```

**AI 타입**:
- `brute`: 돌격형 (50% 전진, 공격 위주)
- `kiter`: 견제형 (40% 후퇴, 거리 유지)
- `balanced`: 균형형 (랜덤)

**새 적 추가 절차**:
1. `enemies.json`에 새 객체 추가
2. `map.json`의 해당 스테이지 `pool` 배열에 ID 추가
3. 새로운 AI 타입이 필요하면 `index.html`의 `aiDecideIntent()`와 `aiDecideCommand()` 함수 수정

### weapons.json - 무기 추가/수정

```json
{
  "katana": {
    "id": "katana",
    "name": "카타나",
    "nameEn": "Katana",
    "damageBonus": 0,
    "balanceBonus": 0,
    "durability": 100,
    "maxDurability": 100
  }
}
```

**새 무기 추가 절차**:
1. `weapons.json`에 새 객체 추가
2. `classes.json`에서 `weapon` 필드에 ID 지정
3. 무기별 특수 효과가 필요하면 전투 로직에 추가

### balance.json - 게임 밸런스 조정

이 파일은 게임의 핵심 수치를 담고 있습니다. 자세한 내용은 `data/README.md` 참조.

**주요 섹션**:
- `startingGold`: 시작 골드 (기본 100)
- `combat`: 전투 데미지 배율, 균형 회복률
- `weapon`: 무기 내구도 시스템 상수
- `disarm`: 검 놓침 확률
- `shop` / `forge` / `dojo`: 가격 및 강화 옵션
- `ai`: AI 행동 가중치

---

## 코드 구조

### index.html 내부 JS 섹션

코드는 주석으로 13개 섹션으로 나뉩니다:

1. **DATA LOADER**: JSON 로드 및 파싱
2. **GAME STATE & UTILITIES**: 전역 상태, 헬퍼 함수
3. **CLASS SELECT SCREEN**: 클래스 선택 화면
4. **MAP SCREEN**: 스테이지 맵 화면
5. **COMBAT SCREEN & RENDERING**: 전투 UI 렌더링
6. **AI DECISION SYSTEM**: 적 AI 로직
7. **TURN RESOLUTION ENGINE**: 턴 해결, 충돌 계산
8. **GAME OUTCOME HANDLERS**: 승리/패배 처리
9. **SHOP SCREEN**: 상점 화면
10. **FORGE SCREEN**: 대장간 화면
11. **DOJO SCREEN**: 도장 화면
12. **ICON GALLERY**: 아이콘 도감
13. **ADMIN CONSOLE**: 런타임 데이터 편집
14. **INITIALIZATION**: 앱 시작점

### 주요 함수

#### 전투 턴 흐름

```
playerCommit()
  → resolveTurn()
    → resolveIntents()      // 거리 계산
    → executeCommand()      // 기예 효과 적용
    → checkGameOver()       // 승패 확인
    → renderCombat()        // UI 갱신
```

#### 거리 충돌 해결

```javascript
function resolveIntents(playerIntent, enemyIntent) {
  const delta = -playerIntent + enemyIntent;
  game.combat.distance = clamp(game.combat.distance + delta, 0, 4);
}
```

#### 기예 충돌 계산

```javascript
function executeCommand(pCmd, eCmd) {
  // 1. 우선순위 체크
  const pSpeed = COMMANDS[pCmd].speed;
  const eSpeed = COMMANDS[eCmd].speed;

  if (BALANCE.combat.speedDifferenceBlocks && pSpeed !== eSpeed) {
    // 빠른 쪽만 적중
    if (pSpeed < eSpeed) {
      // 플레이어만 적중
    } else {
      // 적만 적중
    }
  } else {
    // 같은 우선순위 - 충돌 처리
  }
}
```

---

## 새 기능 추가하기

### 예시: 새로운 기예 타입 추가

#### 1. 데이터 추가 (commands.json)

```json
{
  "backstab": {
    "id": "backstab",
    "name": "암습",
    "nameEn": "Backstab",
    "icon": "i-feint",
    "cost": 3,
    "hp": [8, 12],
    "bal": [0, 0],
    "ranges": [0],
    "speed": 0,
    "backstab": true,
    "desc": "후퇴 중인 적에게 2배 데미지"
  }
}
```

#### 2. 로직 추가 (index.html의 executeCommand)

```javascript
// executeCommand 함수 내부에 추가
if (pCmdData.backstab && enemyIntent < 0) {
  // 적이 후퇴 중이면 2배
  pDmg *= 2;
  result.narrative += ` (암습!)`;
}
```

#### 3. 클래스 덱에 추가 (classes.json)

```json
{
  "dobu": {
    ...
    "deck": ["hwando", "shuriken", "parry", "dodge", "ambush", "feint", "backstab"]
  }
}
```

### 예시: 새로운 화면 추가

#### 1. HTML 섹션 추가

```html
<section id="screen-newfeature" class="screen">
  <button class="btn-home" onclick="goHome()">← 메인</button>
  <header class="scene-header">
    <div class="scene-title">새 기능</div>
  </header>
  <div id="newfeature-content"></div>
  <button class="shop-back-btn" onclick="enterMap()">돌아가기</button>
</section>
```

#### 2. CSS 스타일 추가 (styles.css)

```css
#newfeature-content {
  flex: 1;
  overflow-y: auto;
  padding: 10px;
}
```

#### 3. JS 함수 추가 (index.html)

```javascript
function renderNewFeature() {
  const content = $('newfeature-content');
  content.innerHTML = '...';
  showScreen('newfeature');
}
```

---

## Admin Console 사용법

Admin Console은 브라우저에서 직접 게임 데이터를 수정하고 GitHub에 자동 commit할 수 있는 도구입니다.

### 설정 방법

1. 메인 화면 우상단 ⚙ 버튼 클릭
2. "GitHub 설정" 버튼 클릭
3. 다음 정보 입력:
   - **Repository**: `yourname/bladesword`
   - **Branch**: `main`
   - **Personal Access Token**: [GitHub Settings](https://github.com/settings/tokens)에서 발급
     - Fine-grained token 사용
     - Contents: Read and write 권한 필요
     - Metadata: Read 권한 필요

### 사용 방법

1. Admin Console에서 원하는 탭 선택 (Commands, Classes, Enemies 등)
2. 행 클릭하여 펼치기
3. 필드 수정
4. **"저장"** 버튼 클릭
   - localStorage에 즉시 저장 (같은 기기에서 즉시 반영)
   - GitHub API로 자동 commit (다른 기기에서 1~2분 후 반영)

### 주의사항

- **localStorage 우선**: Admin Console로 수정한 값은 localStorage에 저장되며, 원본 JSON 파일보다 우선합니다
- **"원본 리셋"** 버튼: localStorage를 비우고 원본 JSON 파일로 되돌립니다
- **충돌 가능성**: 여러 기기에서 동시 수정 시 마지막 commit이 이전 변경사항을 덮어씁니다

---

## 배포 워크플로우

### GitHub Pages 자동 배포

```bash
# 로컬에서 변경 후
git add .
git commit -m "feat: 새 기예 추가"
git push origin main

# → 1~2분 후 https://yourname.github.io/bladesword/ 에 자동 배포
```

### 로컬 테스트

```bash
# HTTP 서버 실행 (file:// 프로토콜은 fetch 차단됨)
python3 -m http.server 8000

# 브라우저에서 http://localhost:8000/ 접속
```

### 모바일 테스트

1. GitHub Pages 배포 후 핸드폰 브라우저에서 접속
2. 또는 로컬 서버를 같은 Wi-Fi 네트워크에서 접속:
   ```bash
   # 내 IP 확인
   ifconfig | grep "inet "

   # 핸드폰에서 http://192.168.x.x:8000/ 접속
   ```

---

## 주의사항

### 절대 하지 말 것

1. **빌드 시스템 도입 금지**
   - npm, webpack, bundler, TypeScript 추가 금지
   - 단일 HTML + JSON의 단순함이 핵심 가치

2. **JS 코드 분리 금지**
   - `index.html`을 여러 .js 파일로 쪼개지 말 것
   - 데이터만 JSON으로 분리, 코드는 한 파일

3. **하드코딩 금지**
   - `balance.json` 값을 코드에 하드코딩하지 말 것
   - 항상 `BALANCE.foo` 형태로 참조

4. **localStorage 키 변경 금지**
   - `ikkiuchi-data-*`, `ikkiuchi-github-*` 키 이름 변경 시 기존 사용자 데이터 손실

5. **종이 프로토 없는 메커니즘 추가 금지**
   - 새 시스템은 종이로 5번 테스트 후 코드화

### 권장 사항

1. **데이터 우선 설계**
   - 새 기능을 만들기 전에 "이 값을 사용자가 수정하고 싶어할까?" 자문
   - 답이 yes면 JSON으로 분리

2. **모바일 우선 테스트**
   - viewport 380px 기준으로 레이아웃 검증
   - 터치 영역은 최소 44x44px

3. **커밋 메시지 컨벤션**
   - `feat:` 새 기능
   - `fix:` 버그 수정
   - `data:` 데이터 조정
   - 예: `feat: 자객 클래스 추가`, `data: 베기 데미지 3-5 → 4-6`

4. **성능 고려**
   - 모든 데이터는 메모리에 로드됨 (총 ~30KB)
   - 거대한 JSON 파일 생성 지양

---

## 코드 컨벤션

### JavaScript

```javascript
// ES2020+ vanilla JS
// 화살표 함수 OK, async/await OK
const foo = (a, b) => a + b;

// DOM 접근: $() 헬퍼 사용
const elem = $('my-id');  // document.getElementById의 약칭

// 이벤트 핸들러: id 기반
$('my-btn').onclick = () => { ... };

// 상태는 module-level 객체에 모임
const state = { ... };
const game = { ... };
```

### CSS

```css
/* 변수 적극 사용 */
:root {
  --bg: #1a1612;
  --enemy: #a4323a;
  --player: #6b8e8f;
}

/* BEM 컨벤션 미사용, 단순 클래스명 */
.class-card { ... }
.class-card:hover { ... }
```

---

## 디버깅 팁

### 브라우저 콘솔 활용

```javascript
// 전역 상태 확인
console.log(game);
console.log(COMMANDS);
console.log(BALANCE);

// 전투 시뮬레이션
playerCommit();  // 턴 강제 실행

// localStorage 초기화
localStorage.clear();
location.reload();
```

### Admin Console JSON 내보내기

1. Admin Console 열기
2. "JSON 내보내기" 버튼 클릭
3. 전체 데이터를 JSON 파일로 다운로드
4. 문제 발생 시 백업 복원 가능

---

## 자주 하는 작업

| 작업 | 파일 | 섹션 |
|---|---|---|
| 적 데미지 조정 | `data/enemies.json` | `deck` 배열의 기예 |
| 시작 자원 변경 | `data/balance.json` | `startingGold` 등 |
| 새 기예 추가 | `data/commands.json` + `index.html` | `executeCommand` |
| 상점 가격 조정 | `data/balance.json` | `shop` 섹션 |
| AI 행동 수정 | `data/balance.json` | `ai` 섹션 가중치 |
| 새 화면 추가 | `index.html` | `<section class="screen">` + 라우터 |
| UI 색상 변경 | `styles.css` | `:root` 변수 |
| 새 폰트 추가 | `index.html` | Google Fonts CDN |
| 아이콘 추가 | `index.html` | `<svg><defs>` 섹션 + 아이콘 도감 함수 |

---

## 고급 주제

### 새로운 클래스 고유 능력 구현

예시: "피의 검무" - 체력 50% 이하일 때 데미지 +30%

#### 1. classes.json 수정

```json
{
  "berserker": {
    ...
    "mechanic": "피의 검무",
    "mechanicDesc": "HP 50% 이하일 때 데미지 +30%"
  }
}
```

#### 2. executeCommand 함수 수정

```javascript
// index.html의 executeCommand 함수 내부
let pDmg = randomRange(pCmdData.hp);

// 클래스 고유 능력
if (game.classId === 'berserker' && game.player.hp <= game.player.maxHp * 0.5) {
  pDmg = Math.round(pDmg * 1.3);
  result.narrative += ` (피의 검무!)`;
}
```

### 새로운 전투 메커니즘 추가

예시: "출혈" 상태 이상

#### 1. balance.json에 상수 추가

```json
{
  "bleed": {
    "damagePerTurn": 2,
    "duration": 3
  }
}
```

#### 2. game state에 추적 변수 추가

```javascript
// GAME STATE 섹션
const game = {
  ...
  combat: {
    ...
    playerBleed: 0,  // 출혈 턴 카운터
    enemyBleed: 0
  }
};
```

#### 3. 턴 시작/종료에 처리 로직 추가

```javascript
function resolveTurn() {
  // 턴 시작 시 출혈 데미지
  if (game.combat.playerBleed > 0) {
    game.player.hp -= BALANCE.bleed.damagePerTurn;
    game.combat.playerBleed--;
    result.narrative += ` (출혈 ${BALANCE.bleed.damagePerTurn} 데미지)`;
  }

  // ... 기존 로직 ...
}
```

---

## 참고 자료

- [Google Fonts](https://fonts.google.com/) - 폰트 추가
- [MDN Web Docs](https://developer.mozilla.org/) - JS/CSS 레퍼런스
- [GitHub Pages 문서](https://docs.github.com/en/pages) - 배포 가이드
- [data/README.md](data/README.md) - 데이터 튜닝 가이드
- [GAMEPLAY.md](GAMEPLAY.md) - 게임 시스템 설명
- [CLAUDE.md](CLAUDE.md) - 프로젝트 개요

---

**Happy Coding!** ⚔️

문의사항은 [GitHub Issues](https://github.com/yourusername/bladesword/issues)로 남겨주세요.
