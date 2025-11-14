# Local LLM Deployment Guide for M1 MacBook

## System Specifications
- **Current Setup**: M1 MacBook with 16GB RAM
- **Future Consideration**: 128GB RAM machine
- **Primary Tool**: Ollama
- **Focus**: CLI tooling and file system interaction

---

## Table of Contents
1. [Initial Setup](#initial-setup)
2. [Recommended Stack Architecture](#recommended-stack-architecture)
3. [Step-by-Step Installation](#step-by-step-installation)
4. [Model Selection by RAM](#model-selection-by-ram)
5. [CLI Tools Integration](#cli-tools-integration)
6. [Scaling to 128GB RAM](#scaling-to-128gb-ram)
7. [Troubleshooting](#troubleshooting)

---

## Initial Setup

### Prerequisites
- macOS with M1/M2/M3 chip
- Homebrew installed
- Terminal access
- At least 10GB free disk space

### Install Homebrew (if not installed)
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

---

## Recommended Stack Architecture

### For 16GB RAM: **Direct Ollama + CLI Tool Integration**

**Recommended Approach**: Skip Gollama and integrate CLI tools directly with Ollama.

**Rationale**:
- Ollama provides a robust REST API that CLI tools can consume directly
- Gollama is primarily a TUI (Terminal User Interface) for managing Ollama models
- Direct integration is simpler, more maintainable, and has less overhead
- Open-Interpreter and similar tools have native Ollama support

**Stack Components**:
1. **Ollama** - Model runtime and API server
2. **CLI Tool** (choose one or combine):
   - **Open-Interpreter** - For interactive coding and file system operations
   - **Aider** - For AI pair programming
   - **Continue.dev** - For IDE integration
3. **Optional**: Custom scripts using Ollama's API

---

## Step-by-Step Installation

### Step 1: Install Ollama

```bash
# Download and install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Verify installation
ollama --version

# Start Ollama service (runs automatically on install)
# To manually start: ollama serve
```

### Step 2: Pull Recommended Models (16GB RAM)

```bash
# Small, fast model for quick tasks (4GB)
ollama pull phi3:mini

# Medium model - best balance for 16GB (7-8GB)
ollama pull llama3.2:3b

# Larger model if you close other apps (13GB+)
ollama pull codellama:13b

# Verify models are downloaded
ollama list
```

### Step 3: Test Ollama

```bash
# Interactive chat
ollama run llama3.2:3b

# Type your message and press Enter
# Type /bye to exit

# Test via API
curl http://localhost:11434/api/generate -d '{
  "model": "llama3.2:3b",
  "prompt": "Why is the sky blue?",
  "stream": false
}'
```

### Step 4: Install Open-Interpreter

```bash
# Install Python if not installed
brew install python3

# Install Open-Interpreter
pip3 install open-interpreter

# Verify installation
interpreter --version
```

### Step 5: Configure Open-Interpreter with Ollama

```bash
# Create configuration file
mkdir -p ~/.config/open-interpreter
cat > ~/.config/open-interpreter/config.yaml << 'EOF'
model: "ollama/llama3.2:3b"
api_base: "http://localhost:11434"
temperature: 0.7
max_tokens: 4000
context_window: 8192
EOF

# Or configure via command line flags
interpreter --model ollama/llama3.2:3b --api_base http://localhost:11434
```

### Step 6: Alternative CLI Tools

#### Option A: Install Aider (Code-focused)
```bash
pip3 install aider-chat

# Run with Ollama
aider --model ollama/codellama:13b --no-auto-commits
```

#### Option B: Install Continue.dev (VS Code/JetBrains)
```bash
# Install VS Code extension
code --install-extension continue.continue

# Or download from: https://marketplace.visualstudio.com/items?itemName=Continue.continue

# Configure in VS Code settings:
# ~/.continue/config.json
```

Example `~/.continue/config.json`:
```json
{
  "models": [
    {
      "title": "Llama 3.2",
      "provider": "ollama",
      "model": "llama3.2:3b",
      "apiBase": "http://localhost:11434"
    }
  ]
}
```

---

## Model Selection by RAM

### 16GB RAM Recommendations

| Model | Size | Use Case | RAM Usage |
|-------|------|----------|-----------|
| `phi3:mini` | 3.8GB | Quick queries, testing | ~5GB |
| `llama3.2:3b` | 2GB | General purpose, balanced | ~4GB |
| `codellama:7b` | 3.8GB | Code generation | ~6GB |
| `mistral:7b` | 4.1GB | Strong reasoning | ~7GB |
| `codellama:13b` | 7.3GB | Advanced coding (close other apps) | ~10GB |

**Best for 16GB**: `llama3.2:3b` or `mistral:7b`

### Quantization Options
```bash
# Default is Q4_0 (good balance)
ollama pull llama3.2:3b

# Smaller, faster (lower quality)
ollama pull llama3.2:3b-q2_K

# Larger, better quality
ollama pull llama3.2:3b-q8_0
```

---

## CLI Tools Integration

### Open-Interpreter Usage Examples

```bash
# Start interactive session
interpreter

# Execute with specific model
interpreter --model ollama/llama3.2:3b

# Non-interactive mode
echo "Create a Python script that lists all files in current directory" | interpreter --model ollama/llama3.2:3b

# Safe mode (asks before executing)
interpreter --safe_mode always
```

### File System Operations with Open-Interpreter

```bash
# Analyze local files
interpreter "Analyze all Python files in ./src and create a summary"

# Batch operations
interpreter "Rename all .txt files in ./documents to follow kebab-case naming"

# Code refactoring
interpreter "Refactor all JavaScript files to use async/await instead of promises"
```

### Custom Script Integration

Create `~/bin/llm-cli`:
```bash
#!/bin/bash
# Simple CLI wrapper for Ollama

MODEL="${OLLAMA_MODEL:-llama3.2:3b}"
PROMPT="$*"

if [ -z "$PROMPT" ]; then
    echo "Usage: llm-cli <prompt>"
    exit 1
fi

curl -s http://localhost:11434/api/generate -d "{
  \"model\": \"$MODEL\",
  \"prompt\": \"$PROMPT\",
  \"stream\": false
}" | jq -r '.response'
```

Make executable:
```bash
chmod +x ~/bin/llm-cli
export PATH="$HOME/bin:$PATH"

# Usage
llm-cli "Explain Python decorators"
```

---

## Scaling to 128GB RAM

### What Changes with 128GB RAM?

#### 1. Model Selection Expands Significantly
```bash
# You can now run much larger models:
ollama pull llama3.1:70b       # ~40GB RAM
ollama pull codellama:70b      # ~40GB RAM
ollama pull mixtral:8x7b       # ~26GB RAM
ollama pull llama3.1:405b-q4   # ~230GB (quantized, still too large)

# Or run multiple models simultaneously
ollama run llama3.1:70b &      # General purpose
ollama run codellama:70b &     # Code-specific
```

#### 2. Parallel Model Loading
```bash
# Create a model pool script
cat > ~/bin/ollama-pool << 'EOF'
#!/bin/bash
# Load multiple models into memory

ollama run llama3.1:70b "preload" &
ollama run codellama:34b "preload" &
ollama run mistral:7b "preload" &

echo "Model pool loaded"
EOF

chmod +x ~/bin/ollama-pool
```

#### 3. Context Window Benefits
With more RAM, you can increase context windows:
```bash
# In Open-Interpreter config
interpreter --model ollama/llama3.1:70b --context_length 32000
```

#### 4. Stack Architecture Remains the Same
- **No fundamental changes** to the Ollama + CLI tool approach
- Same API endpoints
- Same integration patterns
- Just better performance and model choices

### Recommended Models for 128GB

| Model | Size | Use Case | RAM Usage |
|-------|------|----------|-----------|
| `llama3.1:70b` | 40GB | Best reasoning, long context | ~45GB |
| `codellama:70b` | 40GB | Advanced code generation | ~45GB |
| `mixtral:8x7b` | 26GB | Mixture of experts, versatile | ~30GB |
| `qwen2.5:72b` | 41GB | Strong multilingual support | ~46GB |
| `deepseek-coder:33b` | 19GB | Code-specialized | ~22GB |

You could run 2-3 large models simultaneously with 128GB.

---

## Troubleshooting

### Ollama Service Issues
```bash
# Check if Ollama is running
ps aux | grep ollama

# Restart Ollama
killall ollama
ollama serve

# Check logs
tail -f ~/.ollama/logs/server.log
```

### Out of Memory Errors
```bash
# Switch to smaller model
ollama run phi3:mini

# Or use quantized version
ollama pull llama3.2:3b-q2_K

# Check available RAM
vm_stat | grep "Pages free"
```

### API Connection Issues
```bash
# Test API endpoint
curl http://localhost:11434/api/tags

# Check firewall settings
sudo pfctl -s rules | grep 11434

# Verify port binding
lsof -i :11434
```

### Model Download Issues
```bash
# Clear cache and retry
rm -rf ~/.ollama/models/<model-name>
ollama pull <model-name>

# Check disk space
df -h
```

### Performance Optimization
```bash
# Increase thread count (for larger RAM)
export OLLAMA_NUM_THREADS=8

# Set GPU layers (M1 uses unified memory)
export OLLAMA_NUM_GPU=1

# Restart Ollama with new settings
ollama serve
```

---

## Quick Reference Commands

```bash
# Model Management
ollama list                    # List installed models
ollama pull <model>           # Download a model
ollama rm <model>             # Remove a model
ollama run <model>            # Interactive chat
ollama show <model>           # Model details

# Service Management
ollama serve                  # Start server
killall ollama               # Stop server
ollama --version             # Check version

# Open-Interpreter
interpreter                   # Start interactive session
interpreter --local          # Use local models only
interpreter --model <model>  # Specify model
interpreter --help           # Show all options

# API Testing
curl http://localhost:11434/api/generate -d '{"model":"llama3.2:3b","prompt":"test"}'
```

---

## Summary

### For 16GB RAM:
- **Use**: Ollama + Open-Interpreter (direct integration)
- **Best Model**: `llama3.2:3b` or `mistral:7b`
- **Avoid**: Gollama (unnecessary layer)
- **CLI Tools**: Open-Interpreter, Aider, or custom scripts

### For 128GB RAM:
- **Stack**: Same architecture (Ollama + CLI tools)
- **Best Models**: `llama3.1:70b`, `codellama:70b`, `mixtral:8x7b`
- **New Capability**: Run multiple large models simultaneously
- **No architectural changes**: Just better model selection

### Key Takeaway:
Direct Ollama integration with CLI tools is the cleanest, most maintainable approach for both RAM configurations. The core stack remains unchanged—only model selection scales with available RAM.
