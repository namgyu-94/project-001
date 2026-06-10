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

## 공통 테스트 Agent 데이터

| Agent명 | 상태 | 등록자 |
|--------|------|--------|
| TestAgent_Active_001 | 서비스 중 | dev_test01 |
| TestAgent_Pending_001 | 승인 대기 | dev_test01 |
| TestAgent_Validation_001 | Validation 대기 | dev_test01 |
| TestAgent_Suspended_001 | 일시 중단 | dev_test01 |
| TestAgent_DisposalReq_001 | 폐기 신청 | dev_test01 |
| TestAgent_A2A_001 | 서비스 중 (A2A 허용) | dev_test01 |

---

## TC-01: Admin 권한 Market 정보 화면 출력

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. 브라우저에서 agentMarket UI 접속<br>2. Admin 계정(admin_test)으로 로그인<br>3. 사이드바 내 자동 펼침 섹션 확인<br>4. `MarketPlace > AgentMarket` 메뉴 클릭<br>5. 메인 영역에 Agent 카드 목록 출력 여부 확인<br>6. 사이드바 `Admin > 승인 목록 조회` 메뉴 노출 여부 확인<br>7. 사이드바 `Developer > Agent 관리` 메뉴 노출 여부 확인<br>8. 사이드바 `My > 구독Agent` 메뉴 노출 여부 확인 |
| **사전 조건** | - Admin 계정이 시스템에 등록되어 있음<br>- 서비스 중인 Agent가 1개 이상 존재함 |
| **테스트 데이터** | - 계정: admin_test / Admin@1234 |
| **예상 결과** | - 로그인 후 Admin 섹션이 자동으로 펼쳐짐<br>- `Admin > 승인 목록 조회` 메뉴 노출됨<br>- `MarketPlace > AgentMarket` 클릭 시 Agent 카드 목록 정상 출력<br>- `Developer > Agent 관리` 메뉴 미노출<br>- `My > 구독Agent` 메뉴 미노출 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## TC-02: Developer 권한 Market 화면 출력

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. 브라우저에서 agentMarket UI 접속<br>2. Developer 계정(dev_test01)으로 로그인<br>3. 사이드바 내 자동 펼침 섹션 확인<br>4. `MarketPlace > AgentMarket` 메뉴 클릭<br>5. 메인 영역에 Agent 카드 목록 출력 여부 확인<br>6. 사이드바 `Developer > Agent 관리` 메뉴 노출 여부 확인<br>7. 사이드바 `Admin > 승인 목록 조회` 메뉴 노출 여부 확인<br>8. 사이드바 `My > 구독Agent` 메뉴 노출 여부 확인 |
| **사전 조건** | - Developer 계정이 시스템에 등록되어 있음<br>- 서비스 중인 Agent가 1개 이상 존재함 |
| **테스트 데이터** | - 계정: dev_test01 / Dev@1234 |
| **예상 결과** | - 로그인 후 Developer 섹션이 자동으로 펼쳐짐<br>- `Developer > Agent 관리` 메뉴 노출됨<br>- `MarketPlace > AgentMarket` 클릭 시 Agent 카드 목록 정상 출력<br>- `Admin > 승인 목록 조회` 메뉴 미노출<br>- `My > 구독Agent` 메뉴 미노출 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## TC-03: 일반 사용자 Market 화면 출력

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. 브라우저에서 agentMarket UI 접속<br>2. User 계정(user_test01)으로 로그인<br>3. 사이드바 내 자동 펼침 섹션 확인<br>4. `MarketPlace > AgentMarket` 메뉴 클릭<br>5. 메인 영역에 Agent 카드 목록 출력 여부 확인<br>6. 사이드바 `My > 구독Agent` 메뉴 노출 여부 확인<br>7. 사이드바 `Admin > 승인 목록 조회` 메뉴 노출 여부 확인<br>8. 사이드바 `Developer > Agent 관리` 메뉴 노출 여부 확인 |
| **사전 조건** | - 일반 사용자 계정이 시스템에 등록되어 있음<br>- 서비스 중인 Agent가 1개 이상 존재함 |
| **테스트 데이터** | - 계정: user_test01 / User@1234 |
| **예상 결과** | - 로그인 후 My 섹션이 자동으로 펼쳐짐<br>- `My > 구독Agent` 메뉴 노출됨<br>- `MarketPlace > AgentMarket` 클릭 시 Agent 카드 목록 정상 출력<br>- `Admin > 승인 목록 조회` 메뉴 미노출<br>- `Developer > Agent 관리` 메뉴 미노출 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## TC-04: 일반 사용자 Agent 구독 신청

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인<br>2. `MarketPlace > AgentMarket` 클릭<br>3. Agent 카드 목록에서 TestAgent_Active_001 확인<br>4. 해당 Agent의 구독 신청 버튼 클릭<br>5. 구독 신청 완료 메시지 확인<br>6. `My > 구독Agent` 메뉴 클릭<br>7. TestAgent_Active_001이 '승인 대기' 상태로 노출되는지 확인<br>8. 동일 Agent 구독 신청 버튼 재클릭하여 중복 신청 불가 처리 확인<br>9. Developer 계정으로 재로그인 후 `Developer > Agent 관리` 접속<br>10. user_test01의 구독 신청이 목록에 노출되는지 확인 |
| **사전 조건** | - 일반 사용자 계정 로그인 상태<br>- TestAgent_Active_001이 서비스 중 상태로 존재함<br>- user_test01의 해당 Agent 구독 이력 없음 |
| **테스트 데이터** | - 구독 신청 계정: user_test01 / User@1234<br>- 구독 대상 Agent: TestAgent_Active_001 |
| **예상 결과** | - 구독 신청 버튼 클릭 시 완료 메시지 출력<br>- `My > 구독Agent`에 TestAgent_Active_001이 '승인 대기' 상태로 노출됨<br>- 동일 Agent에 구독 신청 버튼 비활성화 또는 중복 신청 불가 안내 출력<br>- Developer `Agent 관리` 화면에 user_test01의 구독 신청 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 중복 신청 방어 로직 및 Developer 연동 확인 필수 |

