# Tools Reference

## Complete Tool Definitions

### bash_exec

```typescript
type bash_exec = (command: string, workdir?: string) => string;
```

Execute bash shell commands on the local machine.

**Parameters**:
- `command` (required): The bash command to execute
- `workdir` (optional): Working directory for the command

**Returns**: Command output (up to 8000 chars)

**Errors**: 
- Command timeouts after 30 seconds
- Permission errors

**Examples**:
```bash
# Read file
cat /path/to/file.txt

# Write file
echo "content" > file.txt

# Create directory
mkdir -p new/folder

# Git operations
git add .
git commit -m "message"
git push

# Run npm/pnpm
pnpm install
pnpm run build

# Check processes
ps aux | grep node
```

---

### doc_to_pdf

```typescript
type doc_to_pdf = (inputPath: string, outputPath?: string) => string;
```

Convert Markdown or AsciiDoc to PDF.

**Parameters**:
- `inputPath` (required): Path to source file (.md, .markdown, .adoc, .asciidoc)
- `outputPath` (optional): Output PDF path (defaults to same directory)

**Returns**: Path to generated PDF

**Examples**:
```
doc_to_pdf(inputPath="/docs/guide.md")
doc_to_pdf(inputPath="/docs/readme.adoc", outputPath="/docs/readme.pdf")
```

---

### web_search

```typescript
type web_search = (query: string, count?: number) => SearchResult[];
```

Search the web for current information.

**Parameters**:
- `query` (required): Search query
- `count` (optional): Number of results (1-10, default 5)

**Returns**: Array of search results with title, url, snippet

**Examples**:
```
web_search(query="AutoGen framework tutorial")
web_search(query="latest TypeScript features", count=10)
```

---

### gmail_read

```typescript
type gmail_read = (options?: {
  labelIds?: string[];
  maxResults?: number;
  query?: string;
  downloadAttachmentsTo?: string;
}) => Email[];
```

Read recent Gmail messages.

**Parameters**:
- `labelIds` (optional): Gmail labels (default: INBOX, UNREAD)
- `maxResults` (optional): Max emails (default 10)
- `query` (optional): Gmail search query
- `downloadAttachmentsTo` (optional): Directory for attachments

**Returns**: Array of emails with sender, subject, date, preview

**Examples**:
```
gmail_read(maxResults=5)
gmail_read(query="from:boss@company.com subject:report")
gmail_read(labelIds=["INBOX"], downloadAttachmentsTo="/tmp")
```

---

### gmail_send

```typescript
type gmail_send = (options: {
  to: string;
  subject: string;
  body: string;
  cc?: string;
  attachmentPaths?: string[];
}) => EmailSendResult;
```

Send email via Gmail.

**Parameters**:
- `to` (required): Recipient email
- `subject` (required): Email subject
- `body` (required): Plain text body
- `cc` (optional): CC recipients
- `attachmentPaths` (optional): File paths to attach

**Returns**: Send confirmation

**Examples**:
```
gmail_send(to="user@example.com", subject="Hello", body="Hi there!")
gmail_send(to="team@company.com", subject="Report", body="...", attachmentPaths=["/tmp/report.pdf"])
```

---

### telegram_send

```typescript
type telegram_send = (options: {
  message: string;
  chatId?: string;
  filePath?: string;
}) => MessageResult;
```

Send Telegram message.

**Parameters**:
- `message` (required): Message text
- `chatId` (optional): Chat ID (uses default)
- `filePath` (optional): File to attach

**Returns**: Sent message confirmation

**Examples**:
```
telegram_send(message="Task complete!")
telegram_send(message="Here's the file", filePath="/tmp/doc.pdf")
```

---

### spawn_subagent

```typescript
type spawn_subagent = (options: {
  task: string;
  modelOverride?: string;
}) => SubAgentResult;
```

Delegate to a sub-agent for independent tasks.

**Parameters**:
- `task` (required): Task description
- `modelOverride` (optional): Override the brain model

**Returns**: Sub-agent's final response

**Rules**:
- One level only (no recursive spawning)
- Use for self-contained, well-defined subtasks
- Don't use for simple questions

**Examples**:
```
spawn_subagent(task="Research quantum computing basics and summarize in 3 paragraphs")
spawn_subagent(task="Find and list all TypeScript errors in the codebase", modelOverride="ollama/llama3")
```

---

### transcribe_audio

```typescript
type transcribe_audio = (filePath: string, language?: string) => string;
```

Transcribe audio to text using Whisper.

**Parameters**:
- `filePath` (required): Audio file path
- `language` (optional): BCP-47 language code

**Supported formats**: .ogg, .mp3, .wav, .m4a, .flac, .mp4, .mpeg, .webm

**Returns**: Transcribed text

**Examples**:
```
transcribe_audio(filePath="/path/to/voice.ogg")
transcribe_audio(filePath="/tmp/interview.mp3", language="en")
```

---

## Error Handling

Tools can fail. Common errors:

| Error | Cause | Solution |
|-------|-------|----------|
| `Command timed out` | Command took >30s | Break into smaller commands |
| `Permission denied` | No file access | Check permissions |
| `Module not found` | Package not installed | Install dependencies |
| `API rate limit` | Too many requests | Wait and retry |
| `File not found` | Wrong path | Verify file exists |

---

## Best Practices

1. **Use right tool for job** — bash for files, web_search for info, etc.
2. **Check before overwrite** — Use `ls` first
3. **Handle errors** — Tools return errors, don't assume success
4. **Be specific** — Exact paths, clear queries
5. **Stay local** — Prefer local tools over external APIs when possible

---

*Last updated: 2025-02-28*
