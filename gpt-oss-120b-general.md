# Local LLM Deployment on an M1 MacBook Pro (16 GB) – From Scratch  

---  

## 1️⃣ Prerequisites  

| Item | Minimum | Recommended |
|------|---------|------------|
| macOS | 11 Big Sur | 12 Monterey or later |
| RAM | 8 GB (tiny models) | **16 GB** for 7‑13 B models, **32 GB+** for >13 B |
| Disk | 10 GB free | 30 GB+ (models are several GB each) |
| CPU | Apple Silicon (M1/M2) or recent Intel | Apple Silicon (M1/M2/M3) for best speed |
| Network | ‑ | Broadband for initial model pulls |  

*Ollama requires at least 8 GB RAM for 7 B‑parameter models and 16 GB for 13 B‑parameter models; larger models need proportionally more memory*【Source】(https://github.com/ollama/ollama).  

---  

## 2️⃣ Install Ollama  

You can install via the official GUI installer or Homebrew – both work on Apple Silicon.  

### 2.1 GUI (quickest)  

1. Open <https://ollama.com> and click **Download for macOS**.  
2. Drag `Ollama.app` into **Applications**.  
3. Launch it; the icon appears in the menu bar, indicating the server is running.  

### 2.2 Homebrew (CLI‑friendly)  

```bash
# Install Homebrew if you don’t have it
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Ollama (cask) and start it as a background service
brew install --cask ollama
brew services start ollama
```

Verify the installation:

```bash
ollama --version          # should print the version number
curl http://localhost:11434   # returns “Ollama is running”
```  

*Both methods are covered in the official docs and in community guides*【Source】(https://johnwlittle.com/ollama-on-mac-silicon-local-ai-for-m-series-macs/)【Source】(https://www.metriccoders.com/post/how-to-install-and-run-ollama-on-macos).  

---  

## 3️⃣ Pull & Run Your First Model  

### 3.1 Choose a model that fits 16 GB RAM  

| Model | Parameters | Approx. size (quantized) | Typical use |
|------|------------|--------------------------|-------------|
| **Mistral‑7B** | 7 B | ~4 GB (Q4) | General‑purpose chat & coding |
| **Llama 3.1‑8B** | 8 B | ~4.7 GB | Good balance of quality & speed |
| **CodeLlama‑7B** | 7 B | ~3.8 GB | Code generation / explanation |
| **Gemma‑3‑4B** | 4 B | ~3.3 GB | Faster response, lower RAM |  

*The table in John W. Little’s article shows that an M1/M2 MacBook with 16 GB can comfortably run 7‑13 B models when quantized*【Source】(https://johnwlittle.com/ollama-on-mac-silicon-local-ai-for-m-series-macs).  

### 3.2 Pull the model  

```bash
ollama pull mistral          # pulls the default (7B) quantized version
# or, for a specific variant:
ollama pull llama3.1:8b
```

### 3.3 Interactive CLI chat  

```bash
ollama run mistral
# type your prompt, end with /bye to quit
```  

---  

## 4️⃣ Optional UI Layers  

| UI | Installation method | Pros | Cons |
|----|--------------------|------|------|
| **Open‑WebUI** (web‑based ChatGPT‑like) | Docker (`docker run -d -p 3000:8080 -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 ghcr.io/open-webui/open-webui:main`) | Rich chat history, prompt library, easy multi‑model switching | Requires Docker, extra container overhead |
| **LM Studio** (desktop GUI) | Download from <https://lmstudio.ai> (macOS ARM) | Native macOS app, model manager, built‑in quantization | No built‑in web UI, limited to single model at a time |
| **Continue (VS Code extension)** | Install “Continue” from VS Code Marketplace; configure Ollama endpoint | Inline code autocomplete, chat sidebar, works inside IDE | Requires VS Code, less full‑screen chat experience |

*Open‑WebUI Docker setup is described in the “WiseCat” video and Saltyoldgeek blog*【Source】(https://www.youtube.com/watch?v=d2Ib-fWYikc)【Source】(https://www.saltyoldgeek.com/posts/ollama-llama3-openwebui/).  

---  

## 5️⃣ CLI‑Driven Tooling (coding & scripting)  

| Tool | What it does | Integration steps |
|------|--------------|-------------------|
| **Open‑Interpreter** | Executes Python/JS code via an LLM, returns results in terminal | `pip install open-interpreter` → set `OPENAI_API_BASE=http://localhost:11434` and `OPENAI_MODEL=mistral` (or any Ollama model) |
| **Open‑Code** | Provides a REPL that can edit files, run shell commands, and browse the filesystem | `pip install open-code` → same env vars as above |
| **Gollama** (experimental) | Packages an Ollama model as a Docker image for easy sharing | `ollama create mymodel -f Modelfile` → `docker build -t mymodel .` (see Ollama docs) |

*Open‑Interpreter and Open‑Code rely on the OpenAI‑compatible REST API that Ollama exposes*【Source】(https://github.com/ollama/ollama).  

---  

## 6️⃣ Managing Models & Quantization  

1. **List installed models** – `ollama list`  
2. **Remove a model** – `ollama rm <model>` (frees disk space)  
3. **Change runtime parameters** (temperature, context length, etc.) inside a session:  

```bash
/set parameter temperature 0.7
/set parameter num_ctx 2048
```  

*Parameter tweaking is documented in the Ollama CLI reference*【Source】(https://github.com/ollama/ollama).  

---  

## 7️⃣ Performance Tips for the M1 (16 GB)  

| Tip | Why it helps |
|-----|--------------|
| **Use Q4/K quantized models** (default for most Ollama pulls) | Reduces RAM & VRAM usage, speeds inference |
| **Close other apps** – leave ~4‑6 GB free for the OS | Prevents swapping, keeps the Neural Engine responsive |
| **Store models on the internal SSD** (not external HDD) | Faster model loading |
| **Set `OLLAMA_NUM_THREADS` to the number of CPU cores (8 for M1)** – `launchctl setenv OLLAMA_NUM_THREADS 8` | Improves parallel token generation |
| **Prefer the built‑in Ollama server (no Docker) for the main model** | Avoids extra container overhead that competes for memory |  

*Performance benchmarks in the John W. Little article show that the M1’s Neural Engine accelerates quantized models, but memory pressure still caps model size*【Source】(https://johnwlittle.com/ollama-on-mac-silicon-local-ai-for-m-series-macs).  

---  

## 8️⃣ Scaling to a 128 GB Machine  

| Change | Effect |
|--------|--------|
| **RAM ↑ → run 30‑70 B models** (e.g., Llama 3.3‑70B, Qwen‑2‑72B) | Larger context windows, higher quality responses |
| **Run multiple models concurrently** (e.g., one for chat, one for code) | Enables multi‑agent pipelines (e.g., Open‑Interpreter + CodeLlama) |
| **Use less‑aggressive quantization (Q8 or FP16)** – better quality, still fits in 128 GB | Improves generation fidelity for demanding tasks |
| **Add a local vector store (e.g., Chroma) for RAG** – plenty of RAM for large document embeddings | Enables retrieval‑augmented generation for long‑form writing or knowledge‑base queries |
| **Docker‑compose stack** (Ollama + Open‑WebUI + PostgreSQL) becomes practical – you can allocate dedicated memory to each container | Gives a production‑like environment on a single workstation |

*The “Mac Studio” section of the John W. Little guide illustrates that 64‑128 GB Apple Silicon can handle 70 B‑plus models and multiple 13 B models simultaneously*【Source】(https://johnwlittle.com/ollama-on-mac-silicon-local-ai-for-m-series-macs).  

---  

## 9️⃣ Full End‑to‑End Example (M1 16 GB)  

```bash
# 1️⃣ Install Homebrew (if missing)
 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2️⃣ Install Ollama and start it as a service
brew install --cask ollama
brew services start ollama

# 3️⃣ Pull a 7B model (Mistral) – fits comfortably in 16 GB
ollama pull mistral

# 4️⃣ (Optional) Install Open‑WebUI for a web chat UI
docker run -d \
  --network=host \
  -v open-webui:/app/backend/data \
  -e OLLAMA_BASE_URL=http://127.0.0.1:11434 \
  --name open-webui \
  --restart always \
  ghcr.io/open-webui/open-webui:main

# 5️⃣ Verify UI
open http://localhost:8080   # creates an account, then select “mistral”

# 6️⃣ Use Open‑Interpreter for code assistance in terminal
pip install open-interpreter
export OPENAI_API_BASE=http://localhost:11434/v1
export OPENAI_MODEL=mistral
interpreter               # now you can ask it to write Python scripts, manage files, etc.

# 7️⃣ Fine‑tune a small adapter (optional)
#   - Convert LoRA adapter to GGML (see Ollama docs)
#   - Create a custom model:
#     echo -e "FROM mistral\nADAPTER ./my_adapter.gguf" > Modelfile
#     ollama create my-mistral -f Modelfile
#   - Run: ollama run my-mistral
```

---  

## 🔧 Summary of Recommended Stack  

| Layer | Recommended Tool | Reason |
|------|------------------|--------|
| **Model runtime** | **Ollama** (CLI) | Native Apple‑Silicon support, simple API, low‑overhead |
| **Chat UI** | **Open‑WebUI** (Docker) *or* **LM Studio** (native) | Choose based on whether you prefer a web interface or a desktop app |
| **IDE integration** | **Continue** (VS Code) or **Open‑Interpreter** | Gives inline code suggestions & file‑management commands |
| **Model sharing / custom images** | **Gollama** (create a Docker image from a Modelfile) | Useful if you need to ship a tuned model to teammates |
| **Scaling to 128 GB** | Keep Ollama server, add **RAG vector store**, run **multiple models** concurrently, use **higher‑precision quantization** | Leverages extra RAM for larger or multiple models |

---  

### TL;DR  

1. Install Ollama (GUI or Homebrew).  
2. Pull a 7‑8 B quantized model (Mistral, Llama 3.1, CodeLlama).  
3. Run it via `ollama run …` or expose it to a UI (Open‑WebUI Docker) or IDE (Continue).  
4. For coding/scripting, use Open‑Interpreter or Open‑Code with the same endpoint.  
5. On a 128 GB machine, switch to 30‑70 B models, run several models at once, and add a local vector store for RAG.  

All steps are fully offline, keep your data private, and run efficiently on Apple Silicon.  

---  

*References*  
-  Ollama official docs & GitHub (model list, CLI, API) – <https://github.com/ollama/ollama>【Source】  
-  John W. Little, “Ollama on Mac Silicon” – performance tables & hardware guide【Source】  
-  MetricCoders, “How to Install and Run Ollama on macOS”【Source】  
-  Byteplus guide, “Ollama download mac – 2025” (performance & UI options)【Source】  
-  dev.to article, “Run LLMs locally with Ollama on macOS for Developers” (VS Code integration)【Source】  
-  Saltyoldgeek blog, “Ollama + Open‑WebUI on M1/M2” (Docker UI setup)【Source】  
