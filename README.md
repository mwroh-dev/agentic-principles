# 에이전트 시스템 베스트 프랙티스

멀티에이전트 시스템 설계와 운영에서 논문/공식 문서로 검증된 원칙들을 정리한 지식베이스입니다.

---

## 이 저장소의 목적과 워크플로우

에이전트·오케스트레이션·스킬을 만들 때마다 반복적으로 입력하게 되는 설계 원칙들을 한 곳에 모은 것이다. 이 저장소의 문서들이 생성·수정·검증의 기준값으로 쓰인다.

### KB는 판례다 — 닫힌 결론이 아니다

이 KB의 문서들은 **법률 조항이 아니라 판례**처럼 읽어야 한다.

- 판례는 "이 상황에서는 이런 속성을 만족해야 한다"를 알려준다.
- 판례는 "반드시 이 방법을 써라"를 강제하지 않는다.
- 에이전트·모델은 이 문서들을 읽고 **어떤 속성이 복원되어야 하는지**를 판단해야 한다. 구체적인 구현 방법은 컨텍스트에 따라 다를 수 있다.

각 confirmed 문서의 `Properties to Restore` 섹션은 이 원칙에 따라 작성되었다:
- 위반이 감지됐을 때 수정된 결과가 **만족해야 할 속성들**을 기술한다.
- 특정 해법("Y로 바꿔라")을 처방하지 않는다.
- 모델의 판단 공간을 닫지 않는다.

> `AGENTS.md` 또는 시스템 컨텍스트에 이 KB를 주입할 때: "이 원칙들을 따르면서 만들어라" 또는 "이 원칙을 위반하는지 확인하라"는 지시는 유효하다. 단, 위반이 감지되면 모델은 `Properties to Restore`를 기준으로 방향을 제시해야 하며, 단일 해법으로 결론을 닫지 않아야 한다.

### 아이디어 → 배포 흐름

1. **노트** — 작업 중 떠오른 raw idea를 메모한다.
2. **블로그 또는 초안** — 정리된 아이디어를 블로그 글로 쓰거나 로컬 초안으로 올린다.
3. **논문 서치** — 비슷한 학술 논문이 있는지 웹서치로 확인한다. 나만의 생각인지, 이미 검증된 패턴인지 판단하기 위해서다.
4. **5차원 점수** — 19점 이상일 때만 `confirmed/`로 승격한다.
5. **배포** — `confirmed/` 문서만 에이전트 컨텍스트에 주입하거나 팀 공유 기준으로 쓴다.

### 문서 생성 방식

문서는 직접 쓰지 않는다. raw idea를 던지면 4단계 페르소나 파이프라인이 문서를 만든다:


| 단계  | 페르소나                   | 역할                                             |
| --- | ---------------------- | ---------------------------------------------- |
| 1   | **Architect**          | 원칙의 구조와 경계를 정의한다. 언제 적용하고 언제 적용하지 않는지를 명확히 한다. |
| 2   | **AI Abstractor**      | 논문과 공식 문서에서 근거를 찾아 연결한다. 주장이 아닌 증거로 뒷받침한다.     |
| 3   | Technical Editor       | 언어를 정제하고 일관성을 맞춘다. 모호한 표현을 제거한다.               |
| 4   | **Principal Engineer** | 실무 적용 가능성을 검토한다. Known Limits와 엣지 케이스를 추가한다.   |


---

## Tier 구분


| Tier          | 기준                            | 용도                     |
| ------------- | ----------------------------- | ---------------------- |
| **confirmed** | 논문 + 공식 문서 + 5차원 점수 19+/25 통과 | 에이전트 컨텍스트 주입에 바로 사용 가능 |
| **candidate** | 실무에서 관찰된 패턴, 논문 근거 확보         | 독립 검증 대기 중 — 참고용       |


### 5차원 점수

각 차원은 1–5점이며 합계 25점 만점이다. Confirmed 승격 기준은 19점 이상.


