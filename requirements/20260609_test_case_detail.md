# 통합 테스트 케이스 상세 명세서

**프로젝트**: sky-project-001 (agentMarket UI)
**작성일**: 2026-06-09
**원본 시나리오**: [20260608_recommand.md](20260608_recommand.md)

---

## 공통 테스트 계정

| 역할 | 계정 ID | 비밀번호 |
|------|---------|---------|
| Admin | admin_test | Admin@1234 |
| Developer | dev_test01 | Dev@1234 |
| User | user_test01 | User@1234 |
| User (보조) | user_test02 | User@5678 |

## 공통 테스트 Agent 데이터

| Agent명 | 상태 | 등록자 |
|--------|------|--------|
| TestAgent_Active_001 | 서비스 중 | dev_test01 |
| TestAgent_Pending_001 | 승인 대기 | dev_test01 |
| TestAgent_Pending_002 | 승인 대기 | dev_test01 |
| TestAgent_Validation_001 | Validation 대기 | dev_test01 |
| TestAgent_Suspended_001 | 일시 중단 | dev_test01 |
| TestAgent_DisposalReq_001 | 폐기 신청 | dev_test01 |
| TestAgent_DisposalReq_002 | 폐기 신청 | dev_test01 |
| TestAgent_DisposalApproved_001 | 폐기 승인 | dev_test01 |
| TestAgent_A2A_001 | 서비스 중 (A2A 허용) | dev_test01 |
| TestAgent_Modifiable_001 | 서비스 중 | dev_test01 |
| TestAgent_Subscribed_001 | 서비스 중 (user_test01 구독 중) | dev_test01 |

---

## 시나리오 1: Admin 권한 Market 정보 화면 출력

| 항목 | 내용 |
|------|------|
| **업무기능** | Admin 권한 사용자의 Market 접근 및 권한별 UI 출력 확인 |
| **흐름 요약** | 로그인(Admin) → 사이드바 섹션 자동 펼침 확인 → MarketPlace > AgentMarket 접근 → 메뉴 노출/미노출 검증 |
| **공통 사전 조건** | Admin 계정이 시스템에 등록되어 있음 / 서비스 중인 Agent 1개 이상 존재 |
| **공통 테스트 데이터** | 계정: admin_test / Admin@1234 |

### TC-01-01: Admin 로그인 시 Admin 섹션 자동 펼침 및 전용 메뉴 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. 브라우저에서 agentMarket UI 접속<br>2. Admin 계정(admin_test / Admin@1234)으로 로그인<br>3. 사이드바에서 Admin 섹션이 자동으로 펼쳐지는지 확인<br>4. `Admin > 승인 목록 조회` 메뉴가 노출되는지 확인 |
| **예상 결과** | - 로그인 직후 사이드바 Admin 섹션이 자동 펼침됨<br>- `Admin > 승인 목록 조회` 메뉴 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-01-02: Admin 로그인 시 Developer / My 전용 메뉴 미노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. 사이드바에서 `Developer > Agent 관리` 메뉴 노출 여부 확인<br>3. 사이드바에서 `My > 구독Agent` 메뉴 노출 여부 확인 |
| **예상 결과** | - `Developer > Agent 관리` 메뉴 미노출됨<br>- `My > 구독Agent` 메뉴 미노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-01-03: Admin 접속 시 AgentMarket Agent 카드 목록 정상 출력 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `MarketPlace > AgentMarket` 메뉴 클릭<br>3. 메인 영역에 Agent 카드 목록이 출력되는지 확인 |
| **예상 결과** | - Agent 카드 목록이 메인 영역에 정상 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 2: Developer 권한 Market 화면 출력

| 항목 | 내용 |
|------|------|
| **업무기능** | Developer 권한 사용자의 Market 접근 및 권한별 UI 출력 확인 |
| **흐름 요약** | 로그인(Developer) → 사이드바 섹션 자동 펼침 확인 → MarketPlace > AgentMarket 접근 → 메뉴 노출/미노출 검증 |
| **공통 사전 조건** | Developer 계정이 시스템에 등록되어 있음 / 서비스 중인 Agent 1개 이상 존재 |
| **공통 테스트 데이터** | 계정: dev_test01 / Dev@1234 |

### TC-02-01: Developer 로그인 시 Developer 섹션 자동 펼침 및 전용 메뉴 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. 브라우저에서 agentMarket UI 접속<br>2. Developer 계정(dev_test01 / Dev@1234)으로 로그인<br>3. 사이드바에서 Developer 섹션이 자동으로 펼쳐지는지 확인<br>4. `Developer > Agent 관리` 메뉴가 노출되는지 확인 |
| **예상 결과** | - 로그인 직후 사이드바 Developer 섹션이 자동 펼침됨<br>- `Developer > Agent 관리` 메뉴 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-02-02: Developer 로그인 시 Admin 메뉴 미노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. 사이드바에서 `Admin > 승인 목록 조회` 메뉴 노출 여부 확인<br>3. 사이드바에서 `My > 구독Agent` 메뉴 노출 여부 확인 |
| **예상 결과** | - `Admin > 승인 목록 조회` 메뉴 미노출됨<br>- `My > 구독Agent` 메뉴 미노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-02-03: Developer 접속 시 AgentMarket Agent 카드 목록 정상 출력 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `MarketPlace > AgentMarket` 메뉴 클릭<br>3. 메인 영역에 Agent 카드 목록이 출력되는지 확인 |
| **예상 결과** | - Agent 카드 목록이 메인 영역에 정상 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 3: 일반 사용자 Market 화면 출력

| 항목 | 내용 |
|------|------|
| **업무기능** | 일반 사용자의 Market 접근 및 권한별 UI 출력 확인 |
| **흐름 요약** | 로그인(User) → 사이드바 섹션 자동 펼침 확인 → MarketPlace > AgentMarket 접근 → 메뉴 노출/미노출 검증 |
| **공통 사전 조건** | 일반 사용자 계정이 시스템에 등록되어 있음 / 서비스 중인 Agent 1개 이상 존재 |
| **공통 테스트 데이터** | 계정: user_test01 / User@1234 |

### TC-03-01: User 로그인 시 My 섹션 자동 펼침 및 전용 메뉴 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. 브라우저에서 agentMarket UI 접속<br>2. User 계정(user_test01 / User@1234)으로 로그인<br>3. 사이드바에서 My 섹션이 자동으로 펼쳐지는지 확인<br>4. `My > 구독Agent` 메뉴가 노출되는지 확인 |
| **예상 결과** | - 로그인 직후 사이드바 My 섹션이 자동 펼침됨<br>- `My > 구독Agent` 메뉴 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-03-02: User 로그인 시 Admin / Developer 메뉴 미노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인<br>2. 사이드바에서 `Admin > 승인 목록 조회` 메뉴 노출 여부 확인<br>3. 사이드바에서 `Developer > Agent 관리` 메뉴 노출 여부 확인 |
| **예상 결과** | - `Admin > 승인 목록 조회` 메뉴 미노출됨<br>- `Developer > Agent 관리` 메뉴 미노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-03-03: User 접속 시 AgentMarket Agent 카드 목록 정상 출력 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인<br>2. `MarketPlace > AgentMarket` 메뉴 클릭<br>3. 메인 영역에 Agent 카드 목록이 출력되는지 확인 |
| **예상 결과** | - Agent 카드 목록이 메인 영역에 정상 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 4: 일반 사용자 Agent 구독 신청

