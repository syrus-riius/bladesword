# 데이터 튜닝 가이드

이 폴더의 JSON만 수정하면 게임 균형의 90%를 조정할 수 있다. 코드(src/)는 안 건들어도 된다.

## 파일별 역할

| 파일 | 무엇이 들어있나 | 자주 수정? |
|---|---|---|
| `commands.json` | 모든 기예 (베기, 찌르기, 막기 등) | ★★★ 매주 |
| `enemies.json` | 적 데이터 (HP, 덱, 맵 크기) | ★★★ 매주 |
| `classes.json` | 플레이어 클래스 (시작 능력치, 시작 덱) | ★★ 가끔 |
| `weapons.json` | 무기 정의 | ★ 드물게 |
| `map.json` | 스테이지 진행 + 상점 풀 | ★ 드물게 |
| `balance.json` | 시작 자원, 비용, 가격 등 핵심 상수 | ★★★ 매주 |

## 가장 빠른 튜닝 — 어디 만지면 뭐가 바뀌나

### "한 합이 너무 길어 / 짧아"

`balance.json` → `staminaPerTurn` 키우면 매 합 더 많은 행동 가능 → 빠른 결판

### "기예 X 너무 강해"

`commands.json` → 그 기예의 `hp` 또는 `bal` 배열 줄이기. 예: 베기 데미지 약화
```json
"slash": { ... "hp": [3, 5] ... }  →  "hp": [2, 4]
```

### "적 X 너무 어려워 / 쉬워"

`enemies.json` → 그 적의 `hp` 줄이거나 `deck`에서 강한 기예 빼기. 또는 `mapSize` 키워서 도주 공간 늘림(자객·표창쟁이에 효과적).

### "기력 시스템이 너무 빡빡 / 헐렁"

세 개 동시에 봐야 함:
- `balance.json` → `staminaPerTurn` (매 합 회복)
- `balance.json` → `intent.costs` (이동 비용)
- `commands.json` → 각 기예의 `cost`

### "골드 / 가격 균형"

- `balance.json` → `startingGold` (시작 소지금)
- `balance.json` → `shop.prices.*` (가격대)
- `balance.json` → `forge.options[].cost` (대장간 비용)

### "광기 강화 너무 도박"

`balance.json` → `forge.options` 배열의 `reckless` 객체:
- `failChance`: 0.35 → 0.20 (안전화)
- `bonus`: 4 → 6 (보상 ↑)
- `recklessFailPenalty`: 2 → 1 (실패 시 덜 아픔)

## 데이터 형식 규칙 (어기면 게임 안 켜짐)

1. **JSON 따옴표는 큰따옴표 `"`만**. JS는 작은따옴표 OK지만 JSON은 안 됨.
2. **마지막 항목 뒤에 쉼표 X**. `{ "a": 1, "b": 2, }` 안 됨.
3. **숫자는 따옴표 X**. `"hp": 30` (○), `"hp": "30"` (X).
4. **참/거짓은 `true`/`false`**, 따옴표 없음.
5. **수정 후 https://jsonlint.com 에 붙여넣어 검증**하면 안전.

## 새 기예 추가하기

1. `commands.json`에 새 항목 추가 (예: `"crush": { ... }`)
2. 그 기예를 쓸 클래스의 `classes.json` `deck` 배열에 키 추가
3. 또는 적의 `enemies.json` `deck`에 추가
4. 새 충돌 타입(`type`)을 만들었다면 `src/combat.js` 수정 필요 — 이건 코드 영역

## 기예 필드 레퍼런스

| 필드 | 의미 | 예시 |
|---|---|---|
| `icon` | SVG 아이콘 ID (HTML의 `<symbol id="...">`와 매치) | `"i-attack"` |
| `name` | 한글 표시명 | `"베기"` |
| `ranges` | 사거리. 배열 또는 `"all"`(전 거리) | `[1, 2]` |
| `cost` | 기력 비용 | `2` |
| `type` | 충돌 표 키. attack/thrust/block/dodge/parry/rest/feint/push 중 하나 | `"attack"` |
| `hp` | HP 데미지 굴림 [min, max] | `[3, 5]` |
| `bal` | 균형 데미지 굴림 [min, max] | `[2, 3]` |
| `counter` | 받아치기 반격 데미지 [min, max] | `[2, 4]` |
| `staGain` | 사용 시 기력 회복량 | `2` (견제) |
| `balGain` | 사용 시 균형 회복량 | `5` (호흡) |
| `goldGain` | 사용 시 골드 획득 | `5` (흥정) |
| `pierce` | 회피·받아치기 무시 | `true` |
| `onHitBalDmg` | 명중 시 추가 균형 데미지 | `4` (발도술) |
| `bonusOnAdvance` | 전진하며 사용 시 데미지 가산 (백분율) | `0.6` |
| `limit` | 전투당 사용 가능 횟수 | `1` (일격필살) |

## 적 필드 레퍼런스

| 필드 | 의미 |
|---|---|
| `name` | 표시명 |
| `hp` / `bal` / `sta` | 시작 자원 |
| `evade` | 자체 회피율 (0~1) |
| `deck` | 사용 기예 키 배열 |
| `style` | AI 가중치 키 (balance.json의 ai.styleBonus 참조) |
| `goldReward` | 처치 시 골드 |
| `mapSize` | 이 적과 싸울 때의 맵 칸 수 |
| `isElite` / `isBoss` | 표시용 플래그 |
| `flavor` | 적 소개 텍스트 |

## 클래스 필드 레퍼런스

| 필드 | 의미 |
|---|---|
| `id` | 영문 식별자, JSON 키와 같아야 함 |
| `name` / `nameEn` / `culture` / `blurb` | 표시 텍스트 |
| `mechanic` / `mechanicDesc` | 클래스 고유 메커니즘 설명 |
| `hp` / `bal` / `sta` | 시작 자원 |
| `evade` | 회피율 (0~1) |
| `weapon` | weapons.json 키 |
| `deck` | commands.json 키 배열 (시작 덱) |
