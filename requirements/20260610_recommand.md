# Knowledge 기능 요건 케이스 정리

> 작성일: 2026-06-10  
> 대상: agentMarket UI 프로토타입 (`docs/ui-requirements.html`) 기준  
> 목적: Knowledge 기능 도입 시 UI 화면 구성 및 S3 아키텍처 결정을 위한 케이스별 옵션 제시

---

## 의사결정 필요 항목 요약

| # | 결정 항목 | 케이스 수 |
|---|----------|---------|
| 1 | Developer의 Knowledge 기능 활성화 설정 위치 (UI 화면) | 3개 |
| 2 | 문서 저장소 및 파이프라인 아키텍처 | 2개 |
| 3 | DAG 인터페이스 방식 | 2개 |
| 4 | 문서 승인/반려 후 처리 흐름 | 2개 |

---

## 1. Developer의 Knowledge 기능 활성화 설정 위치

Developer가 해당 Agent에 Knowledge 기능을 사용할지 여부를 어느 화면에서 선택하는가.

---

### Case 1-A: Agent 등록 신청(apply) 폼에서만 설정

- **화면**: `apply` 패널 (Developer > Agent 관리 > 등록 신청)
- **흐름**: 등록 신청 폼에 "Knowledge 기능 사용 여부" 토글 추가 → 활성화 시 DAG ID 입력 필드 노출
- **영향 범위**: `apply` 패널에만 UI 변경 필요
- **장점**
  - 등록 시점에 일괄 설계 가능, 화면 추가 최소화
  - Admin 승인 전 Knowledge 설계 포함 가능
- **단점**
  - 등록 완료 후 Knowledge 사용 여부 변경 시 재신청(수정 신청) 필요
  - 운영 중 DAG 교체 시 Agent 상태 변경 불가피

---

### Case 1-B: Developer > Agent 관리(developer 패널) 상세/수정 화면에서 설정

- **화면**: `developer` 패널 (Developer > Agent 관리) 내 Agent 상세 또는 수정 모달
- **흐름**: 서비스 중 / Validation 대기 등 상태와 관계없이 Knowledge 탭에서 별도 설정
- **영향 범위**: `developer` 패널에 Knowledge 설정 섹션 추가
- **장점**
  - 등록 이후에도 언제든 활성화/비활성화 가능
  - 기존 Agent에 Knowledge 기능 추가 도입 가능
- **단점**
  - Admin 승인 전 설계 범위에서 누락 가능
  - 화면 복잡도 증가

---

### Case 1-C: 등록 신청(apply)에서 초기 선택 + Agent 관리(developer 패널)에서 수정 가능 ✅ 권장

- **화면**: `apply` 패널에서 초기 설정 + `developer` 패널에서 수정
- **흐름**
  1. 등록 신청 시 Knowledge 사용 여부 선택 (DAG ID 포함)
  2. 이후 `developer` 패널 > Agent 상세에서 Knowledge 설정 수정 가능 (단, 수정 신청 프로세스 불필요 — Knowledge 설정은 Admin 승인 외 항목으로 처리)
- **장점**
  - 초기 설계 완성도 확보 + 운영 중 유연성 보장
  - DAG 교체 시 Agent 재신청 없이 처리 가능
- **단점**
  - 두 화면에서 Knowledge 설정 관리 필요 → 상태 동기화 로직 필요

---

## 2. 문서 저장소 및 파이프라인 아키텍처

End-user가 업로드한 문서를 어디에 보관하고 누가 관리하는가.

---

### Case 2-A: Developer 자체 S3 사용

- **구조**: Developer가 Agent 등록 시 자신의 S3 버킷 정보(Bucket명, 경로, 접근 Key)를 플랫폼에 등록 → End-user 업로드 시 플랫폼이 Developer S3로 직접 전송
- **흐름**
  ```
  End-user 업로드 (doc-upload 패널)
    → 플랫폼 → Developer S3 버킷 전송
    → Developer doc-approve 패널에서 승인
    → 승인: Developer S3 파일 유지 → DAG 실행
    → 반려: Developer S3 파일 삭제
  ```
