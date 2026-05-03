---
name: student-helper-spec-orchestrator
description: 초중학교 생활기록부 PDF 입력 → 고1~고3 TODO 생성 서비스의 빌드 스펙(STUDENT_HELPER_SPEC.md)을 작성하는 전체 워크플로우 오케스트레이터. 도메인 분석 → 프로덕트 청사진 → XML 스펙 작성 → QA 검토를 4명 에이전트 팀으로 조율. "스펙 작성", "spec 작성", "프로덕트 정의", "학생 도우미 스펙", "고교 TODO 서비스 기획", "재실행", "스펙 보완", "특정 섹션 수정", "이전 결과 기반 개선" 등 학생 도우미 서비스 스펙 관련 모든 요청 시 반드시 이 스킬을 사용. 단순 도메인 질문 답변은 직접 응답 가능.
---

# Student Helper Spec Orchestrator

학생 도우미 서비스의 빌드 스펙 문서(`STUDENT_HELPER_SPEC.md`)를 만드는 전체 워크플로우를 조율하는 오케스트레이터.

**실행 모드:** 에이전트 팀 (TeamCreate 기반, 4명)
- `education-domain-expert` (도메인)
- `product-strategist` (프로덕트)
- `spec-writer` (XML 스펙 작성)
- `spec-reviewer` (QA)

## Phase 0: 컨텍스트 확인

워크플로우 시작 시 즉시 확인:

1. `/Users/robin/Downloads/student-helper/_workspace/` 디렉토리 존재 여부
2. `STUDENT_HELPER_SPEC.md` 존재 여부
3. 사용자 요청 유형 분류:
   - **초기 실행**: workspace 없음, spec 없음 → Phase 1부터
   - **부분 재실행**: workspace 있음 + 사용자가 특정 섹션/에이전트만 재호출 요청 → Phase 4로 직행 (해당 에이전트만 호출)
   - **새 실행**: workspace 있음 + 사용자가 새 입력 또는 처음부터 재작성 요청 → 기존 `_workspace/` → `_workspace_prev_{타임스탬프}/`로 이동 후 Phase 1
   - **검토 라운드 진행**: workspace + spec 있음 + 사용자가 "검토 계속" 요청 → Phase 5로 직행

부분 재실행 매트릭스:
| 사용자 요청 | 호출 에이전트 |
|-----------|------------|
| 도메인 정확성 / 입시 트랙 보강 | education-domain-expert → spec-writer |
| 페르소나 / 기능 추가 | product-strategist → spec-writer |
| 특정 섹션(데이터 모델/UI 등) 수정 | spec-writer (단독) |
| QA 재검토 | spec-reviewer (단독) |

## Phase 1: 사용자 요청 명확화

사용자가 충분한 정보를 줬는지 확인. 부족하면 1~3개 질문만:

기본 질문:
1. 타깃 기기 (웹, 모바일 앱, 양쪽)?
2. 기술 스택 선호 (없으면 Next.js + Tailwind + PostgreSQL 기본값)?
3. 출시 시점 가설 (3개월 / 6개월 / 1년)?

사용자가 "알아서 해줘"라고 하면 모두 기본값으로 진행.

## Phase 2: 팀 구성

`TeamCreate`로 다음 4명 에이전트 팀 구성:
- education-domain-expert
- product-strategist
- spec-writer
- spec-reviewer

`TaskCreate`로 다음 작업 등록:
1. `01_domain_insights` (담당: education-domain-expert)
2. `02_product_blueprint` (담당: product-strategist, blockedBy: 01)
3. `03_spec_draft` (담당: spec-writer, blockedBy: 02)
4. `04_review_round_1` (담당: spec-reviewer, blockedBy: 03)
5. `05_revision` (담당: spec-writer, blockedBy: 04, 조건부)

## Phase 3: 도메인 + 프로덕트 분석 (Pipeline)

**실행 모드:** 에이전트 팀

1. **education-domain-expert**가 먼저 실행 → `_workspace/01_domain_insights.md` 산출
2. 완료되면 `SendMessage`로 product-strategist에게 통보
3. **product-strategist**가 도메인 인사이트를 받아 → `_workspace/02_product_blueprint.md` 산출
4. 도메인 모호성 발견 시 `SendMessage`로 education-domain-expert에게 질의 (양방향)

산출물 검증: 두 파일이 모두 생성되고 비어있지 않은지 확인.

## Phase 4: 스펙 작성

**실행 모드:** 에이전트 팀 (spec-writer 단독 작업이지만 팀 컨텍스트 유지)

