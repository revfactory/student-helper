# 프로덕트 청사진 — student-helper

> 본 문서는 `_workspace/01_domain_insights.md`(이하 "도메인 인사이트")를 토대로 학생 도우미 서비스의 프로덕트 결정 사항을 정리한 청사진이다. spec-writer가 곧바로 XML 빌드 스펙(`STUDENT_HELPER_SPEC.md`)으로 변환할 수 있도록 모든 핵심 결정에 `[DECISION]` 표식을 단다.
>
> 작성 기준일: **2026-05-02**
> 적용 입시 사이클: **2025학년도(현 고3) ~ 2028학년도(현 중3)**
> 타깃 디바이스: **웹 (반응형, 모바일 우선)** `[DECISION]`
> 권장 기술 스택: **Next.js 14+ App Router + TypeScript + Tailwind CSS + PostgreSQL + Prisma** `[DECISION]`
> 출시 가설: **6개월 MVP** `[DECISION]`

---

## 0. 한 줄 요약 (TL;DR)

**"초·중학교 생활기록부 PDF 한 장으로, 고1~고3 3년치 진학 로드맵과 이번 주 할 일까지 자동 생성해주는 학생 진학 코치"**

- 입력: 학생/학부모가 업로드한 NEIS 생활기록부 PDF + 학년/입시 사이클 정보
- 처리: 항목별 신호 추출 → 흥미·강약점·트랙 가설 → 룰+LLM 하이브리드 TODO 로드맵
- 출력: 학년 × 학기 × 카테고리(내신/수능/세특/입시/생활) × 트랙(정시/학종/교과/논술/실기) 격자 위에 정렬된 TODO 카드 + 주간 체크인 + 결정 도우미

---

## 1. 페르소나

### 1.1 Primary 페르소나 — 김지민 (예비 고1, 중3 겨울방학) `[DECISION: 1차 타깃]`

| 항목 | 내용 |
|------|------|
| 나이 / 학년 | 만 15세, 중3 → 예비 고1 (2026년 3월 입학 예정) |
| 적용 입시 사이클 | **2028학년도 대입 개편 적용 대상** (내신 5등급제, 수능 융합형) |
| 거주 / 학교 유형 | 수도권 일반계 고교 진학 예정 |
| 디지털 친숙도 | 매우 높음 (스마트폰 네이티브, 노트북 보유, 노션·구글 클래스룸 익숙) |
| 핵심 통증 (Pain) | 1) 고교 입학 후 무엇부터 해야 할지 막막함<br>2) 진로가 흐릿함 — "AI에 관심 있는 것 같은데..." 수준<br>3) 친구·인터넷 정보가 단편적·과장됨, 어디부터 믿어야 할지 모름<br>4) 부모님이 학원 컨설팅을 권하는데 비용·압박 부담 |
| 원하는 성과 (Gain) | 1) "올해 안에 끝내야 할 것" 리스트가 한눈에 보임<br>2) 자유학기·중학교 활동에서 무엇이 의미 있었는지 객관적 피드백<br>3) 친구들에게 "이거 한번 해봐" 추천할 수 있을 정도로 신뢰됨 |
| 사용 빈도 | 주 1~2회 (주간 체크인) + 학기 시작/방학 시점 집중 사용 |
| 결제 의사 (구매 의사) | 본인은 무료 강하게 선호, 학부모 결제 가능성 있음 → **학생용 핵심 기능 무료 + 학부모 리포트 유료** 가설 `[DECISION]` |
| 사용 채널 | 모바일 70% / 노트북(PC) 30% — **모바일 우선 설계** `[DECISION]` |

### 1.2 Secondary 페르소나 — 박미경 (지민이 어머니, 47세) `[DECISION: 2차 타깃 / 결제자]`

| 항목 | 내용 |
|------|------|
| 직업 / 가족 | 직장인, 자녀 1명 (지민) |
| 디지털 친숙도 | 중간 (카카오톡/네이버/구글 검색은 능숙, 노션/대시보드형 UI는 학습 필요) |
| 핵심 통증 (Pain) | 1) 입시 정보가 매년 바뀌어 따라잡기 어려움 (2028 개편 특히 혼란)<br>2) 학원 컨설팅 비용(월 30~80만원) 부담<br>3) 자녀와의 대화에서 학습/진로 화제로 갈등<br>4) "잘 하고 있는지" 객관적 확인 어려움 |
| 원하는 성과 (Gain) | 1) 자녀의 현재 위치와 다음 단계가 한눈에 보이는 요약<br>2) 갈등 없이 자녀와 진학 대화를 시작할 수 있는 공통 자료<br>3) 학원 등록 전 "정말 필요한지" 객관 판단 |
| 사용 빈도 | 월 2~4회 (학기 시작/방학/시험 직후) |
| 결제 의사 | **결제자 (월 9,900~19,900원 가설)** `[DECISION]` |
| 사용 채널 | 모바일 50% / PC 50% |

### 1.3 Secondary 페르소나 — 이서연 (고1~고2 재학생, 16~17세) `[DECISION: 보조 타깃]`

| 항목 | 내용 |
|------|------|
| 학년 | 고1 1학기 또는 고2 1학기 재학 중 |
| 핵심 통증 (Pain) | 1) 중간고사·수행평가·학평이 동시에 몰려 우선순위 혼란<br>2) 세특·동아리 자료 정리 압박 (작년에 미리 했어야 했는데...)<br>3) 트랙(학종 vs 정시) 결정이 부담<br>4) 친구마다 다른 전략 — 내 케이스에 맞는 가이드가 없음 |
| 원하는 성과 (Gain) | 1) 이번 주/이번 달 우선순위 명확화<br>2) 누적된 활동을 세특·자기소개서(폐지됐지만) 형태로 정리<br>3) 트랙 결정의 객관적 근거 (내신/모의 데이터 기반) |
| 사용 빈도 | 주 2~3회 (시험·학평 시기 집중) |
| 결제 의사 | 본인 결제 의사 약함, 부모 결제 활용 |

### 1.4 페르소나 통증 → 기능 매핑 (요약)

| 통증 | 매핑 기능 (MVP) |
|------|----------------|
| 막막함 ("뭐부터 해야 하지") | 학년·학기 그리드 TODO 로드맵 |
| 진로 흐릿함 | 흥미 가설 + 검증 활동 추천 |
| 정보 신뢰성 | 추천 근거(explanation) 노출 + 면책 표시 |
| 시간 부족 / 우선순위 혼란 | 주간 체크인 + 오늘의 TODO 5개 (밸런스 룰) |
| 학부모-자녀 갈등 | 학부모 요약 리포트 (별도 화면) |
| 입시 룰 변화 (2028) | 입시 사이클 자동 분기 + 컨텍스트 배너 |

---

## 2. 가치 제안 (Value Proposition)

### 2.1 한 문장 가치 제안 `[DECISION]`

> **"중학교 생기부 한 장으로 시작해, 고등학교 3년의 모든 결정 시점을 미리 안내받는 첫 번째 진학 코치"**

### 2.2 핵심 차별점 (3가지) `[DECISION]`

