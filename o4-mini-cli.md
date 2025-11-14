# Deploying Local LLMs on an M1 16 GB MacBook with Ollama

This guide walks you through setting up a local LLM workflow on your M1 MacBook Air/Pro (16 GB RAM) using Ollama as your primary engine. You’ll see three stack options—and recommendations—and learn how upgrading to a 128 GB machine affects your choices.

---

## Table of Contents

* [Prerequisites](#prerequisites)  
* [Option Comparison](#option-comparison)  
* [Recommended Stack](#recommended-stack)  
* [Step-by-Step Setup (16 GB)](#step-by-step-setup-16gb)  
* [Adapting for 128 GB](#adapting-for-128gb)  
* [Appendix: Troubleshooting & Tips](#appendix)

---

## Prerequisites

* macOS 12.0+ on Apple Silicon (M1/M2)  
* Homebrew installed (`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`)  
* 16 GB RAM (for 7B–13B models comfortably)  
* Basic familiarity with Terminal (bash/zsh)

---

## Option Comparison

| Stack Option                     | Pros                                                                           | Cons                                                             |
|----------------------------------|--------------------------------------------------------------------------------|------------------------------------------------------------------|
| **1. Ollama-only**               | * Native image pulling & runtime<br>* Simple CLI (`ollama pull`, `ollama run`)| * Limited custom CLI hooks<br>* Less extensible for tooling      |
| **2. Gollama + Integration CLI** | * Shareable OCI-style images<br>* Works with OpenCode, Open-Interpreter        | * Extra layer to manage<br>* More complex initial setup         |
| **3. Custom CLI + Ollama API**   | * Fully tailored workflow<br>* Direct host-FS integration via API             | * You must build & maintain the CLI<br>* Longer ramp-up time     |

---

## Recommended Stack

For most users on 16 GB machines, **Option 1 (Ollama-only)** is the smoothest:

* Leverages Ollama’s native M1 acceleration  
* Zero-conf: `brew install ollama && ollama pull ...`  
* Built-in file‐mount flags (`--mount`) for host FS access  

If you foresee collaborating or distributing images across teams, consider **Option 2**. For maximum flexibility and custom commands, **Option 3** is ideal—but expect to write Go/Python scripts around Ollama’s HTTP API.

---

## Step-by-Step Setup (16 GB)

### 1. Install Ollama

```bash
brew install ollama
```

Verify:
```bash
ollama version
```

### 2. Pull Your LLM Image

```bash
ollama pull llama2:7b
```

> Tip: On 16 GB, stick to 7B–13B models. 70B models will OOM.

### 3. Run a REPL with Host FS Access

```bash
ollama run llama2:7b \
  --mount "$(pwd)/data:/app/data" \
  --cpu 4 --memory 8GB
```

* `--mount host_path:container_path`  
* Adjust CPU threads & memory flags to fit 16 GB

### 4. CLI Integration

You can wrap common calls in a shell function or script:

```bash
# file: ./run_model.sh
#!/usr/bin/env bash
ollama run llama2:7b \
  --mount "$(pwd)/$1:/input" \
  --json <<EOF
{"prompt": "Process file /input"}
EOF
```

Make it executable:
```bash
chmod +x run_model.sh
./run_model.sh data.txt
```

### 5. Automate with Make

```makefile
.PHONY: serve chat

serve:
    ollama serve llama2:7b \
      --port 11434 \
      --mount ./data:/app/data

chat:
    ollama run llama2:7b --prompt "$(msg)"

# Usage:
# make chat msg="Hello, world!"
```

---

## Adapting for a 128 GB Mac

With 128 GB RAM:

* **Model size**  
  * You can comfortably load 30B+ models.  
* **Memory flags**  
  * Increase `--memory` to half your RAM (e.g., `64GB`).  
* **Stack impact**  
  * **Option 1:** unchanged—just use larger models.  
  * **Option 2:** even easier to share big-model images.  
  * **Option 3:** no change in tooling architecture; just adjust resource allocations.  

---

## Appendix: Troubleshooting & Tips

* **OOM Errors**: lower `--cpu` threads or `--memory`; try a smaller model.  
* **Networking**: use `ollama serve` + `curl http://localhost:11434/v1/completions` for HTTP API.  
* **Disk Space**: LLM images can be multi-GB. Clean with `ollama remove llama2:7b` when done.  
* **Versioning**: pin specific versions (`ollama pull llama2:7b@v2.0`) to ensure reproducibility.

---

Copy this Markdown into your editor of choice and customize paths, model names, or scripts as needed. Enjoy your local LLM deployment!
