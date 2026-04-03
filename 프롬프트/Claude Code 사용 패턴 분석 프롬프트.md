# Claude Code 사용 패턴 분석 프롬프트

## 💡 사용 팁

### 처음 사용하는 경우

1. 먼저 터미널에서 `/insights` 실행 (Facets 생성)
2. 아래 프롬프트 복사-붙여넣기
3. 결과 확인

### 정기적으로 사용하는 경우

**주간 루틴** (매주 월요일):

```
지난 주 Claude Code 사용 패턴 변화를 분석해줘. ~/facets_stats.py를 실행하고 이전 주와 비교해서 변화가 있는지 확인해줘.
```

**월간 루틴** (매월 1일):

```
지난 달 Claude Code 사용 패턴을 심층 분석해줘. Conductor vs CLI 비교, 효율성 트렌드, 마찰 지점 변화를 분석하고 다음 달 개선 방안을 제안해줘.
```

---

## 🚀 복사해서 사용하세요

```
나의 Claude Code 사용 패턴을 분석해줘. (1) ~/facets_stats.py 스크립트를 만들어서 ~/.claude/usage-data/facets/*.json 파일들을 읽고 session_type, helpfulness, outcome, primary_success, friction_counts를 집계해. (2) ~/conductor_vs_cli.py 스크립트를 만들어서 각 세션이 conductor-workspaces에 있는지 확인하고 Conductor vs CLI를 비교해. (3) 두 스크립트를 실행한 결과를 한국어로 요약하고, 핵심 인사이트 3가지와 액션 플랜을 제안해줘.
```

---

## 📋 더 상세한 버전 (선택사항)

아래 프롬프트를 사용하면 더 체계적인 분석을 받을 수 있습니다:

```
나의 Claude Code 사용 패턴을 자동 분석해줘:

1단계: ~/.claude/usage-data/facets/ 디렉토리의 모든 JSON 파일을 읽어서 다음을 집계:
- session_type 분포 (single_task, multi_task, iterative_refinement, exploration, debugging, quick_question)
- claude_helpfulness 분포 (essential, very_helpful, moderately_helpful, slightly_helpful, not_helpful)
- outcome 분포 (fully_achieved, mostly_achieved, partially_achieved, not_achieved, unclear)
- primary_success 빈도
- friction_counts 합계

2단계: 각 세션의 .jsonl 파일 위치를 찾아서 conductor-workspaces 경로 포함 여부로 Conductor vs CLI 구분

3단계: 결과를 다음 형식으로 한국어 요약:
### 전체 통계
- 총 세션, 가장 흔한 유형, Very Helpful 비율, Success 비율, 주요 성공 요인, 주요 마찰

### Conductor vs CLI 비교
- 세션 수, Very Helpful 비율, Success 비율, 주요 유형 비교표

### 핵심 인사이트 (3가지)
- 발견, 의미, 활용

### 나만의 작업 패턴
- 패턴명, 특징 3가지, 강점/약점

### 액션 플랜
- 즉시 실행 3가지, 1주일 실험 2가지

### 한 줄 요약

스크립트를 ~/facets_stats.py와 ~/conductor_vs_cli.py로 저장하고 실행해서 결과를 보여줘.
```
