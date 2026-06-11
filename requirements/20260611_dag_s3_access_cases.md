# DAG → S3 파일 접근 방식 요건 정의

**관련 파일**
- `requirements/20260611_requirements.json` — Trigger payload 구조 참조
- `requirements/20260610_knowledge_requirements.md` — Knowledge 파이프라인 전체 흐름

---

## 개요

플랫폼 백엔드가 Developer의 Airflow DAG을 Trigger하여 문서 임베딩을 처리할 때,  
DAG이 S3의 파일에 접근하는 경로가 두 가지로 나뉜다.

- **Case A**: Airflow DAG → 플랫폼 백엔드 → S3
- **Case B**: Airflow DAG → S3 직접 접근

두 방식은 **플랫폼 백엔드가 중간에서 처리하는 정보**, **Trigger 시 conf 구조**, **DAG 템플릿** 모두 달라진다.

---

## Case A — Airflow → Backend → S3

### 개념

DAG이 직접 S3에 접근하지 않는다.  
플랫폼 백엔드가 S3 접근의 **게이트웨이** 역할을 하며, DAG은 백엔드 API를 통해서만 파일 접근 정보를 얻는다.

### Trigger 시 conf 구조

```json
{
  "dag_run_id": "agentmarket_doc_20260611_abc123",
  "logical_date": "2026-06-11T10:00:00+00:00",
  "conf": {
    "request_doc": {
      "doc_id": "doc_abc123",
      "file_name": "계약서_v3.pdf",
      "source_type": "user"
    },
    "method": "embed"
  }
}
```

- `file_path` 미포함 — S3 경로는 백엔드가 보관, DAG에 사전 노출 없음
- DAG은 `doc_id`만 받고, 파일 접근 정보는 실행 중에 백엔드 API로 조회

### 플랫폼 백엔드 처리 흐름

```
[승인 처리]
Developer 승인 클릭
  → DB: doc 상태 = "승인"
  → Airflow REST API: DAG Trigger (conf에 doc_id만 포함)
  → 응답 완료

[DAG 실행 중 — 파일 접근 요청 수신]
GET /api/documents/{doc_id}/file-access
  → 인증 토큰 검증 (DAG이 정당한 요청자인지 확인)
  → DB: 해당 doc이 "승인" 상태인지 재확인
  → S3에서 Presigned URL 생성 (유효시간: 처리 예상 시간 기준)
  → 응답: { "presigned_url": "https://s3.../...", "expires_in": 3600 }

[DAG 완료 후]
POST /api/documents/{doc_id}/status
  → DB: doc 상태 = "completed" or "failed"
  → 필요 시 S3 파일 삭제 or 보존
```

### 백엔드 책임

| 역할 | 내용 |
|------|------|
| 실시간 상태 재검증 | DAG 실행 중 doc 상태가 바뀌었으면 접근 거부 가능 |
| S3 자격증명 관리 | DAG/Pod에 AWS 자격증명 불필요. 백엔드만 S3 권한 보유 |
| 접근 감사 로그 | 파일 접근 요청 시각, DAG 식별자 기록 |
| Presigned URL 생성 | 처리 시간에 맞는 만료 시간 설정 |
| 처리 결과 수신 | `POST /status` 콜백으로 최종 상태 업데이트 |

### DAG 템플릿