| 항목 | 내용 |
|------|------|
| **업무기능** | 일반 사용자의 Marketplace에서 Agent 구독 신청 처리 |
| **흐름 요약** | 로그인(User) → AgentMarket 접근 → Agent 선택 → 구독 신청 → My > 구독Agent 상태 확인 → Developer 연동 확인 |
| **공통 사전 조건** | 일반 사용자 계정 로그인 상태 / TestAgent_Active_001이 서비스 중 / user_test01의 해당 Agent 구독 이력 없음 |
| **공통 테스트 데이터** | 계정: user_test01 / User@1234 / 대상 Agent: TestAgent_Active_001 |

### TC-04-01: 구독 신청 완료 및 My > 구독Agent '승인 대기' 상태 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인<br>2. `MarketPlace > AgentMarket` 클릭<br>3. TestAgent_Active_001의 구독 신청 버튼 클릭<br>4. 구독 신청 완료 메시지 확인<br>5. `My > 구독Agent` 클릭<br>6. TestAgent_Active_001이 '승인 대기' 상태로 노출되는지 확인 |
| **예상 결과** | - 구독 신청 완료 메시지 출력됨<br>- `My > 구독Agent`에 TestAgent_Active_001이 '승인 대기' 상태로 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-04-02: 동일 Agent 중복 구독 신청 불가 처리 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-04-01 수행 후 (TestAgent_Active_001 구독 신청 완료 상태)<br>2. `MarketPlace > AgentMarket`에서 TestAgent_Active_001 구독 신청 버튼 재클릭<br>3. 중복 신청 방어 처리 여부 확인 (버튼 비활성화 또는 안내 메시지) |
| **예상 결과** | - 구독 신청 버튼이 비활성화되거나 중복 신청 불가 안내 메시지가 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-04-03: Developer Agent 관리 화면에 구독 신청 내역 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-04-01 수행 후 (구독 신청 완료 상태)<br>2. Developer 계정(dev_test01)으로 재로그인<br>3. `Developer > Agent 관리` 클릭<br>4. user_test01의 TestAgent_Active_001 구독 신청 내역이 목록에 노출되는지 확인 |
| **예상 결과** | - Developer `Agent 관리` 화면에 user_test01의 구독 신청 내역이 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 5: 일반 사용자 Agent 구독 취소

| 항목 | 내용 |
|------|------|
| **업무기능** | 일반 사용자의 구독 중인 Agent 구독 취소 처리 |
| **흐름 요약** | 로그인(User) → My > 구독Agent 접근 → 구독 취소 → 목록 제거 확인 → MarketPlace 버튼 복구 확인 |
| **공통 사전 조건** | user_test01이 TestAgent_Subscribed_001을 구독 중인 상태 |
| **공통 테스트 데이터** | 계정: user_test01 / User@1234 / 취소 대상: TestAgent_Subscribed_001 |

### TC-05-01: 구독 취소 완료 메시지 출력 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인<br>2. `My > 구독Agent` 클릭<br>3. TestAgent_Subscribed_001의 구독 취소 버튼 클릭<br>4. 확인 팝업이 있는 경우 확인 버튼 클릭<br>5. 취소 완료 메시지 출력 여부 확인 |
| **예상 결과** | - 구독 취소 완료 메시지가 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 사전에 TC-04 또는 TC-22 수행하여 구독 중 상태 준비 필요 |

### TC-05-02: 취소 후 My > 구독Agent 목록 제거 및 MarketPlace 구독 버튼 재활성화 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-05-01 수행 후 (구독 취소 완료 상태)<br>2. `My > 구독Agent` 목록에서 TestAgent_Subscribed_001 제거 여부 확인<br>3. `MarketPlace > AgentMarket` 접속<br>4. TestAgent_Subscribed_001의 구독 버튼 다시 활성화 여부 확인 |
| **예상 결과** | - `My > 구독Agent` 목록에서 TestAgent_Subscribed_001이 제거됨<br>- `MarketPlace > AgentMarket`에서 해당 Agent의 구독 버튼이 다시 활성화됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 6: 일반 사용자 구독 중인 Agent 현황 조회

| 항목 | 내용 |
|------|------|
| **업무기능** | 일반 사용자의 구독 Agent 사용 현황(수치) 조회 |
| **흐름 요약** | 로그인(User) → My > 구독Agent 접근 → 목록 출력 확인 → Agent별 사용 수치 표시 확인 |
| **공통 사전 조건** | user_test01이 1개 이상의 Agent를 구독 중인 상태 |
| **공통 테스트 데이터** | 계정: user_test01 / User@1234 / 구독 중 Agent: TestAgent_Subscribed_001 |

### TC-06-01: 구독 중인 Agent 목록 정상 출력 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인<br>2. `My > 구독Agent` 클릭<br>3. 구독 Agent 목록이 정상 출력되는지 확인 |
| **예상 결과** | - `My > 구독Agent` 목록에 구독 중인 Agent 카드가 정상 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-06-02: Agent별 사용 수치(쿼리 수·토큰·응답속도·문서 현황) 표시 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인<br>2. `My > 구독Agent` 클릭<br>3. 각 Agent 카드에서 쿼리 수 수치 표시 여부 확인<br>4. 토큰 사용량 수치 표시 여부 확인<br>5. 응답속도 수치 표시 여부 확인<br>6. 문서 현황 항목 표시 여부 확인<br>7. 데이터가 없는 경우 0 또는 '-' 표시 여부 확인 |
| **예상 결과** | - 각 Agent 카드에 쿼리 수, 토큰 사용량, 응답속도, 문서 현황 수치가 표시됨<br>- 데이터 없는 항목은 0 또는 '-'로 표시됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 7: Admin 권한 사용자 Agent 승인 목록 조회

| 항목 | 내용 |
|------|------|
| **업무기능** | Admin의 Agent 등록 승인 대기 목록 조회 및 상세 정보 확인 |
| **흐름 요약** | 로그인(Admin) → Admin > 승인 목록 조회 접근 → 대기 목록 출력 확인 → Agent 상세 정보 확인 |
| **공통 사전 조건** | Admin 계정 로그인 상태 / TestAgent_Pending_001이 승인 대기 중 |
| **공통 테스트 데이터** | 계정: admin_test / Admin@1234 |

### TC-07-01: 승인 대기 Agent 목록 정상 출력 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. 승인 대기 Agent 목록이 출력되는지 확인<br>4. TestAgent_Pending_001이 목록에 포함되는지 확인 |
| **예상 결과** | - 승인 대기 중인 Agent 목록이 정상 출력됨<br>- TestAgent_Pending_001이 목록에 포함됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 사전에 TC-15 수행하여 승인 대기 Agent 준비 필요 |

