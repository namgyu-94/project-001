# Developer S3 Case B — End-user가 Developer S3에 직접 업로드

**관련 파일**
- `requirements/20260611_developer_s3_case_a.md` — Case A (Platform S3 임시 저장 후 이전)
- `requirements/20260611_dag_s3_access_cases.md` — DAG S3 접근 방식 비교
- `requirements/20260610_knowledge_requirements.md` — Knowledge 파이프라인 전체 흐름

---

## 개념

플랫폼 S3를 거치지 않는다.  
End-user 업로드 시 플랫폼이 **Developer S3의 Presigned PUT URL**을 발급하여 End-user에게 전달하고,  
End-user 브라우저가 해당 URL로 **Developer S3에 직접 업로드**한다.  
플랫폼은 파일 본체를 다루지 않고 **메타데이터만 관리**한다.

```
End-user → [Presigned PUT URL 발급] → Developer S3 직접 업로드
  → 플랫폼 메타데이터 저장 → Developer 승인 → Airflow DAG
```

---

## Presigned PUT URL 발급 방식 (2가지)

### 방식 1 — 플랫폼이 Developer S3 접근 권한으로 직접 발급

플랫폼이 Developer S3에 대한 IAM 권한을 보유하여 Presigned PUT URL을 생성한다.

```
End-user 업로드 요청
  → 플랫폼 백엔드: Developer 등록 정보에서 S3 버킷·IAM Role 조회
  → Assume Role(Developer IAM Role)
  → S3 Presigned PUT URL 생성
  → End-user에게 URL 반환
```

- Developer 등록 시 플랫폼에 IAM Role ARN 제공 (Case A와 동일한 온보딩)
- 차이: Case A는 Copy(PutObject to target), Case B는 Presigned PUT(사용자 직접 업로드)

### 방식 2 — Developer가 Presigned URL 발급 API를 제공 (권장)

Developer가 자체 서비스에 **Presigned URL 발급 엔드포인트**를 구현하고 플랫폼에 등록한다.  
플랫폼은 해당 엔드포인트를 호출하기만 하면 되며, Developer S3 자격증명을 보유하지 않아도 된다.

```
End-user 업로드 요청
  → 플랫폼 백엔드: Developer 등록 정보에서 Presigned URL 발급 API URL 조회
  → POST {developer_upload_api}/presigned-url
       요청: { agent_id, user_id, file_name, doc_id }
       응답: { presigned_put_url, s3_path, expires_in }
  → End-user에게 presigned_put_url 반환
```

> **방식 2가 권장되는 이유**: Developer가 S3 자격증명을 플랫폼에 위임하지 않아도 된다.  
> 플랫폼은 업로드 URL 발급 요청만 중계하며, 실제 권한은 Developer 서버에 유지된다.

---

## 전체 흐름 (방식 2 기준)

```
[업로드 단계 — Case B 핵심 처리]
End-user 파일 선택 및 업로드 요청 (플랫폼 UI)
  → 플랫폼 백엔드:
      1. doc_id 생성
      2. Developer Presigned URL 발급 API 호출
             POST {developer_upload_api}/presigned-url
             요청: { agent_id, user_id, file_name, doc_id }
             응답: { presigned_put_url, s3_path }
      3. DB: doc 상태 = "업로드 대기", doc_id + s3_path(Developer S3 경로) 저장
      4. End-user에게 presigned_put_url 반환

End-user 브라우저 → Developer S3에 직접 PUT 업로드 (플랫폼 백엔드 트래픽 없음)

업로드 완료 후 End-user → 플랫폼에 업로드 완료 알림
  → 플랫폼 백엔드:
      DB: doc 상태 = "승인 대기"
  → Developer 문서 승인 패널에 표시

[승인 단계]
Developer 승인 클릭
  → 플랫폼 백엔드:
      1. DB: doc 상태 = "승인"
      2. Airflow DAG Trigger
             conf에 Developer S3 경로 포함 (DB에서 조회)

[DAG 실행 단계]
DAG이 conf에서 Developer S3 경로 파싱
  → Developer 자격증명으로 S3에서 파일 직접 조회 및 처리
  → 임베딩 완료
  → POST /api/documents/{doc_id}/status (completed / failed)

[반려 단계]
Developer 반려 클릭
  → 플랫폼 백엔드:
      1. DB: doc 상태 = "반려"
      2. Developer에게 삭제 요청 (선택):
             DELETE {developer_upload_api}/files/{doc_id}
      → Developer S3 파일 삭제 처리 (Developer 서버 책임)
```

