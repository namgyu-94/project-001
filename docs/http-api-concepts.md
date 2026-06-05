# HTTP 메서드 & Chat Completion 개념 정리

---

## HTTP 메서드 (GET / POST / OPTIONS)

| 메서드 | 목적 | Body 전송 | 서버 상태 변경 |
|--------|------|-----------|---------------|
| **GET** | 데이터 조회 | 없음 | 없음 (읽기 전용) |
| **POST** | 데이터 생성/처리 | 있음 | 있음 |
| **OPTIONS** | 서버가 뭘 지원하는지 질문 | 없음 | 없음 |

### GET
"이 주소의 내용을 줘"
```
GET /agents/legal-review-agent
→ 응답: agent 정보 JSON
```

### POST
"이 데이터를 처리해줘"
```
POST /agents/legal-review-agent/invoke
Body: { "text": "계약서 내용..." }
→ 응답: 분석 결과
```

### OPTIONS
"너 어떤 요청 받을 수 있어?"
```
OPTIONS /agents/legal-review-agent
→ 응답 헤더: Allow: GET, POST, OPTIONS
             Access-Control-Allow-Origin: *
```

---

## Preflight란?

실제 요청을 보내기 전에 서버가 응답 가능한 상태인지 가볍게 먼저 확인하는 사전 검사.

### A2A Validation에서의 활용 순서

| 단계 | 메서드 | 목적 |
|------|--------|------|
| 1 | OPTIONS | 서버가 A2A 프로토콜을 지원하는지 확인 (허용 메서드에 POST 포함 여부) |
| 2 | HEAD | 서버 생존 여부를 body 없이 빠르게 확인 |
| 3 | GET | Agent Card를 실제로 받아와 스킬 목록 파싱 |
| 4 | POST | 실제 스킬 호출 |

단계적으로 확인하면 실패 시 **어느 단계에서 막혔는지** 정확히 진단 가능.

---

## OPTIONS는 표준인가?

- HTTP 표준(RFC 7231)에 정의된 공식 메서드
- 특정 프레임워크나 A2A 전용이 아니라 모든 HTTP 서버에서 공통 사용

### 가장 흔한 사용 사례: CORS Preflight

브라우저가 다른 도메인에 POST/PUT 요청을 보내기 전에 자동으로 OPTIONS를 먼저 발송.

```
브라우저 → OPTIONS /api/data   (자동 발송)
서버     → Access-Control-Allow-Origin: *  (허용)
브라우저 → POST /api/data       (본 요청 전송)
```

개발자 도구 네트워크 탭을 열면 OPTIONS 요청이 먼저 찍히는 것을 확인 가능.

---

## Chat Completion

OpenAI가 만든 API 방식의 이름. "대화 내용을 주면, 다음 메시지를 완성(completion)해줘"라는 개념.

### 구조

```json
{
  "messages": [
    { "role": "system",    "content": "너는 법률 전문가야" },
    { "role": "user",      "content": "계약서 검토해줘" },
    { "role": "assistant", "content": "네, 보내주세요" },
    { "role": "user",      "content": "여기 있어요: ..." }
  ]
}
```

→ 이 대화 흐름을 주면, AI가 다음 assistant 메시지를 생성해서 반환.

### 왜 "Completion"이라는 이름인가

원래 GPT 초기에는 텍스트를 주면 이어서 완성해주는 단순한 구조(`text completion`)였음. 거기서 발전해 대화 형식이 되면서 `chat completion`이 된 것.

### 현재는 사실상 업계 표준

OpenAI가 만든 명칭이지만, Anthropic(Claude), Google(Gemini) 등 대부분의 LLM API가 호환 형식으로 채택하거나 비슷한 구조를 사용. "Chat Completion API"라고 하면 업계 공통 용어로 통용됨.
