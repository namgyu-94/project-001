# Knowledge 문서 저장소 설계 고려사항

작성일: 2026-06-15  
대상: Case 1 (플랫폼 S3) / Case 2 (Developer S3)

---

## 문서 상태 머신 (공통)

```
업로드 완료
  → PENDING_APPROVAL   (Developer 승인 대기)
  → APPROVED           (DAG Trigger 완료)
  → PROCESSING         (KubernetesPodOperator 실행 중)
  → COMPLETED          (Milvus 등록 완료)
  → FAILED             (처리 실패 — 재시도 가능)
  → REJECTED           (Developer 반려 — 재업로드 필요)
```

상태 전이는 DAG 콜백 `POST /documents/{doc_id}/status` 로 수행.  
플랫폼 Backend가 상태를 DB에 기록하고 User에게 노출.

---

## Case 1 — 플랫폼 S3(MinIO) 사용

### 파일 흐름

```
User 업로드
  → Platform Backend → 플랫폼 MinIO 저장
  → Developer 승인 → Backend가 Airflow Trigger
      conf: { doc_id, callback_url }
  → DAG Pod: GET /api/documents/{doc_id}/presigned-url  ← 실행 시점에 발급
      → Presigned URL로 플랫폼 MinIO에서 파일 Pull
  → 임베딩 → Milvus 등록 → 콜백
```

> **중요**: Presigned URL은 DAG Trigger 시점이 아닌 **Pod 실행 시점**에 발급.  
> conf에 URL을 담으면 Queue 대기 중 만료될 수 있음.

---

### 고려사항

#### 1. Presigned URL 만료 — Pod 실행 전 만료

| 항목 | 내용 |
|------|------|
| 원인 | DAG Queue 적체 → Pod 실행이 URL 발급 시점보다 늦어짐 |
| 대응 | Pod가 실행 시점에 플랫폼 API 호출로 URL 재발급 (`GET /documents/{doc_id}/presigned-url`) |
| 전제 | Pod → 플랫폼 Backend 접근 가능한 네트워크 구성 필요 |
| 대안 | conf에 URL 대신 `doc_id`만 전달하고 Pod가 직접 MinIO SDK로 접근 (플랫폼 MinIO 자격증명을 K8s Secret으로 Pod에 주입) |

#### 2. DAG 실패 시 파일 보관 정책

| 상태 | 파일 보관 여부 | 이유 |
|------|--------------|------|
| PROCESSING 중 실패 | **유지** | Airflow retry 또는 수동 재시도 가능해야 함 |
| 최대 재시도 초과 → FAILED | **유지** (보관 기간 정책 필요) | 원인 분석 및 수동 재처리를 위해 |
| COMPLETED | 삭제 또는 아카이브 선택 | Milvus에 임베딩됐으므로 원본 유지 여부는 정책 결정 |
| REJECTED | 즉시 삭제 가능 | 반려된 파일은 재업로드가 원칙 |

**권장 정책**: FAILED 파일은 30일 보관 후 자동 삭제 (MinIO lifecycle rule).

#### 3. 저장소 누적 — 용량 관리

- 모든 테넌트(User) 파일이 플랫폼 MinIO 단일 버킷에 집적됨
- 멀티테넌트 격리: `platform-docs/{agent_id}/{user_id}/{doc_id}/` 형태로 prefix 분리
- COMPLETED 후 원본 삭제 정책이 없으면 무한 누적

#### 4. 재시도 전략

```
Airflow 레벨 retry: retries=3, retry_delay=5min
  → Transient 오류(네트워크, MinIO 일시 불가) 자동 복구

비즈니스 오류(파일 없음, 임베딩 모델 오류):
  → 자동 retry 무의미
  → FAILED 상태로 전환 + User 알림
  → UI에서 "재처리 요청" 버튼으로 수동 재시도
```

#### 5. 멱등성 — 중복 Trigger

- Developer가 실수로 동일 문서를 두 번 승인 → DAG 두 번 Trigger → Milvus 중복 임베딩
- 대응: `doc_id` 기준 Milvus 중복 체크 또는 PROCESSING 상태에서 재승인 차단

#### 6. 접근 제어

- Presigned URL은 시간 제한이 있으나, URL 유출 시 해당 시간 내 누구나 접근 가능
- URL 유효시간: **15분 이하 권장** (Pod 실행 직전 발급이므로 짧아도 무방)

---

## Case 2 — Developer S3(MinIO) 사용

### 파일 흐름

```
User 업로드 요청
  → Platform Backend: Developer MinIO에 Presigned PUT URL 생성
      (Agent 등록 시 입력된 minio_endpoint + AccessKey + SecretKey 사용)
  → Frontend: Presigned PUT URL로 Developer MinIO에 직접 PUT
  → 업로드 완료 신호 → 플랫폼 DB 메타데이터 기록 (doc_id, object_path)
  → Developer 승인 → Backend가 Airflow Trigger
      conf: { minio_endpoint, bucket, object_path, doc_id, callback_url }
  → DAG Pod: K8s Secret의 Developer MinIO 자격증명으로 직접 접근
  → 임베딩 → Milvus 등록 → 콜백
```

