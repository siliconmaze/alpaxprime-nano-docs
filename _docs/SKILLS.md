# LPEX Prime Nano — Skills & Tools

## How I Work

I'm an autonomous AI co-pilot. When you send me a message:

1. **Receive** — Get your message via Telegram, API, or UI
2. **Understand** — LLM processes the request
3. **Plan** — Decide which tools to use (if any)
4. **Execute** — Run tools sequentially
5. **Respond** — Return the result

## Available Tools

### 1. bash_exec
**Purpose**: Run bash commands on the local machine

**Capabilities**:
- Read/write files
- Run scripts
- Check system state
- Process data
- Create directories
- Git operations
- npm/pnpm commands

**Example Usage**:
```bash
# List files
ls -la

# Create file
echo "hello" > file.txt

# Run a script
./deploy.sh

# Git operations
git status
git commit -m "fix bug"
```

### 2. doc_to_pdf
**Purpose**: Convert Markdown/AsciiDoc to PDF

**Parameters**:
- `inputPath` — Source file (.md, .adoc)
- `outputPath` — Optional output path

**Example**:
```
doc_to_pdf(inputPath="/path/to/document.md", outputPath="/path/to/output.pdf")
```

### 3. web_search
**Purpose**: Search the web for current information

**Parameters**:
- `query` — Search query
- `count` — Number of results (1-10)

**Example**:
```
web_search(query="latest AI news", count=5)
```

### 4. gmail_read
**Purpose**: Read recent Gmail messages

**Parameters**:
- `labelIds` — Gmail labels to filter
- `maxResults` — Max messages to return
- `query` — Gmail search query
- `downloadAttachmentsTo` — Local directory for attachments

**Example**:
```
gmail_read(maxResults=10, query="from:boss@company.com")
```

### 5. gmail_send
**Purpose**: Send email via Gmail

**Parameters**:
- `to` — Recipient email
- `subject` — Email subject
- `body` — Email body (plain text)
- `cc` — CC recipients
- `attachmentPaths` — File paths to attach

**Example**:
```
gmail_send(to="user@example.com", subject="Report", body="Here is the report...", attachmentPaths=["/path/to/file.pdf"])
```

### 6. telegram_send
**Purpose**: Send Telegram message

**Parameters**:
- `message` — Message text
- `chatId` — Optional chat ID (uses default)
- `filePath` — Optional file to attach

**Example**:
```
telegram_send(message="Task complete!", filePath="/path/to/report.pdf")
```

### 7. spawn_subagent
**Purpose**: Delegate a subtask to a separate agent

**Parameters**:
- `task` — Description of the subtask
- `modelOverride` — Optional model to use

**Example**:
```
spawn_subagent(task="Research the latest AutoGen features and summarize")
```

**Rules**:
- One level only (no recursive spawning)
- Use for independent, well-defined subtasks
- Don't use for simple questions

### 8. transcribe_audio
**Purpose**: Transcribe audio to text

**Parameters**:
- `filePath` — Audio file path (.ogg, .mp3, .wav, etc.)
- `language` — Optional language hint

**Example**:
```
transcribe_audio(filePath="/path/to/voice.ogg", language="en")
```

## Skills by Domain

### Content Engine 🛠️
- Writing blog posts
- Creating course outlines
- Drafting documentation
- Content strategy

### Build Engine ⚙️
- Writing code
- Managing repos
- DevOps tasks
- Kubernetes deployments
- Code reviews

### Operations Engine 📊
- Monitoring system health
- Managing alerts
- Running cron jobs
- Database operations

## Conversation Context

I maintain context across messages using **LangGraph checkpoints**:
- Thread ID groups related conversations
- Full history is persisted in LibSQL
- Can resume from any point

## System Prompt

```
You are Alpax Prime 🦙 — a focused, practical AI co-pilot.

Your available tools:
- bash_exec: run any bash command on the local machine
- doc_to_pdf: convert Markdown/AsciiDoc to PDF
- web_search: search the web
- gmail_read: read Gmail
- gmail_send: send email
- telegram_send: send Telegram messages
- spawn_subagent: delegate to sub-agent

Rules:
- Use bash_exec freely for local tasks
- Use web_search for current events
- Only use tools when needed
- Don't spawn sub-agents recursively
- Keep responses concise
```

## Task Routing

When you ask me to create something, I typically:

1. **Analyze** — Understand what you want
2. **Plan** — Decide which tools needed
3. **Execute** — Run commands in order
4. **Verify** — Check the result
5. **Report** — Summarize what was done

---

*Last updated: 2025-02-28*
