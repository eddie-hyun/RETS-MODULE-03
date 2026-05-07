# 데이터 설계서 (DDD)

**문서명:** 데이터 설계서 (Data Design Document)
**산출물 ID:** MD05
**모듈:** RETS-MODULE-03 — 요구사항 기반 규모 산정 및 투입 계획 도구
**버전:** 1.0
**작성일:** 2026-05-07
**참고 문서:** `rets_module_03_03_srs.xlsx` (MD03), `rets_02c_req_schema.json`, `rets_02c_req_schema_explanation.md`

---

## 1. 개요

### 1.1 목적

본 문서는 RETS-MODULE-03 내에서 처리하는 주요 데이터의 구조, 속성, 관계, 제약 조건, 입출력 관점의 데이터 항목을 정의하고, 각 데이터 객체와 SRS 요구사항 간의 추적 관계를 정리한다.

### 1.2 데이터 처리 방식

RETS-MODULE-03은 백엔드 서버 없이 순수 브라우저 기반(Vanilla JS)으로 동작하므로, 모든 데이터는 다음 방식으로 관리된다.

| 저장 위치 | 대상 데이터 | 특징 |
|-----------|-------------|------|
| **메모리(Runtime)** | 요구사항 배열, LLM 추천 임시 상태, 간트 계획 | 탭 닫기 시 소멸 |
| **LocalStorage** | 프로젝트 설정, 인력 구성, 언어/테마 설정 | 브라우저 재방문 시 복원 |
| **File I/O** | DEV REQ JSON (입력), 산정 결과 JSON (출력) | 사용자 직접 제어 |

### 1.3 외부 스키마 준수

입력 및 출력 파일은 `rets_02c_req_schema.json` (v3.0.0)의 `DevRequirement` 스키마를 준수한다.

---

## 2. 데이터 객체 목록

| 데이터 객체 ID | 객체명 | 설명 | 저장 위치 | 연계 요구사항 |
|---------------|--------|------|-----------|---------------|
| DO-01 | Requirement | 개발 요구사항 항목 (RETS 공통 스키마) | Memory + File I/O | DEV_FR_001, DEV_FR_003 |
| DO-02 | ProjectConfig | 프로젝트 기본 정보 설정 | LocalStorage | DEV_FR_002_01 |
| DO-03 | PersonnelConfig | 투입 인력 구성 설정 | LocalStorage | DEV_FR_002_02 |
| DO-04 | PersonnelItem | 개별 인력 항목 (DO-03의 배열 요소) | LocalStorage | DEV_FR_002_02_01 |
| DO-05 | LlmSession | LLM 릴레이 서버 세션 정보 | Memory | DEV_NFR_004_01 |
| DO-06 | LlmStoryPointResult | AI 스토리포인트 추천 결과 (임시) | Memory | DEV_FR_004_01 |
| DO-07 | LlmRoleResult | AI 역할 배정 추천 결과 (임시) | Memory | DEV_FR_005_01 |
| DO-08 | GanttPlan | 간트 차트 계획 데이터 (산출) | Memory | DEV_FR_007 |
| DO-09 | AppState | 전역 애플리케이션 상태 | Memory + LocalStorage | 전반 |
| DO-10 | I18nConfig | 다국어 설정 | LocalStorage | DEV_NFR_002 |

---

## 3. 데이터 객체 상세 정의

### 3.1 DO-01 — Requirement (개발 요구사항)

**설명:** RETS 공통 스키마(`rets_02c_req_schema.json`)의 `DevRequirement`를 기반으로 한 요구사항 데이터 객체. 입력 JSON 파일에서 로드되며, 모듈 내에서 공수 산정 관련 필드(story_points, estimated_effort, developer, predecessor_dev_ids, dev_status)가 추가/수정된다.

**연계 요구사항:** DEV_FR_001, DEV_FR_003, DEV_FR_004, DEV_FR_005, DEV_FR_006

#### 필드 정의