---

## 플랫폼 백엔드 처리 내용

### 업로드 시
| 처리 | 내용 |
|------|------|
| Presigned URL 발급 | Developer API 호출로 PUT URL 취득 |
| 파일 저장 | 없음 — 플랫폼 S3 사용 안 함 |
| 메타데이터 | doc_id, Developer S3 경로(`s3_path`), 파일명, 업로드 요청 시각 DB 저장 |
| 상태 | `업로드 대기` → 업로드 완료 후 `승인 대기` |

### 승인 시
| 처리 | 내용 |
|------|------|
| S3 파일 이동 | 없음 — 파일은 이미 Developer S3에 있음 |
| DB 업데이트 | 상태 = `승인` |
| DAG Trigger | conf에 Developer S3 경로 포함 (DB s3_path 사용) |

### 반려 시
| 처리 | 내용 |
|------|------|
| Platform S3 파일 삭제 | 없음 — 플랫폼 S3 미사용 |
| Developer S3 파일 삭제 | Developer 삭제 API 호출 (선택) or Developer 자체 S3 Lifecycle 관리 |
| DB 업데이트 | 상태 = `반려` |

---

## Trigger conf 구조

Case A와 동일 — 승인 시점에 DB에 저장된 Developer S3 경로 포함.

```json
{
  "dag_run_id": "agentmarket_doc_20260611_abc123",
  "logical_date": "2026-06-11T10:00:00+00:00",
  "conf": {
    "request_doc": {
      "doc_id": "doc_abc123",
      "file_path": "s3://developer-bucket/agent01/user01/doc_abc123/계약서_v3.pdf",
      "file_name": "계약서_v3.pdf",
      "source_type": "user"
    },
    "method": "embed"
  }
}
```

- `file_path`는 업로드 시 Developer Presigned URL API 응답에서 받아 DB에 저장해 둔 경로
- Case A의 conf와 동일한 구조 — DAG 템플릿은 Case A와 공유 가능

---

## Developer가 플랫폼에 사전 등록해야 하는 정보

### 방식 1 (플랫폼이 IAM 권한 보유)
| 항목 | 설명 |
|------|------|
| Developer S3 버킷명 | Presigned PUT URL 발급 대상 버킷 |
| 저장 경로 Prefix | `agents/{agent_id}/` 등 Developer가 원하는 경로 구조 |
| Cross-Account IAM Role ARN | 플랫폼이 Presigned PUT URL 생성에 사용할 Role |

### 방식 2 (Developer가 API 제공, 권장)
| 항목 | 설명 |
|------|------|
| Presigned URL 발급 API URL | `POST /presigned-url` 엔드포인트 |
| 파일 삭제 API URL | `DELETE /files/{doc_id}` 엔드포인트 (반려 시 사용) |
| API 인증 방식 | API Key 또는 Bearer Token (플랫폼이 요청 시 사용) |

**Developer Presigned URL API 인터페이스 규격 (방식 2)**
```
POST {developer_upload_api}/presigned-url
Authorization: Bearer {platform_api_key}

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
  "presigned_put_url": "https://developer-bucket.s3.amazonaws.com/...?X-Amz-...",
  "s3_path": "s3://developer-bucket/agent01/user01/doc_abc123/계약서_v3.pdf",
  "expires_in": 1800
}
```

