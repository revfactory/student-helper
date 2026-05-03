<project_specification>

<project_name>StudentHelper - 생기부 PDF 기반 고1~고3 단계별 진학 코치</project_name>

<overview>
StudentHelper(이하 "본 서비스")는 한국의 초·중학교 생활기록부(NEIS) PDF를 입력받아, 학생 개인의 흥미·강점·약점·트랙 가설을 자동으로 도출한 뒤, 고1~고3 3년치 학년·학기·카테고리별 맞춤 TODO 로드맵을 30초 안에 생성·관리해주는 웹 기반 진학 코치 서비스이다. 1차 타깃은 예비 고1(중3 겨울방학)과 고1~고2 재학생 본인이며, 학부모는 결제자이자 요약 리포트 소비자로서 보조 페르소나로 다룬다.

핵심 사용자 흐름은 (1) 회원가입 + 14세 미만 보호자 동의 → (2) 분리 동의 + 생기부 PDF 업로드 → (3) PDF 파싱(텍스트 우선, 실패 시 OCR 폴백) + 수동 보정 → (4) 신호 추출 → 학생 프로파일 생성 → 트랙 듀얼 추천 → (5) 룰+LLM 하이브리드로 30~80개 TODO 자동 생성 → (6) 학년×학기 격자 대시보드 + 오늘의 TODO 5개 + 주간 체크인 루프 → (7) 학부모 요약 리포트 공유의 7단계로 진행된다. 추천에는 항상 1~2문장의 근거(explanation)와 면책 문구가 노출되며, 수상·독서·봉사 등 2024 대입 미반영 항목은 "흥미 신호"로만 활용한다.

CRITICAL: 본 서비스는 다음 시스템 정책을 코드 수준으로 강제한다 — (1) 주민번호 앞6자리, 보호자 정보, 가족관계, 주소 상세, 학교폭력 조치사항, 보건 항목은 PDF에서 추출 자체를 하지 않는다. (2) 원본 PDF는 업로드 후 30일에 자동 hard delete (soft delete 금지). (3) 학생 출생연도로 entrance_year·exam_cycle을 자동 분기하며, 시드 TODO·트랙 매트릭스·진로 가설 룰은 모두 외부 JSON 파일(`src/rules/*.json`)로 관리하여 코드 배포 없이 갱신 가능해야 한다. (4) LLM은 시드 TODO의 표현·개인화만 담당하며, "무엇을 시킬지(What)"는 룰 엔진이 단독으로 결정한다(할루시네이션 통제). (5) 모든 추천 결과·LLM 출력에는 "사교육 직접 추천 금지", "합격 보장 표현 금지", "약점 표현은 다음 단계 형태로" 가드레일을 적용한다.

본 서비스의 출시 가설은 6개월 MVP이며, 모바일 우선 반응형 웹(Next.js 14 App Router)으로 구축한다. 웹만 지원(네이티브 앱 없음), 한국어 only, 일반계 고교 진학자만 MVP 범위로 한정한다.
</overview>

<scope_boundaries>
  <in_scope>
    - 학생/학부모 회원가입 + 14세 미만 보호자 동의 흐름 (NextAuth.js 기반)
    - 생기부 PDF 업로드 (단일 파일, 최대 20MB) + pdf-parse 텍스트 추출
    - 텍스트 추출 실패 시 OCR 폴백 (Claude Vision 또는 외부 OCR)
    - 항목별 수동 보정 UI (인라인 편집)
    - PII 자동 마스킹·폐기 (주민번호 앞6자리, 가족관계, 주소 상세, 학폭, 보건 항목)
    - 신호 추출 + 학생 프로파일 생성 (흥미 가설 / 강점 / 약점 / 트랙 가설)
    - 트랙 듀얼 추천 (정시/학종/교과/논술/실기 중 주력+보조 + 신뢰도 + 신호 근거)
    - 룰 엔진(시드 TODO) + LLM(개인화 표현) 하이브리드 TODO 30~80개 자동 생성
    - 학년×학기 격자 대시보드 + 오늘의 TODO 5개(카테고리 밸런스) + 결정 포인트 배너
    - TODO 상태 관리 (backlog → today → in_progress → done | snoozed | skipped)
    - 주간 체크인 (지난 주 회고 + 다음 주 5개 추천)
    - 학부모 요약 리포트 (1~2장 요약 + 공유 링크 + PDF 다운로드)
    - 개인정보 설정 (PDF 즉시 삭제, 계정 삭제, 데이터 내보내기 JSON)
    - 모든 추천에 explanation(1~2문장 근거) + 면책 문구
    - 입시 사이클 자동 분기 (2025/2026/2027/2028+ 룰셋)
    - PDF 30일 자동 삭제 배치 (매일 자정)
    - AuditLog (PDF 열람·삭제·내보내기·학부모 열람 기록, 1년 보관)
  </in_scope>
  <out_of_scope>
    - 직업계(특성화고) 트랙 — 입시 전형 자체가 다름
    - 검정고시·해외 IB/AP — 생기부 부재 또는 양식 상이
    - 학교폭력 조치사항·보건 항목 추출 — 시스템 정책으로 거부
    - 사교육 학원·강사 추천 — 공식 자원(EBS, 학교 프로그램) 우선
    - 자기소개서 작성 도우미 — 2024년부터 대입 자소서 폐지
    - 합격 보장·예측 점수·합격 시뮬레이션 (수시 6장 / 정시 3장 시뮬)
    - 카메라 스캔 입력 (모바일 카메라로 종이 생기부 촬영) — v2
    - 학교/지역 통계 비교 (익명화 통계) — v2
    - AI 챗봇 Q&A — v2
    - 진로 캠프·외부 프로그램 큐레이션 — v2
    - 모집요강 공식 API 연동 — MVP는 사용자 입력 + 캐시
    - 모의/내신 점수 수동 입력 화면 — Should Have, MVP 후순위
    - 결정 도우미 (선택과목/트랙 비교표) — Should Have
    - 푸시·이메일 알림 — Should Have (MVP는 이메일 1종만)
    - 다중 자녀 (학부모 1명이 자녀 N명 관리) — Should Have
    - 네이티브 모바일 앱 (iOS/Android) — 웹만 지원, 반응형
    - 다국어 (한국어 only)
    - 오프라인 모드, 다중 디바이스 동기화
  </out_of_scope>
  <future_considerations>
    - 카메라 스캔 입력 + 한국어 OCR 정확도 개선 (Phase 2)
    - 결정 도우미 화면 (선택과목 비교, 트랙 변경 시뮬) (Phase 2)
    - 모의/내신 점수 수동 입력 + 트랙 추천 정밀화 (Phase 2)
    - 학부모 다중 자녀 관리 + 가족 대시보드 (Phase 2)
    - 모집요강 캐시 조회 (5~10개 대학) (Phase 2)
    - 푸시 알림 + 마감 D-N 알림 (Phase 2)
    - AI 도메인 챗봇 ("학종 세특이 뭐야?") (Phase 3)
    - 학교/지역 익명 통계 비교 (Phase 3)
    - 합격 시뮬레이션 (수시 6장 / 정시 3장) — 라이선스 확보 후 (Phase 3)
    - 영문 UI (해외 거주 한국 학생 대상) (Phase 3)
  </future_considerations>
</scope_boundaries>

<technology_stack>
  <frontend_application>
    <framework>Next.js v14.2 (App Router, React Server Components, Server Actions)</framework>
    <language>TypeScript v5.4 (strict mode 활성화)</language>
    <build_tool>Next.js 내장 (Turbopack dev, Webpack prod)</build_tool>
    <styling>Tailwind CSS v3.4 + tailwindcss-animate v1.0</styling>
    <component_library>shadcn/ui (Radix UI 기반, 필요 시 컴포넌트 단위로 추가)</component_library>
    <routing>Next.js App Router (파일 기반)</routing>
    <state_management>
      - 서버 상태: TanStack Query v5 (`@tanstack/react-query`)
      - 클라 상태: React `useState`/`useReducer` 우선, 복잡 시 Zustand v4
      - 폼: React Hook Form v7 + @hookform/resolvers/zod
    </state_management>
    <font>Pretendard Variable v1.3 (CDN 또는 self-host)</font>
  </frontend_application>
  <backend>
    <runtime>Node.js v20.x LTS</runtime>
    <framework>Next.js Route Handlers + Server Actions (별도 서버 없음)</framework>
    <auth>NextAuth.js v5 (Auth.js) — 카카오·구글 OAuth + 이메일+OTP</auth>
    <api_style>Server Actions (mutations) + REST Route Handlers (file upload, webhooks)</api_style>
    <validation>Zod v3.23 — 모든 Server Action / Route Handler 입력 스키마</validation>
    <background_jobs>BullMQ v5 + Upstash Redis (PDF 파싱, LLM 호출, 30일 자동삭제)</background_jobs>
  </backend>
  <data_layer>
    <database>PostgreSQL v15 (Neon Serverless 또는 Supabase)</database>
    <orm>Prisma v5.15 + Prisma Client (Edge runtime 호환)</orm>
    <object_storage>S3 호환 (AWS S3 또는 Cloudflare R2) — 원본 PDF 암호화 저장</object_storage>
    <encryption>서버측 AES-256 (storage 측 + 필드 암호화는 application layer에서 `node:crypto`)</encryption>
  </data_layer>
  <ai_layer>
    <llm>Anthropic Claude API — 1차 `claude-haiku-4-5` (개인화·회고 등 다회 호출), 2차 `claude-sonnet-4-7` (signal extraction 1회). 환경변수 `LLM_PERSONALIZE_MODEL`/`LLM_SIGNAL_MODEL`로 갱신. 배포 전 Anthropic 공식 ID 재확인.</llm>
    <sdk>@anthropic-ai/sdk v0.27</sdk>
    <pdf_parsing>pdf-parse v1.1.1 (서버측 텍스트 추출), pdfjs-dist v4.x (클라 미리보기)</pdf_parsing>
    <ocr_fallback>Claude Vision (PDF 페이지를 PNG로 렌더 후 multimodal API 호출). 외부 OCR 옵션: Google Cloud Vision (KOREAN_DOCUMENT_TEXT_DETECTION)</ocr_fallback>
    <korean_nlp>형태소 분석은 LLM 단일 호출로 통합 처리. 별도 koNLPy 등 사용 안 함 (Node 환경 단순화)</korean_nlp>
  </ai_layer>
  <infrastructure>
    <hosting>Vercel (Next.js 앱) — Edge Functions + Serverless Functions</hosting>
    <db_hosting>Neon Serverless Postgres (또는 Supabase 무료 티어)</db_hosting>
    <queue_redis>Upstash Redis (BullMQ 백엔드)</queue_redis>
    <object_storage_provider>Cloudflare R2 (egress free) 또는 AWS S3</object_storage_provider>
    <email>Resend v3 — 회원가입 OTP, 보호자 동의 링크, 학부모 공유 알림</email>
    <observability>Sentry v8 (에러), PostHog v1 (제품 분석), Vercel Analytics (웹 vitals)</observability>
  </infrastructure>
  <libraries>
    <ui>shadcn/ui (Radix UI Primitives 기반, 컴포넌트 단위 설치)</ui>
    <icons>lucide-react v0.395 (모든 아이콘 통일, 24px 기본)</icons>
    <date>date-fns v3.6 + date-fns-tz v3.1 (Asia/Seoul 고정)</date>
    <pdf_render>pdfjs-dist v4.4 (수동 보정 화면 PDF 미리보기)</pdf_render>
    <pdf_export>jspdf v2.5 + html2canvas v1.4 (학부모 리포트 PDF 다운로드)</pdf_export>
    <toast>sonner v1.5 (토스트 알림)</toast>
    <markdown>react-markdown v9 (TODO description 렌더링, GFM 지원)</markdown>
    <chart>recharts v2.12 (학부모 리포트 약식 차트)</chart>
    <crypto>본 서비스는 외부 라이브러리 없이 Node `crypto` 모듈 사용 (AES-256-GCM)</crypto>
    <queue>bullmq v5.10</queue>
    <rate_limit>@upstash/ratelimit v1.2 (API 레이트 리밋)</rate_limit>
  </libraries>
  <build_output>
    <build_command>pnpm build (= next build)</build_command>
    <output_dir>.next/</output_dir>
    <node_version>20.x</node_version>
    <package_manager>pnpm v9.x (lockfile 필수)</package_manager>
    <note>Edge runtime은 인증 미들웨어와 정적 라우트만, DB·LLM·BullMQ는 Node runtime.</note>
  </build_output>
</technology_stack>

<prerequisites>
  <environment_setup>
    - Node.js 20.x LTS 설치
    - pnpm 9.x 설치 (`npm i -g pnpm`)
    - PostgreSQL 15+ (로컬 또는 Neon dev branch)
    - Redis 7+ (로컬 또는 Upstash)
    - Anthropic API key (claude-haiku-4-5 + claude-sonnet-4-7 접근 권한, 배포 전 공식 ID 재확인)
    - S3 호환 스토리지 (AWS S3 버킷 또는 Cloudflare R2 버킷)
    - Resend API key (이메일)
    - 카카오/구글 OAuth 앱 (선택, 이메일+OTP만으로 시작 가능)
  </environment_setup>
  <build_configuration>
    - `next.config.js`: experimental.serverActions.bodySizeLimit = '25mb' (PDF 업로드 대응)
    - `tsconfig.json`: strict, noUncheckedIndexedAccess, paths(`@/*` → `src/*`)
    - `tailwind.config.ts`: content 경로 + custom theme + tailwindcss-animate plugin
    - `prisma/schema.prisma`: provider postgresql, previewFeatures 비활성화 (MVP에서 full-text search 미사용. 필요 시 v2에서 추가)
    - `.env.example`: 모든 변수 example value 포함, 실제 secret은 .env.local에만
  </build_configuration>
</prerequisites>

