# AIDER - HƯỚNG DẪN FEATURES VÀ EXTENSION

## Mục Lục
1. [Core Features Implementation](#1-core-features-implementation)
2. [Extension Points](#2-extension-points)
3. [Custom Edit Format](#3-custom-edit-format)
4. [Custom Commands](#4-custom-commands)
5. [Custom Models](#5-custom-models)
6. [Custom Linters](#6-custom-linters)
7. [Plugin Development](#7-plugin-development)
8. [API Usage](#8-api-usage)

---

## 1. Core Features Implementation

### A. Multi-Format Editing

#### Cách Hoạt Động

**Workflow:**
```
User Request
    ↓
[Select Edit Format]
    ↓
[Generate Prompt]
    ↓
[Send to LLM]
    ↓
[Parse Response] ← Format-specific
    ↓
[Apply Edits] ← Format-specific
    ↓
[Done]
```

#### Supported Formats

**1. Diff Format (EditBlockCoder)**

**Tốt nhất cho:**
- Complex multi-line edits
- Refactoring
- Khi cần chính xác cao

**Example:**
```python
# User: "Add error handling to the read_file function"

# LLM Response:
path/to/file.py
<<<<<<< SEARCH
def read_file(fname):
    with open(fname) as f:
        return f.read()
=======
def read_file(fname):
    try:
        with open(fname) as f:
            return f.read()
    except FileNotFoundError:
        print(f"File {fname} not found")
        return None
    except Exception as e:
        print(f"Error reading {fname}: {e}")
        return None
>>>>>>> REPLACE
```

**2. Whole Format (WholeFileCoder)**

**Tốt nhất cho:**
- New files
- Complete rewrites
- Small files

**Example:**
```python
# User: "Create a new utils.py with helper functions"

# LLM Response:
utils.py
```python
def helper1():
    pass

def helper2():
    pass
```
```

**3. UDiff Format (UnifiedDiffCoder)**

**Tốt nhất cho:**
- Small targeted changes
- Integration with git tools
- Precise line edits

**Example:**
```diff
--- a/file.py
+++ b/file.py
@@ -10,6 +10,7 @@
 def main():
+    print("Starting...")
     do_something()
```

**4. Architect Format (ArchitectCoder)**

**Tốt nhất cho:**
- Complex features
- Need planning first
- Large changes across multiple files

**Workflow:**
```
User: "Add authentication system"
    ↓
Architect Model:
    "Plan:
    1. Create auth.py with User class
    2. Add login/logout functions
    3. Update main.py to use auth
    4. Add tests"
    ↓
Editor Model:
    [Implements each step]
    ↓
Verify & Iterate
```

#### Switching Formats

```bash
# Command line
aider --edit-format diff
aider --edit-format whole
aider --edit-format architect

# In chat
/set edit-format diff
/architect  # Switch to architect mode

# Config file
# .aider.conf.yml
edit-format: diff
```

---

### B. Repository Mapping

#### Purpose
Provide LLM với "map" của codebase để hiểu structure mà không load toàn bộ code.

#### How It Works

**1. Parse Code (Tree-sitter)**
```python
# For each file:
parse_result = tree_sitter.parse(file_content)

# Extract:
- class MyClass:
- def function_name(args):
- async def async_function():
```

**2. Build Tag Map**
```
src/main.py:
├── class Application
│   ├── __init__(self, config)
│   ├── run(self)
│   └── shutdown(self)
├── main()
└── parse_args()

src/utils.py:
├── helper_function(arg1, arg2)
└── format_output(data)
```

**3. Rank by Relevance**
```python
Priority:
1. Tags in files being edited (highest)
2. Tags mentioned in chat
3. Tags in same directory
4. Tags in imported modules
5. Other tags (lowest)
```

**4. Fit in Token Budget**
```python
max_tokens = 1024  # Configurable
current_tokens = 0

for tag in sorted_tags:
    tag_tokens = count_tokens(tag)
    if current_tokens + tag_tokens > max_tokens:
        break
    include_tag(tag)
    current_tokens += tag_tokens
```

#### Configuration

```yaml
# .aider.conf.yml
map-tokens: 1024      # Max tokens for repo map
map-refresh: auto     # Refresh strategy
```

```bash
# Command line
aider --map-tokens 2048
aider --no-map  # Disable repo map
```

#### Caching

**SQLite Cache:**
- File hashes (detect changes)
- Parsed tags
- Rankings

**Invalidation:**
- File modification
- Git commit
- Manual refresh

---

### C. Git Integration

#### Automatic Commits

**Enable:**
```bash
aider --auto-commits
```

```yaml
# .aider.conf.yml
auto-commits: true
```

**How It Works:**
```
Code Changes Applied
    ↓
[Stage Files]
    ↓
[Generate Commit Message] ← LLM
    ↓
[Create Commit]
    ↓
[Done]
```

**Example Commit Message:**
```
feat: add error handling to file operations

- Add try/except blocks
- Handle FileNotFoundError
- Log errors appropriately
```

#### Attribution Options

**1. Attribute Author**
```bash
aider --attribute-author
```

Result:
```
Author: Aider <aider@aider.chat>
```

**2. Attribute Committer**
```bash
aider --attribute-committer
```

Result:
```
Committer: Aider <aider@aider.chat>
```

**3. Co-Authored-By**
```bash
aider --attribute-co-authored-by
```

Result:
```
Author: Your Name <you@example.com>

Co-authored-by: Aider <aider@aider.chat>
```

#### Manual Commit

```bash
# In chat
/commit
/commit "custom message"

# Manual git
/git add .
/git commit -m "message"
/git push
```

---

### D. Linting & Testing

#### Auto-Lint

**Enable:**
```bash
aider --auto-lint
```

**How It Works:**
```
Code Changes Applied
    ↓
[Run Linter]
    ↓
Errors Found?
    ├─ Yes → [Show to LLM] → [Fix] → [Re-run]
    └─ No → [Continue]
```

**Default Linters:**
- Python: flake8
- JavaScript: eslint (if available)
- TypeScript: tsc (if available)

**Custom Linter:**
```bash
aider --lint-cmd "python: pylint"
aider --lint-cmd "javascript: eslint --fix"
```

#### Auto-Test

**Enable:**
```bash
aider --auto-test
```

**Custom Test Command:**
```bash
aider --test-cmd "pytest tests/"
aider --test-cmd "npm test"
```

**How It Works:**
```
Code Changes Applied
    ↓
[Run Tests]
    ↓
Tests Pass?
    ├─ Yes → [Continue]
    └─ No → [Show failures to LLM] → [Fix] → [Re-run]
```

---

### E. Watch Mode

#### Purpose
Automatically process AI requests in code comments.

#### Enable

```bash
aider --watch-files
```

#### How It Works

**1. Monitor Files**
```python
# Uses watchfiles library
watcher = watch(repo_path, ignore_patterns=['.git', '__pycache__'])

for changes in watcher:
    process_changes(changes)
```

**2. Detect AI Comments**
```python
# Python
# ai: add error handling here

# JavaScript
// ai: refactor this function

# Rust
// ai: optimize this loop
```

**3. Process Request**
```
Comment Detected
    ↓
[Extract Request]
    ↓
[Determine Mode] (code or ask)
    ↓
[Process with LLM]
    ↓
[Apply Changes]
    ↓
[Remove Comment]
```

#### Example Workflow

**Before:**
```python
def process_data(data):
    # ai: add input validation
    result = expensive_operation(data)
    return result
```

**After (Automatic):**
```python
def process_data(data):
    if not data:
        raise ValueError("Data cannot be empty")
    if not isinstance(data, dict):
        raise TypeError("Data must be a dictionary")

    result = expensive_operation(data)
    return result
```

---

### F. Voice Coding

#### Setup

**Install Dependencies:**
```bash
pip install aider-chat[voice]
```

**Enable:**
```bash
aider --voice
```

Or in chat:
```
/voice
```

#### How It Works

```
[Press Space] (start recording)
    ↓
[Speak Request]
    ↓
[Release Space] (stop recording)
    ↓
[Transcribe with Whisper] ← OpenAI API
    ↓
[Process as Text]
```

#### Configuration

```yaml
# .aider.conf.yml
voice-language: en
voice-format: text  # or markdown
```

```bash
aider --voice-language vi  # Vietnamese
aider --voice-language zh  # Chinese
```

---

### G. Web Scraping

#### Purpose
Fetch documentation from web pages to use as context.

#### Usage

```bash
# In chat
/web https://docs.python.org/3/library/pathlib.html
```

**LLM can then use the docs:**
```
User: "Use pathlib to handle file paths"
[LLM has pathlib docs as context]
```

#### How It Works

**1. Fetch Page**
```python
# Option 1: Simple (requests + beautifulsoup)
response = requests.get(url)
html = response.text

# Option 2: JavaScript (playwright)
page = browser.new_page()
page.goto(url)
html = page.content()
```

**2. Convert to Markdown**
```python
markdown = pypandoc.convert_text(
    html,
    'markdown',
    format='html'
)
```

**3. Add to Context**
```python
read_only_files.add(f"web:{url}")
```

#### Configuration

```bash
# Use playwright for JS-heavy sites
pip install aider-chat[playwright]
aider --playwright
```

---

### H. Copy/Paste Mode

#### Purpose
Work with LLMs via web chat (ChatGPT, Claude web, etc.)

#### Enable

```bash
aider --copy-paste
```

#### How It Works

**Workflow:**
```
1. Aider copies context to clipboard
2. Paste into web chat
3. Get response from LLM
4. Copy response
5. Paste back into aider
6. Aider applies changes
7. Repeat
```

**Automatic Clipboard Monitoring:**
- Detect when you copy from browser
- Auto-paste into aider
- Apply edits automatically

---

## 2. Extension Points

### Overview

Aider được thiết kế để dễ extend:

1. **Edit Formats** - Add new ways to edit code
2. **Commands** - Add new slash commands
3. **Models** - Add custom LLM models
4. **Linters** - Add language-specific linters
5. **Plugins** - Future plugin system

---

## 3. Custom Edit Format

### Step-by-Step Guide

#### 1. Create New File

```python
# aider/coders/my_format_coder.py

from .base_coder import Coder
from .my_format_prompts import MyFormatPrompts

class MyFormatCoder(Coder):
    """My custom edit format"""

    edit_format = "myformat"  # Format name
    gpt_prompts = MyFormatPrompts()
```

#### 2. Implement Required Methods

```python
class MyFormatCoder(Coder):
    """My custom edit format"""

    edit_format = "myformat"
    gpt_prompts = MyFormatPrompts()

    def get_edits(self):
        """
        Parse LLM response to extract edits

        Returns: list of (fname, edit_data)
        """
        edits = []

        # Parse self.partial_response_content
        # Extract file paths and changes
        # Example:
        for block in self.parse_response():
            fname = block['file']
            content = block['content']
            edits.append((fname, content))

        return edits

    def apply_edits(self, edits):
        """
        Apply parsed edits to files

        Args:
            edits: list of (fname, edit_data)

        Returns:
            edited: set of modified file paths
        """
        edited = set()

        for fname, edit_data in edits:
            # Apply changes to file
            self.io.write_text(fname, edit_data)
            edited.add(fname)

        return edited

    def parse_response(self):
        """
        Custom parsing logic
        """
        # Parse self.partial_response_content
        # Return structured data
        pass
```

#### 3. Create Prompts Class

```python
# aider/coders/my_format_prompts.py

from .base_prompts import CoderPrompts

class MyFormatPrompts(CoderPrompts):
    """Prompts for my custom format"""

    main_system = """You are an AI coding assistant.

When editing code, use this format:

FILE: path/to/file.py
---
complete file content here
---

Rules:
1. Always include full path
2. Include entire file content
3. Make changes inline
"""

    example_messages = [
        dict(
            role="user",
            content="Add error handling to read_file"
        ),
        dict(
            role="assistant",
            content="""FILE: utils.py
---
def read_file(fname):
    try:
        with open(fname) as f:
            return f.read()
    except FileNotFoundError:
        return None
---
"""
        )
    ]
```

#### 4. Register Coder

```python
# aider/coders/__init__.py

from .my_format_coder import MyFormatCoder

__all__ = [
    # Existing coders
    EditBlockCoder,
    WholeFileCoder,
    # ... other coders

    # Your new coder
    MyFormatCoder,
]
```

#### 5. Use Your Format

```bash
aider --edit-format myformat
```

```yaml
# .aider.conf.yml
edit-format: myformat
```

---

## 4. Custom Commands

### Step-by-Step Guide

#### 1. Add Method to Commands Class

```python
# aider/commands.py

class Commands:
    # ... existing code

    def cmd_mycommand(self, args):
        """
        Brief description of my command

        Usage: /mycommand [args]

        Detailed help text here.
        """
        # Parse args
        if not args:
            self.io.tool_error("Usage: /mycommand <arg>")
            return

        # Your implementation
        result = self.do_something(args)

        # Output
        self.io.tool_output(f"Result: {result}")
```

#### 2. Command Examples

**Example 1: Simple Info Command**
```python
def cmd_info(self, args):
    """Show project information"""

    info = {
        "Files in chat": len(self.coder.abs_fnames),
        "Model": self.coder.main_model.name,
        "Edit format": self.coder.edit_format,
        "Git repo": bool(self.coder.repo),
    }

    for key, value in info.items():
        self.io.tool_output(f"{key}: {value}")
```

**Example 2: File Analysis Command**
```python
def cmd_analyze(self, args):
    """Analyze code complexity"""

    if not args:
        files = self.coder.abs_fnames
    else:
        files = [args]

    for fname in files:
        content = self.io.read_text(fname)
        lines = len(content.split('\n'))
        functions = content.count('def ')

        self.io.tool_output(f"\n{fname}:")
        self.io.tool_output(f"  Lines: {lines}")
        self.io.tool_output(f"  Functions: {functions}")
```

**Example 3: Interactive Command**
```python
def cmd_refactor(self, args):
    """Interactive refactoring assistant"""

    # Get files to refactor
    if not args:
        self.io.tool_error("Usage: /refactor <file>")
        return

    fname = args

    # Confirm
    if not self.io.confirm_ask(f"Refactor {fname}?"):
        return

    # Ask for refactoring type
    self.io.tool_output("Refactoring options:")
    self.io.tool_output("1. Extract functions")
    self.io.tool_output("2. Rename variables")
    self.io.tool_output("3. Add type hints")

    choice = self.io.get_input("Choose option: ")

    # Process based on choice
    prompt = self.build_refactor_prompt(fname, choice)
    self.coder.send_message(prompt)
```

#### 3. Use Your Command

```bash
# In chat
/mycommand arg1 arg2
/info
/analyze utils.py
/refactor main.py
```

---

## 5. Custom Models

### Add Custom Model

#### 1. Create Model Settings File

```yaml
# .aider.model.settings.yml

- name: my-custom-model
  # Edit format to use
  edit_format: diff

  # Simpler model for easy tasks
  weak_model_name: gpt-4o-mini

  # Use repository mapping
  use_repo_map: true

  # Support streaming
  streaming: true

  # Lazy load model (faster startup)
  lazy: true

  # Where to show reminders
  reminder: sys  # or "user" or null

  # How to show examples
  examples_as_sys_msg: false

  # Edit format for architect mode
  editor_edit_format: editor-diff

  # Context window size
  max_input_tokens: 128000
  max_output_tokens: 8192

  # Prompt caching support
  cache_control: true
```

#### 2. Add Model Metadata

```json
// .aider.model.metadata.json
{
  "my-custom-model": {
    "max_input_tokens": 128000,
    "max_output_tokens": 8192,
    "input_cost_per_token": 0.000003,
    "output_cost_per_token": 0.000015,
    "litellm_provider": "openai",
    "mode": "chat"
  }
}
```

#### 3. Configure API Access

```bash
# .env
MY_CUSTOM_API_KEY=your-key-here
```

```yaml
# .aider.conf.yml
model: my-custom-model
api-key: my-custom=${MY_CUSTOM_API_KEY}
```

#### 4. Use Custom Model

```bash
aider --model my-custom-model
```

### Local Models

#### Ollama

```bash
# Start Ollama
ollama serve

# Pull model
ollama pull codellama

# Use with aider
aider --model ollama/codellama
```

#### LM Studio

```bash
# Start LM Studio server
# Default: http://localhost:1234

# Use with aider
aider --model openai/my-model \
  --openai-api-base http://localhost:1234/v1
```

#### vLLM

```bash
# Start vLLM
vllm serve model-name --port 8000

# Use with aider
aider --model openai/model-name \
  --openai-api-base http://localhost:8000/v1
```

---

## 6. Custom Linters

### Per-Language Linters

```bash
# Python with pylint
aider --lint-cmd "python: pylint --rcfile=.pylintrc"

# JavaScript with eslint
aider --lint-cmd "javascript: eslint --fix"

# TypeScript
aider --lint-cmd "typescript: tsc --noEmit"

# Go
aider --lint-cmd "go: golint"

# Rust
aider --lint-cmd "rust: cargo clippy"
```

### Global Linter

```bash
# Apply to all files
aider --lint-cmd "shellcheck"
```

### Multiple Linters

```yaml
# .aider.conf.yml
lint-cmd:
  - "python: pylint"
  - "python: mypy"
  - "javascript: eslint"
  - "css: stylelint"
```

### Custom Linter Script

```bash
# lint.sh
#!/bin/bash
FILE=$1

case "${FILE##*.}" in
  py)
    pylint "$FILE" && mypy "$FILE"
    ;;
  js)
    eslint "$FILE"
    ;;
  *)
    echo "No linter for ${FILE}"
    ;;
esac
```

```bash
aider --lint-cmd "./lint.sh"
```

---

## 7. Plugin Development

### Current Plugin Points

#### 1. Edit Format Plugins

See [Custom Edit Format](#3-custom-edit-format)

#### 2. Command Plugins

See [Custom Commands](#4-custom-commands)

#### 3. Model Plugins

See [Custom Models](#5-custom-models)

#### 4. Linter Plugins

See [Custom Linters](#6-custom-linters)

### Future Plugin System

**Planned Features:**
- Plugin discovery
- Plugin marketplace
- Hot reloading
- Plugin dependencies
- Plugin configuration

**Proposed API:**
```python
# aider_plugin_example.py

from aider.plugin import Plugin

class MyPlugin(Plugin):
    name = "my-plugin"
    version = "1.0.0"

    def on_startup(self, aider):
        """Called when aider starts"""
        pass

    def on_message(self, message):
        """Called for each user message"""
        pass

    def on_edit(self, files, edits):
        """Called before applying edits"""
        pass

    def on_commit(self, files, message):
        """Called before git commit"""
        pass
```

---

## 8. API Usage

### Programmatic Usage

#### Basic Usage

```python
from aider.main import main

# Run aider programmatically
coder = main(
    argv=['--model', 'sonnet', 'main.py'],
    return_coder=True
)

# Send a message
response = coder.run(with_message="Add error handling")

# Get files
files = coder.abs_fnames
print(f"Files: {files}")
```

#### Advanced Usage

```python
from aider.coders import Coder
from aider.models import Model
from aider.io import InputOutput
from aider.repo import GitRepo

# Setup
model = Model("gpt-4o")
io = InputOutput(pretty=True, fancy_input=False)
repo = GitRepo()

# Create coder
coder = Coder.create(
    main_model=model,
    io=io,
    edit_format="diff",
    auto_commits=True,
    auto_lint=True
)

# Add files
coder.add_rel_fname("main.py")
coder.add_rel_fname("utils.py")

# Send message
coder.send_message("Refactor the main function")

# Check results
for fname in coder.abs_fnames:
    print(f"Modified: {fname}")
```

#### Batch Processing

```python
from aider.main import main

# Process multiple files
files = [
    "file1.py",
    "file2.py",
    "file3.py"
]

for file in files:
    coder = main(
        argv=['--model', 'sonnet', file],
        return_coder=True
    )

    coder.run(with_message="Add docstrings")
```

#### Custom Integration

```python
class MyCodeAssistant:
    def __init__(self):
        self.coder = main(
            argv=['--model', 'sonnet'],
            return_coder=True
        )

    def add_feature(self, feature_description, files):
        # Add files
        for f in files:
            self.coder.add_rel_fname(f)

        # Generate feature
        self.coder.run(with_message=f"Implement: {feature_description}")

    def fix_bugs(self, bug_report):
        # Automatic bug fixing
        self.coder.run(with_message=f"Fix this bug: {bug_report}")

    def refactor(self, files):
        # Refactoring
        for f in files:
            self.coder.add_rel_fname(f)

        self.coder.run(with_message="Refactor for better maintainability")

# Usage
assistant = MyCodeAssistant()
assistant.add_feature("user authentication", ["auth.py", "main.py"])
assistant.fix_bugs("Login fails for users with @ in username")
assistant.refactor(["utils.py"])
```

---

## Summary

Tài liệu này cung cấp:

1. ✅ Detailed implementation của tất cả core features
2. ✅ How to extend với custom edit formats
3. ✅ How to add custom commands
4. ✅ How to integrate custom models
5. ✅ Custom linter configuration
6. ✅ Future plugin system
7. ✅ Programmatic API usage

Mỗi section bao gồm:
- Concept explanation
- Step-by-step implementation
- Code examples
- Use cases
- Best practices

**Next:** Xem REPORT_04 cho quick reference và cheat sheets.

---

**Generated:** 2025-11-13
**Scope:** Complete feature and extension guide
