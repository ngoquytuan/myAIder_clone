# AIDER - BÁO CÁO TỔNG QUAN VÀ KIẾN TRÚC

## Tổng Quan Dự Án

**Aider** là một công cụ AI pair programming chạy trong terminal, cho phép lập trình viên làm việc cùng với các mô hình ngôn ngữ lớn (LLM) để code, refactor và sửa lỗi.

### Thông Tin Cơ Bản

- **Tên dự án:** Aider (AI Pair Programming)
- **Repository:** https://github.com/Aider-AI/aider
- **Ngôn ngữ chính:** Python 3.10+
- **License:** Apache 2.0
- **Tổng số dòng code:** ~13,276 dòng trong package chính
- **Số file Python:** 79+ files
- **Hỗ trợ ngôn ngữ:** 100+ ngôn ngữ lập trình

### Mục Đích và Tính Năng Chính

**Mục đích:**
- Pair programming với AI trực tiếp trong terminal
- Tự động commit code với git messages thông minh
- Hỗ trợ nhiều định dạng editing khác nhau
- Tích hợp sâu với git workflow
- Làm việc với codebase lớn thông qua repo mapping

**Tính năng nổi bật:**
1. **Multi-LLM Support:** Hỗ trợ 50+ models (Claude, GPT-4, DeepSeek, v.v.)
2. **Repository Mapping:** Tự động map toàn bộ codebase
3. **Git Integration:** Auto commit với AI-generated messages
4. **Multiple Edit Formats:** 9 định dạng edit khác nhau
5. **Voice Coding:** Code bằng giọng nói
6. **Watch Mode:** Tự động xử lý code comments
7. **Linting/Testing:** Tích hợp tự động lint và test
8. **Web Scraping:** Fetch documentation từ web

---

## Kiến Trúc Hệ Thống

### 1. Cấu Trúc Thư Mục

```
/home/user/myAIder_clone/aider/
├── aider/                          # Package chính
│   ├── coders/                     # Các implementation của edit formats
│   │   ├── __init__.py
│   │   ├── base_coder.py          # Base class (2,485 dòng)
│   │   ├── editblock_coder.py     # Search/replace format
│   │   ├── wholefile_coder.py     # Whole file replacement
│   │   ├── udiff_coder.py         # Unified diff format
│   │   ├── architect_coder.py     # Two-model collaboration
│   │   └── ...                    # Các formats khác
│   │
│   ├── resources/                  # Configuration files
│   │   ├── model-settings.yml     # Cấu hình models
│   │   └── model-metadata.json    # Metadata của models
│   │
│   ├── queries/                    # Tree-sitter queries
│   │   ├── tree-sitter-languages/
│   │   └── tree-sitter-language-pack/
│   │
│   ├── website/                    # Documentation site
│   │
│   ├── main.py                     # Entry point (1,274 dòng)
│   ├── models.py                   # Model management (1,302 dòng)
│   ├── commands.py                 # Slash commands (1,694 dòng)
│   ├── io.py                       # Input/Output (1,191 dòng)
│   ├── repo.py                     # Git operations (621 dòng)
│   ├── repomap.py                  # Repo mapping (848 dòng)
│   ├── linter.py                   # Linting (304 dòng)
│   ├── args.py                     # Argument parsing (945 dòng)
│   └── ...                         # Support modules
│
├── tests/                          # Test suite
│   ├── basic/                      # Core tests
│   ├── fixtures/                   # Test data
│   │   └── languages/              # 40+ language fixtures
│   └── ...
│
├── benchmark/                      # Benchmarking tools
├── scripts/                        # Utility scripts
├── requirements/                   # Dependencies
├── pyproject.toml                  # Project config
└── requirements.txt                # Main dependencies
```

### 2. Kiến Trúc Pattern

#### A. Strategy Pattern (Chính)

Aider sử dụng **Strategy Pattern** với Factory Method để xử lý các định dạng edit khác nhau.

