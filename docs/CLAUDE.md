# ui-requirements.html — 수정 가이드

**AgentMarket UI 프로토타입** 단일 HTML 파일.  
외부 의존성 없음 — 브라우저에서 파일을 직접 열면 즉시 실행된다.

---

## 파일 열기 / 실행

```bash
open docs/ui-requirements.html        # macOS
start docs/ui-requirements.html       # Windows
xdg-open docs/ui-requirements.html    # Linux
```

또는 VS Code Live Server, 로컬 HTTP 서버 없이도 `file://` 경로로 동작한다.

---

## 파일 구조 한눈에 보기

```
ui-requirements.html
├── <style>           (line ~1–660)   CSS 전체
├── <nav id="sidebar"> (line ~668)   왼쪽 메뉴
├── <div id="main">   (line ~749)
│   ├── #topbar                      상단 타이틀 바
│   └── #content                     패널 컨테이너
│       ├── #panel-marketplace
│       ├── #panel-developer
│       ├── #panel-apply
│       ├── #panel-register
│       ├── #panel-approve
│       ├── #panel-gwkey
│       ├── #panel-milvus
│       ├── #panel-user-status
│       ├── #panel-doc-upload
│       ├── #panel-doc-status
│       ├── #panel-doc-approve
│       └── #panel-admin-status
├── 로그인 오버레이   (line ~2778)
├── Kong 토큰 발급 모달 (line ~2820)
├── Validation 모달  (line ~2868)
└── <script>          (line ~3015)   JS 전체
```

---

## 로그인 데모 계정

파일 내 `const ACCOUNTS` (line ~3022) 에서 관리한다.

| 이메일 | 비밀번호 | 역할 |
|--------|---------|------|
| `admin@sky.com` | `1234` | Admin |
| `dev@sky.com` | `1234` | Developer |
| `user@sky.com` | `1234` | User |

**계정 추가/변경**
```js
// line ~3022 ACCOUNTS 객체에 추가
const ACCOUNTS = {
  'admin@sky.com': { password: '1234', name: '관리자', role: 'admin' },
  'dev2@sky.com':  { password: '5678', name: '박설계', role: 'dev'  },  // ← 추가
  ...
};
```

---

## 패널 구조

### 패널 목록과 접근 역할

| panel ID | 메뉴 경로 | 접근 역할 |
|----------|----------|----------|
| `marketplace` | MY > MARKETPLACE | 전체 |
| `user-status` | MY > 내 현황 | User (로그인 후 기본) |
| `doc-upload` | MY > 문서 업로드 | User · Developer |
| `doc-status` | MY > 내 문서 현황 | User · Developer |
| `developer` | DEVELOPER > 내 현황 | Developer (로그인 후 기본) |
| `doc-approve` | DEVELOPER > 문서 승인 | Developer |
| `apply` | DEVELOPER > 등록 신청 | Developer |
| `register` | ADMIN > Agent 등록 | Admin |
| `approve` | ADMIN > 등록 승인 | Admin (로그인 후 기본) |
| `gwkey` | ADMIN > Gateway Key | Admin |
| `milvus` | ADMIN > Milvus 컬렉션 | Admin |
| `admin-status` | ADMIN > 내 현황 | Admin |

### 패널 전환 방식

```js
show('marketplace');   // JS에서 호출
// 또는 HTML에서
onclick="show('marketplace')"
```

`show()` 함수는 line ~3082. 호출 시 상단 타이틀·설명도 자동 변경된다.

### 상단 타이틀 수정

패널 전환 시 표시되는 타이틀·설명은 `const META` (line ~3067) 에서 관리한다.

```js
const META = {
  marketplace: { title: 'MARKETPLACE', desc: '...', crumb: '...' },
  // 새 패널 추가 시 여기에 항목 추가
};
```

---

## 새 패널 추가하기

### 1. HTML — 패널 div 추가