| 필드명 | 타입 | 필수 | 출처 | 설명 |
|--------|------|------|------|------|
| `req_type` | `"DEV"` | ✅ | 입력 | 고정값 "DEV" |
| `req_id` | string | ✅ | 입력 | 패턴: `^DEV_(FR\|NFR)_\d{3}(_\d{2}){0,4}$` |
| `req_version` | string | ✅ | 입력 | 예: `"1.0"` |
| `req_level` | integer (1~5) | ✅ | 입력 | req_id 계층 깊이와 일치 |
| `req_name` | string | ✅ | 입력 | 최대 200자 |
| `functional_class` | `"기능"\|"비기능"` | ✅ | 입력 | FR/NFR 구분 |
| `req_category` | enum | ✅ | 입력 | 11가지 유형 |
| `domain_area` | enum | — | 입력 | 11가지 도메인 |
| `description` | string | ✅ | 입력 | 요구사항 상세 설명 |
| `acceptance_criteria` | string[] | — | 입력 | 완료 기준 목록 |
| `verification_method` | enum | — | 입력 | 검증 방법 |
| `priority` | `"MUST"\|"SHOULD"\|"COULD"\|"WONT"` | — | 입력/편집 | MoSCoW 우선순위 — **모듈 내 편집 가능** |
| `importance` | `"Critical"\|"High"\|"Medium"\|"Low"` | — | 입력/편집 | 중요도 — **모듈 내 편집 가능** |
| `stakeholders` | string[] | — | 입력 | 연관 이해관계자 |
| `created_by` | string | ✅ | 입력 | 발의자 |
| `last_modified_date` | string (date) | ✅ | 입력/출력 | 최종수정일 — 내보내기 시 갱신 |
| `req_status` | enum | ✅ | 입력 | 요구사항 상태 |
| `manager` | string | — | 입력 | 관리 담당자 |
| `reviewer` | string | — | 입력 | 검토자 |
| `change_history` | ChangeHistoryEntry[] | — | 입력 | 변경이력 |
| `io_spec` | IOSpec | — | 입력 | 입력/출력 명세 |
| `story_points` | integer (0~100) | — | **산정 입력** | 스토리포인트 — 수기 또는 AI 추천 후 수락 시 설정 |
| `estimated_effort` | number (MD) | — | **산정 입력** | 예상공수(Man-Day) — 자동 산출 또는 수기 |
| `source_rfp_ids` | string[] | — | 입력 | 근거 RFP 요구사항 ID 목록 |
| `predecessor_dev_ids` | string[] | — | **산정 입력** | 선행 개발요구사항 ID — 의존관계 설정 시 수정 |
| `dev_status` | enum | — | **산정 입력** | 개발 진행 상태 |
| `developer` | string | — | **산정 입력** | 개발 담당자/역할 — AI 추천 또는 수기 |

**제약 조건:**
- req_level과 req_id의 계층 깊이가 반드시 일치해야 한다
- story_points: 0 이상 100 이하의 정수 (피보나치 수열 권장: 1, 2, 3, 5, 8, 13, 21)
- estimated_effort: 0 이상의 실수, 소수점 2자리 이내
- predecessor_dev_ids의 각 항목은 동일 배열 내 다른 req_id를 참조해야 한다 (순환 참조 불가)

**런타임 확장 필드** (내보내기 시 제외되는 UI 상태 필드):

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `_sp_pending` | integer \| null | AI 추천 스토리포인트 (수락 전 임시) |
| `_role_pending` | string \| null | AI 추천 역할 (수락 전 임시) |
| `_sp_reasoning` | string \| null | 스토리포인트 추천 이유 (툴팁용) |
| `_role_reasoning` | string \| null | 역할 추천 이유 (툴팁용) |
| `_highlight` | boolean | 대시보드에서 클릭 시 그리드 강조 표시 여부 |

---

### 3.2 DO-02 — ProjectConfig (프로젝트 기본 정보)

**설명:** 프로젝트의 기본 설정 정보. LocalStorage에 `rets_m03_project_config` 키로 저장된다.

**연계 요구사항:** DEV_FR_002_01, DEV_FR_007_01_01

#### 필드 정의

