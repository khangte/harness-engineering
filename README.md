# 하네스 엔지니어링 기초 템플릿

## 프로젝트 구조

```
harness-project/
├── CLAUDE.md                          ← 오케스트레이터 지시서 (Claude Code가 자동으로 읽음)
├── AGENTS.md                          ← 에이전트 목록
├── agents/
│   ├── planner.md                     ← Planner 서브에이전트 지시서
│   ├── generator.md                   ← Generator 서브에이전트 지시서
│   └── evaluator.md                   ← Evaluator 서브에이전트 지시서
├── docs/
│   ├── harness_workflow.md            ← 워크플로우 실행 흐름
│   ├── evaluation_criteria.md         ← 공용 평가 기준
│   ├── report_template.md             ← QA 리포트 템플릿
│   └── exec-plans/                    ← 태스크별 SPEC.md, QA_REPORT.md 저장
│       └── <type>-<task-name>/
│           ├── SPEC.md
│           ├── SELF_CHECK.md          ← 중대형 태스크(기능 6개 이상)만 생성
│           └── QA_REPORT.md
├── src/                               ← Generator가 생성하는 구현 코드
└── tests/                             ← Generator가 생성하는 테스트 코드
```

---

## 실행 방법

### 1단계: 이 폴더에서 Claude Code를 실행합니다

```bash
cd harness-project
claude
```

Claude Code가 CLAUDE.md를 자동으로 읽고 오케스트레이터 역할을 합니다.

### 2단계: 프롬프트 한 줄을 입력합니다

```
CSV 파일을 읽어 통계를 계산하는 데이터 파이프라인을 만들어줘
```

이것만 치면 됩니다.
오케스트레이터가 태스크 규모를 판단하고 자동으로:

**소규모** (버그 수정·유틸 추가·단일 모듈 수정):
1. Generator 서브에이전트가 코드를 구현합니다
2. Evaluator 서브에이전트가 검수합니다
3. 합격이면 커밋합니다

**중대형** (신규 모듈·파이프라인·다중 모듈 연동):
1. Planner 서브에이전트가 `docs/exec-plans/<type>-<task-name>/SPEC.md`를 생성합니다
2. Generator 서브에이전트가 `src/`, `tests/`에 코드를 구현합니다
3. Evaluator 서브에이전트가 ruff·mypy·pytest를 실행하고 QA_REPORT.md를 생성합니다
4. 불합격이면 Generator가 피드백을 반영하여 재작업합니다 (최대 2회)
5. 합격이면 커밋하고 완료 보고가 나옵니다

### 3단계: 결과를 확인합니다

```bash
ls src/
python -m pytest tests/ -v
```

---

## 평가 기준을 바꾸고 싶다면

`docs/evaluation_criteria.md`를 수정하세요.

---

## Solo 비교 실험을 하고 싶다면

하네스 없이 Solo로 실행한 결과를 비교하고 싶으면:

```bash
# 다른 폴더에서 Claude Code 실행 (CLAUDE.md가 없는 곳)
mkdir solo-test && cd solo-test
claude

# 같은 프롬프트 입력
> CSV 파일을 읽어 통계를 계산하는 데이터 파이프라인을 만들어줘
```

Solo 결과와 하네스 결과를 나란히 비교하면 차이가 명확히 보입니다.