**Factory Method** (base_coder.py:124-201):
```python
@classmethod
def create(cls, main_model=None, edit_format=None, io=None, **kwargs):
    """Factory method chọn coder phù hợp dựa trên edit_format"""
    for coder in coders.__all__:
        if hasattr(coder, "edit_format") and coder.edit_format == edit_format:
            return coder(main_model, io, **kwargs)
```

**Các Strategy (Edit Formats):**

1. **EditBlockCoder** - `diff` format
   - Search/replace blocks
   - Đáng tin cậy nhất cho edits phức tạp
   - Location: `coders/editblock_coder.py`

2. **WholeFileCoder** - `whole` format
   - Thay thế toàn bộ file
   - Tốt cho file mới
   - Location: `coders/wholefile_coder.py`

3. **UnifiedDiffCoder** - `udiff` format
   - Standard unified diff patches
   - Location: `coders/udiff_coder.py`

4. **ArchitectCoder** - `architect` format
   - Hai model làm việc cùng nhau
   - Location: `coders/architect_coder.py`

5. **ContextCoder** - `context` format
   - Tự động chọn files
   - Location: `coders/context_coder.py`

6. **AskCoder** - `ask` format
   - Read-only queries
   - Location: `coders/ask_coder.py`

7-9. Các formats khác: PatchCoder, EditorEditBlockCoder, HelpCoder

#### B. Observer Pattern

**FileWatcher** (watch.py:65-318):
- Monitor file changes
- Detect AI comments trong code
- Trigger tự động AI actions

#### C. Command Pattern

**Commands System** (commands.py:36-1694):
- Tất cả slash commands
- Interface nhất quán cho user actions
- Dễ dàng extend với commands mới

#### D. Lazy Loading

**LiteLLM Integration** (llm.py:21-45):
- Load LLM library on-demand
- Giảm startup time từ ~1.5s xuống milliseconds

### 3. Core Components Chi Tiết

#### A. Coder System (Base Class)

**Location:** `aider/coders/base_coder.py:88-2485`

**Responsibilities:**
- Main interaction loop
- LLM communication
- Code change application
- Error handling và recovery
- Git commit automation

**Key Methods:**
- `run()` - Main loop (line 876)
- `send_message()` - Giao tiếp với LLM
- `apply_updates()` - Apply code changes
- `get_edits()` - Parse LLM responses (abstract)
- `format_messages()` - Chuẩn bị prompts

#### B. Model Management

**Location:** `aider/models.py:~300+`

**Responsibilities:**
- Model registry và configuration
- Token counting và cost tracking
- API request handling
- Support 50+ LLM models

**Supported Providers:**
- OpenAI (GPT-4o, o1, o3-mini)
- Anthropic (Claude 3.x, Claude 4)
- DeepSeek (Chat, Reasoner)
- Google (Gemini)
- OpenRouter (100+ models)
- Local models (Ollama, LM Studio)

**Model Aliases:**
```python
"sonnet" → "anthropic/claude-sonnet-4-20250514"
"4o" → "gpt-4o"
"deepseek" → "deepseek/deepseek-chat"
"opus" → "claude-opus-4-20250514"
```

#### C. Repository Management

**Location:** `aider/repo.py:52-621`

**Responsibilities:**
- Git operations wrapper
- Automatic commit generation
- File tracking và diffing
- .aiderignore support

**Key Methods:**
- `commit()` - Create commits (line 131)
- `get_tracked_files()` - List repo files
- `diff_commits()` - Generate diffs

#### D. Repository Mapping

**Location:** `aider/repomap.py:41-848`

**Công nghệ:**
- Tree-sitter parsing cho 100+ languages
- SQLite caching
- Intelligent file ranking

**Algorithm:**
1. Parse code với tree-sitter
2. Extract classes/functions
3. Rank theo relevance
4. Fit within token budget
5. Cache results

**Purpose:**
- Provide context về codebase
- Help LLM hiểu structure
- Optimize token usage

#### E. Input/Output System

**Location:** `aider/io.py:~200+`

