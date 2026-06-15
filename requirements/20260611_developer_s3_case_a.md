# Developer S3 Case A — 플랫폼이 Developer MinIO 자격증명으로 Presigned URL 생성

**전제**: 플랫폼 및 Developer 모두 AWS S3가 아닌 **MinIO(S3 호환) 오브젝트 스토리지** 사용.

**관련 파일**
- `requirements/20260611_developer_s3_case_b.md` — Case B (Developer가 Upload API 직접 제공)
- `requirements/20260611_dag_s3_access_cases.md` — DAG S3 접근 방식 비교
- `requirements/20260610_knowledge_requirements.md` — Knowledge 파이프라인 전체 흐름

---

## 개념

Developer가 Agent 등록 시 자신의 **MinIO 엔드포인트 + 자격증명(AccessKey/SecretKey)** 을 플랫폼에 등록한다.  
End-user 업로드 요청 시 플랫폼이 Developer의 MinIO 자격증명으로 **Presigned PUT URL을 직접 생성**하여 End-user에게 전달한다.  
End-user 브라우저는 해당 URL로 **Developer MinIO에 바로 업로드**한다.  
플랫폼 자체 스토리지를 경유하지 않으며, **메타데이터만 관리**한다.

```
End-user → 플랫폼 백엔드 (Presigned URL 요청)
  → 플랫폼이 Developer MinIO 자격증명으로 Presigned PUT URL 생성
  → End-user 브라우저 → Developer MinIO 직접 PUT 업로드
  → Developer 승인 → Airflow DAG
```

---

## Agent 등록 시 입력 항목 (Knowledge 활성화 전제)

Developer가 Agent 등록 신청 Step 1에서 Knowledge 토글 ON 후 아래 항목을 입력한다.

| 항목 | 설명 |
|------|------|
| MinIO Endpoint URL | Developer MinIO 서버 주소 (예: `http://minio.developer.internal:9000`) |
| Bucket 명 | 파일이 저장될 버킷 (예: `agent-knowledge`) |
| 저장 경로 Prefix | 파일 저장 경로 접두사 (예: `agents/agent01/knowledge/`) |
| AccessKey | Presigned URL 생성에 사용할 MinIO 액세스 키 |
| SecretKey | 위 AccessKey의 시크릿 키 |
| Airflow DAG ID | 임베딩 처리에 사용할 DAG |

> 등록된 자격증명은 플랫폼 내부 Secret Store(Vault 또는 K8s Secret)에 암호화 저장.  
> 업로드용 최소 권한(PutObject)만 부여된 전용 키 사용 권장.

---

## 전체 흐름

```
[업로드 단계 — Case A 핵심]
End-user 파일 선택 및 업로드 요청 (플랫폼 UI)
  → 플랫폼 백엔드:
      1. doc_id 생성
      2. Agent 등록 정보에서 Developer MinIO 자격증명 조회
             endpoint, bucket, prefix, access_key, secret_key
      3. MinIO Python SDK로 Presigned PUT URL 생성
             minio_client = Minio(endpoint, access_key, secret_key)
             url = minio_client.presigned_put_object(bucket, object_name, expires=timedelta(minutes=30))
      4. DB: doc 상태 = "업로드 대기", object_path 저장
             object_path = {bucket}/{prefix}/{user_id}/{doc_id}/{file_name}
      5. End-user에게 Presigned PUT URL 반환

End-user 브라우저 → Developer MinIO에 직접 PUT 업로드 (플랫폼 트래픽 없음)

업로드 완료 후 End-user → 플랫폼에 업로드 완료 알림
  → DB: doc 상태 = "승인 대기"
  → Developer 문서 승인 패널에 표시

[승인 단계]
Developer 승인 클릭
  → 플랫폼 백엔드:
      DB: doc 상태 = "승인"
      Airflow DAG Trigger
        conf에 Developer MinIO 엔드포인트 + object_path 포함

[DAG 실행 단계]
DAG이 conf에서 MinIO 경로 파싱
  → K8s Secret에서 Developer MinIO 자격증명 로드
  → MinIO에서 파일 직접 처리 (임베딩)
  → POST /api/documents/{doc_id}/status (completed / failed)

[반려 단계]
Developer 반려 클릭
  → 플랫폼 백엔드:
      DB: doc 상태 = "반려"
      Developer MinIO 자격증명으로 파일 삭제
        minio_client.remove_object(bucket, object_name)
```

---

## 플랫폼 백엔드 처리 내용

| 단계 | 처리 | 내용 |
|------|------|------|
| 업로드 요청 | Presigned PUT URL 생성 | Developer MinIO 자격증명으로 직접 생성 |
| 업로드 완료 알림 | 상태 업데이트 | `업로드 대기` → `승인 대기` |
| 승인 | DAG Trigger | conf에 MinIO 엔드포인트 + object_path 포함 |
| 반려 | 파일 삭제 | Developer MinIO 자격증명으로 `remove_object` 호출 |

> 플랫폼 자체 스토리지 저장·이전 없음. 파일은 처음부터 끝까지 Developer MinIO에만 존재.

---

## Trigger conf 구조

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

- `file_path`(S3 URI) 대신 `minio_endpoint` + `bucket` + `object_path` 분리 전달
- MinIO 자격증명(AccessKey/SecretKey)은 conf에 포함하지 않음 → DAG이 K8s Secret에서 직접 로드

---

## DAG 템플릿

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

    doc_id        = req["doc_id"]
    minio_endpoint = req["minio_endpoint"]
    bucket        = req["bucket"]
    object_path   = req["object_path"]
    method        = conf.get("method", "embed")

    if not doc_id or not object_path:
        raise ValueError(f"conf 누락: doc_id={doc_id}, object_path={object_path}")

    context["ti"].xcom_push(key="doc_id",         value=doc_id)
    context["ti"].xcom_push(key="minio_endpoint", value=minio_endpoint)
    context["ti"].xcom_push(key="bucket",         value=bucket)
    context["ti"].xcom_push(key="object_path",    value=object_path)
    context["ti"].xcom_push(key="method",         value=method)


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

## 장단점

| 구분 | 내용 |
|------|------|
| 장점 | Developer가 별도 서버/API 구현 없이 MinIO 자격증명 등록만으로 연동 가능 |
| 장점 | 반려 시 플랫폼이 직접 파일 삭제 처리 |
| 단점 | Developer MinIO 자격증명을 플랫폼에 위임 — 최소 권한 키 사용 및 Secret 관리 필요 |
| 단점 | End-user 브라우저에서 Developer MinIO로 직접 업로드하므로 CORS 설정 필요 |
