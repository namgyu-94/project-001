# Developer S3 Case A — Platform S3 임시 저장 후 승인 시 Developer S3로 이전

**관련 파일**
- `requirements/20260611_dag_s3_access_cases.md` — DAG S3 접근 방식 비교
- `requirements/20260610_knowledge_requirements.md` — Knowledge 파이프라인 전체 흐름

---

## 개념

End-user 업로드는 기존과 동일하게 **플랫폼 S3에 임시 저장**된다.  
Developer가 **승인**하는 시점에 플랫폼이 파일을 **Developer S3로 복사** 후 임시 파일을 삭제한다.  
DAG은 Developer S3 경로를 전달받아 직접 접근한다.

```
End-user → Platform S3 (임시) → [Developer 승인] → Developer S3 → Airflow DAG
```

---

## 전체 흐름

```
[업로드 단계 — 기존과 동일]
End-user 파일 선택 및 업로드 요청
  → 플랫폼 백엔드: 파일 수신
  → Platform S3 임시 저장
      platform-s3/staging/agents/{agent_id}/users/{user_id}/{doc_id}/{file_name}
  → DB: doc 상태 = "승인 대기", s3_path = 플랫폼 S3 임시 경로 저장
  → Developer 문서 승인 패널에 표시

[승인 단계 — Case A 핵심 처리]
Developer 승인 클릭
  → 플랫폼 백엔드:
      1. Platform S3 → Developer S3로 파일 복사 (S3 Copy API)
             developer-s3://{developer_bucket}/{agent_id}/{user_id}/{doc_id}/{file_name}
      2. Platform S3 임시 파일 삭제
      3. DB: doc 상태 = "승인", s3_path = Developer S3 경로로 업데이트
      4. Airflow DAG Trigger
             conf에 Developer S3 경로 포함

[DAG 실행 단계]
DAG이 conf에서 Developer S3 경로 파싱
  → Developer 자격증명으로 S3에서 파일 직접 조회 및 처리
  → 임베딩 완료
  → POST /api/documents/{doc_id}/status (completed / failed)

[반려 단계]
Developer 반려 클릭
  → Platform S3 임시 파일 즉시 삭제
  → DB: doc 상태 = "반려"
  → Developer S3로 복사 없음
```

---

## 플랫폼 백엔드 처리 내용

### 업로드 시
| 처리 | 내용 |
|------|------|
| 파일 저장 | Platform S3 `staging/` 경로에 임시 저장 |
| 메타데이터 | doc_id, 임시 s3_path, 파일명, 크기, 업로드 시각 DB 저장 |
| 상태 | `승인 대기` |

### 승인 시 (Case A 핵심)
| 처리 | 내용 |
|------|------|
| S3 Copy | Platform S3 → Developer S3 파일 복사 |
| 임시 파일 삭제 | Platform S3 `staging/` 파일 삭제 |
| DB 업데이트 | s3_path를 Developer S3 경로로 교체, 상태 = `승인` |
| DAG Trigger | Airflow REST API 호출, conf에 Developer S3 경로 포함 |

**S3 Copy를 위해 플랫폼이 필요한 권한**
```
Developer가 플랫폼에 사전 제공해야 하는 것:
  - Developer S3 버킷명
  - 플랫폼이 해당 버킷에 PutObject 가능한 IAM Role ARN
    (Cross-Account IAM Role 또는 Bucket Policy로 플랫폼 계정 허용)
```

---

## Trigger conf 구조

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

- `file_path`는 Developer S3 경로 (승인 시 복사 완료된 경로)
- 플랫폼 S3 임시 경로는 포함되지 않음

---

## Developer가 플랫폼에 사전 등록해야 하는 정보

| 항목 | 설명 |
|------|------|
| Developer S3 버킷명 | 파일이 복사될 목적지 버킷 |
| 저장 경로 Prefix | `agents/{agent_id}/` 등 Developer가 원하는 경로 구조 |
| Cross-Account IAM Role ARN | 플랫폼 계정이 Assume하여 Developer S3에 PutObject 가능한 Role |

**Developer S3 버킷 정책 예시 (플랫폼 계정 허용)**
```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": "arn:aws:iam::{platform_account_id}:role/PlatformS3CopyRole"
    },
    "Action": ["s3:PutObject", "s3:DeleteObject"],
    "Resource": "arn:aws:s3:::developer-bucket/agent01/*"
  }]
}
```

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
    """
    conf에서 Developer S3 경로 직접 파싱.
    승인 시 플랫폼이 이미 Developer S3로 파일을 복사한 상태.
    """
    conf = context["dag_run"].conf
    req  = conf["request_doc"]

    doc_id    = req["doc_id"]
    file_path = req["file_path"]  # Developer S3 경로 (s3://developer-bucket/...)
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

        # Developer 자신의 S3이므로 Developer 자격증명으로 접근
        service_account_name="{{ developer_sa_with_s3_role }}",  # IRSA 방식
        # 또는 env_from으로 Developer S3 자격증명 Secret 주입

        env_vars=[
            k8s.V1EnvVar(name="DOC_ID",
                value="{{ ti.xcom_pull(key='doc_id') }}"),
            k8s.V1EnvVar(name="S3_FILE_PATH",  # Developer S3 경로 직접 사용
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

### Developer가 채워야 할 항목

| 항목 | 설명 |
|------|------|
| `{{ developer_dag_id }}` | agentMarket 등록 시 입력한 DAG ID |
| `{{ developer_image }}` | 임베딩 처리 컨테이너 이미지 |
| `{{ k8s_namespace }}` | Pod 실행 네임스페이스 |
| `{{ developer_sa_with_s3_role }}` | Developer S3 접근 권한이 바인딩된 K8s ServiceAccount |
| `agentmarket_api_base_url` | Airflow Variable — 플랫폼 API URL |
| `agentmarket_platform_token` | Airflow Variable — 플랫폼 발급 토큰 |

> Platform S3 자격증명 불필요. Developer가 자신의 S3에 직접 접근.

---

## 장단점

| 구분 | 내용 |
|------|------|
| 장점 | 비승인 파일이 Developer S3에 저장되지 않음 (승인 후에만 복사) |
| 장점 | End-user 업로드 단계는 기존 플랫폼 S3 방식과 동일 — 변경 최소 |
| 장점 | 반려 시 Developer S3에 파일 잔류 없음 |
| 단점 | 플랫폼이 Developer S3에 PutObject 권한 필요 (Cross-Account IAM 설정) |
| 단점 | 승인 시 S3 Copy 지연 발생 (파일 크기에 따라 Trigger 딜레이) |
| 단점 | Developer가 IAM Role ARN을 플랫폼에 등록하는 온보딩 절차 필요 |