### TC-07-02: Agent 상세 정보(이름·보안등급·신청자·신청일) 표시 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. 목록에서 TestAgent_Pending_001 선택 또는 행 클릭<br>4. Agent 이름 표시 여부 확인<br>5. 보안등급 표시 여부 확인<br>6. 신청자(dev_test01) 표시 여부 확인<br>7. 신청일 표시 여부 확인 |
| **예상 결과** | - Agent 이름, 보안등급, 신청자(dev_test01), 신청일 등 상세 정보가 모두 표시됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 8: Admin 권한 사용자 서비스 요청 Agent 승인

| 항목 | 내용 |
|------|------|
| **업무기능** | Admin의 서비스 요청 Agent 승인 처리 및 상태 전이 확인 |
| **흐름 요약** | 로그인(Admin) → 승인 목록 조회 → Agent 선택 → 승인 → 상태 변경 확인 → Developer 화면 동기화 확인 |
| **공통 사전 조건** | Admin 계정 로그인 상태 / TestAgent_Pending_001이 승인 대기 중 |
| **공통 테스트 데이터** | Admin 계정: admin_test / Admin@1234 / 승인 대상: TestAgent_Pending_001 |

### TC-08-01: 승인 완료 메시지 출력 및 Agent 상태 'Validation 대기' 변경 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. TestAgent_Pending_001 선택<br>4. 승인 버튼 클릭<br>5. 승인 완료 메시지 확인<br>6. TestAgent_Pending_001 상태가 'Validation 대기'로 변경되었는지 확인 |
| **예상 결과** | - 승인 완료 메시지 출력됨<br>- TestAgent_Pending_001 상태가 'Validation 대기'로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-08-02: Developer Agent 관리 화면에 'Validation 대기' 상태 동기화 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-08-01 수행 후 (Admin 승인 완료 상태)<br>2. Developer 계정(dev_test01)으로 재로그인<br>3. `Developer > Agent 관리` 클릭<br>4. TestAgent_Pending_001 상태가 'Validation 대기'로 반영되었는지 확인 |
| **예상 결과** | - Developer `Agent 관리` 화면에서도 TestAgent_Pending_001이 'Validation 대기' 상태로 표시됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 승인 후 TC-19(Validation 수행) 연계 테스트 가능 |

---

## 시나리오 9: Admin 권한 사용자 서비스 요청 Agent 반려

| 항목 | 내용 |
|------|------|
| **업무기능** | Admin의 서비스 요청 Agent 반려 처리 및 사유 저장 확인 |
| **흐름 요약** | 로그인(Admin) → 승인 목록 조회 → Agent 선택 → 반려 → 사유 입력 → 상태 변경 확인 → Developer 화면 반영 확인 |
| **공통 사전 조건** | Admin 계정 로그인 상태 / TestAgent_Pending_002가 승인 대기 중 |
| **공통 테스트 데이터** | Admin 계정: admin_test / Admin@1234 / 반려 대상: TestAgent_Pending_002 / 반려 사유: "보안등급 기준 미달" |

### TC-09-01: 반려 사유 미입력 시 유효성 검증 메시지 출력 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. TestAgent_Pending_002 선택<br>4. 반려 버튼 클릭<br>5. 반려 사유 입력 없이 확인 버튼 클릭<br>6. 유효성 검증 메시지 출력 여부 확인 |
| **예상 결과** | - 반려 사유 미입력 시 유효성 검증 메시지가 출력되며 반려 처리가 차단됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-09-02: 반려 완료 메시지 출력 및 Agent 상태 '반려' 변경 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. TestAgent_Pending_002 선택<br>4. 반려 버튼 클릭<br>5. 반려 사유 입력: "보안등급 기준 미달"<br>6. 반려 확인 버튼 클릭<br>7. 반려 완료 메시지 확인<br>8. TestAgent_Pending_002 상태가 '반려'로 변경되었는지 확인 |
| **예상 결과** | - 반려 완료 메시지 출력됨<br>- TestAgent_Pending_002 상태가 '반려'로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-09-03: Developer Agent 관리 화면에 반려 결과 및 반려 사유 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-09-02 수행 후 (반려 완료 상태)<br>2. Developer 계정(dev_test01)으로 재로그인<br>3. `Developer > Agent 관리` 클릭<br>4. TestAgent_Pending_002의 반려 상태 반영 여부 확인<br>5. 반려 사유 "보안등급 기준 미달" 노출 여부 확인 |
| **예상 결과** | - Developer 화면에 반려 상태가 반영됨<br>- 입력한 반려 사유가 저장되어 Developer 화면에 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 10: Admin 권한 사용자 서비스 중인 Agent 일시 중단

| 항목 | 내용 |
|------|------|
| **업무기능** | Admin의 서비스 중인 Agent 일시 중단 처리 및 MarketPlace 반영 확인 |
| **흐름 요약** | 로그인(Admin) → 승인 목록 조회 → 서비스 중 Agent 선택 → 일시 중단 → 상태 변경 확인 → MarketPlace 비노출 확인 |
| **공통 사전 조건** | Admin 계정 로그인 상태 / TestAgent_Active_001이 서비스 중 상태 |
| **공통 테스트 데이터** | 계정: admin_test / Admin@1234 / 중단 대상: TestAgent_Active_001 |

### TC-10-01: 일시 중단 완료 메시지 출력 및 Agent 상태 변경 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. TestAgent_Active_001(서비스 중) 선택<br>4. 일시 중단 버튼 클릭<br>5. 일시 중단 완료 메시지 확인<br>6. TestAgent_Active_001 상태가 '일시 중단'으로 변경되었는지 확인 |
| **예상 결과** | - 일시 중단 완료 메시지 출력됨<br>- TestAgent_Active_001 상태가 '일시 중단'으로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | TC-11(서비스 재개) 수행을 위해 상태를 원복할 것 |

### TC-10-02: MarketPlace > AgentMarket에서 일시 중단 Agent 비노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-10-01 수행 후 (일시 중단 완료 상태)<br>2. `MarketPlace > AgentMarket` 접속<br>3. TestAgent_Active_001이 목록에서 비노출되는지 확인 |
| **예상 결과** | - `MarketPlace > AgentMarket`에서 TestAgent_Active_001이 노출되지 않음 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 11: Admin 권한 사용자 일시 중단 Agent 서비스 재개

| 항목 | 내용 |
|------|------|
| **업무기능** | Admin의 일시 중단된 Agent 서비스 재개 처리 및 MarketPlace 재노출 확인 |
| **흐름 요약** | 로그인(Admin) → 승인 목록 조회 → 일시 중단 Agent 선택 → 서비스 재개 → 상태 변경 확인 → MarketPlace 재노출 확인 |
| **공통 사전 조건** | Admin 계정 로그인 상태 / TestAgent_Suspended_001이 일시 중단 상태 |
| **공통 테스트 데이터** | 계정: admin_test / Admin@1234 / 재개 대상: TestAgent_Suspended_001 |