1. **PDF → 즉시 로드맵**: 컨설팅처럼 1:1 상담을 기다릴 필요 없이, 업로드 30초 내 학년 × 학기 × 카테고리 격자 TODO를 받는다.
2. **결정 포인트 중심**: 단순 체크리스트가 아니라 "선택과목 결정 D-30", "트랙 잠정 결정 D-60" 같은 **결정 도우미** UI.
3. **2028 개편 자동 반영**: 사용자 출생연도 기반 입시 사이클 자동 분기 — 다른 서비스가 따라잡지 못하는 정책 변화 적응성.

### 2.3 경쟁 비교

| 항목 | 학원 컨설팅 (예: 종로학원/메가스터디) | 진학사 (모의지원 SaaS) | 메가스터디 마이맥 (학습) | **본 서비스 (student-helper)** |
|------|--------------------------------------|----------------------|------------------------|------------------------------|
| 입력 데이터 | 학생 면담 + 성적표 수동 입력 | 모의/내신 수동 입력 | 강의 수강 이력 | **생기부 PDF 자동 파싱** |
| 가격 | 월 30~80만원 (1:1) | 월 1~3만원 | 월 5~10만원 | **무료 + 프리미엄 1~2만원대** `[DECISION]` |
| 코칭 깊이 | 매우 높음 (사람) | 합격 시뮬에 한정 | 학습 중심 | **TODO·결정 도우미 (자동화)** |
| 학년 범위 | 고1~고3 위주 | 고2~고3 | 고1~고3 | **예비 고1 (중3 겨울방학)부터** ← 차별 |
| 2028 개편 대응 | 강사별 편차 큼 | 시뮬 중심이라 부분적 | 강의 갱신 속도 느림 | **룰셋 외부화로 빠른 반영** |
| 학부모 뷰 | 별도 미팅 | 약함 | 약함 | **학부모 요약 리포트 내장** |
| 진입 장벽 | 비용·지역 | 가입+수동 입력 | 강의 결제 | **PDF 업로드 1회** |

### 2.4 차별화 포지셔닝 한 줄

> "학원 컨설팅의 1/30 가격으로, **첫 30초 안에** 진학사보다 풍부하고 학원 컨설팅보다 따라가기 쉬운 로드맵을 제공한다."

---

## 3. MVP 기능 범위 (MoSCoW)

> 6개월 안에 한 명의 풀스택 + 디자이너 + 도메인 자문이 출시 가능한 범위로 제안. 도메인 인사이트 §12(프로덕트 결정 체크리스트) 항목을 모두 매핑한다.

### 3.1 Must Have (MVP, 6개월 내 필수) `[DECISION]`

| # | 기능 | 한 줄 설명 | 핵심 입력 → 출력 | 도메인 인사이트 매핑 |
|---|------|-----------|-----------------|--------------------|
| M1 | **회원가입·역할 분기 (학생/학부모)** | 학생 또는 학부모로 가입, 14세 미만 시 보호자 동의 흐름 | 이메일/소셜 + 출생연도 + 역할 → 계정 | §7.2 (개정 개인정보보호법 동의), §12.1 |
| M2 | **생기부 PDF 업로드 + 파싱** | NEIS 표준 PDF를 항목별로 추출, 실패 시 OCR 폴백 + 수동 보정 | PDF 파일 → ParsedReportCard 엔티티 (학년·학기별 항목 JSON) | §1.1 항목 매핑, §1.2 형식 특징, §9 데이터 플로우 [1][2] |
| M3 | **신호 추출 + 학생 프로파일 생성** | 흥미 가설, 강·약점, 트랙 가설을 자동 산출 | ParsedReportCard + 학년/사이클 → StudentProfile (interest_hypothesis, strengths, weaknesses, track_hypothesis) | §1.3 신호 강도, §5 개인화 신호, §9 [3][4] |
| M4 | **TODO 로드맵 자동 생성 (학년 × 학기 × 카테고리 × 트랙)** | 룰 엔진 시드 + LLM 개인화로 30개 이상 TODO 생성, 우선순위·근거 포함 | StudentProfile → Todo[] (category A~E, priority, due_term, explanation) | §2 학년별 과업, §4 카테고리, §11 시드, §9 [5][6] |
| M5 | **트랙 듀얼 추천 (주력 + 보조)** | 정시/학종/교과/논술/실기 중 주력+보조 트랙 추천 + 근거 | StudentProfile → TrackRecommendation (primary, secondary, confidence, signals) | §3.1, §3.2 |
| M6 | **학년·학기 격자 대시보드 + 오늘의 TODO 5개** | 메인 화면. 격자 + 오늘 카드 + 결정 포인트 배너 | Todo[] + 현재 날짜 → 정렬된 뷰 | §2 결정 포인트, §4.3 우선순위, §4.4 상태 모델 |
| M7 | **주간 체크인 (TODO 진행 / 회고)** | 주 1회 푸시·알림 → 완료/스누즈/스킵 마킹, 다음 주 재추천 | 사용자 액션 → Todo.state 갱신 + 추천 보정 | §4.4 상태 모델, §5 가설→검증 루프 |
| M8 | **학부모 요약 리포트 (PDF/공유 링크)** | 자녀 프로파일·로드맵·이번 달 핵심을 1~2장 요약 | StudentProfile + Todo[] → 요약 뷰 + 공유 링크 | 페르소나 1.2, §1 학부모 통증 |

### 3.2 Should Have (3~6개월 내 가능, 후순위) `[DECISION]`

| # | 기능 | 한 줄 설명 |
|---|------|-----------|
| S1 | 모의/내신 점수 수동 입력 | 학평·평가원 모의 등급, 학기별 내신 입력 → 트랙 추천 정밀화 |
| S2 | 결정 도우미 (선택과목/트랙) | 의사결정 트리 UI + 비교표 (예: 미적분 vs 확통) |
| S3 | 모집요강 캐시 조회 | 사용자가 지정한 5~10개 대학 모집요강 요약 (사용자 입력+캐시) |
| S4 | 푸시·이메일 알림 | 마감 D-7/D-1 알림, 주간 체크인 리마인드 |
| S5 | 다중 자녀 (학부모 계정) | 학부모 1명이 자녀 여러 명 관리 |

### 3.3 Could Have (v2 이후) `[DECISION: MVP 제외]`

| # | 기능 | 비고 |
|---|------|------|
| C1 | 카메라 스캔 입력 | 모바일 카메라로 종이 생기부 촬영 → OCR |
| C2 | 학교/지역 통계 비교 | 익명화 통계 ("같은 지역 학생들의 평균 TODO 완료율") |
| C3 | AI 챗봇 Q&A | "학종 세특이 뭐야?" 등 도메인 질의응답 |
| C4 | 진로 캠프·프로그램 추천 (외부 연계) | 공식 자원 (EBS, 학교 프로그램) 큐레이션 |
| C5 | 합격 시뮬레이션 (수시 6장 / 정시 3장) | 진학사 유사 기능 (라이선스/데이터 이슈 큼) |

### 3.4 Won't Have (명시적 제외) `[DECISION]`

| # | 항목 | 이유 |
|---|------|------|
| W1 | 직업계(특성화고) 트랙 | §10 — 입시 전형 자체 다름, 별도 도메인 |
| W2 | 검정고시·해외 IB/AP | §10 — 생기부 부재 또는 양식 다름 |
| W3 | 학교폭력 조치사항·보건 항목 추출 | §7.1 — 추출 자체 회피 (시스템 정책) |
| W4 | 사교육 학원·강사 추천 | §8 — 사교육 권유 오해 회피, 공식 자원 우선 |
| W5 | 자기소개서 작성 도우미 | §6.1 — 2024년부터 대입 자소서 폐지 |
| W6 | 합격 보장·예측 점수 | §8 — 클레임 위험, 면책 톤 유지 |