---

## TC-05: 일반 사용자 Agent 구독 취소

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인<br>2. `My > 구독Agent` 클릭<br>3. 구독 중인 Agent(TestAgent_Subscribed_001) 확인<br>4. 해당 Agent의 구독 취소 버튼 클릭<br>5. 구독 취소 확인 팝업이 있는 경우 확인 버튼 클릭<br>6. 취소 완료 메시지 확인<br>7. `My > 구독Agent` 목록에서 TestAgent_Subscribed_001 제거 여부 확인 |
| **사전 조건** | - 일반 사용자 계정 로그인 상태<br>- user_test01이 TestAgent_Subscribed_001을 구독 중인 상태 |
| **테스트 데이터** | - 계정: user_test01 / User@1234<br>- 취소 대상 Agent: TestAgent_Subscribed_001 (구독 중 상태) |
| **예상 결과** | - 구독 취소 완료 메시지 출력됨<br>- `My > 구독Agent` 목록에서 해당 Agent 제거됨<br>- `MarketPlace > AgentMarket`에서 해당 Agent의 구독 버튼 다시 활성화됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 사전에 TC-04 또는 TC-22 수행하여 구독 중 상태 Agent 준비 필요 |

---

## TC-06: 일반 사용자 구독 중인 Agent 현황 조회

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인<br>2. `My > 구독Agent` 클릭<br>3. 구독 Agent 목록 출력 여부 확인<br>4. 각 Agent별 쿼리 수 수치 표시 여부 확인<br>5. 각 Agent별 토큰 사용량 수치 표시 여부 확인<br>6. 각 Agent별 응답속도 수치 표시 여부 확인<br>7. 문서 현황 항목 표시 여부 확인 |
| **사전 조건** | - 일반 사용자 계정 로그인 상태<br>- user_test01이 1개 이상의 Agent를 구독 중인 상태 |
| **테스트 데이터** | - 계정: user_test01 / User@1234<br>- 구독 중 Agent: TestAgent_Subscribed_001 |
| **예상 결과** | - `My > 구독Agent` 목록에 구독 중인 Agent 카드 정상 출력됨<br>- 각 Agent 카드에 쿼리 수, 토큰 사용량, 응답속도, 문서 현황 수치 표시됨<br>- 데이터가 없는 경우 0 또는 '-' 로 표시됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## TC-07: Admin 권한 사용자 Agent 승인 목록 조회

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. 승인 대기 Agent 목록 출력 여부 확인<br>4. TestAgent_Pending_001 항목 선택 또는 행 클릭<br>5. Agent 상세 정보(이름·보안등급·신청자·신청일 등) 표시 여부 확인 |
| **사전 조건** | - Admin 계정 로그인 상태<br>- Developer가 등록 신청한 Agent(TestAgent_Pending_001)가 승인 대기 중 |
| **테스트 데이터** | - 계정: admin_test / Admin@1234<br>- 조회 대상 Agent: TestAgent_Pending_001 (승인 대기 상태) |
| **예상 결과** | - 승인 대기 중인 Agent 목록 정상 출력됨<br>- 각 Agent 항목에 이름, 보안등급, 신청자(dev_test01), 신청일 등 상세 정보 표시됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 사전에 TC-15 수행하여 승인 대기 Agent 준비 필요 |

