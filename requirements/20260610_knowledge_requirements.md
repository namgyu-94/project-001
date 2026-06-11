# Knowledge 기능 요건 정의

**참조 프로토타입**
- `docs/ui-requirements_case1.html` — Case 1: Agent 등록 시 DAG 사전 지정
- `docs/ui-requirements_case2.html` — Case 2: 문서 승인 시 DAG 직접 입력

---

## 개요

Developer가 agent 등록 시 Knowledge 기능(end-user 문서 업로드 → VectorDB 지식화) 사용 여부를 결정하며, 이에 따라 문서 업로드 파이프라인이 활성화된다.

두 케이스의 차이는 **Airflow DAG ID를 언제, 누가 지정하느냐**에 있다.

---

## 공통 결정 사항

### S3 아키텍처: 플랫폼 S3 관리 + DAG Pull 방식

End-user 업로드 문서를 플랫폼이 소유한 S3에 저장하고, DAG이 플랫폼 API를 통해 S3 경로를 조회(Pull)하여 처리한다. (두 케이스 공통 적용)

```
DAG Trigger 시 doc_id 수신
  → GET /api/documents/{doc_id}/path    # S3 경로 조회
  → (DAG이 S3에서 파일 로드 후 임베딩 처리)
  → POST /api/documents/{doc_id}/status  # 처리 결과 업데이트 (completed / failed)
```

- S3 경로 패턴: `platform-s3/agents/{agent_id}/users/{user_id}/{filename}`
- 반려 시 플랫폼이 S3 파일 즉시 삭제

### 문서 처리 파이프라인 (5단계, 공통)

```
① S3 저장 → ② 승인 대기 → ③ DAG 실행 → ④ 임베딩 → ⑤ VectorDB 등록
```

### Knowledge 기능 활성화 위치 (공통)

Agent 등록 신청 **Step 1 기본 정보 하단**에 Knowledge 기능 토글 존재.  
두 케이스 모두 이 위치에서 ON/OFF를 결정하며, 이후 DAG 처리 방식에서 갈린다.

---

## Case 1 — Agent 등록 시 DAG 사전 지정

> 프로토타입: `docs/ui-requirements_case1.html`

### 핵심 개념

Developer가 **Agent 등록 신청 시** Knowledge 토글 ON + Airflow DAG ID를 함께 입력한다.  
이후 사용자가 문서를 업로드하면 해당 문서의 승인 시 사전 등록된 DAG이 자동으로 Trigger된다.

### 프로세스

```
[Developer] Agent 등록 신청 Step 1
  → Knowledge 토글 ON
  → Airflow DAG ID 입력 (예: legal_doc_embedding_dag)
  → Airflow Host URL 입력
  → DAG 템플릿 코드 다운로드 (인터페이스 규격 확인)
  → 신청 제출

[Admin] 승인

[Developer] Validation 통과 → Agent 서비스 중

[User] MY > 문서 업로드
  → 구독 중인 Agent 선택
  → 처리 DAG 자동 표시 (Agent에 사전 등록된 값)
  → 파일 업로드 + 임베딩 파라미터 설정
  → 업로드 요청

[Developer] Developer > 문서 승인
  → 테이블에 DAG 컬럼이 읽기 전용으로 표시 (사전 등록된 DAG)
  → 파일 내용 검토
  → 승인 클릭 → 플랫폼이 사전 등록된 DAG 자동 Trigger
  → (반려 시 S3 파일 즉시 삭제)

[Airflow DAG] GET /api/documents/{doc_id}/path 로 S3 경로 조회
  → 임베딩 처리
  → POST /api/documents/{doc_id}/status (completed / failed)

[User] MY > 내 문서 현황에서 처리 완료 확인
```

### 화면별 UI 차이점

| 화면 | Case 1 동작 |
|------|------------|
| 등록 신청 Step 1 | Knowledge 토글 ON → DAG ID 입력 필드 노출 (필수) |
| 등록 신청 Step 3 확인 | Knowledge: 🧠 활성 + DAG ID 표시 |
| MY > 문서 업로드 | 처리 DAG 필드 자동 채워짐 (Agent 선택 시) |
| Developer > 문서 승인 | DAG 컬럼 = 사전 등록된 DAG명 (읽기 전용), 승인 버튼만 클릭 |

### 장단점

| 구분 | 내용 |
|------|------|
| 장점 | DAG 설정이 Agent 등록 시 완결 — 승인 시 별도 입력 불필요 |
| 장점 | 동일 Agent의 모든 문서가 동일 DAG으로 처리 → 일관성 |
| 장점 | Developer 승인 UX 단순 (클릭 1회) |
| 단점 | Agent 등록 후 DAG 변경 시 재등록 또는 별도 수정 절차 필요 |
| 단점 | 문서 유형별로 다른 DAG을 적용하려면 복수 Agent 필요 |

---

## Case 2 — 문서 승인 시 DAG 직접 입력

> 프로토타입: `docs/ui-requirements_case2.html`

### 핵심 개념

Developer가 **Agent 등록 신청 시** Knowledge 토글 ON만 하고 DAG ID는 입력하지 않는다.  
문서 업로드 후 Developer가 **문서 승인 시점에** DAG ID를 직접 입력하고 승인한다.

