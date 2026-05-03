---
name: spec-writing
description: 도메인 인사이트와 프로덕트 청사진을 입력받아 project-spec-writer 스킬로 학생 도우미 서비스의 XML 빌드 스펙(STUDENT_HELPER_SPEC.md)을 작성. 기술 스택·데이터 모델·UI/UX·API·에러 처리·보안 등 모든 기술적 결정을 포함한 빌드 가능한 스펙을 산출. 학생 도우미 서비스의 최종 XML 스펙 작성이 필요할 때 반드시 이 스킬을 사용.
---

# Spec Writing

학생 도우미 서비스의 최종 산출물인 XML 스펙을 작성하는 스킬. `project-spec-writer` 스킬을 wrapper로 활용하여 한국 교육 도메인 특화 스펙을 만든다.

## 워크플로우

### 1. 입력 통합

다음 파일을 모두 읽는다:
- `_workspace/01_domain_insights.md` (필수)
- `_workspace/02_product_blueprint.md` (필수)
- `_workspace/04_review_feedback.md` (반복 라운드 시)
- 사용자 원본 요청

또한 project-spec-writer 스킬의 가이드를 참조한다:
- `~/.claude/skills/project-spec-writer/SKILL.md`
- `~/.claude/skills/project-spec-writer/references/xml-schema.md`
- `~/.claude/skills/project-spec-writer/references/example-spec.md`

### 2. 본 프로젝트 고유 결정 사항 (CRITICAL)

이 프로젝트는 다음 특성이 있으므로 spec에 반드시 반영한다:

**기술 스택 권장 기본값** (사용자 별도 지정 없으면):
- 프론트엔드: **Next.js 14 (App Router) + React 18 + TypeScript 5**
- 스타일: **Tailwind CSS v3.4** + **shadcn/ui**
- 데이터베이스: **PostgreSQL 16 + Prisma ORM v5**
- 인증: **NextAuth.js v5** (이메일 + 소셜 로그인)
- LLM: **Anthropic Claude API (claude-opus-4-7)** — 생기부 분석
- PDF 처리: **pdf-parse** (서버) + **pdfjs-dist** (클라이언트 미리보기)
- 파일 저장: **AWS S3** (또는 호환 스토리지) — 암호화 저장
- 배포: **Vercel** (프론트) + **Railway/Supabase** (DB)
- 모니터링: **Sentry**

**도메인 데이터 엔티티** (최소):
- User, StudentProfile, Record (PDF 메타), AnalysisResult, Track, TODOTemplate, TODOItem, Progress, ConsentLog

**보안/개인정보 (CRITICAL)**:
- 생활기록부 = 민감정보. 모든 처리에 명시적 동의 필요.
- PDF는 S3에 AES-256 암호화 저장. 분석 후 자동 삭제 옵션 (default: 30일).
- LLM 호출 시 PII 마스킹 (이름→[학생], 학교명→[학교A] 등).
- 사용자 삭제 요청 시 24시간 내 완전 삭제 + 로그 기록.
- 만 14세 미만은 법정대리인 동의 절차.
- LLM 응답에 학생 식별 정보가 그대로 들어가지 않도록 후처리.

**핵심 기능 흐름 (CRITICAL)**:
- TODO는 *템플릿(도메인 마스터)* + *개인화(LLM 분석 기반)* 의 조합으로 생성.
- 학년 전환은 사용자 명시 액션 (자동 변경 금지) — 사용자가 "고2로 전환" 클릭.
- 트랙 변경 시 TODO 재생성 옵션 제공.

### 3. XML 스펙 작성

project-spec-writer가 정의한 섹션 순서를 그대로 따른다:

```
<project_specification>
  <project_name>STUDENT_HELPER</project_name>
  <overview>...</overview>
  <scope_boundaries>
    <in_scope>...</in_scope>
    <out_of_scope>...</out_of_scope>
  </scope_boundaries>
  <technology_stack>...</technology_stack>
  <prerequisites>...</prerequisites>
  <environment_variables>...</environment_variables>
  <file_structure>...</file_structure>
  <core_data_entities>...</core_data_entities>
  <authentication>...</authentication>
  <route_definitions>...</route_definitions>
  <component_hierarchy>...</component_hierarchy>
  <pages_and_interfaces>...</pages_and_interfaces>
  <core_functionality>...</core_functionality>
  <error_handling>...</error_handling>
  <third_party_integrations>...</third_party_integrations>
  <aesthetic_guidelines>...</aesthetic_guidelines>
  <security_considerations>...</security_considerations>
  <advanced_functionality>...</advanced_functionality>
  <final_integration_test>...</final_integration_test>
  <success_criteria>...</success_criteria>
  <build_output>...</build_output>
  <key_implementation_notes>...</key_implementation_notes>
</project_specification>
```