| 차원        | 의미                            | 높은 점수 기준                                  |
| --------- | ----------------------------- | ----------------------------------------- |
| **코어성**   | 에이전트 시스템의 핵심 속성을 직접 다루는가      | 에이전트 정의·경계·책임 분리에 직접 영향을 미침               |
| **리스크감소** | 실패 모드와 운영 리스크를 얼마나 줄이는가       | 이 원칙 없을 때 발생하는 실패가 명확하고 심각함               |
| **확장성**   | 시스템이 커질수록 원칙의 가치가 커지는가        | 단일 에이전트보다 multi-agent 환경에서 더 중요해짐         |
| **제어성**   | 인간이 시스템 동작을 관찰하고 개입할 수 있게 하는가 | audit, override, checkpoint를 구조적으로 가능하게 함 |
| **기록성**   | 결정 근거와 실행 흐름을 추적 가능하게 하는가     | 장애 재현과 근본 원인 귀인이 가능해짐                     |


---

## 아키텍처

### [에이전트는 계약이지 코드가 아니다](confirmed/architecture/agent-as-contract-not-code.md)

에이전트 정의 단위에는 역할 계약(입력/출력/허용 스킬/실패 모드)만 둔다. 절차 실행 로직을 에이전트 정의 안에 두면 판단 계층과 실행 계층이 구조적으로 결합되어 컨텍스트 오염, 격리 테스트 불가, 재사용 불가 세 가지 실패가 동시에 발생한다.

### [규칙은 스킬이 아니다](confirmed/architecture/rule-is-not-skill.md)

정적 강제 규칙(schema validation, linter, policy constraint)과 에이전트가 상황에 따라 호출하는 절차(skill)는 다른 계층에 속한다. 규칙은 LLM 상태에 관계없이 항상 강제되고, 스킬은 컨텍스트 매칭 시에만 발동된다.

### [금지 계층화 — 수보다 수준을 높여라](confirmed/architecture/constraint-hierarchy-over-accumulation.md)

금지 조항을 쌓는 것은 안전을 더하는 것이 아니다. 임계점 이후에는 제약들이 서로 경쟁해 노이즈가 되고, 진짜 중요한 금지마저 신뢰도를 잃는다.

### [직렬(Phase) vs 병렬(Lane) 실행](confirmed/architecture/phase-vs-lane-execution.md)

태스크 그래프의 의존성 구조가 dispatch topology를 결정한다. 의존성이 있으면 직렬, 없으면 병렬. 이 결정을 runtime 추론에 맡기는 것은 아키텍처 결함이다.

### [목적별 권한 분리](confirmed/architecture/purpose-scoped-authority.md)

코드, 명세, 지식, 스킬, 에이전트, 문서는 각자의 도메인에 대해 독립적 권한을 갖는다. 단일 정본(SSOT)으로 강제 통합하면 각 artifact의 목적 적합성이 저하된다.

### [Orchestrator-Gated 컨텍스트 분배](confirmed/architecture/orchestrator-gated-context-distribution.md)

오케스트레이터는 전체 아티팩트의 유일한 보유자다. 하위 에이전트는 자신의 역할에 필요한 필드만 받고, 결정 결과 아티팩트만 반환한다.

### [Artifact 배치 결정 Framework](confirmed/architecture/artifact-placement-decision-framework.md)

새로운 파일/결과물을 어디에 둘지 결정할 때 5가지 질문을 순서대로 적용한다: runtime 집행 필요 → 인간 문서 → 재사용 절차 → 진화 휴리스틱 → 복합 관심사.

### [Repository 디렉토리 구조](confirmed/architecture/agent-directory-structure.md)

소유권과 생명주기 기준으로 5개 영역으로 분리한다: `agents/` `skills/` `artifacts/` `knowledge/` `docs/`.

### [스킬 실행 Surface 타입](confirmed/architecture/skill-surface-types.md)