- **플랫폼 제공 요소**: S3 경로 등록 폼 (apply 또는 developer 패널)
- **장점**
  - 플랫폼 스토리지 비용 없음
  - Developer 인프라 내 데이터 통제 가능
- **단점**
  - Developer마다 S3 구성 필요 → 진입 장벽 높음
  - 플랫폼이 외부 S3 접근 권한 관리 필요 → 보안 복잡도 증가
  - Developer S3 장애 시 플랫폼 책임 범위 불명확

---

### Case 2-B: 플랫폼 S3 통합 관리 + 백엔드 API 인터페이스 ✅ 권장

- **구조**: 플랫폼이 S3 제공 및 관리 → End-user 업로드 파일을 플랫폼 S3에 저장 → Developer의 Airflow DAG이 플랫폼 백엔드 API를 호출하여 파일 경로 수신 후 처리
- **흐름**
  ```
  End-user 업로드 (doc-upload 패널)
    → 플랫폼 S3 임시 보관 (agent_id / user_id / 파일명)
    → Developer doc-approve 패널에서 승인/반려
    → 승인: 플랫폼 백엔드가 DAG Trigger → DAG이 API 호출로 S3 경로 수신 → 문서 처리 → VectorDB(Milvus) 적재
    → 반려: 플랫폼 S3에서 파일 삭제
  ```
- **플랫폼 제공 요소**
  - DAG 인터페이스 템플릿 코드 (Developer에게 제공)
  - 백엔드 API: `GET /api/documents/{doc_id}/path` → S3 경로 반환
  - 백엔드 API: `POST /api/documents/{doc_id}/status` → 처리 상태 업데이트 (성공/실패)
- **장점**
  - 플랫폼이 스토리지 일원화 관리 → 보안 일관성
  - Developer는 S3 인프라 구성 불필요
  - 승인/반려 후 파일 생명주기를 플랫폼이 통제
  - DAG 인터페이스 표준화로 개발 가이드 제공 가능
- **단점**
  - 플랫폼 S3 비용 발생
  - 백엔드 API 및 DAG 템플릿 설계 선행 필요

---

## 3. DAG 인터페이스 방식

Developer가 Airflow DAG을 작성할 때 플랫폼과 어떻게 연결하는가. (Case 2-B 선택 시 해당)

---

### Case 3-A: DAG에서 플랫폼 API 호출 방식 (Pull 방식)

- **방식**: DAG Trigger 시 `doc_id`를 파라미터로 수신 → DAG 내부에서 플랫폼 API 호출 → S3 경로 획득 → 문서 처리
- **DAG 템플릿 핵심 코드 구조**
  ```python
  # 플랫폼 제공 템플릿
  def get_document_path(doc_id: str, jwt_token: str) -> str:
      resp = requests.get(f"{PLATFORM_API}/documents/{doc_id}/path",
                          headers={"Authorization": f"Bearer {jwt_token}"})
      return resp.json()["s3_path"]

  def update_status(doc_id: str, status: str, jwt_token: str):
      requests.post(f"{PLATFORM_API}/documents/{doc_id}/status",
                    json={"status": status},
                    headers={"Authorization": f"Bearer {jwt_token}"})
  ```
- **DAG 입력 파라미터**: `{ "doc_id": "...", "jwt_token": "..." }`
- **장점**: DAG 트리거 단순, 표준 REST 인터페이스
- **단점**: DAG에서 플랫폼 API 호출 의존성 발생, JWT 토큰 관리 필요

---

### Case 3-B: 플랫폼이 S3 경로를 DAG Trigger 파라미터로 직접 전달 (Push 방식)

- **방식**: 플랫폼 백엔드가 DAG Trigger 시 S3 경로·파일명을 conf로 직접 전달
- **DAG 입력 파라미터**
  ```json
  {
    "doc_id": "...",
    "s3_bucket": "platform-docs",
    "s3_key": "agents/{agent_id}/users/{user_id}/{filename}",
    "callback_url": "https://platform-api/documents/{doc_id}/status"
  }
  ```
- **장점**: DAG이 플랫폼 API에 재호출 불필요, 간단한 DAG 구조
- **단점**: Trigger 시 경로 노출, callback_url 처리 로직 DAG에 포함 필요

