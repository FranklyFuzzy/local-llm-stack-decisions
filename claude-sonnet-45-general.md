# Local LLM Deployment Guide for M1 MacBook (16GB)

## Table of Contents
1. [System Requirements & Preparation](#system-requirements--preparation)
2. [Stack Recommendation for Your Use Case](#stack-recommendation-for-your-use-case)
3. [Installation Guide](#installation-guide)
4. [Model Selection for 16GB RAM](#model-selection-for-16gb-ram)
5. [Integration with Development Tools](#integration-with-development-tools)
6. [Scaling to 128GB RAM](#scaling-to-128gb-ram)
7. [Troubleshooting & Optimization](#troubleshooting--optimization)

---

## System Requirements & Preparation

### Prerequisites
- macOS 12.0 or later
- M1/M2/M3 chip (Apple Silicon)
- At least 16GB unified memory
- 50GB+ free disk space for models
- Xcode Command Line Tools

### Initial Setup
```bash
# Install Xcode Command Line Tools
xcode-select --install

# Install Homebrew (if not already installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

---

## Stack Recommendation for Your Use Case

### Recommended Architecture

Based on your requirements (writing, file management, coding, scripting, creative writing), I recommend a **hybrid approach**:

**Primary Stack:**
1. **Ollama** - Core LLM runtime engine
2. **Open WebUI** - Primary chat interface for writing & creative work
3. **Continue.dev or Aider** - Code editor integration (VS Code/Cursor)
4. **Open Interpreter** - CLI automation for file management & scripting

### Why This Stack?

| Tool | Use Case | Advantages |
|------|----------|------------|
| **Ollama** | Base runtime | Native Apple Silicon support, efficient memory management, easy model switching |
| **Open WebUI** | Writing & creative work | Clean UI, conversation history, document uploads, model comparison, no internet required |
| **Continue.dev/Aider** | Coding in editor | Inline code suggestions, refactoring, direct IDE integration |
| **Open Interpreter** | Automation & scripting | Execute code, file operations, system tasks via natural language |

### Alternative: LM Studio
**Consider LM Studio if you prefer:**
- All-in-one GUI application
- Simple drag-and-drop model management
- Built-in API server
- Less CLI comfort required

**Skip Gollama** - It's primarily a TUI (terminal user interface) for Ollama management. Open WebUI provides better functionality for your use case.

---

## Installation Guide

### Step 1: Install Ollama

```bash
# Download and install Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Verify installation
ollama --version

# Start Ollama service (runs automatically after install)
# Check if running:
curl http://localhost:11434
# Should return: "Ollama is running"
```

### Step 2: Pull Recommended Models (16GB RAM)

```bash
# For general use - excellent quality/speed balance
ollama pull llama3.2:3b          # Fast, lightweight (2GB)
ollama pull mistral:7b           # Best all-rounder (4.1GB)
ollama pull qwen2.5:7b           # Strong at coding (4.7GB)

# For creative writing
ollama pull llama3.1:8b          # Creative and coherent (4.7GB)

# For coding (if you have room)
ollama pull codellama:7b         # Code-specialized (3.8GB)
ollama pull deepseek-coder:6.7b  # Excellent for code (3.8GB)

# List installed models
ollama list

# Test a model
ollama run mistral:7b "Write a haiku about coding"
```

### Step 3: Install Open WebUI

```bash
# Install using Docker (recommended)
# First, install Docker Desktop for Mac from docker.com

# Run Open WebUI
docker run -d -p 3000:8080 \
  --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main

# Access at: http://localhost:3000
```

**Alternative: Install via Python (if you prefer no Docker)**
```bash
pip install open-webui
open-webui serve
```

**First-time setup:**
1. Open http://localhost:3000
2. Create an admin account
3. Models from Ollama will auto-populate
4. Configure settings → Interface → Theme/Preferences

### Step 4: Install Continue.dev (VS Code/Cursor Integration)

```bash
# Open VS Code or Cursor
# Install Continue extension from marketplace
# Or via command line:
code --install-extension continue.continue

# Configure Continue (creates ~/.continue/config.json)
```

**Continue configuration for Ollama:**
```json
{
  "models": [
    {
      "title": "Mistral 7B",
      "provider": "ollama",
      "model": "mistral:7b"
    },
    {
      "title": "Qwen2.5 Coder",
      "provider": "ollama",
      "model": "qwen2.5:7b"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Qwen2.5 3B",
    "provider": "ollama",
    "model": "qwen2.5:3b"
  }
}
```

### Step 5: Install Open Interpreter (CLI Automation)

```bash
# Install Open Interpreter
pip install open-interpreter

# Configure for Ollama
interpreter --local

# Or create config file at ~/.config/open-interpreter/config.yaml
```

**Open Interpreter config for Ollama:**
```yaml
# ~/.config/open-interpreter/config.yaml
model: "ollama/mistral:7b"
api_base: "http://localhost:11434"
temperature: 0.7
auto_run: false  # Set to true to skip confirmation prompts
```

**Usage examples:**
```bash
# Start interactive session
interpreter

# Example commands within interpreter:
# "Organize my Downloads folder by file type"
# "Create a Python script that renames files in batch"
# "Find all TODO comments in my project"
```

---

## Model Selection for 16GB RAM

### Memory Budget Breakdown
- macOS + apps: ~4-6GB
- Available for LLM: ~10-12GB
- Recommended model size: 7B parameters or smaller

### Recommended Models by Task

#### General Chat & Writing
```bash
ollama pull llama3.1:8b          # 4.7GB - Best overall
ollama pull mistral:7b           # 4.1GB - Fast and capable
ollama pull phi3:3.8b            # 2.3GB - Very efficient
```

#### Coding & Technical
```bash
ollama pull qwen2.5:7b           # 4.7GB - Strong reasoning
ollama pull deepseek-coder:6.7b  # 3.8GB - Code specialist
ollama pull codellama:7b         # 3.8GB - Meta's code model
```

#### Creative Writing
```bash
ollama pull llama3.1:8b          # 4.7GB - Excellent prose
ollama pull neural-chat:7b       # 4.1GB - Conversational
```

#### Lightweight/Fast
```bash
ollama pull llama3.2:3b          # 2GB - Quick responses
ollama pull phi3:mini            # 2.3GB - Microsoft's efficient model
```

### Model Switching Strategy
Keep 2-3 models installed:
1. One 7B model for quality work (Mistral or Qwen2.5)
2. One 3B model for quick tasks (Llama3.2:3b)
3. One specialized model (coding or creative)

Switch models on-demand in Open WebUI dropdown.

---

## Integration with Development Tools

### VS Code/Cursor with Continue.dev

**Keyboard shortcuts:**
- `Cmd+I` - Inline edit/suggestion
- `Cmd+Shift+M` - Open Continue sidebar
- `Cmd+Shift+L` - Quick chat

**Common workflows:**
```
# Highlight code, then:
"Add error handling to this function"
"Write unit tests for this code"
"Refactor this to be more efficient"
"Explain what this code does"
"Convert this to TypeScript"
```

### Alternative: Aider (Terminal-based)

```bash
# Install Aider
pip install aider-chat

# Configure for Ollama
aider --model ollama/qwen2.5:7b --no-auto-commits

# Use in your project directory
cd ~/my-project
aider

# Example commands:
# /add file.py  - Add file to context
# /help         - Show commands
# "Refactor the User class to use dataclasses"
```

### Open Interpreter for File Management

**Example automation scripts:**

```bash
# Organize downloads
interpreter
> "Move all PDFs from Downloads to Documents/PDFs organized by month"

# Batch file operations
> "Rename all images in this folder to include today's date"

# Development automation
> "Find all Python files with TODO comments and create a task list"

# Data processing
> "Convert all CSV files in this folder to JSON"
```

---

## Scaling to 128GB RAM

### What Changes with 128GB?

#### Model Size Upgrade Paths

**16GB Limitations:**
- Max practical size: 7-8B parameters
- One model loaded at a time
- Quantized models only (Q4, Q5)

**128GB Capabilities:**
- Run 70B+ parameter models
- Multiple models simultaneously
- Higher precision quantization (Q6, Q8)
- Larger context windows

#### Recommended Models for 128GB

```bash
# Top-tier models
ollama pull llama3.1:70b         # 40GB - GPT-4 class performance
ollama pull mixtral:8x7b         # 26GB - Fast, expert routing
ollama pull qwen2.5:72b          # 42GB - Excellent reasoning
ollama pull deepseek-coder:33b   # 19GB - Superior coding

# Run multiple models simultaneously
ollama pull llama3.1:70b         # Primary reasoning
ollama pull codellama:34b        # Dedicated coding
ollama pull mistral:7b           # Quick tasks

# Higher quality quantizations
ollama pull llama3.1:70b-q6      # Better quality than Q4
ollama pull mixtral:8x7b-q8      # Near-FP16 quality
```

### Stack Changes for 128GB

**Core stack remains the same**, but with enhanced capabilities:

1. **Ollama** - Same, but can handle larger models
2. **Open WebUI** - Can compare responses from multiple large models side-by-side
3. **Continue.dev** - Can use 70B models for complex refactoring
4. **Open Interpreter** - Can tackle more complex multi-step tasks

**Additional options at 128GB:**
- Run local embedding models alongside chat models
- Host multiple specialized models (coding, writing, analysis)
- Use larger context windows (32K+ tokens)
- Run quantization experiments

### Performance Considerations

**16GB System:**
- Model load time: 2-5 seconds
- Tokens/second: 20-40 (7B models)
- Context limit: 4K-8K tokens practical

**128GB System:**
- Model load time: 10-30 seconds (70B models)
- Tokens/second: 8-15 (70B models), 30-50 (7B models)
- Context limit: 32K+ tokens practical

**Note:** Larger models don't always mean better results for simple tasks. Use 7B models for quick work, 70B for complex reasoning.

---

## Troubleshooting & Optimization

### Performance Optimization

#### Ollama Environment Variables
```bash
# Add to ~/.zshrc or ~/.bashrc

# Increase context window
export OLLAMA_NUM_CTX=8192

# Adjust parallel processing
export OLLAMA_NUM_PARALLEL=2

# Set GPU layers (all layers for M1)
export OLLAMA_NUM_GPU=99

# Memory management
export OLLAMA_MAX_LOADED_MODELS=1  # For 16GB
# export OLLAMA_MAX_LOADED_MODELS=3  # For 128GB

# Reload shell
source ~/.zshrc
```

#### Model Selection Tips
```bash
# Check model size before pulling
ollama show mistral:7b --modelfile

# Remove unused models
ollama rm model-name

# Check system resources
ollama ps  # Show running models
```

### Common Issues & Fixes

#### Issue: Slow Generation Speed
```bash
# Solution 1: Use smaller model
ollama pull llama3.2:3b

# Solution 2: Reduce context size
# Add to model Modelfile:
PARAMETER num_ctx 2048

# Solution 3: Check Activity Monitor
# Ensure no other heavy apps running
```

#### Issue: Out of Memory
```bash
# Check available memory
vm_stat

# Kill Ollama and restart
pkill ollama
ollama serve

# Use more aggressive quantization
ollama pull mistral:7b-q4_0  # Smaller than default
```

#### Issue: Open WebUI Not Connecting to Ollama
```bash
# Verify Ollama is running
curl http://localhost:11434

# Restart Docker container
docker restart open-webui

# Check Docker host configuration
# Ensure --add-host=host.docker.internal:host-gateway is set
```

#### Issue: Continue.dev Not Finding Models
```bash
# Verify Ollama API
curl http://localhost:11434/api/tags

# Check Continue config
cat ~/.continue/config.json

# Restart VS Code
# Cmd+Shift+P → "Developer: Reload Window"
```

### Monitoring & Maintenance

#### Check Model Performance
```bash
# Benchmark a model
time ollama run mistral:7b "Write a short story about AI" --verbose

# Monitor resource usage
sudo powermetrics --samplers cpu_power,gpu_power -i 1000
```

#### Regular Maintenance
```bash
# Update Ollama
curl -fsSL https://ollama.com/install.sh | sh

# Update models (pulls latest version)
ollama pull mistral:7b

# Clean up unused models
ollama list
ollama rm old-model-name

# Check disk usage
du -sh ~/Library/Application\ Support/Ollama
```

### Optimization Tips by Use Case

#### For Writing & Creative Work
- Use Llama3.1:8b for quality prose
- Enable longer context in Open WebUI (Settings → Models → Context Length: 8192)
- Create custom system prompts for your writing style
- Use conversation branches to explore different directions

#### For Coding
- Use Qwen2.5:7b or DeepSeek-Coder for best results
- Set Continue.dev to use smaller model for autocomplete (3B)
- Use larger model for refactoring (7B+)
- Enable "Apply Diff" mode for code changes

#### For File Management & Scripting
- Use Open Interpreter with `auto_run: false` for safety
- Create shell aliases for common tasks:
  ```bash
  # Add to ~/.zshrc
  alias ai="interpreter"
  alias aiauto="interpreter --auto_run"
  ```

---

## Quick Start Cheat Sheet

### Daily Workflow

```bash
# Morning startup
ollama list  # Check available models

# Writing session
# Open http://localhost:3000 (Open WebUI)
# Select llama3.1:8b

# Coding session
# Open VS Code with Continue.dev
# Cmd+I for inline edits

# File automation
interpreter
> "Your task here"

# Switch models on the fly
ollama run qwen2.5:7b  # CLI
# Or use Open WebUI dropdown
```

### Essential Commands

```bash
# Ollama
ollama list                    # List models
ollama pull model:tag          # Download model
ollama rm model:tag            # Remove model
ollama run model "prompt"      # Quick test
ollama ps                      # Running models

# Docker (Open WebUI)
docker ps                      # Check status
docker logs open-webui         # View logs
docker restart open-webui      # Restart

# Open Interpreter
interpreter                    # Start session
interpreter --local            # Configure for local
interpreter --model ollama/mistral:7b  # Specify model
```

---

## Additional Resources

### Official Documentation
- Ollama: https://ollama.com/docs
- Open WebUI: https://docs.openwebui.com
- Continue.dev: https://continue.dev/docs
- Open Interpreter: https://docs.openinterpreter.com

### Model Repositories
- Ollama Library: https://ollama.com/library
- Hugging Face: https://huggingface.co/models

### Community
- Ollama Discord: https://discord.gg/ollama
- Open WebUI GitHub: https://github.com/open-webui/open-webui
- r/LocalLLaMA: https://reddit.com/r/LocalLLaMA

---

## Summary

**Recommended Stack:**
- **Ollama** (Runtime)
- **Open WebUI** (Chat interface)
- **Continue.dev** (IDE integration)
- **Open Interpreter** (CLI automation)

**For 16GB:** Focus on 7B models, keep 2-3 specialized models installed

**For 128GB:** Core stack unchanged, upgrade to 70B models, run multiple models simultaneously

**Key takeaway:** Start with Ollama + Open WebUI for general use, add Continue.dev when you start coding, and integrate Open Interpreter for automation tasks. This modular approach lets you scale based on your needs.