### TC-11-01: 서비스 재개 완료 메시지 출력 및 Agent 상태 '서비스 중' 변경 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. TestAgent_Suspended_001(일시 중단) 선택<br>4. 서비스 재개 버튼 클릭<br>5. 재개 완료 메시지 확인<br>6. TestAgent_Suspended_001 상태가 '서비스 중'으로 변경되었는지 확인 |
| **예상 결과** | - 서비스 재개 완료 메시지 출력됨<br>- TestAgent_Suspended_001 상태가 '서비스 중'으로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-11-02: MarketPlace > AgentMarket에서 재개 Agent 재노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-11-01 수행 후 (서비스 재개 완료 상태)<br>2. `MarketPlace > AgentMarket` 접속<br>3. TestAgent_Suspended_001이 목록에 다시 노출되는지 확인 |
| **예상 결과** | - `MarketPlace > AgentMarket`에 TestAgent_Suspended_001이 재노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 12: Admin 권한 사용자 폐기 신청 Agent 폐기 승인

| 항목 | 내용 |
|------|------|
| **업무기능** | Admin의 Developer 폐기 신청 Agent 승인 처리 및 상태 전이 확인 |
| **흐름 요약** | 로그인(Admin) → 승인 목록 조회 → 폐기 신청 Agent 선택 → 폐기 승인 → 상태 변경 확인 → Developer 화면 동기화 |
| **공통 사전 조건** | Admin 계정 로그인 상태 / TestAgent_DisposalReq_001이 폐기 신청 상태 |
| **공통 테스트 데이터** | 계정: admin_test / Admin@1234 / 대상: TestAgent_DisposalReq_001 |

### TC-12-01: 폐기 승인 완료 메시지 출력 및 Agent 상태 '폐기 승인' 변경 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. TestAgent_DisposalReq_001(폐기 신청) 선택<br>4. 폐기 승인 버튼 클릭<br>5. 폐기 승인 완료 메시지 확인<br>6. TestAgent_DisposalReq_001 상태가 '폐기 승인'으로 변경되었는지 확인 |
| **예상 결과** | - 폐기 승인 완료 메시지 출력됨<br>- TestAgent_DisposalReq_001 상태가 '폐기 승인'으로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 사전에 TC-18 수행하여 폐기 신청 상태 Agent 준비 필요 |

### TC-12-02: Developer Agent 관리 화면에 폐기 승인 상태 동기화 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-12-01 수행 후 (폐기 승인 완료 상태)<br>2. Developer 계정(dev_test01)으로 재로그인<br>3. `Developer > Agent 관리` 클릭<br>4. TestAgent_DisposalReq_001 상태가 '폐기 승인'으로 반영되었는지 확인 |
| **예상 결과** | - Developer `Agent 관리` 화면에서도 '폐기 승인' 상태로 표시됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 13: Admin 권한 사용자 폐기 신청 Agent 반려

| 항목 | 내용 |
|------|------|
| **업무기능** | Admin의 Developer 폐기 신청 Agent 반려 처리 및 상태 복귀 확인 |
| **흐름 요약** | 로그인(Admin) → 승인 목록 조회 → 폐기 신청 Agent 선택 → 반려 → 사유 입력 → '일시 중단' 복귀 확인 → Developer 화면 반영 |
| **공통 사전 조건** | Admin 계정 로그인 상태 / TestAgent_DisposalReq_002가 폐기 신청 상태 |
| **공통 테스트 데이터** | 계정: admin_test / Admin@1234 / 대상: TestAgent_DisposalReq_002 / 반려 사유: "폐기 사유 불충분" |

### TC-13-01: 반려 완료 메시지 출력 및 Agent 상태 '일시 중단' 복귀 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. TestAgent_DisposalReq_002 선택<br>4. 반려 버튼 클릭<br>5. 반려 사유 입력: "폐기 사유 불충분"<br>6. 반려 확인 버튼 클릭<br>7. 반려 완료 메시지 확인<br>8. TestAgent_DisposalReq_002 상태가 '일시 중단'으로 복귀되었는지 확인 |
| **예상 결과** | - 반려 완료 메시지 출력됨<br>- TestAgent_DisposalReq_002 상태가 '일시 중단'으로 정확히 복귀됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 반려 후 상태가 '일시 중단'으로 정확히 복귀되는지 중점 확인 |

### TC-13-02: Developer Agent 관리 화면에 반려 사유 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-13-01 수행 후 (반려 완료 상태)<br>2. Developer 계정(dev_test01)으로 재로그인<br>3. `Developer > Agent 관리` 클릭<br>4. TestAgent_DisposalReq_002의 반려 결과 반영 여부 확인<br>5. 반려 사유 "폐기 사유 불충분" 노출 여부 확인 |
| **예상 결과** | - Developer 화면에 반려 결과가 반영됨<br>- 입력한 반려 사유가 저장되어 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 14: Admin 권한 사용자 Agent 영구 폐기

| 항목 | 내용 |
|------|------|
| **업무기능** | Admin의 폐기 승인 Agent 영구 폐기 처리 및 목록 제거 확인 |
| **흐름 요약** | 로그인(Admin) → 승인 목록 조회 → 폐기 승인 Agent 선택 → 영구 폐기 → 확인 팝업 → 목록 제거 확인 |
| **공통 사전 조건** | Admin 계정 로그인 상태 / TestAgent_DisposalApproved_001이 폐기 승인 상태 |
| **공통 테스트 데이터** | 계정: admin_test / Admin@1234 / 대상: TestAgent_DisposalApproved_001 |

### TC-14-01: 영구 폐기 클릭 시 복구 불가 확인 팝업 출력 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. TestAgent_DisposalApproved_001(폐기 승인 상태) 선택<br>4. 영구 폐기 버튼 클릭<br>5. "복구 불가" 확인 팝업 출력 여부 확인 |
| **예상 결과** | - 영구 폐기 버튼 클릭 시 "복구 불가" 확인 팝업이 표시됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 비가역 작업이므로 테스트 전 데이터 확인 필수 / 사전에 TC-12 수행 필요 |

### TC-14-02: 영구 폐기 완료 메시지 출력 및 승인 목록/전체 목록에서 Agent 제거 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-14-01 수행 후 확인 팝업에서 확인 버튼 클릭<br>2. 영구 폐기 완료 메시지 확인<br>3. 승인 목록에서 TestAgent_DisposalApproved_001 제거 여부 확인<br>4. `MarketPlace > AgentMarket`에서 해당 Agent 미노출 여부 확인 |
| **예상 결과** | - 영구 폐기 완료 메시지 출력됨<br>- 승인 목록 및 MarketPlace 전체 목록에서 해당 Agent 제거됨<br>- 영구 폐기 후 재복구 불가 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 15: Developer 권한 사용자 Agent 등록 신청