---

## TC-08: Admin 권한 사용자 서비스 요청 Agent 승인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. 목록에서 TestAgent_Pending_001 선택<br>4. 승인 버튼 클릭<br>5. 승인 완료 메시지 확인<br>6. TestAgent_Pending_001 상태가 'Validation 대기'로 변경되었는지 확인<br>7. Developer 계정으로 재로그인 후 `Developer > Agent 관리` 접속<br>8. TestAgent_Pending_001 상태가 'Validation 대기'로 반영되었는지 확인 |
| **사전 조건** | - Admin 계정 로그인 상태<br>- TestAgent_Pending_001이 승인 대기 중 |
| **테스트 데이터** | - Admin 계정: admin_test / Admin@1234<br>- 승인 대상 Agent: TestAgent_Pending_001 |
| **예상 결과** | - 승인 완료 메시지 출력됨<br>- TestAgent_Pending_001 상태가 'Validation 대기'로 변경됨<br>- Developer `Agent 관리` 화면에도 동일 상태 반영됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 승인 후 TC-19 (Validation 수행) 연계 테스트 가능 |

---

## TC-09: Admin 권한 사용자 서비스 요청 Agent 반려

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. 목록에서 TestAgent_Pending_002 선택<br>4. 반려 버튼 클릭<br>5. 반려 사유 입력 팝업/폼에 반려 사유 입력: "보안등급 기준 미달"<br>6. 반려 확인 버튼 클릭<br>7. 반려 완료 메시지 확인<br>8. TestAgent_Pending_002 상태가 '반려'로 변경되었는지 확인<br>9. Developer 계정으로 재로그인 후 `Developer > Agent 관리` 접속<br>10. TestAgent_Pending_002 반려 상태 및 반려 사유 반영 여부 확인 |
| **사전 조건** | - Admin 계정 로그인 상태<br>- TestAgent_Pending_002가 승인 대기 중 |
| **테스트 데이터** | - Admin 계정: admin_test / Admin@1234<br>- 반려 대상 Agent: TestAgent_Pending_002<br>- 반려 사유: "보안등급 기준 미달" |
| **예상 결과** | - 반려 완료 메시지 출력됨<br>- TestAgent_Pending_002 상태가 '반려'로 변경됨<br>- 입력한 반려 사유가 저장 및 Developer 화면에 노출됨<br>- Developer `Agent 관리`에 반려 결과 반영됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 반려 사유 입력 없이 반려 버튼 클릭 시 유효성 검증 메시지 출력 여부 추가 확인 권장 |

---

## TC-10: Admin 권한 사용자 서비스 중인 Agent 일시 중단

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. 목록에서 TestAgent_Active_001(서비스 중) 선택<br>4. 일시 중단 버튼 클릭<br>5. 일시 중단 완료 메시지 확인<br>6. TestAgent_Active_001 상태가 '일시 중단'으로 변경되었는지 확인<br>7. `MarketPlace > AgentMarket` 접속<br>8. TestAgent_Active_001이 목록에서 비노출 여부 확인 |
| **사전 조건** | - Admin 계정 로그인 상태<br>- TestAgent_Active_001이 서비스 중 상태 |
| **테스트 데이터** | - 계정: admin_test / Admin@1234<br>- 중단 대상 Agent: TestAgent_Active_001 |
| **예상 결과** | - 일시 중단 완료 메시지 출력됨<br>- TestAgent_Active_001 상태가 '일시 중단'으로 변경됨<br>- `MarketPlace > AgentMarket`에서 TestAgent_Active_001 미노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | TC-11(서비스 재개) 수행을 위해 상태를 원복할 것 |