---

### 고려사항

#### 1. 업로드 완료 확인 — 프론트엔드 신뢰 문제

| 항목 | 내용 |
|------|------|
| 문제 | 브라우저가 Presigned PUT 완료 후 플랫폼에 신호를 안 보내면 메타데이터 미기록 |
| 대응 | Frontend가 PUT 성공 후 `POST /documents/complete` 호출 (필수 단계) |
| 추가 검증 | Backend가 MinIO HEAD Object로 파일 실제 존재 여부 확인 후 PENDING_APPROVAL 전환 |

#### 2. DAG 실행 전 Developer가 파일 삭제

| 항목 | 내용 |
|------|------|
| 원인 | Developer가 자신의 MinIO를 직접 관리하다가 실수로 파일 삭제 |
| 결과 | DAG Pod에서 `NoSuchKey` 오류 → FAILED |
| 대응 | 플랫폼은 복구 불가. User에게 재업로드 안내 |
| 예방 | Agent 등록 문서에 "플랫폼 처리 완료 전 파일 삭제 금지" 명시 |

#### 3. Developer 자격증명 변경 (Key Rotation)

| 항목 | 내용 |
|------|------|
| 문제 | Developer가 MinIO AccessKey를 교체하면 플랫폼이 보관 중인 키가 무효화 |
| 영향 | Presigned URL 생성 실패 → 신규 업로드 불가 / DAG 실행 실패 |
| 대응 | Agent 설정 화면에서 MinIO 자격증명 재입력 기능 제공 필요 |

#### 4. 플랫폼이 Developer 자격증명 보관 — 보안 리스크

| 항목 | 내용 |
|------|------|
| 리스크 | 플랫폼 DB/설정에 Developer MinIO 키가 저장 → 플랫폼 침해 시 Developer 스토리지 노출 |
| 대응 | 자격증명은 K8s Secret으로만 저장, DB 암호화 |
| 대안 | Developer가 Upload API를 직접 제공하는 방식 (Case B — 플랫폼이 키 미보유) 으로 전환 가능 |

#### 5. DAG 실패 시 파일 보관 정책

| 상태 | 파일 위치 | 플랫폼 개입 |
|------|----------|-----------|
| FAILED | Developer MinIO에 존재 | 플랫폼이 삭제 불가 (소유권 없음) |
| REJECTED | Developer MinIO에 존재 | Developer가 직접 삭제 |
| COMPLETED | Developer MinIO에 존재 | Developer 정책에 따름 |

플랫폼은 파일 생명주기를 통제할 수 없으므로 **메타데이터 상태 관리만** 담당.

#### 6. Presigned URL 만료 — PUT 실패

- PUT용 Presigned URL을 발급하고 User가 대용량 파일을 업로드하는 중 만료
- URL 유효시간: 업로드 예상 시간을 고려하여 **30~60분** 권장
- 만료 시 Frontend에서 재발급 요청 후 재업로드 필요

---

## Case 1 vs Case 2 비교 요약

| 고려사항 | Case 1 (플랫폼 S3) | Case 2 (Developer S3) |
|---------|------------------|----------------------|
| Presigned URL 만료 | **Pod 실행 시점 재발급**으로 해결 | PUT URL은 길게 설정, GET은 K8s Secret 직접 접근으로 불필요 |
| DAG 실패 시 파일 | 플랫폼이 보관·삭제 통제 가능 | Developer 스토리지라 플랫폼 개입 불가 |
| 파일 무결성 | 플랫폼이 보장 | Developer가 파일 삭제 시 복구 불가 |
| 자격증명 관리 | 플랫폼 MinIO 단일 키 (단순) | Developer별 키 보관 (복잡, 보안 리스크) |
| 용량 관리 | 플랫폼이 책임 (lifecycle 정책 필요) | Developer 책임 |
| 재시도 | 플랫폼이 전체 통제 | FAILED 후 User 재업로드 필요한 경우 발생 |
| 멱등성 | `doc_id` 기준 중복 체크 필요 | 동일 |

---

## 공통 결정 필요 사항

1. **FAILED 문서 보관 기간**: 며칠 유지 후 삭제할 것인가?
2. **최대 재시도 횟수**: Airflow retry 3회 후에도 실패 시 자동 FAILED 전환
3. **User 알림**: 상태 변경 시 이메일/UI 알림 여부
4. **COMPLETED 후 원본 삭제**: 임베딩 완료 후 S3 원본 파일 삭제 여부 (Case 1)
5. **재처리 요청**: UI에서 User/Developer가 FAILED 문서를 수동으로 재시도할 수 있어야 하는가?