| 항목 | 내용 |
|------|------|
| **업무기능** | Developer의 신규 Agent 등록 신청 및 유효성 검증 처리 |
| **흐름 요약** | 로그인(Developer) → Agent 관리 접근 → 등록 신청 폼 입력 → 중복 확인 → 신청 완료 → Admin 승인 목록 노출 확인 |
| **공통 사전 조건** | Developer 계정 로그인 상태 / NewTestAgent_001이 시스템에 존재하지 않음 |
| **공통 테스트 데이터** | 계정: dev_test01 / Dev@1234 / Agent명: NewTestAgent_001 / 설명: 테스트용 신규 Agent / 보안등급: 일반 / 공개설정: 공개 / 보안검토 ID: SEC-20260609-001 / A2A 허용: 미허용 |

### TC-15-01: 필수 항목 미입력 시 유효성 검증 메시지 출력 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. 등록 신청 버튼 클릭<br>4. 필수 항목(Agent명 등) 미입력 상태로 등록 신청 버튼 클릭<br>5. 유효성 검증 메시지 출력 여부 확인 |
| **예상 결과** | - 필수 항목 미입력 시 유효성 검증 메시지가 출력되며 신청이 차단됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-15-02: 중복 Agent명 입력 시 중복 확인 오류 처리 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` → 등록 신청 버튼 클릭<br>3. 이미 존재하는 Agent명(TestAgent_Active_001) 입력<br>4. 중복 확인 버튼 클릭<br>5. 오류 메시지 출력 여부 확인 |
| **예상 결과** | - 중복된 Agent명 입력 시 "이미 사용 중인 이름" 등 오류 메시지가 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-15-03: 정상 등록 신청 완료 및 Developer 관리 화면 '승인 대기' 상태 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` → 등록 신청 버튼 클릭<br>3. Agent명: "NewTestAgent_001" 입력 후 중복 확인 → 사용 가능 메시지 확인<br>4. 나머지 필수 항목 모두 입력 (설명·보안등급·공개설정·보안검토 ID·A2A 허용)<br>5. 등록 신청 버튼 클릭<br>6. 등록 신청 완료 메시지 확인<br>7. `Developer > Agent 관리` 목록에서 NewTestAgent_001이 '승인 대기' 상태로 노출되는지 확인 |
| **예상 결과** | - 중복 확인 시 "사용 가능" 메시지 출력됨<br>- 등록 신청 완료 메시지 출력됨<br>- Developer 관리 목록에 NewTestAgent_001이 '승인 대기' 상태로 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-15-04: Admin 승인 목록 조회에 신규 신청 Agent 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-15-03 수행 후 (등록 신청 완료 상태)<br>2. Admin 계정(admin_test)으로 재로그인<br>3. `Admin > 승인 목록 조회` 클릭<br>4. NewTestAgent_001이 목록에 노출되는지 확인 |
| **예상 결과** | - `Admin > 승인 목록 조회`에 NewTestAgent_001이 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 16: Developer 권한 사용자 Agent 정보 수정/신청

| 항목 | 내용 |
|------|------|
| **업무기능** | Developer의 등록된 Agent 정보 수정 후 재신청 처리 |
| **흐름 요약** | 로그인(Developer) → Agent 관리 접근 → 수정 대상 Agent 선택 → 정보 수정 → 수정 신청 → Admin 승인 목록 반영 확인 |
| **공통 사전 조건** | Developer 계정 로그인 상태 / TestAgent_Modifiable_001이 존재함 |
| **공통 테스트 데이터** | 계정: dev_test01 / Dev@1234 / 수정 대상: TestAgent_Modifiable_001 / 수정 설명: 수정된 Agent 설명 v2 / 보안등급 변경: 보안 |

### TC-16-01: Agent 선택 시 수정 가능 항목 편집 활성화 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Modifiable_001 선택<br>4. 정보 수정 버튼 또는 수정 가능 항목이 편집 상태로 활성화되는지 확인 |
| **예상 결과** | - 수정 가능 항목(설명·보안등급 등)이 편집 가능한 상태로 활성화됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-16-02: 수정 신청 완료 및 Agent 상태 '승인 대기' 변경 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. TestAgent_Modifiable_001 선택 후 수정 활성화<br>3. Agent 설명 수정: "수정된 Agent 설명 v2"<br>4. 보안등급 변경: 보안<br>5. 수정 신청 버튼 클릭<br>6. 수정 신청 완료 메시지 확인<br>7. TestAgent_Modifiable_001 상태가 '승인 대기'로 변경되었는지 확인 |
| **예상 결과** | - 수정 신청 완료 메시지 출력됨<br>- 수정 내용이 저장됨<br>- TestAgent_Modifiable_001 상태가 '승인 대기'로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-16-03: Admin 승인 목록에 수정 신청 내역 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-16-02 수행 후 (수정 신청 완료 상태)<br>2. Admin 계정(admin_test)으로 재로그인<br>3. `Admin > 승인 목록 조회` 클릭<br>4. TestAgent_Modifiable_001 수정 신청 내역이 목록에 노출되는지 확인 |
| **예상 결과** | - `Admin > 승인 목록 조회`에 수정 신청 내역이 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 17: Developer 권한 사용자 Agent 등록 취소

| 항목 | 내용 |
|------|------|
| **업무기능** | Developer의 승인 대기 중인 Agent 등록 취소 및 서비스 중 Agent 취소 방어 처리 확인 |
| **흐름 요약** | 로그인(Developer) → Agent 관리 접근 → 승인 대기 Agent 선택 → 등록 취소 → 목록 제거 확인 → 서비스 중 Agent 취소 불가 확인 |
| **공통 사전 조건** | Developer 계정 로그인 상태 / TestAgent_Pending_001이 승인 대기 중 / TestAgent_Active_001이 서비스 중 |
| **공통 테스트 데이터** | 계정: dev_test01 / Dev@1234 / 취소 대상: TestAgent_Pending_001 |

### TC-17-01: 승인 대기 Agent 등록 취소 완료 및 Developer 관리 목록 제거 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Pending_001(승인 대기) 선택<br>4. 등록 취소 버튼 클릭<br>5. 취소 완료 메시지 확인<br>6. `Developer > Agent 관리` 목록에서 TestAgent_Pending_001 제거 여부 확인 |
| **예상 결과** | - 취소 완료 메시지 출력됨<br>- `Developer > Agent 관리` 목록에서 TestAgent_Pending_001이 제거됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-17-02: Admin 승인 목록 조회에서도 취소된 Agent 제거 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-17-01 수행 후 (등록 취소 완료 상태)<br>2. Admin 계정(admin_test)으로 재로그인<br>3. `Admin > 승인 목록 조회` 클릭<br>4. TestAgent_Pending_001이 목록에서 제거되었는지 확인 |
| **예상 결과** | - `Admin > 승인 목록 조회`에서도 TestAgent_Pending_001이 제거됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-17-03: 서비스 중 Agent 등록 취소 불가 처리 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Active_001(서비스 중) 선택<br>4. 등록 취소 버튼 활성화 여부 확인<br>5. 취소 버튼 클릭 시 취소 불가 메시지 출력 여부 확인 |
| **예상 결과** | - 서비스 중 Agent의 등록 취소 버튼이 비활성화되거나 취소 불가 메시지가 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 18: Developer 권한 사용자 폐기 신청

