# ui-requirements_copy.html — 수정 가이드

**AgentMarket UI 프로토타입** 단일 HTML 파일.  
외부 의존성 없음 — 브라우저에서 파일을 직접 열면 즉시 실행된다.

---

## 파일 열기 / 실행

```bash
open docs/ui-requirements_copy.html        # macOS
start docs/ui-requirements_copy.html       # Windows
xdg-open docs/ui-requirements_copy.html    # Linux
```

또는 VS Code Live Server, 로컬 HTTP 서버 없이도 `file://` 경로로 동작한다.

---

## 파일 구조 한눈에 보기

```
ui-requirements_copy.html
├── <style>              CSS 전체
├── <nav id="sidebar">   왼쪽 메뉴
├── <div id="main">
│   ├── #topbar          상단 타이틀 바
│   └── #content         패널 컨테이너
│       ├── #panel-marketplace
│       ├── #panel-developer
│       ├── #panel-apply
│       ├── #panel-register
│       ├── #panel-approve
│       ├── #panel-gwkey
│       ├── #panel-milvus
│       ├── #panel-user-status
│       ├── #panel-knowledge       ← MY > Knowledge (업로드 + 내 문서 현황 통합)
│       ├── #panel-doc-approve     ← DEVELOPER > Knowledge 관리
│       └── #panel-admin-status
├── 로그인 오버레이
├── Kong 토큰 발급 모달
├── Validation 모달
└── <script>             JS 전체
```

---

## 로그인 데모 계정

파일 내 `const ACCOUNTS` 에서 관리한다.

| 이메일 | 비밀번호 | 역할 |
|--------|---------|------|
| `admin@sky.com` | `1234` | Admin |
| `dev@sky.com` | `1234` | Developer |
| `user@sky.com` | `1234` | User |

---

## 패널 구조

### 패널 목록과 접근 역할

| panel ID | 메뉴 경로 | 접근 역할 |
|----------|----------|----------|
| `marketplace` | MY > MARKETPLACE | 전체 |
| `user-status` | MY > 내 현황 | User (로그인 후 기본) |
| `knowledge` | MY > Knowledge | User · Developer |
| `developer` | DEVELOPER > 내 현황 | Developer (로그인 후 기본) |
| `doc-approve` | DEVELOPER > Knowledge 관리 | Developer |
| `apply` | DEVELOPER > 등록 신청 | Developer |
| `register` | ADMIN > Agent 등록 | Admin |
| `approve` | ADMIN > 등록 승인 | Admin (로그인 후 기본) |
| `gwkey` | ADMIN > Gateway Key | Admin |
| `milvus` | ADMIN > Milvus 컬렉션 | Admin |
| `admin-status` | ADMIN > 내 현황 | Admin |

### 패널 전환 방식

```js
show('marketplace');
// 또는 HTML에서
onclick="show('marketplace')"
```

### 상단 타이틀 수정

```js
const META = {
  marketplace: { title: 'MARKETPLACE', desc: '...', crumb: '...' },
  // 새 패널 추가 시 여기에 항목 추가
};
```

---

## Knowledge 기능 (`#panel-knowledge`)

MY > Knowledge 패널. 업로드와 내 문서 현황을 최상위 탭으로 구분한다.

```
kn-tab-upload   → #kn-section-upload   (업로드)
kn-tab-status   → #kn-section-status   (내 문서 현황)
```

탭 전환 함수: `switchUserKnowledgeTab('upload' | 'status')`

### 업로드 폼 구조

| 단계 | 내용 |
|------|------|
| ① 대상 Agent / Collection 선택 | Agent 드롭다운 (`#uploadAgentSelect`) |
| ② 파일 업로드 또는 텍스트 입력 | 타입 토글 버튼으로 전환 |
| ③ 처리 설정 | 저장 경로만 표시 (플랫폼 S3 자동 할당, 읽기 전용) |

> **주의**: Vector DB Collection 표시 필드, 처리 DAG, 임베딩 파라미터는 제거됨.  
> 처리 설정은 백엔드가 Agent 설정 기준으로 자동 처리하므로 UI에 노출하지 않는다.

업로드 타입 전환: `switchUploadType('file' | 'text')`  
업로드 실행: `simulateUpload()`  
폼 초기화: `resetUploadForm()`

### 내 문서 현황 탭

