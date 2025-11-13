# AIDER - BÁO CÁO PHÂN TÍCH CÁC COMPONENT CHI TIẾT

## Mục Lục
1. [Base Coder System](#1-base-coder-system)
2. [Edit Format Implementations](#2-edit-format-implementations)
3. [Model Management](#3-model-management)
4. [Repository Operations](#4-repository-operations)
5. [Repository Mapping](#5-repository-mapping)
6. [IO System](#6-io-system)
7. [Commands System](#7-commands-system)
8. [Configuration System](#8-configuration-system)

---

## 1. Base Coder System

### File: `aider/coders/base_coder.py` (2,485 dòng)

### Mục Đích
Base class cho tất cả các coders. Cung cấp core functionality cho interaction với LLM và file management.

### Class Hierarchy

```python
Coder (base)
├── EditBlockCoder (diff format)
├── WholeFileCoder (whole format)
├── UnifiedDiffCoder (udiff format)
├── PatchCoder (patch format)
├── ArchitectCoder (architect format)
├── ContextCoder (context format)
├── AskCoder (ask format)
├── EditorEditBlockCoder (editor-diff format)
└── HelpCoder (help format)
```

### Key Attributes

```python
class Coder:
    # Core
    main_model: Model           # Primary LLM
    io: InputOutput             # I/O handler
    repo: GitRepo              # Git operations

    # Files
    abs_fnames: set            # Files in chat
    abs_read_only_fnames: set  # Read-only files

    # State
    cur_messages: list         # Chat history
    done_messages: list        # Completed messages

    # Configuration
    auto_commits: bool         # Auto git commit
    auto_lint: bool           # Auto lint after changes
    auto_test: bool           # Auto test after changes
    edit_format: str          # Edit format name
```

### Key Methods

#### A. Initialization
```python
def __init__(self, main_model, io, **kwargs):
    """
    Initialize coder instance
    Location: line ~200
    """
    - Setup model
    - Initialize file sets
    - Load configuration
    - Setup git repo
    - Initialize chat history
```

#### B. Main Loop
```python
def run(self, with_message=None):
    """
    Main interaction loop
    Location: line 876

    Flow:
    1. Get user input
    2. Process message
    3. Apply changes
    4. Handle errors
    5. Repeat
    """
```

#### C. Message Handling
```python
def send_message(self, inp):
    """
    Send message to LLM
    Location: line ~1100

    Steps:
    1. Format prompt with context
    2. Send to LLM API
    3. Stream response
    4. Parse output
    5. Apply edits
    """
```

#### D. Edit Application
```python
def apply_updates(self, edited):
    """
    Apply code changes to files
    Location: line ~1500

    Process:
    1. Parse edits from LLM
    2. Validate changes
    3. Apply to files
    4. Run linters
    5. Run tests
    6. Git commit (if auto_commits)
    """
```

#### E. Prompt Formatting
```python
def format_messages(self):
    """
    Format messages for LLM
    Location: line ~1800

    Components:
    - System prompt
    - File contents
    - Repository map
    - Few-shot examples
    - User message
    """
```

### Abstract Methods (Must Implement)

```python
@abstractmethod
def get_edits(self):
    """Parse edits from LLM response"""
    pass

@abstractmethod
def apply_edits(self, edits):
    """Apply parsed edits to files"""
    pass
```

### Error Handling

**Reflection Mechanism** (line ~2000):
```python
def reflect_on_error(self, error):
    """
    Nếu LLM tạo invalid edits:
    1. Detect error type
    2. Generate explanation
    3. Ask LLM to fix
    4. Retry (max 3 times)
    """
```

### Git Integration

```python
def auto_commit(self, edited):
    """
    Automatic git commit
    Location: line ~2200

    Steps:
    1. Stage changed files
    2. Generate commit message (LLM)
    3. Create commit
    4. Handle attribution
    """
```

---

## 2. Edit Format Implementations

### A. EditBlockCoder (diff format)

**File:** `aider/coders/editblock_coder.py` (657 dòng)

**Format:**
```
<<<<<<< SEARCH
old code
=======
new code
>>>>>>> REPLACE
```

**Strengths:**
- Most reliable
- Good for complex edits
- Fuzzy matching support
- Clear intent

**Implementation:**
```python
def get_edits(self):
    """
    Line 21-36

    Parse search/replace blocks:
    1. Find SEARCH markers
    2. Extract old code
    3. Find REPLACE markers
    4. Extract new code
    5. Match to files
    """

def apply_edits(self, edits):
    """
    Line 41-74

    Apply with fuzzy matching:
    1. Find best match in file
    2. Calculate similarity
    3. Apply if threshold met
    4. Report conflicts
    """
```

**Use Cases:**
- Complex multi-line edits
- Refactoring
- When precision needed

### B. WholeFileCoder (whole format)

**File:** `aider/coders/wholefile_coder.py` (144 dòng)

**Format:**
```markdown
path/to/file.py
```python
entire file content
```
```

**Strengths:**
- Simple format
- Good for new files
- No ambiguity

**Implementation:**
```python
def get_edits(self):
    """
    Line 22-96

    Parse file blocks:
    1. Find file path markers
    2. Extract code blocks
    3. Match language
    4. Build file content
    """
```

**Use Cases:**
- Creating new files
- Complete rewrites
- Small files

### C. UnifiedDiffCoder (udiff format)

**File:** `aider/coders/udiff_coder.py` (429 dòng)

**Format:**
```diff
--- a/file.py
+++ b/file.py
@@ -1,3 +1,3 @@
-old line
+new line
```

**Strengths:**
- Standard format
- Tool compatible
- Precise line numbers

**Implementation:**
```python
def get_edits(self):
    """
    Parse unified diff format
    Using diff-match-patch library
    """

def apply_edits(self, edits):
    """
    Apply patches with validation
    Handle line number shifts
    """
```

**Use Cases:**
- Small targeted changes
- When line precision needed
- Integration with other tools

### D. ArchitectCoder (architect format)

**File:** `aider/coders/architect_coder.py` (48 dòng)

**Concept:**
Two-model collaboration:
1. **Architect Model:** Plans changes
2. **Editor Model:** Implements changes

**Flow:**
```python
def run_with_architect(self):
    """
    1. User request → Architect
    2. Architect creates plan
    3. Plan → Editor model
    4. Editor implements
    5. Verify completion
    6. Repeat if needed
    """
```

**Use Cases:**
- Complex features
- Need planning
- Large codebases

### E. ContextCoder (context format)

**File:** `aider/coders/context_coder.py`

**Purpose:**
Automatically select relevant files

**Flow:**
```python
1. Analyze user request
2. Search codebase
3. Select relevant files
4. Add to context
5. Process normally
```

**Use Cases:**
- User doesn't know which files
- Large codebases
- Exploration

### F. AskCoder (ask format)

**File:** `aider/coders/ask_coder.py`

**Purpose:**
Read-only queries about code

**Features:**
- No file editing
- Only read operations
- Good for learning
- Answer questions

**Use Cases:**
- Understanding code
- Documentation
- Code review

---

## 3. Model Management

### File: `aider/models.py` (1,302 dòng)

### Model Class

```python
class Model:
    """
    Represents an LLM model

    Attributes:
    - name: str               # Model identifier
    - edit_format: str        # Preferred edit format
    - weak_model_name: str    # Simpler model for easy tasks
    - max_input_tokens: int   # Context window
    - max_output_tokens: int  # Max output
    - input_cost_per_token: float
    - output_cost_per_token: float
    - use_repo_map: bool      # Use repo mapping
    - streaming: bool         # Support streaming
    """
```

### Model Registry

**Configuration:** `aider/resources/model-settings.yml`

**Example Entry:**
```yaml
- name: gpt-4o
  edit_format: diff
  weak_model_name: gpt-4o-mini
  use_repo_map: true
  lazy: true
  streaming: true
  max_input_tokens: 128000
  max_output_tokens: 16384
```

### Key Functions

#### A. Model Registration
```python
def register_models(model_settings_fnames):
    """
    Line 135-141

    Load model configs:
    1. Read YAML files
    2. Parse settings
    3. Register in registry
    4. Setup aliases
    """
```

#### B. Model Selection
```python
def select_default_model(args):
    """
    Select model based on:
    1. Command line arg
    2. Environment variable
    3. Config file
    4. Default (sonnet)
    """
```

#### C. Token Counting
```python
def count_tokens(text, model):
    """
    Use tiktoken for accurate counting
    Important for cost tracking
    """
```

#### D. Cost Tracking
```python
def calculate_cost(input_tokens, output_tokens, model):
    """
    input_cost = input_tokens * model.input_cost_per_token
    output_cost = output_tokens * model.output_cost_per_token
    total = input_cost + output_cost
    """
```

### Supported Models

**OpenAI:**
- GPT-4o (128K context)
- GPT-4o-mini (128K context)
- o1-preview (128K context)
- o1-mini (128K context)
- o3-mini (200K context)

**Anthropic:**
- Claude Sonnet 4 (200K context)
- Claude Opus 4 (200K context)
- Claude 3.5 Sonnet (200K context)
- Claude 3.5 Haiku (200K context)

**DeepSeek:**
- DeepSeek Chat V3 (64K context)
- DeepSeek Reasoner R1 (64K context)

**Google:**
- Gemini 2.0 Flash (1M context)
- Gemini 1.5 Pro (2M context)

**Local:**
- Ollama models
- LM Studio models
- vLLM models

### Model Aliases

```python
MODEL_ALIASES = {
    "sonnet": "anthropic/claude-sonnet-4-20250514",
    "opus": "claude-opus-4-20250514",
    "4o": "gpt-4o",
    "deepseek": "deepseek/deepseek-chat",
    "gemini": "gemini/gemini-2.0-flash-exp",
}
```

---

## 4. Repository Operations

### File: `aider/repo.py` (621 dòng)

### GitRepo Class

```python
class GitRepo:
    """
    Wrapper around GitPython

    Attributes:
    - repo: git.Repo          # GitPython repo
    - root: Path              # Repo root
    - aider_ignore: set       # Ignored patterns
    """
```

### Key Operations

#### A. Repository Detection
```python
def find_git_repo(path="."):
    """
    Line ~50

    Search for .git directory:
    1. Check current directory
    2. Check parent directories
    3. Return root path
    """
```

#### B. File Tracking
```python
def get_tracked_files(self):
    """
    Line ~150

    Get all tracked files:
    1. Run git ls-files
    2. Filter .aiderignore
    3. Return set of paths
    """
```

#### C. Commit Creation
```python
def commit(self, fnames, context, message=None):
    """
    Line 131-200

    Create git commit:
    1. Stage specified files
    2. Generate message (LLM if not provided)
    3. Create commit
    4. Handle attribution

    Attribution options:
    - --attribute-author
    - --attribute-committer
    - --attribute-co-authored-by
    """
```

#### D. Commit Message Generation
```python
def get_commit_message(self, diffs, context):
    """
    Use LLM to generate:
    1. Analyze diffs
    2. Summarize changes
    3. Follow conventional commits
    4. Keep under 50 chars (title)
    """
```

#### E. Diff Operations
```python
def diff_commits(self, from_commit, to_commit):
    """
    Generate diff between commits
    Used for undo/redo
    """

def get_diffs(self, fnames):
    """
    Get current working tree diffs
    """
```

#### F. Ignore Patterns
```python
def load_aiderignore(self):
    """
    Load .aiderignore:
    1. Read .aiderignore file
    2. Parse patterns
    3. Apply to file operations

    Format similar to .gitignore:
    *.log
    temp/
    __pycache__/
    """
```

### Attribution

**Options:**
```bash
--attribute-author      # Set AI as author
--attribute-committer   # Set AI as committer
--attribute-co-authored-by  # Add Co-authored-by
```

**Result:**
```
Author: AI Assistant <ai@aider.chat>
Co-authored-by: Human Developer <human@example.com>
```

---

## 5. Repository Mapping

### File: `aider/repomap.py` (848 dòng)

### Purpose
Tạo "map" của codebase để LLM hiểu structure mà không cần load toàn bộ code.

### RepoMap Class

```python
class RepoMap:
    """
    Repository code map generator

    Uses:
    - Tree-sitter for parsing
    - SQLite for caching
    - Graph algorithms for ranking
    """
```

### How It Works

#### 1. Code Parsing
```python
def parse_file(self, fname):
    """
    Line ~200

    Use tree-sitter:
    1. Detect language
    2. Load grammar
    3. Parse to AST
    4. Extract definitions

    Extracts:
    - Class definitions
    - Function definitions
    - Method definitions
    - Important variables
    """
```

#### 2. Tag Extraction
```python
def get_tags(self, fname):
    """
    Line ~300

    Extract tags:
    - ClassName
    - ClassName.method_name
    - function_name
    - module.function_name

    Format:
    {
        "file": "path/to/file.py",
        "line": 42,
        "name": "MyClass.method",
        "kind": "method"
    }
    """
```

#### 3. Ranking Algorithm
```python
def rank_tags(self, tags, chat_fnames):
    """
    Line ~400

    Rank by relevance:
    1. Tags in chat files = highest
    2. Tags mentioned in chat = high
    3. Tags in same directory = medium
    4. Tags in imported files = medium
    5. Other tags = low

    Uses networkx for graph analysis
    """
```

#### 4. Map Generation
```python
def get_repo_map(self, chat_fnames, max_tokens=1024):
    """
    Line 102

    Generate map:
    1. Parse all files
    2. Extract tags
    3. Rank by relevance
    4. Fit in token budget
    5. Format as map

    Output format:
    ```
    src/main.py:
    ├── class Application
    │   ├── __init__(config)
    │   ├── run()
    │   └── shutdown()
    └── main()

    src/utils.py:
    ├── helper_function(arg1, arg2)
    └── another_helper()
    ```
    """
```

### Caching Strategy

**SQLite Cache:**
```python
def init_cache(self):
    """
    Cache tables:
    - file_hashes: Track file changes
    - parsed_tags: Store extracted tags
    - rankings: Store ranking results

    Invalidation:
    - On file modification
    - On git commit
    """
```

**Disk Cache:**
```python
# Using diskcache library
cache = diskcache.Cache('.aider.cache')
```

### Tree-sitter Integration

**Supported Languages:** 100+

**Query Files:** `aider/queries/tree-sitter-languages/`

**Example Query (Python):**
```scheme
(class_definition
  name: (identifier) @name)

(function_definition
  name: (identifier) @name)

(decorated_definition
  definition: (function_definition
    name: (identifier) @name))
```

### Performance

**Optimizations:**
- Parse only changed files
- SQLite caching
- Lazy loading
- Parallel processing (planned)

**Benchmarks:**
- Small project (<100 files): <1s
- Medium project (<1000 files): <5s
- Large project (<10000 files): <30s

---

## 6. IO System

### File: `aider/io.py` (1,191 dòng)

### InputOutput Class

```python
class InputOutput:
    """
    Terminal I/O handler

    Features:
    - Fancy prompts (prompt_toolkit)
    - Markdown rendering (rich)
    - Syntax highlighting (pygments)
    - Autocomplete
    - History
    """
```

### Components

#### A. Input Handling
```python
def get_input(self, prompt="", multiline=False):
    """
    Line ~300

    Features:
    - Multiline editing (Ctrl+E)
    - History (up/down arrows)
    - Autocomplete (Tab)
    - Syntax highlighting
    - Vi/Emacs keybindings

    Powered by prompt_toolkit
    """
```

#### B. Autocomplete
```python
class Autocomplete(Completer):
    """
    Line ~400

    Autocomplete for:
    - Slash commands (/add, /commit, etc.)
    - File paths
    - Model names
    - Git branches
    """
```

#### C. Output Formatting
```python
def tool_output(self, text, style="info"):
    """
    Line ~500

    Formatted output:
    - info: Blue
    - warning: Yellow
    - error: Red
    - success: Green

    Uses rich library
    """
```

#### D. Markdown Rendering
```python
def render_markdown(self, text):
    """
    Line ~600

    Render with rich:
    - Headers
    - Lists
    - Code blocks (with syntax highlighting)
    - Links
    - Bold/Italic
    """
```

#### E. File Operations
```python
def read_text(self, fname):
    """Read file with encoding detection"""

def write_text(self, fname, content):
    """Write file with backup"""
```

#### F. User Confirmations
```python
def confirm_ask(self, question, default="y"):
    """
    Yes/no questions

    Examples:
    - Confirm file overwrite?
    - Create git commit?
    - Run tests?
    """
```

### Terminal Features

**Rich Rendering:**
- Colors (256 colors, true color)
- Tables
- Progress bars
- Panels
- Trees

**Prompt Features:**
- Vi mode
- Emacs mode
- Custom keybindings
- Mouse support

---

## 7. Commands System

### File: `aider/commands.py` (1,694 dòng)

### Commands Class

```python
class Commands:
    """
    Slash command handler

    All commands start with /
    """
```

### Command Categories

#### A. File Management

```python
def cmd_add(self, args):
    """
    /add <files>

    Add files to chat context

    Features:
    - Glob patterns (*.py)
    - Multiple files
    - Directory recursion
    - Git tracked only
    """

def cmd_drop(self, args):
    """
    /drop <files>

    Remove files from chat
    """

def cmd_read_only(self, args):
    """
    /read-only <files>

    Add as read-only reference
    Won't be edited by AI
    """

def cmd_ls(self, args):
    """
    /ls

    List files in chat
    """
```

#### B. Git Operations

```python
def cmd_commit(self, args):
    """
    /commit [message]

    Create git commit
    AI generates message if not provided
    """

def cmd_undo(self, args):
    """
    /undo

    Undo last change
    Uses git reset
    """

def cmd_diff(self, args):
    """
    /diff

    Show git diff
    """

def cmd_git(self, args):
    """
    /git <command>

    Run git command
    Example: /git status
    """
```

#### C. Model Operations

```python
def cmd_model(self, args):
    """
    /model [name]

    Switch model or list available

    Examples:
    /model              # List models
    /model sonnet       # Switch to Claude Sonnet
    /model 4o           # Switch to GPT-4o
    """

def cmd_models(self, args):
    """
    /models

    List all available models
    """
```

#### D. Mode Switching

```python
def cmd_architect(self, args):
    """
    /architect

    Enable architect mode
    Two-model collaboration
    """

def cmd_ask(self, args):
    """
    /ask

    Switch to ask mode (read-only)
    """

def cmd_code(self, args):
    """
    /code

    Switch to code mode (editing)
    """
```

#### E. Settings

```python
def cmd_settings(self, args):
    """
    /settings

    Show current settings
    """

def cmd_set(self, args):
    """
    /set <key> <value>

    Change settings

    Examples:
    /set auto-commits true
    /set edit-format diff
    """
```

#### F. Utility Commands

```python
def cmd_help(self, args):
    """
    /help [command]

    Show help
    """

def cmd_clear(self, args):
    """
    /clear

    Clear chat history
    """

def cmd_tokens(self, args):
    """
    /tokens

    Show token usage
    """

def cmd_cost(self, args):
    """
    /cost

    Show cost statistics
    """

def cmd_run(self, args):
    """
    /run <command>

    Run shell command
    Example: /run pytest
    """

def cmd_voice(self, args):
    """
    /voice

    Enable voice input
    """

def cmd_paste(self, args):
    """
    /paste

    Paste from clipboard
    """

def cmd_web(self, args):
    """
    /web <url>

    Scrape web page
    Add as context
    """

def cmd_exit(self, args):
    """
    /exit or /quit

    Exit aider
    """
```

### Command Registration

```python
def get_commands(self):
    """
    Auto-discover commands:
    1. Find methods starting with cmd_
    2. Extract command name
    3. Get docstring as help
    4. Build command dict
    """
```

### Command Execution

```python
def run_command(self, inp):
    """
    1. Parse command and args
    2. Find handler method
    3. Execute
    4. Handle errors
    5. Return result
    """
```

---

## 8. Configuration System

### Configuration Files

#### A. Main Config: `.aider.conf.yml`

**Search Locations:**
1. Current directory
2. Git root
3. Home directory (~/)

**Format:**
```yaml
# Model settings
model: sonnet
weak-model: haiku
edit-format: diff

# Git settings
auto-commits: true
attribute-author: false
attribute-committer: false

# Editor settings
auto-lint: true
auto-test: false
stream: true
pretty: true

# Context settings
map-tokens: 1024
cache-prompts: true

# UI settings
dark-mode: true
show-diffs: true
fancy-input: true

# Voice settings
voice-language: en
```

#### B. Model Settings: `.aider.model.settings.yml`

**Default:** `aider/resources/model-settings.yml`

**Custom:** `.aider.model.settings.yml` in project

**Format:**
```yaml
- name: my-custom-model
  edit_format: diff
  weak_model_name: gpt-4o-mini
  use_repo_map: true
  streaming: true
  lazy: true
  reminder: sys
  examples_as_sys_msg: false
  editor_edit_format: editor-diff
  max_input_tokens: 128000
  max_output_tokens: 8192
```

#### C. Model Metadata: `.aider.model.metadata.json`

**Default:** `aider/resources/model-metadata.json`

**Format:**
```json
{
  "gpt-4o": {
    "max_input_tokens": 128000,
    "max_output_tokens": 16384,
    "input_cost_per_token": 0.0000025,
    "output_cost_per_token": 0.00001,
    "litellm_provider": "openai",
    "mode": "chat"
  }
}
```

#### D. Environment Variables: `.env`

**Search Locations:**
1. `~/.aider/oauth-keys.env`
2. `~/.env`
3. Git root `.env`
4. Current directory `.env`

**Format:**
```bash
# API Keys
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
DEEPSEEK_API_KEY=...
GEMINI_API_KEY=...

# Model defaults
AIDER_MODEL=sonnet
AIDER_WEAK_MODEL=haiku

# Settings
AIDER_AUTO_COMMITS=true
AIDER_DARK_MODE=true
AIDER_EDIT_FORMAT=diff
```

### Configuration Hierarchy

**Priority (highest to lowest):**
1. Command-line arguments
2. Environment variables (AIDER_*)
3. --config specified file
4. Project `.aider.conf.yml`
5. Git root `.aider.conf.yml`
6. User `~/.aider.conf.yml`
7. Default values

**Example:**
```bash
# All equivalent, but different priority:
aider --model sonnet              # Highest
AIDER_MODEL=sonnet aider          # High
# model: sonnet in .aider.conf.yml  # Medium
```

### Argument Parsing

**File:** `aider/args.py` (945 dòng)

**Uses:** `configargparse`

**Categories:**
- Model options
- Edit options
- Git options
- Context options
- Output options
- Voice options
- Advanced options

---

## Summary

Tài liệu này cung cấp phân tích chi tiết về:

1. ✅ Base Coder system và architecture
2. ✅ Tất cả edit format implementations
3. ✅ Model management và integration
4. ✅ Git operations và automation
5. ✅ Repository mapping algorithm
6. ✅ Terminal I/O system
7. ✅ Commands system với tất cả commands
8. ✅ Configuration system đầy đủ

Mỗi component được giải thích với:
- Purpose và use cases
- Implementation details
- Key methods và attributes
- Code locations
- Examples

**Next:** Xem REPORT_03 cho implementation guides và extension points.

---

**Generated:** 2025-11-13
**Analysis Scope:** Aider codebase complete