| 필드명 | 타입 | 필수 | 기본값 | 설명 |
|--------|------|------|--------|------|
| `projectName` | string | — | `""` | 프로젝트 이름 |
| `startDate` | string (date) | ✅ | — | 시작일 (ISO 8601) |
| `endDate` | string (date) | ✅ | — | 종료일 (ISO 8601) |
| `totalWorkDays` | integer | ✅ | 자동 계산 | 전체 근무일 수 (startDate~endDate 기간 내 평일 기준) |
| `totalWeeks` | integer | ✅ | 자동 계산 | 전체 주 수 (ceil(totalWorkDays / 5)) |
| `totalBudgetMD` | number | — | `null` | 총 예산 공수(MD) — 수기 입력 또는 PersonnelConfig 기반 자동 계산 |
| `mdPerSP` | number | ✅ | `0.5` | 스토리포인트당 MD 계수 (조정 가능) |
| `updatedAt` | string (datetime) | ✅ | 자동 | 최종 수정 시각 |

**제약 조건:**
- endDate ≥ startDate (위반 시 오류)
- mdPerSP > 0 (양수)
- totalBudgetMD > 0 (입력 시)

**LocalStorage 키:** `rets_m03_project_config`

---

### 3.3 DO-03 — PersonnelConfig (투입 인력 구성)

**설명:** 프로젝트에 투입되는 인력 구성 전체를 관리하는 컨테이너 객체. PersonnelItem 배열을 포함한다.

**연계 요구사항:** DEV_FR_002_02, DEV_FR_005_02_01

#### 필드 정의

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `items` | PersonnelItem[] | ✅ | 인력 항목 목록 (DO-04) |
| `totalAvailableMD` | number | ✅ | 총 가용 공수 — 자동 계산 (SUM of item.availableMD) |
| `updatedAt` | string (datetime) | ✅ | 최종 수정 시각 |

**총 가용 공수 계산 공식:**
```
totalAvailableMD = Σ (item.count × item.dailyHours / 8 × projectConfig.totalWorkDays)
```

**LocalStorage 키:** `rets_m03_personnel_config`

---

### 3.4 DO-04 — PersonnelItem (개별 인력 항목)

**설명:** DO-03.items 배열의 각 요소. 하나의 역할-등급 조합에 대한 인력 정보를 정의한다.

**연계 요구사항:** DEV_FR_002_02_01, DEV_FR_002_02_02

#### 필드 정의

| 필드명 | 타입 | 필수 | 기본값 | 제약 |
|--------|------|------|--------|------|
| `id` | string (UUID) | ✅ | 자동 생성 | 항목 고유 식별자 |
| `role` | string | ✅ | — | 역할명 (예: "시니어 개발자", "주니어 디자이너") |
| `grade` | `"시니어"\|"미들"\|"주니어"\|"기타"` | ✅ | `"미들"` | 등급 |
| `count` | integer | ✅ | `1` | 인원 수 (1 이상) |
| `dailyHours` | integer | ✅ | `8` | 일일 가용 시간 (1~24) |
| `availableMD` | number | ✅ | 자동 계산 | 이 항목의 가용 MD = count × dailyHours / 8 × totalWorkDays |

---

### 3.5 DO-05 — LlmSession (LLM 세션 정보)

**설명:** LLM 릴레이 서버와의 세션 정보. 페이지 로드 시 1회 생성되어 메모리에 유지된다.

**연계 요구사항:** DEV_NFR_004_01, DEV_NFR_003_03_02

#### 필드 정의

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `sessionId` | string (UUID) | ✅ | 릴레이 서버에서 발급된 세션 ID |
| `teamId` | string | ✅ | 사용자 입력 team_id (1~64자) |
| `createdAt` | string (datetime) | ✅ | 세션 생성 시각 |
| `expiresAt` | string (datetime) | ✅ | 세션 만료 시각 |
| `status` | `"initializing"\|"active"\|"expired"\|"error"` | ✅ | 세션 상태 |
| `inputTokensTotal` | integer | ✅ | 누적 입력 토큰 수 |
| `outputTokensTotal` | integer | ✅ | 누적 출력 토큰 수 |

**세션 초기화 흐름:**
```
DOMContentLoaded
  → POST /chat/new { team_id, system_prompt }
  → 응답: { session_id, created_at, expires_at }
  → LlmSession 객체 생성 및 전역 변수에 저장
  → status = "active"
```

---

### 3.6 DO-06 — LlmStoryPointResult (SP 추천 결과)

**설명:** AI 스토리포인트 추천 응답의 파싱 결과. 수락/거부 전 임시 상태로 메모리에 유지된다.

**연계 요구사항:** DEV_FR_004_01_01, DEV_FR_004_01_02, DEV_FR_004_01_03