내부 탭으로 파일 문서 / 텍스트를 구분한다.

```
inner-tab: 파일 문서  → #dst-file
inner-tab: 텍스트     → #dst-text
```

탭 전환 함수: `docStatusTab(el, tabId)`

**파일 문서 탭 검색 필터**

```html
<input id="dst-file-search">   <!-- 파일명 키워드 검색 -->
<select id="dst-file-agent">   <!-- 대상 Agent 필터 -->
```

필터 함수: `filterDstFile()` — oninput / onchange 이벤트에 연결됨.

**텍스트 탭 검색 필터**

```html
<input id="dst-text-search">   <!-- 제목 키워드 검색 -->
<select id="dst-text-agent">   <!-- Agent 필터 -->
```

필터 함수: `filterDstText()`

---

## Knowledge 관리 (`#panel-doc-approve`)

DEVELOPER > Knowledge 관리 패널. 문서 승인 / 텍스트 승인 최상위 탭으로 구분한다.

```
kt-doc   → #knowledge-tab-doc    (문서 승인)
kt-text  → #knowledge-tab-text   (텍스트 승인)
```

탭 전환 함수: `switchKnowledgeTab('doc' | 'text')`

### 요약 카드

개별 Agent 카드 없이 **전체 합산(이번 달)** 수치만 표시한다.  
대기 / 승인 / 반려 카운트 + 총 용량(문자 수) 한 줄 인라인 구성.

### Agent 선택 — 커스텀 드롭다운

기본 `<select>` 대신 커스텀 드롭다운을 사용한다.  
각 옵션에 **Agent 이름 + 대기 · 승인 · 반려 카운트**를 함께 표시한다.

| 요소 ID | 설명 |
|---------|------|
| `docAgentWrap` | 문서 승인 드롭다운 래퍼 |
| `docAgentTrigger` | 트리거 버튼 (클릭 시 패널 열림) |
| `docAgentPanel` | 옵션 패널 (`.cs-panel.open` 으로 표시) |
| `textAgentWrap` | 텍스트 승인 드롭다운 래퍼 |

관련 JS 함수:

```js
toggleCs('doc' | 'text')            // 드롭다운 열기/닫기
pickDocAgent(el, value, label)      // 문서 승인 Agent 선택 → filterDocByAgent(value) 호출
pickTextAgent(el, value, label)     // 텍스트 승인 Agent 선택 → filterTextByAgent(value) 호출
```

외부 클릭 시 드롭다운이 자동으로 닫힌다 (`document.addEventListener('click', ...)`).

#### Agent 목록에 항목 추가하기

```html
<!-- #docAgentPanel 안에 추가 -->
<div class="cs-option" onclick="pickDocAgent(this,'new-agent','🆕 new-agent')">
  <span class="cs-option-name">🆕 new-agent</span>
  <span class="cs-counts">
    <span class="cs-pend">대기 0</span>
    <span class="cs-appr">승인 0</span>
    <span class="cs-rej">반려 0</span>
  </span>
</div>
```

테이블 행의 `data-agent` 속성 값이 `pickDocAgent` 두 번째 인수와 일치해야 필터가 동작한다.

### 승인 대기 / 처리 내역 검색 필터

각 탭 내부에 키워드 검색 input + Agent select + 결과 select가 있다.

**문서 승인**

| 요소 ID | 필터 대상 |
|---------|----------|
| `pend-search` | 파일명 · 사용자 키워드 |
| `hist-search` | 파일명 · 사용자 키워드 |
| `hist-result-filter` | 승인 / 반려 결과 |

필터 함수: `filterPending()`, `filterHistory()`

**텍스트 승인**

| 요소 ID | 필터 대상 |
|---------|----------|
| `text-pend-search` | 제목 · 사용자 키워드 |
| `text-hist-search` | 제목 · 사용자 키워드 |
| `text-hist-result-filter` | 승인 / 반려 결과 |

필터 함수: `filterTextPending()`, `filterTextHistory()`

### DAG 설정 (승인 대기 테이블)

**기본값 없음** — 초기 상태는 `⚠ DAG 설정 필요` 칩으로 표시된다.  
DAG이 설정되기 전까지 승인 버튼은 비활성화된다.

