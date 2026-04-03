# OpenClaw LCK Results Pack

경량 로컬 파일 기반 파이프라인이 포함된 OpenClaw용 LCK 스킬 팩입니다.

## 구성

- `SKILL.md`: OpenClaw에 바로 줄 수 있는 스킬 문서
- `scripts/sync-oracle.js`: Oracle-style CSV를 historical cache JSON으로 변환
- `scripts/analyze-live-game.js`: live game analysis 생성
- `scripts/build-match-report.js`: 날짜별 match analysis 생성
- `samples/oracle-lck-sample.csv`: 샘플 historical CSV

## 빠른 시작

### 1) 패키지 설치

```bash
npm install -g lck-results
```

### 2) historical cache 생성

```bash
node openclaw/lck-results/scripts/sync-oracle.js \
  --csv openclaw/lck-results/samples/oracle-lck-sample.csv
```

### 3) 날짜별 match report 생성

```bash
node openclaw/lck-results/scripts/build-match-report.js \
  --date 2026-04-01
```

### 4) live game analysis 생성

실시간 fetch를 쓰거나, window/details fixture 파일을 넘길 수 있습니다.

```bash
node openclaw/lck-results/scripts/analyze-live-game.js \
  --game 115548128962840652
```

## 기본 cache 위치

- `.openclaw-lck-cache/historical-analysis.json`
- `.openclaw-lck-cache/reports/*.json`

`--cache <dir>` 로 다른 위치를 지정할 수 있습니다.
