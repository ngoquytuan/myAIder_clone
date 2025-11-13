# AIDER - QUICK REFERENCE & CHEAT SHEET

## 📚 Tài Liệu Nhanh

### Basic Commands

```bash
# Start aider
aider

# With specific model
aider --model sonnet
aider --model 4o
aider --model deepseek

# With files
aider main.py utils.py
aider *.py

# Configuration
aider --config .aider.conf.yml
```

---

## 🎯 Slash Commands

### File Management
```
/add <file>          # Add file to chat
/drop <file>         # Remove file from chat
/ls                  # List files in chat
/read-only <file>    # Add as read-only
/clear               # Clear chat history
```

### Git Operations
```
/commit [message]    # Create git commit
/undo                # Undo last change
/diff                # Show git diff
/git <command>       # Run git command
```

### Model & Mode
```
/model [name]        # Switch/list models
/models              # List all models
/architect           # Enable architect mode
/ask                 # Switch to ask mode (read-only)
/code                # Switch to code mode (editing)
```

### Settings
```
/settings            # Show current settings
/set <key> <value>   # Change setting
/tokens              # Show token usage
/cost                # Show cost stats
```

### Utilities
```
/help [command]      # Show help
/voice               # Enable voice input
/run <command>       # Run shell command
/web <url>           # Scrape web page
/paste               # Paste from clipboard
/exit or /quit       # Exit aider
```

---

## ⚙️ Configuration File

### .aider.conf.yml

```yaml
# Model Settings
model: sonnet
weak-model: haiku
edit-format: diff

# Git Settings
auto-commits: true
dirty-commits: false
attribute-author: false
attribute-committer: false
attribute-co-authored-by: true

# Editor Settings
auto-lint: true
auto-test: false
stream: true
pretty: true
show-diffs: true

# Context Settings
map-tokens: 1024
map-refresh: auto
cache-prompts: true

# UI Settings
dark-mode: true
fancy-input: true

# Voice Settings
voice-language: en

# Linting
lint-cmd: "python: flake8"
```

---

## 🔧 Environment Variables

### .env

```bash
# API Keys
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...
DEEPSEEK_API_KEY=...
GEMINI_API_KEY=...

# Defaults
AIDER_MODEL=sonnet
AIDER_WEAK_MODEL=haiku
AIDER_EDIT_FORMAT=diff

# Settings
AIDER_AUTO_COMMITS=true
AIDER_AUTO_LINT=true
AIDER_DARK_MODE=true
AIDER_STREAM=true
AIDER_PRETTY=true
```

---

## 📊 Model Reference

### Popular Models

| Alias      | Full Name                          | Provider   | Context |
|------------|------------------------------------|------------|---------|
| sonnet     | claude-sonnet-4-20250514           | Anthropic  | 200K    |
| opus       | claude-opus-4-20250514             | Anthropic  | 200K    |
| haiku      | claude-3-5-haiku-20241022          | Anthropic  | 200K    |
| 4o         | gpt-4o                             | OpenAI     | 128K    |
| 4o-mini    | gpt-4o-mini                        | OpenAI     | 128K    |
| o1         | o1                                 | OpenAI     | 200K    |
| o3-mini    | o3-mini                            | OpenAI     | 200K    |
| deepseek   | deepseek/deepseek-chat             | DeepSeek   | 64K     |
| deepseek-r1| deepseek/deepseek-reasoner         | DeepSeek   | 64K     |
| gemini     | gemini/gemini-2.0-flash-exp        | Google     | 1M      |

### Model Costs (Approximate)

| Model          | Input (per 1M tokens) | Output (per 1M tokens) |
|----------------|----------------------|------------------------|
| GPT-4o         | $2.50                | $10.00                 |
| GPT-4o-mini    | $0.15                | $0.60                  |
| Claude Sonnet  | $3.00                | $15.00                 |
| Claude Haiku   | $0.80                | $4.00                  |
| DeepSeek Chat  | $0.14                | $0.28                  |
| Gemini Flash   | Free (limited)       | Free (limited)         |

---

## 🎨 Edit Formats

### Comparison

| Format     | File  | Use Case                | Speed | Accuracy |
|------------|-------|-------------------------|-------|----------|
| diff       | Any   | Complex edits           | ⭐⭐⭐  | ⭐⭐⭐⭐⭐ |
| whole      | Small | New files, rewrites     | ⭐⭐⭐⭐⭐| ⭐⭐⭐   |
| udiff      | Any   | Precise line edits      | ⭐⭐⭐⭐ | ⭐⭐⭐⭐  |
| architect  | Any   | Complex features        | ⭐⭐    | ⭐⭐⭐⭐⭐ |
| ask        | N/A   | Questions only          | ⭐⭐⭐⭐⭐| N/A      |

### Switching Formats

```bash
# Command line
aider --edit-format diff
aider --edit-format whole
aider --edit-format architect

# In chat
/set edit-format diff
/architect
```

---

## 🚀 Common Workflows

### 1. Start New Feature

