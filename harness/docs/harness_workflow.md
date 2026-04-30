# 하네스 실행 워크플로우

해당 워크플로우는 선택이 아니라 **필수**다.

## 태스크 규모 판단 (2-tier)

워크플로우 진입 전, 오케스트레이터가 아래 기준으로 태스크 규모를 먼저 판단한다.

| 구분 | 기준 | 적용 흐름 |
|------|------|-----------|
| **소규모** | 버그 수정, 유틸 함수 추가, 단일 모듈 수정 | Planner 생략 → Generator → Evaluator 1회 → 커밋 |
| **중대형** | 신규 모듈, 파이프라인 설계, 다중 모듈 연동 | 아래 1~4단계 전체 진행 |

소규모 태스크는 SPEC.md, SELF_CHECK.md 작성을 생략하고, Generator에게 사용자 요청을 직접 전달한다.

## 주의사항

- Generator와 Evaluator는 반드시 다른 서브에이전트로 호출(분리가 핵심).
- 각 단계 완료 후 결과 파일 존재 여부 확인

## 서브에이전트 호출 방법

각 단계에서 Task 도구를 사용하여 서브에이전트를 호출한다.
서브에이전트에게 전달할 프롬프트는 아래 "단계별 실행 지시"를 따른다.

중요: 각 서브에이전트는 독립된 컨텍스트에서 실행된다.
이것이 "만드는 AI와 평가하는 AI를 분리"하는 핵심.

## 단계별 실행 지시

### 1 단계: Planner 호출 (중대형 태스크만)

소규모 태스크는 이 단계를 건너뛴다.

서브에이전트에게 아래 내용을 전달합니다:

```
`harness/agents/planner.md` 파일을 읽고, 그 지시를 따라라.
`harness/docs/evaluation_criteria.md` 파일도 읽고 참고하라.

사용자 요청: [사용자가 준 프롬프트]
핵심 의도: [오케스트레이터가 방향·완료 기준을 한두 줄로 명시]

위 내용을 바탕으로 `harness/docs/exec-plans/<type>-<task-name>/SPEC.md`를 처음부터 완성하라.
목표·접근법·완료 기준·데이터 흐름·기능 목록(입출력·검증 기준 포함)을 모두 포함해야 한다.
```

Planner 서브에이전트가 `SPEC.md`를 생성하면 다음 단계로 진행합니다.

### 2 단계: Generator 호출

서브에이전트에게 아래 내용을 전달합니다:

최초 실행 시 (중대형):
```
`harness/agents/generator.md` 파일을 읽고, 그 지시를 따라라.
`harness/docs/evaluation_criteria.md` 파일도 읽고 참고하라.
`harness/docs/exec-plans/<type>-<task-name>/SPEC.md` 파일을 읽고, 전체 기능을 한 번에 구현하라.

구현 코드는 `src/` 디렉토리에, 테스트 코드는 `tests/` 디렉토리에 저장하라.
기능이 6개 이상이면 완료 후 `harness/docs/exec-plans/<type>-<task-name>/SELF_CHECK.md`를 작성하라.
```

최초 실행 시 (소규모):
```
`harness/agents/generator.md` 파일을 읽고, 그 지시를 따라라.
`harness/docs/evaluation_criteria.md` 파일도 읽고 참고하라.

사용자 요청: [사용자가 준 프롬프트]

구현 코드는 `src/` 디렉토리에, 테스트 코드는 `tests/` 디렉토리에 저장하라.
SELF_CHECK.md는 작성하지 않아도 된다.
```

피드백 반영 시 (2회차):
```
`harness/agents/generator.md` 파일을 읽고, 그 지시를 따라라.
`harness/docs/evaluation_criteria.md` 파일도 읽고 참고하라.
`harness/docs/exec-plans/<type>-<task-name>/SPEC.md` 파일을 읽어라. (소규모는 생략)
`src/` 디렉토리의 코드를 읽어라. 이것이 현재 코드다.
`harness/docs/exec-plans/<type>-<task-name>/QA_REPORT.md` 파일을 읽어라. 이것이 QA 피드백이다.

QA 피드백의 "구체적 개선 지시"를 모두 반영하여 코드를 수정하라.
"방향 판단"이 "완전히 다른 접근 시도"이면 아키텍처 자체를 재설계하라.
중대형 태스크이고 기능이 6개 이상이면 완료 후 SELF_CHECK.md를 업데이트하라.
```

### 3 단계: Evaluator 호출

서브에이전트에게 아래 내용을 전달합니다:

```
`harness/agents/evaluator.md` 파일을 읽고, 그 지시를 따라라.
`harness/docs/evaluation_criteria.md` 파일을 읽어라. 이것이 채점 기준이다.
`harness/docs/exec-plans/<type>-<task-name>/SPEC.md` 파일을 읽어라. 이것이 설계서다. (소규모는 생략)
`src/` 디렉토리의 코드를 읽어라. 이것이 검수 대상이다.

검수 절차:
1. `src/` 코드를 분석하라
2. SPEC.md의 기능이 구현되었는지 확인하라 (소규모는 사용자 요청 기준으로 확인)
3. 아래 순서로 실행하고 결과를 기록하라
   - `ruff check src/`
   - `mypy src/`
   - `python -m pytest tests/ -v`
4. `harness/docs/evaluation_criteria.md`에 따라 4개 항목을 채점하라
5. 최종 판정(합격/조건부/불합격)을 내려라
6. 불합격 또는 조건부 시, 구체적 개선 지시를 작성하라

결과를 `harness/docs/exec-plans/<type>-<task-name>/QA_REPORT.md` 파일로 저장하라.
소규모 태스크는 `QA_REPORT.md`를 프로젝트 루트에 저장해도 된다.
```

### 4 단계: 판정 확인

`QA_REPORT.md`를 읽고 판정을 확인합니다.

- "합격" → 5단계(커밋)로 진행.
- "조건부 합격" 또는 "불합격" → 2단계로 돌아가 피드백 반영.
- 최대 반복 횟수: **2회**. 2회 후에도 불합격이면 현재 상태와 이슈를 사용자에게 보고하고 중단한다.

### 5 단계: 커밋 및 완료

합격 판정 후에만 진행한다.

1. `python -m pytest tests/ -v` 가 통과하는지 최종 확인한다.
2. 변경 사항을 커밋한다 (Conventional Commits: `<type>(scope): 설명`).
3. 사용자에게 완료 보고. `src/` 변경 범위를 안내한다.
