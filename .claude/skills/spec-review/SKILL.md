---
name: spec-review
description: spec-writer가 작성한 STUDENT_HELPER_SPEC.md의 완결성·일관성·실현 가능성을 QA 검증. 데이터 모델↔UI↔API↔보안 정책의 경계면 교차 비교, project-spec-writer quality checklist 검사, 도메인 정확성 확인, Blocker/Major/Minor 분류 피드백 산출. 학생 도우미 스펙 문서를 검토·QA·개선 권장이 필요할 때 반드시 이 스킬을 사용.
---

# Spec Review

스펙 문서의 품질을 보장하는 QA 스킬. AI 코딩 에이전트가 막힘 없이 빌드를 시작할 수 있는지를 기준으로 검토한다. **존재 확인이 아닌 경계면 교차 비교**가 핵심이다.

## 워크플로우

### 1. 입력 확인

- `STUDENT_HELPER_SPEC.md` (필수)
- `_workspace/01_domain_insights.md`
- `_workspace/02_product_blueprint.md`
- 직전 라운드 `_workspace/04_review_feedback.md` (있으면)

### 2. 라운드 번호 결정

직전 보고서 존재 시 round 번호 +1, 없으면 Round 1.

### 3. 검증 (7개 영역)

#### A. 구조 완결성

XML 섹션 누락 점검:
- [ ] project_name, overview, scope_boundaries
- [ ] technology_stack, prerequisites, environment_variables, file_structure
- [ ] core_data_entities, authentication, route_definitions, component_hierarchy
- [ ] pages_and_interfaces, core_functionality, error_handling
- [ ] third_party_integrations, aesthetic_guidelines, security_considerations
- [ ] advanced_functionality, final_integration_test, success_criteria
- [ ] build_output, key_implementation_notes

#### B. 데이터 모델 ↔ UI 교차

각 데이터 엔티티의 모든 필드를 표로 만들고, 어떤 페이지에서 어떻게 다루는지 매핑한다:

| 엔티티.필드 | 표시 페이지 | 편집 페이지 | 빈상태/로딩/에러 |
|------------|----------|-----------|----------------|

역방향: UI에 등장하는 모든 데이터 항목이 데이터 모델에 있는가?

#### C. UI ↔ 인터랙션 교차

각 페이지에 대해:
- [ ] 호버/클릭/드래그 동작 명시
- [ ] 키보드 단축키 정의 (특히 TODO 완료 체크)
- [ ] 애니메이션 duration / easing 명시
- [ ] 반응형 breakpoint와 모바일 적응
- [ ] 빈 상태 (empty state)
- [ ] 로딩 상태
- [ ] 에러 상태
- [ ] 접근성 (ARIA, 색대비, 키보드 네비)

#### D. API ↔ 데이터 모델 교차

- [ ] 모든 데이터 작업(CRUD)이 API 또는 server action으로 정의됨
- [ ] 요청/응답 스키마와 데이터 모델 필드 일치
- [ ] 인증/인가 규칙 일관됨 (예: 본인 데이터만 조회 가능)
- [ ] 페이지네이션, 정렬, 필터 파라미터 명시

#### E. 보안 / 개인정보 (이 프로젝트 특화 — 가중치 높음)

- [ ] 생활기록부 처리 동의 흐름 명시 (가입 시 + 업로드 시)
- [ ] PDF 저장 옵션 (분석 후 즉시 삭제 / N일 보관)
- [ ] PDF 암호화 (AES-256, 키 관리)
- [ ] LLM 호출 전 PII 마스킹 (이름→[학생], 학교→[학교A])
- [ ] 사용자 삭제 요청 처리 흐름 (24시간 내 완전 삭제)
- [ ] 만 14세 미만 법정대리인 동의 절차
- [ ] 환경 변수에 비밀키 노출 없음 (env.example만 공개)
- [ ] CORS, Rate Limit 설정

#### F. 도메인 정확성

