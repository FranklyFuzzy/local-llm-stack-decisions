I'll search for current best practices on deploying local LLMs with Ollama on M1 Macs and the various integration options you're asking about.

Based on the search results, I now have comprehensive information to provide a detailed guide. Let me create a well-structured markdown document.

---

# Local LLM Deployment Guide: Ollama on M1 MacBook

## Table of Contents
1. [System Requirements & Prerequisites](#system-requirements--prerequisites)
2. [Installation & Setup](#installation--setup)
3. [Architecture & Stack Decision](#architecture--stack-decision)
4. [Filesystem Access & CLI Integration](#filesystem-access--cli-integration)
5. [Model Selection & Optimization](#model-selection--optimization)
6. [Memory Scaling: 16GB vs 128GB](#memory-scaling-16gb-vs-128gb)
7. [Troubleshooting & Performance](#troubleshooting--performance)

---

## System Requirements & Prerequisites

### M1 16GB MacBook Specifications
- **Minimum RAM**: 16GB (sufficient for 7B-13B models)
- **Storage**: 20GB+ free space (models range 2.7GB-43GB+)
- **macOS**: Monterey or later (Big Sur 11+)
- **Processor**: M1 chip with unified memory architecture

### What You'll Need
- Homebrew (package manager)
- Terminal/iTerm2
- Python 3.8+ (optional, for advanced integrations)
- ~30-50GB total disk space for multiple models

---

## Installation & Setup

### Method 1: Homebrew (Recommended)

```bash
# Install Ollama via Homebrew
brew install ollama

# Verify installation
ollama --version

# Start Ollama as background service
brew services start ollama

# Test connectivity
curl http://localhost:11434/
# Should return: "Ollama is running"
```

### Method 2: Direct Download

```bash
# Download and install
curl -L https://ollama.com/download/Ollama-darwin.zip -o ~/Downloads/Ollama.zip
unzip ~/Downloads/Ollama.zip -d /Applications/

# Add to PATH
echo 'export PATH="/Applications/Ollama.app/Contents/MacOS:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Initial Configuration

```bash
# Set model storage location (optional, for larger drives)
export OLLAMA_MODELS=/path/to/larger/drive/models

# Enable remote access (if needed)
export OLLAMA_HOST=0.0.0.0:11434
export OLLAMA_ORIGINS=*

# Add to ~/.zshrc for persistence
echo 'export OLLAMA_MODELS=~/.ollama/models' >> ~/.zshrc
```

---

## Architecture & Stack Decision

### Option Analysis

#### **1. Native Ollama (Recommended for 16GB)**

**Pros:**
- Direct GPU acceleration via Metal Performance Shaders
- Unified memory architecture utilization
- Fastest inference speeds on M1
- No Docker overhead
- Simple CLI integration

**Cons:**
- Limited to local machine access
- No containerization benefits

**Best for:** Single-user development, local AI assistants

```bash
# Quick start
ollama pull llama3.2:7b
ollama run llama3.2:7b
```

#### **2. Ollama + Docker (Not Recommended for M1)**

**Critical Issue:** Docker Desktop on macOS does NOT expose Apple GPU to containers—only CPU access. Performance drops ~50%.

**Only use if:**
- You need multi-machine deployment
- You're willing to accept CPU-only performance

```bash
# If you must use Docker
docker run -d -p 11434:11434 ollama/ollama:latest
```

#### **3. Ollama + MCP (Model Context Protocol) for Filesystem Access**

**Best for:** CLI tools needing filesystem interaction

**Architecture:**
```
Your App → MCP Client → MCP Server (Filesystem Module) → Local Filesystem
                     ↓
              Ollama LLM (via MCP)
```

**Pros:**
- Standardized tool integration
- Filesystem access with security boundaries
- Works with Open-Interpreter, OpenCode patterns
- Modular architecture

**Implementation:**

```bash
# Install MCP Ollama Server
git clone https://github.com/Sethuram2003/mcp-ollama_server.git
cd mcp-ollama_server

# Install dependencies
pip install -r requirements.txt

# Run MCP server
python client_mcp/client.py file_system/file_system.py
```

#### **4. Gollama (Image Sharing) - Not Recommended**

**Status:** Gollama is not a standard tool in the Ollama ecosystem (2025). Focus on native Ollama instead.

**Why skip it:**
- No clear advantage over native Ollama for 16GB setup
- Adds unnecessary complexity
- Better alternatives exist (MCP for tool integration)

### **Recommended Stack for M1 16GB**

```
┌─────────────────────────────────────────┐
│     Your CLI Tool / Application         │
└────────────────┬────────────────────────┘
                 │
        ┌────────▼─────────┐
        │  Ollama (Native) │ ◄─── Direct Metal GPU
        │  Port 11434      │      Acceleration
        └────────┬─────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
┌───▼───┐  ┌────▼────┐  ┌───▼────┐
│ Chat  │  │ API     │  │ MCP    │
│ CLI   │  │ Access  │  │ Server │
└───────┘  └─────────┘  └────────┘
                           │
                    ┌──────▼──────┐
                    │ Filesystem  │
                    │ Access      │
                    └─────────────┘
```

---

## Filesystem Access & CLI Integration

### Option A: Direct API Access (Simplest)

```python
# Python integration for CLI tools
import ollama

response = ollama.chat(
    model='llama3.2:7b',
    messages=[
        {'role': 'user', 'content': 'Analyze this file: /path/to/file'}
    ]
)
print(response['message']['content'])
```

### Option B: MCP Integration (For Tool Calling)

```bash
# Install MCP filesystem module
pip install mcp

# Create config file: ~/.config/mcp/mcp-config.json
{
  "mcpServers": {
    "filesystem": {
      "command": "python",
      "args": ["-m", "mcp.server.filesystem"],
      "env": {
        "MCP_ALLOWED_PATHS": "/Users/yourname/projects"
      }
    }
  }
}
```

### Option C: Shell Integration (For Open-Interpreter Pattern)

```bash
# Create wrapper script: ~/bin/ai-interpreter
#!/bin/bash
PROMPT="$@"
echo "$PROMPT" | ollama run llama3.2:7b

# Make executable
chmod +x ~/bin/ai-interpreter

# Usage
ai-interpreter "List all Python files in current directory"
ai-interpreter "Explain the code in main.py"
```

### Option D: VS Code Integration (Recommended for Developers)

```bash
# Install Continue extension
code --install-extension Continue.continue

# Configure ~/.continue/config.json
{
  "models": [
    {
      "title": "Llama 3.2 7B",
      "model": "llama3.2:7b",
      "provider": "ollama",
      "apiBase": "http://localhost:11434"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Qwen2.5-Coder 1.5B",
    "model": "qwen2.5-coder:1.5b",
    "provider": "ollama",
    "apiBase": "http://localhost:11434"
  }
}
```

---

## Model Selection & Optimization

### For M1 16GB MacBook

| Model | Size | Speed | Quality | Recommendation |
|-------|------|-------|---------|-----------------|
| Phi-3 Mini | 2.7GB | ⚡⚡⚡ | ⭐⭐ | Quick tasks |
| Llama 3.2 3B | 2GB | ⚡⚡⚡ | ⭐⭐⭐ | Best for 8GB |
| Llama 3.2 7B | 4.7GB | ⚡⚡ | ⭐⭐⭐⭐ | **Recommended** |
| Mistral 7B | 4.1GB | ⚡⚡ | ⭐⭐⭐⭐ | Fast & capable |
| Llama 3.1 13B | 7.3GB | ⚡ | ⭐⭐⭐⭐⭐ | High quality |
| CodeLlama 7B | 3.8GB | ⚡⚡ | ⭐⭐⭐⭐ | Code tasks |

### Installation Commands

```bash
# Recommended: Llama 3.2 7B
ollama pull llama3.2:7b
ollama run llama3.2:7b

# Alternative: Mistral 7B (faster)
ollama pull mistral
ollama run mistral

# For coding tasks
ollama pull codellama:7b
ollama run codellama:7b

# List all installed models
ollama list

# Remove a model
ollama rm llama3.2:7b
```

### Performance Tuning for 16GB

```bash
# Reduce context window for faster responses
ollama run llama3.2:7b --ctx-size 512

# Adjust temperature for consistency
ollama run llama3.2:7b --temperature 0.5

# Set keep-alive to prevent model unloading
export OLLAMA_KEEP_ALIVE=10m
```

---

## Memory Scaling: 16GB vs 128GB

### What Changes with 128GB?

#### **Stack Architecture: No Change**
- Still use native Ollama (not Docker)
- Same MCP integration approach
- Same CLI tooling patterns
- **No architectural changes needed**

#### **What Actually Changes**

| Aspect | 16GB M1 | 128GB M1 Ultra |
|--------|---------|-----------------|
| **Model Size** | 7B-13B max | 70B+ easily |
| **Quantization** | Q4_K_M required | Q8_0 or F16 possible |
| **Concurrent Models** | 1-2 models | 5-10 models simultaneously |
| **Context Window** | 512-2048 tokens | 4096-8192 tokens |
| **Inference Speed** | 15-25 tok/s | 25-40 tok/s |
| **Batch Processing** | Single requests | Parallel batches |

### Model Recommendations by RAM

**16GB Configuration:**
```bash
ollama pull llama3.2:7b          # Primary model
ollama pull codellama:7b         # Secondary for coding
```

**128GB Configuration:**
```bash
ollama pull llama3.1:70b         # Large model
ollama pull mixtral:8x7b         # Mixture of Experts
ollama pull llama3.1:13b         # Secondary model
ollama pull codellama:34b        # Advanced coding
# Run multiple simultaneously without performance loss
```

### Configuration for 128GB

```bash
# ~/.zshrc additions for 128GB machine
export OLLAMA_MAX_LOADED_MODELS=5
export OLLAMA_NUM_PARALLEL=4
export OLLAMA_KEEP_ALIVE=30m

# Increase context window
export OLLAMA_CONTEXT_LENGTH=8192
```

### Performance Comparison

```bash
# 16GB M1: Llama 3.2 7B
# ~20 tokens/second

# 128GB M1 Ultra: Llama 3.1 70B
# ~15 tokens/second (acceptable for large model)

# 128GB M1 Ultra: Multiple 13B models
# ~25 tokens/second each (parallel processing)
```

---

## Troubleshooting & Performance

### Common Issues & Solutions

#### Issue 1: "Port 11434 already in use"
```bash
# Find process using port
lsof -i :11434

# Kill existing process
pkill -f ollama

# Use different port
export OLLAMA_HOST=127.0.0.1:11435
ollama serve
```

#### Issue 2: Out of Memory on 16GB
```bash
# Use quantized models (Q4 instead of Q8)
ollama pull llama3.2:7b-q4_0

# Close unnecessary applications
osascript -e 'tell application "Safari" to close every tab of every window'

# Check memory usage
vm_stat | grep "Pages free"
```

#### Issue 3: Slow Performance
```bash
# Verify Metal GPU is being used
log stream --predicate 'subsystem == "com.apple.ane"' --info

# Disable Spotlight indexing for models
sudo mdutil -i off ~/.ollama

# Use SSD for model storage (not HDD)
# Move models to faster drive if needed
```

#### Issue 4: Model Download Failures
```bash
# Clear corrupted downloads
rm -rf ~/.ollama/models/blobs/incomplete/*

# Retry with verbose output
OLLAMA_DEBUG=1 ollama pull llama3.2:7b

# Check available disk space
df -h
```

### Performance Monitoring

```bash
# Monitor in real-time
top -o cpu

# Check GPU usage (M1/M2/M3)
sudo powermetrics --samplers gpu_power -i1000 -n5

# Monitor thermal status
sudo powermetrics --samplers thermal -i1000

# Check memory pressure
vm_stat
```

### Optimization Checklist

- [ ] Use native Ollama (not Docker)
- [ ] Select appropriate model size for RAM
- [ ] Use quantized models (Q4_K_M for 16GB)
- [ ] Store models on SSD
- [ ] Close unnecessary applications
- [ ] Set `OLLAMA_KEEP_ALIVE` appropriately
- [ ] Disable Spotlight indexing on model directory
- [ ] Use MCP for filesystem access (not direct file I/O)
- [ ] Monitor with Activity Monitor during heavy use
- [ ] Keep macOS and Ollama updated

---

## Quick Reference: Essential Commands

```bash
# Installation & Service
brew install ollama
brew services start ollama
brew services stop ollama
ollama --version

# Model Management
ollama pull llama3.2:7b
ollama list
ollama rm model-name
ollama show llama3.2:7b

# Running Models
ollama run llama3.2:7b
ollama run llama3.2:7b "Your prompt here"

# API Access
curl http://localhost:11434/api/generate \
  -d '{"model":"llama3.2:7b","prompt":"Hello"}'

# Debugging
cat ~/.ollama/logs/server.log
OLLAMA_DEBUG=1 ollama serve
```

---

## Summary: 16GB vs 128GB Stack

**The stack remains identical.** The only differences are:

1. **Model choices** (7B → 70B models)
2. **Quantization levels** (Q4 → Q8/F16)
3. **Concurrent model loading** (1-2 → 5-10)
4. **Context window size** (512 → 8192 tokens)
5. **Batch processing capability** (single → parallel)

**No architectural changes needed.** Use native Ollama with MCP for filesystem access on both configurations.
