# 하네스 엔지니어링 템플릿

Claude Code에서 3-Agent 하네스 구조(Planner → Generator → Evaluator)로 백엔드/데이터 처리 코드를 설계·구현·검수하는 템플릿입니다.

하네스 없이 단일 LLM에게 코드를 맡기면 설계와 구현, 평가가 같은 컨텍스트에서 이루어져 자기 결과물에 관대해지는 경향이 있습니다. 이 템플릿은 역할을 분리된 서브에이전트에 위임함으로써 각 단계의 독립성을 보장합니다.

---

## 프로젝트 구조

```
my-service/
├── CLAUDE.md                          ← 오케스트레이터 지시서 (Claude Code가 자동으로 읽음)
├── harness/
│   ├── AGENTS.md                      ← 에이전트 목록
│   ├── agents/
│   │   ├── planner.md                 ← Planner 서브에이전트 지시서
│   │   ├── generator.md               ← Generator 서브에이전트 지시서
│   │   └── evaluator.md               ← Evaluator 서브에이전트 지시서
│   └── docs/
│       ├── harness_workflow.md        ← 워크플로우 실행 흐름
│       ├── evaluation_criteria.md     ← 공용 평가 기준
│       └── exec_plans/                ← 태스크별 산출물 저장
│           └── <type>_<task_name>/
│               ├── SPEC.md
│               ├── SELF_CHECK.md      ← 중대형 태스크(기능 6개 이상)만 생성
│               └── QA_REPORT.md
├── src/                               ← Generator가 생성하는 구현 코드
└── tests/                             ← Generator가 생성하는 테스트 코드
```

`harness/` 폴더는 서비스 코드와 완전히 분리된 하네스 전용 영역입니다. 새 서비스 프로젝트에 적용할 때는 `harness/` 폴더와 `CLAUDE.md`만 복사하면 됩니다.

---

## 동작 방식

Claude Code가 `CLAUDE.md`를 읽고 오케스트레이터 역할을 합니다. 오케스트레이터는 태스크 규모를 판단하여 아래 두 흐름 중 하나를 실행합니다.

**소규모** (버그 수정·유틸 추가·단일 모듈 수정):
1. Generator 서브에이전트가 코드를 구현합니다
2. Evaluator 서브에이전트가 검수합니다
3. 합격이면 커밋합니다

**중대형** (신규 모듈·파이프라인·다중 모듈 연동):
1. Planner 서브에이전트가 `docs/exec_plans/<type>_<task_name>/SPEC.md`를 생성합니다
2. Generator 서브에이전트가 `src/`, `tests/`에 코드를 구현합니다
3. Evaluator 서브에이전트가 ruff·mypy·pytest를 실행하고 QA_REPORT.md를 생성합니다
4. 불합격이면 Generator가 피드백을 반영하여 재작업합니다 (최대 2회)
5. 합격이면 커밋합니다

각 에이전트는 독립된 컨텍스트에서 실행되어 서로의 판단에 영향을 주지 않습니다.

---

## 사용 방법

### 1단계: 이 폴더를 서비스 프로젝트에 적용합니다

```bash
cp -r harness/ my-service/
cp CLAUDE.md my-service/
cd my-service
```

### 2단계: Claude Code를 실행합니다

```bash
claude
```

### 3단계: 프롬프트 한 줄을 입력합니다

```
CSV 파일을 읽어 통계를 계산하는 데이터 파이프라인을 만들어줘
```

### 4단계: 결과를 확인합니다

```bash
ls src/
python -m pytest tests/ -v
```

---

## 평가 기준을 바꾸고 싶다면

`harness/docs/evaluation_criteria.md`를 수정하세요.

---

## Solo 비교 실험을 하고 싶다면

```bash
mkdir solo-test && cd solo-test
claude

> CSV 파일을 읽어 통계를 계산하는 데이터 파이프라인을 만들어줘
```

하네스 결과와 나란히 비교하면 차이가 명확히 보입니다.