1. **spec-writer**가 도메인 인사이트 + 청사진 + 사용자 요청을 종합
2. project-spec-writer 스킬을 활용하여 `STUDENT_HELPER_SPEC.md` 작성
3. 사본을 `_workspace/03_spec_draft.md`에 저장
4. `SendMessage`로 spec-reviewer에게 통보 (라운드 번호 포함)

크기 가이드: 800~1200줄 (Medium~Complex 복잡도).

## Phase 5: QA 검토 + 반복

**실행 모드:** 에이전트 팀

검토 루프 (최대 3 라운드):

1. **spec-reviewer**가 `STUDENT_HELPER_SPEC.md` 검토
2. `_workspace/04_review_feedback.md` 산출 (Round 1, 2, 3...)
3. 결과 분기:
   - ✅ Blocker 0 + Major 0 → 종료, 사용자 보고
   - ⚠️ Blocker 또는 Major 있음 → spec-writer에게 통보, 수정 후 다음 라운드
   - ❌ Round 3에서 동일 이슈 미해결 → 사용자 에스컬레이션
4. spec-writer가 수정 → 재검토

각 라운드 후 사용자에게 진행 상황 1줄 보고:
> "Round 1 완료: Blocker 2개, Major 5개 발견. spec-writer가 수정 중."

## Phase 6: 최종 산출물 확인

- 최종 `STUDENT_HELPER_SPEC.md` 사용자에게 안내
- `_workspace/` 디렉토리 보존 (감사 추적용)
- 사용자에게 피드백 요청:
  - 추가 보완할 영역?
  - 특정 섹션 더 자세히?
  - 만족하면 다음 단계 (예: 코드 빌드)로?

## 데이터 전달 프로토콜

**작업 디렉토리**: `/Users/robin/Downloads/student-helper/_workspace/`

**파일 컨벤션**:
- `01_domain_insights.md` — education-domain-expert
- `02_product_blueprint.md` — product-strategist
- `03_spec_draft.md` — spec-writer (사본)
- `04_review_feedback.md` — spec-reviewer (라운드별 누적)
- `_workspace_prev_{YYYYMMDD_HHMMSS}/` — 이전 실행 보존

**최종 산출물**: `/Users/robin/Downloads/student-helper/STUDENT_HELPER_SPEC.md`

**통신 방식**:
- 작업 조율: `TaskCreate` / `TaskUpdate`
- 실시간 알림: `SendMessage` (완료 / 질의 / 피드백)
- 산출물 전달: 파일 기반 (`_workspace/`)

## 에러 핸들링

| 에러 | 처리 |
|-----|-----|
| 에이전트 실패 (1차) | 1회 재시도 |
| 재시도 실패 | 사용자에게 "어느 단계에서 실패했는지" 보고 + 진행 옵션 제시 (스킵/수동/중단) |
| 도메인 정보 충돌 | education-domain-expert 의견 우선, 다른 에이전트 의견은 보고서에 병기 |
| Round 3 미해결 | 사용자 에스컬레이션, 결정 위임 |
| 사용자 요청 모호 | Phase 1로 돌아가 재명확화 |

## 테스트 시나리오

### 정상 흐름
- 입력: "초중학교 생활기록부 PDF로 고1~고3 TODO 만드는 서비스 스펙 작성"
- 기대: 4명 팀 → 4단계 파일 + 최종 spec.md (800~1200줄) + Round 1~2 후 통과

### 에러 흐름 1: 사용자 모호 요청
- 입력: "학생 서비스 만들고 싶어"
- 기대: Phase 1에서 1~3개 명확화 질문 후 진행

### 에러 흐름 2: 부분 재실행
- 입력 (이전 실행 후): "보안 섹션만 더 자세히"
- 기대: spec-writer 단독 호출, 보안 섹션만 재작성, 다른 섹션 보존

### 에러 흐름 3: 검토 무한 루프
- spec-writer가 같은 Blocker를 3 라운드 미해결
- 기대: 사용자 에스컬레이션, 자동 진행 중단

## 후속 작업 키워드 (트리거 보강)

이 스킬은 다음 표현에도 트리거되어야 한다:
- "스펙 다시 작성", "spec 재실행", "스펙 보완"
- "특정 섹션 수정", "데이터 모델 보강", "UI 추가"
- "QA 재검토", "검토 계속", "Round 다시"
- "이전 결과 기반 개선", "피드백 반영"
- "학생 도우미 스펙", "생기부 TODO 스펙", "고교 TODO 서비스"
