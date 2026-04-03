---
name: lck-analytics
description: Riot 공식 LoL Esports 데이터와 Oracle's Elixir 스타일 historical 데이터로 LCK 경기 결과, 현재 순위, live turning point, 밴픽 matchup/synergy, patch meta, 팀 파워 레이팅을 조회한다.
license: MIT
metadata:
  category: sports
  locale: ko-KR
  phase: v2
  install_target: openclaw
---

# LCK Results + Advanced Analysis

## What this skill does

이 스킬은 LCK 조회/분석 전용이다.

- 특정 날짜 LCK 경기 결과 조회
- 특정 팀 alias 정규화 후 필터링
- 현재 스플릿 순위 조회
- 진행 중 경기 live stats 조회
- live timeline 기반 turning point 분석
- Oracle's Elixir 스타일 historical row / CSV 기반
  - 팀 파워 레이팅
  - 챔피언 matchup / synergy 분석
  - patch meta 요약
- 날짜별 match analysis 생성

## When to use

- "오늘 LCK 경기 결과 알려줘"
- "2026-04-01 한화 경기 결과랑 순위 보여줘"
- "지금 T1 경기 킬/골드/오브젝트 요약해줘"
- "이 경기 turning point가 뭐였어?"
- "이 밴픽에서 어느 쪽 조합이 더 좋았는지 설명해줘"
- "현재 패치에서 어떤 챔피언이 메타 픽인지 보여줘"
- "LCK 팀 파워 레이팅 보여줘"

## Prerequisites

- Node.js 18+
- `npm install -g lck-results`

패키지가 없으면 다른 방법으로 우회하지 말고 먼저 전역 설치를 시도한다.

```bash
npm install -g lck-results
```

## Inputs

### 기본 입력

- 날짜: `YYYY-MM-DD`
- 선택 사항: 팀명, 과거 팀명, 한글/영문 약칭 alias

### 고급 분석 입력

- Oracle's Elixir 스타일 CSV 문자열 또는 row 배열
- game id / match id
- live window/details payload 또는 실시간 fetch 권한
- patch version

## Team alias normalization

다음 이름들은 같은 canonical team 으로 인식한다.

- `DN SOOPers`
- `DN FREECS`
- `광동 프릭스`
- `Afreeca Freecs`

추가로 `T1`, `SKT T1`, `담원`, `Dplus KIA`, `브리온`, `한화` 등도 alias 정규화를 지원한다.

## Official surfaces

이 스킬은 Riot 공식 / 공식 웹앱 표면을 우선 사용한다.

- 일정/결과: `getSchedule`
- 토너먼트 목록: `getTournamentsForLeague`
- 순위: `getStandings`
- 이벤트 상세: `getEventDetails`
- 라이브 window: `https://feed.lolesports.com/livestats/v1/window/{gameId}`
- 라이브 details: `https://feed.lolesports.com/livestats/v1/details/{gameId}`

historical 고급 분석은 Oracle's Elixir 스타일 데이터 입력을 사용한다.

## Workflow

### Included lightweight local pipeline

이 OpenClaw 팩에는 경량 로컬 파일 기반 파이프라인 스크립트가 포함된다.

- `scripts/sync-oracle.js`: Oracle-style CSV → historical cache JSON
- `scripts/build-match-report.js`: 날짜별 match analysis 생성
- `scripts/analyze-live-game.js`: game analysis 생성
- 기본 cache 위치: `.openclaw-lck-cache/`

### 0. Install package globally when missing

```bash
npm install -g lck-results
```

### 1. Load the package from global npm root

```bash
GLOBAL_NPM_ROOT="$(npm root -g)" node --input-type=module - <<'JS'
import path from "node:path";
import { pathToFileURL } from "node:url";

const entry = pathToFileURL(
  path.join(process.env.GLOBAL_NPM_ROOT, "lck-results", "src", "index.js"),
).href;

const pkg = await import(entry);
console.log(Object.keys(pkg));
JS
```

### 2. Basic scoreboard / standings query

```bash
GLOBAL_NPM_ROOT="$(npm root -g)" node --input-type=module - <<'JS'
import path from "node:path";
import { pathToFileURL } from "node:url";

const entry = pathToFileURL(
  path.join(process.env.GLOBAL_NPM_ROOT, "lck-results", "src", "index.js"),
).href;
const { getLckSummary } = await import(entry);

const summary = await getLckSummary("2026-04-01", {
  team: "한화",
  includeStandings: true,
});

console.log(JSON.stringify(summary, null, 2));
JS
```