### 프로세스

```
[Developer] Agent 등록 신청 Step 1
  → Knowledge 토글 ON
  → DAG ID 입력 없음 (승인 시 결정 예정)
  → 신청 제출

[Admin] 승인

[Developer] Validation 통과 → Agent 서비스 중

[User] MY > 문서 업로드
  → 구독 중인 Agent 선택
  → 처리 DAG 필드: "개발자 승인 시 DAG ID 입력" 안내 표시 (비어 있음)
  → 파일 업로드 + 임베딩 파라미터 설정
  → 업로드 요청

[Developer] Developer > 문서 승인
  → 테이블에 Airflow DAG ID 인라인 입력 필드 존재 (★ 승인 시 입력)
  → 파일 내용 검토 + DAG ID 직접 입력
  → 승인 클릭 → 플랫폼이 입력된 DAG Trigger
  → (반려 시 S3 파일 즉시 삭제)

[Airflow DAG] GET /api/documents/{doc_id}/path 로 S3 경로 조회
  → 임베딩 처리
  → POST /api/documents/{doc_id}/status (completed / failed)

[User] MY > 내 문서 현황에서 처리 완료 확인
```

### 화면별 UI 차이점

| 화면 | Case 2 동작 |
|------|------------|
| 등록 신청 Step 1 | Knowledge 토글 ON → DAG ID 입력 없음, "승인 시 결정" 안내만 표시 |
| 등록 신청 Step 3 확인 | Knowledge: 🧠 활성 (DAG: 승인 시 입력) |
| MY > 문서 업로드 | 처리 DAG 필드 비어 있음 + "개발자 승인 시 입력" 안내 표시 |
| Developer > 문서 승인 | DAG 컬럼 = 인라인 입력 필드 (각 행마다 DAG ID 입력 후 승인) |

### 장단점

| 구분 | 내용 |
|------|------|
| 장점 | 문서별로 다른 DAG 적용 가능 → 유연성 |
| 장점 | Agent 등록 시 DAG이 미결정인 상태에서도 Knowledge 활성화 가능 |
| 단점 | 매 문서 승인마다 DAG ID 입력 필요 → 승인 UX 복잡 |
| 단점 | 입력 누락·오입력 가능성 (운영 오류 리스크) |
| 단점 | 동일 Agent 내 문서별 다른 DAG 적용 시 관리 추적 어려움 |

---

## Case 비교 요약

| 항목 | Case 1 (사전 지정) | Case 2 (승인 시 입력) |
|------|------------------|---------------------|
| DAG 지정 시점 | Agent 등록 신청 시 | 문서 승인 시 |
| DAG 지정 주체 | Developer (등록 시) | Developer (승인 시) |
| 문서 업로드 패널 | DAG 자동 표시 | DAG 비어 있음 (안내만) |
| 문서 승인 패널 | DAG 읽기 전용 표시 → 버튼 클릭만 | DAG 인라인 입력 → 입력 후 클릭 |
| 일관성 | 높음 (Agent당 1 DAG) | 낮음 (문서별 상이 가능) |
| 유연성 | 낮음 | 높음 |
| 운영 오류 리스크 | 낮음 | 중간 (입력 누락 가능) |
| 권장 상황 | 대부분의 경우, 초기 구축 시 | DAG을 문서 유형별로 달리해야 하는 경우 |

---

## 화면별 Knowledge 기능 정리 (공통)

| 화면 | 역할 | Knowledge 관련 기능 |
|------|------|-------------------|
| Developer > 등록 신청 Step 1 | Developer | Knowledge 사용 여부 토글 / (Case 1) DAG ID·Host URL 입력 |
| Developer > 내 현황 (목록) | Developer | Knowledge 상태 칩 표시 (🧠 활성 / 미설정) |
| Developer > 문서 승인 | Developer | (Case 1) DAG 읽기 전용 + 승인/반려 / (Case 2) DAG 인라인 입력 + 승인/반려 |
| MY > 내 현황 | User | 구독 Agent별 업로드 문서 수 및 5단계 파이프라인 미니 표시 |
| MY > 문서 업로드 | User | Agent·Collection 선택, 파일 업로드, 임베딩 파라미터 설정 |
| MY > 내 문서 현황 | User | 업로드 문서 목록 및 5단계 처리 상태 확인 |
| Admin > Milvus 컬렉션 | Admin | Knowledge 활성 여부(🧠), 등록 문서 수, 벡터 수 확인 |

---

## Agent 등록 신청 Wizard 구성

| Step | 제목 | Knowledge 관련 입력 |
|------|------|-------------------|
| 1 | 기본 정보 | 🧠 Knowledge 기능 토글 / (Case 1만) DAG ID·Host URL·DAG 템플릿 다운로드 |
| 2 | Skills / Tags | 해당 없음 |
| 3 | 확인 절차 | Knowledge 활성 여부 + (Case 1) DAG ID 요약 표시 |
| 4 | 신청 완료 | 등록 대기 → 관리자 승인 → Kong Key 발급 |