스킬은 실행 메커니즘에 따라 `local_module`, `repo_skill`, `codex_subagent`, `model_replay` 4가지 타입으로 구분한다.

### [서브에이전트 컨텍스트 격리](confirmed/architecture/subagent-per-task-isolation.md)

서브에이전트 프롬프트는 완전히 자기완결적이어야 한다. 오케스트레이터 히스토리를 전달하는 것은 오류다.

### [Schema-Valid Is Not Runtime-Executable](confirmed/architecture/schema-valid-not-runtime-executable.md)

LLM 출력이 schema를 통과해도 실행 가능하다는 보장이 없다. 오케스트레이터 dispatch 경계에서 shape → semantic → fact → supportability 4개 게이트를 순서대로 통과해야 한다. 어느 게이트에서든 실패하면 blocked — 자동 보정하지 않는다.

### [스킬 디스커버리는 인가가 아니다](confirmed/architecture/skill-discovery-is-not-authorization.md)

모델이 발견 가능한 스킬은 공개 capability surface이며 특정 에이전트의 사적 구현이 아니다. `skills:` preload는 allowlist가 아니고, 폴더 이름과 프롬프트 경고는 인가 경계가 아니다. 에이전트별 배타적 사용은 tool 권한(`disallowedTools: Skill`, `Skill(name)` deny rule), 스코프된 서브에이전트 설정, 디스커버리 제어로 강제해야 한다.

### [계획은 제어 구조다](confirmed/architecture/plan-as-control-structure.md)

계획은 실행 지시서가 아니라 이탈 감지를 위한 제어 구조다. 동적 우선순위 태스크 큐로 분해하고 각 checkpoint에서 비교해야 한다.

### [반복 작업은 코드로](confirmed/architecture/code-over-prompt-for-repetitive-tasks.md)

동일한 절차가 반복 실행된다면 LLM 호출이 아닌 코드로 처리한다. 손익분기점은 약 17회 실행이다.

---

## 지시문 설계

### [위임 전 명세 선행](confirmed/instructions/front-loaded-specification-for-delegation.md)

위임 품질은 위임 시점에 결정된다. 수정 루프는 명세 갭의 후행 신호이며 비용은 이미 발생한 뒤다. 목표, 제약, 완료 조건, 엣지 케이스를 넘기기 전에 명시하라.

### [프롬프트 모호성은 근본 원인이다](confirmed/instructions/prompt-ambiguity-as-root-cause.md)

에이전트는 모호한 입력에 대해 모호성 신호를 내보내지 않는다. 자체적으로 해석해 침묵하고 진행한다. 모호성은 구현 갭보다 선행하는 근본 원인이다.

### [구조화된 Seed 지시문](confirmed/instructions/structured-seed-directive.md)

에이전트 지시문은 목표(Goal), 제약(Constraint), 우발 상황(Contingency)을 분리해 명시해야 한다. 구조 없이 목표만 주면 에이전트의 지시 따르기 능력 상한이 낮아진다.

---

## 실행

### [평가 기준 선행](confirmed/execution/evaluation-before-implementation.md)

에이전트 시스템 설계에서 가장 먼저 정해야 하는 것은 구현 방법이 아니라 "무엇을 평가할 것인가"다. 평가 기준이 없으면 artifact 모델, 에이전트 경계, 스킬 surface가 구현자 편의로 수렴한다.

### [이중 극성 평가](confirmed/execution/dual-polarity-evaluation.md)

평가 스위트는 반드시 통과해야 하는 황금 케이스와 반드시 실패해야 하는 레드 케이스를 모두 포함해야 한다. 단방향 평가는 지름길 활용을 감지할 수 없다.

### [원자적 커밋 추적 가능성](confirmed/execution/atomic-commit-traceability.md)

에이전트 작업의 각 단계는 독립적으로 검증하고 되돌릴 수 있는 단위여야 한다. 원자적이지 않으면 오류가 침묵하며 누적되고 실패 지점을 격리할 수 없다.