| 항목 | 내용 |
|------|------|
| **업무기능** | Developer의 일시 중단된 Agent 폐기 신청 및 서비스 중 Agent 폐기 신청 방어 처리 확인 |
| **흐름 요약** | 로그인(Developer) → Agent 관리 접근 → 일시 중단 Agent 선택 → 폐기 신청 → 사유 입력 → 상태 변경 확인 → Admin 노출 확인 |
| **공통 사전 조건** | Developer 계정 로그인 상태 / TestAgent_Suspended_001이 일시 중단 상태 / TestAgent_Active_001이 서비스 중 |
| **공통 테스트 데이터** | 계정: dev_test01 / Dev@1234 / 폐기 신청 대상: TestAgent_Suspended_001 / 폐기 사유: "서비스 종료 예정" |

### TC-18-01: 폐기 신청 완료 및 Agent 상태 '폐기 신청' 변경 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Suspended_001(일시 중단) 선택<br>4. 폐기 신청 버튼 클릭<br>5. 폐기 사유 입력: "서비스 종료 예정"<br>6. 신청 확인 버튼 클릭<br>7. 폐기 신청 완료 메시지 확인<br>8. TestAgent_Suspended_001 상태가 '폐기 신청'으로 변경되었는지 확인 |
| **예상 결과** | - 폐기 신청 완료 메시지 출력됨<br>- TestAgent_Suspended_001 상태가 '폐기 신청'으로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-18-02: Admin 승인 목록 조회에 폐기 신청 내역 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-18-01 수행 후 (폐기 신청 완료 상태)<br>2. Admin 계정(admin_test)으로 재로그인<br>3. `Admin > 승인 목록 조회` 클릭<br>4. TestAgent_Suspended_001의 폐기 신청 내역이 목록에 노출되는지 확인 |
| **예상 결과** | - `Admin > 승인 목록 조회`에 폐기 신청 내역이 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-18-03: 서비스 중 Agent 폐기 신청 불가 처리 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Active_001(서비스 중) 선택<br>4. 폐기 신청 버튼 활성화 여부 확인<br>5. 폐기 신청 버튼 클릭 시 신청 불가 메시지 출력 여부 확인 |
| **예상 결과** | - 서비스 중 Agent의 폐기 신청 버튼이 비활성화되거나 신청 불가 메시지가 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 19: Developer 권한 사용자 Agent Validation 검증 및 Marketplace 등록

| 항목 | 내용 |
|------|------|
| **업무기능** | Admin 승인 후 Developer의 Agent 3단계 유효성 검증 수행 및 Marketplace 최종 등록 |
| **흐름 요약** | 로그인(Developer) → Validation 대기 Agent 선택 → Validation 모달 → 3단계 검증 → 등록 버튼 활성화 → 등록 → MarketPlace 노출 확인 |
| **공통 사전 조건** | Developer 계정 로그인 상태 / TestAgent_Validation_001이 Admin 승인 완료된 'Validation 대기' 상태 / Gateway Key 발급 완료 |
| **공통 테스트 데이터** | 계정: dev_test01 / Dev@1234 / 대상: TestAgent_Validation_001 / Gateway Key: GW-TEST-KEY-20260609-001 / A2A 허용: 미허용 |

### TC-19-01: Validation 모달 열림 및 1단계 Router health-check 성공 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Validation_001 선택<br>4. Validation 버튼 클릭<br>5. Validation 모달이 열리는지 확인<br>6. [1단계] Router health-check 실행 버튼 클릭<br>7. 1단계 성공 상태 표시 여부 확인 |
| **예상 결과** | - Validation 모달이 정상적으로 열림<br>- 1단계 Router health-check 성공 상태가 표시됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-19-02: 2단계 Gateway Key 인증 성공 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Validation 모달에서 1단계 완료 후<br>2. [2단계] Gateway Key 인증 실행 버튼 클릭<br>3. 발급된 Key(GW-TEST-KEY-20260609-001)로 인증<br>4. 2단계 성공 상태 표시 여부 확인<br>5. 2단계 실패 시 등록 버튼 비활성화 여부 추가 확인 |
| **예상 결과** | - 2단계 Gateway Key 인증 성공 상태가 표시됨<br>- 실패 시 등록 버튼이 비활성화됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | Gateway Key는 Admin > gwkey 패널에서 사전 발급 필요 |

### TC-19-03: A2A 미허용 Agent 3단계 skip 처리 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Validation 모달에서 2단계 완료 후<br>2. TestAgent_Validation_001(A2A 미허용) 기준으로 3단계 처리 여부 확인<br>3. 3단계가 skip 처리되는지 확인 |
| **예상 결과** | - A2A 미허용 Agent의 경우 3단계가 skip 처리됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | A2A 허용 Agent(TestAgent_A2A_001) 기준 재테스트 시 3단계 A2A 호환성 확인 실행 및 성공 상태 표시 확인 |

### TC-19-04: 전체 단계 통과 후 Agent 등록 버튼 활성화 및 등록 완료 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Validation 전체 단계 통과 후<br>2. Agent 등록 버튼 활성화 여부 확인<br>3. Agent 등록 버튼 클릭<br>4. 등록 완료 메시지 확인<br>5. TestAgent_Validation_001 상태가 '서비스 중'으로 변경되었는지 확인 |
| **예상 결과** | - 전체 통과 후 Agent 등록 버튼이 활성화됨<br>- 등록 완료 메시지 출력됨<br>- TestAgent_Validation_001 상태가 '서비스 중'으로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-19-05: 등록 완료 후 MarketPlace > AgentMarket에 Agent 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-19-04 수행 후 (Agent 등록 완료 상태)<br>2. `MarketPlace > AgentMarket` 클릭<br>3. TestAgent_Validation_001이 카드 목록에 노출되는지 확인 |
| **예상 결과** | - `MarketPlace > AgentMarket`에 TestAgent_Validation_001이 정상 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 20: Developer 권한 사용자 Agent 서비스 일시 중단

| 항목 | 내용 |
|------|------|
| **업무기능** | Developer의 서비스 중인 Agent 직접 일시 중단 처리 (Admin 승인 없이 직접 처리) |
| **흐름 요약** | 로그인(Developer) → Agent 관리 접근 → 서비스 중 Agent 선택 → 일시 중단 → 상태 변경 확인 → MarketPlace 비노출 확인 → Admin 동기화 확인 |
| **공통 사전 조건** | Developer 계정 로그인 상태 / TestAgent_Active_001이 서비스 중 상태 |
| **공통 테스트 데이터** | 계정: dev_test01 / Dev@1234 / 중단 대상: TestAgent_Active_001 |