```
DAG 칩 클릭 → 아코디언 행 펼침 → DAG 목록 렌더 (renderDocDagAcc)
DAG 선택 (selectDocDag) → 칩이 선택된 DAG 이름으로 변경 → 승인 버튼 활성화
```

행 ID 규칙: `r0`, `r1`, ... — 새 행 추가 시 순번을 늘린다.

```html
<!-- 새 승인 대기 행 추가 시 DAG 칩 -->
<span class="dag-cell-tag dag-unset" id="dag-tag-r2"
  onclick="toggleDagDetail('r2')" title="클릭하여 DAG 설정">
  ⚠ DAG 설정 필요
</span>

<!-- 승인 버튼 — DAG 설정 전 disabled -->
<button class="btn btn-success btn-sm" id="approve-btn-r2"
  onclick="approveDoc(this)" disabled style="opacity:.4;cursor:not-allowed"
  title="DAG을 먼저 설정하세요">승인</button>

<!-- DAG 아코디언 행 -->
<tr class="dag-detail-tr" id="dag-detail-r2" style="display:none">
  <td colspan="8">
    <div class="dag-detail-inner">
      <div id="dag-acc-r2" class="dag-acc-wrap"></div>
    </div>
  </td>
</tr>
```

그리고 `toggleDagDetail` 호출 시 `renderDocDagAcc(rowId)`가 자동 실행된다.

---

## 새 패널 추가하기

### 1. HTML — 패널 div 추가

```html
<div class="panel" id="panel-mypage">
  <div class="card">
    <div class="card-title">제목</div>
    내용
  </div>
</div>
```

### 2. 사이드바 메뉴 추가

```html
<div class="nav-item" id="nav-mypage" onclick="show('mypage')">
  <span class="ni-icon">🆕</span> 새 메뉴
</div>
```

### 3. META 등록

```js
mypage: { title: '새 페이지', desc: '설명', crumb: 'agentMarket > 새 페이지' },
```

---

## Marketplace Agent 카드

```html
<div class="agent-card">
  <div class="ac-head">
    <div class="ac-icon" style="background:#fff0e8">🔬</div>
    <div>
      <div class="ac-name">공정 불량 분석 Agent</div>
      <div class="ac-team">공정개발팀 · 이공정</div>
    </div>
  </div>
  <div class="ac-desc">한 줄 설명</div>
  <div class="ac-tags">
    <span class="ac-tag">공정</span>
  </div>
  <div class="ac-meta">
    <div class="ac-meta-item"><div class="m-val">1,284</div><div class="m-lbl">구독자</div></div>
    <div class="ac-meta-item"><div class="m-val">1.2s</div><div class="m-lbl">평균 응답</div></div>
    <div class="ac-meta-item"><div class="m-val">4.8★</div><div class="m-lbl">평점</div></div>
  </div>
  <div class="ac-footer">
    <div>
      <span class="a2a-chip">A2A 지원</span>
      <span class="kb-chip">🧠 Knowledge</span>
    </div>
    <button class="btn btn-sub btn-sm">구독</button>
  </div>
</div>
```

카드 추가 위치: `<div class="agent-grid">` 안.

---

## Agent Type — Builder vs Framework

| 타입 | 설명 | Validation 절차 |
|------|------|----------------|
| **Builder** 🧱 | 플랫폼 내장 빌더 기반 | 승인 즉시 Validation 진행 |
| **Framework** ⚙️ | 외부 프레임워크 기반 (LangChain 등) | Admin이 Kong Access Token 발급 → Developer가 Agent에 적용 → Validation |

---

## Validation 흐름

```
Developer: [검증 시작] 클릭
  └─ Phase 0 (사전 입력)
       ├─ Ingress Endpoint 입력 (필수)
       ├─ Agent Type 선택: Builder / Framework
       └─ Framework 선택 시: Kong Access Token 입력 필드 표시
            ↓ [검증 진행 →] 클릭
  └─ Phase 1 (자동 실행)
       ├─ Step 1: Ingress Endpoint Health-check (GET /health)
       ├─ Step 2: Kong Token 인증 (Framework) / 건너뜀 (Builder)
       └─ Step 3: A2A 검증 (A2A 활성화 시) / 건너뜀
```

```js
openValidation('agent-name', hasA2A, agentType);
// hasA2A: true/false
// agentType: 'builder' | 'framework'
```

---

## Admin Kong 토큰 발급 흐름

