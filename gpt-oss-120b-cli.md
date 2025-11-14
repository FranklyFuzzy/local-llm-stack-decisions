# Deploying Local LLMs on an M1 16 GB MacBook (and scaling to 128 GB)

Below is a **complete, copy‑and‑paste ready** guide that takes you from a fresh macOS install to a fully functional local LLM stack using **Ollama** as the core engine.  
It also covers optional UI/CLI tools, model‑sharing with **gollama**, and how the stack changes (or doesn’t) when you upgrade to a 128 GB machine.

---

## 1️⃣ Prerequisites

| Requirement | Details |
|------------|---------|
| macOS version | 11 Big Sur or later (M1 MacBook Pro 2020 works fine) |
| RAM | 16 GB (minimum for 7‑13 B models) – 128 GB lets you run 34 B‑70 B models with quantization |
| Disk | ≥ 8 GB free for a 7 B model; larger models need proportionally more space |
| Admin rights | Needed for Homebrew, Docker, and port configuration |
| Optional (UI) | Docker Desktop for Apple Silicon (to run Open WebUI) |
| Optional (IDE) | VS Code + **Continue** or **Open‑Interpreter** for code assistance |

*Sources: Ollama hardware guide, Apple Silicon performance tables*【Source】.

---

## 2️⃣ Install the Core Stack

### 2.1 Install Homebrew (if you don’t have it)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 2.2 Install Ollama

```bash
brew install ollama          # CLI + background service
brew services start ollama   # start Ollama as a macOS service
```

*You can also use the one‑liner `curl https://ollama.ai/install.sh | sh`*【Source】.

### 2.3 Verify the installation

```bash
ollama --version
```

You should see something like `0.12.10` (or newer).

---

## 3️⃣ Pull & Run a Model (CLI‑first workflow)

> **Tip:** Use a quantized version (`q4_0` or `q8_0`) to fit into 16 GB RAM.

```bash
# Example: 7‑B Llama 3.1 quantized to 4‑bit
ollama pull llama3.1:8b-q4_0
```

Start an interactive session:

```bash
ollama run llama3.1:8b-q4_0
```

You’ll get a REPL prompt (`>>>`) where you can type questions.

### 3.1 Adjust runtime parameters (optional)

```bash
# Set max tokens
/set parameter num_predict 512

# Set temperature (creativity)
/set parameter temperature 0.7

# Set context length (tokens)
#set parameter num_ctx 2048
```

*All parameters are documented in the Ollama CLI reference*【Source】.

---

## 4️⃣ Adding a Friendly UI (optional but highly recommended)

### 4.1 Install Docker Desktop (Apple Silicon build)

Download from <https://docs.docker.com/desktop/install/mac-install/> and run the installer.

### 4.2 Run Open WebUI (self‑hosted chat UI)

```bash
docker run -d \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --add-host=host.docker.internal:host-gateway \
  ghcr.io/open-webui/open-webui:main
```

Open `http://localhost:3000` in a browser, create an admin account, then:

1. **Settings → Backend** → select **Ollama**.  
2. **API endpoint**: `http://localhost:11434` (default).  
3. **Test connection** → should succeed.  

Now you can pick any installed Ollama model from the dropdown and chat via a GUI.

*Open WebUI works out‑of‑the‑box with Ollama*【Source】.

---

## 5️⃣ Integrating with Development Tools

### 5.1 VS Code + Continue (local code‑assistant)

1. Install the **Continue** extension from the VS Code Marketplace.  
2. In `continue.json` (or UI settings) add:

```json
{
  "models": [
    {
      "title": "Llama 3.1 8B",
      "model": "llama3.1:8b-q4_0",
      "provider": "ollama",
      "apiBase": "http://localhost:11434"
    }
  ],
  "tabAutocompleteModel": {
    "title": "Qwen2‑Coder 1.5B",
    "model": "qwen2.5-coder:1.5b",
    "provider": "ollama",
    "apiBase": "http://localhost:11434"
  }
}
```

Now you get inline autocomplete and a chat sidebar powered entirely locally.

*See the “AI coding assistant in VS Code” guide*【Source】.

### 5.2 Open‑Interpreter (CLI + file‑system)

```bash
pip install open-interpreter
export OLLAMA_HOST=http://localhost:11434   # optional, defaults to this
interpreter
```

You can now run commands like `ls`, `cat`, or even edit files; the interpreter calls Ollama via its REST API (`/api/generate`).

*Open‑Interpreter works with any Ollama model*【Source】.

---