---

## TC-11: Admin 권한 사용자 서비스 일시 중단 Agent 서비스 재개

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. 목록에서 TestAgent_Suspended_001(일시 중단) 선택<br>4. 서비스 재개 버튼 클릭<br>5. 재개 완료 메시지 확인<br>6. TestAgent_Suspended_001 상태가 '서비스 중'으로 변경되었는지 확인<br>7. `MarketPlace > AgentMarket` 접속<br>8. TestAgent_Suspended_001이 목록에 재노출되는지 확인 |
| **사전 조건** | - Admin 계정 로그인 상태<br>- TestAgent_Suspended_001이 일시 중단 상태 |
| **테스트 데이터** | - 계정: admin_test / Admin@1234<br>- 재개 대상 Agent: TestAgent_Suspended_001 |
| **예상 결과** | - 서비스 재개 완료 메시지 출력됨<br>- TestAgent_Suspended_001 상태가 '서비스 중'으로 변경됨<br>- `MarketPlace > AgentMarket`에 TestAgent_Suspended_001 재노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## TC-12: Admin 권한 사용자 폐기 신청 Agent 폐기 승인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. 목록에서 TestAgent_DisposalReq_001(폐기 신청) 선택<br>4. 폐기 승인 버튼 클릭<br>5. 폐기 승인 완료 메시지 확인<br>6. TestAgent_DisposalReq_001 상태가 '폐기 승인'으로 변경되었는지 확인<br>7. Developer 계정으로 재로그인 후 `Developer > Agent 관리` 접속<br>8. TestAgent_DisposalReq_001 폐기 승인 상태 반영 여부 확인 |
| **사전 조건** | - Admin 계정 로그인 상태<br>- TestAgent_DisposalReq_001이 폐기 신청 상태 (일시 중단 → Developer 폐기 신청 완료) |
| **테스트 데이터** | - 계정: admin_test / Admin@1234<br>- 폐기 승인 대상 Agent: TestAgent_DisposalReq_001 |
| **예상 결과** | - 폐기 승인 완료 메시지 출력됨<br>- TestAgent_DisposalReq_001 상태가 '폐기 승인'으로 변경됨<br>- Developer `Agent 관리`에 폐기 승인 상태 반영됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 사전에 TC-18 수행하여 폐기 신청 상태 Agent 준비 필요 |

---

## TC-13: Admin 권한 사용자 폐기 신청 Agent 반려

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. 목록에서 TestAgent_DisposalReq_002(폐기 신청) 선택<br>4. 반려 버튼 클릭<br>5. 반려 사유 입력: "폐기 사유 불충분"<br>6. 반려 확인 버튼 클릭<br>7. 반려 완료 메시지 확인<br>8. TestAgent_DisposalReq_002 상태가 '일시 중단'으로 복귀되었는지 확인<br>9. Developer 계정으로 재로그인 후 `Developer > Agent 관리` 접속<br>10. TestAgent_DisposalReq_002 반려 결과 및 반려 사유 반영 여부 확인 |
| **사전 조건** | - Admin 계정 로그인 상태<br>- TestAgent_DisposalReq_002가 폐기 신청 상태 |
| **테스트 데이터** | - 계정: admin_test / Admin@1234<br>- 반려 대상 Agent: TestAgent_DisposalReq_002<br>- 반려 사유: "폐기 사유 불충분" |
| **예상 결과** | - 반려 완료 메시지 출력됨<br>- TestAgent_DisposalReq_002 상태가 '일시 중단'으로 복귀됨<br>- 입력한 반려 사유가 저장되어 Developer 화면에 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 반려 후 상태가 '일시 중단'으로 정확히 복귀되는지 중점 확인 |

---

