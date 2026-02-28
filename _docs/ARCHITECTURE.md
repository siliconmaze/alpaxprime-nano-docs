# Alpax Prime Nano — Architecture Documentation

## Overview

Alpax Prime Nano is an autonomous AI co-pilot built with Microsoft's AutoGen framework. It runs locally, connects to multiple LLM providers, and can execute tasks across Content, Build, and Ops domains.

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Alpax Prime Nano                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐         │
│  │   Telegram   │    │     REST     │    │    SSE      │         │
│  │   Bot        │    │   API        │    │  Streaming  │         │
│  └──────┬───────┘    └──────┬───────┘    └──────┬───────┘         │
│         │                    │                    │                 │
│         └────────────────────┼────────────────────┘                 │
│                              │                                      │
│                      ┌───────▼───────┐                              │
│                      │   Engine      │                              │
│                      │  (Hono/Node)  │                              │
│                      └───────┬───────┘                              │
│                              │                                      │
│         ┌────────────────────┼────────────────────┐               │
│         │                    │                    │               │
│  ┌──────▼──────┐    ┌───────▼───────┐    ┌──────▼──────┐        │
│  │   Graph     │    │   Tools       │    │  Database   │        │
│  │  (LangGraph)│    │  (AutoGen)    │    │  (LibSQL)   │        │
│  └─────────────┘    └───────────────┘    └─────────────┘        │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────┐     │
│  │                    LLM Providers                           │     │
│  │  ┌─────────┐  ┌──────────┐  ┌─────────┐  ┌──────────┐  │     │
│  │  │ Ollama  │  │ DeepSeek │  │ MiniMax │  │ Anthropic│  │     │
│  │  │ (Local) │  │   (API)  │  │  (API)  │  │  (API)   │  │     │
│  │  └─────────┘  └──────────┘  └─────────┘  └──────────┘  │     │
│  └──────────────────────────────────────────────────────────┘     │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

## Core Components

### 1. Engine Server (`engine/server.ts`)
- **Framework**: Hono (fast HTTP server)
- **Port**: 3901
- **Responsibilities**:
  - Route handling (REST + SSE)
  - Request/response processing
  - Middleware (CORS, auth)

### 2. Agent Graph (`engine/agent/graph.ts`)
- **Framework**: LangGraph (LangChain)
- **Pattern**: ReAct (Reasoning + Action)
- **Flow**:
  ```
  User Message → Agent Node → [Tools?] → Tools Node → Agent Node → Response
       │                                              │
       └──────────────── Checkpoint ──────────────────┘
  ```
- **Features**:
  - Persistent conversation history (LibSQL)
  - Dangling tool call repair
  - Error recovery & retry logic

### 3. Tools System (`engine/agent/tools/`)
- **AutoGen** binds tools to the LLM
- **Tool Types**:
  - File operations (bash_exec, doc_to_pdf)
  - Communication (gmail, telegram)
  - Web (web_search)
  - AI delegation (spawn_subagent)
  - Media (transcribe_audio, analyze_image)

### 4. Database (`engine/lib/db.ts`)
- **Engine**: LibSQL (Turso/SQLite-compatible)
- **Tables**:
  - `threads` — Conversation threads
  - `messages` — Individual messages
  - `cards` — Kanban tasks
  - `run_logs` — Execution logs
  - `settings` — Configuration key-value store (includes `card_type_rules` and `execution_rules` JSON blobs)

### 5. Rules System (`engine/routes/rules.ts`)
- **Card Type Rules** (`CardTypeRule`): Per-kind rules that control which tools are allowed/forbidden and what prompt text is prepended when a card of that kind runs. Fields: `id`, `kind`, `name`, `description`, `allowedTools[]`, `forbiddenTools[]`, `prompt`, `enabled`.
- **Execution Rules** (`ExecutionRule`): Cross-thread behavioural rules injected into the agent system prompt. Fields: `id`, `name`, `trigger` (`on_process` | `on_complete_check` | `on_workflow`), `description`, `enforcement`, `enabled`.
- **Seed data**: Five `CardTypeRule` records (code, content, monitor, todo, review) and four `ExecutionRule` records are inserted at migrate time.
- **CRUD API**: `GET/POST/PUT/DELETE /rules`, `/rules/card-type/:kind`, `/rules/card-type`, `/rules/execution`.
- **Injection**: Enabled `CardTypeRule` records are injected into the card execution prompt in `run-card.ts`. Enabled `ExecutionRule` records are injected into every thread's system prompt via `getSystemPrompt()` in `graph.ts`.

### 6. Persistent Memory (`engine/agent/graph.ts`, `engine/lib/`)
- **MEMORY.md injection**: `getSystemPrompt(threadId)` reads `MEMORY.md` from `media_base_dir` and prepends it to the system prompt for every new thread. Thread ID and data dir are also injected so the agent knows its own context.
- **Compaction warning**: When `MEMORY.md` exceeds 180 lines, a warning is appended to the system prompt prompting the agent to compact its memory.
- **Context hash**: `buildContextHash(dataDir)` computes a 12-char SHA-256 over enabled rule IDs + `updatedAt` timestamps + `MEMORY.md` mtime. Used to detect staleness.
- **Auto-refresh**: `autoRefreshContextIfStale(threadId)` is called on every non-first turn (skipped for card- and sub- threads). When the hash has changed, `injectContextUpdate(threadId)` injects a `SystemMessage` into the existing thread checkpoint via `compiledGraph.updateState()`.
- **HTTP endpoint**: `POST /threads/:id/refresh-context` exposes manual refresh.

### 7. Telegram Bot (`engine/lib/telegram-bot.ts`)
- **Library**: Grammy
- **Features**:
  - Voice message transcription (Whisper)
  - File uploads
  - Typing indicators
  - Rate limiting

## Data Flow

### 1. User sends message via Telegram
```
Telegram → Webhook → downloadTelegramFile → transcribeAudio → runAgent → Reply
```

### 2. User sends message via UI
```
HTTP POST /api/agent → runAgent → SSE Response → UI
```

### 3. Tool Execution
```
Agent decides tool → ToolNode executes → Result returned to Agent → Final response
```

## Configuration

### Environment Variables (`.env.local`)
```bash
# LLM Providers
ANTHROPIC_API_KEY=sk-...
DEEPSEEK_API_KEY=sk-...
MINIMAX_API_KEY=sk-...
OPENAI_API_KEY=sk-...

# Services
TELEGRAM_BOT_TOKEN=...
GMAIL_CLIENT_ID=...
GMAIL_CLIENT_SECRET=...

# Database
DATABASE_URL=file:prime-agent.db

# Server
PORT=3901
```

### Brain Config (Database)
- `provider` — LLM provider to use
- `model` — Model name
- `systemPrompt` — Custom system prompt
- `disabledTools` — Tools to disable

## Checkpoint & Recovery

The graph uses **LangGraph checkpointer** with LibSQL:

1. **Every turn** is checkpointed
2. **Dangling tool calls** (crashed mid-execution) are repaired
3. **Error 2013** triggers history rebuild
4. **Thread resumption** works across restarts

## Security

- Tools have full local access (intentional for autonomous operation)
- No external tool execution sandbox
- Telegram bot token required for bot features
- Gmail requires OAuth setup

---

*Last updated: 2026-02-28 (rev 4)*
