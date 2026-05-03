# student-helper

초중학교 생활기록부 PDF를 입력받아 고1~고3 단계별 맞춤 TODO 로드맵을 생성하는 학생용 진학 코치 서비스. 본 디렉토리에서는 해당 서비스의 **빌드 스펙 문서**(`STUDENT_HELPER_SPEC.md`)를 작성·개선한다.

## 하네스: 학생 도우미 스펙 작성

**목표:** 도메인 분석 → 프로덕트 청사진 → XML 빌드 스펙 → QA 검토를 4명 에이전트 팀으로 조율하여, AI 코딩 에이전트가 그대로 빌드 가능한 수준의 `STUDENT_HELPER_SPEC.md`를 산출한다.

**트리거:** 학생 도우미 서비스 스펙 작성·보완·검토 요청 시 `student-helper-spec-orchestrator` 스킬을 사용하라. 단순 도메인 질문(예: "학종이 뭐야?")은 직접 응답 가능.

**산출물:**
- 메인: `STUDENT_HELPER_SPEC.md`
- 작업 산출물: `_workspace/01_domain_insights.md`, `02_product_blueprint.md`, `03_spec_draft.md`, `04_review_feedback.md`

**변경 이력:**
| 날짜 | 변경 내용 | 대상 | 사유 |
|------|----------|------|------|
| 2026-05-02 | 초기 구성 (4 에이전트 + 4 스킬 + 오케스트레이터) | 전체 | 신규 프로젝트 하네스 구축 |