### TC-20-01: 일시 중단 완료 메시지 출력 및 Agent 상태 '일시 중단' 변경 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Active_001(서비스 중) 선택<br>4. 일시 중단 버튼 클릭<br>5. 중단 완료 메시지 확인<br>6. TestAgent_Active_001 상태가 '일시 중단'으로 변경되었는지 확인 |
| **예상 결과** | - 일시 중단 완료 메시지 출력됨<br>- TestAgent_Active_001 상태가 '일시 중단'으로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | Admin 승인 없이 Developer가 직접 중단 처리 가능한 점 확인 |

### TC-20-02: MarketPlace > AgentMarket에서 일시 중단 Agent 비노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-20-01 수행 후 (일시 중단 완료 상태)<br>2. `MarketPlace > AgentMarket` 접속<br>3. TestAgent_Active_001이 목록에서 비노출되는지 확인 |
| **예상 결과** | - `MarketPlace > AgentMarket`에서 TestAgent_Active_001이 노출되지 않음 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-20-03: Admin 승인 목록 조회에 '일시 중단' 상태 동기화 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-20-01 수행 후 (일시 중단 완료 상태)<br>2. Admin 계정(admin_test)으로 재로그인<br>3. `Admin > 승인 목록 조회` 클릭<br>4. TestAgent_Active_001 상태가 '일시 중단'으로 동기화되었는지 확인 |
| **예상 결과** | - Admin `승인 목록 조회`에 TestAgent_Active_001이 '일시 중단' 상태로 동기화됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 21: Developer 권한 사용자 Agent 서비스 재개

| 항목 | 내용 |
|------|------|
| **업무기능** | Developer의 일시 중단된 Agent 직접 서비스 재개 처리 (Admin 승인 없이 직접 처리) |
| **흐름 요약** | 로그인(Developer) → Agent 관리 접근 → 일시 중단 Agent 선택 → 서비스 재개 → 상태 변경 확인 → MarketPlace 재노출 확인 |
| **공통 사전 조건** | Developer 계정 로그인 상태 / TestAgent_Suspended_001이 일시 중단 상태 |
| **공통 테스트 데이터** | 계정: dev_test01 / Dev@1234 / 재개 대상: TestAgent_Suspended_001 |

### TC-21-01: 서비스 재개 완료 메시지 출력 및 Agent 상태 '서비스 중' 변경 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Suspended_001(일시 중단) 선택<br>4. 서비스 재개 버튼 클릭<br>5. 재개 완료 메시지 확인<br>6. TestAgent_Suspended_001 상태가 '서비스 중'으로 변경되었는지 확인 |
| **예상 결과** | - 서비스 재개 완료 메시지 출력됨<br>- TestAgent_Suspended_001 상태가 '서비스 중'으로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | Admin 승인 없이 Developer가 직접 재개 처리 가능한 점 확인 |

### TC-21-02: MarketPlace > AgentMarket에서 재개 Agent 재노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-21-01 수행 후 (서비스 재개 완료 상태)<br>2. `MarketPlace > AgentMarket` 접속<br>3. TestAgent_Suspended_001이 목록에 다시 노출되는지 확인 |
| **예상 결과** | - `MarketPlace > AgentMarket`에 TestAgent_Suspended_001이 재노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 22: Developer 권한 사용자 일반 사용자 구독 신청 승인

| 항목 | 내용 |
|------|------|
| **업무기능** | Developer의 일반 사용자 Agent 구독 신청 승인 처리 및 User 구독 상태 변경 확인 |
| **흐름 요약** | User 구독 신청 → 로그인(Developer) → 구독 신청 목록 확인 → 승인 → User 구독 상태 확인 |
| **공통 사전 조건** | user_test01이 TestAgent_Active_001에 구독 신청한 상태(승인 대기) / TestAgent_Active_001이 서비스 중 |
| **공통 테스트 데이터** | Developer: dev_test01 / Dev@1234 / 구독 신청자: user_test01 / 승인 대상: TestAgent_Active_001 |

### TC-22-01: Developer Agent 관리에 구독 신청 목록 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인 후 TestAgent_Active_001 구독 신청 (TC-04-01 참조)<br>2. Developer 계정(dev_test01)으로 재로그인<br>3. `Developer > Agent 관리` 클릭<br>4. user_test01의 TestAgent_Active_001 구독 신청 내역이 목록에 노출되는지 확인 |
| **예상 결과** | - `Developer > Agent 관리`에 user_test01의 구독 신청 내역이 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | TC-04 수행 후 연계 진행 |

### TC-22-02: 구독 신청 승인 완료 메시지 출력 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. user_test01의 TestAgent_Active_001 신청 항목 선택<br>4. 승인 버튼 클릭<br>5. 승인 완료 메시지 출력 여부 확인 |
| **예상 결과** | - 승인 완료 메시지가 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

### TC-22-03: User My > 구독Agent에서 '구독 중' 상태 변경 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-22-02 수행 후 (승인 완료 상태)<br>2. User 계정(user_test01)으로 재로그인<br>3. `My > 구독Agent` 클릭<br>4. TestAgent_Active_001 상태가 '구독 중'으로 변경되었는지 확인 |
| **예상 결과** | - user_test01의 `My > 구독Agent`에서 TestAgent_Active_001이 '구독 중' 상태로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 시나리오 23: Developer 권한 사용자 일반 사용자 구독 신청 반려

| 항목 | 내용 |
|------|------|
| **업무기능** | Developer의 일반 사용자 Agent 구독 신청 반려 처리 및 User 구독 상태 변경 확인 |
| **흐름 요약** | User 구독 신청 → 로그인(Developer) → 구독 신청 목록 확인 → 반려 → 반려 사유 입력 → User 구독 상태 확인 |
| **공통 사전 조건** | user_test02가 TestAgent_Active_001에 구독 신청한 상태(승인 대기) |
| **공통 테스트 데이터** | Developer: dev_test01 / Dev@1234 / 구독 신청자: user_test02 / User@5678 / 반려 대상: TestAgent_Active_001 / 반려 사유: "이용 약관 미동의" |

### TC-23-01: 반려 완료 메시지 출력 및 반려 사유 저장 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test02)으로 로그인 후 TestAgent_Active_001 구독 신청<br>2. Developer 계정(dev_test01)으로 재로그인<br>3. `Developer > Agent 관리` 클릭<br>4. user_test02의 신청 항목 선택<br>5. 반려 버튼 클릭<br>6. 반려 사유 입력: "이용 약관 미동의"<br>7. 반려 확인 버튼 클릭<br>8. 반려 완료 메시지 출력 여부 확인 |
| **예상 결과** | - 반려 완료 메시지가 출력됨<br>- 입력한 반려 사유가 저장됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | TC-22와 독립적인 User 계정(user_test02) 사용 |