## TC-14: Admin 권한 사용자 Agent 영구 폐기

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Admin 계정(admin_test)으로 로그인<br>2. `Admin > 승인 목록 조회` 클릭<br>3. 목록에서 TestAgent_DisposalApproved_001(폐기 승인 상태) 선택<br>4. 영구 폐기 버튼 클릭<br>5. 복구 불가 확인 팝업 출력 여부 확인<br>6. 팝업에서 확인 버튼 클릭<br>7. 영구 폐기 완료 메시지 확인<br>8. 승인 목록에서 TestAgent_DisposalApproved_001 제거 여부 확인<br>9. `MarketPlace > AgentMarket`에서 해당 Agent 미노출 여부 확인 |
| **사전 조건** | - Admin 계정 로그인 상태<br>- TestAgent_DisposalApproved_001이 폐기 승인 상태 |
| **테스트 데이터** | - 계정: admin_test / Admin@1234<br>- 영구 폐기 대상 Agent: TestAgent_DisposalApproved_001 |
| **예상 결과** | - 영구 폐기 전 "복구 불가" 확인 팝업 표시됨<br>- 영구 폐기 완료 메시지 출력됨<br>- 승인 목록 및 전체 목록에서 해당 Agent 제거됨<br>- 영구 폐기 후 재복구 시도 불가 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 비가역 작업이므로 테스트 전 데이터 확인 필수. 사전에 TC-12 수행하여 폐기 승인 상태 준비 필요 |

---

## TC-15: Developer 권한 사용자 Agent 등록 신청

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. 등록 신청 버튼 클릭<br>4. Agent 이름 입력: "NewTestAgent_001"<br>5. Agent 이름 중복 확인 버튼 클릭 → 사용 가능 메시지 확인<br>6. Agent 설명 입력: "테스트용 신규 Agent"<br>7. 보안등급 선택: 일반<br>8. 공개설정 선택: 공개<br>9. 보안검토 ID 입력: "SEC-20260609-001"<br>10. A2A 허용 여부 선택: 미허용<br>11. 등록 신청 버튼 클릭<br>12. 등록 신청 완료 메시지 확인<br>13. `Developer > Agent 관리` 목록에서 NewTestAgent_001 '승인 대기' 상태 확인<br>14. Admin 계정으로 재로그인 후 `Admin > 승인 목록 조회`에 NewTestAgent_001 노출 여부 확인 |
| **사전 조건** | - Developer 계정 로그인 상태<br>- 동일한 Agent명(NewTestAgent_001)이 시스템에 존재하지 않음 |
| **테스트 데이터** | - 계정: dev_test01 / Dev@1234<br>- Agent명: NewTestAgent_001<br>- Agent 설명: 테스트용 신규 Agent<br>- 보안등급: 일반<br>- 공개설정: 공개<br>- 보안검토 ID: SEC-20260609-001<br>- A2A 허용: 미허용 |
| **예상 결과** | - 중복 확인 시 "사용 가능" 메시지 출력됨<br>- 필수 항목 누락 시 유효성 검증 메시지 출력됨<br>- 등록 신청 완료 메시지 출력됨<br>- `Developer > Agent 관리`에 'NewTestAgent_001'이 '승인 대기' 상태로 노출됨<br>- `Admin > 승인 목록 조회`에 NewTestAgent_001 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 중복된 Agent명 입력 후 중복 확인 클릭 시 오류 메시지 출력 여부 추가 확인 권장 |

---

## TC-16: Developer 권한 사용자 Agent 정보 수정/신청

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. 수정할 Agent(TestAgent_Modifiable_001) 선택<br>4. 정보 수정 버튼 또는 수정 가능 항목 활성화 확인<br>5. Agent 설명 수정: "수정된 Agent 설명 v2"<br>6. 보안등급 변경: 보안<br>7. 수정 신청 버튼 클릭<br>8. 수정 신청 완료 메시지 확인<br>9. TestAgent_Modifiable_001 상태가 '승인 대기'로 변경되었는지 확인<br>10. Admin 계정으로 재로그인 후 `Admin > 승인 목록 조회`에 수정 신청 내역 노출 여부 확인 |
| **사전 조건** | - Developer 계정 로그인 상태<br>- 수정 가능 상태의 TestAgent_Modifiable_001이 존재함 |
| **테스트 데이터** | - 계정: dev_test01 / Dev@1234<br>- 수정 대상 Agent: TestAgent_Modifiable_001<br>- 수정 설명: 수정된 Agent 설명 v2<br>- 보안등급 변경: 보안 |
| **예상 결과** | - 수정 가능 항목이 편집 가능 상태로 활성화됨<br>- 수정 신청 완료 메시지 출력됨<br>- TestAgent_Modifiable_001 상태가 '승인 대기'로 변경됨<br>- 수정 내용이 저장됨<br>- `Admin > 승인 목록 조회`에 수정 신청 내역 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | |

---

## TC-17: Developer 권한 사용자 Agent 등록 취소

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Pending_001(승인 대기 상태) 선택<br>4. 등록 취소 버튼 클릭<br>5. 취소 완료 메시지 확인<br>6. `Developer > Agent 관리` 목록에서 TestAgent_Pending_001 제거 여부 확인<br>7. Admin 계정으로 재로그인 후 `Admin > 승인 목록 조회`에서도 제거 여부 확인<br>8. 서비스 중인 TestAgent_Active_001의 등록 취소 버튼 활성화 여부 확인 (취소 불가 처리 확인) |
| **사전 조건** | - Developer 계정 로그인 상태<br>- TestAgent_Pending_001이 승인 대기 중<br>- TestAgent_Active_001이 서비스 중 상태 |
| **테스트 데이터** | - 계정: dev_test01 / Dev@1234<br>- 등록 취소 대상 Agent: TestAgent_Pending_001 (승인 대기) |
| **예상 결과** | - 취소 완료 메시지 출력됨<br>- `Developer > Agent 관리` 목록에서 TestAgent_Pending_001 제거됨<br>- `Admin > 승인 목록 조회`에서도 제거됨<br>- 서비스 중 Agent(TestAgent_Active_001)는 등록 취소 버튼 비활성화 또는 취소 불가 메시지 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 서비스 중 Agent에 대한 취소 방어 처리 확인 필수 |

---

## TC-18: Developer 권한 사용자 폐기 신청

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Suspended_001(일시 중단 상태) 선택<br>4. 폐기 신청 버튼 클릭<br>5. 폐기 사유 입력: "서비스 종료 예정"<br>6. 신청 확인 버튼 클릭<br>7. 폐기 신청 완료 메시지 확인<br>8. TestAgent_Suspended_001 상태가 '폐기 신청'으로 변경되었는지 확인<br>9. Admin 계정으로 재로그인 후 `Admin > 승인 목록 조회`에 폐기 신청 내역 노출 여부 확인<br>10. 서비스 중인 TestAgent_Active_001의 폐기 신청 버튼 활성화 여부 확인 (신청 불가 처리 확인) |
| **사전 조건** | - Developer 계정 로그인 상태<br>- TestAgent_Suspended_001이 일시 중단 상태<br>- TestAgent_Active_001이 서비스 중 상태 |
| **테스트 데이터** | - 계정: dev_test01 / Dev@1234<br>- 폐기 신청 대상 Agent: TestAgent_Suspended_001<br>- 폐기 사유: "서비스 종료 예정" |
| **예상 결과** | - 폐기 신청 완료 메시지 출력됨<br>- TestAgent_Suspended_001 상태가 '폐기 신청'으로 변경됨<br>- `Admin > 승인 목록 조회`에 폐기 신청 내역 노출됨<br>- 서비스 중 Agent는 폐기 신청 버튼 비활성화 또는 신청 불가 메시지 출력됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 서비스 중 Agent 폐기 신청 방어 처리 확인 필수 |

---

## TC-19: Developer 권한 사용자 Agent Validation 검증 및 Marketplace 등록

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Validation_001(Validation 대기 상태) 선택<br>4. Validation 버튼 클릭 → Validation 모달 열림 확인<br>5. [1단계] Router health-check 실행 버튼 클릭 → 성공 상태 표시 확인<br>6. [2단계] Gateway Key 인증 실행 버튼 클릭 → 발급된 Key로 인증 → 성공 상태 표시 확인<br>7. [3단계] A2A 허용 Agent인 경우: A2A 호환성 확인 실행 → 성공 상태 확인<br>　　　　　A2A 미허용 Agent인 경우: 3단계 skip 처리 여부 확인<br>8. 전체 단계 통과 후 Agent 등록 버튼 활성화 여부 확인<br>9. Agent 등록 버튼 클릭<br>10. 등록 완료 메시지 확인<br>11. TestAgent_Validation_001 상태가 '서비스 중'으로 변경되었는지 확인<br>12. `MarketPlace > AgentMarket`에서 TestAgent_Validation_001 노출 여부 확인 |
| **사전 조건** | - Developer 계정 로그인 상태<br>- TestAgent_Validation_001이 Admin 승인 완료된 'Validation 대기' 상태<br>- Gateway Key가 발급되어 있음 |
| **테스트 데이터** | - 계정: dev_test01 / Dev@1234<br>- Validation 대상 Agent: TestAgent_Validation_001<br>- Gateway Key: GW-TEST-KEY-20260609-001<br>- A2A 허용 여부: 미허용 (3단계 skip) |
| **예상 결과** | - Validation 모달이 정상 열림<br>- 1단계 Router health-check 성공 표시됨<br>- 2단계 Gateway Key 인증 성공 표시됨<br>- 3단계: A2A 미허용 시 skip 처리됨<br>- 전체 통과 후 Agent 등록 버튼 활성화됨<br>- 등록 완료 후 상태 '서비스 중'으로 변경됨<br>- `MarketPlace > AgentMarket`에 Agent 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | 2단계 실패 시 등록 버튼 비활성화 여부 추가 확인 권장. Gateway Key는 TC-Admin(gwkey 패널)에서 사전 발급 필요 |