**Technologies:**
- prompt_toolkit - Advanced input
- Rich - Markdown rendering
- Pygments - Syntax highlighting

**Features:**
- Multiline editing
- Autocomplete
- Markdown rendering
- File operations
- Notifications

#### F. Commands System

**Location:** `aider/commands.py:36-1694`

**Available Commands:**
- `/add` - Add files to chat
- `/drop` - Remove files
- `/commit` - Create git commit
- `/undo` - Undo last change
- `/model` - Switch models
- `/architect` - Enable architect mode
- `/ask` - Switch to ask mode
- `/code` - Switch to code mode
- `/help` - Show help
- `/ls` - List files
- `/git` - Run git commands
- `/run` - Run shell commands
- `/voice` - Enable voice input
- ... và nhiều commands khác

---

## Flow Execution

### Application Startup Flow

```
1. ENTRY POINT
   __main__.py → main.main()

2. INITIALIZATION (main.py:451-740)
   ├─> Parse command-line arguments
   ├─> Load environment variables (.env)
   ├─> Detect git repository
   ├─> Load config files (.aider.conf.yml)
   ├─> Setup InputOutput system
   └─> Initialize analytics

3. MODEL SETUP (main.py:756-877)
   ├─> Register model metadata
   ├─> Select main model
   ├─> Configure weak model
   ├─> Setup editor model (architect)
   └─> Validate API keys

4. REPOSITORY SETUP (main.py:903-933)
   ├─> Initialize GitRepo
   ├─> Sanity check repository
   ├─> Load .aiderignore
   └─> Count tracked files

5. CODER CREATION (main.py:973-1017)
   ├─> Create Commands instance
   ├─> Create ChatSummary
   ├─> Coder.create() (Factory method)
   └─> Initialize RepoMap

6. OPTIONAL FEATURES
   ├─> FileWatcher (--watch-files)
   ├─> ClipboardWatcher (--copy-paste)
   └─> Voice input

7. MAIN LOOP (base_coder.py:876-893)
   └─> while True: get input → process → apply changes
```

### Request Processing Flow

```
USER INPUT
   ↓
[1] COMMAND DETECTION
   ├─> Slash command? → Execute command
   └─> Regular message → Continue
   ↓
[2] CONTEXT GATHERING
   ├─> Load file contents
   ├─> Generate repo map
   └─> Build chat history
   ↓
[3] PROMPT FORMATTING
   ├─> System prompt
   ├─> File contents
   ├─> Repo map
   ├─> Examples
   └─> User message
   ↓
[4] LLM COMMUNICATION
   ├─> Send request (via litellm)
   ├─> Stream response
   └─> Handle errors/retries
   ↓
[5] RESPONSE PARSING
   ├─> Extract code blocks
   ├─> Parse edit instructions
   └─> Identify files to modify
   ↓
[6] CODE APPLICATION
   ├─> Validate edits
   ├─> Apply changes to files
   ├─> Run linters (if auto_lint)
   └─> Run tests (if auto_test)
   ↓
[7] GIT COMMIT (if auto_commits)
   ├─> Stage changes
   ├─> Generate commit message (AI)
   └─> Create commit
   ↓
[8] DISPLAY RESULTS
   └─> Show diffs to user
```

---

## Integration Points

### 1. LLM Integration

**Primary Library:** LiteLLM
**Location:** `aider/llm.py`

**Flow:**
```
Application
    ↓
models.py (Model management)
    ↓
LiteLLM (Unified API)
    ↓
[OpenAI | Anthropic | DeepSeek | Google | OpenRouter | Local]
```

**Features:**
- Unified API cho tất cả providers
- Automatic retries
- Token counting
- Cost tracking
- Streaming support

### 2. Git Integration

**Library:** GitPython
**Location:** `aider/repo.py`

**Operations:**
- Repository detection
- File tracking
- Commit creation (with AI messages)
- Diff generation
- Branch operations
- Attribution management

### 3. Editor Integration

**Watch Mode** (watch.py):
```python
# Trong code, thêm comment:
# ai: add error handling

# Aider tự động detect và process
```