---

## DAG 템플릿

Case A와 동일한 템플릿 사용 가능.  
conf 구조가 동일하고 (doc_id + file_path), DAG은 Developer S3 경로로 직접 접근한다.

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
    """
    conf에서 Developer S3 경로 파싱.
    End-user가 Developer S3에 직접 업로드한 파일 경로가 포함되어 있음.
    """
    conf = context["dag_run"].conf
    req  = conf["request_doc"]

    doc_id    = req["doc_id"]
    file_path = req["file_path"]  # Developer S3 경로 (업로드 시점부터 Developer S3)
    method    = conf.get("method", "embed")

    if not doc_id or not file_path:
        raise ValueError(f"conf 누락: doc_id={doc_id}, file_path={file_path}")

    context["ti"].xcom_push(key="doc_id",    value=doc_id)
    context["ti"].xcom_push(key="file_path", value=file_path)
    context["ti"].xcom_push(key="method",    value=method)


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
        service_account_name="{{ developer_sa_with_s3_role }}",  # IRSA 방식
        env_vars=[
            k8s.V1EnvVar(name="DOC_ID",
                value="{{ ti.xcom_pull(key='doc_id') }}"),
            k8s.V1EnvVar(name="S3_FILE_PATH",
                value="{{ ti.xcom_pull(key='file_path') }}"),
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

> Case A DAG 템플릿과 동일. conf 구조가 동일하기 때문에 공유 가능하다.

---

## Case A vs Case B 비교 (Developer S3 시나리오)

| 항목 | Case A (Platform S3 경유 후 이전) | Case B (Developer S3 직접 업로드) |
|------|----------------------------------|----------------------------------|
| **업로드 경로** | End-user → Platform S3 → (승인 후) Developer S3 | End-user → Developer S3 직접 |
| **Platform S3 사용** | 임시 저장용으로 사용 | 사용 안 함 |
| **파일 이전 시점** | 승인 시 (Platform S3 → Developer S3 Copy) | 없음 (처음부터 Developer S3) |
| **플랫폼 IAM 요구** | Developer S3 PutObject 권한 필요 | 방식 1: 동일 / 방식 2: 불필요 |
| **비승인 파일 위치** | Platform S3 임시 저장 → 반려 시 삭제 | Developer S3에 저장됨 → 반려 시 Developer가 삭제 |
| **플랫폼 트래픽** | 파일 업로드 트래픽 플랫폼 경유 | 업로드 트래픽 없음 (Presigned URL 중계만) |
| **DAG Trigger conf** | 동일 구조 | 동일 구조 |
| **DAG 템플릿** | 동일 | 동일 |
| **Developer 온보딩** | IAM Role ARN 등록 | 방식 1: IAM Role ARN / 방식 2: API 엔드포인트 등록 |
| **반려 시 파일 처리** | 플랫폼이 Platform S3에서 즉시 삭제 | Developer 측 삭제 (API 또는 S3 Lifecycle) |

---

## 장단점

| 구분 | 내용 |
|------|------|
| 장점 | 파일이 처음부터 Developer S3에 저장 — 이전(Copy) 작업 불필요 |
| 장점 | 플랫폼 S3 저장 비용 및 트래픽 없음 (대용량 파일에 유리) |
| 장점 | 방식 2 선택 시 플랫폼이 Developer S3 자격증명을 보유할 필요 없음 |
| 단점 | 반려된 파일이 Developer S3에 존재 — 삭제 책임이 Developer에게 있음 |
| 단점 | Developer가 Presigned URL 발급 API를 직접 구현해야 함 (방식 2) |
| 단점 | 업로드 완료 후 플랫폼에 별도 완료 알림 필요 (비동기 처리) |
| 단점 | 플랫폼이 파일 업로드를 직접 검증할 수 없음 (바이러스 검사, 포맷 확인 등) |
