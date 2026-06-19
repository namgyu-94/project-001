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

## Case 2 — Developer S3(MinIO) 사용 (Backend Proxy 방식)

### 파일 흐름

```
User 파일 선택
  → Platform Backend (POST /documents/upload)   ← 동일 Origin, CORS 없음
      Backend가 Developer MinIO에 스트리밍 PUT
      (Agent 등록 시 저장된 minio_endpoint + K8s Secret 자격증명 사용)
  → Backend 업로드 완료 확인 → DB 메타데이터 기록 (doc_id, object_path)
  → PENDING_APPROVAL
  → Developer 승인 → Backend가 Airflow Trigger
      conf: { minio_endpoint, bucket, object_path, doc_id, callback_url }
  → DAG Pod: K8s Secret의 Developer MinIO 자격증명으로 직접 접근
  → 임베딩 → Milvus 등록 → 콜백
```

**Presigned URL 불필요**: 브라우저가 Developer MinIO에 직접 접근하지 않으므로 PUT/GET 모두 서버 간 통신으로 처리.

---

### 고려사항

#### 1. 플랫폼 Backend 대역폭 — 파일 경유 부하

| 항목 | 내용 |
|------|------|
| 문제 | 모든 업로드 파일이 플랫폼 Backend를 경유 |
| 대응 | **스트리밍 처리 필수** — 파일을 메모리에 버퍼링하지 않고 MinIO SDK `put_object(stream)` 으로 즉시 전달 |
| 대용량 파일 | Multipart Upload 적용 (`part_size=10MB`) |
| 파일 크기 제한 | 업로드 허용 최대 크기 정책 결정 필요 (예: 100MB) |

#### 2. Backend → Developer MinIO 업로드 실패

| 원인 | 결과 | 대응 |
|------|------|------|
| Developer MinIO 네트워크 불가 | 업로드 실패 → User에게 오류 반환 | User 재시도 |
| 자격증명 오류 (만료·변경) | 403 → 업로드 차단 | Developer가 Agent 설정에서 키 재입력 |
| 버킷 미존재 | NoSuchBucket 오류 | Agent 등록 시 버킷 존재 여부 사전 검증 필요 |

#### 3. Developer 자격증명 관리

| 항목 | 내용 |
|------|------|
| 저장 위치 | K8s Secret — DB에는 Secret 이름만 참조 |
| Key Rotation | Developer가 키 교체 시 Agent 설정 화면에서 재입력 필요 |
| 보안 리스크 | 플랫폼이 Developer MinIO 키를 보관 → 플랫폼 침해 시 Developer 스토리지 노출 가능 |
| 완화 방안 | 키를 암호화 저장, Agent별 최소 권한(해당 버킷 PUT만 허용) 정책 권고 |

#### 4. DAG 실행 전 Developer가 파일 삭제

| 항목 | 내용 |
|------|------|
| 원인 | Developer가 자신의 MinIO를 직접 관리하다 실수로 삭제 |
| 결과 | DAG Pod `NoSuchKey` 오류 → FAILED |
| 대응 | 플랫폼 복구 불가. User 재업로드 안내 |
| 예방 | Agent 등록 가이드에 "플랫폼 처리 완료 전 파일 삭제 금지" 명시 |

#### 5. DAG 실패 시 파일 보관 정책

| 상태 | 파일 위치 | 플랫폼 개입 |
|------|----------|-----------|
| FAILED | Developer MinIO에 존재 | 삭제 불가 (소유권 없음) |
| REJECTED | Developer MinIO에 존재 | Developer가 직접 삭제 |
| COMPLETED | Developer MinIO에 존재 | Developer 정책에 따름 |

플랫폼은 파일 생명주기를 통제할 수 없으므로 **메타데이터 상태 관리만** 담당.

#### 6. Agent 등록 시 사전 검증 항목

Developer가 Agent 등록 시 아래를 자동 검증하여 운영 오류 예방:

```
① MinIO 엔드포인트 접근 가능 여부 (ping)
② AccessKey / SecretKey 유효성 (LIST Objects)
③ 지정 버킷 존재 여부 (HEAD Bucket)
④ 버킷 PUT 권한 존재 여부 (작은 테스트 파일 PUT 후 삭제)
```

---

## Case 1 vs Case 2 비교 요약

| 고려사항 | Case 1 (플랫폼 S3) | Case 2 (Developer S3, Proxy) |
|---------|------------------|------------------------------|
| 브라우저 CORS | 없음 | **없음** (Backend 경유로 해결) |
| Presigned URL | Pod 실행 시 재발급 or K8s Secret 직접 접근 | **불필요** (PUT·GET 모두 서버 간) |
| 플랫폼 대역폭 부하 | 업로드 수신만 | **업로드 수신 + Developer MinIO 전달** |
| DAG 실패 시 파일 | 플랫폼이 보관·삭제 통제 | Developer 스토리지라 플랫폼 개입 불가 |
| 파일 무결성 | 플랫폼이 보장 | Developer 삭제 시 복구 불가 |
| 자격증명 관리 | 플랫폼 MinIO 단일 키 (단순) | Developer별 키 보관 (K8s Secret) |
| 용량 관리 | 플랫폼 책임 (lifecycle 정책 필요) | Developer 책임 |
| 재시도 | 플랫폼 전체 통제 | FAILED 후 User 재업로드 필요 경우 있음 |
| 멱등성 | `doc_id` 기준 중복 체크 필요 | 동일 |

---

## 파일 보관 정책 상세 분석

작성일: 2026-06-16

---

### Case 1 — 플랫폼 S3가 있다고 가정

**처리 흐름**

```
User 업로드
  → Platform B/E → 플랫폼 S3 저장 → PENDING_APPROVAL
  → Developer 승인
  → Market B/E가 Airflow Trigger
  → DAG Pod: GET /api/documents/{doc_id}/presigned-url  ← 실행 시점에 발급
      Presigned URL로 플랫폼 S3에서 파일 Pull
  → 임베딩 → Milvus 등록 → 콜백
```

---

#### 1-A. Developer 승인 이후 End-User가 문서를 다운로드하지 않는다는 가정

```
Developer 승인
  → Market B/E가 Airflow Trigger (conf: { doc_id, callback_url })
  → DAG Pod 실행 시점에 Presigned URL 발급 요청
      GET /api/documents/{doc_id}/presigned-url
  → DAG Pod가 Presigned URL로 플랫폼 S3에서 파일 Pull
  → DAG Pod → B/E 콜백: pulling 완료 알림
      POST /api/documents/{doc_id}/status { status: "processing" }
  → B/E: 플랫폼 S3에서 해당 파일 즉시 삭제
      (Milvus 임베딩 파이프라인이 파일을 수신한 것이 확인됐으므로 원본 불필요)
  → 임베딩 완료 후 최종 콜백
      POST /api/documents/{doc_id}/status { status: "completed" }
```

**보관 정책 결정:**

| 시점 | 파일 상태 | 처리 |
|------|----------|------|
| PENDING_APPROVAL | 플랫폼 S3 보관 | TTL 7일 — 미승인 시 자동 삭제 |
| DAG pulling 완료 콜백 수신 | 역할 종료 | **즉시 삭제** |
| FAILED (pulling 실패) | 재시도 가능 | 30일 보관 후 자동 삭제 |
| REJECTED | 재업로드 원칙 | 즉시 삭제 |

> pulling 완료 콜백을 받은 시점이 삭제 트리거. DAG이 파일을 정상 수신했음이 확인됐으므로 플랫폼 S3에 원본을 유지할 이유가 없다.

---

#### 1-B. Developer 승인 이후에도 End-User가 문서를 다운로드한다는 가정