#### 필드 정의 (배열 요소)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `req_id` | string | ✅ | 대상 요구사항 ID |
| `recommended_sp` | integer | ✅ | AI 추천 스토리포인트 (0~100) |
| `reasoning` | string | ✅ | 추천 근거 (한국어 텍스트) |
| `accepted` | boolean \| null | ✅ | 수락(true) / 거부(false) / 미결(null) |

**LLM 요청 JSON 출력 스키마 (user prompt에 포함):**
```json
{
  "results": [
    {
      "req_id": "<string>",
      "recommended_sp": "<integer: 1|2|3|5|8|13|21>",
      "reasoning": "<string: 추천 근거 설명>"
    }
  ]
}
```

---

### 3.7 DO-07 — LlmRoleResult (역할 배정 추천 결과)

**설명:** AI 역할 배정 추천 응답의 파싱 결과. 수락/거부 전 임시 상태로 메모리에 유지된다.

**연계 요구사항:** DEV_FR_005_01_01, DEV_FR_005_01_02

#### 필드 정의 (배열 요소)

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `req_id` | string | ✅ | 대상 요구사항 ID |
| `assigned_role` | string | ✅ | 추천 담당 역할명 (PersonnelItem.role과 일치 권장) |
| `reasoning` | string | ✅ | 배정 근거 |
| `accepted` | boolean \| null | ✅ | 수락/거부/미결 |

**LLM 요청 JSON 출력 스키마:**
```json
{
  "results": [
    {
      "req_id": "<string>",
      "assigned_role": "<string>",
      "reasoning": "<string>"
    }
  ]
}
```

---

### 3.8 DO-08 — GanttPlan (간트 차트 계획)

**설명:** 간트 차트 렌더링을 위해 산출되는 계획 데이터. Requirement 배열, ProjectConfig, PersonnelConfig로부터 파생 계산된다.

**연계 요구사항:** DEV_FR_007

#### 필드 정의

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `tasks` | GanttTask[] | ✅ | 간트 차트 작업 목록 |
| `projectStartDate` | string (date) | ✅ | 프로젝트 시작일 |
| `projectEndDate` | string (date) | ✅ | 프로젝트 종료일 |
| `totalEstimatedMD` | number | ✅ | 전체 예상 공수 합산 |
| `isOverDeadline` | boolean | ✅ | 일정 초과 여부 |
| `overflowWeeks` | number | — | 초과 주 수 (isOverDeadline=true인 경우) |
| `mermaidSyntax` | string | ✅ | Mermaid Gantt 문법 문자열 |

#### GanttTask 필드 정의

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `req_id` | string | ✅ | 연계 요구사항 ID |
| `req_name` | string | ✅ | 요구사항명 (간트 표시용) |
| `developer` | string | — | 담당 역할 |
| `estimatedMD` | number | ✅ | 예상 공수(MD) |
| `startDate` | string (date) | ✅ | 계산된 시작일 |
| `endDate` | string (date) | ✅ | 계산된 종료일 |
| `predecessors` | string[] | — | 선행 req_id 목록 |
| `priority` | string | — | MoSCoW 우선순위 |
| `topologicalOrder` | integer | ✅ | 위상 정렬 순서 |

