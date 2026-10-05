Architectural & Engineering Decisions (DECISIONS.md)

This log records autonomous decisions made for **Dadi ki Rasoi**, adhering to the rule: *Never ask design questions; decide yourself and note the decision in DECISIONS.md.*

---

## 1. Storage & Drive Optimization
- **Detected Storage**:
  - Drive C: 129.56 GB (Used: 129.51 GB, Free: ~0.05 GB) — Severely constrained.
  - Drive D: 107.42 GB (Free: **93.89 GB**) — Ample high-speed capacity.
- **Decision**:
  - Direct all model weights (`OLLAMA_MODELS=D:\ollama_models`) to Drive D:.
  - Place external tooling and caching in `D:\tools` (`ffmpeg`, `ollama`).
  - This completely shields the host OS from disk-exhaustion crashes while pulling LLM weights and audio models.

---

## 2. Hardware & Model Selection (Phase 0)
- **Detected Hardware**:
  - Total Physical RAM: **3.70 GB** (Available: ~0.20 GB)
  - OS: Windows 11
- **Rule**: If RAM < 12GB, fallback to 3B models.
- **Decision**:
  - Primary LLM: `qwen2.5:3b` (and lightweight fallback `qwen2.5:1.5b` if physical memory pressure requires it).
  - Secondary comparison LLM: `llama3.2:3b` (or `llama3.2:1b` for constrained RAM).
  - `faster-whisper`: Use `base` or `small` model with CPU `int8` quantization for optimal latency and memory footprint on 4GB RAM machines.

---

## 3. System Software & Package Management
- **Detected Tools**:
  - `node` (v24.20.0): Installed
  - `npm` (v11.19.0): Installed
  - `python` (v3.12.6): Installed
  - `pip` (v24.2): Installed
  - `git` (v2.45.1): Installed
  - `gh` (v2.102.0): Installed in project `bin/` (and linked)
  - `ffmpeg`: Configured in `D:\tools\ffmpeg`
  - `ollama`: Configured with `OLLAMA_MODELS=D:\ollama_models`

---

## 4. Architecture & Service Decomposition
- **Repository Structure**:
  - `/web`: React 18 + Vite + Vanilla CSS design system (Cream & Saffron palette `#FFFDF7`, `#FF9933`, `#800020`, Noto Sans Devanagari typography).
  - `/api`: Node.js + Express + Zod schema validation + Desi measurement conversion engine with automatic 2x retry on JSON parsing failures.
  - `/stt`: Python 3.12 + FastAPI + `faster-whisper` (`int8` compute) + audio pre-processing with ffmpeg.
  - `/samples`: 3 authentic Hinglish audio voice notes + ground truth transcripts.
  - `/docs`: Architecture, test results, and benchmark comparisons (`results.md`).
  - `/scripts`: One-command cross-platform runner (`run.bat` / `run.sh`).

---

## 5. Desi Measurement Normalization Engine
- Indian grandmothers rarely use metric measurements like grams or milliliters. Instead, they use traditional vernacular units:
  - *ek mutthi* (~40-50g for rice/dal, ~15-20g for leafy greens/herbs)
  - *ek chutki* (~0.5 - 1g)
  - *ek katori* (~150ml liquid / ~120g solid)
  - *thoda sa* / *swaad anusaar* (~2-5g / to taste)
  - *aankh se andaza* (visual approximation, flagged with `confidence: "low"`)
- **Decision**:
  - Maintain a centralized `desi_measures.json` lookup table.
  - LLM prompt explicitly binds to this dictionary for consistent conversions.
  - Crucially: **ALWAYS preserve the raw `original_phrase`** in italics beside the metric estimate so grandmother's voice and authenticity are never erased.

---

## 6. Offline & Privacy Assurance
- An active `Offline Mode Verified` indicator in the frontend continuously checks `navigator.onLine` and ensures all fetch endpoints are strictly bound to `localhost` (`/api`, `/stt`, `ollama`), ensuring zero data leaks outside the home machine.