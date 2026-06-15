# Developer S3 Case B — Developer가 Upload API 직접 제공

**전제**: 플랫폼 및 Developer 모두 AWS S3가 아닌 **MinIO(S3 호환) 오브젝트 스토리지** 사용.

**관련 파일**
- `requirements/20260611_developer_s3_case_a.md` — Case A (플랫폼이 MinIO 자격증명으로 Presigned URL 생성)
- `requirements/20260611_dag_s3_access_cases.md` — DAG S3 접근 방식 비교
- `requirements/20260610_knowledge_requirements.md` — Knowledge 파이프라인 전체 흐름

---

## 개념

Developer가 **자체 서버에 Upload API 엔드포인트를 구현**하고 플랫폼에 URL만 등록한다.  
End-user 업로드 요청 시 플랫폼이 Developer API를 호출하여 Presigned PUT URL을 받고, End-user에게 전달한다.  
End-user 브라우저는 해당 URL로 **Developer MinIO에 바로 업로드**한다.  
플랫폼은 Developer MinIO 자격증명을 보유하지 않으며, **메타데이터만 관리**한다.

```
End-user → 플랫폼 백엔드 (Presigned URL 요청)
  → 플랫폼이 Developer Upload API 호출
  → Developer 서버가 자체 MinIO 자격증명으로 Presigned PUT URL 생성 후 반환
  → End-user 브라우저 → Developer MinIO 직접 PUT 업로드
  → Developer 승인 → Airflow DAG
```

---

## Agent 등록 시 입력 항목 (Knowledge 활성화 전제)

Developer가 Agent 등록 신청 Step 1에서 Knowledge 토글 ON 후 아래 항목을 입력한다.

| 항목 | 설명 |
|------|------|
| Upload API URL | Presigned PUT URL 발급 엔드포인트 (예: `https://api.developer.com/upload/presigned-url`) |
| 파일 삭제 API URL | 반려 시 호출할 삭제 엔드포인트 (예: `https://api.developer.com/upload/files/{doc_id}`) |
| API 인증 키 | 플랫폼이 Developer API 호출 시 사용할 Bearer Token |
| Airflow DAG ID | 임베딩 처리에 사용할 DAG |

> MinIO 엔드포인트, 버킷명, 자격증명은 Developer API 응답에 포함되므로 플랫폼에 직접 노출하지 않아도 된다.

---

## Developer Upload API 인터페이스 규격

플랫폼이 호출하는 요청/응답 형식. Developer가 이 규격에 맞게 구현해야 한다.

### Presigned URL 발급
```
POST {developer_upload_api_url}
Authorization: Bearer {developer_api_key}
Content-Type: application/json

요청 본문:
{
  "agent_id": "agent01",
  "user_id": "user01",
  "doc_id": "doc_abc123",
  "file_name": "계약서_v3.pdf",
  "content_type": "application/pdf"
}

응답:
{
  "presigned_put_url": "http://minio.developer.internal:9000/agent-knowledge/...?X-Amz-...",
  "minio_endpoint": "http://minio.developer.internal:9000",
  "bucket": "agent-knowledge",
  "object_path": "agents/agent01/user01/doc_abc123/계약서_v3.pdf",
  "expires_in": 1800
}
```

> Developer 서버 내부에서 MinIO Python SDK로 Presigned URL 생성:
> ```python
> minio_client = Minio(endpoint, access_key, secret_key)
> url = minio_client.presigned_put_object(bucket, object_name, expires=timedelta(minutes=30))
> ```

### 파일 삭제 (반려 시)
```
DELETE {developer_delete_api_url}/{doc_id}
Authorization: Bearer {developer_api_key}

응답:
HTTP 200 OK
```

---

## 전체 흐름