```bash
# Start aider with relevant files
aider src/main.py src/utils.py

# In chat:
Add user authentication with the following requirements:
- Login/logout functions
- Session management
- Password hashing with bcrypt
```

### 2. Fix Bugs

```bash
# Add file with bug
aider src/buggy_file.py

# In chat:
The function `process_data` fails when input is empty.
Fix this bug and add input validation.
```

### 3. Refactor Code

```bash
# Add files to refactor
aider src/*.py

# In chat:
Refactor the code to:
1. Extract common functionality into utils
2. Add type hints
3. Improve naming
4. Add docstrings
```

### 4. Add Tests

```bash
# Add implementation and test file
aider src/feature.py tests/test_feature.py

# In chat:
Add comprehensive unit tests for all functions in feature.py
Include edge cases and error conditions.
```

### 5. Documentation

```bash
# Read-only mode
aider --ask

# In chat:
/add src/complex_module.py
Explain how this module works and document all functions.
```

### 6. Code Review

```bash
# Read-only
aider --ask

# In chat:
/add src/pull_request_code.py
Review this code for:
- Potential bugs
- Performance issues
- Security vulnerabilities
- Best practices
```

---

## 📁 File Structure Reference

### Important Files

```
.aider.conf.yml              # Main config
.aider.model.settings.yml    # Custom models
.aider.model.metadata.json   # Model metadata
.env                         # API keys
.aiderignore                 # Ignore patterns

~/.aider/
├── .aider.conf.yml         # User config
├── oauth-keys.env          # OAuth keys
└── .aider.cache/           # Cache directory
```

### Ignore Patterns (.aiderignore)

```
# Build artifacts
build/
dist/
*.egg-info/

# Dependencies
node_modules/
venv/
__pycache__/

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db

# Logs
*.log
logs/

# Secrets
*.key
*.pem
.env.local
```

---

## 🎤 Voice Commands

### Setup

```bash
# Install voice support
pip install aider-chat[voice]

# Enable voice
aider --voice
```

### In Chat

```
/voice                # Enable voice
[Press Space]         # Start recording
[Speak]              # Say your request
[Release Space]      # Stop recording
```

### Supported Languages

- English (en)
- Spanish (es)
- French (fr)
- German (de)
- Chinese (zh)
- Japanese (ja)
- Korean (ko)
- Vietnamese (vi)
- And many more...

```bash
aider --voice-language vi
```

---

## 🔍 Watch Mode

### Setup

```bash
aider --watch-files
```

### Usage

**In your code:**
```python
def process_data(data):
    # ai: add error handling
    result = expensive_operation(data)
    return result
```

**Aider automatically:**
1. Detects comment
2. Processes request
3. Applies changes
4. Removes comment

### Comment Patterns

```python
# ai: <request>           # Python
// ai: <request>          # JavaScript, C++, Go, Rust
<!-- ai: <request> -->    # HTML, XML
/* ai: <request> */       # CSS, Java
```

---

## 🌐 Web Scraping

### Fetch Documentation

```bash
# In chat
/web https://docs.python.org/3/library/pathlib.html
```

**Then use in requests:**
```
Use pathlib to handle all file operations
```

### With Playwright (JS sites)

```bash
# Install
pip install aider-chat[playwright]

# Use
aider --playwright
```

---

## 🧪 Testing & Linting

### Auto-Lint

```bash
# Enable auto-lint
aider --auto-lint

# Custom linter
aider --lint-cmd "python: pylint"
aider --lint-cmd "javascript: eslint --fix"
```

### Auto-Test

```bash
# Enable auto-test
aider --auto-test

# Custom test command
aider --test-cmd "pytest tests/ -v"
aider --test-cmd "npm test"
```

### In Config

```yaml
# .aider.conf.yml
auto-lint: true
auto-test: true
lint-cmd: "python: flake8 --max-line-length=100"
test-cmd: "pytest tests/"
```

---

## 🔐 Security Best Practices

### API Keys

```bash
# ✅ Good - Use .env file
echo "OPENAI_API_KEY=sk-..." >> .env

# ✅ Good - Use environment variable
export OPENAI_API_KEY=sk-...

# ❌ Bad - Command line (visible in history)
aider --api-key openai=sk-...

# ❌ Bad - In config file committed to git
# .aider.conf.yml
# api-key: sk-...  # DON'T DO THIS
```

### Git Safety

```bash
# Always use .gitignore
echo ".env" >> .gitignore
echo ".aider.cache/" >> .gitignore

# Use .aiderignore for sensitive files
echo "secrets/" >> .aiderignore
echo "*.key" >> .aiderignore
```

---

## 🐛 Troubleshooting

### Common Issues

**1. "No API key found"**
```bash
# Check .env file exists
cat .env

# Check environment variable
echo $OPENAI_API_KEY

# Set explicitly
export OPENAI_API_KEY=your-key
```

**2. "Git repository not found"**
```bash
# Initialize git repo
git init
git add .
git commit -m "Initial commit"
```

**3. "Model not found"**
```bash
# List available models
aider --models

# Use full name instead of alias
aider --model openai/gpt-4o
```

