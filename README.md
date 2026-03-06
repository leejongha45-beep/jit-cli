# JIT CLI

**C++ in-memory code graph engine + LLM chat CLI**

Any codebase, indexed in seconds. Symbol queries in microseconds. No database, no server — just a single binary.

코드베이스를 초 단위로 인덱싱하고, 마이크로초 단위로 심볼을 검색합니다. DB 없이, 서버 없이 — 바이너리 하나로 동작합니다.

---

## What it does / 소개

JIT indexes your codebase into an in-memory symbol graph, then lets you chat with an LLM using precise code context. No hallucination — the LLM only sees what you inject.

JIT는 코드베이스를 인메모리 심볼 그래프로 인덱싱한 뒤, 정확한 코드 컨텍스트를 주입하여 LLM과 대화합니다. LLM은 주입된 심볼만 보기 때문에 할루시네이션이 없습니다.

```
$ jit /path/to/your/project
1118 source files found. Index? [Y/n]
Indexed 1118 files, 11963 symbols (0.14s)
```

---

## Benchmarks / 벤치마크

| Codebase | Files | Symbols | Index Time | Query |
|----------|-------|---------|------------|-------|
| FastAPI | 1,118 | 11,963 | 0.14s | < 1ms |
| Unreal Engine 5.7 | 28,000 | 888,000 | 49s | 2.6ms |
| Linux Kernel | 63,090 | 7,480,270 | 88s | 0.19ms |

---

## Tech Stack / 기술 스택

- **Language / 언어**: C++20
- **Parser / 파서**: tree-sitter (C, C++, Python)
- **Architecture / 아키텍처**: DOD (Data-Oriented Design) — SOA 연속 배열, StringPool 인터닝, 심볼당 힙 할당 0
- **Indexing / 인덱싱**: 멀티스레드 병렬 파싱 + 배치 전처리
- **Query / 쿼리**: 인메모리 그래프 양방향 BFS (SmartQuery)
- **LLM**: Claude, Gemini, Ollama (스트리밍)
- **Security / 보안**: libsodium (Argon2id KDF + XSalsa20-Poly1305) API 키 암호화
- **Dependencies / 의존성**: tree-sitter, libsodium, libgit2, libcurl, nlohmann/json

---

## Install / 설치

