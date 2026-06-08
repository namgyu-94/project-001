# 테스트 시나리오 목록

---

## 1. Admin 권한 Market 정보 화면 출력

- **업무기능**: Admin 권한 사용자의 Market 접근 및 화면 출력
- **흐름 요약**: 로그인(Admin) → MarketPlace > AgentMarket 접근 → Agent 목록 출력 확인 → Admin > 승인 목록 조회 메뉴 활성화 확인
- **검증포인트**: Admin 전용 메뉴(승인 목록 조회) 표시 여부 / Developer·My 전용 메뉴 미노출 여부 / AgentMarket Agent 목록 정상 출력 여부
- **사전 요구사항**: Admin 계정 로그인 상태

---

## 2. Developer 권한 Market 화면 출력

- **업무기능**: Developer 권한 사용자의 Market 접근 및 화면 출력
- **흐름 요약**: 로그인(Developer) → MarketPlace > AgentMarket 접근 → Agent 목록 출력 확인 → Developer > Agent 관리 메뉴 활성화 확인
- **검증포인트**: Developer 전용 메뉴(Agent 관리) 표시 여부 / Admin 전용 메뉴 미노출 여부 / AgentMarket Agent 목록 정상 출력 여부
- **사전 요구사항**: Developer 계정 로그인 상태

---

## 3. 일반 사용자 Market 화면 출력

- **업무기능**: 일반 사용자의 Market 접근 및 화면 출력
- **흐름 요약**: 로그인(User) → MarketPlace > AgentMarket 접근 → Agent 목록 출력 확인 → My > 구독Agent 메뉴 활성화 확인
- **검증포인트**: My 전용 메뉴(구독Agent) 표시 여부 / Admin·Developer 전용 메뉴 미노출 여부 / AgentMarket Agent 목록 정상 출력 여부
- **사전 요구사항**: 일반 사용자 계정 로그인 상태

---

## 4. 일반 사용자 Agent 구독 신청

- **업무기능**: 일반 사용자의 Marketplace에서 Agent 구독 신청
- **흐름 요약**: 로그인(User) → MarketPlace > AgentMarket 접근 → Agent 선택 → 구독 신청 버튼 클릭 → 구독 완료 확인 → My > 구독Agent에서 반영 확인
- **검증포인트**: 구독 신청 버튼 활성화 여부 / 구독 완료 후 My > 구독Agent 반영 여부 / 이미 구독 중인 Agent 중복 신청 불가 처리 여부
- **사전 요구사항**: 일반 사용자 계정 로그인 상태 / 서비스 중인 Agent 존재

---

## 5. 일반 사용자 Agent 구독 취소

- **업무기능**: 일반 사용자의 구독 중인 Agent 구독 취소
- **흐름 요약**: 로그인(User) → My > 구독Agent 접근 → 구독 중인 Agent 선택 → 구독 취소 버튼 클릭 → 취소 완료 확인 → 목록에서 해당 Agent 제거 확인
- **검증포인트**: 취소 후 My > 구독Agent 목록에서 제거 여부 / 취소 완료 메시지 출력 여부
- **사전 요구사항**: 일반 사용자 계정 로그인 상태 / 구독 중인 Agent 존재

---

## 6. 일반 사용자 구독 중인 Agent 현황 조회

- **업무기능**: 일반 사용자의 구독 Agent 사용 현황 조회
- **흐름 요약**: 로그인(User) → My > 구독Agent 접근 → 구독 Agent 목록 확인 → Agent별 사용 현황(쿼리 수·토큰·응답속도) 확인
- **검증포인트**: My > 구독Agent 목록 정상 출력 여부 / Agent별 사용 수치 표시 여부
- **사전 요구사항**: 일반 사용자 계정 로그인 상태 / 구독 중인 Agent 존재

---

## 7. Admin 권한 사용자 Agent 승인 목록 조회

- **업무기능**: Admin의 Agent 등록 승인 대기 목록 조회
- **흐름 요약**: 로그인(Admin) → Admin > 승인 목록 조회 접근 → 승인 대기 Agent 목록 출력 확인 → Agent 상세 정보 확인
- **검증포인트**: 승인 대기 Agent 목록 정상 출력 여부 / Agent 상세 정보(이름·보안등급·신청자 등) 표시 여부
- **사전 요구사항**: Admin 계정 로그인 상태 / Developer가 등록 신청한 Agent 존재

---

## 8. Admin 권한 사용자 서비스 요청 Agent 승인

- **업무기능**: Admin의 서비스 요청 Agent 승인 처리
- **흐름 요약**: 로그인(Admin) → Admin > 승인 목록 조회 접근 → 승인 대기 Agent 선택 → 승인 버튼 클릭 → 승인 완료 확인 → Agent 상태 '서비스 중' 변경 확인
- **검증포인트**: 승인 후 Agent 상태 변경 여부 / 승인 완료 메시지 출력 여부 / Developer > Agent 관리에 결과 반영 여부
- **사전 요구사항**: Admin 계정 로그인 상태 / 승인 대기 중인 Agent 존재