```
Admin: 승인 목록에서 Framework Agent [승인] 클릭
  → 행 상태 "승인 완료" + [🔑 Kong 토큰 발급] 버튼 표시
  → 버튼 클릭 → Kong 토큰 발급 모달
  → [🔑 토큰 발급] 클릭 → 토큰 생성 + Developer 이메일 전송 (프로토타입: 화면 표시)
```

---

## 주요 CSS 클래스

### 버튼

```html
<button class="btn btn-primary">파란 주버튼</button>
<button class="btn btn-secondary">회색 보조버튼</button>
<button class="btn btn-success">초록 승인버튼</button>
<button class="btn btn-danger">빨간 반려버튼</button>
<button class="btn btn-ghost">테두리 버튼</button>
<button class="btn btn-sm">작은 버튼</button>
<button class="btn btn-xs">아주 작은 버튼</button>
```

### 뱃지 / 상태 표시

```html
<span class="badge badge-blue">파란 뱃지</span>
<span class="badge badge-green">초록 뱃지</span>
<span class="badge badge-purple">보라 뱃지</span>
<span class="badge badge-gray">회색 뱃지</span>
<span class="badge badge-orange">주황 뱃지</span>

<span class="status-pill sp-active">서비스 중</span>
<span class="status-pill sp-pending">승인 대기</span>
<span class="status-pill sp-rejected">반려</span>
<span class="status-pill sp-warning">일시 중단</span>
```

### Validation 뱃지

```html
<span class="val-badge vb-none">미검증</span>
<span class="val-badge vb-pass">✅ 검증 완료</span>
<span class="val-badge vb-fail">검증 실패</span>
<span class="val-badge vb-run">검증 중</span>
<span class="val-badge vb-token">⏳ 토큰 대기</span>
```

### DAG 칩

```html
<!-- DAG 미설정 (주황, 클릭 가능) -->
<span class="dag-cell-tag dag-unset" onclick="toggleDagDetail('r0')">⚠ DAG 설정 필요</span>

<!-- DAG 설정 완료 (파란, 클릭 시 변경 가능) -->
<span class="dag-cell-tag selectable" onclick="toggleDagDetail('r0')">
  <span id="dag-label-r0">dag_id</span>
  <span style="font-size:10px;color:#7a8fd4">✏</span>
</span>
```

### 커스텀 드롭다운 (Agent 선택)

```html
<div class="cs-wrap" id="docAgentWrap">
  <div class="cs-trigger" id="docAgentTrigger" onclick="toggleCs('doc')">
    <span id="docAgentLabel">전체</span>
    <span class="cs-arrow">▾</span>
  </div>
  <div class="cs-panel" id="docAgentPanel">
    <div class="cs-option cs-selected" onclick="pickDocAgent(this,'all','전체')">
      <span class="cs-option-name">전체</span>
      <span class="cs-counts">
        <span class="cs-pend">대기 3</span>
        <span class="cs-appr">승인 14</span>
        <span class="cs-rej">반려 3</span>
      </span>
    </div>
    <!-- 추가 옵션... -->
  </div>
</div>
```

### 카드 / 레이아웃

```html
<div class="card">...</div>
<div class="card-title">제목</div>
<div class="info-box">파란 안내 박스</div>
<div class="stat-card">숫자 통계 카드</div>
```

### 폼 요소

```html
<div class="form-grid">
  <div class="fg">
    <label>레이블</label>
    <input type="text">
  </div>
  <div class="fg full">  <!-- 전체 폭 -->
    <label>레이블</label>
    <textarea></textarea>
  </div>
</div>
```

---

## 사이드바 구조

3개 섹션 (MY / DEVELOPER / ADMIN) — 항상 펼쳐진 상태로 고정.

```html
<nav id="sidebar">
  <div class="acc-header" id="st-my">MY</div>
  <div class="acc-body" id="stnav-my">
    <div class="nav-item" onclick="show('marketplace')">🛒 MARKETPLACE</div>
    <div class="nav-item" onclick="show('knowledge')">🧠 Knowledge</div>
  </div>

  <div class="acc-header" id="st-developer">DEVELOPER</div>
  <div class="acc-body" id="stnav-developer">
    <div class="nav-item" onclick="show('doc-approve')">📁 Knowledge 관리</div>
  </div>

  <div class="acc-header" id="st-admin">ADMIN</div>
  <div class="acc-body" id="stnav-admin">...</div>
</nav>
```

