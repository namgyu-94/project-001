# S3 방식별 문서 처리 프로세스 및 백엔드 조치

작성일: 2026-06-19  
대상: ui-requirements_copy.html Knowledge 기능 — 플랫폼 S3 / Developer S3 비교

---

## Case 1: 플랫폼 S3

```
User 파일 선택
  → Platform B/E → 플랫폼 MinIO 저장
       (object_path: platform-s3/agents/{agent_id}/users/{uid}/{filename})
  → doc-upload 화면에 "승인 대기" 상태 노출

Developer 승인 (doc-approve 패널)
  → Platform B/E가 Airflow REST API로 DAG Trigger
       conf: { "doc_id": "xxx" }   ← 파일 위치 정보 없음, doc_id만 전달

DAG Pod 실행
  → GET /api/documents/{doc_id}/presigned-url   ← 플랫폼 API 호출
  → 반환된 Presigned URL로 파일 pull
  → 임베딩 → Milvus VectorDB 등록
  → POST /api/documents/{doc_id}/status  { status: "completed" }

Platform B/E (콜백 수신)
  → 플랫폼 MinIO에서 파일 즉시 삭제 (Case 1-A)
```

**DAG이 파일을 찾는 방법**: `doc_id`로 플랫폼 API를 쿼리 → 플랫폼이 Presigned URL 발급. DAG은 자격증명 없이 URL만으로 파일에 접근.

---

## Case 2: Developer S3

```
User 파일 선택
  → Platform B/E (Streaming Proxy) → Developer MinIO 직접 PUT
       (플랫폼 MinIO 경유 없음 — 이중 저장 금지)
  → doc-upload 화면에 "승인 대기" 상태 노출

Developer 승인 (doc-approve 패널)
  → Platform B/E가 Airflow REST API로 DAG Trigger
       conf: {
         "doc_id":         "xxx",
         "minio_endpoint": "http://minio.developer.internal:9000",
         "bucket":         "agent-knowledge-docs",
         "object_path":    "agents/{agent_id}/users/{uid}/{filename}",
         "callback_url":   "https://platform/api/documents/xxx/status"
       }   ← 파일 위치 정보를 conf에 직접 담아서 전달

DAG Pod 실행
  → conf에서 minio_endpoint / bucket / object_path 읽음
  → K8s Secret에서 AccessKey / SecretKey 읽어 직접 인증
  → Developer MinIO에 직접 GET → 파일 pull
  → 임베딩 → Milvus VectorDB 등록
  → POST {callback_url}  { status: "completed" }
  → (Case 2-A) Developer MinIO에서 파일 DELETE

Platform B/E (콜백 수신)
  → DB 상태만 업데이트 (파일 삭제는 DAG이 처리)
```

**DAG이 파일을 찾는 방법**: conf에 파일 위치가 포함되어 있으므로 플랫폼 API 호출 없이 바로 접근. 인증은 K8s Secret 경유.

---

## 백엔드 조치 — Airflow가 Developer S3를 찾을 수 있으려면

### 1. 자격증명 → K8s Secret 저장 (Agent 등록 시)

Developer가 apply 패널에서 Developer MinIO 정보를 입력하고 연결 검증이 통과되면, 플랫폼 백엔드가 해당 자격증명을 K8s Secret으로 저장합니다.

```yaml
# Secret 명칭 규칙: minio-creds-{agent_id}
apiVersion: v1
kind: Secret
metadata:
  name: minio-creds-process-defect-agent
  namespace: airflow          # DAG Pod가 실행되는 네임스페이스
type: Opaque
stringData:
  MINIO_ACCESS_KEY: "AKIAIOSFODNN7EXAMPLE"
  MINIO_SECRET_KEY: "wJalrXUtnFEMI/K7MDENG/..."
```

- 백엔드(플랫폼 API)가 K8s API를 통해 Secret을 생성·갱신
- Agent 폐기 시 Secret도 함께 삭제

### 2. DAG Trigger conf에 파일 위치 정보 포함

Developer 승인 이벤트 발생 시, 백엔드가 Airflow DAG을 Trigger하면서 아래 정보를 conf에 담습니다.

```json
POST /api/v1/dags/{dag_id}/dagRuns
Authorization: Bearer {jwt_token}

{
  "conf": {
    "doc_id":         "doc-20260619-abc123",
    "minio_endpoint": "http://minio.developer.internal:9000",
    "bucket":         "agent-knowledge-docs",
    "object_path":    "agents/process-defect-agent/users/uid-001/report.pdf",
    "k8s_secret":     "minio-creds-process-defect-agent",
    "callback_url":   "https://platform.internal/api/documents/doc-20260619-abc123/status"
  }
}
```

- `k8s_secret` 필드를 넣어두면 DAG 코드가 어떤 Secret을 마운트할지 동적으로 판단 가능
- `callback_url`을 conf에 포함시키면 DAG이 플랫폼 엔드포인트를 하드코딩하지 않아도 됨

---

## Case 2 전체 흐름 요약

```
[Agent 등록 신청]
  Developer MinIO 자격증명 입력·검증 통과
    → Platform B/E: K8s Secret 생성 (minio-creds-{agent_id})

[문서 업로드]
  User 업로드 요청
    → Platform B/E: Developer MinIO에 Streaming PUT
    → DB에 (object_path, bucket, minio_endpoint, agent_id) 저장

[Developer 승인]
  Developer 승인 클릭
    → Platform B/E:
        ① DB에서 doc 정보 조회 (object_path 등)
        ② Airflow DAG Trigger (conf에 위치 + k8s_secret 포함)
        ③ doc 상태 → "처리 중"

[DAG 실행]
  conf에서 파일 위치 파싱
    → K8s Secret 마운트 → AccessKey/SecretKey 로드
    → Developer MinIO: GET {object_path}
    → 임베딩 → Milvus 등록
    → callback_url로 completed 콜백
    → Developer MinIO: DELETE {object_path}
```

---

## 케이스 비교 요약

| 항목 | 플랫폼 S3 | Developer S3 |
|------|-----------|--------------|
| conf 내용 | `doc_id`만 | `doc_id` + 파일 위치 + k8s_secret |
| DAG 인증 | Presigned URL (자격증명 불필요) | K8s Secret 마운트 |
| 파일 위치 획득 | 플랫폼 API 쿼리 (`/presigned-url`) | conf 직접 파싱 |
| 파일 삭제 주체 | Platform B/E (콜백 수신 후) | DAG Pod (임베딩 완료 후) |
| 백엔드 추가 조치 | 없음 | K8s Secret 생성·관리 필요 |
