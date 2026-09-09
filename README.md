# AskHR — 사내 규정 AI 챗봇

AIO 2기 삼 팀 프로젝트. 임직원이 사내 규정(급여 · 휴가 · 복리후생 · 출장 · 보안 등)을 자연어로 질문하면 규정 문서를 근거로 답변하고, 운영자는 사용량 대시보드에서 응답 지연시간과 토큰 사용량을 확인한다.

- 백엔드 배포: https://two026-aio2-ask-hr-backend.onrender.com (Swagger UI: [/docs](https://two026-aio2-ask-hr-backend.onrender.com/docs))
- 스택: FastAPI · Supabase (Auth + PostgreSQL, RLS) · Redis · Google Gemini · Streamlit

## 필수 산출물

| 산출물 | 파일 |
|---|---|
| PRD | [산출물 모음/ASKHR_PRD.pdf](산출물%20모음/ASKHR_PRD.pdf) |
| 화면 설계서 | [산출물 모음/필수 산출물 3종/삼 팀 AskHR 화면 설계서_최종수정.html](산출물%20모음/필수%20산출물%203종/삼%20팀%20AskHR%20화면%20설계서_최종수정.html) |
| API 설계 문서 | [HTML](산출물%20모음/필수%20산출물%203종/API%20설계%20문서.html) · [Markdown](산출물%20모음/API%20설계%20문서.md) |
| 데이터베이스 설계서 | [HTML](산출물%20모음/필수%20산출물%203종/데이터베이스%20설계서.html) · [Markdown](산출물%20모음/데이터베이스%20설계서.md) |
| 대시보드 구현 결과물 | 배포 URL에서 라이브 시연 (`로그 및 데이터 분석` 메뉴) |

HTML 문서는 다운로드 후 브라우저에서 열면 된다 (외부 의존성 없는 단일 파일, `Ctrl+P`로 PDF 저장 가능).

## 저장소 구조

```
.
├── backend/        FastAPI API 서버 (app/, data/regulations/, docs/, tests/)
├── frontend/       Streamlit 웹 프런트엔드 (채팅 · 분석 대시보드)
├── 규정 샘플/       챗봇이 근거로 사용하는 사내 규정 문서 8종 + 활용 가이드
└── 산출물 모음/     PRD · 화면 설계서 · API 설계 문서 · DB 설계서
```

`backend/`, `frontend/`는 팀 조직 저장소의 스냅샷이다 (커밋 히스토리 없음). 원본:
- https://github.com/poohoot-ai/2026_aio2_ask-hr-backend
- https://github.com/poohoot-ai/2026_aio2_ask-hr-frontend

## 로컬 실행

### 백엔드

```bash
cd backend
cp .env.example .env   # Supabase · Redis · Gemini 키 입력
uv sync
uv run uvicorn app.main:app --reload
# http://127.0.0.1:8000/docs
```

### 프런트엔드

```bash
cd frontend
uv sync
uv run streamlit run streamlit_app.py
```

`.env`와 `.venv/`는 커밋하지 않는다 (`.gitignore` 참고).

## 주요 설계 포인트

- **인증 · 소유권**: Supabase JWT(Bearer)로 인증하고, 대화 소유권은 앱 코드가 아닌 PostgreSQL RLS가 판단한다. 없는 대화와 남의 대화는 동일하게 404.
- **답변 생성**: 질문 키워드로 관련 규정 MD를 골라 시스템 프롬프트에 넣고 Gemini 스트리밍 응답을 SSE(`text` / `done` / `error` 이벤트)로 전달한다. 답변 마지막 줄에 `[규정명 > 항목명]` 근거를 표기한다.
- **운영 지표**: 응답 지연 · 토큰 사용량은 Redis 리스트에 대화당 최근 50건만 기록하고, 대시보드 API가 이를 집계한다. 질문 · 답변 본문과 개인정보는 로그에 넣지 않는다.