---

## 9. Admin 권한 사용자 서비스 요청 Agent 반려

- **업무기능**: Admin의 서비스 요청 Agent 반려 처리
- **흐름 요약**: 로그인(Admin) → Admin > 승인 목록 조회 접근 → 승인 대기 Agent 선택 → 반려 버튼 클릭 → 반려 사유 입력 → 반려 완료 확인 → Agent 상태 '반려' 변경 확인
- **검증포인트**: 반려 후 Agent 상태 변경 여부 / 반려 사유 저장 여부 / Developer > Agent 관리에 반려 결과 반영 여부
- **사전 요구사항**: Admin 계정 로그인 상태 / 승인 대기 중인 Agent 존재

---

## 10. Admin 권한 사용자 서비스 중인 Agent 서비스 일시 중단

- **업무기능**: Admin의 서비스 중인 Agent 일시 중단 처리
- **흐름 요약**: 로그인(Admin) → Admin > 승인 목록 조회 접근 → 서비스 중인 Agent 선택 → 일시 중단 버튼 클릭 → 중단 완료 확인 → Agent 상태 '일시 중단' 변경 확인
- **검증포인트**: 일시 중단 후 Agent 상태 변경 여부 / 일시 중단 완료 메시지 출력 여부 / MarketPlace > AgentMarket에서 해당 Agent 비노출 여부
- **사전 요구사항**: Admin 계정 로그인 상태 / 서비스 중인 Agent 존재

---

## 11. Admin 권한 사용자 서비스 일시 중단 Agent 서비스 재개

- **업무기능**: Admin의 일시 중단된 Agent 서비스 재개 처리
- **흐름 요약**: 로그인(Admin) → Admin > 승인 목록 조회 접근 → 일시 중단 Agent 선택 → 서비스 재개 버튼 클릭 → 재개 완료 확인 → Agent 상태 '서비스 중' 변경 확인
- **검증포인트**: 재개 후 Agent 상태 변경 여부 / 재개 완료 메시지 출력 여부 / MarketPlace > AgentMarket에서 해당 Agent 재노출 여부
- **사전 요구사항**: Admin 계정 로그인 상태 / 일시 중단 중인 Agent 존재

---

## 12. Admin 권한 사용자 서비스 일시 중단 Agent 폐기 신청 승인

- **업무기능**: Admin의 Developer 폐기 신청 Agent 승인 처리
- **흐름 요약**: 로그인(Admin) → Admin > 승인 목록 조회 접근 → 폐기 신청 Agent 선택 → 폐기 승인 버튼 클릭 → 승인 완료 확인 → Agent 상태 '폐기 승인' 변경 확인
- **검증포인트**: 폐기 승인 후 Agent 상태 변경 여부 / 승인 완료 메시지 출력 여부 / Developer > Agent 관리에 결과 반영 여부
- **사전 요구사항**: Admin 계정 로그인 상태 / Developer가 폐기 신청한 Agent 존재

---

## 13. Admin 권한 사용자 서비스 일시 중단 Agent 폐기 신청 반려

- **업무기능**: Admin의 Developer 폐기 신청 Agent 반려 처리
- **흐름 요약**: 로그인(Admin) → Admin > 승인 목록 조회 접근 → 폐기 신청 Agent 선택 → 반려 버튼 클릭 → 반려 사유 입력 → 반려 완료 확인 → Agent 상태 '일시 중단' 복귀 확인
- **검증포인트**: 반려 후 Agent 상태 일시 중단 복귀 여부 / 반려 사유 저장 여부 / Developer > Agent 관리에 반려 결과 반영 여부
- **사전 요구사항**: Admin 계정 로그인 상태 / Developer가 폐기 신청한 Agent 존재

---

## 14. Admin 권한 사용자 서비스 일시 중단 Agent 영구 폐기

- **업무기능**: Admin의 일시 중단 Agent 영구 폐기 처리
- **흐름 요약**: 로그인(Admin) → Admin > 승인 목록 조회 접근 → 일시 중단 Agent 선택 → 영구 폐기 버튼 클릭 → 확인 팝업 → 영구 폐기 완료 확인 → 목록에서 Agent 제거 확인
- **검증포인트**: 영구 폐기 후 Agent 목록 제거 여부 / 폐기 완료 메시지 출력 여부 / 복구 불가 확인 팝업 표시 여부
- **사전 요구사항**: Admin 계정 로그인 상태 / 폐기 승인 상태의 Agent 존재

---

## 15. Developer 권한 사용자 Agent 등록 신청