**4. "Token limit exceeded"**
```bash
# Reduce repo map size
aider --map-tokens 512

# Disable repo map
aider --no-map

# Use fewer files
aider file1.py  # Instead of *.py
```

**5. "Edit failed to apply"**
```bash
# Try different edit format
aider --edit-format whole

# Clear cache
rm -rf .aider.cache/

# Update aider
pip install --upgrade aider-chat
```

---

## 📊 Performance Tips

### Speed Up Startup

```yaml
# .aider.conf.yml
lazy: true           # Lazy load LLM
cache-prompts: true  # Cache prompts
```

### Reduce Costs

```bash
# Use cheaper models
aider --model 4o-mini
aider --model haiku

# Disable repo map for small projects
aider --no-map

# Use weak model for simple tasks
aider --weak-model 4o-mini
```

### Improve Accuracy

```bash
# Use better models
aider --model sonnet
aider --model o3-mini

# Use architect mode
aider --architect

# Enable auto-lint
aider --auto-lint

# Provide context
/read-only docs/
/web https://docs.example.com
```

---

## 💡 Pro Tips

### 1. Use Architect Mode for Complex Tasks

```bash
aider --architect
```

Better planning = better results

### 2. Add Read-Only Context

```bash
# In chat
/read-only README.md
/read-only docs/architecture.md
```

Provides context without allowing edits

### 3. Use Voice for Speed

```bash
aider --voice
```

Faster than typing for long requests

### 4. Watch Mode for Quick Fixes

```bash
aider --watch-files
```

Edit in your IDE, let AI fix issues

### 5. Combine with Git

```bash
# Make changes
aider --auto-commits

# Review
git log
git diff HEAD~1

# Undo if needed
/undo
```

### 6. Use Repo Map for Large Projects

```yaml
map-tokens: 2048    # Larger map
map-refresh: auto   # Auto update
```

### 7. Custom Model for Specific Tasks

```yaml
# .aider.model.settings.yml
- name: code-review
  edit_format: ask
  use_repo_map: true

- name: quick-fix
  edit_format: diff
  streaming: true
```

---

## 📖 File Locations Quick Reference

### Linux/Mac

```
~/.aider/
├── .aider.conf.yml
└── oauth-keys.env

~/.env
~/.cache/aider/
```

### Windows

```
%USERPROFILE%\.aider\
├── .aider.conf.yml
└── oauth-keys.env

%USERPROFILE%\.env
%LOCALAPPDATA%\aider\cache\
```

---

## 🔗 Important Links

- **GitHub:** https://github.com/Aider-AI/aider
- **Docs:** https://aider.chat/docs/
- **Discord:** https://discord.gg/Y7X7bhMQFV
- **Blog:** https://aider.chat/blog/
- **Leaderboards:** https://aider.chat/docs/leaderboards/

---

## 📝 Command Line Reference

### Most Used Options

```bash
# Model
--model <name>              # LLM model
--weak-model <name>         # For simple tasks
--editor-model <name>       # For architect mode

# Files
--read <file>               # Read-only file
--yes                       # Auto-approve all

# Edit Format
--edit-format <format>      # diff/whole/udiff/architect

# Git
--auto-commits              # Auto git commit
--no-git                    # Disable git
--dirty-commits             # Allow dirty git

# Context
--map-tokens <n>            # Repo map size
--no-map                    # Disable repo map
--cache-prompts             # Enable prompt caching

# Behavior
--auto-lint                 # Auto lint
--auto-test                 # Auto test
--stream                    # Stream responses
--no-stream                 # No streaming

# UI
--dark-mode                 # Dark mode
--no-pretty                 # Disable pretty output
--fancy-input               # Enhanced input

# Special Modes
--ask                       # Ask mode only
--architect                 # Architect mode
--watch-files               # Watch mode
--voice                     # Voice input
--gui                       # Streamlit GUI

# Other
--config <file>             # Config file
--upgrade                   # Upgrade aider
--version                   # Show version
--help                      # Show help
```

---

## 🎯 Quick Start Scenarios

### Scenario 1: First Time User

```bash
# Install
pip install aider-chat

# Setup API key
echo "OPENAI_API_KEY=sk-..." > ~/.env

# Start with a file
aider main.py

# Try simple request
> Add a function to read a JSON file
```

### Scenario 2: Existing Project

```bash
# Navigate to project
cd my-project

# Create config
cat > .aider.conf.yml << EOF
model: sonnet
auto-commits: true
auto-lint: true
EOF

# Start with relevant files
aider src/main.py src/utils.py

# Add feature
> Implement user authentication
```

### Scenario 3: Code Review

```bash
# Read-only mode
aider --ask

# Add file
/add src/new_feature.py

# Review
> Review this code for bugs, performance issues, and best practices
```

### Scenario 4: Quick Fix

```bash
# Watch mode
aider --watch-files &

# In your editor, add comment:
# ai: fix this function to handle None values

# Aider processes automatically
```

---

**Quick Reference Version:** 1.0
**Last Updated:** 2025-11-13
**Aider Version:** Latest

---

*Lưu file này để tham khảo nhanh! 📌*