```html
<!-- #content 안, 다른 패널 옆에 추가 -->
<div class="panel" id="panel-mypage">
  <div class="card">
    <div class="card-title">제목</div>
    내용
  </div>
</div>
```

### 2. 사이드바 메뉴 추가

```html
<!-- nav > .acc-body 안 원하는 섹션에 추가 -->
<div class="nav-item" id="nav-mypage" onclick="show('mypage')">
  <span class="ni-icon">🆕</span> 새 메뉴
</div>
```

### 3. META 등록

```js
// const META 객체에 추가
mypage: { title: '새 페이지', desc: '설명', crumb: 'agentMarket > 새 페이지' },
```

---

## Marketplace Agent 카드

### 카드 1개 구조

```html
<div class="agent-card">
  <div class="ac-head">
    <div class="ac-icon" style="background:#fff0e8">🔬</div>   <!-- 아이콘 색·이모지 -->
    <div>
      <div class="ac-name">공정 불량 분석 Agent</div>           <!-- Agent 이름 -->
      <div class="ac-team">공정개발팀 · 이공정</div>            <!-- 팀·담당자 -->
    </div>
  </div>
  <div class="ac-desc">한 줄 설명</div>
  <div class="ac-tags">
    <span class="ac-tag">공정</span>
    <span class="ac-tag">불량</span>
  </div>
  <div class="ac-meta">
    <div class="ac-meta-item"><div class="m-val">1,284</div><div class="m-lbl">구독자</div></div>
    <div class="ac-meta-item"><div class="m-val">1.2s</div><div class="m-lbl">평균 응답</div></div>
    <div class="ac-meta-item"><div class="m-val">4.8★</div><div class="m-lbl">평점</div></div>
  </div>
  <div class="ac-footer">
    <div>
      <span class="a2a-chip">A2A 지원</span>      <!-- A2A 지원 시 표시 -->
      <span class="kb-chip">🧠 Knowledge</span>   <!-- Knowledge 지원 시 표시 -->
    </div>
    <button class="btn btn-sub btn-sm">구독</button>
  </div>
</div>
```

### 카드 추가 위치

`<div class="agent-grid">` (line ~766) 안에 추가한다.

### 카테고리 필터 칩 추가

```html
<!-- .cat-bar 안에 추가 -->
<div class="cat-chip">새 카테고리</div>
```

---

## Agent Type — Builder vs Framework

등록 신청 시 Agent 구현 방식에 따라 선택한다.

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

### Validation 모달 열기

```js
openValidation('agent-name', hasA2A, agentType);
// hasA2A: true/false
// agentType: 'builder' | 'framework'

// 예시
openValidation('process-defect-agent', true, 'framework');
```

---

## Admin Kong 토큰 발급 흐름

```
Admin: 승인 목록에서 Framework Agent [승인] 클릭
  → 행 상태가 "승인 완료"로 바뀌고 [🔑 Kong 토큰 발급] 버튼 표시
  → 버튼 클릭 → Kong 토큰 발급 모달 오픈
  → [🔑 토큰 발급] 클릭 → 토큰 생성 + 발급 완료 화면 표시
  → Developer 이메일로 자동 전송 (프로토타입에서는 화면 표시만)
```

관련 JS 함수: `approveAgent()`, `openKongModal()`, `issueKongToken()` (line ~3351)

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

### Validation 뱃지 (테이블 셀)

```html
<span class="val-badge vb-none">미검증</span>
<span class="val-badge vb-pass">✅ 검증 완료</span>
<span class="val-badge vb-fail">검증 실패</span>
<span class="val-badge vb-run">검증 중</span>
<span class="val-badge vb-token">⏳ 토큰 대기</span>
```

### Agent Type 뱃지

```html
<span class="badge" style="background:#e8f4ff;color:#2471a3;font-size:11px">⚙️ Framework</span>
<span class="badge" style="background:#f0e8ff;color:#7d3c98;font-size:11px">🧱 Builder</span>
```