```
Developer 승인
  → Market B/E가 Airflow Trigger
  → DAG Pod가 Presigned URL로 플랫폼 S3에서 파일 Pull → 임베딩 완료
  → End-User가 원본 파일 다운로드 요청
      GET /api/documents/{doc_id}/download
  → B/E: Presigned URL 재발급 → End-User에게 전달
  → End-User가 플랫폼 S3에서 직접 다운로드
```

**보관 정책 결정:**

| 시점 | 파일 상태 | 처리 |
|------|----------|------|
| PENDING_APPROVAL | 플랫폼 S3 보관 | TTL 7일 |
| COMPLETED (임베딩 완료) | 원본 보관 필요 | **보관 유지** (User 다운로드 목적) |
| 보관 만료 | 정책 기간 경과 | N일 후 자동 삭제 (MinIO lifecycle rule) |

> 이 가정에서는 COMPLETED 이후에도 원본이 플랫폼 S3에 남아있어야 한다.
> 보관 기간 N을 결정해야 함 (예: 90일, 1년, 영구).
> 보관 기간이 길어질수록 플랫폼 S3 비용이 비례하여 증가한다.

**1-A vs 1-B 선택 기준:**

| 고려사항 | 1-A (삭제) | 1-B (보관) |
|---------|-----------|-----------|
| S3 비용 | 최소 | 보관 기간에 비례 |
| End-User 원본 다운로드 | 불가 | 가능 |
| 개인정보 노출 기간 | 최소화 | 보관 기간만큼 존재 |
| 구현 복잡도 | 단순 | 다운로드 API 추가 필요 |

---

### Case 2 — Developer 할당 S3가 있다고 가정

**처리 흐름**

```
User 업로드
  → Platform B/E (Streaming Proxy)
      Agent 등록 시 저장된 minio_endpoint + K8s Secret 자격증명으로
      Developer MinIO에 스트리밍 PUT
  → B/E 업로드 완료 확인 → DB 메타데이터 기록 (doc_id, object_path)
  → PENDING_APPROVAL
  → Developer 승인
  → Market B/E가 Airflow Trigger
      conf: { minio_endpoint, bucket, object_path, doc_id, callback_url }
  → DAG Pod: K8s Secret의 Developer MinIO 자격증명으로 직접 GET
      (Presigned URL 불필요 — 서버 간 직접 접근)
  → 임베딩 → Milvus 등록 → 콜백
```

---

#### 2-A. Developer 승인 이후 End-User가 문서를 다운로드하지 않는다는 가정

```
Developer 승인
  → Market B/E가 Airflow Trigger
      conf: { minio_endpoint, bucket, object_path, doc_id, callback_url }
  → DAG Pod: K8s Secret으로 Developer MinIO에서 직접 GET
  → DAG Pod → B/E 콜백: pulling 완료 알림
      POST /api/documents/{doc_id}/status { status: "processing" }
  → B/E: 상태 기록 (PROCESSING)
      ※ 플랫폼은 Developer MinIO 파일을 삭제할 수 없음 — 소유권 없음
  → 임베딩 완료 후 최종 콜백
      POST /api/documents/{doc_id}/status { status: "completed" }
  → B/E: Developer에게 "처리 완료, 원본 파일 삭제 권장" 알림 전송
```

**보관 정책 결정:**

| 시점 | 파일 위치 | 플랫폼 개입 가능 여부 | 처리 |
|------|----------|---------------------|------|
| PENDING_APPROVAL | Developer MinIO | ❌ 삭제 불가 | 메타데이터 TTL 7일 — 만료 시 "미승인 파일 삭제 요청" 알림 |
| pulling 완료 (PROCESSING) | Developer MinIO | ❌ | 상태 기록만 |
| COMPLETED | Developer MinIO | ❌ | Developer에게 삭제 권고 알림 전송 |
| FAILED | Developer MinIO | ❌ | FAILED 기록 + 재시도 가능 (conf 재사용) |
| REJECTED | Developer MinIO | ❌ | "반려 파일 삭제 요청" 알림 전송 |