### TC-23-02: User My > 구독Agent에서 '반려' 상태 및 반려 사유 노출 확인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. TC-23-01 수행 후 (반려 완료 상태)<br>2. User 계정(user_test02)으로 재로그인<br>3. `My > 구독Agent` 클릭<br>4. TestAgent_Active_001 상태가 '반려'로 변경되었는지 확인<br>5. 반려 사유 "이용 약관 미동의"가 노출되는지 확인 |
| **예상 결과** | - user_test02의 `My > 구독Agent`에서 TestAgent_Active_001이 '반려' 상태로 변경됨<br>- 입력한 반려 사유가 user_test02 화면에 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## 테스트 케이스 요약

| 시나리오 | TC | 제목 | 역할 | 수행결과 |
|---------|-----|------|------|---------|
| 시나리오 1 | TC-01-01 | Admin 로그인 시 Admin 섹션 자동 펼침 및 전용 메뉴 노출 | Admin | |
| | TC-01-02 | Admin 로그인 시 Developer/My 메뉴 미노출 | Admin | |
| | TC-01-03 | Admin 접속 시 AgentMarket 카드 목록 출력 | Admin | |
| 시나리오 2 | TC-02-01 | Developer 로그인 시 Developer 섹션 자동 펼침 및 전용 메뉴 노출 | Developer | |
| | TC-02-02 | Developer 로그인 시 Admin 메뉴 미노출 | Developer | |
| | TC-02-03 | Developer 접속 시 AgentMarket 카드 목록 출력 | Developer | |
| 시나리오 3 | TC-03-01 | User 로그인 시 My 섹션 자동 펼침 및 전용 메뉴 노출 | User | |
| | TC-03-02 | User 로그인 시 Admin/Developer 메뉴 미노출 | User | |
| | TC-03-03 | User 접속 시 AgentMarket 카드 목록 출력 | User | |
| 시나리오 4 | TC-04-01 | 구독 신청 완료 및 '승인 대기' 상태 노출 | User | |
| | TC-04-02 | 중복 구독 신청 불가 처리 | User | |
| | TC-04-03 | Developer Agent 관리에 구독 신청 노출 | User | |
| 시나리오 5 | TC-05-01 | 구독 취소 완료 메시지 출력 | User | |
| | TC-05-02 | 취소 후 목록 제거 및 MarketPlace 버튼 재활성화 | User | |
| 시나리오 6 | TC-06-01 | 구독 Agent 목록 정상 출력 | User | |
| | TC-06-02 | Agent별 사용 수치 표시 | User | |
| 시나리오 7 | TC-07-01 | 승인 대기 Agent 목록 정상 출력 | Admin | |
| | TC-07-02 | Agent 상세 정보 표시 | Admin | |
| 시나리오 8 | TC-08-01 | 승인 완료 및 'Validation 대기' 상태 변경 | Admin | |
| | TC-08-02 | Developer 화면 'Validation 대기' 상태 동기화 | Admin | |
| 시나리오 9 | TC-09-01 | 반려 사유 미입력 시 유효성 검증 메시지 | Admin | |
| | TC-09-02 | 반려 완료 및 '반려' 상태 변경 | Admin | |
| | TC-09-03 | Developer 화면 반려 사유 노출 | Admin | |
| 시나리오 10 | TC-10-01 | 일시 중단 완료 및 상태 변경 | Admin | |
| | TC-10-02 | MarketPlace에서 일시 중단 Agent 비노출 | Admin | |
| 시나리오 11 | TC-11-01 | 서비스 재개 완료 및 상태 변경 | Admin | |
| | TC-11-02 | MarketPlace에서 재개 Agent 재노출 | Admin | |
| 시나리오 12 | TC-12-01 | 폐기 승인 완료 및 '폐기 승인' 상태 변경 | Admin | |
| | TC-12-02 | Developer 화면 폐기 승인 상태 동기화 | Admin | |
| 시나리오 13 | TC-13-01 | 반려 완료 및 '일시 중단' 상태 복귀 | Admin | |
| | TC-13-02 | Developer 화면 반려 사유 노출 | Admin | |
| 시나리오 14 | TC-14-01 | 영구 폐기 클릭 시 복구 불가 팝업 출력 | Admin | |
| | TC-14-02 | 영구 폐기 완료 및 목록 제거 | Admin | |
| 시나리오 15 | TC-15-01 | 필수 항목 미입력 시 유효성 검증 | Developer | |
| | TC-15-02 | 중복 Agent명 입력 시 오류 처리 | Developer | |
| | TC-15-03 | 정상 등록 신청 완료 및 '승인 대기' 상태 | Developer | |
| | TC-15-04 | Admin 승인 목록에 신규 신청 Agent 노출 | Developer | |
| 시나리오 16 | TC-16-01 | 수정 가능 항목 편집 활성화 | Developer | |
| | TC-16-02 | 수정 신청 완료 및 '승인 대기' 상태 변경 | Developer | |
| | TC-16-03 | Admin 승인 목록에 수정 신청 노출 | Developer | |
| 시나리오 17 | TC-17-01 | 승인 대기 Agent 등록 취소 및 목록 제거 | Developer | |
| | TC-17-02 | Admin 승인 목록에서도 제거 | Developer | |
| | TC-17-03 | 서비스 중 Agent 취소 불가 처리 | Developer | |
| 시나리오 18 | TC-18-01 | 폐기 신청 완료 및 '폐기 신청' 상태 변경 | Developer | |
| | TC-18-02 | Admin 승인 목록에 폐기 신청 노출 | Developer | |
| | TC-18-03 | 서비스 중 Agent 폐기 신청 불가 처리 | Developer | |
| 시나리오 19 | TC-19-01 | Validation 모달 열림 및 1단계 health-check 성공 | Developer | |
| | TC-19-02 | 2단계 Gateway Key 인증 성공 | Developer | |
| | TC-19-03 | A2A 미허용 Agent 3단계 skip 처리 | Developer | |
| | TC-19-04 | 전체 통과 후 등록 버튼 활성화 및 등록 완료 | Developer | |
| | TC-19-05 | MarketPlace에 등록 Agent 노출 | Developer | |
| 시나리오 20 | TC-20-01 | 일시 중단 완료 및 상태 변경 | Developer | |
| | TC-20-02 | MarketPlace에서 일시 중단 Agent 비노출 | Developer | |
| | TC-20-03 | Admin 승인 목록 상태 동기화 | Developer | |
| 시나리오 21 | TC-21-01 | 서비스 재개 완료 및 상태 변경 | Developer | |
| | TC-21-02 | MarketPlace에서 재개 Agent 재노출 | Developer | |
| 시나리오 22 | TC-22-01 | Developer 관리에 구독 신청 목록 노출 | Developer | |
| | TC-22-02 | 구독 신청 승인 완료 메시지 출력 | Developer | |
| | TC-22-03 | User My > 구독Agent '구독 중' 상태 변경 | Developer | |
| 시나리오 23 | TC-23-01 | 반려 완료 및 반려 사유 저장 | Developer | |
| | TC-23-02 | User My > 구독Agent '반려' 상태 및 사유 노출 | Developer | |