- **업무기능**: Developer의 신규 Agent 등록 신청
- **흐름 요약**: 로그인(Developer) → Developer > Agent 관리 접근 → 등록 신청 버튼 클릭 → Agent 정보 입력(이름·설명·보안등급·공개설정 등) → 중복 확인 → 등록 신청 완료 확인 → 승인 대기 상태 전환 확인
- **검증포인트**: 필수 항목 입력 유효성 검증 여부 / 중복 Agent명 불가 처리 여부 / 신청 후 승인 대기 상태 전환 여부 / Admin > 승인 목록 조회에 노출 여부
- **사전 요구사항**: Developer 계정 로그인 상태

---

## 16. Developer 권한 사용자 Agent 정보 수정/신청

- **업무기능**: Developer의 등록된 Agent 정보 수정 후 재신청
- **흐름 요약**: 로그인(Developer) → Developer > Agent 관리 접근 → 수정할 Agent 선택 → 정보 수정 입력 → 수정 신청 버튼 클릭 → 수정 신청 완료 확인 → 승인 대기 상태 전환 확인
- **검증포인트**: 수정 가능 항목 활성화 여부 / 수정 후 승인 대기 상태 전환 여부 / 수정 내용 저장 여부 / Admin > 승인 목록 조회에 수정 신청 노출 여부
- **사전 요구사항**: Developer 계정 로그인 상태 / 등록된 Agent 존재

---

## 17. Developer 권한 사용자 Agent 등록 취소

- **업무기능**: Developer의 승인 대기 중인 Agent 등록 취소
- **흐름 요약**: 로그인(Developer) → Developer > Agent 관리 접근 → 승인 대기 Agent 선택 → 등록 취소 버튼 클릭 → 취소 완료 확인 → 목록에서 Agent 제거 확인
- **검증포인트**: 등록 취소 후 Developer > Agent 관리 목록 제거 여부 / 서비스 중 Agent 취소 불가 처리 여부 / Admin > 승인 목록 조회에서도 제거 여부
- **사전 요구사항**: Developer 계정 로그인 상태 / 승인 대기 중인 Agent 존재

---

## 18. Developer 권한 사용자 폐기 신청

- **업무기능**: Developer의 일시 중단된 Agent 폐기 신청
- **흐름 요약**: 로그인(Developer) → Developer > Agent 관리 접근 → 일시 중단 Agent 선택 → 폐기 신청 버튼 클릭 → 폐기 사유 입력 → 신청 완료 확인 → Agent 상태 '폐기 신청' 변경 확인
- **검증포인트**: 폐기 신청 후 Agent 상태 변경 여부 / Admin > 승인 목록 조회에 폐기 신청 노출 여부 / 서비스 중 Agent에 대한 폐기 신청 불가 처리 여부
- **사전 요구사항**: Developer 계정 로그인 상태 / 일시 중단 중인 Agent 존재

---

## 19. Developer 권한 사용자 Agent 유효성(validation) 검증

- **업무기능**: Developer의 Agent 유효성 3단계 검증 수행
- **흐름 요약**: 로그인(Developer) → Developer > Agent 관리 접근 → Agent 선택 → Validation 버튼 클릭 → 1단계 Router health-check 실행 → 2단계 Gateway Key 인증 실행 → 3단계 A2A 호환성 확인 → 검증 결과 확인
- **검증포인트**: 각 단계별 성공/실패 상태 표시 여부 / A2A 미허용 Agent 3단계 skip 처리 여부 / 최종 검증 결과 메시지 출력 여부
- **사전 요구사항**: Developer 계정 로그인 상태 / 등록된 Agent 존재 / Gateway Key 발급 완료

---

## 20. Developer 권한 사용자 Agent 서비스 일시 중단

- **업무기능**: Developer의 서비스 중인 Agent 일시 중단
- **흐름 요약**: 로그인(Developer) → Developer > Agent 관리 접근 → 서비스 중인 Agent 선택 → 일시 중단 버튼 클릭 → 중단 완료 확인 → Agent 상태 '일시 중단' 변경 확인
- **검증포인트**: 일시 중단 후 Agent 상태 변경 여부 / Admin > 승인 목록 조회에 상태 동기화 여부 / MarketPlace > AgentMarket에서 해당 Agent 비노출 여부
- **사전 요구사항**: Developer 계정 로그인 상태 / 서비스 중인 Agent 존재

---

## 21. Developer 권한 사용자 Agent 서비스 재개

- **업무기능**: Developer의 일시 중단된 Agent 서비스 직접 재개
- **흐름 요약**: 로그인(Developer) → Developer > Agent 관리 접근 → 일시 중단 Agent 선택 → 서비스 재개 버튼 클릭 → 재개 완료 확인 → Agent 상태 '서비스 중' 변경 확인
- **검증포인트**: 재개 후 Agent 상태 '서비스 중' 변경 여부 / MarketPlace > AgentMarket에서 해당 Agent 재노출 여부
- **사전 요구사항**: Developer 계정 로그인 상태 / 일시 중단 중인 Agent 존재