---

## 에이전트 설계

### [방향 유지 vs 출력 일관성](confirmed/agents/direction-maintenance-over-output-consistency.md)

Harness는 출력 형식 일관성보다 방향 피드백-복구 메커니즘을 우선시해야 한다. 방향 감지는 출력 형식 검사가 아닌 의사결정 패턴 관찰로 한다.

### [설계에 의한 Human Checkpoint](confirmed/agents/human-checkpoint-by-design.md)

인간 감독 gate는 사후 개입이 아니라 사전 설계 요소다. 언제 어떤 조건에서 인간 판단이 필요한지를 에이전트 정의에 명시해야 한다.

### [Eval이 곧 행동 명세](confirmed/agents/evaluation-as-behavioral-specification.md)

Eval은 구현 후 검증 도구가 아니다 — 구현 전에 쓰는 행동 명세다. 출력이 맞아 보여도 trace가 잘못됐다면 실패다.

### [Harness Hill Climbing](confirmed/agents/harness-hill-climbing.md)

반복당 정확히 하나의 요소만 변경한다. 2개 이상 동시 변경은 인과 귀속을 불가능하게 만든다.

### [에이전트별 지식 패턴](confirmed/agents/per-agent-knowledge-patterns.md)

에이전트 유형에 따라 최적 지식 업데이트 전략이 다르다: 즉시(A), 배치(B), 예측 오류 트리거(B+prediction-error).

---

## 관측 가능성

### [3계층 관측 모델](confirmed/observability/3-tier-observability-model.md)

관측 데이터는 3계층으로 분리해야 한다: raw telemetry(LLM I/O, tool call) → evidence artifact(시나리오 단위 증거) → report(분석). 계층을 합치면 재현 불가능한 observability가 생긴다.

### [실패 로깅 반생존자 편향](confirmed/observability/failure-logging-anti-survivorship-bias.md)

성공한 실행 추적만 보존하면 실패 모드 분포와 근본 원인 귀인이 비가시화된다. 실패 추적은 평가 케이스로 전환하고, 성공 추적은 샘플링한다.

### [교훈은 Anti-Pattern Registry다](confirmed/observability/lessons-as-anti-pattern-registry.md)

전용 컴팩트 실패 로그는 원시 세션 로그보다 교차 세션 실수 회피에 효과적이다. 성공 기록은 방향을 보여주지만, 실패 기록은 조건과 취약점과 반복 패턴을 노출한다.

---

## 메모리

### [Artifact vs Knowledge 구분](confirmed/memory/artifact-vs-knowledge.md)

Artifact(에피소딕, append-only, gitignore)와 Knowledge(시맨틱, 런 간 갱신, commit)를 같은 저장소에 두면 감사 무결성도 잃고 지식 갱신 능력도 잃는다.

### [지식 업데이트 전략](confirmed/memory/knowledge-update-strategies.md)

Strategy A(즉시), B(배치), B+prediction-error 세 전략의 적합 조건과 위험이 에이전트 유형에 따라 다르다.

---

## 안전

### [코드 기반 4계층 안전 집행](confirmed/safety/multi-layered-safety-via-code.md)

프롬프트 지시문은 안전 정책을 집행할 수 없다. 코드 hook만 가능하다. Hook만 결정론적 계층이다.

---

## 디렉토리 구조

```
confirmed/          ← 검증된 원칙 (19+/25, 논문 근거 필수)
  architecture/     ← 시스템 구조 원칙
  instructions/     ← 지시문 설계 원칙
  execution/        ← 실행 및 평가 원칙
  agents/           ← 에이전트 설계 원칙
  observability/    ← 관측 가능성 원칙
  memory/           ← 메모리 관리 원칙
  safety/           ← 안전 원칙
papers/             ← 인용 논문 중앙 citation index
```