```python
import requests
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.cncf.kubernetes.operators.kubernetes_pod import KubernetesPodOperator
from airflow.models import Variable
from kubernetes.client import models as k8s
from datetime import datetime, timedelta

PLATFORM_API_BASE  = Variable.get("agentmarket_api_base_url")
PLATFORM_API_TOKEN = Variable.get("agentmarket_platform_token")


def parse_and_fetch_access(**context):
    """conf에서 doc_id 파싱 → 백엔드 API로 Presigned URL 조회"""
    conf = context["dag_run"].conf
    doc_id    = conf["request_doc"]["doc_id"]
    file_name = conf["request_doc"]["file_name"]
    method    = conf.get("method", "embed")

    if not doc_id:
        raise ValueError("conf에 doc_id 없음")

    # 백엔드에서 파일 접근 URL 조회 (S3 경로 직접 노출 없음)
    resp = requests.get(
        f"{PLATFORM_API_BASE}/documents/{doc_id}/file-access",
        headers={"Authorization": f"Bearer {PLATFORM_API_TOKEN}"},
        timeout=10,
    )
    resp.raise_for_status()
    presigned_url = resp.json()["presigned_url"]

    context["ti"].xcom_push(key="doc_id",        value=doc_id)
    context["ti"].xcom_push(key="presigned_url", value=presigned_url)
    context["ti"].xcom_push(key="method",        value=method)


def report_status(status: str, **context):
    doc_id = context["ti"].xcom_pull(key="doc_id")
    requests.post(
        f"{PLATFORM_API_BASE}/documents/{doc_id}/status",
        json={"status": status},
        headers={"Authorization": f"Bearer {PLATFORM_API_TOKEN}"},
        timeout=10,
    ).raise_for_status()


def on_success(**context):
    report_status("completed", **context)

def on_failure(**context):
    report_status("failed", **context)


with DAG(
    dag_id="{{ developer_dag_id }}",
    start_date=datetime(2026, 1, 1),
    schedule_interval=None,
    catchup=False,
    on_failure_callback=on_failure,
) as dag:

    fetch = PythonOperator(
        task_id="fetch_file_access",
        python_callable=parse_and_fetch_access,
        provide_context=True,
    )

    embed = KubernetesPodOperator(
        task_id="embedding_pod",
        image="{{ developer_image }}",
        namespace="{{ k8s_namespace }}",
        env_vars=[
            k8s.V1EnvVar(name="DOC_ID",
                value="{{ ti.xcom_pull(key='doc_id') }}"),
            k8s.V1EnvVar(name="FILE_ACCESS_URL",   # Presigned URL — AWS 자격증명 불필요
                value="{{ ti.xcom_pull(key='presigned_url') }}"),
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

    fetch >> embed >> report
```

### Developer가 채워야 할 항목

| 항목 | 설명 |
|------|------|
| `{{ developer_dag_id }}` | agentMarket 등록 시 입력한 DAG ID |
| `{{ developer_image }}` | 임베딩 처리 컨테이너 이미지 |
| `{{ k8s_namespace }}` | Pod 실행 네임스페이스 |
| `agentmarket_api_base_url` | Airflow Variable — 플랫폼 API URL |
| `agentmarket_platform_token` | Airflow Variable — 플랫폼 발급 토큰 |

> AWS 자격증명 불필요 — Presigned URL로 S3 접근

---

## Case B — Airflow → S3 직접 접근

### 개념

플랫폼 백엔드가 Trigger 시점에 S3 접근에 필요한 정보를 모두 conf에 담아서 전달한다.  
DAG 실행 중 백엔드 API 호출 없이 S3에 직접 접근한다.

### Trigger 시 conf 구조

```json
{
  "dag_run_id": "agentmarket_doc_20260611_abc123",
  "logical_date": "2026-06-11T10:00:00+00:00",
  "conf": {
    "request_doc": {
      "doc_id": "doc_abc123",
      "file_path": "s3://platform-bucket/agents/agent01/users/user01/계약서_v3.pdf",
      "file_name": "계약서_v3.pdf",
      "source_type": "user",
      "presigned_url": "https://platform-bucket.s3.amazonaws.com/...?X-Amz-Expires=3600&..."
    },
    "method": "embed"
  }
}
```

- `file_path` + `presigned_url` 모두 포함
- DAG 실행 중 백엔드 호출 없음 — conf만으로 S3 접근 완결

### 플랫폼 백엔드 처리 흐름

```
[승인 처리]
Developer 승인 클릭
  → DB: doc 상태 = "승인"
  → S3 Presigned URL 생성 (유효시간 설정) ← Case A와 달리 이 시점에 미리 생성
  → Airflow REST API: DAG Trigger
      conf에 file_path + presigned_url 모두 포함
  → 응답 완료

[DAG 실행 중]
백엔드 API 호출 없음
DAG이 conf의 presigned_url로 S3에 직접 접근

[DAG 완료 후]
POST /api/documents/{doc_id}/status
  → DB: doc 상태 = "completed" or "failed"
```

### 백엔드 책임