---

## TC-20: Developer 권한 사용자 Agent 서비스 일시 중단

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Active_001(서비스 중 상태) 선택<br>4. 일시 중단 버튼 클릭<br>5. 중단 완료 메시지 확인<br>6. TestAgent_Active_001 상태가 '일시 중단'으로 변경되었는지 확인<br>7. `MarketPlace > AgentMarket` 접속<br>8. TestAgent_Active_001 미노출 여부 확인<br>9. Admin 계정으로 재로그인 후 `Admin > 승인 목록 조회`에서 상태 동기화 여부 확인 |
| **사전 조건** | - Developer 계정 로그인 상태<br>- TestAgent_Active_001이 서비스 중 상태 |
| **테스트 데이터** | - 계정: dev_test01 / Dev@1234<br>- 중단 대상 Agent: TestAgent_Active_001 |
| **예상 결과** | - 일시 중단 완료 메시지 출력됨<br>- TestAgent_Active_001 상태가 '일시 중단'으로 변경됨<br>- `MarketPlace > AgentMarket`에서 TestAgent_Active_001 미노출됨<br>- Admin 승인 목록에 '일시 중단' 상태 동기화됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | Admin 승인 없이 Developer가 직접 중단 처리 가능한 점 확인 |

---

## TC-21: Developer 권한 사용자 Agent 서비스 재개

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. Developer 계정(dev_test01)으로 로그인<br>2. `Developer > Agent 관리` 클릭<br>3. TestAgent_Suspended_001(일시 중단 상태) 선택<br>4. 서비스 재개 버튼 클릭<br>5. 재개 완료 메시지 확인<br>6. TestAgent_Suspended_001 상태가 '서비스 중'으로 변경되었는지 확인<br>7. `MarketPlace > AgentMarket` 접속<br>8. TestAgent_Suspended_001 재노출 여부 확인 |
| **사전 조건** | - Developer 계정 로그인 상태<br>- TestAgent_Suspended_001이 일시 중단 상태 |
| **테스트 데이터** | - 계정: dev_test01 / Dev@1234<br>- 재개 대상 Agent: TestAgent_Suspended_001 |
| **예상 결과** | - 서비스 재개 완료 메시지 출력됨<br>- TestAgent_Suspended_001 상태가 '서비스 중'으로 변경됨<br>- `MarketPlace > AgentMarket`에 TestAgent_Suspended_001 재노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | Admin 승인 없이 Developer가 직접 재개 처리 가능한 점 확인 |

---

## TC-22: Developer 권한 사용자 일반 사용자 구독 신청 승인

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test01)으로 로그인 후 TestAgent_Active_001 구독 신청 (TC-04 참조)<br>2. Developer 계정(dev_test01)으로 재로그인<br>3. `Developer > Agent 관리` 클릭<br>4. 구독 신청 목록 확인 (user_test01 신청 내역 노출 여부 확인)<br>5. user_test01의 신청 항목 선택<br>6. 승인 버튼 클릭<br>7. 승인 완료 메시지 확인<br>8. User 계정(user_test01)으로 재로그인<br>9. `My > 구독Agent`에서 TestAgent_Active_001 상태가 '구독 중'으로 변경되었는지 확인 |
| **사전 조건** | - Developer 계정 로그인 상태<br>- user_test01이 TestAgent_Active_001에 구독 신청한 상태 (승인 대기)<br>- TestAgent_Active_001이 서비스 중 상태 |
| **테스트 데이터** | - Developer 계정: dev_test01 / Dev@1234<br>- 구독 신청자: user_test01<br>- 승인 대상 Agent: TestAgent_Active_001 |
| **예상 결과** | - `Developer > Agent 관리`에 user_test01의 구독 신청 내역 노출됨<br>- 승인 완료 메시지 출력됨<br>- user_test01의 `My > 구독Agent`에서 TestAgent_Active_001 상태가 '구독 중'으로 변경됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | TC-04 수행 후 연계하여 진행 |