### 4. 단계적 작성 (큰 스펙 처리)

500줄 초과가 예상되므로 다음 순서로 단계적 작성:

1. **개요 단계** (skeleton): project_name, overview, scope_boundaries 먼저 확정
2. **기술 결정 단계**: technology_stack, prerequisites, environment_variables, file_structure
3. **데이터 단계**: core_data_entities — 모든 필드, 타입, 제약, 관계, 인덱스 완성
4. **라우팅/구조 단계**: authentication, route_definitions, component_hierarchy
5. **UI 단계**: pages_and_interfaces — 각 페이지 레이아웃·치수·색상·인터랙션·빈상태·로딩·에러
6. **기능 단계**: core_functionality, third_party_integrations, advanced_functionality
7. **운영 단계**: error_handling, security_considerations, aesthetic_guidelines
8. **마감 단계**: final_integration_test, success_criteria, build_output, key_implementation_notes

### 5. 디자인 시스템 (기본값)

`aesthetic_guidelines`에 다음을 포함:

**컬러 팔레트** (학생용 톤, 차분 + 동기부여):
- Primary: `#2563EB` (청사진 블루) — 신뢰
- Secondary: `#7C3AED` (학습 보라) — 집중
- Accent: `#10B981` (완료 그린) — 진행/성취
- Warning: `#F59E0B` (주의 앰버) — 시급한 TODO
- Danger: `#EF4444` (위험 레드)
- Background: `#FAFAFA` (라이트), `#0F172A` (다크)
- Text: `#0F172A` (라이트), `#F1F5F9` (다크)

**타이포그래피**:
- 본문: `Pretendard Variable` (한글) → fallback `system-ui, -apple-system`
- 코드/숫자: `JetBrains Mono`
- 사이즈 스케일: 12 / 14 / 16 / 18 / 24 / 32 / 48px

**스페이싱**: 4px 베이스 — 4 / 8 / 12 / 16 / 24 / 32 / 48 / 64

**라운드**: 4 / 8 / 12 / 16 / 24px (카드 12, 버튼 8, 모달 16)

**애니메이션**:
- 페이드: 200ms ease-out
- 슬라이드: 300ms cubic-bezier(0.4, 0, 0.2, 1)
- TODO 완료 체크: 400ms spring

### 6. 최종 산출물 저장

- **메인**: `STUDENT_HELPER_SPEC.md` (사용자 워킹 디렉토리, 즉 `/Users/robin/Downloads/student-helper/`)
- **사본**: `_workspace/03_spec_draft.md` (검토용 동일 사본)

### 7. 검토자에게 알림

`SendMessage`로 spec-reviewer에게 알린다:
- "초안 완료, `STUDENT_HELPER_SPEC.md` 위치"
- 라운드 번호 (Round 1, 2, ...)

## 반복 모드 (피드백 반영)

`_workspace/04_review_feedback.md`가 존재하면:

1. 보고서 읽기 → Blocker → Major → Minor 순으로 처리
2. 변경 부분이 다른 섹션에 미치는 영향 확인 (예: 데이터 필드 추가 시 → UI에서 표시 추가, API 갱신)
3. 변경된 섹션과 영향받은 섹션을 모두 일관 갱신
4. 변경 이력을 spec 본문 끝의 주석으로 기록 (선택, 너무 길면 생략)
5. spec-reviewer에게 재검토 요청

## 작성 원칙

- **구체값**: 색상 hex, 치수 px, 라이브러리 버전 명시
- **빈 상태/로딩/에러 상태** 모든 페이지에 정의
- **CRITICAL 표시**: 아키텍처 제약은 `CRITICAL:` 접두사
- **scope_boundaries**: in_scope + out_of_scope 모두 명시
- **AI 친화**: 산문 줄이고 구조화된 리스트와 구체값
- **반응형**: mobile / tablet / desktop breakpoint 명시 (640 / 1024 / 1280)
- **접근성**: 색대비 WCAG AA, ARIA, 키보드 네비게이션
- **국제화**: 한국어 우선, i18n 구조는 v2+로 보류 가능 (scope_boundaries에 명시)

## 분량 가이드

복잡도 Medium~Complex → **800~1200줄** 목표.
- 데이터 엔티티: 8~10개
- UI 페이지: 10~12개
- 테스트 시나리오: 10~12개

500줄 초과 시 단계적 작성 (4번 워크플로우 참조).