| 역할 | 내용 |
|------|------|
| Presigned URL 사전 생성 | 승인 시점에 생성 — DAG 실행 전 만료되지 않도록 유효시간 설계 필요 |
| S3 자격증명 관리 | 백엔드만 S3 권한 보유 (Presigned URL 방식 사용 시) |
| 처리 결과 수신 | `POST /status` 콜백으로 최종 상태 업데이트 |

> 실행 중 실시간 접근 제어 없음 — Trigger 이후 취소 불가

### DAG 템플릿

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
    """conf에서 파일 접근 정보 직접 파싱 — 백엔드 API 호출 없음"""
    conf = context["dag_run"].conf
    req  = conf["request_doc"]

    doc_id        = req["doc_id"]
    presigned_url = req["presigned_url"]  # Trigger 시 백엔드가 미리 생성한 URL
    file_name     = req["file_name"]
    method        = conf.get("method", "embed")

    if not doc_id or not presigned_url:
        raise ValueError(f"conf 누락: doc_id={doc_id}, presigned_url 여부={bool(presigned_url)}")

    context["ti"].xcom_push(key="doc_id",        value=doc_id)
    context["ti"].xcom_push(key="presigned_url", value=presigned_url)
    context["ti"].xcom_push(key="method",        value=method)


def report_status(status: str, **context):
    doc_id = context["ti"].xcom_pull(key="doc_id")
    requests.post(
        f"{PLATFORM_API_BASE}/documents/{doc_id}/status",
        json={"status": status},
        headers={"Authorization": f"Bearer {PLATFORM_API_TOKEN}"},
        timeout=10,
    ).raise_for_status()


def on_success(**context):
    report_status("completed", **context)

def on_failure(**context):
    report_status("failed", **context)


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
        env_vars=[
            k8s.V1EnvVar(name="DOC_ID",
                value="{{ ti.xcom_pull(key='doc_id') }}"),
            k8s.V1EnvVar(name="FILE_ACCESS_URL",   # Presigned URL — 백엔드 경유 없이 S3 직접
                value="{{ ti.xcom_pull(key='presigned_url') }}"),
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
| `agentmarket_api_base_url` | Airflow Variable — 플랫폼 API URL |
| `agentmarket_platform_token` | Airflow Variable — 플랫폼 발급 토큰 |

> AWS 자격증명 불필요 — Presigned URL로 S3 접근 (Case A와 동일)

---

## 두 케이스 비교

| 항목 | Case A (Backend 경유) | Case B (S3 직접) |
|------|----------------------|-----------------|
| **Trigger conf** | `doc_id`만 포함 | `doc_id` + `file_path` + `presigned_url` 포함 |
| **Presigned URL 생성 시점** | DAG 실행 중 (GET API 요청 시) | 승인 시점 (Trigger 전) |
| **DAG 실행 중 백엔드 호출** | GET `/file-access` 필수 | 없음 |
| **실시간 접근 제어** | 가능 (실행 중 상태 재검증) | 불가 (Trigger 이후 취소 불가) |
| **Presigned URL 만료 리스크** | 낮음 (요청 직후 생성) | 있음 (승인 ~ DAG 실행 대기 시간 고려 필요) |
| **DAG 복잡도** | 높음 (API 호출 태스크 추가) | 낮음 (conf 파싱만) |
| **백엔드 복잡도** | 낮음 (Trigger 단순) | 높음 (Trigger 시 URL 생성 포함) |
| **DAG 템플릿 차이** | `fetch_file_access` 태스크 존재 | `parse_conf` 태스크만 존재 |
| **AWS 자격증명 (Developer)** | 불필요 | 불필요 (Presigned URL 방식 공통) |

---

## 권장 방향

| 상황 | 권장 케이스 |
|------|-----------|
| 승인 취소·긴급 중단 기능이 필요한 경우 | **Case A** |
| DAG 구현 단순화, Developer 부담 최소화 | **Case B** |
| 초기 구축 단계 (운영 복잡도 낮춤) | **Case B** |
| 보안 감사 로그, 파일 접근 이력 관리 필요 | **Case A** |

> 현재 `20260611_requirements.json` 페이로드는 `file_path`가 포함된 **Case B** 구조에 해당한다.  
> `presigned_url` 필드 추가 여부를 결정하면 Case B로 확정 가능하다.
