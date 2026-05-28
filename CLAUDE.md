# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

**sky-project-001** — agentMarket UI 요건 정의 프로젝트.  
현재 단계는 AL 개발 전 **UI 프로토타입 설계 단계**로, `docs/` 폴더 산출물이 핵심이다.

---

## 산출물 목록

| 파일 | 설명 |
|------|------|
| `docs/process-definition.md` | 프로세스 정의서 + Mermaid 시퀀스 다이어그램 (Git 렌더링용) |
| `docs/ui-requirements.html` | 브라우저에서 실행되는 단일 파일 UI 프로토타입 |
| `nomal_consider.txt` | 기획 초안 메모 (원본 참고용) |

---

## 시스템 개요

| 시스템 | 역할 | UI 포함 여부 |
|--------|------|-------------|
| **HPP** | 개발자 클라우드 자원 할당 · agent 개발·배포 Ops 플랫폼 | 현재 범위 제외 |
| **agentMarket** | agent 등록·관리·승인 플랫폼 | **설계 대상** |
| **agentChat** | 사용자 프롬프트 UI | 현재 범위 제외 |

---

## 역할 정의

| 역할 | 설명 |
|------|------|
| **User** | MARKETPLACE 탐색, agent 구독, 문서 업로드 |
| **Developer** | agent 등록 신청, 문서 승인, 구독 현황 조회 |
| **Admin** | agent 등록 및 승인, Gateway Key 발급, Milvus 컬렉션 관리 |

---

## UI 프로토타입 구조 (`docs/ui-requirements.html`)

단일 HTML 파일, 외부 의존성 없음. 브라우저에서 바로 열람 가능.

### 사이드바 네비게이션

아코디언 방식 3개 섹션. 헤더 클릭 시 해당 섹션이 펼쳐지고 나머지는 닫힌다.  
로그인 시 역할에 따라 해당 섹션이 자동으로 펼쳐진다.

```
▶ MY                          ← User 로그인 시 자동 펼침 (id: st-my / stnav-my)
    🛒 MARKETPLACE
    📊 내 현황
    📤 문서 업로드
    📋 내 문서 현황

▶ DEVELOPER                   ← Developer 로그인 시 자동 펼침 (id: st-developer / stnav-developer)
    📊 내 현황
    🗂 문서 승인
    📝 등록 신청

▶ ADMIN                       ← Admin 로그인 시 자동 펼침 (id: st-admin / stnav-admin)
    📊 내 현황
    ➕ Agent 등록
    ✅ 등록 승인
    🔑 Gateway Key
    🗄 Milvus 컬렉션

역할 선택: roleSelect (User / Developer / Admin) — 숨김 처리, JS 내부 호환용
```

**아코디언 관련 JS 함수**
- `sidebarTab(tab)` — 헤더 클릭 시 토글 (열려 있으면 닫힘)
- `openSidebarTab(tab)` — 지정 섹션을 강제로 열고 나머지 닫음 (로그인 시 사용)
- `switchRole(role)` — 로그인 후 역할 매핑: `user→my`, `dev→developer`, `admin→admin`

### 패널 목록 (META 오브젝트로 라우팅)

| panel ID | 접근 역할 | 설명 |
|----------|----------|------|
| `marketplace` | 전체 | Agent 카드 목록, 보안등급 필터, 구독 버튼 |
| `user-status` | User | 구독 Agent별 사용 현황 (쿼리 수·토큰·응답속도·문서 현황) |
| `doc-upload` | User·Dev | Airflow DAG + JWT + Request Body JSON으로 문서 업로드 |
| `doc-status` | User·Dev | 업로드한 문서 목록 및 VectorDB 등록 상태 확인 |
| `developer` | Developer | 등록·승인된 Agent 목록 + 구독자·토큰·응답속도 통계 |
| `doc-approve` | Developer | Agent별 사용자 문서 업로드 요청 승인/거절 |
| `apply` | Developer | Agent 등록 신청 폼 (중복확인·보안검토ID·공개설정 등) |
| `register` | Admin | Agent 등록 (Admin 직접 등록) |
| `approve` | Admin | 등록 요청 목록 검토·승인 + Validation 모달 실행 |
| `gwkey` | Admin | Gateway Key 발급 및 조회 |
| `milvus` | Admin | Milvus 컬렉션 목록 관리 |

### 역할 전환 기본 랜딩 패널

| 역할 | 기본 패널 |
|------|----------|
| User | `user-status` |
| Developer | `developer` |
| Admin | `approve` |

---

## 핵심 기능 설명

### Validation (등록 승인 후 3단계 검증)
Admin이 agent를 승인한 뒤 Validation 버튼을 누르면 모달에서 단계별 애니메이션 실행:
1. **Router health-check** — ping/응답 확인
2. **Gateway Key 인증** — 발급된 Key로 접근 인증
3. **A2A 호환성** — A2A 허용 agent만 수행 (skip 가능)

### 문서 업로드 파이프라인
```
사용자 파일 선택
  → Airflow DAG 선택 + JWT Token + Request Body JSON 입력
  → 업로드 버튼 → S3 저장 (시뮬레이션)
  → Developer 문서 승인 패널에 대기
  → 승인 → Airflow 임베딩 → VectorDB(Milvus) 등록
  → 거절 → 반려 처리
```
- Developer가 여러 agent를 보유한 경우 agent 탭으로 필터링 (`data-agent` 속성)

### A2A (Agent-to-Agent)
- agent 등록 시 A2A 허용 여부 공개설정에서 선택
- A2A 허용 agent끼리 그룹 구성 가능
- agentChat에서 A2A 그룹 선택 시 agent 간 체이닝 호출

---

## 프로세스 정의서 (`docs/process-definition.md`)

Mermaid 시퀀스 다이어그램 4개 포함:
1. Agent 등록 흐름 (개발자 → agentMarket → Admin)
2. Agent 사용 흐름 — 단일 agent
3. Agent 사용 흐름 — A2A
4. Admin Gateway Key 발급 흐름

---

## 개발 단계 구분

| 단계 | 포함 기능 |
|------|----------|
| **1단계** | agent 등록·승인, Gateway Key 발급, 전체 agent 현황 |
| **2단계** | agent validation/health check 자동화, 각 agent 모니터링 |

---

## AL Extension (향후 구현 대상)

현재는 UI 설계 단계이며, 아래는 AL 구현 시 참고용으로 유지.

**Setup**
1. VS Code + [AL Language extension](https://marketplace.visualstudio.com/items?itemName=ms-dynamics-smb.al) 설치
2. `.vscode/launch.json` — BC 서버 연결 설정 (gitignore됨)
3. `AL: Download Symbols` 로 `.alpackages/` 심볼 다운로드

**Build**

| Action | VS Code Command |
|--------|----------------|
| Package | `AL: Package` → `*.app` 생성 |
| Publish | `F5` 또는 `AL: Publish without Debugging` |

**AL 패턴 요약**
- 오브젝트 번호: 50000–99999 (per-tenant 범위)
- `Rec.Insert(true)` / `Rec.Modify(true)` — boolean이 트리거 실행 여부 제어
- `Error()` — 롤백 포함 중단, `Message()` — 정보성 출력만