### 카드 / 레이아웃

```html
<div class="card">...</div>                          <!-- 기본 흰 카드 -->
<div class="card-title">제목</div>                   <!-- 카드 안 타이틀 -->
<div class="info-box">안내 텍스트</div>              <!-- 파란 안내 박스 -->
<div class="stat-card">...</div>                     <!-- 숫자 통계 카드 -->
```

### 폼 요소

```html
<div class="form-grid">
  <div class="fg">           <!-- 1칸 -->
    <label>레이블</label>
    <input type="text">
  </div>
  <div class="fg full">      <!-- 2칸 전체 폭 -->
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
    <div class="nav-item" id="nav-marketplace" onclick="show('marketplace')">
      <span class="ni-icon">🛒</span> MARKETPLACE
    </div>
    <!-- 메뉴 항목 추가 위치 -->
  </div>

  <div class="acc-header" id="st-developer">DEVELOPER</div>
  <div class="acc-body" id="stnav-developer">...</div>

  <div class="acc-header" id="st-admin">ADMIN</div>
  <div class="acc-body" id="stnav-admin">...</div>
</nav>
```

---

## 자주 하는 수정

### Agent 이름 일괄 변경

에디터의 전체 바꾸기(Ctrl+H)로 영문 ID와 한글 이름을 동시에 교체한다.

```
process-defect-agent  →  (새 영문 ID)
공정 불량 분석 Agent   →  (새 한글 이름)
```

### 데모 데이터 숫자 변경

구독자 수, Token 소비량, 응답속도 등의 숫자는 각 패널의 테이블/카드에 하드코딩되어 있다.  
`m-val`, `ss-val`, `sc-value` 클래스를 가진 요소를 검색해 수정한다.

### 새 역할 추가

1. `const ACCOUNTS`에 계정 추가 (role 값 지정)
2. `const META`에 기본 패널 지정 (필요 시)
3. `switchRole()` 함수 (line ~3106) 에 역할 매핑 추가
4. 사이드바에 해당 역할의 섹션/메뉴 추가

### 모달 새로 추가

```html
<!-- </body> 직전에 추가 -->
<div class="modal-overlay" id="myModal">
  <div class="modal-box">
    <button class="modal-close" onclick="document.getElementById('myModal').classList.remove('open')">✕</button>
    <div class="modal-title">제목</div>
    내용
  </div>
</div>
```

```js
// 열기
document.getElementById('myModal').classList.add('open');
```

---

## 주요 JS 함수 레퍼런스

| 함수 | 위치(line) | 설명 |
|------|-----------|------|
| `show(id)` | ~3082 | 패널 전환 |
| `login()` / `logout()` | ~3032 | 로그인/로그아웃 |
| `switchRole(role)` | ~3106 | 역할 전환 후 기본 패널 이동 |
| `openValidation(name, a2a, type)` | ~3457 | Validation 모달 오픈 |
| `proceedToValidation()` | ~3394 | Phase 0 → Phase 1 전환 |
| `runValidation()` | ~3500 | Validation 단계 자동 실행 |
| `approveAgent(name, type, btn)` | ~3351 | Admin 승인 버튼 처리 |
| `openKongModal(name, btn)` | ~3362 | Kong 토큰 발급 모달 오픈 |
| `issueKongToken()` | ~3375 | Kong 토큰 발급 시뮬레이션 |
| `selectValType(type, el)` | ~3387 | Validation 모달 내 타입 선택 |
| `simulateUpload()` | ~3217 | 문서 업로드 시뮬레이션 |
| `approveDoc(btn)` / `rejectDoc(btn)` | ~3280 | 문서 승인/반려 |
| `devTab(el, tabId)` | ~3116 | Developer 내 탭 전환 |
| `applyGoStep(n)` | ~3655 | 등록 신청 Wizard 단계 이동 |
