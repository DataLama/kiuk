# kiuk Project Guidelines

## Overview
kiuk (기억) — 로컬 우선, 프라이버시 우선의 개인 메모리 스토어.
SQLite + sqlite-vec + usearch 기반으로 텍스트와 벡터를 단일 파일에 저장하고,
MCP 서버를 통해 AI 에이전트에게 개인 컨텍스트를 제공한다.

## Project Structure
```
kiuk/
├── src/kiuk/
│   ├── __init__.py        # Kiuk 클래스 및 public API export
│   ├── store.py           # SQLite + sqlite-vec + FTS5 코어 스토리지
│   ├── search.py          # Hybrid search (vector + keyword)
│   ├── embeddings.py      # 임베딩 어댑터 (default: Gemma Embedding 300M)
│   ├── cli.py             # CLI entrypoint (kiuk serve/remember/recall)
│   └── mcp/               # MCP server (optional, pip install kiuk[mcp])
│       ├── __init__.py
│       └── server.py
├── tests/
├── pyproject.toml
└── LICENSE                # Apache-2.0
```

## Dev Commands
```bash
uv sync                    # 기본 설치 (sqlite-vec + usearch만)
uv sync --group dev        # 개발 환경 (pytest, ruff)
uv sync --all-extras       # 전체 설치 (embedding + mcp + dev)
uv run pytest              # 테스트
uv run ruff check src/     # 린트
uv run ruff format src/    # 포맷
```

## Architecture Principles
1. **Local-first**: 모든 데이터는 SQLite 단일 파일에 저장. 클라우드 동기화 없음.
2. **Privacy-first**: 네트워크 전송 없음. MCP도 stdio transport (로컬 전용).
3. **Lightweight**: 최소 의존성. Core는 sqlite-vec + usearch만.
4. **Pluggable Embedding**: 기본 Gemma Embedding 300M, 사용자 지정 embed_fn 또는 HF 모델 지원.
5. **Standalone**: 외부 프레임워크 의존 없는 독립 오픈소스 라이브러리.

## Core API Design
```python
from kiuk import Kiuk

db = Kiuk("~/kiuk.db")
db.remember("나는 polars를 선호한다", tags=["preference"])
results = db.recall("어떤 dataframe 라이브러리?", top_k=5)
db.forget(id=results[0].id)
```

## Embedding Strategy
- **Default**: Gemma Embedding 300M (`google/gemma-embedding-300m`)
- **Custom**: `Kiuk(embed_fn=my_func)` 또는 `Kiuk(model_name="my-org/my-model")`
- HF private model 지원 (HF_TOKEN 환경변수)

## Rules
- SQLite 스키마 변경 시 migration 전략 필수 문서화
- MCP 관련 코드는 `kiuk.mcp` 모듈에 격리 (core에서 import 금지)
- 테스트는 반드시 in-memory SQLite (`:memory:`)로 실행

## Information Boundary
이 프로젝트는 **독립 오픈소스 라이브러리**이다.
- 외부 프로젝트의 규칙/컨벤션/내부 정보를 kiuk 코드에 반영하지 않는다
- 의존성은 `pyproject.toml`에 명시된 것만 사용
- 다른 workspace의 rules와 충돌 시, 이 CLAUDE.md가 우선한다