Download the latest binary from [Releases](https://github.com/leejongha45-beep/jit-cli/releases).

[Releases](https://github.com/leejongha45-beep/jit-cli/releases) 페이지에서 OS에 맞는 바이너리를 다운로드하세요.

### Windows
```
jit.exe [path]
```

### macOS / Linux
```bash
chmod +x jit
./jit [path]
```

경로를 생략하면 현재 작업 디렉토리를 사용합니다.
If no path is given, JIT uses the current working directory.

---

## Supported Languages / 지원 언어

| Language | Extensions |
|----------|-----------|
| C | `.c` |
| C++ | `.cpp`, `.hpp`, `.h`, `.cc`, `.cxx` |
| Python | `.py` |

---

## Commands / 명령어

### Global Commands / 전역 명령어

모든 모드에서 사용 가능합니다. Available in all modes.

| Command | Description / 설명 |
|---------|-------------|
| `/chat` | Chat 모드로 전환 — Switch to Chat mode |
| `/query` | Query 모드로 전환 — Switch to Query mode |
| `/book` | Book 모드로 전환 — Switch to Book mode |
| `/searchws <name>` | 워크스페이스 심볼 검색 — Search workspace symbols |
| `/searchbook <name>` | 교재 심볼 검색 — Search book symbols |
| `/symws <name>` | 심볼 그래프 표시 (멤버, 엣지) — Show symbol graph |
| `/symbook <name>` | 교재 심볼 그래프 표시 — Show book symbol graph |
| `/opencode <name>` | 심볼의 전체 소스코드 표시 — Show full source code |
| `/patch <file>` | LLM이 제시한 코드를 파일에 적용 — Apply LLM code edits |
| `/config` | LLM 프로바이더/API 키 설정 — Configure LLM provider |
| `/rc` | 심볼 카트 비우기 — Clear symbol cart |
| `/help` | 도움말 — Show help |
| `/quit` | 종료 — Quit |

### Chat Mode / 채팅 모드

LLM과 대화하면서 심볼 컨텍스트를 주입합니다. Talk with an LLM using symbol context injection.

```
chat> @FastAPI explain the middleware stack
[1 symbol context(s) injected]

assistant: Based on the FastAPI class definition...
```

- `@심볼이름`을 메시지에 입력하면 해당 심볼의 코드 컨텍스트가 주입됩니다
- Type `@SymbolName` to inject that symbol's code context
- Query/Book 모드에서 `/finish`로 카트에 담은 심볼은 매 메시지마다 자동 주입됩니다

### Query Mode / 쿼리 모드 — `/query`

인메모리 심볼 그래프를 탐색합니다. Explore the in-memory symbol graph.

| Command | Description / 설명 |
|---------|-------------|
| `<name>` | 심볼 빠른 검색 — Quick search |
| `/smart <name>` | 양방향 BFS 쿼리 — Bidirectional BFS query |
| `/tree [name]` | 심볼 트리 표시 — Show symbol tree |
| `/symbols <name>` | 이름으로 심볼 목록 — List matching symbols |
| `/search <name>` | 검색 결과 표시 — Search and display |
| `/choose <number>` | 결과 번호로 카트에 추가 — Add to cart |
| `/remove <number>` | 카트에서 제거 — Remove from cart |
| `/finish` | 카트를 Chat 모드로 전송 — Send cart to Chat mode |

### Book Mode / 교재 모드 — `/book`

외부 코드베이스(로컬에 클론된 저장소 또는 로컬 경로)를 참고 교재로 인덱싱합니다.
Index external codebases (local cloned repos or local paths) as reference material.

| Command | Description / 설명 |
|---------|-------------|
| `/index <local-path>` | 로컬 경로의 코드베이스를 교재로 인덱싱 — Index a local codebase as book |
| `/search <name>` | 교재 심볼 검색 — Search book symbols |
| `/choose <number>` | 카트에 추가 — Add to cart |
| `/remove <number>` | 카트에서 제거 — Remove from cart |
| `/removebook <path>` | 인덱싱된 교재 제거 — Remove a book |
| `/finish` | 카트를 Chat 모드로 전송 — Send cart to Chat mode |

---

## LLM Configuration / LLM 설정

최초 실행 시 LLM 프로바이더를 설정하세요. On first run, configure your LLM provider:

```
/config
```

Supported providers / 지원 프로바이더:
- **Claude** (Anthropic) — `ANTHROPIC_API_KEY` 필요
- **Gemini** (Google) — `GEMINI_API_KEY` 필요
- **Ollama** (Local) — 키 불필요, 로컬 실행

API 키는 libsodium으로 암호화되어 저장되며, 마스터 비밀번호로 보호됩니다.
API keys are encrypted at rest with libsodium and protected by a master password.

---

## How It Works / 동작 원리

```
Source Files → tree-sitter Parse → Symbol Graph (in-memory)
                                        ↓
                              SmartQuery (bidirectional BFS)
                                        ↓
                              Symbol Context Injection
                                        ↓
                                   LLM Chat
```

1. **Index / 인덱싱**: tree-sitter로 소스 파일을 POD 심볼(Function, Class, Variable, Include)과 관계 엣지(DefinedIn, MemberOf, Calls, Inherits, Returns)로 파싱
2. **Query / 쿼리**: SmartQuery가 그래프에서 양방향 BFS 실행 — 순방향은 심볼이 사용하는 것, 역방향은 심볼을 사용하는 것을 탐색
3. **Inject / 주입**: 채팅에서 `@심볼이름` 입력 시 해당 심볼의 메타데이터를 컨텍스트로 주입
4. **Generate / 생성**: LLM은 주입된 심볼 컨텍스트만을 기반으로 코드 생성 — 추측 없음, 할루시네이션 없음

---

## License

All rights reserved.