### 3.5 도메인 인사이트 §12 체크리스트 매핑 `[DECISION]`

| 인사이트 §12 항목 | 청사진 결정 |
|------------------|-----------|
| 12.1 타깃 사용자 세그먼트 | 학생 본인 + 학부모 (역할 분기, M1) |
| 12.1 타깃 학년 범위 | 중3(예비 고1) ~ 고2 1차, 고3은 v2 (Won't에는 없으나 우선순위 ↓) |
| 12.1 MVP 학교 유형 | 일반계 한정 (W1) |
| 12.1 지역 범위 | 전국 (한국어 only) |
| 12.2 PDF 입력 채널 | 업로드 only (카메라는 C1) |
| 12.2 OCR 사용 여부 | 필요 (텍스트 추출 실패 시 폴백) — M2 |
| 12.2 수동 보정 UI | 항목별 텍스트 편집 가능 — M2 |
| 12.2 민감정보 동의 흐름 | 분리 동의 + 14세 미만 보호자 동의 — M1 |
| 12.3 TODO 단위 | **학기 단위 + 주 단위 슬라이스** `[DECISION]` |
| 12.3 카테고리 체계 | §4 5×다 분류 채택 (A 내신, B 수능, C 세특, D 입시, E 생활) `[DECISION]` |
| 12.3 우선순위 알고리즘 | **가중치 기반** (시간 임박도×0.4 + 학년 단계×0.2 + 트랙 적합도×0.2 + 약점 신호×0.1 + 흥미 신호×0.1) `[DECISION]` |
| 12.3 추천 근거(explanation) 노출 | **O** — 모든 TODO에 1~2문장 근거 표시 `[DECISION]` |
| 12.3 톤 가이드라인 | 학생용 / 학부모용 분리 `[DECISION]` |
| 12.4 트랙 추천 방식 | **듀얼 (주력+보조)** `[DECISION]` |
| 12.4 신뢰도 표기 | **정성 라벨 (높음/중간/낮음) + 신호 카운트** `[DECISION]` |
| 12.4 모집요강 데이터 소스 | MVP는 사용자 입력 + 캐시 (S3), v2에서 공식 API 검토 |
| 12.4 2028 룰셋 분기 | **출생연도 → 입학년도 자동 계산** `[DECISION]` |
| 12.5 저장 정책 | **서버 암호화 저장** (PDF 원본은 30일 후 자동 삭제) `[DECISION]` |
| 12.5 원본 PDF 보존 | **30일** `[DECISION]` |
| 12.5 민감정보 폐기 | **추출 즉시** (주민번호 앞6자리, 가족관계, 주소, 학폭, 보건) `[DECISION]` |
| 12.5 로그 보관 | **열람 로그 1년** `[DECISION]` |
| 12.6 면책 위치 | **결과 화면 + TODO 항목별 모두** `[DECISION]` |
| 12.6 사교육 추천 | 비추천 (공식 자원 우선) `[DECISION]` |
| 12.6 수상·독서 안내 | 흥미 신호로만 + 명시적 미반영 안내 `[DECISION]` |
| 12.6 학폭·보건 | 추출 안 함 (시스템 정책) `[DECISION]` |
| 12.7 핵심 KPI | TODO 완료율 / 7일 리텐션 / 추천 수용률 (§8 참조) |
| 12.8 응답 시간 | 업로드→TODO 생성 ≤ 30초 (목표 20초) `[DECISION]` |
| 12.8 오프라인 / 동기화 | 미지원 / 미지원 (웹만) `[DECISION]` |

---

## 4. 핵심 사용자 흐름 (User Flows)

### Flow 1 — 온보딩 + 생기부 업로드 + 첫 로드맵 생성 (Primary Path)

> 페르소나: 김지민 (예비 고1) 또는 이서연 (고1)
> 목표: 가입 후 30분 안에 본인 맞춤 TODO 로드맵 첫 화면 도달

| Step | 화면 / 액션 | 시스템 처리 | 예상 소요 |
|------|-------------|-------------|----------|
| 1 | 랜딩 → "무료로 시작" 클릭 | — | 5초 |
| 2 | 회원가입 (이메일 또는 카카오/구글 소셜) | 계정 생성, 14세 미만 시 보호자 동의 분기 | 30초 |
| 3 | 역할 선택 (학생 / 학부모) + 출생연도 입력 | 입시 사이클 자동 계산 (예: 2010년생 → 2028 개편 적용) | 15초 |
| 4 | 학교 유형 선택 (일반계 / 자사·특목) + 거주 지역 (시도) | 일반계 외 선택 시 "MVP 미지원" 안내 (W1) | 10초 |
| 5 | **민감정보 처리 동의** (분리 체크 3개: 필수 처리 / PDF 30일 보관 / 학부모 공유) | 동의 미체크 시 진행 차단 | 30초 |
| 6 | **PDF 업로드** (드래그&드롭 또는 파일 선택) | 클라이언트에서 1차 텍스트 추출 시도 → 실패 시 서버 OCR 폴백 | 5~25초 |
| 7 | 파싱 진행 화면 (프로그레스 바 + 단계 표시) | "항목 추출 중 → 신호 분석 중 → 로드맵 생성 중" | 15~30초 |
| 8 | **수동 보정 화면** (선택 사항) — 추출 신뢰도 낮은 항목 빨간 테두리, 사용자가 수정 | 사용자 수정 시 ParsedReportCard 갱신 | 1~5분 (선택) |
| 9 | **트랙 듀얼 추천 결과** — 주력+보조, 신뢰도, 근거 신호 | 사용자가 "동의" 또는 "수동 변경" | 30초 |
| 10 | **메인 대시보드** — 학년 격자 + 오늘의 TODO 5개 + 결정 포인트 배너 | 진입 완료 | — |

**분기 / 예외:**
- **E1 — PDF 파싱 완전 실패**: OCR도 실패 시 "수동 입력 모드" 진입 (간소화 폼: 학년별 주요 활동 텍스트 입력) → Step 9로 점프.
- **E2 — 14세 미만**: Step 2 직후 보호자 이메일 입력 → 보호자 동의 링크 발송 → 보호자 동의 완료 후 Step 3 진행.
- **E3 — 신호 부족 (생기부 분량 적음)**: Step 9에서 트랙 추천 대신 "**탐색 모드**" 안내 + 진로 검사 유도 TODO.

### Flow 2 — TODO 로드맵 첫 조회 + 우선순위 이해

> 페르소나: 김지민 (Flow 1 직후) 또는 박미경 (학부모 공유 링크 진입)
> 목표: 학년 전체 그림 → 이번 학기 → 이번 주 → 오늘 순으로 자연스럽게 줌인

| Step | 화면 / 액션 | 시스템 처리 |
|------|-------------|-------------|
| 1 | 메인 대시보드 진입 (격자 뷰가 기본) | 학년 × 학기 격자, 현재 학기 하이라이트 |
| 2 | 학기 셀 탭 → 학기 상세 (해당 학기 TODO 목록, 카테고리별 그룹) | TODO 정렬 (우선순위 desc) |
| 3 | TODO 카드 탭 → 상세 (설명, 추천 근거, 마감일, 카테고리, 시작/완료 버튼) | — |
| 4 | "시작하기" 클릭 → 상태 in_progress, "노트" 입력 가능 | Todo.state 갱신 |
| 5 | "오늘 화면" 탭 전환 → 오늘 5개 (각 카테고리 1개씩 밸런스) | 우선순위 + 밸런스 룰 |
| 6 | TODO 완료 마킹 → 짧은 회고 (5점 만족도 + 한 줄 코멘트) | 추천 보정 데이터 |

**분기 / 예외:**
- **E1 — TODO 너무 많아 압도됨**: "이번 주 5개만 보기" 토글 (밸런스 룰 강제).
- **E2 — TODO 거부 (Skip)**: 사유 입력 (3가지 옵션: "이미 함" / "관심 없음" / "시기 안 맞음") → 추천 보정.

### Flow 3 — 주간 체크인 (Retention Loop)

> 페르소나: 김지민 (가입 후 1주차~매주)
> 목표: 매주 일요일 저녁, 5분 안에 지난 주 회고 + 다음 주 우선순위 확정

| Step | 화면 / 액션 | 시스템 처리 |
|------|-------------|-------------|
| 1 | 일요일 18:00 푸시·이메일 알림 ("이번 주 체크인 5분") | 알림 발송 |
| 2 | 체크인 화면 진입 — 지난 주 TODO 목록 (완료 / 미완료) | — |
| 3 | 미완료 TODO에 빠른 액션 (다음 주로 / 스누즈 / 스킵) | Todo.state, snooze_until |
| 4 | "이번 주 한 줄 회고" 입력 (자유 텍스트, 최대 200자) | WeeklyCheckIn 저장 |
| 5 | 다음 주 추천 TODO 5개 자동 생성 (이전 주 데이터로 보정) | 추천 엔진 재실행 |
| 6 | 사용자 확인 → 다음 주 시작 | — |

**분기 / 예외:**
- **E1 — 2주 이상 미접속**: 재참여 푸시 ("로드맵이 1주 밀렸어요. 5분만 정리해볼까요?") + 자동 스누즈 처리.
- **E2 — 학기 전환 시점 (3월/9월/방학)**: 체크인 대신 "**학기 시작 준비**" 특별 화면 (선택과목/트랙 결정 도우미 등).

---

## 5. 화면 구성 (Information Architecture)

### 5.1 라우트 / 화면 목록 `[DECISION: Next.js App Router 기준]`

| # | 라우트 | 화면명 | 주 사용자 | 핵심 컴포넌트 / 데이터 |
|---|--------|--------|----------|----------------------|
| 1 | `/` | 랜딩 | 비로그인 | 가치 제안, 데모, 가입 CTA |
| 2 | `/auth/signup` | 회원가입 | 비로그인 | 이메일·소셜·14세미만 분기 |
| 3 | `/auth/login` | 로그인 | 비로그인 | — |
| 4 | `/onboarding/role` | 역할/출생연도 입력 | 신규 가입자 | role(학생/학부모), birth_year, school_type, region |
| 5 | `/onboarding/consent` | 민감정보 동의 | 신규 가입자 | 분리 체크 3종 + 보호자 동의 (조건부) |
| 6 | `/onboarding/upload` | PDF 업로드 + 파싱 진행 | 신규 가입자 | 업로드, 진행 상태, 에러 폴백 |
| 7 | `/onboarding/correct` | 수동 보정 | 신규 가입자 | 항목별 추출 결과, 신뢰도 색상, 인라인 편집 |
| 8 | `/onboarding/track` | 트랙 추천 결과 | 신규 가입자 | 듀얼 추천 카드, 신호 근거, 수동 변경 |
| 9 | `/dashboard` | **메인 대시보드 (격자)** | 학생 | 학년×학기 격자, 결정 포인트 배너, 오늘 TODO 5 |
| 10 | `/today` | 오늘의 TODO | 학생 | 5개 카드, 카테고리 밸런스, 빠른 액션 |
| 11 | `/roadmap/[grade]/[term]` | 학기 상세 | 학생 | 카테고리별 TODO 그룹, 정렬·필터 |
| 12 | `/todo/[id]` | TODO 상세 | 학생 | 설명, 근거, 노트, 상태 변경, 회고 |
| 13 | `/checkin` | 주간 체크인 | 학생 | 지난 주 회고, 다음 주 5개 |
| 14 | `/profile` | 학생 프로파일 | 학생 / 학부모 | 흥미 가설, 강·약점, 트랙, 입시 사이클 |
| 15 | `/parent/report` | 학부모 요약 리포트 | 학부모 | 1~2장 요약, 공유 링크, PDF 다운로드 |
| 16 | `/settings/privacy` | 개인정보·삭제 | 모든 사용자 | PDF 삭제, 계정 삭제, 내보내기 |
| 17 | `/settings/account` | 계정 설정 | 모든 사용자 | 알림, 비밀번호, 자녀 추가 (학부모) |

> **MVP 화면 수: 17개** (목표 8~15개 대비 약간 초과 — 온보딩 단계가 보안·동의로 분리 필수)

### 5.2 네비게이션 구조 `[DECISION]`

- **모바일 (≤ 768px)**: 하단 탭 4개 — `오늘` / `로드맵` / `체크인` / `프로필`. 학부모는 `리포트` 탭으로 대체.
- **태블릿/PC (≥ 769px)**: 좌측 사이드바 (5개 항목 + 설정). 메인은 70/30 분할 (격자 + 오늘 TODO 패널).
- **모달 정책**: TODO 상세, 동의 확인, 삭제 확인은 모달. 온보딩은 풀페이지(라우트). 설정은 풀페이지.

### 5.3 디자인 원칙 (high-level) `[DECISION]`

1. 모바일 우선, 한 손 조작 가능한 터치 영역 (44pt+).
2. 정보 밀도는 **점진적 노출** (격자→학기→TODO 카드 깊이별).
3. 톤: 코칭 (지시 X, 제안 O), 약점 표현은 "다음 단계" 형태로 변환.
4. 색상은 카테고리별 페일 톤 5색 (A 내신=파랑 / B 수능=주황 / C 세특=초록 / D 입시=보라 / E 생활=회색).
5. 접근성: 색상 외 아이콘 동시 사용, 대체텍스트, 키보드 전체 탐색 가능.

---

## 6. 개념 데이터 모델

> 필드 디테일·타입은 spec-writer가 채움. 본 절은 엔티티·관계·핵심 속성·PII 분리 정책만 정의한다. 모든 결정은 `[DECISION]`.

### 6.1 엔티티 목록 (10개) `[DECISION]`

| # | 엔티티 | 역할 | PII 등급 |
|---|--------|------|----------|
| 1 | **User** | 인증·계정 (학생 본인 또는 학부모) | 중 (이메일, 출생연도) |
| 2 | **Student** | 분석 대상 학생 (User와 1:1 또는 학부모-자녀 N:1) | 중 |
| 3 | **ParentChildLink** | 학부모-자녀 관계 + 동의 상태 | 중 |
| 4 | **ParsedReportCard** | 생기부 PDF 파싱 결과 (학년별·항목별) | 높음 (마스킹 후 저장) |
| 5 | **PdfFile** | 원본 PDF 메타 (저장 경로, 보존 기간) | 매우 높음 (30일 자동 삭제) |
| 6 | **StudentProfile** | 신호 추출 결과 (흥미·강·약·트랙 가설) | 중 |
| 7 | **TrackRecommendation** | 트랙 듀얼 추천 결과 + 근거 신호 | 낮음 |
| 8 | **Todo** | 단일 할 일 (학년·학기·카테고리·우선순위·상태·근거) | 낮음 |
| 9 | **WeeklyCheckIn** | 주간 회고 + 추천 보정 데이터 | 낮음 |
| 10 | **AuditLog** | 민감정보 열람·삭제 감사 로그 | 시스템 |

### 6.2 엔티티 관계 다이어그램 (개념)

```
User ──┬──(1:1)── Student          [학생 본인 가입]
       └──(1:N)── ParentChildLink ──(N:1)── Student   [학부모 가입]

Student ──(1:N)── PdfFile (history)
Student ──(1:N)── ParsedReportCard (학년별)
Student ──(1:1)── StudentProfile (latest)
Student ──(1:N)── TrackRecommendation (history)
Student ──(1:N)── Todo
Student ──(1:N)── WeeklyCheckIn

User    ──(1:N)── AuditLog
```

### 6.3 핵심 속성 (개념 수준) `[DECISION]`

#### User
- id, email, password_hash (또는 oauth_provider+oauth_id), role (student | parent), birth_year, created_at, deleted_at, locale='ko'
- **PII 분리**: 이메일·비밀번호는 별도 테이블/스키마 분리 권장 (spec-writer가 결정)

#### Student
- id, owner_user_id (학생 본인 가입 시 = User.id, 학부모 가입 시 학생 별도 생성), birth_year, school_type (일반계/자사·특목), region (시도), entrance_year (자동 계산), exam_cycle (2025|2026|2027|2028+, 자동 계산)
- **신호 매핑**: birth_year → exam_cycle 자동 분기 (도메인 §6.3)

#### ParentChildLink
- parent_user_id, student_id, consent_status (pending/granted/revoked), granted_at, revoked_at

#### PdfFile
- id, student_id, storage_url (암호화), uploaded_at, **expires_at = uploaded_at + 30일** `[DECISION]`, sha256_hash, file_size, ocr_used (boolean)
- **자동 삭제 잡**: 매일 expires_at 도래 시 storage 객체 + 레코드 삭제 (소프트 삭제 X)

#### ParsedReportCard
- id, student_id, source_pdf_id (PdfFile.id, **삭제 후에도 결과는 유지**), school_level (초/중), grade (1~6), term, sections (JSONB)
- sections JSONB 구조 (도메인 §1.1 11개 항목):
  - `attendance` (출결 카운트만, 사유 텍스트 폐기)
  - `awards` (수상 분야 분류만)
  - `creative_activities` (자율/동아리/봉사/진로 4영역, 텍스트)
  - `subject_grades` (과목별 성취도 A~E 또는 등급)
  - `subject_records` (세특 텍스트, 키워드 추출 결과)
  - `free_semester_outputs` (자유학기 산출물 — **흥미 신호 1순위**)
  - `reading` (분야 분류만)
  - `comprehensive_opinion` (행특 텍스트, 키워드 추출 결과)
- **추출 안 함**: 주민번호 앞6자리, 보호자 정보, 가족관계, 주소 상세, 학교폭력 조치사항, 보건 항목 — 시스템 정책 `[DECISION]`
- **신호 매핑** (도메인 §1.3, §5):
  - `free_semester_outputs.topic` → StudentProfile.interest_hypothesis (강도 ★★★★★)
  - `creative_activities.club` 누적 일관성 → activity_consistency 점수
  - `subject_records.keywords` (NLP) → strengths/weaknesses
  - `comprehensive_opinion.keywords` (NLP) → 인성/공동체역량 신호
  - `attendance.unexcused_count` → attendance_clean (학종 약점 신호)

#### StudentProfile
- id, student_id, version (시간순), interest_hypothesis (JSON: {topic, confidence, evidence[]}[]), strengths (string[]), weaknesses (string[]), track_hypothesis (JSON: {primary, secondary, confidence}), generated_at
- **신호 매핑** (도메인 §5.3 진로 가설 룰): 룰 ID + LLM 보정 두 단계 결과 모두 evidence에 저장하여 explanation 노출.

#### TrackRecommendation
- id, student_id, primary_track (enum: jeongsi | hakjong | gyogwa | nonsul | silgi), secondary_track, confidence_label (high|mid|low), signal_summary (JSON: 어떤 신호가 영향), generated_at, user_override (사용자가 변경한 경우)
- **신호 매핑** (도메인 §3.2): 입력 변수 9개 (grade_avg_main 등) → 추천 룰 매칭 → 결과

#### Todo
- id, student_id, category (A1~E5, 22개 enum, 도메인 §4.2), grade (1~3), term (1H1|1Hsummer|1H2|1Hwinter|2H1|...), title, description, explanation (왜 추천했는지 1~2문장), priority_score (0~100), state (backlog|today|in_progress|done|snoozed|skipped), due_date (optional), completed_at, snooze_until, skip_reason, created_at, source (rule_seed_id | llm_generated)
- **상태 전이**: 도메인 §4.4 모델 그대로 채택
- **신호 매핑**: priority_score 계산식 = 시간임박도×0.4 + 학년단계×0.2 + 트랙적합도×0.2 + 약점신호×0.1 + 흥미신호×0.1

#### WeeklyCheckIn
- id, student_id, week_start (월요일), week_end, completed_count, missed_count, snoozed_count, skipped_count, retro_text (≤200자), satisfaction_score (1~5), generated_at

#### AuditLog
- id, user_id, target_student_id, action (view_pdf | delete_pdf | export_profile | parent_view), ip, user_agent, occurred_at
- **보존 기간**: 1년 `[DECISION]`

### 6.4 신호 매핑 종합 (인사이트 §5 → 데이터 모델)

| 인사이트 신호 | 저장 위치 | 활용처 |
|---|---|---|
| 자유학기 진로 산출물 (★★★★★) | ParsedReportCard.sections.free_semester_outputs | StudentProfile.interest_hypothesis 1순위 evidence |
| 동아리 누적 일관성 (★★★★) | ParsedReportCard.sections.creative_activities.club + 학년별 비교 | StudentProfile.track_hypothesis activity_consistency 점수 |
| 세특 키워드 (★★★★) | ParsedReportCard.sections.subject_records.keywords | StudentProfile.strengths/weaknesses + Todo 우선순위 보정 |
| 행특 키워드 (★★★★) | ParsedReportCard.sections.comprehensive_opinion.keywords | StudentProfile (공동체역량 평가 보조) |
| 출결 미인정 (★★★) | ParsedReportCard.sections.attendance.unexcused_count | 학종 트랙 confidence 차감 |
| 수상 분야 (★★★) | ParsedReportCard.sections.awards.categories | 흥미 신호 보조 (대입 미반영 명시) |
| 독서 분야 (★★★) | ParsedReportCard.sections.reading.categories | 인문/자연 분기 1차 신호 |
| 봉사 (★★) | ParsedReportCard.sections.creative_activities.volunteer | 학교 봉사만 카운트 |

---

## 7. AI / 추천 엔진 동작 정의

### 7.1 입력 / 출력 계약 `[DECISION]`

**입력:**
```
{
  parsedReportCard: ParsedReportCard,    // 1~3년치
  student: { birth_year, entrance_year, exam_cycle, school_type, region },
  context: { current_grade (1~3 | -1=중학생), current_term, today },
  user_pref: { track_preference?, university_targets?[] }
}
```

**출력:**
```
{
  studentProfile: {
    interest_hypothesis: [{topic, confidence: 0~1, evidence: [signal_id]}],
    strengths: [string],
    weaknesses: [string],
    track_hypothesis: {primary, secondary, confidence_label}
  },
  trackRecommendation: TrackRecommendation,
  todos: [Todo]    // 30~80개 (3년치 시드 + 개인화)
}
```

### 7.2 처리 단계 (도메인 §9 데이터 플로우 매핑)

```
Stage 1: PDF 파싱
   ├─ pdfminer/pdfplumber로 텍스트 추출 시도
   ├─ 한글 폰트 미임베디드/스캔본 감지 → OCR 폴백 (Tesseract Ko 또는 클라우드 OCR)
   └─ 항목 앵커 키워드(§1.2) 매칭 → sections JSONB

Stage 2: 신호 추출
   ├─ 정량: 등급·출결·봉사 시간 카운트
   ├─ 정성: 세특·행특 키워드 추출 (LLM 또는 한국어 NLP)
   └─ 분류: 자유학기·동아리·독서 주제 분야 라벨링

Stage 3: 학생 프로파일 생성
   ├─ 흥미 가설 = 자유학기(가중치 5) + 동아리 일관성(4) + 수상 분야(3)
   ├─ 강·약점 = 교과 성취 + 세특 키워드 + 행특 키워드
   └─ 트랙 가설 = §3.2 알고리즘 매트릭스 적용

Stage 4: TODO 생성 (룰 + LLM 하이브리드)
   ├─ 룰 엔진: 학년·학기·트랙별 시드 TODO 로드 (§11 시드 JSON)
   ├─ 필터: 시기 안 맞는 시드는 snoozed 상태로
   ├─ LLM: 시드 TODO의 title/description/explanation을 학생 프로파일 반영하여 개인화
   └─ 우선순위 계산: priority_score 공식

Stage 5: 검증 / 톤 가드
   ├─ 약점 표현 변환 ("부족" → "다음 단계")
   ├─ 사교육 직접 추천 차단
   └─ 면책 문구 자동 첨부
```

### 7.3 추천 로직 의사코드 `[DECISION]`

```pseudo
function generateRoadmap(input):
  profile = analyzeSignals(input.parsedReportCard, input.student)

  # 트랙 추천
  tracks = recommendTracks(profile, input.student.exam_cycle)
  # → primary + secondary, 도메인 §3.2 매트릭스

  # 시드 TODO 로드
  seed_todos = loadSeedTodos(
    grades = [1,2,3],
    exam_cycle = input.student.exam_cycle,
    track_primary = tracks.primary,
    track_secondary = tracks.secondary
  )

  # 개인화 — LLM
  personalized = []
  for seed in seed_todos:
    if shouldSnooze(seed, input.context):
      seed.state = 'snoozed'
    else:
      seed.title, seed.description, seed.explanation = LLM.personalize(
        seed_template = seed,
        profile = profile,
        tone = "encouraging-coach",
        context = input.context,
        guard_rails = ["no_paid_education", "no_admission_guarantee", "weakness_to_next_step"]
      )
    seed.priority_score = computePriority(seed, profile, input.context)
    personalized.append(seed)

  # 정렬 + 밸런스 (오늘 5개는 카테고리 밸런스 룰)
  return {
    profile, tracks,
    todos = personalized.sortBy(priority_score desc)
  }


function computePriority(todo, profile, context):
  return (
    timeUrgency(todo.due_date, context.today) * 0.4 +
    gradeStageFit(todo.grade, context.current_grade) * 0.2 +
    trackFit(todo.category, profile.track_hypothesis) * 0.2 +
    weaknessFit(todo.category, profile.weaknesses) * 0.1 +
    interestFit(todo.category, profile.interest_hypothesis) * 0.1
  ) * 100


function recommendTracks(profile, exam_cycle):
  # 도메인 §3.2 매트릭스 — 룰 매칭
  if profile.grade_avg_main <= 2.5 and profile.senthuk_density == 'high' and profile.activity_consistency == 'high':
    return {primary: 'hakjong', secondary: 'gyogwa', confidence: 'high'}
  if profile.grade_avg_main <= 1.5 and profile.senthuk_density in ['mid','low']:
    return {primary: 'gyogwa', secondary: 'hakjong', confidence: 'high'}
  if profile.mock_main_avg <= 2.0 and profile.grade_trend == 'flat':
    return {primary: 'jeongsi', secondary: 'nonsul', confidence: 'mid'}
  if profile.has_practical_records:
    return {primary: 'silgi', secondary: 'jeongsi', confidence: 'mid'}
  # 신호 부족 (중학생 입력)
  return {primary: 'undecided', secondary: null, confidence: 'low', message: '탐색 모드'}
```

### 7.4 룰 엔진 vs LLM 책임 분담 `[DECISION]`

| 책임 | 룰 엔진 | LLM |
|------|--------|-----|
| **무엇을 시켜야 하는가 (What)** | ✅ 학년·학기·트랙 시드 22개 카테고리 누락 방지 | — |
| **언제 보여줘야 하는가 (When)** | ✅ 시기 매칭, 자동 스누즈 | — |
| **어떻게 표현하는가 (How)** | — | ✅ 톤·문장·개인화 |
| **왜 이걸 추천했는가 (Why)** | ✅ 신호 ID + 룰 ID | ✅ 자연어 1~2문장 |
| **우선순위 (Priority)** | ✅ 가중치 공식 | — |
| **2028 개편 룰 적용** | ✅ 외부 JSON 룰셋 분기 | — |

> **원칙**: 룰 엔진이 누락 없이 시드를 깔고 LLM은 표현·개인화만. LLM이 시드 자체를 만들지 않게 한다 (할루시네이션 통제).

### 7.5 룰셋 외부화 `[DECISION]`

- 시드 TODO·트랙 매트릭스·진로 가설 룰은 **외부 JSON/YAML 파일**로 관리.
- 위치: `src/rules/seed_todos.{exam_cycle}.json`, `src/rules/track_matrix.json`, `src/rules/career_hypothesis.json`
- 매년 입시 변화 반영 시 코드 배포 없이 룰 파일만 갱신 → 도메인 §6.3 (2028 개편 대응) 핵심 요구.

---

## 8. 성공 지표 (KPI)

### 8.1 North Star Metric `[DECISION]`

> **"Weekly Active Roadmap Followers (WARF)"** — 주간 1회 이상 TODO 상태를 변경(완료/스누즈/스킵)한 활성 학생 수.

가입·1회 사용은 쉽지만, **로드맵을 따라가는** 행동이 진짜 가치 발생 시점이라 판단.

### 8.2 Primary KPIs (5개) `[DECISION]`

| # | KPI | 정의 | Phase별 목표 |
|---|-----|------|-------------|
| K1 | **PDF 업로드 → 로드맵 도달률** | 가입자 중 첫 로드맵 화면 도달 비율 | M3 60% → M6 80% |
| K2 | **D7 리텐션** | 가입 후 7일 내 재방문 비율 | M3 25% → M6 40% |
| K3 | **주간 체크인 완료율** | 활성 학생 중 주간 체크인 완료 비율 | M3 30% → M6 50% |
| K4 | **TODO 추천 수용률** | 추천된 TODO 중 done+in_progress / 전체 (skipped 제외) | M3 40% → M6 60% |
| K5 | **유료 전환율 (학부모)** | 학부모 가입자 중 프리미엄 결제 비율 | M3 5% → M6 10% |

### 8.3 보조 KPI (품질 / 안정성)

| # | KPI | 목표 |
|---|-----|------|
| Q1 | PDF 파싱 성공률 (수동 보정 X) | ≥ 70% |
| Q2 | 업로드→로드맵 응답 시간 (P95) | ≤ 30초 |
| Q3 | 추천 만족도 (TODO 회고 평균 점수) | ≥ 3.5 / 5 |
| Q4 | 30일 내 PDF 자동 삭제 잡 성공률 | 100% |
| Q5 | NPS (3개월 사용자 대상) | ≥ 30 |

### 8.4 Phase별 가설 수치

| Phase | 기간 | 누적 가입자 | WARF | 비고 |
|-------|------|-------------|------|------|
| M0 (출시) | 6개월 | 1,000 | 200 | 초기 사용자 베타 |
| M3 (출시 + 3개월) | 9개월 | 5,000 | 1,500 | 입소문·SEO |
| M6 (출시 + 6개월) | 12개월 | 15,000 | 6,000 | 학부모 채널 활성화 |

---

## 9. 개인정보·보안 정책 (개념 수준)

### 9.1 PDF 보관 정책 `[DECISION]`

| 항목 | 정책 |
|------|------|
| 저장 위치 | 서버 객체 스토리지 (S3 호환), 필드별 AES-256 암호화 |
| 보존 기간 | **30일** (사용자 명시 동의 필요) |
| 자동 삭제 | 매일 자정 배치, expires_at 도래 시 객체+레코드 삭제 |
| 수동 즉시 삭제 | `/settings/privacy`에서 1클릭 삭제 |
| ParsedReportCard와의 관계 | PDF 삭제 후에도 마스킹된 ParsedReportCard는 유지 (사용자가 별도 삭제 필요) |

### 9.2 마스킹·폐기 룰 (추출 즉시) `[DECISION]`

| 항목 | 처리 |
|------|------|
| 주민번호 앞 6자리 | **즉시 폐기** (저장 안 함) |
| 보호자 성명·관계·직업 | **즉시 폐기** |
| 가족관계 (형제 학년 등) | **즉시 폐기** |
| 주소 상세 | 시도 단위로 추상화 후 상세 폐기 |
| 학교명 | 학교 유형(일반/자사·특목)로 추상화, 학교명은 사용자 동의 시만 저장 |
| 학교폭력 조치사항 | **추출 자체 안 함** (시스템 정책) |
| 보건·신체발달 | **추출 자체 안 함** |
| 출결 사유 텍스트 | 카운트만 저장, 텍스트 폐기 |

### 9.3 동의 흐름 (단계별) `[DECISION]`

1. **필수 동의** (서비스 이용 가능 조건):
   - 이용약관, 개인정보 처리방침, 민감정보 처리(생기부 분석)
2. **선택 동의**:
   - PDF 30일 보관 (미동의 시 24시간 내 자동 삭제)
   - 학부모와 결과 공유 (학생 가입 시)
   - 통계·서비스 개선 활용 (가명화)
3. **분리 동의 UX**: 체크박스 묶음 금지, 항목별 별도 체크.

### 9.4 14세 미만 처리 `[DECISION]`

- 가입 시 출생연도 → 만 14세 미만 자동 판별
- 보호자 이메일 입력 → 동의 링크 발송 → 보호자 본인인증 + 동의 완료 후 가입 허용
- 보호자 동의 만료 (자녀 만 14세 도달) 시 학생 본인 동의 갱신 필요
- 중학생(만 13~15세) 사용자는 **보호자 동의 흐름 빈도 높음** → UX 친화적 안내 필수

### 9.5 삭제 권리 / 데이터 이동권 `[DECISION]`

- 계정 삭제: 30일 유예 후 모든 데이터 hard delete
- 데이터 내보내기: JSON + PDF (학생 프로파일 + TODO 이력)
- 열람 로그 (AuditLog): 사용자가 자신의 PII 열람 이력 조회 가능

---

## 10. 비기능 요구사항 핵심

### 10.1 성능 `[DECISION]`

| 지표 | 목표 |
|------|------|
| PDF 파싱 (텍스트형, ≤ 10MB) | P95 ≤ 8초 |
| PDF 파싱 (OCR 폴백) | P95 ≤ 25초 |
| 추천 엔진 (룰+LLM) | P95 ≤ 12초 (LLM 호출 포함) |
| 업로드 → 로드맵 화면 도달 (전체 파이프라인) | P95 ≤ 30초, P50 ≤ 18초 |
| 대시보드 첫 페인트 | P95 ≤ 2.5초 (모바일 4G) |

### 10.2 가용성 / 확장성

- MVP 가용성 목표: **99.5%** (월 ~3.6시간 다운 허용)
- DB: PostgreSQL 단일 인스턴스 (M0~M3) → M6 시 read replica 추가 검토
- 파싱·추천: 비동기 큐 (BullMQ 등) 분리 — 동시 업로드 폭주 방어
- LLM 호출: 캐시·재시도·타임아웃 (12초) 필수, 실패 시 룰 엔진만으로 fallback

### 10.3 접근성 (학생용 서비스 특화) `[DECISION]`

- WCAG 2.1 AA 수준 목표
- 색상만으로 정보 전달 금지 (카테고리는 색+아이콘+라벨)
- 모든 인터랙티브 요소 키보드 탐색 가능
- 동적 콘텐츠 변경 시 ARIA live region
- **학생 접근성 특화**:
  - 한 번에 보여주는 TODO 수 5개 제한 (인지부하)
  - 모바일 텍스트 최소 16px
  - 다크모드 지원 (수면 위생 — 야간 학습)
  - 한글 가독성 폰트 (Pretendard 권장)

### 10.4 국제화 / 브라우저 지원

- 언어: 한국어 only (MVP), v2 영문 검토
- 브라우저: 최신 2버전 지원 (Chrome / Safari / Edge / Firefox)
- 모바일: iOS 15+, Android 9+

---

## 11. 리스크와 가정

### 11.1 비즈니스 리스크 (3개) `[DECISION]`

| # | 리스크 | 영향 | 완화 전략 |
|---|-------|------|----------|
| B1 | **학원 컨설팅 시장 진입장벽** — 부모는 사람의 1:1 신뢰를 선호 | 유료 전환율 저조 | 무료 학생 사용 → 부모에게 자연스럽게 노출되는 공유 리포트 채널 |
| B2 | **모집요강 데이터 라이선스** — 진학사 등 기존 사업자가 데이터 독점 | S3 모집요강 기능 차질 | MVP는 사용자 입력 + 공식 출처 링크. v2에서 공공 API/제휴 검토 |
| B3 | **학교/교육청 부정적 반응** — 사교육 조장 비판 가능성 | PR·정책 리스크 | 공식 자원(EBS·학교 프로그램) 우선, 사교육 직접 추천 금지 (W4) |

### 11.2 기술 리스크 (3개) `[DECISION]`

| # | 리스크 | 영향 | 완화 전략 |
|---|-------|------|----------|
| T1 | **PDF 파싱 다양성** — 학교마다 양식 변형, OCR 정확도 한계 | 첫 사용 경험 붕괴 (K1 저하) | 텍스트→OCR 2단 폴백 + 수동 보정 UI 필수, 파싱 실패 분석 대시보드로 양식 학습 |
| T2 | **LLM 비용·지연** — 사용자 폭증 시 토큰 비용·응답 지연 | 단가·응답시간 KPI 위협 | 시드 템플릿 캐싱, 동일 프로파일 결과 재사용, 작은 모델로 단계별 호출 |
| T3 | **민감정보 유출 (생기부)** — 보안 사고 시 법적·신뢰 붕괴 | 서비스 종료 수준 | §9 정책 엄수, 외부 보안 감사 출시 전 1회, 침해 통지 SOP |

### 11.3 도메인 정확성 리스크 (3개) `[DECISION]`

| # | 리스크 | 영향 | 완화 전략 |
|---|-------|------|----------|
| D1 | **2028 개편 세부 미확정** — 정책 발표 지연/변경 | 룰셋 부정확 → 잘못된 가이드 | 룰셋 외부화, 발표 시점 빠른 갱신, 사용자에게 "현 시점 가이드라인" 명시 |
| D2 | **트랙 추천 오류** — 학생 진로에 부정적 영향 | 클레임·신뢰 붕괴 | 모든 추천에 "참고용" 면책 + 신뢰도 표시 + 사용자 수동 변경 가능, 추천 근거 투명 노출 |
| D3 | **학년·학기 시기성 오류** — "이미 지난 결정"을 추천 | 사용자 신뢰 손실 | 학사일정 캘린더 외부화, 자동 스누즈 룰, 시기 지난 TODO는 회고용 표시로 변환 |

### 11.4 가정 (Assumptions)

- A1. 한국 일반계 고교 진학 학생이 MVP 99% 차지
- A2. 사용자가 NEIS 표준 PDF를 발급받을 수 있음 (학생 본인 또는 학부모가 학교·NEIS 사이트 통해 가능)
- A3. 학부모의 결제 의사가 학생보다 높다
- A4. 6개월 내 1만 가입자 도달 가능 (SEO + 학부모 커뮤니티 마케팅)
- A5. LLM API (예: Claude / GPT-4o-mini) 호출 단가가 사용자당 월 평균 200원 이내

---

## 12. spec-writer를 위한 메모

### 12.1 기술 스택 권장 사항 (참고) `[DECISION]`

| 영역 | 권장 |
|------|------|
| 프론트엔드 | Next.js 14+ App Router, TypeScript, Tailwind CSS, shadcn/ui |
| 상태/데이터 | React Server Components + Server Actions, TanStack Query (클라 캐시) |
| 백엔드 | Next.js Route Handlers (API), Node 20+ |
| DB / ORM | PostgreSQL 15+, Prisma 5+ |
| 인증 | NextAuth (Auth.js) — 카카오·구글 OAuth + 이메일 |
| 파일 스토리지 | S3 호환 (AWS S3 or Cloudflare R2), 서명 URL |
| PDF 파싱 | `pdfjs-dist` 또는 서버측 `pdfminer.six` (Python 마이크로서비스 옵션) |
| OCR | Tesseract Korean 또는 Google Cloud Vision (한국어 정확도 우수) |
| LLM | Anthropic Claude (한국어 우수, 본 환경 호환) — 캐시·배치 활용 |
| 배경 작업 | BullMQ + Redis (비동기 파싱·LLM 호출) |
| 호스팅 | Vercel (프론트) + Railway/Fly.io (DB/Redis) — MVP |
| 관측성 | Sentry, PostHog (제품 분석), Vercel Analytics |
| 알림 | 이메일 (Resend), 웹푸시 (M3 이후) |

### 12.2 spec에 반드시 반영되어야 할 제약 `[DECISION]`

1. **민감정보 추출 금지 항목**: 주민번호 앞6자리, 가족관계 상세, 주소 상세, 학교폭력, 보건 — 시스템 정책으로 코드에 박을 것 (도메인 §7.1).
2. **룰셋 외부 JSON/YAML**: 시드 TODO·트랙 매트릭스·진로 가설 룰 — 코드 배포 없이 갱신 가능해야 함 (§7.5).
3. **입시 사이클 자동 분기**: birth_year → entrance_year → exam_cycle 자동 계산, 룰셋 분기 (§9 도메인, §6.3 청사진).
4. **PDF 30일 자동 삭제 배치**: 매일 자정, expires_at 만료 객체 hard delete (§9.1).
5. **추천 근거 (explanation) 필수 필드**: 모든 Todo·TrackRecommendation에 1~2문장 근거.
6. **면책 문구**: 결과 화면 + TODO 항목별 — UI 컴포넌트 수준에서 강제.
7. **톤 가드레일**: LLM 호출 시 시스템 프롬프트에 "사교육 직접 추천 금지", "약점→다음 단계", "합격 보장 표현 금지" 명시.
8. **14세 미만 보호자 동의**: 회원가입 흐름에서 분기 화면 + 보호자 이메일 인증.
9. **학년 × 학기 × 카테고리 enum**: 도메인 §4.2의 22개 카테고리 코드 (A1~E5)를 그대로 enum으로.
10. **상태 전이 모델**: `backlog → today → in_progress → done | snoozed | skipped` (§4.4).
11. **응답 시간 SLO**: 업로드 → 로드맵 P95 30초 — 비동기 큐 + 진행 상태 화면 필수.
12. **AuditLog**: PDF 열람·삭제·내보내기 시 자동 기록.

### 12.3 spec 우선 작성 권장 순서

1. 데이터 모델 (10개 엔티티, 관계 + PII 분리 명시)
2. 인증·동의 흐름 (회원가입, 14세 미만, 분리 동의)
3. PDF 업로드·파싱 파이프라인 (큐, OCR 폴백, 수동 보정)
4. 추천 엔진 (룰셋 외부화, LLM 책임 경계)
5. 메인 대시보드·TODO·체크인 UI
6. 학부모 리포트 + 공유 링크
7. 개인정보 설정·삭제·내보내기
8. 보안·감사 로그·자동 삭제 배치
9. 비기능 요구사항 (성능 SLO, 접근성)
10. 면책·톤 가이드라인 (콘텐츠 정책 별도 섹션)

---

## 13. 변경 이력

| 날짜 | 변경 | 작성자 |
|------|------|--------|
| 2026-05-02 | 초안 작성 (도메인 인사이트 기반) | product-strategy |

---

*문서 끝. 다음 단계: `spec-writer`가 본 청사진을 입력으로 `STUDENT_HELPER_SPEC.md` (XML 빌드 스펙)을 작성한다.*