---

## 자주 하는 수정

### Agent 이름 일괄 변경

에디터 전체 바꾸기(Ctrl+H)로 영문 ID와 한글 이름을 동시에 교체한다.

```
process-defect-agent  →  (새 영문 ID)
공정 불량 분석 Agent   →  (새 한글 이름)
```

커스텀 드롭다운의 `pickDocAgent` / `pickTextAgent` 인수값과 테이블 행의 `data-agent` 값도 함께 바꿔야 필터가 동작한다.

### 데모 데이터 숫자 변경

`m-val`, `ss-val`, `sc-value` 클래스 또는 커스텀 드롭다운의 `.cs-pend`, `.cs-appr`, `.cs-rej` 텍스트를 검색해 수정한다.

### 새 역할 추가

1. `const ACCOUNTS`에 계정 추가
2. `const META`에 기본 패널 지정
3. `switchRole()` 함수에 역할 매핑 추가
4. 사이드바에 해당 역할의 섹션/메뉴 추가

### 모달 새로 추가

```html
<div class="modal-overlay" id="myModal">
  <div class="modal-box">
    <button class="modal-close" onclick="document.getElementById('myModal').classList.remove('open')">✕</button>
    <div class="modal-title">제목</div>
    내용
  </div>
</div>
```

```js
document.getElementById('myModal').classList.add('open');
```

---

## 주요 JS 함수 레퍼런스

| 함수 | 설명 |
|------|------|
| `show(id)` | 패널 전환 |
| `login()` / `logout()` | 로그인/로그아웃 |
| `switchRole(role)` | 역할 전환 후 기본 패널 이동 |
| `switchUserKnowledgeTab(tab)` | Knowledge 패널 업로드/내 문서 현황 탭 전환 |
| `docStatusTab(el, tabId)` | 내 문서 현황 파일/텍스트 탭 전환 |
| `filterDstFile()` | 내 문서 현황 > 파일 문서 검색 필터 |
| `filterDstText()` | 내 문서 현황 > 텍스트 검색 필터 |
| `simulateUpload()` | 문서 업로드 시뮬레이션 |
| `resetUploadForm()` | 업로드 폼 초기화 |
| `switchUploadType(type)` | 파일/텍스트 업로드 타입 전환 |
| `switchKnowledgeTab(tab)` | Knowledge 관리 문서/텍스트 최상위 탭 전환 |
| `toggleCs(id)` | 커스텀 Agent 드롭다운 열기/닫기 |
| `pickDocAgent(el, value, label)` | 문서 승인 Agent 선택 |
| `pickTextAgent(el, value, label)` | 텍스트 승인 Agent 선택 |
| `filterDocByAgent(agent)` | 문서 승인 전체 탭 Agent 필터 |
| `filterTextByAgent(agent)` | 텍스트 승인 전체 탭 Agent 필터 |
| `filterPending()` | 문서 승인 > 승인 대기 검색 필터 |
| `filterHistory()` | 문서 승인 > 처리 내역 검색 필터 |
| `filterTextPending()` | 텍스트 승인 > 승인 대기 검색 필터 |
| `filterTextHistory()` | 텍스트 승인 > 처리 내역 검색 필터 |
| `approveDoc(btn)` / `rejectDoc(btn)` | 문서 승인/반려 |
| `approveTextDoc(btn)` / `rejectTextDoc(btn)` | 텍스트 승인/반려 |
| `toggleDagDetail(rowId)` | DAG 설정 아코디언 열기/닫기 + on-demand 렌더 |
| `renderDocDagAcc(rowId)` | DAG 아코디언 목록 렌더 (클릭 시 자동 호출) |
| `selectDocDag(rowId, dagId, idx)` | DAG 선택 → 칩 업데이트 + 승인 버튼 활성화 |
| `approveAgent(name, type, btn)` | Admin Agent 등록 승인 |
| `openKongModal(name, btn)` | Kong 토큰 발급 모달 오픈 |
| `issueKongToken()` | Kong 토큰 발급 시뮬레이션 |
| `openValidation(name, a2a, type)` | Validation 모달 오픈 |
| `devTab(el, tabId)` | Developer 내 탭 전환 |
| `applyGoStep(n)` | 등록 신청 Wizard 단계 이동 |