## 6️⃣ Sharing / Distributing Models (gollama)

-  **gollama** is a small Go utility that lets you **push/pull** Ollama model “images” to a shared registry (e.g., a private Git repo or an internal HTTP server).  
-  Use it **only if you need to distribute a custom‑built model** across multiple machines.  
-  For a single‑machine workflow you can skip it entirely and rely on `ollama pull`.

```bash
# Publish a local model
gollama push my‑fine‑tuned‑model

# Pull on another Mac
gollama pull my‑fine‑tuned‑model
```

*The gollama repo lists it as the official way to share Ollama images*【Source】.

---

## 7️⃣ Scaling to a 128 GB Machine

| RAM | Practical model size (quantized) | Typical use‑case |
|-----|----------------------------------|------------------|
| 16 GB | 7‑13 B (Q4) | General chat, small code assistance |
| 32 GB | 13‑34 B (Q4/Q8) | More nuanced reasoning, longer context |
| 64‑128 GB | 34‑70 B (Q4) or 70‑110 B (Q8) | Complex multi‑step tasks, RAG with large knowledge bases |
| 192 GB (Mac Studio) | 110 B+ (4‑bit) | Research‑grade LLMs, MoE models (Mixtral) |

**What changes?**

1. **Model choice** – you can now pull larger models (`llama3.2:34b-q4_0`, `llama4:70b-q4_0`, etc.).  
2. **No stack change** – the same `ollama`, `docker`, `open-webui`, `continue`, `open-interpreter` tools work unchanged.  
3. **Resource tuning** – increase `num_ctx` (e.g., 8192 tokens) and optionally disable swap (`launchctl limit maxfiles …`) for smoother performance.  
4. **Optional multi‑model serving** – with abundant RAM you can run several models simultaneously (`ollama serve & ollama run … & …`) and route different tasks (code vs. prose) to dedicated models.

*Hardware‑model mapping from the Ollama documentation*【Source】.

---

## 8️⃣ Full End‑to‑End Script (copy‑paste)

```bash
# 1️⃣ Install Homebrew (skip if already installed)
#!/usr/bin/env bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2️⃣ Install Ollama & start as a service
brew install ollama
brew services start ollama

# 3️⃣ Verify
ollama --version

# 4️⃣ Pull a 7B quantized model (fits 16 GB)
ollama pull llama3.1:8b-q4_0

# 5️⃣ Test CLI
ollama run llama3.1:8b-q4_0 <<'EOF'
What is the capital of Spain?
EOF

# 6️⃣ (Optional) Install Docker Desktop manually from Docker website

# 7️⃣ Run Open WebUI UI
docker run -d \
  -p 3000:8080 \
  -v open-webui:/app/backend/data \
  --add-host=host.docker.internal:host-gateway \
  ghcr.io/open-webui/open-webui:main

# 8️⃣ Open http://localhost:3000, configure backend → Ollama → http://localhost:11434

# 9️⃣ (Optional) VS Code Continue config – add the JSON snippet from section 5.1

# 10️⃣ (Optional) Install Open‑Interpreter
pip install open-interpreter
export OLLAMA_HOST=http://localhost:11434
interpreter
```

Run the script line‑by‑line (or paste into a terminal). After step 8 you have a full web chat; after step 9 you have IDE autocomplete; after step 10 you can script file‑system operations.

---

## 9️⃣ Recommendations & Best Practices

| Goal | Recommended Stack |
|------|-------------------|
| **Pure CLI workflow** | Ollama CLI only (`ollama run …`) – fastest, no extra dependencies |
| **Web UI for occasional use** | Ollama + Docker‑run Open WebUI |
| **IDE code assistance** | Ollama + VS Code **Continue** (or **aichat**) |
| **File‑system automation** | Open‑Interpreter (calls Ollama via REST) |
| **Sharing custom models across a team** | Use **gollama** to push/pull model images |
| **Scaling to 128 GB** | Same stack; just pull larger models and increase `num_ctx`/`temperature` as needed |

*All recommendations stem from the collective community guides and official docs*【Source】.

---

### 🎉 You’re ready!

-  **Run `ollama list`** to see installed models.  
-  **Use `ollama rm <model>`** to free space when you need to test a larger model.  
-  **Monitor memory** with Activity Monitor; stay ~2 GB below total RAM to keep macOS responsive.  

Enjoy a **private, offline AI** that runs directly on your M‑series Mac—no cloud, no API keys, full control over data and model versions.  

---  

*All commands are ready to be copied into your terminal or markdown editor.*