**간트 계획 산출 알고리즘:**
1. Requirement 배열에서 estimated_effort가 설정된 항목을 추출
2. predecessor_dev_ids 기반 위상 정렬(Kahn's Algorithm) 수행
3. MUST 항목 우선 배치
4. 각 항목의 startDate = MAX(projectStartDate, predecessor 최대 endDate + 1일)
5. endDate = startDate + ceil(estimatedMD) 근무일
6. Mermaid Gantt 문법 문자열 생성

**Mermaid Gantt 문법 생성 예시:**
```
gantt
  title RETS-MODULE-03 프로젝트 간트 차트
  dateFormat YYYY-MM-DD
  axisFormat %m/%d
  section MUST
    DEV_FR_001_01 데이터 입출력 :a1, 2026-05-07, 2d
    DEV_FR_002 프로젝트 설정  :a2, after a1, 3d
  section SHOULD
    DEV_FR_007 일정 계획 :a3, after a2, 5d
```

---

### 3.9 DO-09 — AppState (전역 애플리케이션 상태)

**설명:** 모듈의 전역 런타임 상태를 관리하는 객체. 메모리 내 싱글턴.

**연계 요구사항:** 전반

#### 필드 정의

| 필드명 | 타입 | 필수 | 설명 |
|--------|------|------|------|
| `requirements` | Requirement[] | ✅ | 현재 로드된 요구사항 목록 |
| `projectConfig` | ProjectConfig | — | 프로젝트 설정 |
| `personnelConfig` | PersonnelConfig | — | 인력 구성 |
| `llmSession` | LlmSession | — | LLM 세션 정보 |
| `spResults` | LlmStoryPointResult[] | — | 현재 SP 추천 결과 (임시) |
| `roleResults` | LlmRoleResult[] | — | 현재 역할 배정 추천 결과 (임시) |
| `ganttPlan` | GanttPlan | — | 간트 계획 (파생) |
| `activeTab` | string | ✅ | 현재 활성 탭 ID |
| `theme` | `"light"\|"dark"` | ✅ | 현재 테마 |
| `language` | `"ko"\|"en"` | ✅ | 현재 언어 |
| `isLoading` | boolean | ✅ | 전역 로딩 상태 |
| `loadingMessage` | string | — | 로딩 메시지 |

---

### 3.10 DO-10 — I18nConfig (다국어 설정)

**설명:** 한국어/영어 UI 텍스트 키-값 쌍을 관리하는 다국어 설정 객체.

**연계 요구사항:** DEV_NFR_002

#### 구조

```javascript
const i18n = {
  ko: {
    // 헤더
    "header.moduleTitle": "요구사항 기반 규모 산정 및 투입 계획",
    "header.btn.load": "불러오기",
    "header.btn.export": "내보내기",
    "header.btn.settings": "설정",
    // 탭
    "tab.dashboard": "대시보드",
    "tab.requirements": "요구사항 산정",
    "tab.gantt": "일정 계획",
    // 그리드
    "grid.col.reqId": "요구사항 ID",
    "grid.col.reqName": "요구사항명",
    "grid.col.priority": "우선순위",
    "grid.col.importance": "중요도",
    "grid.col.storyPoints": "스토리포인트",
    "grid.col.effort": "예상공수(MD)",
    "grid.col.developer": "담당 역할",
    "grid.col.devStatus": "개발 상태",
    // 버튼
    "btn.aiSP": "AI 스토리포인트 추천",
    "btn.aiRole": "AI 역할 배정",
    "btn.acceptAll": "전체 수락",
    "btn.rejectAll": "전체 거부",
    // 토스트
    "toast.loadSuccess": "{{count}}개의 요구사항을 불러왔습니다.",
    "toast.exportSuccess": "산정 결과를 내보냈습니다.",
    "toast.llmError": "LLM 서버 연결에 실패했습니다.",
    "toast.parseError": "AI 응답 파싱에 실패했습니다.",
    // 대시보드
    "dashboard.totalReqs": "전체 요구사항",
    "dashboard.spMissing": "SP 미배정",
    "dashboard.effortMissing": "공수 미산출",
    "dashboard.effortRatio": "공수 활용률",
    // ... (전체 키 정의)
  },
  en: {
    "header.moduleTitle": "Requirements-Based Sizing & Resource Planning",
    "header.btn.load": "Load",
    "header.btn.export": "Export",
    "header.btn.settings": "Settings",
    "tab.dashboard": "Dashboard",
    "tab.requirements": "Requirement Sizing",
    "tab.gantt": "Schedule",
    "grid.col.reqId": "Req ID",
    "grid.col.reqName": "Requirement Name",
    "grid.col.priority": "Priority",
    "grid.col.importance": "Importance",
    "grid.col.storyPoints": "Story Points",
    "grid.col.effort": "Est. Effort (MD)",
    "grid.col.developer": "Assigned Role",
    "grid.col.devStatus": "Dev Status",
    "btn.aiSP": "AI SP Recommendation",
    "btn.aiRole": "AI Role Assignment",
    "btn.acceptAll": "Accept All",
    "btn.rejectAll": "Reject All",
    "toast.loadSuccess": "{{count}} requirements loaded.",
    "toast.exportSuccess": "Sizing results exported.",
    "toast.llmError": "Failed to connect to LLM server.",
    "toast.parseError": "Failed to parse AI response.",
    "dashboard.totalReqs": "Total Requirements",
    "dashboard.spMissing": "SP Not Assigned",
    "dashboard.effortMissing": "Effort Not Estimated",
    "dashboard.effortRatio": "Effort Utilization",
    // ...
  }
};
```

**LocalStorage 키:** `rets_m03_language`

---

## 4. 데이터 흐름 및 관계 다이어그램

### 4.1 데이터 입출력 흐름

```
[외부 JSON 파일]
       │
       ▼  불러오기 (File I/O)
[DO-01 Requirement[] ]─────────────────┐
       │                               │
       │ 파생                          │ 파생
       ▼                               ▼
[DO-06 LlmStoryPointResult]    [DO-08 GanttPlan]
[DO-07 LlmRoleResult     ]            │
       │                               │
       │ 수락 반영                      │ Mermaid 렌더링
       ▼                               ▼
[DO-01 Requirement[] (갱신)]   [간트 차트 화면]
       │
       ▼  내보내기 (File I/O)
[산정 결과 JSON 파일]

[DO-02 ProjectConfig ] ─┐
[DO-03 PersonnelConfig]─┴─▶ [DO-08 GanttPlan 산출]

[DO-05 LlmSession] ─▶ LLM 릴레이 서버 연동
```

### 4.2 데이터 객체 관계

```
AppState (DO-09)
 ├── requirements: Requirement[] (DO-01)  ←→  LlmStoryPointResult[] (DO-06)
 │                                        ←→  LlmRoleResult[]       (DO-07)
 │                                        ──▶ GanttPlan             (DO-08)
 ├── projectConfig: ProjectConfig (DO-02)  ──▶ GanttPlan (DO-08)
 ├── personnelConfig: PersonnelConfig (DO-03)
 │    └── items: PersonnelItem[] (DO-04)
 ├── llmSession: LlmSession (DO-05)
 └── [i18nConfig: I18nConfig (DO-10)]  ← 별도 전역 관리
```

---

## 5. 입출력 데이터 명세

### 5.1 입력 데이터 — DEV REQ JSON

| 항목 | 내용 |
|------|------|
| 파일 형식 | JSON Array (`DevRequirement[]`) |
| 스키마 | `rets_02c_req_schema.json` — DevRequirement |
| 벤치마크 파일 | `rets_benchmark_02_dev_req.json` |
| 최소 항목 수 | 1개 이상 |
| 최대 항목 수 | 300개 이하 (성능 기준) |
| 인코딩 | UTF-8 |

**입력 유효성 검증 규칙:**
1. JSON 파싱 성공 여부 확인
2. 배열 형식 확인
3. 각 항목의 `req_type === "DEV"` 확인
4. 필수 필드(`req_id`, `req_name`, `functional_class`, `description`, `req_status`, `created_by`, `last_modified_date`) 존재 확인
5. req_id 패턴 검증: `^DEV_(FR|NFR)_\d{3}(_\d{2}){0,4}$`
6. req_level과 req_id 계층 깊이 일치 확인

### 5.2 출력 데이터 — 산정 결과 JSON

| 항목 | 내용 |
|------|------|
| 파일명 | `rets_module_03_sizing_result_{YYYYMMDD}.json` |
| 파일 형식 | JSON Array (`DevRequirement[]`) |
| 스키마 | `rets_02c_req_schema.json` — DevRequirement |
| 추가 필드 | `story_points`, `estimated_effort`, `predecessor_dev_ids`, `developer`, `dev_status` |
| 인코딩 | UTF-8 |
| 들여쓰기 | 2 spaces |

**출력 시 처리 규칙:**
1. `_sp_pending`, `_role_pending`, `_sp_reasoning`, `_role_reasoning`, `_highlight` 등 런타임 전용 필드 제거
2. `last_modified_date`를 내보내기 시점의 날짜로 갱신
3. req_type = "DEV"인 항목만 포함
4. req_id 기준 정렬 (오름차순)

---

## 6. LocalStorage 데이터 관리

| 키 | 데이터 객체 | 설명 |
|----|-------------|------|
| `rets_m03_project_config` | DO-02 ProjectConfig | JSON 직렬화 저장 |
| `rets_m03_personnel_config` | DO-03 PersonnelConfig | JSON 직렬화 저장 |
| `rets_m03_language` | string (`"ko"` \| `"en"`) | 선택 언어 |
| `rets_m03_theme` | string (`"light"` \| `"dark"`) | 선택 테마 |
| `rets_m03_team_id` | string | 사용자 입력 team_id |

**주의 사항:**
- 요구사항 배열(DO-01)은 LocalStorage에 저장하지 않는다 (데이터 크기 및 민감성 고려)
- LLM 추천 임시 결과(DO-06, DO-07)는 LocalStorage에 저장하지 않는다
- 간트 계획(DO-08)은 항상 런타임에 재계산한다

---

## 7. LLM Response JSON 설계

### 7.1 스토리포인트 추천 요청/응답 스키마

**요청 (user prompt 포함):**
```json
{
  "requirements": [
    {
      "req_id": "DEV_FR_001_01_01",
      "req_name": "JSON 파일 선택 및 파싱",
      "description": "브라우저 FileReader API를 통해 JSON 파일을 읽고...",
      "importance": "Critical",
      "priority": "MUST"
    }
  ]
}

## Output Format (STRICT JSON ONLY)
반드시 아래 JSON 스키마만 출력하라. 설명·마크다운·코드 펜스는 포함하지 말 것.
{
  "results": [
    {
      "req_id": "<string>",
      "recommended_sp": <integer: 1|2|3|5|8|13|21>,
      "reasoning": "<string: 추천 근거>"
    }
  ]
}
```

**응답 예시:**
```json
{
  "results": [
    {
      "req_id": "DEV_FR_001_01_01",
      "recommended_sp": 2,
      "reasoning": "FileReader API 사용은 표준적이며 구현 복잡도가 낮습니다."
    }
  ]
}
```

### 7.2 역할 배정 추천 요청/응답 스키마

**요청 (user prompt 포함):**
```json
{
  "requirements": [
    {
      "req_id": "DEV_FR_001_01_01",
      "req_name": "JSON 파일 선택 및 파싱",
      "story_points": 2,
      "importance": "Critical"
    }
  ],
  "personnel": [
    { "role": "시니어 개발자", "grade": "시니어", "count": 3 },
    { "role": "주니어 개발자", "grade": "주니어", "count": 2 }
  ]
}

## Output Format (STRICT JSON ONLY)
{
  "results": [
    {
      "req_id": "<string>",
      "assigned_role": "<string>",
      "reasoning": "<string>"
    }
  ]
}
```

---

## 8. 요구사항 추적 매트릭스

| 데이터 객체 | 연계 SRS 요구사항 |
|-------------|------------------|
| DO-01 Requirement | DEV_FR_001, DEV_FR_003_01, DEV_FR_004_01_03, DEV_FR_005_01_02, DEV_FR_005_02_01, DEV_FR_006_01 |
| DO-02 ProjectConfig | DEV_FR_002_01, DEV_FR_005_02_01, DEV_FR_007_01_01 |
| DO-03 PersonnelConfig | DEV_FR_002_02, DEV_FR_005_02_01 |
| DO-04 PersonnelItem | DEV_FR_002_02_01, DEV_FR_002_02_02, DEV_FR_002_02_03 |
| DO-05 LlmSession | DEV_NFR_003_03_02, DEV_NFR_004_01_01 |
| DO-06 LlmStoryPointResult | DEV_FR_004_01_01, DEV_FR_004_01_02, DEV_FR_004_01_03 |
| DO-07 LlmRoleResult | DEV_FR_005_01_01, DEV_FR_005_01_02 |
| DO-08 GanttPlan | DEV_FR_007_01_01, DEV_FR_007_01_02, DEV_FR_007_01_03 |
| DO-09 AppState | DEV_NFR_001_01_01, DEV_NFR_002_01, 전반 |
| DO-10 I18nConfig | DEV_NFR_002_01, DEV_NFR_002_01_01 |

---

*본 문서는 RETS-MODULE-03 개발팀이 작성하였으며, 스키마 파일(`rets_module_03_05_ddd_schema.json`)과 함께 활용한다. SRS(MD03)의 `rets_module_03_03_srs.xlsx` > 추적성_매트릭스 시트의 DDD_data_objects 컬럼을 함께 업데이트한다.*