### 4. Voice Integration

**Library:** sounddevice + OpenAI Whisper
**Location:** `aider/voice.py`

**Features:**
- Voice activity detection
- Multiple audio formats
- Automatic transcription

### 5. Web Integration

**Scraping** (scrape.py):
- Playwright cho JS-heavy sites
- pypandoc cho HTML → Markdown
- Fetch documentation

---

## Dependencies

### Core Dependencies

**AI/ML:**
- `litellm` - Unified LLM API
- `openai` - OpenAI client
- `google-generativeai` - Google models
- `tiktoken` - Token counting

**Git:**
- `gitpython` - Git operations

**Parsing:**
- `tree-sitter` - Code parsing
- `grep-ast` - AST searching
- `beautifulsoup4` - HTML parsing

**Terminal UI:**
- `prompt-toolkit` - Advanced input
- `rich` - Markdown rendering
- `pygments` - Syntax highlighting

**Audio:**
- `sounddevice` - Audio recording
- `soundfile` - Audio file handling
- `pydub` - Audio processing

**Utilities:**
- `configargparse` - Config + args
- `diskcache` - Disk caching
- `networkx` - Graph operations
- `pyyaml` - YAML parsing
- `jsonschema` - JSON validation

### Optional Dependencies

**Browser:**
- `playwright` - Browser automation

**Development:**
- `pytest` - Testing
- `flake8` - Linting
- `pre-commit` - Git hooks

---

## Performance Considerations

### 1. Lazy Loading
- LiteLLM loaded on-demand
- Startup time: milliseconds thay vì seconds

### 2. Caching
- RepoMap caching với SQLite
- Disk cache cho expensive operations
- Prompt caching support

### 3. Token Optimization
- Smart context selection
- Dynamic token budgeting
- Chat history summarization
- Repository map pruning

### 4. Streaming
- Stream LLM responses
- Real-time display
- Lower perceived latency

---

## Security Considerations

### 1. API Key Management
- Load từ `.env` files
- Environment variables
- Never commit keys
- OAuth support

### 2. Git Safety
- Automatic .gitignore
- .aiderignore support
- Safe commit practices

### 3. Code Execution
- Linting before commit
- Test running support
- User confirmation for dangerous operations

---

## Summary Statistics

- **Total Python Files:** 79+
- **Total Lines:** ~13,276
- **Largest Module:** base_coder.py (2,485 lines)
- **Edit Formats:** 9 strategies
- **Supported Models:** 50+
- **Supported Languages:** 100+
- **Test Files:** 20+
- **Configuration Types:** 3 (YAML, JSON, ENV)

---

## Key Architectural Strengths

1. ✅ **Modularity:** Separation of concerns rõ ràng
2. ✅ **Extensibility:** Dễ thêm formats, commands, models
3. ✅ **Robustness:** Error handling và retry logic tốt
4. ✅ **Performance:** Lazy loading, caching, optimization
5. ✅ **Flexibility:** Multiple edit formats cho use cases khác nhau
6. ✅ **Integration:** Deep git integration
7. ✅ **UX:** Rich terminal UI với feedback tốt
8. ✅ **Testing:** Comprehensive test coverage

---

## Điểm Yếu và Cơ Hội Cải Thiện

### Điểm Yếu
1. **Complexity:** Codebase khá phức tạp cho người mới
2. **Documentation:** In-code comments có thể nhiều hơn
3. **Type Hints:** Chưa consistent 100%
4. **Testing:** Coverage có thể cao hơn

### Cơ Hội Cải Thiện
1. **Plugin System:** Formal plugin architecture
2. **API:** Public Python API cho programmatic usage
3. **IDE Extensions:** Official plugins cho VSCode, PyCharm
4. **Cloud Integration:** Cloud-based caching/sharing
5. **Team Features:** Multi-user collaboration

---

**Generated:** 2025-11-13
**Aider Version:** Latest from GitHub
**Analysis Tool:** Claude Code Agent