```
[업로드 단계 — Case B 핵심]
End-user 파일 선택 및 업로드 요청 (플랫폼 UI)
  → 플랫폼 백엔드:
      1. doc_id 생성
      2. Agent 등록 정보에서 Developer Upload API URL + 인증 키 조회
      3. POST {developer_upload_api_url}
             요청: { agent_id, user_id, doc_id, file_name, content_type }
             응답: { presigned_put_url, minio_endpoint, bucket, object_path }
      4. DB: doc 상태 = "업로드 대기", minio_endpoint + bucket + object_path 저장
      5. End-user에게 presigned_put_url 반환

End-user 브라우저 → Developer MinIO에 직접 PUT 업로드 (플랫폼 트래픽 없음)

업로드 완료 후 End-user → 플랫폼에 업로드 완료 알림
  → DB: doc 상태 = "승인 대기"
  → Developer 문서 승인 패널에 표시

[승인 단계]
Developer 승인 클릭
  → 플랫폼 백엔드:
      DB: doc 상태 = "승인"
      Airflow DAG Trigger
        conf에 minio_endpoint + bucket + object_path 포함

[DAG 실행 단계]
DAG이 conf에서 MinIO 경로 파싱
  → K8s Secret에서 Developer MinIO 자격증명 로드
  → MinIO에서 파일 직접 처리 (임베딩)
  → POST /api/documents/{doc_id}/status (completed / failed)

[반려 단계]
Developer 반려 클릭
  → 플랫폼 백엔드:
      DB: doc 상태 = "반려"
      DELETE {developer_delete_api_url}/{doc_id}  (Developer 서버가 MinIO 파일 삭제)
```

---

## 플랫폼 백엔드 처리 내용

| 단계 | 처리 | 내용 |
|------|------|------|
| 업로드 요청 | Developer API 호출 | Presigned PUT URL 취득 후 End-user에게 전달 |
| 업로드 완료 알림 | 상태 업데이트 | `업로드 대기` → `승인 대기` |
| 승인 | DAG Trigger | conf에 MinIO 경로 정보 포함 |
| 반려 | Developer 삭제 API 호출 | Developer 서버가 MinIO 파일 삭제 처리 |

> 플랫폼 자체 스토리지 저장·이전 없음. 플랫폼은 Developer MinIO 자격증명을 보유하지 않음.

---

## Trigger conf 구조

Case A와 동일 구조.

```json
{
  "dag_run_id": "agentmarket_doc_20260611_abc123",
  "logical_date": "2026-06-11T10:00:00+00:00",
  "conf": {
    "request_doc": {
      "doc_id": "doc_abc123",
      "minio_endpoint": "http://minio.developer.internal:9000",
      "bucket": "agent-knowledge",
      "object_path": "agents/agent01/user01/doc_abc123/계약서_v3.pdf",
      "file_name": "계약서_v3.pdf",
      "source_type": "user"
    },
    "method": "embed"
  }
}
```

---

## DAG 템플릿

Case A와 동일. conf 구조가 동일하므로 DAG 템플릿을 공유한다.