<environment_variables>
  <variable>
    <name>DATABASE_URL</name>
    <description>PostgreSQL 연결 문자열 (Prisma 사용)</description>
    <required>true</required>
    <example>postgresql://user:pass@ep-xxx.neon.tech/student_helper?sslmode=require</example>
  </variable>
  <variable>
    <name>DIRECT_URL</name>
    <description>Prisma 마이그레이션용 직접 연결 (Neon pooler 우회)</description>
    <required>true</required>
    <example>postgresql://user:pass@ep-xxx.neon.tech/student_helper</example>
  </variable>
  <variable>
    <name>REDIS_URL</name>
    <description>Upstash Redis 연결 문자열 (BullMQ 큐 백엔드)</description>
    <required>true</required>
    <example>rediss://default:xxx@steady-xxx.upstash.io:6379</example>
  </variable>
  <variable>
    <name>NEXTAUTH_SECRET</name>
    <description>NextAuth.js 세션 암호화 키 (`openssl rand -base64 32`)</description>
    <required>true</required>
    <example>QyZ3...rand32...==</example>
  </variable>
  <variable>
    <name>NEXTAUTH_URL</name>
    <description>NextAuth 콜백 URL (배포 도메인)</description>
    <required>true</required>
    <example>https://student-helper.vercel.app</example>
  </variable>
  <variable>
    <name>KAKAO_CLIENT_ID</name>
    <description>카카오 OAuth 앱 REST API 키</description>
    <required>false</required>
    <note>이메일+OTP만 사용 시 생략 가능</note>
  </variable>
  <variable>
    <name>KAKAO_CLIENT_SECRET</name>
    <description>카카오 OAuth Client Secret</description>
    <required>false</required>
  </variable>
  <variable>
    <name>GOOGLE_CLIENT_ID</name>
    <description>Google OAuth 클라이언트 ID</description>
    <required>false</required>
  </variable>
  <variable>
    <name>GOOGLE_CLIENT_SECRET</name>
    <description>Google OAuth 클라이언트 Secret</description>
    <required>false</required>
  </variable>
  <variable>
    <name>ANTHROPIC_API_KEY</name>
    <description>Claude API 키 (server-only). claude-haiku-4-5 + claude-sonnet-4-7 접근 가능해야 함 (배포 전 공식 ID 재확인)</description>
    <required>true</required>
    <example>sk-ant-api03-xxx</example>
  </variable>
  <variable>
    <name>S3_ENDPOINT</name>
    <description>S3 호환 스토리지 엔드포인트 (R2의 경우 https://&lt;account&gt;.r2.cloudflarestorage.com)</description>
    <required>true</required>
    <example>https://abc123.r2.cloudflarestorage.com</example>
  </variable>
  <variable>
    <name>S3_BUCKET</name>
    <description>원본 PDF 저장 버킷 이름</description>
    <required>true</required>
    <example>student-helper-pdfs-prod</example>
  </variable>
  <variable>
    <name>S3_ACCESS_KEY_ID</name>
    <description>S3 액세스 키 (server-only)</description>
    <required>true</required>
  </variable>
  <variable>
    <name>S3_SECRET_ACCESS_KEY</name>
    <description>S3 시크릿 키 (server-only)</description>
    <required>true</required>
  </variable>
  <variable>
    <name>PDF_ENCRYPTION_KEY</name>
    <description>업로드된 PDF의 application-layer AES-256-GCM 암호화 키 (32바이트 base64). 키 로테이션 정책 별도</description>
    <required>true</required>
    <example>base64-encoded-32-bytes</example>
  </variable>
  <variable>
    <name>RESEND_API_KEY</name>
    <description>Resend 이메일 API 키 (회원가입 OTP, 보호자 동의 링크)</description>
    <required>true</required>
    <example>re_xxx</example>
  </variable>
  <variable>
    <name>RESEND_FROM_EMAIL</name>
    <description>발신 이메일 주소 (검증된 도메인)</description>
    <required>true</required>
    <example>noreply@student-helper.app</example>
  </variable>
  <variable>
    <name>CRON_SECRET</name>
    <description>Vercel Cron 인증 헤더 (X-Cron-Secret) 검증용 비밀</description>
    <required>true</required>
    <example>random-32-bytes-base64</example>
  </variable>
  <variable>
    <name>SENTRY_DSN</name>
    <description>Sentry 프로젝트 DSN (에러 추적)</description>
    <required>false</required>
  </variable>
  <variable>
    <name>NEXT_PUBLIC_POSTHOG_KEY</name>
    <description>PostHog 프로젝트 키 (client-safe)</description>
    <required>false</required>
  </variable>
  <variable>
    <name>NEXT_PUBLIC_APP_URL</name>
    <description>공개 앱 URL (학부모 공유 링크 생성용, client-safe)</description>
    <required>true</required>
    <example>https://student-helper.app</example>
  </variable>
  <variable>
    <name>LLM_PERSONALIZE_MODEL</name>
    <description>TODO 개인화·주간 회고 모델 ID. 미설정 시 `claude-haiku-4-5` 사용</description>
    <required>false</required>
    <example>claude-haiku-4-5</example>
  </variable>
  <variable>
    <name>LLM_SIGNAL_MODEL</name>
    <description>signal extraction 모델 ID. 미설정 시 `claude-sonnet-4-7` 사용</description>
    <required>false</required>
    <example>claude-sonnet-4-7</example>
  </variable>
  <variable>
    <name>OCR_PROVIDER</name>
    <description>OCR 폴백 제공자: "claude_vision" | "google_vision". 기본 claude_vision</description>
    <required>false</required>
    <example>claude_vision</example>
  </variable>
  <variable>
    <name>GOOGLE_VISION_KEY_JSON</name>
    <description>Google Cloud Vision 서비스 계정 JSON (OCR_PROVIDER=google_vision일 때)</description>
    <required>false</required>
  </variable>
</environment_variables>

<file_structure>
src/
├── app/
│   ├── layout.tsx                       # Root layout (Pretendard 폰트, providers)
│   ├── page.tsx                         # 랜딩 (`/`)
│   ├── globals.css                      # Tailwind + CSS 변수 (theme)
│   ├── (auth)/
│   │   ├── auth/
│   │   │   ├── signup/page.tsx          # 회원가입
│   │   │   ├── login/page.tsx           # 로그인
│   │   │   └── verify-otp/page.tsx      # 이메일 OTP 검증
│   │   └── parent-consent/
│   │       └── [token]/page.tsx         # 보호자 동의 링크 (외부 진입)
│   ├── (onboarding)/
│   │   └── onboarding/
│   │       ├── role/page.tsx            # 역할/출생연도/학교유형/지역
│   │       ├── consent/page.tsx         # 분리 동의
│   │       ├── upload/page.tsx          # PDF 업로드 + 진행 상태
│   │       ├── correct/page.tsx         # 수동 보정
│   │       └── track/page.tsx           # 트랙 추천 결과
│   ├── (app)/
│   │   ├── layout.tsx                   # 인증 가드 + 사이드바/하단탭
│   │   ├── dashboard/page.tsx           # 메인 대시보드 (격자)
│   │   ├── today/page.tsx               # 오늘의 TODO 5
│   │   ├── roadmap/
│   │   │   └── [grade]/[term]/page.tsx  # 학기 상세
│   │   ├── todo/[id]/page.tsx           # TODO 상세
│   │   ├── checkin/page.tsx             # 주간 체크인
│   │   ├── profile/page.tsx             # 학생 프로파일
│   │   └── settings/
│   │       ├── account/page.tsx
│   │       └── privacy/page.tsx
│   ├── (parent)/
│   │   └── parent/
│   │       └── report/
│   │           └── [studentId]/page.tsx # 학부모 요약 리포트
│   ├── share/
│   │   └── [token]/page.tsx             # 공유 링크(읽기 전용)
│   └── api/
│       ├── auth/[...nextauth]/route.ts  # NextAuth handler
│       ├── upload/pdf/route.ts          # PDF 업로드 (multipart)
│       ├── jobs/cleanup-pdfs/route.ts   # cron: 30일 자동삭제
│       ├── share/[token]/route.ts       # 공유 링크 검증
│       └── health/route.ts              # 헬스체크
├── components/
│   ├── ui/                              # shadcn/ui 컴포넌트 (button, dialog, …)
│   ├── layout/
│   │   ├── BottomTabs.tsx               # 모바일 하단탭 (4개)
│   │   ├── Sidebar.tsx                  # 데스크톱 사이드바
│   │   └── DisclaimerBar.tsx            # 면책 배너
│   ├── auth/
│   │   ├── ConsentCheckbox.tsx
│   │   └── ParentConsentForm.tsx
│   ├── onboarding/
│   │   ├── PdfDropzone.tsx
│   │   ├── ParseProgress.tsx
│   │   └── CorrectionEditor.tsx
│   ├── dashboard/
│   │   ├── GradeTermGrid.tsx            # 학년×학기 격자
│   │   ├── DecisionPointBanner.tsx
│   │   └── TodayTodoPanel.tsx
│   ├── todo/
│   │   ├── TodoCard.tsx
│   │   ├── TodoFilters.tsx
│   │   ├── TodoStateActions.tsx
│   │   └── ExplanationTooltip.tsx
│   ├── track/
│   │   └── TrackDualCard.tsx
│   ├── parent/
│   │   ├── ReportSummary.tsx
│   │   └── ShareLinkButton.tsx
│   └── shared/
│       ├── EmptyState.tsx
│       ├── ErrorBoundary.tsx
│       └── LoadingSkeleton.tsx
├── server/
│   ├── auth.ts                          # NextAuth 구성
│   ├── db.ts                            # Prisma client singleton
│   ├── actions/
│   │   ├── consent.ts
│   │   ├── upload.ts
│   │   ├── correction.ts
│   │   ├── todo.ts
│   │   ├── checkin.ts
│   │   ├── parent.ts
│   │   └── privacy.ts
│   ├── services/
│   │   ├── pdf/
│   │   │   ├── parse.ts                 # pdf-parse 텍스트 추출
│   │   │   ├── ocr.ts                   # Claude Vision / Google Vision
│   │   │   ├── anchors.ts               # 항목 앵커 키워드 매칭
│   │   │   ├── mask.ts                  # PII 마스킹
│   │   │   └── pipeline.ts              # 단계별 오케스트레이션
│   │   ├── signal/
│   │   │   ├── extract.ts
│   │   │   ├── interest.ts
│   │   │   ├── strengths.ts
│   │   │   └── tracks.ts                # §3.2 매트릭스
│   │   ├── recommendation/
│   │   │   ├── seedLoader.ts            # rules/*.json 로드
│   │   │   ├── personalize.ts           # LLM 호출
│   │   │   ├── priority.ts              # 우선순위 계산
│   │   │   └── balance.ts               # 오늘 5개 밸런스 룰
│   │   ├── llm/
│   │   │   ├── client.ts                # Anthropic SDK wrapper + 캐싱
│   │   │   ├── prompts/
│   │   │   │   ├── personalize.ts
│   │   │   │   ├── interest.ts
│   │   │   │   └── strength.ts
│   │   │   └── guardrails.ts            # 금칙어/톤 가드
│   │   ├── storage/
│   │   │   ├── s3.ts
│   │   │   └── encryption.ts            # AES-256-GCM
│   │   ├── audit/log.ts
│   │   └── parent/share.ts              # 공유 토큰 발급
│   ├── jobs/
│   │   ├── queue.ts                     # BullMQ 큐 정의
│   │   ├── workers/
│   │   │   ├── parsePdf.ts
│   │   │   ├── generateRoadmap.ts
│   │   │   └── deleteExpiredPdfs.ts
│   │   └── scheduler.ts
│   └── lib/
│       ├── examCycle.ts                 # birth_year → entrance_year → exam_cycle
│       ├── academicCalendar.ts          # 학기 시기 헬퍼
│       └── disclaimer.ts                # 면책 텍스트 상수
├── rules/
│   ├── seed_todos.2025.json             # 2025 입시 사이클 시드
│   ├── seed_todos.2026.json
│   ├── seed_todos.2027.json
│   ├── seed_todos.2028.json             # 2028 개편 분기
│   ├── track_matrix.json
│   ├── career_hypothesis.json
│   ├── disclaimer.ko.json
│   └── tone_guard.json
├── lib/
│   ├── utils.ts                         # cn() 등
│   └── env.ts                           # zod 기반 env 검증
├── types/
│   ├── domain.ts                        # 카테고리 enum 등
│   └── prisma.ts
├── styles/
│   └── tailwind-tokens.ts
├── middleware.ts                        # 인증 가드
└── prisma/
    ├── schema.prisma
    └── migrations/
public/
├── fonts/                               # Pretendard self-host
├── illustrations/                       # 빈상태/에러 SVG
└── favicon.ico
.env.example
.env.local                               # 로컬 비밀 (gitignore)
next.config.js
tailwind.config.ts
postcss.config.js
tsconfig.json
package.json
pnpm-lock.yaml
README.md
</file_structure>

<core_data_entities>

  <user>
    엔티티: User (계정. 학생 본인 또는 학부모)
    - id: string (uuid, primary key)
    - email: string (unique, lowercased, 254자 max)
    - emailVerifiedAt: Date | null
    - passwordHash: string | null (이메일+OTP 사용 시 null 가능)
    - role: enum ("student", "parent")
    - birthYear: number (4자리, 1980~2020 범위 검증)
    - locale: string (default "ko", 향후 확장 대비)
    - createdAt: Date
    - updatedAt: Date
    - deletedAt: Date | null (soft delete 30일 유예 후 hard delete cron)
    - oauthProvider: enum ("kakao", "google", null)
    - oauthSubject: string | null
    Indexes: [email], [oauthProvider+oauthSubject], [deletedAt]
  </user>

  <student>
    엔티티: Student (분석 대상 학생. User와 1:1 또는 1:N(학부모))
    - id: string (uuid, pk)
    - ownerUserId: string (fk User.id, owner는 가입한 본인 또는 학부모)
    - displayName: string (max 30자, 학생 본명 X — 별명 권장 안내)
    - birthYear: number (학생 본인 출생연도)
    - schoolType: enum ("general", "specialized_self_directed", "vocational", "other") — 일반/자사·특목/직업/기타
    - schoolLevel: enum ("elementary", "middle", "high")
    - currentGrade: number (-1=중학생, 1|2|3=고등학년)
    - currentTerm: enum ("1H1", "1Hsummer", "1H2", "1Hwinter", "2H1", "2Hsummer", "2H2", "2Hwinter", "3H1", "3Hsummer", "3H2") — 학년×학기 코드
    - region: string (시도 명, 17개 enum: "서울","부산",...)
    - entranceYear: number (자동 계산: 고1 입학년도)
    - examCycle: enum ("CYCLE_2025", "CYCLE_2026", "CYCLE_2027", "CYCLE_2028") — 자동 분기. Prisma `enum ExamCycle`(2065줄)와 1:1 일치. 룰셋 파일명 매핑은 examCycle.ts의 `examCycleToRuleFileSuffix(cycle: ExamCycle): "2025"|"2026"|"2027"|"2028"` 헬퍼로 변환 (예: CYCLE_2028 → "2028" → `rules/seed_todos.2028.json` 로드). 2028년 이후(2029+) 학생은 cycle 갱신 룰 추가 시까지 CYCLE_2028로 매핑.
    - trackPreference: enum ("undecided", "jeongsi", "hakjong", "gyogwa", "nonsul", "silgi") — 사용자 선호
    - createdAt: Date
    - updatedAt: Date
    Indexes: [ownerUserId], [examCycle+currentGrade]
    CRITICAL: ownerUserId가 학부모 role인 경우, ParentChildLink가 함께 생성되어야 한다.
  </student>

  <parent_child_link>
    엔티티: ParentChildLink (학부모-자녀 관계 + 동의 상태)
    - id: string (uuid, pk)
    - parentUserId: string (fk User.id, role=parent)
    - studentId: string (fk Student.id)
    - consentStatus: enum ("pending", "granted", "granted_by_self", "revoked")
    - consentToken: string | null (보호자 동의 링크 토큰, 7일 유효)
    - grantedAt: Date | null
    - revokedAt: Date | null
    - createdAt: Date
    Indexes: [parentUserId+studentId unique], [consentToken]
  </parent_child_link>

  <pdf_file>
    엔티티: PdfFile (원본 PDF 메타. 30일 후 자동 hard delete)
    - id: string (uuid, pk)
    - studentId: string (fk Student.id)
    - storageKey: string (S3 key, 예 `pdfs/{studentId}/{uuid}.pdf.enc`)
    - sha256Hash: string (64자 hex, 중복 업로드 방지)
    - fileSize: number (bytes, max 20MB)
    - encryptionIv: string (base64, AES-GCM IV)
    - encryptionTag: string (base64, AES-GCM auth tag)
    - uploadedAt: Date
    - expiresAt: Date (uploadedAt + 30일. 매일 자정 cron이 hard delete)
    - retentionConsent: boolean (사용자가 30일 보관 동의했는지)
    - ocrUsed: boolean (텍스트 추출 실패로 OCR 사용했는지)
    - parseStatus: enum ("queued", "parsing_text", "parsing_ocr", "parsed", "failed", "manual_correction", "manual_input")
    - parseError: string | null (실패 사유)
    Indexes: [studentId+uploadedAt desc], [expiresAt asc] (cron 인덱스)
    CRITICAL: retentionConsent=false면 expiresAt = uploadedAt + 24시간으로 단축.
    상태 전이 규칙 (CRITICAL: services/pdf/pipeline.ts에서 강제):
      ```
      queued ──(워커 픽업)──> parsing_text
      parsing_text ──(텍스트 추출 성공 + 한글≥30% or 앵커키워드 1개+)──> parsed
      parsing_text ──(텍스트 부적합 감지)──> parsing_ocr
      parsing_ocr ──(OCR 페이지별 호출 성공)──> parsed
      parsing_ocr ──(OCR 30페이지 초과/실패)──> failed
      parsed ──(사용자 /onboarding/correct에서 편집 시작)──> manual_correction
      manual_correction ──(사용자 [저장] 또는 [건너뛰기])──> parsed
      failed ──(사용자 모달에서 [수동 입력 모드])──> manual_input
      failed ──(사용자 [재시도] 또는 [다른 PDF])──> queued (새 PdfFile row)
      manual_input ──(사용자 입력 저장)──> parsed
      ```
      - 진행 화면 폴링은 status를 직접 노출하지 않고, 5단계 표시 매핑: queued|parsing_text=업로드+텍스트추출, parsing_ocr=OCR(노란 강조), parsed=PII마스킹+신호분석+로드맵생성 진행, failed=에러 모달, manual_correction/manual_input=수동 모드 화면 이동.
      - parseStatus는 단조 증가 X (manual_correction ↔ parsed 가능). 모든 전이는 AuditLog에 별도 기록 안 함 (PII 액션 아님), 단 failed 발생 시 Sentry 캡처.
  </pdf_file>

  <parsed_report_card>
    엔티티: ParsedReportCard (생기부 파싱 결과. PDF 삭제 후에도 유지. PII 마스킹 후 저장)
    - id: string (uuid, pk)
    - studentId: string (fk Student.id)
    - sourcePdfId: string | null (fk PdfFile.id, PDF 삭제 시 null로 SET NULL)
    - schoolLevel: enum ("elementary", "middle")
    - grade: number (1~6)
    - term: enum ("1H", "2H") — 초/중은 학기 단위
    - schoolYear: number (서기, 예 2024)
    - sections: Json (JSONB) — 9개 키 필수, 미존재 시 null
        attendance: { absentExcused: number, absentUnexcused: number, lateUnexcused: number, earlyLeaveUnexcused: number, sickoutCount: number }
        awards: { categories: string[] (예 ["과학탐구","글짓기"]), totalCount: number }
        creativeActivities: { autonomy: string, club: string, volunteer: string, career: string, clubCategoryTags: string[] }
        subjectGrades: Array&lt;{ subject: string, achievement: enum("A","B","C","D","E"), score?: number }&gt;
        subjectRecords: Array&lt;{ subject: string, text: string (≤2000자), keywords: string[] }&gt;
        freeSemesterOutputs: Array&lt;{ topic: string, type: enum("진로","주제선택","예체능","동아리"), text: string }&gt; (중1만)
        reading: { categories: { humanities: number, science: number, social: number, art: number, other: number }, totalCount: number }
        comprehensiveOpinion: { text: string (≤2000자), positiveKeywords: string[], improvementKeywords: string[] }
        meta: { schoolNameMasked: boolean, addressLevel: "sido_only" }
    - extractionConfidence: number (0~100, 항목별 평균)
    - manualCorrected: boolean
    - createdAt: Date
    - updatedAt: Date
    Indexes: [studentId+schoolLevel+grade+term unique], [studentId+createdAt desc]
    CRITICAL: 다음 항목은 sections에 절대 포함되지 않는다 — 주민번호 앞6자리, 보호자 정보, 가족관계, 주소 상세, 학교폭력 조치사항, 보건·신체발달, 출결 사유 텍스트.
    Row 단위 정책 (CRITICAL):
      - 1 ParsedReportCard row = (studentId, schoolLevel, grade, term) unique. 즉 학생당 학교급별·학년별·학기별 각 1행. 하나의 PDF가 다년치를 포함하면 N개 row 동시 생성.
      - 자유학기(freeSemesterOutputs)는 schoolLevel="middle" + grade=1 + term="1H" row의 sections.freeSemesterOutputs에만 채워짐. 다른 row에서는 항상 null.
      - subjectGrades/subjectRecords/creativeActivities는 해당 row의 학기 데이터만 포함.
      - awards/reading는 학기 단위 누적이 어려운 경우 학년 단위로 1H row에 통합 저장 가능 (meta.awardsAggregation: "per_grade").
    신호 집계 흐름 (services/signal/extract.ts):
      - 호출 시 `prisma.parsedReportCard.findMany({ where:{studentId}, orderBy:[{schoolLevel:'asc'},{grade:'asc'},{term:'asc'}] })`로 모든 row 로드.
      - clubConsistency 계산 의사코드:
          ```
          let series = rows.map(r => r.sections.creativeActivities?.clubCategoryTags ?? [])
          let pairs = 0, total = max(1, series.length - 1)
          for i in 1..series.length-1:
            if series[i] ∩ series[i-1] not empty: pairs += 1
          clubConsistency = pairs / total   // 0~1
          ```
      - gradeTrend 계산: subjectGrades.achievement (A=1..E=5)를 학기별 평균낸 뒤 단순 선형 회귀 기울기 (음수면 상승 추세).
      - freeSemesterStrength: middle/grade=1/term=1H row의 freeSemesterOutputs 갯수 × confidence 평균 (0~100).
  </parsed_report_card>

  <student_profile>
    엔티티: StudentProfile (신호 추출 결과. 버전 누적)
    - id: string (uuid, pk)
    - studentId: string (fk Student.id)
    - version: number (1부터 증가)
    - interestHypothesis: Json — Array&lt;{ topic: string, confidence: number 0~1, evidence: string[] (signal_id 리스트), category: enum("ai_sw","humanities_social","natural_science","arts_athletics","education","health_medical","business","undefined") }&gt;
    - strengths: string[] (≤10개, 각 ≤80자)
    - weaknesses: string[] (≤10개, 각 ≤80자, 톤가드 적용 — "다음 단계" 형태)
    - trackHypothesis: Json — { primary: enum trackEnum, secondary: enum trackEnum | null, confidenceLabel: enum("high","mid","low"), reasoningSignals: string[] }
    - signalSummary: Json — 정량 점수 모음 (gradeAvgMain, attendanceClean, freeSemesterStrength, clubConsistency 등)
    - generatedAt: Date
    - generatedBy: enum ("rule_engine", "rule_plus_llm")
    Indexes: [studentId+version unique], [studentId+generatedAt desc]
  </student_profile>

  <track_recommendation>
    엔티티: TrackRecommendation (트랙 듀얼 추천 결과 이력)
    - id: string (uuid, pk)
    - studentId: string (fk Student.id)
    - studentProfileId: string (fk StudentProfile.id)
    - primaryTrack: enum ("jeongsi", "hakjong", "gyogwa", "nonsul", "silgi", "undecided")
    - secondaryTrack: enum (위와 동일, primary와 다른 값 또는 null)
    - confidenceLabel: enum ("high", "mid", "low")
    - signalSummary: Json — Array&lt;{ key: string, value: string, weight: number, label: string (한글) }&gt;
    - userOverrideTrack: enum | null (사용자가 수동 변경한 경우)
    - userOverrideReason: string | null (≤200자)
    - generatedAt: Date
    Indexes: [studentId+generatedAt desc]
  </track_recommendation>

  <todo>
    엔티티: Todo (단일 할 일)
    - id: string (uuid, pk)
    - studentId: string (fk Student.id)
    - category: enum (22개) — "A1","A2","A3","A4","A5","B1","B2","B3","B4","B5","B6","B7","C1","C2","C3","C4","C5","C6","C7","D1","D2","D3","D4","D5","D6","D7","E1","E2","E3","E4","E5"
    - grade: number (1|2|3)
    - term: enum (11종 풀 나열) "1H1", "1Hsummer", "1H2", "1Hwinter", "2H1", "2Hsummer", "2H2", "2Hwinter", "3H1", "3Hsummer", "3H2"  // "3Hwinter" 없음 — 고3 11~12월은 수능·정시 마감 시기로 별도 학기 코드 불필요
    - title: string (max 80자, LLM 개인화)
    - description: string (max 1000자, markdown 허용)
    - explanation: string (1~2문장, max 200자, "왜 추천했는지")
    - priorityScore: number (0~100, 가중치 공식)
    - state: enum ("backlog","today","in_progress","done","snoozed","skipped")
    - dueDate: Date | null
    - completedAt: Date | null
    - snoozeUntil: Date | null
    - skipReason: enum | null ("already_done","not_interested","wrong_timing","other")
    - source: enum ("rule_seed","rule_plus_llm","user_added") — MVP는 user_added 미사용
    - sourceRuleId: string | null (rules/*.json의 시드 ID)
    - track: enum trackEnum | null (이 TODO가 어느 트랙에 속하는지, null=공통)
    - tags: string[] (max 5개, 각 ≤20자) — 시드 JSON `seed_todos.{cycle}.json`의 `tags` 필드에서 채워짐. UI 노출: 로드맵 학기 상세 카드 우측 상단 1~2개 표시(나머지는 +N), TODO 상세 페이지 본문 위. 검색·필터 v2(MVP는 표시 전용). 사용자 수정 불가(읽기 전용)
    - createdAt: Date
    - updatedAt: Date
    Indexes: [studentId+state+priorityScore desc], [studentId+grade+term], [studentId+dueDate asc]
    상태 전이 규칙:
      - backlog → today (사용자 또는 자동 밸런스 룰)
      - today → in_progress (사용자 "시작" 클릭)
      - in_progress → done | snoozed
      - any → skipped (사유 필수)
      - snoozed → backlog (snoozeUntil 도래 시 자동 복귀)
  </todo>

  <weekly_check_in>
    엔티티: WeeklyCheckIn (주간 회고)
    - id: string (uuid, pk)
    - studentId: string (fk Student.id)
    - weekStart: Date (월요일 00:00 KST)
    - weekEnd: Date (일요일 23:59 KST)
    - completedCount: number
    - missedCount: number
    - snoozedCount: number
    - skippedCount: number
    - retroText: string (max 200자)
    - satisfactionScore: number (1~5)
    - generatedAt: Date
    Indexes: [studentId+weekStart unique]
  </weekly_check_in>

  <share_link>
    엔티티: ShareLink (학부모/제3자에게 발급한 읽기 전용 공유 토큰)
    - id: string (uuid, pk)
    - studentId: string (fk Student.id)
    - createdByUserId: string (fk User.id)
    - token: string (32자 url-safe random, unique)
    - audience: enum ("parent_summary", "student_profile_readonly")
    - includeTodoBody: boolean (default false) — 학생이 ShareLink 생성 시 별도 토글로 결정. false면 TODO는 제목만 노출(본문 마스킹), true면 description까지 노출.
    - includeSchoolType: boolean (default false) — false면 학교 유형 비노출.
    - includeRegion: boolean (default false) — false면 시도 비노출.
    - expiresAt: Date (default createdAt + 30일)
    - revokedAt: Date | null
    - viewCount: number (default 0)
    - lastViewedAt: Date | null
    Indexes: [token unique], [studentId+audience]
    비로그인 PII 노출 최소화 정책 (CRITICAL):
      - /share/[token] 페이지는 항상 노출: 학생 displayName(별명, 본명 X), 학년 단계, 입시 사이클 라벨.
      - includeTodoBody=false (기본값)일 때: TODO 제목만, description/explanation 마스킹.
      - includeSchoolType=false일 때: 학교 유형 비노출. includeRegion=false일 때: 시도 비노출.
      - IP rate limit: 동일 IP에서 동일 token 60회/시간 초과 시 410 Gone + 로그 (남용 차단).
      - viewCount가 100을 초과하면 학생에게 자동 알림 이메일 ("공유 링크가 100회 이상 열람됐어요. 안전한가요?") + 검토 권유.
      - 토큰 유출 의심 시 학생이 /settings/privacy에서 즉시 [취소] 가능 (test_scenario_5 참조).
  </share_link>

  <audit_log>
    엔티티: AuditLog (PII·민감 액션 감사 로그. 보존 1년)
    - id: string (uuid, pk)
    - actorType: enum ("user", "system", "anonymous") — system은 cron/worker, anonymous는 비로그인 공유 링크 열람
    - actorUserId: string | null (fk User.id; actorType="system"|"anonymous"일 때 null)
    - targetStudentId: string (fk Student.id)
    - action: enum (
        "view_pdf","delete_pdf","upload_pdf",
        "view_profile","export_data",
        "parent_view_summary","share_link_create","share_link_view","share_link_revoke",
        "consent_grant","consent_revoke","consent_renewal_required",
        "track_override","todo_skip",
        "account_delete_requested","account_delete_completed",
        "schooltype_waitlist_register"
      )
    - ip: string | null (IPv4/IPv6, /24까지만 저장)
    - userAgentHash: string | null (sha256 해시, 원문 저장 안 함)
    - meta: Json | null — action별 추가 정보. 시스템 잡의 경우 `{ systemJobName: "cleanup-pdfs"|"account-hard-delete"|..., batchSize: number, traceId: string }`. consent_revoke의 경우 `{ scope: "retention"|"parent_share"|"all", source: "settings_privacy_toggle" }`.
    - occurredAt: Date
    Indexes: [targetStudentId+occurredAt desc], [actorUserId+occurredAt desc], [occurredAt asc] (1년 후 삭제 cron)
    호출처 매핑 (CRITICAL: 다음 액션은 spec의 해당 위치에서 반드시 기록):
      - view_pdf: PdfFile 다운로드/미리보기 시
      - delete_pdf: Server Action deletePdfFile + cleanup-pdfs cron (actorType=system)
      - upload_pdf: PDF 업로드 완료 직후
      - view_profile: 학부모가 학생 프로파일 열람 시
      - export_data: exportData Server Action 호출 시
      - parent_view_summary: /parent/report/[studentId] 진입 시
      - share_link_create: createShareLink 호출 시 (actor=학생)
      - share_link_view: /share/[token] 진입 시 (actorType=anonymous)
      - share_link_revoke: revokeShareLink 호출 시
      - consent_grant: 보호자 동의 완료 또는 14세 도달 본인 동의 전환
      - consent_revoke: /settings/privacy "동의 관리" 토글 OFF 시 (학부모 공유 / PDF 보관 / 전체 철회)
      - consent_renewal_required: 14세 도달 학생 로그인 시 모달 표시
      - track_override: confirmTrack(override) 호출 시
      - todo_skip: updateTodoState(state="skipped") 시
      - account_delete_requested: requestAccountDeletion 호출 시
      - account_delete_completed: account-hard-delete cron 실행 시 (actorType=system)
      - schooltype_waitlist_register: vocational/other 사용자가 waitlist 등록 시
  </audit_log>

  <waitlist_entry>
    엔티티: WaitlistEntry (vocational/other 학교 유형 사용자의 v2 출시 알림 신청. M2 차단 모달에서 [waitlist 이메일 등록] 선택 시 생성)
    - id: string (uuid, pk)
    - email: string (lowercased, 254자 max) — 가입 전 단계 사용자 이메일. User.id와 미연결.
    - schoolType: enum ("vocational", "other")
    - region: string (시도, 17개 enum 중 1)
    - currentGrade: number (-1=중학생, 1|2|3=고등학년) — 사용자가 onboarding_role_page에서 선택한 값
    - examCycle: enum ("CYCLE_2025","CYCLE_2026","CYCLE_2027","CYCLE_2028") — birthYear 기반 자동 계산
    - source: enum ("schooltype_block_modal", "external_inquiry") default "schooltype_block_modal"
    - createdAt: Date
    - notifiedAt: Date | null — v2 출시 시 일괄 알림 발송 후 채워짐
    - unsubscribedAt: Date | null — 사용자가 이메일 내 unsubscribe 링크 클릭 시
    Indexes: [email unique], [schoolType+createdAt desc], [notifiedAt] (미발송 필터)
    개인정보 정책: 이메일 외 PII 미저장, 90일 미가입 시 cleanup-waitlist cron이 hard delete (별도 cron). v2 출시 후 일괄 발송 → 30일 내 가입 미전환 시 hard delete.
  </waitlist_entry>

</core_data_entities>

<authentication>
  <strategy>NextAuth.js v5 Credentials + Email OTP + OAuth (Kakao, Google). JWT 세션(HttpOnly cookie).</strategy>
  <providers>
    <email_otp>
      - 회원가입: 이메일 입력 → 6자리 OTP 발송 (Resend) → 5분 유효 → 검증 통과 시 emailVerifiedAt 설정
      - 로그인: 동일 OTP 흐름 또는 비밀번호(설정 시)
      - 비밀번호: 선택. 설정 시 최소 10자, 영문+숫자 포함, bcrypt cost 12.
        OAuth-only 사용자(passwordHash=null)는 /settings/account에서 [비밀번호 설정] 버튼으로 신규 설정 가능 (이메일 OTP 재인증 후). 설정 후 [비밀번호 변경]으로 표기 전환. OAuth 연결을 끊어도 비밀번호 미설정 시에는 이메일+OTP 로그인 가능.
      - OTP 발송 레이트 리밋: 1분 1회, 1시간 5회 (이메일 단위, IP 보조)
    </email_otp>
    <oauth>
      <kakao>카카오 OAuth 2.0 (이메일+닉네임 scope만 요구). 1차 권장 채널.</kakao>
      <google>Google OAuth (email scope). 데스크톱 사용자 대안.</google>
    </oauth>
  </providers>
  <session>
    <storage>HttpOnly Secure SameSite=Lax 쿠키, 토큰은 JWT(HS256, NEXTAUTH_SECRET 서명)</storage>
    <duration>30일 sliding window. 7일마다 silent refresh.</duration>
    <invalidation>비밀번호 변경/계정 삭제 요청 시 모든 세션 무효화 (jti blacklist)</invalidation>
  </session>
  <minor_consent>
    CRITICAL: 회원가입 시 birthYear로 만 14세 미만 자동 판별. 알고리즘 (services/lib/age.ts):
      ```
      function requiresParentConsent(birthYear: number, today = new Date()): boolean {
        // 출생일 미수집 환경에서 생일 미도래 가능성을 안전하게 처리:
        // birthYear + 14 > today.getFullYear() 인 경우는 확실히 만 14세 미만.
        // birthYear + 14 == today.getFullYear() 인 경우는 생일 미도래 가능성 → 보호자 동의 흐름으로 분기 (안전 측).
        // birthYear + 14 < today.getFullYear() 인 경우는 확실히 만 14세 이상.
        return (birthYear + 14) >= today.getFullYear();
      }
      ```
      - 추후 birthDate(YYYY-MM-DD) 수집 옵션 추가 시 정확 계산으로 대체 가능. MVP는 birthYear만 수집하므로 보수적 분기.
    14세 미만 흐름:
      1. 학생이 가입 시도 → birthYear 입력
      2. 만 14세 미만 감지 → "보호자 이메일" 입력 화면
      3. ParentChildLink 생성 (consentStatus=pending, consentToken 발급, 7일 유효)
      4. 보호자 이메일로 동의 링크 발송 (Resend)
      5. 보호자가 링크 클릭 → /parent-consent/[token] → 보호자 본인인증(이메일 OTP) + 동의 항목 체크 → consentStatus=granted
      6. granted 후 학생 가입 완료 통지 (이메일)
      7. 동의 미완료 7일 경과 시 학생 계정 자동 삭제 + 안내 이메일
  </minor_consent>
  <ownership_transfer_policy>
    CRITICAL: Student.ownerUserId 소유권 이전 흐름 (학부모 선등록 → 자녀 합류 시)
      케이스 A — 학생이 먼저 가입 (학생 본인 = ownerUserId):
        - 학부모는 Resend로 학생이 보낸 ParentChildLink invite 수락 → 동의 후 ParentChildLink만 생성. ownerUserId는 변경 없음.
      케이스 B — 학부모가 먼저 가입 + Student를 선등록 (ownerUserId = parentUserId):
        - 학부모 가입 후 자녀 정보 입력 시 Student 생성. 이때 ownerUserId=parentUserId.
        - 자녀가 추후 동일 학부모 등록 이메일을 통해 invite 수락 + 본인 가입 시:
          1. 학생 User 신규 생성 (role=student)
          2. ParentChildLink는 유지 (consentStatus=granted_by_self로 전환)
          3. Student.ownerUserId를 parentUserId → 신규 학생 User.id로 이전 (트랜잭션 내 1회).
          4. AuditLog action=consent_grant + meta={"ownership_transferred": true}.
        - 자녀가 본인 가입을 거부하거나 합류 안 함 시: ownerUserId=parentUserId 유지, 학부모만 사용 가능 (서비스 사용은 가능하나 학생 본인 화면은 비활성).
      케이스 C — 자녀가 본인 가입했는데 학부모가 다른 이메일로 별도 학생을 만든 경우:
        - 두 Student row 별도 유지. 합치기는 v2 (사용자 수동 요청 필요).
      이전 후 동일 Student의 PDF/Profile/Todo 데이터는 그대로 유지(소유자만 변경). 1회용 transferToken(7일 유효, ParentChildLink.consentToken 재사용)으로 자녀 본인 인증 후 이전.
  </ownership_transfer_policy>
  <separated_consent_required>
    회원가입과 별도로 `/onboarding/consent`에서 다음 3개 항목을 분리 체크해야 PDF 업로드 진행 가능:
      1. [필수] 민감정보 처리 동의 (생기부 분석 목적)
      2. [선택] PDF 30일 보관 동의 (미동의 시 24시간 후 자동 삭제)
      3. [선택] 학부모와 결과 공유 (학생 가입자만 표시)
    체크박스 묶음 금지(각각 별도 input). 필수 미체크 시 다음 단계 진행 차단.
  </separated_consent_required>
  <authorization>
    <roles>enum (student, parent, system)</roles>
    <rules>
      - student: 본인 Student 엔티티 + 하위 데이터 모두 read/write
      - parent: ParentChildLink.consentStatus=granted인 Student의 한정된 read (요약 리포트 + 트랙 + TODO 카운트만, 개별 TODO 본문은 학생 명시 동의 시 표시)
      - system: cron/worker가 사용하는 시스템 계정 (감사 로그 기록용)
    </rules>
  </authorization>
  <protected_routes>
    - /onboarding/* — 로그인 필요
    - /dashboard, /today, /roadmap/*, /todo/*, /checkin, /profile, /settings/* — 로그인 + role=student 또는 학부모인 경우 자녀 1명 이상 연결 필요
    - /parent/report/[studentId] — 로그인 + role=parent + 해당 studentId에 대한 ParentChildLink granted
    - /share/[token] — 비로그인 허용. 토큰 유효성 + expiresAt + revokedAt 확인
    - /api/upload/pdf — 로그인 + 동의 완료 + 본인 또는 자녀 studentId 검증
  </protected_routes>
  <redirect_flows>
    - 비인증 보호 라우트 접근 → /auth/login?next=&lt;intended&gt;
    - 로그인 후 onboarding 미완 → /onboarding/role
    - 로그인 후 onboarding 완료 + role=student → /dashboard
    - 로그인 후 onboarding 완료 + role=parent → /parent/report/&lt;studentId&gt; (자녀 1명 시 자동, N명 시 선택 화면)
  </redirect_flows>
  <account_deletion>
    - /settings/privacy에서 "계정 삭제 요청" → 30일 유예(deletedAt 설정, 모든 세션 무효화)
    - 30일 내 로그인 시 복구 옵션 제공
    - 30일 경과 후 cron이 hard delete: User, Student(소유), PdfFile (S3 객체 + 레코드), ParsedReportCard, StudentProfile, TrackRecommendation, Todo, WeeklyCheckIn, ShareLink, AuditLog는 익명화(actorUserId=null)하여 1년까지 잔존 후 삭제
  </account_deletion>
</authentication>

<route_definitions>
  <public_routes>
    <route path="/" page="LandingPage" />
    <route path="/auth/signup" page="SignupPage" />
    <route path="/auth/login" page="LoginPage" />
    <route path="/auth/verify-otp" page="VerifyOtpPage" />
    <route path="/parent-consent/:token" page="ParentConsentPage" />
    <route path="/share/:token" page="SharedReportPage" />
    <route path="/legal/terms" page="TermsPage" />
    <route path="/legal/privacy" page="PrivacyPage" />
  </public_routes>
  <protected_routes guard="requireAuth">
    <route path="/onboarding/role" page="OnboardingRolePage" guard="requireAuth+notOnboarded" />
    <route path="/onboarding/consent" page="OnboardingConsentPage" />
    <route path="/onboarding/upload" page="OnboardingUploadPage" />
    <route path="/onboarding/correct" page="OnboardingCorrectPage" />
    <route path="/onboarding/track" page="OnboardingTrackPage" />
    <route path="/dashboard" page="DashboardPage" guard="requireStudentOrParent" />
    <route path="/today" page="TodayPage" guard="requireStudent" />
    <route path="/roadmap/:grade/:term" page="RoadmapTermPage" guard="requireStudent" />
    <route path="/todo/:id" page="TodoDetailPage" guard="requireStudent" />
    <route path="/checkin" page="CheckinPage" guard="requireStudent" />
    <route path="/profile" page="ProfilePage" guard="requireStudentOrParent" />
    <route path="/settings/account" page="AccountSettingsPage" />
    <route path="/settings/privacy" page="PrivacySettingsPage" />
    <route path="/parent/report/:studentId" page="ParentReportPage" guard="requireParentOf(studentId)" />
  </protected_routes>
  <api_routes>
    <route path="/api/auth/*" handler="NextAuth handler" />
    <route path="/api/upload/pdf" handler="POST: PDF 업로드(multipart, max 25MB body)" method="POST" />
    <route path="/api/jobs/cleanup-pdfs" handler="Vercel Cron (매일 00:00 KST). PdfFile.expiresAt &lt; now() hard delete" method="POST" auth="cron-secret-header" />
    <route path="/api/jobs/cleanup-audit-logs" handler="매일 자정. AuditLog.occurredAt &lt; now() - 1y 삭제" method="POST" auth="cron-secret-header" />
    <route path="/api/share/:token" handler="공유 링크 검증(GET) + viewCount 증가" method="GET" />
    <route path="/api/health" handler="헬스체크" method="GET" />
  </api_routes>
  <server_actions>
    Server Actions (mutations, `'use server'` 함수):
    - createUser(email: string, birthYear: number, role: UserRole) → User
    - sendOtp(email: string) → { sent: boolean, expiresAt: Date }
    - verifyOtp(email: string, code: string) → Session
    - submitOnboarding(role: UserRole, birthYear: number, schoolType: SchoolType, region: string, currentGrade: number) → Student
    - submitConsent(consentNecessary: boolean, consentRetention: boolean, consentParentShare: boolean) → void
    - revokeConsent(scope: "retention" | "parent_share" | "all") → void  // /settings/privacy 동의 토글 OFF 시. AuditLog action=consent_revoke + meta.scope 기록. scope="all"이면 PdfFile 즉시 삭제 + ShareLink 모두 revoke.
    - acknowledgeConsentRenewal(decision: "self_consent" | "delete_account") → void  // 14세 도달 학생이 로그인 시 표시되는 모달의 응답. self_consent → ParentChildLink.consentStatus=granted_by_self 전환 + 본인 동의 흐름 진행. delete_account → 계정 삭제 30일 유예 흐름.
    - registerWaitlist(email: string, schoolType: "vocational" | "other", region: string, currentGrade: number) → WaitlistEntry  // M2 차단 모달의 [waitlist 이메일 등록]. 인증 불필요(가입 전 단계). AuditLog action=schooltype_waitlist_register, actorType=anonymous.
    - requestParseJob(pdfFileId: string) → { jobId: string }
    - submitCorrection(parsedReportCardId: string, sectionsPatch: Partial&lt;Sections&gt;) → ParsedReportCard
    - confirmTrack(trackRecommendationId: string, override?: { primaryTrack: Track, reason: string }) → TrackRecommendation
    - updateTodoState(todoId: string, nextState: TodoState, payload?: { skipReason?: SkipReason, snoozeUntil?: Date, completionNote?: string }) → Todo
    - submitWeeklyCheckin(weekStart: Date, retroText: string, satisfactionScore: number) → WeeklyCheckIn
    - createShareLink(studentId: string, audience: ShareAudience, options: { includeTodoBody: boolean, includeSchoolType: boolean, includeRegion: boolean }) → ShareLink
    - revokeShareLink(shareLinkId: string) → void
    - deletePdfFile(pdfFileId: string) → void
    - requestAccountDeletion() → { scheduledHardDeleteAt: Date }
    - exportData() → { signedUrl: string, expiresAt: Date }  // role=student만 호출 가능, role=parent는 403
    - transferStudentOwnership(studentId: string, transferToken: string) → Student  // 학부모 선등록 후 자녀 합류 시 ownerUserId 이전(케이스 B). transferToken 검증 + 트랜잭션 처리.
    모든 액션은 Zod 스키마 검증 + NextAuth `auth()` 권한 체크(registerWaitlist 제외) + AuditLog 자동 기록(해당되는 경우) + Sentry 에러 캡처.
  </server_actions>
  <redirects>
    <redirect from="/login" to="/auth/login" status="301" />
    <redirect from="/signup" to="/auth/signup" status="301" />
  </redirects>
</route_definitions>

<component_hierarchy>
  <app_shell>
    <providers>
      <!-- 외부에서 안쪽으로: SentryErrorBoundary → ThemeProvider(light/dark) → SessionProvider(NextAuth) → QueryProvider(TanStack) → ToastProvider(sonner) -->
      <router>
        <!-- 공개 레이아웃 (랜딩, 인증, 공유 링크) -->
        <public_layout>
          <top_nav_minimal />          <!-- 로고 56px, 로그인/가입 버튼 -->
          <outlet />
          <footer />                   <!-- 면책 + 약관 링크 -->
        </public_layout>

        <!-- 온보딩 레이아웃 (전체 화면 단계) -->
        <onboarding_layout>
          <progress_stepper />         <!-- 5단계 점선 -->
          <outlet />
          <disclaimer_bar />
        </onboarding_layout>

        <!-- 인증된 앱 레이아웃 -->
        <app_layout>
          <!-- ≤768px -->
          <bottom_tabs />              <!-- 56px 고정. 학생: 오늘/로드맵/체크인/프로필 (4개). 학부모: 리포트/프로필/설정 (3개) -->
          <!-- ≥769px -->
          <sidebar />                  <!-- 240px 고정 -->
          <main_area>
            <top_bar>                  <!-- 모바일에서는 sticky 56px -->
              <breadcrumb />
              <decision_point_badge /> <!-- 임박 결정 D-N 배지 -->
              <user_menu />
            </top_bar>
            <page_content>
              <outlet />
            </page_content>
            <disclaimer_bar />         <!-- 항상 하단 12px 면책 -->
          </main_area>
        </app_layout>
      </router>
    </providers>
  </app_shell>

  <shared>
    <Modal />                          <!-- Radix Dialog wrapper -->
    <ConfirmDialog />                  <!-- 삭제 확인 등 -->
    <SonnerToaster />
    <EmptyState />                     <!-- 일러스트 + 메시지 + CTA -->
    <LoadingSkeleton />
    <ErrorBoundary />                  <!-- React Error Boundary, 페이지 단위 -->
    <DisclaimerBar />                  <!-- "추천은 참고용입니다" 고정 문구 -->
    <ExplanationTooltip />             <!-- "왜 이 추천인가요?" -->
  </shared>
</component_hierarchy>

<pages_and_interfaces>

  <global_layout>
    <bottom_tabs_mobile>
      높이 56px, 안전영역(safe-area) 추가. 배경 #FFFFFF (다크 #0B0E14), 상단 보더 1px #E5E7EB.
      4개 탭, 각 활성 탭은 아이콘+라벨 #1F4ED8, 비활성 #6B7280.
      터치 영역 44×44px 보장.
      학생 탭: [오늘 (Sun)] [로드맵 (Map)] [체크인 (CheckCircle)] [프로필 (User)]
      학부모 탭: [리포트 (FileText)] [프로필 (User)] [설정 (Settings)]
    </bottom_tabs_mobile>
    <sidebar_desktop>
      너비 240px 고정. 배경 #F9FAFB. 상단 로고 56px, 그 아래 메뉴 5개(학생) / 3개(학부모).
      메뉴 항목: 패딩 12px 16px, hover bg #F3F4F6, active bg #DBEAFE + 좌측 3px #1F4ED8 인디케이터.
      하단 사용자 메뉴(아바타 32px + 이메일 + 로그아웃).
    </sidebar_desktop>
    <disclaimer_bar>
      위치: 모든 인증 화면 하단 fixed. 높이 32px, 배경 #FFFBEB, 텍스트 12px #92400E, 중앙 정렬.
      문구 (고정 한국어): "본 서비스의 추천은 참고용이며, 합격을 보장하지 않습니다. 매년 입시 정책이 변동될 수 있으니 학교·EBS·대학 공식 자료를 함께 확인하세요."
    </disclaimer_bar>
  </global_layout>

  <landing_page>
    경로 `/`. 비로그인 사용자.
    <hero_section>
      높이 100vh-56px, 배경 그라데이션 #1F4ED8 → #6366F1 (각도 135deg).
      H1 (Pretendard Bold 40px / 모바일 28px, 색 #FFFFFF): "생기부 한 장으로 시작하는 3년 진학 로드맵"
      서브카피 (16px #E0E7FF): "초·중학교 생기부 PDF를 올리면 30초 안에 고1~고3 단계별 TODO를 만들어드립니다."
      CTA 버튼 2개:
        - [무료로 시작하기] (Primary): bg #FFFFFF, text #1F4ED8, 56px height, 240px width, radius 12px
        - [어떻게 작동하나요] (Ghost): border 1px #FFFFFF, text #FFFFFF
    </hero_section>
    <features_section>
      3-column grid (모바일 1열). 각 카드 240px 높이, 패딩 24px, 배경 #FFFFFF, 그림자 0 4px 12px rgba(0,0,0,0.06).
      카드 1: "PDF → 30초 로드맵" (FileSearch 아이콘 32px #1F4ED8)
      카드 2: "결정 포인트 알림" (CalendarClock 아이콘 #F97316)
      카드 3: "2028 개편 자동 반영" (Sparkles 아이콘 #10B981)
    </features_section>
    <demo_section>
      높이 480px. 좌측: 대시보드 스크린샷 mock. 우측: bullet 5개 (모바일은 위/아래 스택).
    </demo_section>
    <pricing_section>
      2-column. "학생 무료" (영구) / "학부모 프리미엄 ₩9,900/월" (요약 리포트 + 다중 자녀 v2).
    </pricing_section>
    <empty_state>해당 없음 (정적 페이지)</empty_state>
  </landing_page>

  <signup_page>
    경로 `/auth/signup`. 카드 형태 480px width 중앙. 모바일 풀폭 패딩 16px.
    <form_block>
      필드:
        - 이메일 (input type=email, placeholder "이메일", 48px height, border 1px #D1D5DB, focus border 2px #1F4ED8)
        - 출생연도 (number, 1980~2020, 즉시 검증)
        - 역할 (radio 2개: 학생 본인 / 학부모)
      버튼: [OTP 받기] Primary 48px, full width, disabled 시 #9CA3AF 배경
      소셜 로그인 (Divider "또는" 위/아래): [카카오] #FEE500 / [구글] #FFFFFF + 1px #D1D5DB. 각 48px.
      하단 텍스트: "이미 계정이 있으신가요? 로그인" (#1F4ED8 링크)
    </form_block>
    <validation_errors>
      이메일 형식 오류: "올바른 이메일을 입력해주세요" (#EF4444 13px, 필드 아래)
      출생연도 범위 오류: "1980~2020 사이 연도를 입력해주세요"
      14세 미만 감지 시: 입력 후 [OTP 받기] 클릭 시 "보호자 동의가 필요합니다" 모달 → 보호자 이메일 입력 폼
    </validation_errors>
  </signup_page>

  <verify_otp_page>
    경로 `/auth/verify-otp`. 6자리 OTP 입력 (개별 박스 6개, 각 48×56px, 자동 포커스 이동).
    OTP 만료 카운트다운: "X분 Y초 남음". 만료 시 [재발송] 버튼 활성.
    검증 통과 시 → /onboarding/role (또는 next 파라미터)
  </verify_otp_page>

  <parent_consent_page>
    경로 `/parent-consent/:token`. 학생이 입력한 보호자 이메일로 발송된 링크 진입.
    <header>학생 이름(별명) + 만 N세 + "다음 항목에 동의해주세요"</header>
    <consent_items>
      체크박스 4개 (모두 필수):
        1. 자녀의 회원가입 및 서비스 이용
        2. 생활기록부 분석을 위한 민감정보 처리
        3. 보호자 본인 확인 (이메일 OTP 검증 별도)
        4. 본 동의는 자녀가 만 14세 도달 시 갱신 필요
      각 항목 옆 "전문 보기" → 모달로 약관 풀텍스트 표시
    </consent_items>
    <verify_block>
      [보호자 본인인증 OTP 받기] → 6자리 OTP 입력
      검증 통과 + 모든 체크 시 [동의 완료] 버튼 활성
    </verify_block>
    <empty_state>토큰 만료/유효하지 않을 때: "이 동의 링크는 만료되었습니다. 자녀에게 새 링크 요청을 부탁드립니다."</empty_state>
  </parent_consent_page>

  <onboarding_role_page>
    경로 `/onboarding/role`. 단계 1/5.
    <fields>
      - 학교 유형 (radio 4: 일반계 / 자사·특목 / 직업계 / 기타)
        * 일반계 선택: 그대로 진행
        * 자사·특목 선택: 알림 박스 (#FEF3C7 배경) "본 MVP는 일반계 학생에게 최적화되어 있습니다. 자사·특목은 best-effort로 일반계 가정 추천을 제공합니다." [그대로 진행] 또는 [돌아가기]
        * 직업계(vocational)·기타(other) 선택: 모달 차단 (#FEF2F2 배경 + 빨강 아이콘) "MVP는 일반계 학생만 정확하게 지원합니다. 직업계·기타 학교 유형은 추후 지원 예정이에요. 알림 신청을 남겨주시겠어요?" → [waitlist 이메일 등록] / [그대로 진행 (best-effort)] 두 옵션. waitlist 등록 시 가입 대기 상태(WaitlistEntry 별도 저장), 그대로 진행 시 Student 생성하되 trackRecommendation 화면·로드맵 화면에 영구 워터마크 "일반계 가정 추천 — 정확도 제한" 표시 + Todo 생성 시 trackFit 가중치 0.5로 강제 다운.
      - 시도 (드롭다운 17개)
      - 현재 학년 (radio: 초6/중1/중2/중3/고1/고2/고3)
      - 현재 학기 (자동 계산 + 사용자 수정 허용)
    </fields>
    <button>[다음 →] 56px Primary, full width 모바일 / 200px 데스크톱</button>
    <exam_cycle_badge>입력 직후 우측 상단에 "당신의 입시 사이클: 2028 (개편 적용)" 동적 표시 (#10B981 배경, 흰 텍스트, 12px 패딩)</exam_cycle_badge>
    <school_type_branching_policy>
      CRITICAL: services/recommendation/seedLoader.ts와 services/signal/tracks.ts에서 `student.schoolType !== "general"`일 때:
        1. seedLoader: track 분기 시드는 적용하되 trackFit 가중치 0.5배.
        2. tracks.ts: 듀얼 추천 confidence를 한 단계 강제 하향 (high→mid, mid→low).
        3. UI: 모든 추천 결과에 "일반계 가정 추천 — 정확도 제한" 워터마크 (16px text, #92400E).
        4. 직업계 사용자는 정시·실기 카테고리 우선 (D1 모집요강, B 수능 일반 가이드).
    </school_type_branching_policy>
  </onboarding_role_page>

  <onboarding_consent_page>
    경로 `/onboarding/consent`. 단계 2/5.
    <consent_blocks>
      각 블록은 카드(border 1px #E5E7EB radius 12px padding 20px), 체크박스 24×24px 좌측, 우측 제목+설명+"전문 보기" 링크.
      1. [필수] 민감정보 처리 동의 — "생기부의 학습·활동·평가 텍스트를 분석하여 TODO를 생성합니다."
      2. [선택] PDF 30일 보관 — "보관 동의 시 30일 이내 추가 분석 가능. 미동의 시 24시간 후 자동 삭제."
      3. [선택] 학부모와 결과 공유 — "학부모 이메일 입력 후 요약 리포트 공유 링크 발송." (학생만 표시)
    </consent_blocks>
    <button>[동의하고 계속] disabled when 1번 미체크</button>
    <link>[돌아가기] 텍스트 링크 12px 회색</link>
  </onboarding_consent_page>

  <onboarding_upload_page>
    경로 `/onboarding/upload`. 단계 3/5.
    <dropzone>
      640×320px (모바일 풀폭×280px), border 2px dashed #1F4ED8, radius 16px, hover bg #EFF6FF.
      중앙 아이콘 (UploadCloud 64px #1F4ED8) + "PDF 파일을 끌어다 놓거나 클릭하세요" + "최대 20MB · NEIS 발급본 또는 학교 발급본"
      파일 선택 시 즉시 sha256 해시 계산 → 중복 업로드 시 기존 PdfFile로 이동
    </dropzone>
    <progress>
      업로드 시작 시 dropzone 위치에 진행 상태 카드 표시. 단계 5개 (각 체크/스피너):
        1. 업로드 (0~100%)
        2. 텍스트 추출
        3. PII 마스킹
        4. 신호 분석
        5. 로드맵 생성
      각 단계 우측 예상 시간 표시 (P95 기준).
    </progress>
    <error_states>
      - 파일 형식 오류: "PDF 파일만 업로드 가능합니다" (Toast 빨강)
      - 크기 초과: "20MB 이하 파일만 업로드 가능합니다"
      - 텍스트 추출 실패 → OCR 자동 폴백 ("스캔본으로 감지되어 OCR로 처리 중... 시간이 더 걸릴 수 있습니다") 진행 카드 노란 강조
      - OCR도 실패: "자동 인식에 실패했어요. 수동 입력 모드로 전환할까요?" 모달 [수동 입력] / [재시도]
    </error_states>
    <empty_state>해당 없음 (드롭존 자체가 비어있는 상태 디자인)</empty_state>
  </onboarding_upload_page>

  <onboarding_correct_page>
    경로 `/onboarding/correct`. 단계 4/5. 좌우 분할(모바일은 탭 전환).
    <left_panel>
      너비 50%, PDF 미리보기 (pdfjs-dist 렌더). 페이지 네비게이션, 줌 인/아웃.
    </left_panel>
    <right_panel>
      너비 50%, 추출 결과를 항목별 아코디언 (9개 sections).
      각 항목 헤더: 제목 + 신뢰도 배지 (≥80 #10B981 / 50~79 #F59E0B / &lt;50 #EF4444 빨강 테두리).
      신뢰도 &lt;80인 항목은 자동 펼침. 인라인 편집 가능 (textarea, contenteditable).
      편집 후 [저장] (autosave 2초 디바운스).
    </right_panel>
    <skip_option>[수정 건너뛰기 →] 우측 상단 (충분히 잘 추출됐을 때)</skip_option>
    <button>[다음으로 (트랙 추천 보기)] 하단 56px full width Primary</button>
    <empty_state>추출된 항목이 0개일 때: "추출 결과가 부족해요. 수동 입력 모드로 전환하시겠어요?"</empty_state>
  </onboarding_correct_page>

  <onboarding_track_page>
    경로 `/onboarding/track`. 단계 5/5.
    <result_card>
      카드 2개 좌우 (모바일 위/아래).
      카드 1 (Primary 트랙): height 280px, border 2px #1F4ED8, radius 16px, 상단 "주력 트랙" 배지.
        제목 (예: "학생부종합전형") 32px Bold #1F4ED8
        신뢰도 라벨 (높음/중간/낮음) 배지 옆
        근거 신호 (bullet 3~5개): "자유학기 진로 산출물 강함 (★★★★★)", "동아리 일관성 높음", ...
      카드 2 (Secondary): 동일 구조. border #6366F1.
    </result_card>
    <override_block>
      "내가 직접 트랙을 정하고 싶어요" 토글 → 6개 트랙 라디오 + 사유 입력 (200자)
    </override_block>
    <message_for_low_signal>
      신호 부족 시 (중학생 입력) 카드 대신 "**탐색 모드**" 안내: "현재로선 모든 트랙이 열려있어요. 고1 진로 탐색 활동을 추천합니다."
    </message_for_low_signal>
    <button>[로드맵 보러가기] 56px Primary</button>
  </onboarding_track_page>

  <dashboard_page>
    경로 `/dashboard`. 학생 메인.
    <decision_point_banner>
      상단 sticky 64px. 배경 #FEF3C7, 좌측 아이콘 (CalendarClock 24px #F59E0B), 본문 "고1 2학년 선택과목 결정 D-32 — 8월 말까지 결정 권장", [지금 보기 →] 버튼 ghost.
      해당 결정이 없을 시 숨김.
    </decision_point_banner>
    <main_grid>
      ≥1024px: 70/30 분할. 좌측: 격자, 우측: 오늘의 TODO 패널.
      &lt;1024px: 단일 컬럼, 격자 위, 오늘의 TODO 아래.
    </main_grid>
    <grade_term_grid>
      구조: 3행(고1/고2/고3) × 11열 (1H1, 1Hsummer, 1H2, 1Hwinter, 2H1, 2Hsummer, 2H2, 2Hwinter, 3H1, 3Hsummer, 3H2). 단 고3 row는 끝열 3Hwinter 셀이 비활성(opacity 0.3, "수능 후" 라벨, 클릭 비활성) — UI 일관성 유지를 위해 격자 자체는 12열로 그리되 3Hwinter 위치만 disabled 표시. 사실상 활성 셀 33개(고1 11 + 고2 11 + 고3 11) — Term enum 11종과 1:1 일치.
      각 셀 96×96px (모바일 72×72px). 셀 내용: 학기 라벨 + TODO 카운트 (예 "5/12") + 진행 막대(높이 4px).
      현재 학기 셀: border 2px #1F4ED8 + glow. 과거 학기: 회색조. 미래 학기: 흐리게(opacity 0.6).
      hover 시 미니 미리보기 툴팁 (카테고리 카운트).
      탭/클릭 → /roadmap/[grade]/[term] 이동.
      반응형:
        - 데스크톱(≥1024px): 11열 가로 스크롤(overflow-x:auto). 현재 학기를 viewport 가운데로 자동 scrollIntoView({behavior:"smooth", inline:"center"}). 좌우 끝 그라데이션 페이드 마스크(20px).
        - 태블릿(640~1023px): 동일 11열 가로 스크롤, 셀 80×80px로 축소.
        - 모바일(≤639px): 학년별로 그룹 — 학년 탭 3개([고1][고2][고3])로 분리, 활성 학년만 11셀 표시(가로 스크롤). 현재 학년 자동 활성. 초기 로드 시 stagger fade-in 적용 (각 셀 +30ms, 총 max 330ms).
        - 키보드 단축키 (pages_and_interfaces 참조): ←/→ 학기 이동, ↑/↓ 학년 이동, Enter 진입.
    </grade_term_grid>
    <today_panel>
      너비 380px (≥1024px) / 풀폭. 카드 5개 세로 스택, 각 96px height.
      카드: 좌측 카테고리 색 5px 띠 + 카테고리 아이콘 24px + 제목 14px Bold + 마감일 12px #6B7280 + 우측 [시작] 버튼.
      비어있을 때: EmptyState ("오늘은 쉬어가요. 잘하고 계세요.")
    </today_panel>
    <empty_state>전체 TODO 0개일 때: "아직 로드맵이 없어요. PDF를 업로드해보세요." [업로드 →] CTA</empty_state>
  </dashboard_page>

  <today_page>
    경로 `/today`. 모바일 메인.
    <header>"오늘의 5가지" + 날짜 (Asia/Seoul)</header>
    <todo_cards>
      5개 카드(카테고리 밸런스 룰 적용: A,B,C,D,E 각 1개 우선). 각 카드 패딩 16px, radius 12px, border 1px #E5E7EB.
      구성: 카테고리 칩 + 제목 + description 2줄 truncate + explanation 인용("💡 ...") + 액션 버튼 4개 [시작][완료][스누즈][스킵].
    </todo_cards>
    <quick_actions>
      각 액션은 모달 또는 인라인:
        - 시작: 즉시 상태 in_progress
        - 완료: 짧은 회고(5점 + 한 줄)
        - 스누즈: 7일 / 다음 학기 / 사용자 지정 날짜
        - 스킵: 사유 4개 라디오 (이미 함/관심 없음/시기 안 맞음/기타)
    </quick_actions>
    <empty_state>"오늘 추천이 없어요. 로드맵에서 직접 골라보세요." [로드맵으로 →]</empty_state>
  </today_page>

  <roadmap_term_page>
    경로 `/roadmap/[grade]/[term]`. 학기 상세.
    <header>
      뒤로가기 + 학기명 (예 "고1 1학기") + 진행률 (12/24 완료, 도넛 차트 32px)
    </header>
    <category_filters>
      칩 5개 (A 내신 / B 수능 / C 세특 / D 입시 / E 생활) + "전체". 활성 칩은 카테고리 색 배경.
    </category_filters>
    <todo_groups>
      카테고리별 그룹 (헤더 + TODO 카드 리스트). 각 카드 76px height. 우선순위 desc 정렬.
      상태별 색상: backlog=기본, today=#DBEAFE 배경, in_progress=#FEF3C7 + 좌측 띠 #F59E0B, done=#D1FAE5 + 취소선, snoozed=흐림 + Clock 아이콘, skipped=숨김(필터로 표시 가능).
    </todo_groups>
    <empty_state>해당 학기 TODO 0: "이 학기에는 추천이 없어요." (자동 스누즈된 학기일 수 있음)</empty_state>
  </roadmap_term_page>

  <todo_detail_page>
    경로 `/todo/[id]`.
    <header>카테고리 칩 + 제목 28px + 마감일</header>
    <body>
      - description (markdown 렌더)
      - explanation 카드 (배경 #EFF6FF, 좌측 띠 #1F4ED8 4px): "💡 왜 이 추천인가요? — {explanation 본문}"
      - 관련 신호 (있을 시): "당신의 자유학기 '환경' 산출물에서 도출"
      - 사용자 노트 (textarea, 자동 저장)
      - 첨부 (URL 추가, 최대 5개)
    </body>
    <state_actions>
      현재 상태에 따라 가능한 다음 상태 버튼만 표시 (상태 전이 규칙 준수).
    </state_actions>
    <history>변경 이력 (state, 시각). 마지막 5건만 표시.</history>
  </todo_detail_page>

  <checkin_page>
    경로 `/checkin`.
    <header>이번 주 (월~일) + "이번 주 5분 체크인"</header>
    <retro_section>
      지난 주 TODO 리스트 (완료/미완료 표시). 미완료 각각 빠른 액션 [다음 주로][스누즈][스킵].
      만족도 5점 (별 5개), 한 줄 회고 textarea (200자 카운터).
    </retro_section>
    <next_week>
      다음 주 추천 5개 미리보기 + [확정] 버튼.
    </next_week>
    <empty_state>첫 주 또는 데이터 없을 때: "체크인은 가입 1주 후부터 가능해요."</empty_state>
  </checkin_page>

  <profile_page>
    경로 `/profile`.
    <sections>
      1. 헤더: 별명 + 학년/학기 배지 + 입시 사이클 배지
      2. 흥미 가설 (3개 카드, confidence bar)
      3. 강점 (chip 리스트, 최대 10개, #D1FAE5 배경)
      4. 다음 단계 (=약점, #FEF3C7 배경)
      5. 트랙 (현재 듀얼 추천 카드 압축 버전)
      6. 최근 업로드한 생기부 (PdfFile 목록 + 만료까지 D-N + [지금 삭제] 버튼)
    </sections>
    <empty_state>온보딩 미완 시 자동 리다이렉트</empty_state>
  </profile_page>

  <parent_report_page>
    경로 `/parent/report/[studentId]`. 학부모 전용.
    <header>"{학생 별명}의 학습 요약" + 마지막 갱신 시각</header>
    <one_pager>
      A4 비율 1장 (인쇄 가능, @media print 스타일):
        - 상단: 학년/학기/입시사이클
        - 트랙 듀얼 추천 (간략 차트, recharts RadialBar)
        - 흥미 가설 top 3
        - 강점 5 + 다음 단계 5
        - 이번 달 핵심 TODO 5개 (제목만, 본문은 학생 명시 동의 시만)
        - 결정 포인트 D-N
      하단 면책 + "본 자료는 자녀와의 대화 시작용입니다."
    </one_pager>
    <actions>
      [PDF 다운로드] (jspdf), [공유 링크 만들기] (ShareLink), [공유 링크 취소]
    </actions>
    <empty_state>학생이 아직 PDF를 업로드하지 않았을 때: "자녀에게 PDF 업로드를 요청하세요. 공유 링크: ..."</empty_state>
  </parent_report_page>

  <shared_report_page>
    경로 `/share/[token]`. 비로그인 가능.
    학부모 리포트와 동일 레이아웃, 단 모든 액션 비활성. 우측 상단 "읽기 전용" 배지.
    토큰 유효성/만료/취소 시 "이 링크는 더 이상 유효하지 않습니다" 빈 상태.
    ShareLink.includeTodoBody/includeSchoolType/includeRegion 플래그에 따라 노출 항목 가변.
  </shared_report_page>

  <legal_pages>
    경로 `/legal/terms`, `/legal/privacy`. 비로그인 가능 정적 페이지.
    텍스트 소스: `src/content/legal/terms.ko.mdx` + `src/content/legal/privacy.ko.mdx` (Markdown 파일, MDX 컴포넌트로 렌더). 변경 이력은 git으로 추적 + 페이지 상단에 "최종 개정: YYYY-MM-DD" 표시.
    레이아웃: max-width 720px 중앙, 본문 16px line-height 1.7, 헤딩 24/20/16px, 표 단순 1px border.
    모바일에서는 좌우 패딩 16px. 인쇄용 @media print 스타일 제공.
    하단: [이전 버전 보기] 링크 (git 이력 기반 정적 빌드 시점 스냅샷 5개).
    초기 텍스트 작성 책임: 법무 자문 검토 후 spec-writer/도메인 전문가가 첫 draft 작성. 출시 전 외부 법무 검토 1회.
  </legal_pages>

  <privacy_settings_page>
    경로 `/settings/privacy`.
    <sections>
      1. PDF 관리: 업로드 이력 + 만료까지 + [지금 삭제] (확인 모달)
      2. 데이터 내보내기: [JSON 내보내기] (서명 URL 1시간 유효)
      3. 동의 관리: 각 동의 항목 토글 (재동의 / 철회)
      4. 학부모 공유: 공유 링크 목록 + 취소 / 새로 발급
      5. 계정 삭제: [계정 삭제 요청] (30일 유예 안내 모달, 이메일 OTP 재인증 필수)
      6. 열람 로그: 최근 30건 AuditLog (action, 시각)
    </sections>
    <empty_state>각 항목 별 ("아직 업로드한 PDF가 없어요" 등)</empty_state>
  </privacy_settings_page>

  <account_settings_page>
    경로 `/settings/account`.
    이메일 변경(OTP 재인증), 비밀번호 설정/변경, 알림 설정(이메일 on/off, 푸시 v2), 다크모드 토글.
  </account_settings_page>

  <keyboard_shortcuts_reference>
    전역:
      - `?` : 단축키 도움말 모달
      - `g d` : Dashboard 이동
      - `g t` : Today 이동
      - `g r` : Roadmap (현재 학기) 이동
      - `g c` : Checkin
      - `g p` : Profile
      - `Esc` : 모달 닫기
    TODO 상세:
      - `s` : 시작
      - `c` : 완료 (회고 모달)
      - `z` : 스누즈 (드롭다운)
      - `x` : 스킵 (사유 모달)
      - `e` : 노트 편집 포커스
    Roadmap 그리드:
      - `←/→` : 학기 이동
      - `↑/↓` : 학년 이동
      - `Enter` : 학기 상세 진입
  </keyboard_shortcuts_reference>

</pages_and_interfaces>

<core_functionality>

  <auth_and_consent>
    - 이메일+OTP 회원가입/로그인 (Resend 발송, 6자리 5분 만료)
    - 카카오·구글 OAuth (선택)
    - 만 14세 미만 자동 감지 + 보호자 동의 흐름 (consentToken 7일 만료)
    - 분리 동의 (필수 1 + 선택 2) — 필수 미체크 시 PDF 업로드 차단
    - 세션: HttpOnly 쿠키 JWT, 30일 sliding, 로그아웃 시 jti blacklist
    - 계정 삭제 30일 유예 + 모든 데이터 hard delete cron
  </auth_and_consent>

  <pdf_upload_and_parsing>
    - 단일 PDF 업로드, 최대 20MB, MIME application/pdf 검증 + magic byte (`%PDF-`) 검증
    - sha256 계산하여 동일 학생의 이전 업로드와 중복 시 기존 PdfFile 재사용
    - 클라이언트 → S3 직접 업로드(presigned URL) 또는 Server Action 경유 (MVP는 Server Action 경유, 25MB body limit)
    - 서버에서 AES-256-GCM 암호화 후 S3 저장 (key=PDF_ENCRYPTION_KEY)
    - BullMQ 큐에 parsePdf 잡 enqueue
    - parsePdf 워커 단계:
        1. S3 다운로드 + 복호화
        2. pdf-parse로 텍스트 추출 시도 (timeout 10초)
        3. 추출 성공 판별 (둘 중 하나 충족):
           (a) 추출 텍스트 길이 ≥ 200자 AND 한글 문자 비율 ≥ 25%
           (b) 한글 비율 ≥ 10% AND anchors.ts의 NEIS 앵커 키워드 ("인적사항","학적사항","출결","수상경력","교과학습","세부능력","행동특성") 중 2개 이상 매칭
           이는 자격증·영문 페이지 등으로 한글 비율이 일시 낮아도 NEIS 양식임이 확실하면 텍스트 추출로 진행하기 위함 (false negative 회피).
        4. 실패 시 OCR 폴백: pdfjs-dist로 페이지 PNG 렌더 → Claude Vision multimodal 호출 (페이지당 1회, 최대 30 페이지)
        5. anchors.ts: §1.2 키워드 매칭으로 항목별 분리
        6. mask.ts: 정규식·컨텍스트로 PII 자동 마스킹/폐기
        7. ParsedReportCard 저장 (extractionConfidence 항목별 평균)
        8. PdfFile.parseStatus 갱신
    - 진행 상태는 Pusher 대신 Server-Sent Events 또는 폴링(2초 interval) — MVP는 폴링
    - 수동 보정: 사용자 편집 시 manualCorrected=true, sections JSONB 부분 패치
  </pdf_upload_and_parsing>

  <pii_masking_and_disposal>
    CRITICAL: 다음 항목은 ParsedReportCard.sections 또는 어디에도 저장되지 않는다.
      - 주민등록번호 앞 6자리 (정규식 `\d{6}-\d{7}` 또는 `\d{6}-?\*+`): 매칭 시 즉시 폐기
      - 보호자 성명·관계·직업 (anchor "보호자")
      - 가족관계 (anchor "가족사항")
      - 주소 상세 (anchor "주소"의 세부 → 시도 단위만 추출)
      - 학교명 (anchor "학교"): meta.schoolNameMasked=true, 학교 유형만 별도 보관
      - 학교폭력 조치사항 (anchor "학교폭력"): 항목 자체를 sections에 미포함
      - 보건·신체발달 (anchor "보건"): 미포함
      - 출결 사유 텍스트 (anchor "출결"의 사유 컬럼): 카운트만 추출
    마스킹 단위 테스트 필수: rules/sample_pii_test_corpus 으로 회귀 테스트.
  </pii_masking_and_disposal>

  <signal_extraction_and_profile>
    - 정량 신호:
        gradeAvgMain (국·수·영·사/과 평균 등급), gradeAvgMinor, gradeTrend (학기별 회귀 기울기)
        attendanceUnexcusedTotal, attendanceClean (boolean)
        clubConsistency (학년간 동일 분야 비율 0~1), volunteerSchoolHours
    - 정성 신호 (LLM 1회 호출):
        subjectRecords[].text → keywords[] (긍정 키워드 / 보완 키워드 분리)
        comprehensiveOpinion.text → positiveKeywords[], improvementKeywords[]
        freeSemesterOutputs[].topic → category 분류 (8 enum)
    - 흥미 가설:
        가중치: 자유학기 ×5 + 동아리 일관성 ×4 + 수상 분야 ×3 + 독서 분야 ×3
        topic별 score 합산 → top 3, confidence = score / max_possible
    - 트랙 가설: services/signal/tracks.ts (§3.2 매트릭스 룰)
    - StudentProfile 새 버전(version+1) 생성. 이전 버전은 보존(이력).
  </signal_extraction_and_profile>

  <todo_generation>
    - seedLoader: rules/seed_todos.{examCycle}.json 로드 + track_matrix.json 적용
    - 시드 필터:
        grade ∈ {1,2,3} 모두 포함
        track == primary || secondary || null(공통)
        시기 부적합 (예: 고3 11월에 고1 학평 추천 X) → state=snoozed로 자동
    - 우선순위 계산: priority.ts
        score = timeUrgency*0.4 + gradeStageFit*0.2 + trackFit*0.2 + weaknessFit*0.1 + interestFit*0.1
        timeUrgency: due_date까지 일수를 0~100 변환 (D-7 이내=100, D-30=70, D-90=30, 없음=20)
        gradeStageFit: (todo.grade == student.currentGrade) ? 100 : (인접 학년)50 : 0
        trackFit: 카테고리가 트랙별 가중표(track_matrix.weights)에서 매칭
        weaknessFit: profile.weaknesses 키워드와 카테고리 매핑
        interestFit: profile.interestHypothesis와 카테고리 매핑
    - LLM 개인화 (services/recommendation/personalize.ts):
        Claude `claude-haiku-4-5` 호출 (temperature 0.3, max_tokens 1000, 환경변수 `LLM_PERSONALIZE_MODEL`로 오버라이드 가능)
        시스템 프롬프트: "당신은 한국 입시 전문 진학 코치입니다. 학생을 격려하는 톤으로...", 가드레일 명시
        입력: 시드 템플릿(title, description) + StudentProfile 요약 + 컨텍스트
        출력 JSON: { title, description, explanation } — Zod 검증
        실패/타임아웃(12초) 시 시드 원문 사용 (fallback)
    - 배치 호출: 시드 30~80개를 5개씩 배치하여 호출 (병렬 5)
    - 톤 가드 (guardrails.ts): 출력에 다음 패턴 발견 시 시드로 fallback:
        "학원", "과외", "사교육", "합격을 보장", "확실히 합격", "반드시 ~ 해야"
        "부족", "못 한다", "약하다" → "다음 단계로", "더 키울 수 있는 부분" 등 자동 치환
    - 결과 Todo[] 일괄 INSERT (단일 트랜잭션)
  </todo_generation>

  <today_balance>
    - 매일 자정 KST: 모든 활성 학생에 대해 오늘 5개 후보 산출 (Worker)
    - 알고리즘:
        1. state ∈ {backlog, snoozed and snoozeUntil ≤ today, today} 후보 필터
        2. priorityScore desc 정렬
        3. 카테고리 A,B,C,D,E 그룹별 top 1씩 선택 (5개)
        4. 5개 미만이면 priorityScore desc로 추가 보충
        5. 선택된 TODO state=today 업데이트
    - 사용자가 직접 today에 추가/제거 가능 (Server Action)
  </today_balance>

  <weekly_checkin>
    - 매주 일요일 18:00 KST: 활성 학생에게 이메일 ("이번 주 체크인 5분")
    - 사용자가 /checkin 진입 시:
        지난 주 (월~일) state 변화 집계
        WeeklyCheckIn 레코드 생성/업데이트
        다음 주 5개 미리 계산 (today_balance와 동일 로직)
    - retroText는 다음 주 priority 보정에 사용 (LLM 1회 추가 호출)
  </weekly_checkin>

  <track_management>
    - 사용자가 /onboarding/track 또는 /profile에서 트랙 수동 변경 가능
    - 변경 시 새 TrackRecommendation 생성 (userOverride*=설정), 이후 TODO 재생성 트리거
    - 학년/학기 전환 (cron 매월 1일): 학생 currentGrade/currentTerm 자동 갱신, TODO 우선순위 재계산
  </track_management>

  <parent_sharing>
    - 학생이 /settings/privacy에서 [학부모 공유 링크 만들기] → ShareLink 생성 (audience=parent_summary)
    - 토큰 32자 url-safe (crypto.randomBytes(24).toString('base64url'))
    - 만료 30일, 학생이 언제든 [취소] 가능
    - 학부모는 가입 없이 /share/[token] 열람 (또는 가입하여 정식 ParentChildLink 연결)
    - 공유 링크 열람 시 AuditLog action=share_link_view 기록 (actorUserId=null, ip 저장)
  </parent_sharing>

  <data_export_and_deletion>
    - 데이터 내보내기 권한 정책 (CRITICAL):
        - role=student: 본인 Student와 그 하위 데이터 전부 export 가능.
        - role=parent: export Server Action 차단(403). 학부모는 /parent/report와 ShareLink 다운로드만 가능 — 자녀 본인 동의 범위를 넘는 내보내기를 막기 위함.
    - 데이터 내보내기 JSON 스키마 (top-level keys):
        ```
        {
          "exportVersion": "1.0",
          "generatedAt": "ISO-8601",
          "user": { id, email, role, birthYear, locale, createdAt },
          "students": [
            {
              student: { id, displayName, birthYear, schoolType, schoolLevel, currentGrade, currentTerm, region, entranceYear, examCycle, trackPreference },
              parsedReportCards: [ { id, schoolLevel, grade, term, schoolYear, sections (마스킹 후), extractionConfidence, manualCorrected } ],
              studentProfiles: [ { id, version, interestHypothesis, strengths, weaknesses, trackHypothesis, signalSummary, generatedAt, generatedBy } ],
              trackRecommendations: [ { id, primaryTrack, secondaryTrack, confidenceLabel, signalSummary, userOverrideTrack, userOverrideReason, generatedAt } ],
              todos: [ { id, category, grade, term, title, description, explanation, priorityScore, state, dueDate, completedAt, snoozeUntil, skipReason, source, sourceRuleId, track, tags, createdAt, updatedAt } ],
              weeklyCheckIns: [ { id, weekStart, weekEnd, completedCount, missedCount, snoozedCount, skippedCount, retroText, satisfactionScore, generatedAt } ],
              shareLinks: [ { id, audience, expiresAt, revokedAt, viewCount, lastViewedAt, includeTodoBody } ]   // token은 보안상 제외
            }
          ],
          "auditLogs": [ { id, actorType, action, occurredAt, meta } ]   // 최근 30건
        }
        ```
        - PdfFile은 export에 미포함 (원본 PDF는 사용자가 직접 보관). 단 PdfFile 메타(id, uploadedAt, expiresAt, ocrUsed, parseStatus)는 students[].pdfFiles에 포함.
        - 모든 PII 마스킹 정책은 export에도 동일 적용 (학교명·주소상세 미포함).
    - signed URL 정책: 1시간 유효, 다회 다운로드 허용 (사용자 편의). URL은 AuditLog.meta에 기록 안 함, 응답 1회만 반환. exports/{userId}/{timestamp}.json 객체는 25시간 후 cron이 일괄 삭제.
    - PDF 즉시 삭제: PdfFile S3 객체 + DB 레코드 hard delete (ParsedReportCard.sourcePdfId는 SET NULL)
    - 계정 삭제: deletedAt 설정 → 30일 cron이 hard delete
  </data_export_and_deletion>

  <cron_jobs>
    공통 정책 (CRITICAL: Vercel Function 30초 한계 회피):
      - 모든 cron은 1회 호출당 최대 50명/배치 청크 처리 후 다음 cursor를 응답으로 반환. Vercel Cron이 같은 분에 여러 번 호출되도록 cron 표현식을 짧게 분할하거나, BullMQ로 큐 enqueue 후 즉시 200 반환 (실제 처리는 워커).
      - 각 cron 시작 시 `traceId`(uuid) 발급, 처리 건수·소요 시간을 AuditLog.meta(actorType=system) 또는 별도 JobRunLog에 기록.
      - 실패 시 Sentry severity=critical + 운영자 이메일 알림.
    - cleanup-pdfs: 매일 00:00 KST. expiresAt &lt; now() PdfFile 일괄 hard delete (S3 + DB). 청크 50개씩, traceId 기록.
    - cleanup-audit-logs: 매일 00:30 KST. occurredAt &lt; now() - 1y AuditLog 삭제. 청크 1000개씩.
    - cleanup-shares: 매일 01:00 KST. expiresAt &lt; now() ShareLink revokedAt 설정.
    - cleanup-exports: 매일 01:30 KST. exports/ 객체 25시간 초과 시 삭제.
    - account-hard-delete: 매일 02:00 KST. deletedAt &lt; now() - 30d User 및 종속 데이터 hard delete. 청크 10명씩 (각 사용자당 다수 row 삭제).
    - parent-consent-reminder: 매일 03:00 KST. 보호자 동의 미완료 ParentChildLink (consentToken 발급 후 6일 경과)의 학생에게 reminder 이메일 ("내일 동의 만료. 부모님께 다시 부탁드려보세요").
    - parent-consent-cleanup: 매일 03:30 KST. consentToken 발급 후 7일 경과 ParentChildLink 삭제 + 학생 계정 자동 삭제 (가입 7일 미완료 시).
    - weekly-checkin-email: 매주 일요일 18:00 KST. 활성 학생 대상 Resend 발송. 청크 100명씩.
    - today-balance: 매일 04:00 KST. 활성 학생 today 5개 prebuilt. 청크 50명씩, BullMQ enqueue 패턴 권장.
    - grade-rollover: 매월 1일 00:00 KST. 학년/학기 자동 갱신 (3월 1일은 학년 변경). 청크 100명씩.
    - age-threshold-check: 학생 14세 도달 감지는 cron 대신 로그인 시 lazy 체크 (consent_renewal_required 모달 표시) — 인프라 단순화.
  </cron_jobs>

  <decision_point_engine_data>
    데이터 출처 (CRITICAL):
      - 학사일정 + 입시 이벤트는 `rules/academic_calendar.{cycle}.json`에 외부화. 구조:
        ```
        [{
          "id": "subject_choice_g1",
          "exam_cycle": "CYCLE_2028",
          "title": "고1 → 고2 선택과목 결정",
          "category_hint": "D4",
          "due_anchor": { "type": "school_year_start", "offset_months": 5 },  // 9월 말 기준
          "lead_time_days": [60, 30, 7, 1],                                    // D-N 강조 시점
          "applies_to_grade": [1],
          "applies_to_track": null
        }, ...]
        ```
      - DecisionPointBanner 컴포넌트는 매 페이지 진입 시 학생 currentGrade·currentTerm·examCycle로 필터링하여 "가장 임박한 1건" 표시.
      - 시드 TODO와 별개로 운영. Todo의 dueDate와 일치하는 항목은 동일 표시 가능하나 중복 카운트 방지.
      - 매년 입시 정책 변경 시 academic_calendar 파일만 갱신 (코드 배포 X).
  </decision_point_engine_data>

</core_functionality>

<error_handling>
  <user_facing>
    <toast_notifications>
      - Success: #10B981 배경, 흰 텍스트, 3초 자동 사라짐, 우측 하단
      - Error: #EF4444, 사용자가 닫을 때까지 유지, 우측 하단, "다시 시도" 버튼 옵션
      - Warning: #F59E0B, 5초 자동
      - Info: #3B82F6, 3초 자동
      - 동시 최대 3개 stack, 오래된 것부터 사라짐
    </toast_notifications>
    <form_validation>
      - 인라인 에러: 필드 아래 13px #EF4444, blur 시점에 표시(키 입력 중 X)
      - 제출 시 첫 에러로 스크롤
      - 잘못된 제출 시 200ms shake 애니메이션
      - Zod 메시지를 한국어로 매핑 (예 z.string().email() → "올바른 이메일을 입력해주세요")
    </form_validation>
    <error_pages>
      - 404: 일러스트 + "찾으시는 페이지가 없어요" + [대시보드로]
      - 500: "잠시 문제가 발생했어요" + [다시 시도] + Sentry id 표시
      - 403: "접근 권한이 없어요" + [홈으로]
      - 410 (만료된 공유 링크): "이 링크는 더 이상 유효하지 않습니다"
    </error_pages>
    <pdf_specific_errors>
      - 파일 형식: "PDF 파일만 업로드 가능합니다"
      - 크기 초과: "20MB 이하로 업로드해주세요"
      - 텍스트 추출 실패 + OCR 자동 폴백: 진행 카드에 "스캔본 감지, OCR 처리 중..." 노란 강조
      - OCR 실패: 모달 "자동 인식에 실패했어요" + [수동 입력] / [재시도] / [다른 PDF 업로드]
      - PII 감지 후 마스킹 시: 알림 인포 ("주민번호 등 민감 정보를 자동으로 제거했어요")
      - 학폭/보건 항목 감지: 알림 인포 ("학교폭력·보건 항목은 시스템 정책에 따라 분석에 사용되지 않습니다")
    </pdf_specific_errors>
  </user_facing>
  <error_boundaries>
    - app/(app)/layout.tsx에 React Error Boundary 적용, 페이지별 fallback UI
    - "잠시 문제가 발생했어요" + [새로고침] + Sentry event id (개발 환경은 stack trace)
  </error_boundaries>
  <api_errors>
    - 네트워크 실패: 지수 백오프 재시도 1s/2s/4s (TanStack Query retry 3회)
    - 401: 즉시 로그아웃 + /auth/login?next=현재경로
    - 403: 권한 없음 토스트 + 대시보드로
    - 429 Rate limit: "요청이 많습니다. 잠시 후 다시 시도해주세요" 토스트
    - 5xx: 일반 에러 토스트 + Sentry 캡처
    - Anthropic API 실패: services/recommendation/personalize.ts에서 시드 원문 fallback (사용자에게는 노출 X)
    - S3 실패: 업로드 시 "업로드 실패. 다시 시도해주세요" + 재시도 1회 자동
  </api_errors>
  <error_classification_for_logging>
    Sentry tag로 분류:
      - category: auth | pdf_parse | ocr | llm | db | s3 | rate_limit | unknown
      - severity: info | warn | error | critical
      - PII 포함 가능성 있는 컨텍스트는 자동 스크럽 (Sentry beforeSend)
  </error_classification_for_logging>
</error_handling>

<third_party_integrations>

  <integration name="Anthropic Claude">
    <purpose>신호 NLP 추출, TODO 개인화, 회고 보정 추천</purpose>
    <sdk>@anthropic-ai/sdk v0.27</sdk>
    <model>
      - 1차 모델: `claude-haiku-4-5` (개인화·회고 등 높은 호출량) — 저비용·저지연 우선
      - 2차 모델 (조건부): `claude-sonnet-4-7` — signal extraction 1회만 (정성 NLP 정확도 필요할 때). 환경변수 `LLM_PERSONALIZE_MODEL`로 전환 가능.
      - CRITICAL: 모델 ID는 환경변수로 관리하여 신모델 출시 시 코드 변경 없이 갱신. 본 spec의 ID는 작성 시점(2026-05) 가정값 — 배포 전 Anthropic 공식 ID 검증 필수.
    </model>
    <usage_pattern>
      - signal extraction: 1회 / 학생 / 새 PDF 업로드 시 (sonnet-4-7, in ~4000 / out ~3000 tokens)
      - personalize batch: 6~16회 / 학생 / 새 PDF (haiku-4-5, 시드 5개 묶음 1회당 in ~3000 / out ~1500 tokens)
      - weekly retro: 1회 / 학생 / 주 (haiku-4-5, in ~1500 / out ~500 tokens)
      - guardrail check: 결정적 정규식, LLM 호출 X
    </usage_pattern>
    <cost_calculation>
      가격 가정 (2026-05 기준, Anthropic 공식 단가 확인 필수):
        - haiku-4-5: $1/1M input, $5/1M output. prompt caching 읽기 90% 할인.
        - sonnet-4-7: $3/1M input, $15/1M output. prompt caching 읽기 90% 할인.
      월간 토큰 (학생 1인 가정 — 신규 가입 후 1개월):
        - signal: in 4000 + out 3000 (sonnet) → $0.012 + $0.045 = $0.057
        - personalize: 12회 평균 × (in 3000 cached × 90% 할인 = effective 300 + out 1500) (haiku) → 12 × ($0.0003 + $0.0075) = $0.094
        - weekly retro: 4회 × (in 1500 + out 500) (haiku) → 4 × ($0.0015 + $0.0025) = $0.016
        - 합계: 약 $0.167 ≈ ₩230/학생/월 (KRW 1380 환율)
      → 가설 ₩200을 약간 초과. 청사진 §11.4 A5 가정 수정 필요 (₩200 → ₩250).
      ₩250 이내 달성 전략 (모두 코드에 반영):
        1. Prompt caching 적극 활용 (system prompt + 시드 템플릿 5분 TTL): personalize 입력 90% 할인.
        2. signal extraction을 sonnet → haiku로 다운그레이드 (정확도 영향 측정 후 결정, A/B).
        3. personalize 출력 max_tokens 1500 → 1000 제한 (시드 원문이 표준이라 큰 손실 없음).
        4. weekly retro는 LLM 호출 → 룰 기반 키워드 매칭으로 대체 가능 (부정 키워드 감지).
        5. 신규 가입 후 첫 달만 비용 집중 (이후는 weekly retro만), 월평균은 더 낮음.
    </cost_calculation>
    <cost_target>학생당 월 ₩250 이내 (≈ $0.18). 청사진 ₩200 가설은 spec 작성 단계에서 ₩250로 정정.</cost_target>
    <prompt_caching>system prompt + 시드 템플릿 + StudentProfile 요약은 prompt caching 활용 (5분 TTL, cache_control: ephemeral)</prompt_caching>
    <timeouts>요청 12초, 재시도 1회 (지수 백오프 1s)</timeouts>
    <fallback>실패/타임아웃 시 시드 원문 그대로 사용 (사용자 경험 보존)</fallback>
  </integration>

  <integration name="NextAuth.js (Auth.js v5)">
    <purpose>인증 (이메일+OTP, 카카오/구글 OAuth)</purpose>
    <sdk>next-auth v5.0.0-beta or stable</sdk>
    <providers>email-otp (custom), KakaoProvider, GoogleProvider</providers>
    <session>JWT, HttpOnly cookie, 30일</session>
  </integration>

  <integration name="Resend">
    <purpose>이메일 발송 (OTP, 보호자 동의 링크, 주간 체크인, 공유 알림)</purpose>
    <sdk>resend v3.x</sdk>
    <templates>
      - otp_signup: 6자리 코드, 5분 만료
      - parent_consent_link: 7일 유효 토큰 링크
      - weekly_checkin: 일요일 18:00 발송
      - share_created: 학부모에게 공유 링크 발송 (학생이 보낼 때)
      - account_deletion_scheduled: 30일 유예 안내
    </templates>
    <rate_limit>sender 단위 60/min</rate_limit>
  </integration>

  <integration name="S3 호환 스토리지 (Cloudflare R2 또는 AWS S3)">
    <purpose>원본 PDF 암호화 저장 + 데이터 내보내기 임시 객체</purpose>
    <sdk>@aws-sdk/client-s3 v3.x + @aws-sdk/s3-request-presigner</sdk>
    <bucket_layout>
      - pdfs/{studentId}/{uuid}.pdf.enc (30일 만료)
      - exports/{userId}/{timestamp}.json (1시간 유효)
    </bucket_layout>
    <encryption>application-layer AES-256-GCM 후 storage 측 SSE-S3 또는 R2 자동 암호화</encryption>
    <cors>업로드 origin은 NEXT_PUBLIC_APP_URL만 허용</cors>
  </integration>

  <integration name="BullMQ + Upstash Redis">
    <purpose>비동기 잡 큐 (PDF 파싱, 로드맵 생성, cron)</purpose>
    <sdk>bullmq v5.10</sdk>
    <queues>
      - pdf-parse (concurrency 5, retry 2회)
      - generate-roadmap (concurrency 3)
      - email-send (concurrency 10)
      - cleanup (concurrency 1, scheduled)
    </queues>
    <worker_deployment>Vercel은 long-running worker 부적합 → 별도 Fly.io 또는 Railway에 worker 프로세스 배포 (선택지). MVP에서는 Vercel Cron + Serverless Function 트리거로 큐를 즉시 처리(짧은 실행) 패턴 채택.</worker_deployment>
  </integration>

  <integration name="Sentry">
    <purpose>에러 추적 + 성능 모니터링</purpose>
    <sdk>@sentry/nextjs v8.x</sdk>
    <config>tracesSampleRate 0.1 (프로덕션), 1.0 (스테이징). beforeSend로 PII 스크럽.</config>
  </integration>

  <integration name="PostHog">
    <purpose>제품 분석 (퍼널, 리텐션, 기능 플래그)</purpose>
    <sdk>posthog-js v1.x</sdk>
    <events>signup_started, signup_completed, consent_given, pdf_uploaded, parse_completed, track_confirmed, todo_state_changed, checkin_completed, share_created</events>
    <pii_policy>이메일·이름 미전송, distinct_id는 User.id 해시</pii_policy>
  </integration>

  <integration name="Vercel Cron">
    <purpose>cron 트리거 (cleanup, weekly-checkin, today-balance)</purpose>
    <auth>cron-secret-header (CRON_SECRET 환경 변수, X-Cron-Secret 헤더 검증)</auth>
  </integration>

  <integration name="Pretendard (Self-host)">
    <purpose>한글 가독성 폰트</purpose>
    <delivery>public/fonts/PretendardVariable.woff2, font-display: swap</delivery>
  </integration>

</third_party_integrations>

<aesthetic_guidelines>
  <design_fusion>모던 한국어 UX × 차분한 코칭 톤. 정보 밀도는 점진적 노출(격자→학기→TODO 카드 깊이별). 색상은 카테고리 식별을 보조하지만 정보 전달의 유일 수단이 아님(아이콘+라벨 동반).</design_fusion>

  <color_palette>
    <primary_colors>
      - Primary Blue: #1F4ED8 - 주요 CTA, 포커스, 활성 상태
      - Primary Hover: #1E40AF - 버튼 hover
      - Indigo Accent: #6366F1 - 보조 트랙 카드, 보조 강조
    </primary_colors>
    <background_colors>
      - Page BG (Light): #F9FAFB
      - Card BG (Light): #FFFFFF
      - Subtle BG: #F3F4F6 - 사이드바, 비활성 영역
      - Page BG (Dark): #0B0E14
      - Card BG (Dark): #1F2937
    </background_colors>
    <text_colors>
      - Heading: #111827 (Light) / #F3F4F6 (Dark)
      - Body: #374151 (Light) / #D1D5DB (Dark)
      - Muted: #6B7280 (Light) / #9CA3AF (Dark)
      - Disabled: #9CA3AF (Light) / #4B5563 (Dark)
    </text_colors>
    <status_colors>
      - Success: #10B981 - 완료, 동의 성공
      - Warning: #F59E0B - 임박 결정, 신뢰도 중간
      - Error: #EF4444 - 검증 오류, 신뢰도 낮음
      - Info: #3B82F6 - 안내 토스트
    </status_colors>
    <category_colors>
      - A 내신·교과: #2563EB (파랑)
      - B 수능·모의: #F97316 (주황)
      - C 세특·창체: #16A34A (초록)
      - D 입시·결정: #7C3AED (보라)
      - E 생활·멘탈: #6B7280 (회색)
      각 카테고리는 색 + 아이콘 + 한글 라벨로 항상 동반 표시.
      A 아이콘 BookOpen, B Target, C Sparkles, D Compass, E Heart.
    </category_colors>
    <track_colors>
      - 정시 (jeongsi): #DC2626
      - 학종 (hakjong): #1F4ED8
      - 교과 (gyogwa): #0891B2
      - 논술 (nonsul): #7C3AED
      - 실기 (silgi): #DB2777
      - 미정 (undecided): #6B7280
    </track_colors>
    <dark_theme>
      Page BG #0B0E14, Card #1F2937, Border #374151, Primary Blue 동일 #1F4ED8 유지(채도 보정 X).
      카테고리 다크 토큰 (Tailwind config에 별도 등록):
        - category-A-dark: #3B82F6
        - category-B-dark: #FB923C
        - category-C-dark: #22C55E
        - category-D-dark: #A78BFA
        - category-E-dark: #9CA3AF
      라이트 토큰: 본 섹션 category_colors 참조. 컴포넌트에서 `bg-category-a` 클래스가 dark variant 자동 적용.
    </dark_theme>
  </color_palette>

  <typography>
    <font_families>
      - Sans: "Pretendard Variable", "Pretendard", -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Noto Sans KR", sans-serif
      - Mono: "JetBrains Mono", "D2Coding", monospace
    </font_families>
    <font_sizes>
      - Display (랜딩 hero): 40px / 모바일 28px, weight 700, line-height 1.2
      - H1 (페이지 타이틀): 28px, weight 700, line-height 1.3
      - H2 (섹션): 20px, weight 600, line-height 1.4
      - H3 (카드 제목): 16px, weight 600, line-height 1.5
      - Body Large: 16px, weight 400, line-height 1.6 (모바일 최소)
      - Body: 14px, weight 400, line-height 1.6
      - Caption: 13px, weight 400, line-height 1.5 (인라인 에러, 메타)
      - Small: 12px, weight 400 (면책 바, 타임스탬프)
    </font_sizes>
    <line_heights>본문 1.6, 헤딩 1.2~1.4, 캡션 1.5</line_heights>
  </typography>

  <spacing>
    8pt grid 시스템.
    토큰: space-1=4px, space-2=8px, space-3=12px, space-4=16px, space-5=20px, space-6=24px, space-8=32px, space-10=40px, space-12=48px, space-16=64px.
    카드 내부 패딩 default 16px, 데스크톱 큰 카드 24px. 페이지 좌우 마진 모바일 16px / 태블릿 24px / 데스크톱 32px (max-content-width 1200px 중앙).
  </spacing>

  <borders_and_shadows>
    <borders>
      - 기본 두께 1px, 색 #E5E7EB (Dark #374151)
      - 강조 두께 2px, 색 #1F4ED8
      - radius: sm 4px (배지), md 8px (input), lg 12px (카드/버튼), xl 16px (큰 카드/모달), full 9999px (chip)
    </borders>
    <shadows>
      - shadow-card: 0 1px 3px rgba(0,0,0,0.05), 0 1px 2px rgba(0,0,0,0.03)
      - shadow-dropdown: 0 4px 12px rgba(0,0,0,0.08)
      - shadow-modal: 0 20px 40px rgba(0,0,0,0.15)
      - shadow-focus: 0 0 0 3px rgba(31,78,216,0.30)
    </shadows>
  </borders_and_shadows>

  <component_styling>
    <buttons>
      높이 sm 36px / md 44px / lg 56px. 패딩 좌우 sm 12px / md 16px / lg 20px.
      Primary: bg #1F4ED8, text #FFFFFF, hover #1E40AF, active #1E3A8A, disabled #9CA3AF + opacity 0.6
      Secondary: bg #FFFFFF, border 1px #1F4ED8, text #1F4ED8
      Ghost: bg transparent, text #1F4ED8, hover bg #EFF6FF
      Destructive: bg #EF4444, text #FFFFFF
      Loading: 좌측 16px Spinner + 텍스트 변경 ("처리 중...")
      Min touch target 44×44 보장 (모바일).
    </buttons>
    <inputs>
      높이 44px (모바일 16px font-size로 줌 방지), 좌우 패딩 12px, border 1px #D1D5DB, focus border 2px #1F4ED8 + shadow-focus, error border 2px #EF4444.
      Label 위 8px, 13px #6B7280. Helper text 아래 4px, 12px #6B7280.
    </inputs>
    <chips>
      높이 28px, 패딩 좌우 12px, radius full. 활성 시 카테고리 색 + 흰 텍스트, 비활성 시 #F3F4F6 배경 + #6B7280 텍스트.
    </chips>
    <cards>
      bg #FFFFFF, border 1px #E5E7EB, radius 12px, padding 16px (md) / 24px (lg).
      hover 시 shadow-card → shadow-dropdown 전환 (200ms).
    </cards>
    <modals>
      backdrop rgba(0,0,0,0.5) blur 4px. 컨테이너 max-width 480px, radius 16px, shadow-modal, padding 24px.
      모바일 ≤640px에서는 bottom-sheet 형태 (높이 자동, 상단 radius 16px).
    </modals>
    <badges>
      높이 20px, 패딩 좌우 8px, font 12px weight 600, radius full.
    </badges>
    <progress_bars>높이 4px (기본) / 8px (강조), bg #E5E7EB, fill #1F4ED8, radius full.</progress_bars>
  </component_styling>

  <animations>
    <micro_interactions>
      - 버튼 hover scale 1.02, 150ms ease-out
      - 카드 hover translateY -2px + shadow 확장, 200ms
      - 체크박스 체크 애니메이션 200ms ease-in-out
      - 토스트 enter slide-up 200ms / exit fade 150ms
    </micro_interactions>
    <page_transitions>없음 (인지부하). 단 온보딩 단계 변경 시 fade 150ms.</page_transitions>
    <loading_states>
      - 스켈레톤: 1.5s shimmer (linear-gradient sweep)
      - 스피너: 16px lucide Loader2 회전 1s linear infinite
      - 진행 단계: 단계별 체크 0.3s pop (scale 0.8 → 1)
    </loading_states>
    <orchestrated_entrance>대시보드 첫 진입 시 격자 셀 stagger fade-in (각 셀 +50ms 지연, 총 max 600ms).</orchestrated_entrance>
  </animations>

  <responsive_design>
    <breakpoints>
      - mobile: 0–639px (단일 컬럼, 하단 탭 56px, 카드 풀폭, 16px 좌우 패딩)
      - tablet: 640–1023px (2-column 일부, 사이드바 collapsible drawer)
      - desktop: 1024–1279px (사이드바 240px 고정, 메인 70/30 분할)
      - wide: 1280px+ (max-content-width 1200px 중앙 정렬)
    </breakpoints>
    <mobile_adaptations>
      - 사이드바 → 하단 탭 4개 (56px)
      - 학년×학기 격자 → 가로 스크롤 (현재 학기 자동 스크롤 위치)
      - 모달 → 바텀시트 (slide-up)
      - PDF 미리보기 ↔ 추출 결과 → 탭 전환 (좌/우 분할 X)
      - 키보드 단축키 hint 숨김 (`?` 모달은 비활성)
    </mobile_adaptations>
    <touch_interactions>
      - TODO 카드 좌측 스와이프(80px) → [완료] 빠른 액션
      - TODO 카드 우측 스와이프(80px) → [스누즈] 빠른 액션
      - 풀투리프레시: 대시보드, 오늘, 로드맵 (60px threshold)
      - 길게 누르기 500ms → 다중 선택 모드 (로드맵 학기 상세)
      - 최소 탭 타겟 44×44px (WCAG 2.5.8)
    </touch_interactions>
  </responsive_design>

  <icons>
    lucide-react v0.395 단일 라이브러리. 기본 24px, stroke 2, color는 컨텍스트 색 상속.
    네비 아이콘 28px, 토스트 아이콘 20px, 칩 내부 아이콘 14px.
  </icons>

  <accessibility>
    - WCAG 2.1 AA 준수
    - 색상 대비 최소 4.5:1 (본문) / 3:1 (큰 텍스트)
    - 모든 인터랙티브 요소 키보드 탐색 가능, focus ring 가시성 (shadow-focus)
    - aria-live="polite" — TODO 상태 변경, 토스트
    - aria-label — 아이콘 only 버튼은 필수
    - role/landmark — main, nav, complementary 적절히
    - 다크모드 + prefers-color-scheme 지원
    - prefers-reduced-motion: 애니메이션 50% 단축 또는 비활성
    - 한 화면 TODO 5개 제한 (인지부하)
    - 모바일 본문 최소 16px
    - 폼 label 명시적 연결, 에러는 aria-describedby
  </accessibility>
</aesthetic_guidelines>

<security_considerations>
  <input_validation>
    - CRITICAL: 모든 Server Action / Route Handler 입력은 Zod 스키마로 검증. 클라이언트 검증은 UX용일 뿐.
    - 텍스트 최대 길이: title 80자, description 1000자, retro 200자, override 사유 200자, note 5000자
    - 파일 업로드: PDF MIME 검증 + magic byte (`%PDF-`) + 최대 20MB + sha256 계산
    - 학교 유형/시도/학년/학기 등은 서버 측 enum 검증
    - 마크다운 입력: react-markdown + rehype-sanitize로 XSS 방지 (HTML 태그 strip)
  </input_validation>

  <pii_policy>
    - CRITICAL: 다음 항목은 PDF에서 추출/저장 자체를 하지 않는다 — 주민번호 앞6자리, 보호자 정보, 가족관계, 주소 상세, 학교폭력, 보건. mask.ts 단위 테스트로 회귀 방지.
    - 출결 사유 텍스트는 폐기, 카운트만 보관
    - 학교명 기본 마스킹, 사용자 명시 동의 시만 평문 저장
    - 로그/Sentry: PII 자동 스크럽 (이메일·이름·주소 패턴)
    - 분석(PostHog): 이메일/이름 미전송, distinct_id는 User.id의 sha256 해시
  </pii_policy>

  <encryption>
    - PDF: application-layer AES-256-GCM (PDF_ENCRYPTION_KEY 32바이트), IV/tag는 PdfFile에 저장
    - PostgreSQL: TLS in transit, at rest는 호스팅 제공자(Neon)
    - 비밀번호: bcrypt cost 12 (NextAuth 기본은 그 이상 권장)
    - 세션 JWT: HS256 + NEXTAUTH_SECRET, 토큰 만료 명시
    - 키 로테이션: PDF_ENCRYPTION_KEY는 분기별 로테이션 (구 키는 90일 보존하여 복호화)
  </encryption>

  <auth_security>
    - OTP: 6자리 숫자, 5분 만료, 1분 1회 발송, 1시간 5회 (이메일+IP 단위)
    - 로그인 시도 레이트 리밋: 1분 5회 / 1시간 20회 (이메일+IP)
    - 세션: HttpOnly + Secure + SameSite=Lax 쿠키
    - CSRF: NextAuth 내장 + Server Action 자체 csrf token (Next.js 14)
    - 비밀번호 변경/계정 삭제 시 OTP 재인증 필수
    - 14세 미만 보호자 동의 토큰: 7일 만료, 1회용 (consume 후 무효)
  </auth_security>

  <data_protection>
    - CRITICAL: 모든 ID는 UUID(v4), 추측 불가. URL에 noted ID 노출 시에도 권한 체크 필수.
    - Row-level access control: 모든 DB 쿼리에 ownerUserId/studentId 필터를 코드 레벨에서 강제 (Prisma extension 또는 service layer wrapper)
    - 학부모 데이터 접근: ParentChildLink.consentStatus=granted 검증 + 학생이 share 동의한 항목만
    - 공유 링크: 토큰 32자 url-safe random, 30일 만료, 학생이 즉시 취소 가능
    - 데이터 내보내기: 1시간 유효 signed URL, Sentry/로그에 URL 미기록
  </data_protection>

  <api_security>
    - 모든 Server Action / Route Handler는 인증 검증 (NextAuth `auth()`) + 권한 체크
    - PDF 업로드 엔드포인트: 인증 + Zod + 파일 크기/타입 + 본인 studentId 검증
    - cron 엔드포인트: X-Cron-Secret 헤더 검증 (timing-safe-equal)
    - Rate limit (Upstash): 일반 API 100/min, OTP 1/min, PDF upload 5/hour per user
    - CORS: NEXT_PUBLIC_APP_URL만 허용
  </api_security>

  <client_security>
    - CRITICAL: 클라이언트 코드/NEXT_PUBLIC_* env에 비밀 노출 금지 (Anthropic key, S3 secret 모두 server-only)
    - CSP 헤더 (Next.js middleware): default-src 'self'; img-src 'self' data:; script-src 'self' 'unsafe-inline' (Next.js 요건); connect-src 'self' https://*.posthog.com https://*.sentry.io; font-src 'self'
    - Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
    - X-Content-Type-Options: nosniff
    - X-Frame-Options: DENY (iframe 임베드 차단)
    - Referrer-Policy: strict-origin-when-cross-origin
    - Permissions-Policy: camera=(), microphone=(), geolocation=()
  </client_security>

  <audit_and_breach>
    - AuditLog: PDF 열람·삭제·내보내기·학부모 열람·트랙 변경·계정 삭제 모두 자동 기록
    - 사용자가 자신의 AuditLog 30건 조회 가능 (/settings/privacy)
    - 침해 통지 SOP: 의심 발견 시 24시간 내 내부 확정 → 72시간 내 사용자 공지 (이메일 + 인앱 배너)
    - 외부 보안 감사: 출시 전 1회 (취약점 진단), 이후 연 1회
  </audit_and_breach>
</security_considerations>

<advanced_functionality>
  <today_balance_rule>
    오늘의 5개는 카테고리 A,B,C,D,E 각 1개 우선 + 부족 시 priority desc 보충. 사용자가 직접 빼고 추가 가능.
  </today_balance_rule>
  <auto_snooze>
    학생의 currentTerm과 시기 부적합 TODO 자동 snoozed 처리 (예: 고1 5월에 고3 정시원서 TODO).
  </auto_snooze>
  <decision_point_engine>
    학사일정 기준 임박 결정 (선택과목·트랙·원서) 감지 시 대시보드 상단 노란 배너 표시. D-30/D-7/D-1에 색 강조.
  </decision_point_engine>
  <track_simulation_lite>
    트랙 수동 변경 시 영향받는 TODO 수를 미리보기 ("학종으로 바꾸면 +12 TODO, -3 TODO")
  </track_simulation_lite>
  <weekly_retro_to_priority>
    retroText에서 부정 키워드("힘들었다","너무 많다") 감지 시 다음 주 추천 수 5→3으로 자동 축소 + 사용자에게 안내.
  </weekly_retro_to_priority>
  <empty_signal_mode>
    중학생 / 신호 부족 시 "탐색 모드" — 트랙 추천 대신 진로 검사·동아리 가입 등 탐색 TODO 위주 생성.
  </empty_signal_mode>
  <bulk_operations>
    로드맵 학기 상세에서 다중 선택 → [모두 스누즈 7일][모두 다음 주로][카테고리 일괄 스킵]
  </bulk_operations>
  <keyboard_shortcuts>
    pages_and_interfaces &lt; keyboard_shortcuts_reference &gt; 절 참조.
  </keyboard_shortcuts>
  <export>
    데이터 내보내기 JSON: User + Student + ParsedReportCard 마스킹본 + StudentProfile 이력 + Todo 전체 + WeeklyCheckIn + AuditLog 30건.
  </export>
  <darkmode>
    시스템 prefers-color-scheme 자동 + 사용자 토글 (/settings/account). 야간 학습용.
  </darkmode>
</advanced_functionality>

<final_integration_test>

  <test_scenario_1>
    <description>예비 고1 (만 13세 5개월) 학생이 보호자 동의 흐름을 거쳐 가입하고 첫 로드맵을 받는다</description>
    <steps>
      1. 학생이 /auth/signup에서 이메일 + 출생연도(2012년) + 역할(학생) 입력
      2. 시스템이 만 14세 미만 감지 → "보호자 이메일" 입력 화면 표시
      3. 학생이 보호자 이메일 입력 → ParentChildLink (consentStatus=pending, token 7일 유효) 생성, Resend로 보호자 이메일 발송
      4. 학생 화면에는 "보호자 동의를 기다리고 있어요" 상태
      5. 보호자가 이메일 링크 클릭 → /parent-consent/[token] → 4개 항목 체크 + OTP 본인 인증 → 동의 완료
      6. 학생 이메일로 가입 완료 알림 수신, 학생이 다시 로그인
      7. /onboarding/role에서 학교 유형(일반계), 시도(서울), 학년(중3) 선택 → examCycle=2028 자동 분기 배지 표시
      8. /onboarding/consent에서 필수 1 + 선택 2 분리 동의
      9. /onboarding/upload에서 NEIS PDF 업로드 (16MB, 텍스트형)
      10. 진행 카드 5단계 표시, P95 30초 이내 완료
      11. /onboarding/correct로 이동, 신뢰도 ≥80 항목은 접힘, &lt;80은 자동 펼침. 사용자가 1개 항목 수정.
      12. /onboarding/track에서 "탐색 모드" 카드 표시 (중학생이라 신호 부족)
      13. /dashboard 진입, 격자 9개 학기 셀 + 결정 포인트 배너 + 오늘의 TODO 5개 표시
      14. 검증: ParsedReportCard.sections에 주민번호·학교폭력·보건 키 부재. 학교명은 마스킹.
      15. 검증: Todo 30개 이상 생성, 각 explanation 1~2문장 존재
    </steps>
  </test_scenario_1>

  <test_scenario_2>
    <description>고2 학생이 스캔본 PDF를 업로드 → OCR 폴백 + 수동 보정 → 트랙 듀얼 추천 확정</description>
    <steps>
      1. 만 17세 학생 가입, 출생연도 2009 → exam_cycle=2027
      2. /onboarding/upload에서 스캔본 PDF 업로드 (텍스트 임베디드 X)
      3. pdf-parse 텍스트 추출 → 한글 비율 5%로 실패 판정 → OCR 폴백 자동 시작
      4. 진행 카드에 노란 강조 "스캔본 감지, OCR 처리 중..." 표시
      5. Claude Vision OCR 페이지별 호출, P95 25초 이내 완료
      6. /onboarding/correct: 신뢰도 평균 60 → 모든 항목 자동 펼침. 사용자가 세특 텍스트 일부 수정
      7. extractionConfidence 항목별 재계산, manualCorrected=true
      8. /onboarding/track: 듀얼 추천 카드 2개 (예: 주력 학종, 보조 교과, 신뢰도 중간)
      9. 사용자가 "수동 변경" 토글 → 정시 우선으로 override + 사유 입력
      10. TrackRecommendation.userOverrideTrack 저장, TODO 재생성 큐 enqueue
      11. /dashboard에 진입, 카테고리 B(수능) 우선순위 상승 확인
      12. 검증: AuditLog에 track_override 기록
    </steps>
  </test_scenario_2>

  <test_scenario_3>
    <description>학생이 PDF를 즉시 삭제하면 ParsedReportCard는 유지되고 PdfFile만 hard delete</description>
    <steps>
      1. 학생 로그인 + /settings/privacy 진입
      2. PDF 관리 섹션에서 가장 최근 업로드 항목의 [지금 삭제] 클릭
      3. 확인 모달 ("정말 삭제할까요? 분석 결과는 유지됩니다") → 확인
      4. Server Action deletePdfFile 호출
      5. S3 객체 삭제 (DeleteObject) + DB PdfFile 레코드 hard delete
      6. ParsedReportCard.sourcePdfId가 NULL로 SET NULL
      7. AuditLog action=delete_pdf 기록 (actorUserId=현재 사용자, ip, ua hash)
      8. 화면에 토스트 "삭제 완료" + 목록에서 항목 제거
      9. /profile에서 흥미 가설/트랙은 그대로 유지 확인
      10. 검증: S3 GetObject 시 NoSuchKey
      11. 검증: 30일 후에도 DB 레코드 없음 (이미 삭제)
    </steps>
  </test_scenario_3>

  <test_scenario_4>
    <description>30일 자동 삭제 cron이 만료된 PDF 일괄 정리</description>
    <steps>
      1. 학생이 retentionConsent=true로 PDF 업로드 (uploadedAt=T0, expiresAt=T0+30d)
      2. 시스템 시간 T0+30d로 진행 (테스트 환경)
      3. POST /api/jobs/cleanup-pdfs (X-Cron-Secret 헤더)
      4. 워커가 expiresAt &lt; now() PdfFile 조회 → S3 객체 삭제 → DB hard delete
      5. ParsedReportCard.sourcePdfId가 NULL로 SET
      6. 작업 로그에 처리 건수 기록
      7. 검증: 다음 날 동일 cron 실행 시 처리 건수 0
      8. 검증: 잘못된 X-Cron-Secret으로 호출 시 401
      9. 학생 /settings/privacy의 PDF 목록이 비어있음
      10. 검증: AuditLog에 system 액션으로 delete_pdf 기록
    </steps>
  </test_scenario_4>

  <test_scenario_5>
    <description>학부모가 공유 링크로 자녀 요약 리포트를 열람, 학생이 링크를 취소</description>
    <steps>
      1. 학생이 /settings/privacy에서 [학부모 공유 링크 만들기] (audience=parent_summary) 클릭
      2. ShareLink 생성 (token, expiresAt=now+30d), 공유 가능 URL 표시 + 복사 버튼
      3. 학부모가 비로그인 상태로 /share/[token] 접속
      4. 시스템이 토큰 검증 + viewCount++ + AuditLog action=share_link_view 기록 (actorUserId=null)
      5. 학부모 리포트 1장 표시 (트랙·흥미·강점·다음단계·이번달 TODO 5개 제목만)
      6. 학부모가 [PDF 다운로드] 클릭 → jspdf로 클라이언트 생성 + 다운로드
      7. 학생이 /settings/privacy에서 해당 링크 [취소] → revokedAt 설정
      8. 학부모가 다시 /share/[token] 접속 → "이 링크는 더 이상 유효하지 않습니다" 빈 상태
      9. 검증: viewCount 1, lastViewedAt 갱신, revokedAt 존재
      10. 검증: AuditLog에 share_link_view 1건 + share_revoke 1건
    </steps>
  </test_scenario_5>

  <test_scenario_6>
    <description>주간 체크인 → 다음 주 추천 보정 + 부정 키워드 감지 시 추천 수 축소</description>
    <steps>
      1. 일요일 18:00 cron이 활성 학생에게 이메일 발송
      2. 학생이 /checkin 진입, 지난 주 TODO 12개 중 4개 done, 5개 missed, 3개 snoozed
      3. 학생이 미완료 5개를 빠른 액션으로 [다음 주로]/[스누즈]/[스킵] 처리
      4. retroText에 "이번 주 너무 힘들었어요. 시험 기간이라 따라가기 어려웠어요" 입력
      5. 만족도 2점 선택, [확정] 클릭
      6. WeeklyCheckIn 레코드 저장
      7. 시스템이 retroText 부정 키워드 ("힘들","어려") 감지 → 다음 주 추천 5개 → 3개 축소
      8. 학생에게 안내 토스트 "이번 주는 3개로 줄였어요. 페이스를 유지해보세요"
      9. /today에서 다음 주 시작 시 3개 카드만 표시
      10. 검증: AuditLog action=todo_skip 기록, WeeklyCheckIn.satisfactionScore=2
    </steps>
  </test_scenario_6>

  <test_scenario_7>
    <description>PII 마스킹 회귀 테스트: 학교폭력·보건 항목이 포함된 PDF는 해당 항목이 sections에 미포함</description>
    <steps>
      1. 테스트용 PDF (학교폭력 조치사항 텍스트, 보건 항목, 주민번호 앞6자리 포함) 업로드
      2. parsePdf 워커 실행
      3. anchors.ts가 "학교폭력", "보건" 키워드 감지 → 해당 섹션 스킵
      4. mask.ts가 주민번호 정규식 매칭 → 폐기
      5. ParsedReportCard.sections 저장 후 검증:
         - sections에 학교폭력 키 없음
         - sections에 보건 키 없음
         - sections.attendance에 사유 텍스트 없음 (카운트만)
         - sections.meta.schoolNameMasked=true
      6. 사용자에게 알림 인포 "학교폭력·보건 항목은 시스템 정책에 따라 분석에 사용되지 않습니다"
      7. 단위 테스트 corpus 10종에 대해 0건 누출 확인
      8. 로그·Sentry에도 원문 미기록 (beforeSend 스크럽)
    </steps>
  </test_scenario_7>

  <test_scenario_8>
    <description>학생이 14세 도달 후 보호자 동의 갱신 흐름</description>
    <steps>
      1. 만 13세였던 학생이 14세 생일 도달 (cron 또는 로그인 시 체크)
      2. 시스템이 학생 다음 로그인 시 "보호자 동의 갱신 안내" 모달 표시
      3. 학생이 [본인 동의로 전환] 선택 → 학생 본인 동의 흐름 (필수+선택 분리)
      4. ParentChildLink는 유지하되 consentStatus=granted_by_self
      5. 학부모 권한은 학생이 별도 동의 시에만 유지
      6. AuditLog action=consent_grant (actor=student) 기록
      7. 검증: 이후 학부모가 /share 접근 시 학생 동의 항목만 표시
    </steps>
  </test_scenario_8>

  <test_scenario_9>
    <description>LLM 호출 실패 시 시드 원문 fallback으로 사용자 영향 없이 로드맵 생성</description>
    <steps>
      1. Anthropic API key 무효화 (테스트 환경)
      2. 학생이 PDF 업로드 → 파싱 성공 → generateRoadmap 워커 시작
      3. personalize.ts 첫 호출 시 401 받음 → 1회 재시도 후 fallback
      4. 시드 원문 title/description/explanation 그대로 사용하여 Todo 생성
      5. /dashboard 진입 → TODO 30개 정상 표시 (개인화는 빠짐)
      6. 사용자에게 노출되는 에러 없음 (UX 보존)
      7. Sentry에 category=llm severity=error 캡처
      8. 검증: Todo.source = 'rule_seed' (rule_plus_llm 아님)
    </steps>
  </test_scenario_9>

  <test_scenario_10>
    <description>계정 삭제 30일 유예 + hard delete cron</description>
    <steps>
      1. 학생이 /settings/privacy에서 [계정 삭제 요청] 클릭
      2. 모달에서 30일 유예 안내 + OTP 재인증 입력
      3. User.deletedAt = now() 설정, 모든 세션 jti blacklist
      4. 즉시 로그아웃 + 안내 이메일 발송 ("30일 내 복구 가능")
      5. 학생이 5일 후 다시 로그인 시도 → "계정이 삭제 예정입니다. [복구]" 옵션
      6. 복구 시 deletedAt=null 처리
      7. 30일 후 cron이 hard delete: User + Student + 종속 데이터 모두 삭제
      8. AuditLog는 actorUserId=null로 익명화하여 1년까지 보존
      9. 검증: 동일 이메일로 재가입 가능
      10. 검증: S3에 해당 학생의 모든 객체 삭제됨
    </steps>
  </test_scenario_10>

  <test_scenario_11>
    <description>접근성: 키보드만으로 회원가입 → PDF 업로드 → TODO 완료 처리 가능</description>
    <steps>
      1. Tab 키로 /auth/signup 진입, 모든 필드 포커스 가능
      2. Enter로 OTP 발송, OTP 입력 박스 자동 포커스 + 좌→우 자동 이동
      3. Tab으로 동의 체크박스 이동, Space로 체크
      4. PDF 업로드는 [파일 선택] 버튼 Enter → 파일 다이얼로그
      5. 진행 화면에서 aria-live로 단계 안내가 스크린리더에 전달
      6. /dashboard에서 `g t` 단축키로 /today 이동
      7. 첫 TODO 카드 Tab → [완료] 버튼 Enter → 모달 진입
      8. 만족도 라디오 ←/→ 이동, 회고 텍스트 입력, Enter 확정
      9. 토스트 "완료 처리됨" aria-live 안내
      10. 검증: 모든 인터랙티브 요소 focus ring 가시
      11. 검증: 모든 아이콘 only 버튼 aria-label 존재
    </steps>
  </test_scenario_11>

  <test_scenario_12>
    <description>다크모드 토글 + prefers-reduced-motion 준수</description>
    <steps>
      1. 시스템 다크모드 활성 → 첫 진입 시 자동 다크 테마 적용
      2. /settings/account에서 [Light]로 명시적 변경 → 라이트 적용 + localStorage 저장
      3. prefers-reduced-motion: reduce 활성화 (OS 설정)
      4. 페이지 전환·카드 stagger 애니메이션이 50% 단축 또는 비활성
      5. 토스트 enter는 fade만 사용 (slide 비활성)
      6. 검증: CSS @media (prefers-reduced-motion: reduce) 규칙 적용 확인
    </steps>
  </test_scenario_12>

</final_integration_test>

<success_criteria>
  <functionality>
    - 17개 라우트 모두 인증 가드 정확히 동작 (비인증 접근 시 /auth/login?next=)
    - 만 14세 미만 보호자 동의 흐름 100% 강제 (미동의 가입 차단)
    - PDF 업로드부터 /dashboard 도달까지 P95 ≤ 30초, P50 ≤ 18초
    - PDF 파싱 성공률 (수동 보정 없이) ≥ 70% (NEIS 표준 PDF 100건 표본)
    - 30일 자동 삭제 cron 성공률 100% (잡 실패 시 알림)
    - PII 추출 누출 0건 (학교폭력·보건·주민번호 회귀 테스트)
    - TODO 추천 30~80개 생성, 100% explanation 포함
    - 트랙 듀얼 추천 100% confidenceLabel + 신호 근거 포함
    - 학부모 공유 링크 30일 만료 + 학생 즉시 취소 동작
    - 데이터 내보내기 JSON 1시간 signed URL 정확
    - 계정 삭제 30일 유예 + hard delete cron 정확
  </functionality>
  <user_experience>
    - 모바일 4G에서 첫 페인트 P95 ≤ 2.5초
    - WCAG 2.1 AA 자동 검사 (axe-core) 0 issue
    - 키보드만으로 모든 핵심 흐름 완수 가능
    - prefers-reduced-motion 준수
    - 토스트 max 3 stack, aria-live 동작
    - 빈 상태 일러스트 + 메시지 + CTA 모든 화면 정의
    - 면책 바 모든 인증 화면 하단 노출
    - 다크모드 토글 동작 + 시스템 prefers 자동
    - 모바일 하단 탭 56px + safe-area 적용
  </user_experience>
  <technical_quality>
    - TypeScript strict + noUncheckedIndexedAccess 적용, any 사용 0건
    - Zod 스키마로 모든 Server Action / Route Handler 입력 검증
    - Prisma 스키마 마이그레이션 의존성 명확 (rollback 가능)
    - 모든 cron 엔드포인트 X-Cron-Secret 검증
    - 모든 ID UUIDv4
    - 단위 테스트: PII 마스킹, examCycle 계산, priorityScore, balance 룰 100% 커버
    - 통합 테스트: 회원가입 + PDF 업로드 + 로드맵 생성 1회 시나리오
    - E2E 테스트: 시나리오 1, 2, 3, 5 자동화 (Playwright)
    - LLM 호출 실패 시 시드 fallback 동작
    - Sentry 에러 캡처 + PII 스크럽 동작
    - PostHog 이벤트 distinct_id 해시 동작
  </technical_quality>
  <visual_design>
    - 모든 색상 hex 토큰화 (Tailwind config)
    - 모든 dimension 8pt grid
    - 카테고리 5색 + 아이콘 + 라벨 동시 사용 (색만으로 식별 금지)
    - 폰트 Pretendard self-host, font-display: swap
    - lucide-react 단일 아이콘 라이브러리
    - radius/shadow/border 토큰 일관 적용
  </visual_design>
  <build>
    - pnpm build 성공, .next 산출
    - Vercel 배포 (preview + production) 자동
    - Lighthouse 점수 Performance ≥ 80, Accessibility ≥ 95, Best Practices ≥ 95, SEO ≥ 90 (모바일)
    - 환경 변수 누락 시 build/start 실패 (env.ts zod 검증)
    - 모든 OAuth 콜백 URL 등록 가이드 README 명시
  </build>
  <kpi_targets>
    - PDF 업로드 → 로드맵 도달률: M3 ≥ 60%, M6 ≥ 80%
    - D7 리텐션: M3 ≥ 25%, M6 ≥ 40%
    - 주간 체크인 완료율: M3 ≥ 30%, M6 ≥ 50%
    - TODO 수용률: M3 ≥ 40%, M6 ≥ 60%
    - 학부모 유료 전환율: M3 ≥ 5%, M6 ≥ 10%
    - PDF 파싱 성공률 (수동 보정 X): ≥ 70%
    - 학생당 LLM 비용 월: ≤ ₩250 ($0.18) — claude-haiku-4-5(personalize) + claude-sonnet-4-7(signal) 기준, prompt caching 활성화 시 산식($0.167≈₩230) 반영. 청사진 §11.4 A5 가정(₩200)은 ₩250로 spec 단계 정정.
  </kpi_targets>
  <milestones>
    M1 (월 1): 인프라 + 인증(NextAuth, OTP, 14세 미만 보호자 동의) + 분리 동의 + DB 스키마 마이그레이션 + env 검증
    M2 (월 2): PDF 업로드 + 암호화 + 텍스트 파싱 + PII 마스킹 + 수동 보정 화면 + 회귀 테스트 corpus
    M3 (월 3): OCR 폴백 + 신호 추출(룰) + StudentProfile + 트랙 듀얼 추천 + 룰 시드 TODO 생성
    M4 (월 4): LLM 개인화(Claude) + 가드레일 + 대시보드 격자 + /today + /roadmap/* + /todo/* UI
    M5 (월 5): 주간 체크인 + 학부모 공유 링크 + 부모 리포트 + 개인정보 설정 + cron(자동삭제·체크인·롤오버)
    M6 (월 6): AuditLog 통합 + 다크모드 + 접근성 audit + Sentry/PostHog + E2E 자동화 + 외부 보안 감사 + 출시
  </milestones>
</success_criteria>

<build_output>
  <build_command>pnpm build</build_command>
  <output_directory>.next/</output_directory>
  <node_version>20.x LTS</node_version>
  <package_manager>pnpm v9.x (lockfile 커밋 필수)</package_manager>
  <deployment>
    - 프론트/API: Vercel (preview = PR per branch, production = main)
    - DB: Neon Serverless Postgres (또는 Supabase) — main branch + preview branch
    - Redis: Upstash (단일 region, ap-northeast-2 우선)
    - 객체 스토리지: Cloudflare R2 또는 AWS S3 (ap-northeast-2)
    - Cron: Vercel Cron (vercel.json)
    - Worker (BullMQ): Vercel Serverless로 시작, 처리량 증가 시 Fly.io 별도 배포
    - 환경 변수: Vercel Project Settings에 모두 등록 (server-only는 NEXT_PUBLIC_ 미접두)
  </deployment>
  <vercel_cron_config>
    vercel.json:
      - /api/jobs/cleanup-pdfs : "0 0 * * *" (00:00 KST)
      - /api/jobs/cleanup-audit-logs : "30 0 * * *"
      - /api/jobs/cleanup-shares : "0 1 * * *"
      - /api/jobs/account-hard-delete : "0 2 * * *"
      - /api/jobs/today-balance : "0 4 * * *"
      - /api/jobs/weekly-checkin-email : "0 9 * * 0" (일요일 18:00 KST = UTC 09:00)
      - /api/jobs/grade-rollover : "0 0 1 * *" (매월 1일)
  </vercel_cron_config>
</build_output>

<key_implementation_notes>

  <critical_paths>
    1. PII 마스킹 (mask.ts) — 회귀 테스트로 누출 0건 보장. 새 PDF 양식 추가 시 corpus 업데이트.
    2. PDF 30일 자동 삭제 cron — 실패 시 데이터 보존 위반 → 알림 + 재시도 정책 명확화.
    3. examCycle 자동 분기 (examCycle.ts) — birthYear → entranceYear → cycle 매핑이 모든 룰셋의 진입점.
    4. 룰셋 외부 JSON (rules/*.json) — 코드 배포 없이 갱신 가능, 스키마 검증 (Zod) 필수.
    5. LLM 가드레일 (guardrails.ts) — 사교육·합격보장·약점표현 정규식 패턴, 위반 시 시드 fallback.
    6. 14세 미만 보호자 동의 흐름 — token 만료, OTP 본인 인증, 7일 후 학생 계정 삭제.
  </critical_paths>

  <recommended_implementation_order>
    1. 인프라 셋업: pnpm + Next.js 14 + TypeScript strict + Tailwind + shadcn/ui 초기 구성, env.ts zod 검증
    2. Prisma 스키마 (10개 엔티티) + 첫 마이그레이션 + DB 시드 스크립트
    3. NextAuth v5 + 이메일 OTP + 카카오/구글 OAuth + 14세 미만 분기 + 보호자 동의 페이지
    4. 분리 동의 페이지 + 동의 미체크 시 가드
    5. examCycle.ts + academicCalendar.ts 유틸 + 단위 테스트
    6. 룰셋 외부 JSON 4종 작성 (seed_todos.{2025,2026,2027,2028}.json, track_matrix.json, career_hypothesis.json) + Zod 스키마
    7. PDF 업로드 (Server Action + S3 암호화) + sha256 중복 검증
    8. parsePdf 워커: pdf-parse → anchors.ts → mask.ts → ParsedReportCard 저장
    9. PII 마스킹 회귀 테스트 corpus 작성 + CI 통합
    10. OCR 폴백 (Claude Vision) + extractionConfidence 계산
    11. 수동 보정 페이지 (좌우 분할 + pdfjs 미리보기 + 인라인 편집)
    12. 신호 추출 services/signal/* + StudentProfile 생성
    13. 트랙 매트릭스 (services/signal/tracks.ts) + 듀얼 추천 페이지
    14. seedLoader + priority.ts + balance.ts + Todo 생성 (룰만으로 먼저)
    15. LLM 개인화 (personalize.ts) + guardrails.ts + 배치 호출 + fallback
    16. 대시보드 격자 + 결정 포인트 배너 + 오늘 TODO 패널
    17. /today + /roadmap/[grade]/[term] + /todo/[id] + 상태 전이
    18. 주간 체크인 (/checkin) + retroText 보정 로직
    19. 학부모 리포트 + ShareLink + /share/[token] 공개 페이지
    20. 개인정보 설정 (즉시 PDF 삭제, 데이터 내보내기, 계정 삭제 30일 유예, AuditLog 표시)
    21. cron 라우트 7종 + Vercel Cron 등록 + X-Cron-Secret 검증
    22. AuditLog 자동 기록 (모든 PII·민감 액션)
    23. 토스트/에러 페이지/Form validation/Error Boundary
    24. 다크모드 + 접근성 audit (axe-core CI) + prefers-reduced-motion
    25. Sentry + PostHog 통합 + PII 스크럽
    26. E2E 테스트 (Playwright) + 시나리오 1·2·3·5 자동화
    27. 면책 바 + 콘텐츠 정책 텍스트 (rules/disclaimer.ko.json)
    28. 배포 (Vercel preview + production) + DNS + OAuth 콜백 URL 등록
  </recommended_implementation_order>

  <database_schema_excerpt>
    Prisma 핵심 enum (CRITICAL: 본문 엔티티 정의와 1:1 동기화 필수):
      enum UserRole { student parent system }
      enum ActorType { user system anonymous }
      enum SchoolType { general specialized_self_directed vocational other }
      enum SchoolLevel { elementary middle high }
      enum ExamCycle { CYCLE_2025 CYCLE_2026 CYCLE_2027 CYCLE_2028 }
      enum Track { jeongsi hakjong gyogwa nonsul silgi undecided }
      enum TodoCategory { A1 A2 A3 A4 A5 B1 B2 B3 B4 B5 B6 B7 C1 C2 C3 C4 C5 C6 C7 D1 D2 D3 D4 D5 D6 D7 E1 E2 E3 E4 E5 }
      enum TodoState { backlog today in_progress done snoozed skipped }
      enum TermCode { TERM_1H1 TERM_1HSUMMER TERM_1H2 TERM_1HWINTER TERM_2H1 TERM_2HSUMMER TERM_2H2 TERM_2HWINTER TERM_3H1 TERM_3HSUMMER TERM_3H2 }
      enum ConsentStatus { pending granted granted_by_self revoked }
      enum ParseStatus { queued parsing_text parsing_ocr parsed failed manual_correction manual_input }
      enum AuditAction {
        view_pdf delete_pdf upload_pdf
        view_profile export_data
        parent_view_summary
        share_link_create share_link_view share_link_revoke
        consent_grant consent_revoke consent_renewal_required
        track_override todo_skip
        account_delete_requested account_delete_completed
        schooltype_waitlist_register
      }
      enum WaitlistSource { schooltype_block_modal external_inquiry }
      enum SkipReason { already_done not_interested wrong_timing other }
      enum ShareAudience { parent_summary student_profile_readonly }
      enum InterestCategory { ai_sw humanities_social natural_science arts_athletics education health_medical business undefined }
    핵심 모델 인덱스:
      @@index([studentId, state, priorityScore(sort: Desc)])  // Todo 조회
      @@index([studentId, grade, term])
      @@index([expiresAt])  // PdfFile cleanup
      @@unique([studentId, schoolLevel, grade, term])  // ParsedReportCard
      @@unique([studentId, weekStart])  // WeeklyCheckIn
      @@unique([token])  // ShareLink
      @@unique([email])  // WaitlistEntry
      @@index([targetStudentId, occurredAt(sort: Desc)])  // AuditLog
  </database_schema_excerpt>

  <performance_considerations>
    - LLM 호출은 prompt caching 활용 (system prompt + 시드 템플릿 5분 TTL)
    - LLM 배치: 시드 5개씩 묶어 호출 (총 호출 수 80→16으로 축소)
    - TanStack Query staleTime: 대시보드 격자 30초, 오늘 TODO 60초
    - 학년×학기 격자 데이터는 학생당 1회 prefetch + 학기 상세는 lazy load
    - 이미지: Pretendard 변수 폰트 self-host + woff2 + font-display: swap
    - PDF 미리보기: 첫 페이지만 즉시 렌더, 나머지는 인터섹션 옵저버
    - DB 쿼리: priority desc 정렬은 @@index 활용, raw SQL 회피
    - Edge runtime은 인증 미들웨어만, 그 외 Node runtime
  </performance_considerations>

  <testing_strategy>
    - 단위 테스트 (Vitest): mask.ts (PII 회귀 corpus 10종), examCycle.ts, priority.ts, balance.ts, anchors.ts
    - 통합 테스트: Server Actions (회원가입, 동의, 업로드, 트랙 변경, TODO 상태)
    - E2E 테스트 (Playwright): 시나리오 1, 2, 3, 5, 11 자동화
    - LLM 모킹: msw로 Anthropic API 모킹, 결정적 출력
    - 성능 테스트: k6 — 동시 100명 PDF 업로드 시 P95 측정
    - 보안 테스트: Zod schema fuzzing, OWASP ZAP 자동 스캔, 출시 전 외부 감사 1회
    - 접근성 테스트: axe-core CI + Lighthouse 모바일 ≥ 95
    - 비율: 단위 60% / 통합 25% / E2E 15%
  </testing_strategy>

  <external_resource_links>
    rules/external_resources.json — 카테고리·시드 ID별로 자동 첨부할 공식 자원 링크 매핑.
    예시:
      ```
      {
        "D1": [
          { "label": "대학어디가 (한국대학교육협의회)", "url": "https://www.adiga.kr" },
          { "label": "EBSi 입시정보", "url": "https://www.ebsi.co.kr" }
        ],
        "D2": [
          { "label": "한국대학교육협의회 모집요강", "url": "https://www.kcue.or.kr" }
        ],
        "B1": [{ "label": "EBS 국어", "url": "https://www.ebs.co.kr" }],
        "B2": [{ "label": "평가원 기출", "url": "https://www.suneung.re.kr" }],
        "C3": [{ "label": "워크넷 진로검사", "url": "https://www.work.go.kr" }, { "label": "커리어넷", "url": "https://www.career.go.kr" }]
      }
      ```
    개인화 단계에서 LLM이 description 끝에 자동으로 1~2개 링크 첨부. 매칭 사교육·학원 링크는 절대 추가 안 함 (가드레일).
    매칭 지원 안내: "본 서비스는 외부 활동 매칭을 지원하지 않습니다. 가족·학교에 도움을 요청해주세요." 문구는 진로 인터뷰·외부 캠프 시드 description 끝에 LLM이 자동 첨부.
  </external_resource_links>

  <tone_content_policy>
    rules/tone_guard.json + LLM 가드레일 (guardrails.ts):
      금칙어 (시드 fallback 트리거): "학원", "과외", "사교육", "합격을 보장", "확실히 합격", "반드시 ~ 해야"
      자동 치환: "부족" → "다음 단계로", "못 한다" → "더 키울 수 있는 부분", "약점" → "성장 영역"
      면책 자동 첨부: 모든 explanation 끝에 (rules/disclaimer.ko.json의 micro disclaimer "참고용 안내입니다")
      카테고리별 톤 가이드:
        - A 내신: 구체적 행동 지시 ("3단원 정리표 작성")
        - B 수능: 도구·자료 명시 (EBS, 평가원 기출)
        - C 세특: 산출물 형태 명시 (발표/보고서/실험)
        - D 입시: 결정 도우미 어조 ("비교해보고 정해도 늦지 않아요")
        - E 생활: 코칭 어조 ("이번 주만 11시 취침 도전?")
  </tone_content_policy>

  <rules_json_schema>
    rules/seed_todos.{cycle}.json 구조:
      [{
        "id": "string",                     // unique seed id
        "category": "A1|...|E5",
        "grade": 1|2|3,
        "term": "1H1|...|3H2",
        "track": "jeongsi|hakjong|gyogwa|nonsul|silgi|null",
        "title_template": "string",
        "description_template": "string",
        "explanation_template": "string",
        "mandatory": true|false,
        "due_offset_days": 0|7|30|...|null,  // 학기 시작일로부터
        "priority_base": 0~100,
        "signal_modifiers": [
          {"if_signal": "freeSemesterStrength", "op": ">=", "value": 70, "delta": +15}
        ]
      }, ...]
    rules/track_matrix.json:
      [{ "if": {"gradeAvgMain":"&lt;=2.5","senthukDensity":"high"}, "primary":"hakjong","secondary":"gyogwa","confidence":"high" }, ...]
  </rules_json_schema>

  <open_questions>
    OPEN_QUESTION_1: NEIS 공식 PDF 양식이 학교마다 변형이 어느 정도 있는지 — 70% 파싱 성공률 달성을 위한 corpus 크기 결정 필요. (제안: MVP 출시 후 실패 케이스 수집하여 anchors 보강)
    OPEN_QUESTION_2: 학부모 유료 전환 가격(₩9,900/월) 검증 필요. 현재 무료 단일 플랜으로 출시 + 6개월 후 도입 가설 (결제 라이브러리는 v2에서 toss/portone 검토).
    OPEN_QUESTION_3: 2028 대입 개편 세부(내신 5등급제 절대평가 비율 등)가 미확정 — rules/seed_todos.2028.json은 발표 시점에 즉시 갱신 가능하도록 외부화 유지.
    OPEN_QUESTION_4: 모집요강 데이터 소스 — MVP는 사용자 입력 + 캐시. 공식 API/제휴는 Phase 2에서 검토.
    OPEN_QUESTION_5: BullMQ 워커 배포 토폴로지 — Vercel Serverless로 시작 가능하나, OCR 폴백 시 30초 초과 가능성 → Fly.io 별도 worker 프로세스 도입 시점 결정 필요(체감 P95 25초 초과 시).
    OPEN_QUESTION_6: 학생 본인이 소유한 Student와 학부모가 만든 Student의 ownerUserId 구분 정책 — 자녀가 직접 가입 후 학부모가 합치는 시나리오 → ParentChildLink로 통합. 학부모가 먼저 만든 후 자녀 합류 시 소유권 이전 흐름 정의 필요.
    OPEN_QUESTION_7: OCR 제공자 선택(claude_vision vs google_vision) — 한국어 정확도 비교 후 결정. 환경변수 OCR_PROVIDER로 전환 가능하게 설계.
    OPEN_QUESTION_8: 14세 도달 갱신 로직(test_scenario_8) — 매월 cron으로 일괄 체크 vs 로그인 시점 lazy 체크. MVP는 로그인 시점 권장(인프라 단순).
    OPEN_QUESTION_9: 톤 가드 자동 치환 vs LLM 재요청 — "부족"이 의도된 표현인 경우 false positive 가능. MVP는 단순 치환, 사용자 피드백 수집 후 정책 조정.
    OPEN_QUESTION_10: 검증된 도메인 자문 채널 — 입시 정책 변동 모니터링 책임자 지정 필요 (본 spec은 도구만 제공).
  </open_questions>

</key_implementation_notes>

</project_specification>