---

## TC-23: Developer 권한 사용자 일반 사용자 구독 신청 반려

| 항목 | 내용 |
|------|------|
| **테스트 케이스(절차)** | 1. User 계정(user_test02)으로 로그인 후 TestAgent_Active_001 구독 신청 (TC-04 참조)<br>2. Developer 계정(dev_test01)으로 재로그인<br>3. `Developer > Agent 관리` 클릭<br>4. 구독 신청 목록에서 user_test02 신청 항목 선택<br>5. 반려 버튼 클릭<br>6. 반려 사유 입력: "이용 약관 미동의"<br>7. 반려 확인 버튼 클릭<br>8. 반려 완료 메시지 확인<br>9. User 계정(user_test02)으로 재로그인<br>10. `My > 구독Agent`에서 TestAgent_Active_001 상태가 '반려'로 변경되었는지 확인<br>11. 반려 사유가 노출되는지 확인 |
| **사전 조건** | - Developer 계정 로그인 상태<br>- user_test02가 TestAgent_Active_001에 구독 신청한 상태 (승인 대기) |
| **테스트 데이터** | - Developer 계정: dev_test01 / Dev@1234<br>- 구독 신청자: user_test02 / User@5678<br>- 반려 대상 Agent: TestAgent_Active_001<br>- 반려 사유: "이용 약관 미동의" |
| **예상 결과** | - 반려 완료 메시지 출력됨<br>- user_test02의 `My > 구독Agent`에서 TestAgent_Active_001 상태가 '반려'로 변경됨<br>- 입력한 반려 사유가 저장되어 user_test02 화면에 노출됨 |
| **수행결과** | ☐ PASS　　☐ FAIL |
| **비고** | TC-22와 독립적인 User 계정(user_test02) 사용 권장 |

---

## 테스트 케이스 요약

| TC | 제목 | 역할 | 수행결과 |
|----|------|------|---------|
| TC-01 | Admin 권한 Market 정보 화면 출력 | Admin | |
| TC-02 | Developer 권한 Market 화면 출력 | Developer | |
| TC-03 | 일반 사용자 Market 화면 출력 | User | |
| TC-04 | 일반 사용자 Agent 구독 신청 | User | |
| TC-05 | 일반 사용자 Agent 구독 취소 | User | |
| TC-06 | 일반 사용자 구독 중인 Agent 현황 조회 | User | |
| TC-07 | Admin 승인 목록 조회 | Admin | |
| TC-08 | Admin Agent 등록 신청 승인 | Admin | |
| TC-09 | Admin Agent 등록 신청 반려 | Admin | |
| TC-10 | Admin 서비스 중인 Agent 일시 중단 | Admin | |
| TC-11 | Admin 일시 중단 Agent 서비스 재개 | Admin | |
| TC-12 | Admin 폐기 신청 Agent 폐기 승인 | Admin | |
| TC-13 | Admin 폐기 신청 Agent 반려 | Admin | |
| TC-14 | Admin Agent 영구 폐기 | Admin | |
| TC-15 | Developer Agent 등록 신청 | Developer | |
| TC-16 | Developer Agent 정보 수정/신청 | Developer | |
| TC-17 | Developer Agent 등록 취소 | Developer | |
| TC-18 | Developer 폐기 신청 | Developer | |
| TC-19 | Developer Validation 검증 및 Marketplace 등록 | Developer | |
| TC-20 | Developer 서비스 중인 Agent 일시 중단 | Developer | |
| TC-21 | Developer 일시 중단 Agent 서비스 재개 | Developer | |
| TC-22 | Developer 구독 신청 승인 | Developer | |
| TC-23 | Developer 구독 신청 반려 | Developer | |