```python
import requests
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator
from airflow.models import Variable
from kubernetes.client import models as k8s
from datetime import datetime

PLATFORM_API_BASE  = Variable.get("agentmarket_api_base_url")
PLATFORM_API_TOKEN = Variable.get("agentmarket_platform_token")


def parse_conf(**context):
    conf = context["dag_run"].conf
    req  = conf["request_doc"]

    doc_id         = req["doc_id"]
    minio_endpoint = req["minio_endpoint"]
    bucket         = req["bucket"]
    object_path    = req["object_path"]
    method         = conf.get("method", "embed")

    if not doc_id or not object_path:
        raise ValueError(f"conf 누락: doc_id={doc_id}, object_path={object_path}")

    context["ti"].xcom_push(key="doc_id",          value=doc_id)
    context["ti"].xcom_push(key="minio_endpoint",  value=minio_endpoint)
    context["ti"].xcom_push(key="bucket",          value=bucket)
    context["ti"].xcom_push(key="object_path",     value=object_path)
    context["ti"].xcom_push(key="method",          value=method)


def report_status(status: str, **context):
    doc_id = context["ti"].xcom_pull(key="doc_id")
    requests.post(
        f"{PLATFORM_API_BASE}/documents/{doc_id}/status",
        json={"status": status},
        headers={"Authorization": f"Bearer {PLATFORM_API_TOKEN}"},
        timeout=10,
    ).raise_for_status()


def on_success(**context): report_status("completed", **context)
def on_failure(**context): report_status("failed",    **context)


with DAG(
    dag_id="{{ developer_dag_id }}",
    start_date=datetime(2026, 1, 1),
    schedule_interval=None,
    catchup=False,
    on_failure_callback=on_failure,
) as dag:

    parse = PythonOperator(
        task_id="parse_conf",
        python_callable=parse_conf,
        provide_context=True,
    )

    embed = KubernetesPodOperator(
        task_id="embedding_pod",
        image="{{ developer_image }}",
        namespace="{{ k8s_namespace }}",

        # MinIO 자격증명은 K8s Secret에서 주입 (conf에 포함하지 않음)
        env_from=[
            k8s.V1EnvFromSource(
                secret_ref=k8s.V1SecretEnvSource(name="{{ developer_minio_secret }}")
            )
        ],
        # Secret에 포함되어야 할 키: MINIO_ACCESS_KEY, MINIO_SECRET_KEY

        env_vars=[
            k8s.V1EnvVar(name="DOC_ID",
                value="{{ ti.xcom_pull(key='doc_id') }}"),
            k8s.V1EnvVar(name="MINIO_ENDPOINT",
                value="{{ ti.xcom_pull(key='minio_endpoint') }}"),
            k8s.V1EnvVar(name="MINIO_BUCKET",
                value="{{ ti.xcom_pull(key='bucket') }}"),
            k8s.V1EnvVar(name="MINIO_OBJECT_PATH",
                value="{{ ti.xcom_pull(key='object_path') }}"),
            k8s.V1EnvVar(name="EMBED_METHOD",
                value="{{ ti.xcom_pull(key='method') }}"),
        ],
        is_delete_operator_pod=True,
        get_logs=True,
    )

    report = PythonOperator(
        task_id="report_success",
        python_callable=on_success,
        provide_context=True,
    )

    parse >> embed >> report
```

### Developer가 채워야 할 항목

| 항목 | 설명 |
|------|------|
| `{{ developer_dag_id }}` | agentMarket 등록 시 입력한 DAG ID |
| `{{ developer_image }}` | 임베딩 처리 컨테이너 이미지 |
| `{{ k8s_namespace }}` | Pod 실행 네임스페이스 |
| `{{ developer_minio_secret }}` | MinIO 자격증명이 담긴 K8s Secret 이름 (`MINIO_ACCESS_KEY`, `MINIO_SECRET_KEY` 포함) |
| `agentmarket_api_base_url` | Airflow Variable — 플랫폼 API URL |
| `agentmarket_platform_token` | Airflow Variable — 플랫폼 발급 토큰 |

---

## Case A vs Case B 비교 (Developer MinIO 공통 전제)

| 항목 | Case A (플랫폼이 자격증명 보유) | Case B (Developer Upload API) |
|------|-------------------------------|-------------------------------|
| **Presigned URL 생성 주체** | 플랫폼 (Developer 자격증명 사용) | Developer 자체 서버 |
| **Developer 등록 항목** | MinIO Endpoint + Bucket + AccessKey + SecretKey | Upload API URL + 삭제 API URL + API 인증 키 |
| **플랫폼의 MinIO 접근** | 있음 (PutObject, DeleteObject) | 없음 |
| **반려 시 파일 삭제** | 플랫폼이 직접 MinIO에서 삭제 | Developer 삭제 API 호출 |
| **Developer 측 구현 필요** | MinIO 키 등록만 | Upload API + 삭제 API 구현 필요 |
| **보안 측면** | 플랫폼이 MinIO 자격증명 보유 | 플랫폼이 자격증명 미보유 (더 안전) |
| **DAG Trigger conf** | 동일 | 동일 |
| **DAG 템플릿** | 동일 | 동일 |

---

## 장단점

| 구분 | 내용 |
|------|------|
| 장점 | 플랫폼이 Developer MinIO 자격증명을 전혀 보유하지 않음 — 보안 우려 없음 |
| 장점 | Developer가 MinIO 경로, 저장 정책, 접근 제어를 완전히 자율 관리 |
| 단점 | Developer가 Upload API + 삭제 API를 직접 구현해야 함 |
| 단점 | Developer 서버 장애 시 플랫폼 업로드 흐름 전체가 영향받음 |
| 단점 | End-user 브라우저에서 Developer MinIO로 직접 업로드하므로 CORS 설정 필요 |