`01_domain_insights.md`와 대조:
- [ ] 한국 교육 시스템 용어 정확 (생기부 항목명, 입시 트랙 명칭)
- [ ] 고1/고2/고3 시기별 활동 매핑이 도메인 인사이트와 일치
- [ ] TODO 카테고리가 도메인 인사이트의 분류 체계와 일치
- [ ] 입시 트랙별 차별화가 spec에 반영됨

#### G. 빌드 가능성

- [ ] 색상 hex, 치수 px, 라이브러리 버전 등 구체 값
- [ ] CRITICAL 마크가 적절히 사용됨
- [ ] 외부 통합(Claude API, S3, PDF 파서)의 키 발급/사용 절차 명시
- [ ] key_implementation_notes에 권장 구현 순서 (의존성 체인 고려)
- [ ] success_criteria가 정량적 (숫자 포함)
- [ ] 테스트 시나리오가 정상+엣지+에러 흐름 모두 포함

### 4. 우선순위 분류

- **Blocker**: 없으면 빌드 불가 또는 보안 사고 위험 (예: 동의 흐름 누락, 핵심 엔티티 빠짐)
- **Major**: 일관성·완결성 문제 (예: UI에 표시되는 필드가 데이터 모델에 없음)
- **Minor**: 개선 사항 (예: 더 명확한 카피, 추가 엣지 케이스)

### 5. 보고서 작성

`_workspace/04_review_feedback.md`로 다음 형식 작성:

```markdown
# 스펙 검토 보고서 — Round {N}

## 종합 평가
- ✅ 통과 / ⚠️ 보완 필요 / ❌ 재작성 필요
- 한 줄 요약

## Blocker (즉시 수정 필요)
| # | 위치 | 문제 | 권장 수정 |
|---|-----|-----|---------|

## Major (다음 라운드 전 수정)

## Minor (개선 제안)

## 체크리스트 통과 현황
- A. 구조 완결성: 22/22
- B. 데이터 ↔ UI: 18/20 (2개 필드 UI 매핑 누락)
- C. UI 인터랙션: ...
- D. API: ...
- E. 보안 (가중): ... ← 8/10 미만이면 자동 재작성
- F. 도메인: ...
- G. 빌드 가능성: ...

## 다음 단계
- spec-writer가 수행할 작업: (구체 항목)
- 또는 사용자 확인 필요 사항: (있으면)

## 라운드 진행 상황
- Round 1: ❌ Blocker 5개, Major 8개
- Round 2: ⚠️ Blocker 0, Major 3개
- Round 3: ✅ 통과
```

### 6. spec-writer에게 통보

`SendMessage`로 spec-writer에게:
- 보고서 위치
- Blocker 개수 + 핵심 1줄 요약
- 라운드 번호

### 7. 종료 조건

다음 조건 중 하나 충족 시 검토 종료:
- ✅ 통과 (Blocker 0, Major 0)
- 사용자가 "충분하다"고 판단 (Minor만 남음)
- 동일 이슈가 Round 3 이상 미해결 → 사용자 에스컬레이션

## 작업 원칙

- **건설적**: "이게 빠짐"이 아니라 "이걸 어떻게 추가하면 좋을지" 동반.
- **우선순위 명확**: Blocker/Major/Minor를 정확히 구분.
- **재검증 의식**: 수정본을 받으면 변경 부분 + 영향받은 섹션 모두 재확인.
- **무한 루프 방지**: Round 3 이상 같은 이슈 반복 시 에스컬레이션.
- **가중치**: 본 프로젝트는 민감정보를 다루므로 보안/개인정보(E) 영역을 가중하여 평가.

## 후속 작업 모드

- **신규 검토**: Round 1, 전체 체크
- **재검토**: 직전 라운드 보고서 읽고 carry-over 이슈 + 새 이슈 통합
- **부분 검토**: 사용자가 특정 영역만 검토 요청 시 해당 영역만 (다른 영역은 보존)