---

## 4. 문서 승인/반려 후 처리 흐름 (UI 화면 관점)

Developer의 `doc-approve` 패널에서 승인/반려 후 각 역할 화면에 어떻게 반영되는가.

---

### Case 4-A: 승인/반려 후 상태만 표시 (현행 구조 유지·확장)

- **현행 패널**: `doc-approve` (Developer), `doc-status` (User·Dev)
- **흐름**
  - 승인 → `doc-status`에서 상태: `VectorDB 등록 중` → `등록 완료`
  - 반려 → `doc-status`에서 상태: `반려` + 반려 사유 표시
- **추가 UI 요소 최소화**: 기존 패널에 상태 값만 추가

---

### Case 4-B: 승인 후 VectorDB 처리 단계별 진행 상태 표시 ✅ 권장

- **흐름**
  ```
  승인
    → 상태: DAG 실행 중
    → 상태: 임베딩 처리 중
    → 상태: VectorDB 등록 완료 (Milvus 컬렉션명 표시)
  반려
    → 상태: 반려 (반려 사유 노출)
    → 파일 삭제 처리
  ```
- **`doc-status` 패널 표시 항목 추가**

  | 컬럼 | 설명 |
  |------|------|
  | 파일명 | 업로드한 원본 파일명 |
  | 업로드 일시 | |
  | 처리 상태 | 승인 대기 / DAG 실행 중 / 임베딩 처리 중 / 등록 완료 / 반려 |
  | Milvus 컬렉션 | 등록 완료 시 컬렉션명 표시 |
  | 반려 사유 | 반려 시 노출 |

- **장점**: End-user가 문서 처리 진행 상황을 투명하게 확인 가능
- **단점**: DAG → 플랫폼 백엔드 상태 콜백 연동 구현 필요

---

## 5. Knowledge 기능 도입 시 화면별 변경 요약

| 패널 ID | 역할 | 변경 내용 |
|---------|------|---------|
| `apply` | Developer | Knowledge 사용 여부 토글 + DAG ID 입력 필드 추가 |
| `developer` | Developer | Agent 상세에 Knowledge 설정 섹션 추가 (Case 1-C 선택 시) |
| `doc-approve` | Developer | 기존 패널 유지 + Agent별 문서 탭 필터 + 승인/반려 시 DAG Trigger 연동 |
| `doc-upload` | User·Dev | 기존 패널 유지 + Knowledge 미활성 Agent는 업로드 버튼 비활성화 처리 |
| `doc-status` | User·Dev | 처리 단계별 상태 컬럼 추가 (Case 4-B 선택 시) |
| `milvus` | Admin | 컬렉션 목록에 연결된 Agent명 / 등록 문서 수 컬럼 추가 |

---

## 6. 권장 조합 요약

| 항목 | 권장 케이스 | 이유 |
|------|-----------|------|
| Knowledge 설정 위치 | **Case 1-C** | 등록 시 설계 완성도 + 운영 유연성 확보 |
| 저장소 아키텍처 | **Case 2-B** | 플랫폼 통합 관리로 보안·운영 일관성 확보 |
| DAG 인터페이스 | **Case 3-A** (Pull) | 표준 REST, 플랫폼 API 중심 설계와 일관성 |
| 처리 상태 표시 | **Case 4-B** | End-user 투명성 확보, 문서 신뢰도 향상 |

---

## 7. 미결 결정 사항 (고객 선택 필요)

| # | 결정 항목 | 옵션 |
|---|----------|------|
| ① | Knowledge 기능 활성화 설정 위치 | Case 1-A / 1-B / **1-C** |
| ② | 문서 저장소 아키텍처 | Case 2-A / **2-B** |
| ③ | DAG 인터페이스 방식 | **Case 3-A** (Pull) / Case 3-B (Push) |
| ④ | 문서 처리 상태 표시 방식 | Case 4-A (단순) / **Case 4-B** (단계별) |
| ⑤ | Knowledge 설정 변경 시 Admin 재승인 필요 여부 | 필요 / **불필요** |