**플랫폼이 직접 삭제하지 못하므로 Developer 자율 처리 방식 3가지:**

| 방식 | 설명 | 전제 조건 |
|------|------|----------|
| A. DAG 내 삭제 | 임베딩 성공 후 DAG Task에서 DELETE 실행 | DAG에 MinIO DELETE 권한 필요 |
| B. 플랫폼 삭제 요청 알림 | COMPLETED 후 Developer에게 알림 전송 | 강제성 없음 — Developer 수동 처리 |
| C. Developer Lifecycle Rule | MinIO 버킷에 N일 자동 삭제 규칙 적용 | Agent 등록 가이드에 의무화 명시 |

> **권장: A 방식** — DAG이 파일을 성공적으로 처리했다면 DAG 자신이 삭제하는 것이 가장 확실하다.  
> DELETE 권한은 Agent 등록 시 AccessKey 검증 단계(④)에 추가하면 된다.

---

#### 2-B. Developer 승인 이후에도 End-User가 문서를 다운로드한다는 가정

```
Developer 승인
  → Market B/E가 Airflow Trigger → DAG 처리 → COMPLETED
  → End-User가 원본 파일 다운로드 요청
      GET /api/documents/{doc_id}/download
  → B/E: Developer MinIO Presigned URL 발급
      minio_client.presigned_get_object(bucket, object_path, expires=15min)
  → End-User가 Developer MinIO에서 직접 다운로드
```

**플랫폼 S3 (Case 1-B)와의 차이점:**

| 항목 | Case 1-B (플랫폼 S3) | Case 2-B (Developer S3) |
|------|---------------------|------------------------|
| 다운로드 Presigned URL 발급 주체 | 플랫폼 B/E (플랫폼 MinIO 키 사용) | 플랫폼 B/E (Developer MinIO 키 사용) |
| 파일 보관 책임 | 플랫폼 | Developer |
| End-User 다운로드 가능 기간 | 플랫폼 보관 정책에 따름 | Developer MinIO에 파일이 존재하는 동안 |
| Developer가 파일 삭제 시 | 플랫폼이 복구 가능 | ❌ 복구 불가 — End-User 다운로드 차단 |

**보관 정책 결정:**

플랫폼은 Developer MinIO 파일의 존재를 보장할 수 없으므로, End-User 다운로드를 지원하려면 Agent 등록 약관에 다음을 명시해야 한다:

> "Knowledge 기능 활성 Agent의 Developer는 구독 사용자가 업로드한 문서를 플랫폼이 지정한 보관 기간 동안 삭제하지 않을 책임이 있습니다."

| 상태 | 파일 | 처리 |
|------|------|------|
| COMPLETED | Developer MinIO 보관 | Developer 책임 보관 (약관) |
| 보관 만료 | 정책 기간 경과 | Developer Lifecycle Rule 또는 DAG 내 지연 삭제 |
| Developer가 임의 삭제 | End-User 다운로드 불가 | 플랫폼은 오류 반환만 가능 |

---

## 공통 결정 필요 사항

1. **FAILED 문서 보관 기간**: 며칠 유지 후 삭제할 것인가?
2. **최대 재시도 횟수**: Airflow retry 3회 후에도 실패 시 자동 FAILED 전환
3. **User 알림**: 상태 변경 시 이메일/UI 알림 여부
4. **COMPLETED 후 원본 삭제**: 임베딩 완료 후 S3 원본 파일 삭제 여부 (Case 1)
5. **재처리 요청**: UI에서 User/Developer가 FAILED 문서를 수동으로 재시도할 수 있어야 하는가?
6. **Case 2 DAG DELETE 권한**: Agent 등록 시 AccessKey에 DELETE 권한을 의무화할 것인가? (2-A 방식 A 전제 조건)
7. **End-User 다운로드 지원 여부**: 1-A/2-A (삭제) vs 1-B/2-B (보관) 중 정책 확정 필요