### 3. Historical analytics from Oracle-style CSV

직접 API를 호출해도 되지만, OpenClaw에서는 아래 파이프라인 스크립트 사용을 우선 권장한다.

```bash
node openclaw/lck-results/scripts/sync-oracle.js \
  --csv openclaw/lck-results/samples/oracle-lck-sample.csv
```

### 3a. Historical analytics from Oracle-style CSV via direct API

CSV 문자열이나 row 배열이 있으면 historical dataset 을 만든다.

```bash
GLOBAL_NPM_ROOT="$(npm root -g)" node --input-type=module - <<'JS'
import fs from "node:fs";
import path from "node:path";
import { pathToFileURL } from "node:url";

const entry = pathToFileURL(
  path.join(process.env.GLOBAL_NPM_ROOT, "lck-results", "src", "index.js"),
).href;
const { buildHistoricalAnalytics, getTeamPowerRatings, getPatchMetaReport } = await import(entry);

const csv = fs.readFileSync("./oracle-lck-sample.csv", "utf8");
const historical = buildHistoricalAnalytics(csv);

console.log(JSON.stringify(getTeamPowerRatings(historical), null, 2));
console.log(JSON.stringify(getPatchMetaReport(historical, "16.6.753.8272"), null, 2));
JS
```

### 4. Match analysis via local pipeline script

```bash
node openclaw/lck-results/scripts/build-match-report.js \
  --date 2026-04-01
```

필요하면 팀 필터도 같이 준다.

```bash
node openclaw/lck-results/scripts/build-match-report.js \
  --date 2026-04-01 \
  --team 한화
```

### 5. Game analysis with turning points via local pipeline script

```bash
node openclaw/lck-results/scripts/analyze-live-game.js \
  --game 115548128962840652
```

fixture 기반으로 분석할 때는 `--window`, `--details` 를 같이 줄 수 있다.

### 5a. Game analysis with turning points via direct API

`getGameAnalysis()` 는 live timeline, turning points, draft edge, patch context 를 계산한다.

```bash
GLOBAL_NPM_ROOT="$(npm root -g)" node --input-type=module - <<'JS'
import path from "node:path";
import { pathToFileURL } from "node:url";

const entry = pathToFileURL(
  path.join(process.env.GLOBAL_NPM_ROOT, "lck-results", "src", "index.js"),
).href;
const { buildHistoricalAnalytics, getGameAnalysis } = await import(entry);

const historical = buildHistoricalAnalytics([]);
const analysis = await getGameAnalysis("115548128962840652", {
  historicalDataset: historical,
});

console.log(JSON.stringify(analysis, null, 2));
JS
```

### 6. Match analysis with team power preview via direct API

`getMatchAnalysis()` 는 날짜별 경기 목록에 게임 분석과 팀 파워 preview 를 붙인다.

```bash
GLOBAL_NPM_ROOT="$(npm root -g)" node --input-type=module - <<'JS'
import path from "node:path";
import { pathToFileURL } from "node:url";

const entry = pathToFileURL(
  path.join(process.env.GLOBAL_NPM_ROOT, "lck-results", "src", "index.js"),
).href;
const { buildHistoricalAnalytics, getMatchAnalysis } = await import(entry);

const historical = buildHistoricalAnalytics([]);
const analysis = await getMatchAnalysis("2026-04-01", {
  historicalDataset: historical,
});

console.log(JSON.stringify(analysis, null, 2));
JS
```

## Output guidelines

사용자에게는 원본 JSON을 길게 그대로 던지지 말고 먼저 아래 순서로 정리한다.

### 경기 결과 요청

- 경기 시각
- 팀1 vs 팀2
- 상태
- 세트 스코어
- 요청 팀 경기만 있으면 해당 경기 우선
- standings 요청이 있으면 현재 순위 같이 표시

### 진행 중 경기 요청

- 현재 게임 번호
- 킬 차이
- 골드 차이
- 타워 / 드래곤 / 바론
- 선수-챔피언 정보
- 필요하면 turning point 후보

### 밴픽 / 조합 해석 요청

- role matchup 별 우세 추정
- 팀별 synergy 요약
- overall draft edge
- patch meta context

