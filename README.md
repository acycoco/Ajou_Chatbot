# 🎓 Ajou University Academic Info & Notice Project RAGGY Backend

이 프로젝트는 아주대학교 **학사요람 정보** 및 **학과/단과대 공지사항**을 검색하고 요약해주는 **RAG 기반 FastAPI 백엔드**입니다.  
LangChain, LangGraph, ChromaDB, LLM API(OpenAI, Anthropic, Google Gemini 등)를 활용하여 자연어 질의에 응답합니다.

---

## 🚀 Features

- **학사 요람 RAG 검색**
  - PDF/Markdown 기반 학사요람 문서를 전처리 및 청킹하여 벡터DB(ChromaDB)에 저장
  - 하이브리드 검색 (BM25 + Dense Embedding + Cross-Encoder 재랭크)
  - 학년/학기 기반 direct term 확장 검색 지원
  - LLM 응답에 출처 표시

- **학과/단과대 공지사항 검색**
  - 크롤링된 DB + ChromaDB 연동
  - 자연어 질의에 따라 학과/단과대/공지유형 필터 자동 적용
  - 공지 리스트 형식으로 정리된 답변 제공

- **LangGraph 기반 파이프라인**
  - 질문 → 분류(node_classify) → 의도 파싱(node_parse_intent) → 검색(node_retrieve) → 컨텍스트 조립(node_build_context) → 답변(node_answer)
  - 노드 기반 구조로 디버깅과 확장 용이

- **LLM 연동**
  - `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GOOGLE_API_KEY` 환경변수 기반
  - 모델은 `.env` 설정에 따라 OpenAI GPT, Claude, Gemini 교체 가능
  - 기본값: `gpt-4o-mini` (학사요람) / `gemini-1.5-flash` (공지)

---

## 🛠️ Tech Stack

- **Backend Framework**: FastAPI, Uvicorn  
- **RAG / LLM**: LangChain, LangGraph, LlamaIndex(일부), HuggingFace Transformers  
- **Vector DB**: ChromaDB  
- **Search**: BM25 (rank-bm25), Dense Embedding (BAAI/bge-m3), Cross-Encoder rerank  
- **Embedding**: sentence-transformers, HuggingFaceBgeEmbeddings  
- **Tokenizer**: kiwipiepy (기본), konlpy.Okt (공지사항 전용)  
- **Scheduler**: Apache Airflow (공지 크롤링/임베딩 DAG)  
- **Infra/ETC**: Docker 지원, dotenv 기반 설정  

---

## 📂 Project Structure

```bash
project-root/
├── .venv/                  # 가상환경
├── airflow/                # Airflow DAGs 및 워크플로우
├── app/                    # 핵심 백엔드 코드
│   ├── agents/             # LLM 에이전트 관련
│   ├── api/                # FastAPI 라우터
│   ├── core/               # 전역 설정, 로깅, 환경변수 로드
│   ├── data/               # 데이터 전처리 및 변환 로직
│   ├── graphs/             # LangGraph 기반 워크플로우 정의
│   ├── models/             # 데이터 모델 정의
│   ├── scripts/            # 관리/유틸리티 스크립트
│   ├── services/           # 서비스 계층 (retriever, pipeline 등)
│   ├── utils/              # 공통 유틸리티 함수
│   ├── .env                # 환경 변수 설정
│   └── main.py             # FastAPI 진입점
├── data/                   # 원본 데이터(.md) 저장소
├── docker/                 # Docker 관련 설정
├── dump/                   # 로그/덤프 파일
├── scripts/                # 독립 실행 스크립트
├── storage/                # ChromaDB 영속 스토리지
├── pytest.ini              # pytest 설정
└── README.md               # 프로젝트 설명 파일
```

---

## ⚙️ Setup & Run

### 1. Clone & Install
```bash
git clone https://github.com/ICTProject11/Ajou_Chatbot.git
cd Ajou_Chatbot

# 가상환경 생성
python -m venv .venv
source .venv/bin/activate

# 의존성 설치
pip install -r requirements.txt
```

### 2. Environment Variables

`.env` 파일을 프로젝트 루트에 생성:  

```env
# LLM
OPENAI_API_KEY=<Your_api_key>
ANTHROPIC_API_KEY=<Your_api_key>
GOOGLE_API_KEY=<Your_api_key>
LLM_MODEL=claude-3-5-sonnet-20240620
LLM_MODEL_NOTICE=gemini-1.5-flash
# LLM_MODEL=gpt-4o-mini
MAX_TOKENS=5000
MAX_TOKENS_OUTPUT=3500
ASSEMBLE_BUDGET_TOKENS=12000
EMBEDDING_MAX_TOKENS=8192    # bge-m3 권장
LLAMA_CLOUD_API_KEY=<Your_api_key>

# 스케줄러 on/off (개발 환경에서 reload 중복 방지용)
ENABLE_SCHEDULER=1

# 스케줄 시간(원하면 코드에서 읽어 쓰기)
NOTICE_CRON_HOUR=0
NOTICE_CRON_MINUTE=0

# Chroma 저장 위치
CHROMA_DIR=./storage

# 컬렉션 이름 커스터마이즈
CHROMA_COLLECTION_MAJOR=major_dm

COLLECTION=acad_docs_bge_m3_clean
NOTICE_COLLECTION=langchain
EMBEDDING_MODEL=BAAI/bge-m3
PERSIST_DIR=storage/chroma-acad
PERSIST_DIR_NOTICE=storage/chroma-notice
```

### 3. Run Server
```bash
uvicorn app.main:app --reload --port 8000
```

- API docs: [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

---

## 📌 Example Usage

### 학사 요람 질의
```bash
curl -X POST http://127.0.0.1:8000/info \
  -H "Content-Type: application/json" \
  -d '{"question":"디지털미디어학과 졸업요건 요약"}'
```

### 공지사항 질의
```bash
curl -X POST http://127.0.0.1:8000/announcement \
  -H "Content-Type: application/json" \
  -d '{"question":"신소재 학사공지?", "departments":["첨단신소재공학과"]}'
```

---

## 🏗️ 특징 및 차별화 포인트

- **하이브리드 검색 강화**  
  Lexical(BM25) + Semantic(Dense) + Cross-Encoder 조합으로 고정밀 검색

- **섹션 확장 기반 Retrieval**  
  학년/학기 기반 질의는 해당 term 전체 섹션으로 확장 → 문맥 보존

- **LangGraph 기반 Graph Pipeline**  
  노드 기반 설계로 디버깅, 분기, 확장성이 뛰어남

- **LLM 호출 비용 최소화**  
  단순 FAQ/고정응답은 LLM 호출 없이 처리 → 비용 절감

- **Airflow 자동화**  
  매일 크롤링된 공지사항을 DB에 저장 및 ChromaDB 업데이트 자동화

---

## 📖 License

MIT License
