
(AS-IS)
Frontend -> FastAPI (Backend) -> Redis Queue -> Celery Worker (LangGraph + in-process) -> Redis Streams(이벤트 발행) -> redis Streams 읽기 -> FastAPI (Backend) -> SSE -> Frontend

(TO-BE)
Frontend -> FastAPI (Backend) -> Redis Queue -> Celery Worker (Post/run) -> SuperAgent FastAPI -> LangGraph 실행 -> RedisGraph실행 -> Redis Streams (직접 발행) -> Redis Streams (읽기) -> FastAPI (backend) -> SSE -> Frontend




현재 시스템에는 여러 Agent가 존재하며, 모두 FastAPI + LangGraph 형식으로 개발되고 있습. 그런데 superagent만 Celery Worker내부에 Langgraph가 내장되어 있어 형식이 다름


<validation>