### 팀 파워 요청

- 팀별 power score
- 최근 경기 수 / 승패
- early game / objective 성향 요약

## Public API reference

- `getMatchResults(date, options)`
- `getStandings(options)`
- `getLckSummary(date, options)`
- `parseOracleCsv(csvText)`
- `buildHistoricalAnalytics(input, options)`
- `getGameAnalysis(gameId, options)`
- `getMatchAnalysis(date, options)`
- `getTeamPowerRatings(input, options)`
- `getPatchMetaReport(input, patch, options)`

## Done when

- 날짜 기준 LCK 경기 요약이 있다
- 팀 요청이면 alias 정규화 후 해당 팀 경기만 남아 있다
- standings 요청이면 현재 순위가 같이 있다
- live game 요청이면 킬/골드/오브젝트/선수 정보가 있다
- turning point 요청이면 timeline 기반 후보가 있다
- draft/meta/power 요청이면 historical analytics 기반 요약이 있다

## Failure modes

- Riot 웹앱용 공식 esports API 응답 구조가 바뀌면 패키지 수정이 필요할 수 있다
- 비시즌 날짜면 결과가 비어 있을 수 있다
- live feed가 비어 있으면 `204` 가 반환될 수 있다
- turning point 분석은 공개 live snapshot 기반 heuristic MVP 이라 GRID event-grade 분석보다 정밀도가 낮을 수 있다
- Oracle-style historical 입력이 없으면 power/meta/matchup 분석은 제한될 수 있다

## Data refresh workflow

모든 LCK 관련 요청에 대해 아래 순서를 따른다.

### 1. Oracle's Elixir 최신 데이터 갱신

요청 시 Oracle's Elixir Google Drive 폴더에서 최신 2026 CSV를 다운로드하여 캐시를 갱신한다.

- Google Drive 파일 ID: `1hnpbrUpBMS1TZI7IovfpKeZfWJH1Aptm`
- 다운로드: `https://drive.usercontent.google.com/download?id=1hnpbrUpBMS1TZI7IovfpKeZfWJH1Aptm&export=download&confirm=t`
- LCK 행만 필터링하여 스킬 호환 CSV로 변환
- `sync-oracle.js`로 historical cache 갱신

### 2. 대회 패치 기준으로 챔피언 정보 조회

**라이브 패치 ≠ 대회 패치**. 반드시 대회에서 사용 중인 패치를 먼저 확인한다.

- Oracle's Elixir 데이터의 `patch` 컬럼에서 해당 대회의 최신 패치 확인
- 또는 Riot 공식 발표 / lolesports.com에서 대회 패치 확인

### 3. 챔피언 솔로큐 데이터 소스 (대회 패치 기준)

에메랄드+ 티어 기준으로 아래 소스에서 조회한다.

#### fow.lol (1순위)
- 티어리스트: `https://fow.lol/stats?patch={patch}&tier=EMERALD%2B`
- 챔피언 상세: `https://fow.lol/stats/{ChampionName}/{position}/{patch}?tier=EMERALD%2B`
- HTML 스크래핑으로 데이터 추출 가능

#### OP.GG API (2순위, fow.lol 실패 시)
- `https://lol-api-champion.op.gg/api/KR/champions/ranked?tier=emerald_plus&version={patch}`
- JSON 응답, champion ID → ddragon 매핑 필요

### 4. 응답 구성

1. Oracle's Elixir 기반 프로 대회 분석 (팀 파워, 매치업, 시너지)
2. 솔로큐 챔피언 티어 (대회 패치 기준, 에메랄드+)
3. 두 데이터를 종합한 인사이트

## Notes

- 이 스킬은 조회 / 분석 전용이다
- 사용자의 "오늘/어제" 요청은 항상 절대 날짜 `YYYY-MM-DD` 로 변환해서 실행한다
- score 요청이면 경기별 한 줄 요약부터 준다
- 특정 팀 요청이면 그 팀 경기와 현재 순위만 먼저 보여준다
- 자세한 구현/설계 문서는 아래를 참고한다
  - `docs/features/lck-results.md`
  - `docs/features/lck-results-advanced-analysis-sources.md`
  - `docs/features/lck-results-advanced-analysis-pipeline.md`
  - `docs/features/lck-results-advanced-analysis-implementation-plan.md`
