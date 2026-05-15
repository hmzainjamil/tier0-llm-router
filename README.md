# tier0-llm-router
Blast all free/cheap LLMs in parallel — Groq, Gemini, DeepSeek, GPT-4o-mini, Ollama. First response wins.

![Shell](https://img.shields.io/badge/Shell-Bash-4EAA25?style=flat&labelColor=555&logo=gnubash)
![Groq](https://img.shields.io/badge/Groq-200tok/s-F55036?style=flat&labelColor=555)
![Gemini](https://img.shields.io/badge/Gemini-Flash-4285F4?style=flat&labelColor=555)
![DeepSeek](https://img.shields.io/badge/DeepSeek-V3-black?style=flat&labelColor=555)
![Ollama](https://img.shields.io/badge/Ollama-Local-white?style=flat&labelColor=555)
![Status](https://img.shields.io/badge/Status-Active-brightgreen?style=flat&labelColor=555)

[Concepts](#-concepts) · [How It Works](#️-how-it-works) · [Install](#-install) · [Commands](#-commands) · [Tips](#-tips-and-tricks-9) · [Startups](#️-startups--businesses)

---

## 🧠 CONCEPTS

| Feature | Location | Description |
|---------|----------|-------------|
| [**tier0-blast**](tier0-blast) | `tier0-blast` | 188-line parallel blaster — fires all Tier 0 models simultaneously |
| [**tier0-blast-async**](tier0-blast-async) | `tier0-blast-async` | Async variant — non-blocking, returns job ID for later retrieval |
| [**tier0-burst**](tier0-burst) | `tier0-burst` | 426-line burst mode — high-throughput batch processing |
| [**tier0-check**](tier0-check) | `tier0-check` | Health check all Tier 0 endpoints — latency, availability, quota |
| [**tier0-cache-inject**](tier0-cache-inject) | `tier0-cache-inject` | Pre-warm prompt cache across all models before task |
| [**tier0-prompt-inject**](tier0-prompt-inject) | `tier0-prompt-inject` | Inject system context into all models simultaneously |
| [**llm-burst**](llm-burst) | `llm-burst` | LLM burst runner — fires single prompt to all, aggregates results |
| [**llm-burst-run**](llm-burst-run) | `llm-burst-run` | Execution wrapper for llm-burst with retry + fallback logic |

### 🔥 Hot

| Feature | Location | Description |
|---------|----------|-------------|
| [**First-wins Mode**](tier0-blast) | `--mode first` | Returns fastest response — Groq usually wins at 200+ tok/s |
| [**Best-of-N**](tier0-blast) | `--mode best` | Returns longest/most complete response from all models |
| [**Zero Claude Quota**](tier0-burst) | `tier0-burst` | Entire burst uses free/cheap APIs — Claude never touched |

---

## ⚙️ HOW IT WORKS

```
tier0-blast "your prompt" --mode first
         ↓
Fires simultaneously to:
  ├── Groq (llama-3.3-70b)     ~200 tok/s  FREE
  ├── Gemini Flash 2.0          ~150 tok/s  FREE tier
  ├── DeepSeek V3               ~120 tok/s  ~$0.001/1k
  ├── GPT-4o-mini               ~100 tok/s  ~$0.002/1k
  └── Ollama local qwen2.5:7b   ~40 tok/s   FREE
         ↓
First response returned immediately
Other requests cancelled
         ↓
Result cached via tier0-cache-inject for reuse
```

**Modes:** `--mode first` | `--mode all` | `--mode best`

---

## 🚀 INSTALL

```bash
git clone https://github.com/hmzainjamil/tier0-llm-router
cd tier0-llm-router
cp tier0-* llm-burst* ~/.claude/bin/
chmod +x ~/.claude/bin/tier0-* ~/.claude/bin/llm-burst*
```

**Configure** `~/.claude/tier0.env`:
```bash
GROQ_API_KEY=gsk_...
GEMINI_API_KEY=AIza...
DEEPSEEK_API_KEY=sk-...
OPENAI_API_KEY=sk-...
OLLAMA_URL=http://localhost:11434
```

---

## 📟 COMMANDS

| Command | Description |
|---------|-------------|
| `tier0-blast "prompt"` | Fire all models, return all responses |
| `tier0-blast "prompt" --mode first` | Return fastest response only |
| `tier0-blast "prompt" --mode best` | Return most complete response |
| `tier0-blast-async "prompt"` | Non-blocking — returns job ID |
| `tier0-burst "prompt" --n 10` | Batch 10 completions in parallel |
| `tier0-check` | Test all endpoints, show latency table |
| `tier0-cache-inject "system prompt"` | Pre-warm all model caches |
| `llm-burst "prompt"` | Aggregate all responses into one |

---

## 💡 TIPS AND TRICKS (9)

[speed](#tips-speed) · [cost](#tips-cost) · [quality](#tips-quality) · [cache](#tips-cache)

<a id="tips-speed"></a>■ **Speed (3)**

| Tip | Source |
|-----|--------|
| Groq wins `--mode first` 90% of the time — use it as default for latency-sensitive tasks | [HMZ](https://github.com/hmzainjamil) |
| `tier0-blast-async` + later retrieval = zero blocking in pipelines | [DigiMinds](https://github.com/hmzainjamil) |
| Pre-warm with `tier0-cache-inject` at session start — 3x faster on repeated prompts | [HMZ](https://github.com/hmzainjamil) |

<a id="tips-cost"></a>■ **Cost (2)**

| Tip | Source |
|-----|--------|
| Groq + Gemini Flash free tiers handle 90% of tasks — DeepSeek for overflow | [HMZ](https://github.com/hmzainjamil) |
| `tier0-check` shows remaining quota per provider — run before heavy batch jobs | [HMZ](https://github.com/hmzainjamil) |

<a id="tips-quality"></a>■ **Quality (2)**

| Tip | Source |
|-----|--------|
| `--mode best` for important tasks — picks longest response (correlates with completeness) | [DigiMinds](https://github.com/hmzainjamil) |
| `llm-burst` aggregation mode merges unique insights from all models | [HMZ](https://github.com/hmzainjamil) |

<a id="tips-cache"></a>■ **Cache (2)**

| Tip | Source |
|-----|--------|
| Cache TTL varies: Groq 5min, Gemini 1hr, OpenAI 1hr — plan prompt-inject accordingly | [HMZ](https://github.com/hmzainjamil) |
| `tier0-cache-inject` at Claude Code SessionStart hook = pre-warmed every session | [DigiMinds](https://github.com/hmzainjamil) |

---

## ☠️ STARTUPS / BUSINESSES

| This Repo / Feature | Replaced |
|-|-|
| **tier0-blast (multi-LLM parallel)** | [RouteLLM](https://github.com/lm-sys/RouteLLM), [LiteLLM](https://litellm.ai), [OpenRouter](https://openrouter.ai) |
| **tier0-burst (batch processing)** | [Replicate Batch](https://replicate.com), [Together AI Batch](https://together.ai), [Anyscale](https://anyscale.com) |
| **tier0-cache-inject** | [Semantic Cache](https://redis.io), [GPTCache](https://github.com/zilliztech/GPTCache), [Momento](https://momentohq.com) |
| **First-wins routing** | [Portkey](https://portkey.ai), [Martian](https://withmartian.com), [Not Diamond](https://notdiamond.ai) |

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=hmzainjamil/tier0-llm-router&type=Date)](https://star-history.com/#hmzainjamil/tier0-llm-router&Date)

---

---

## 🏗 ARCHITECTURE

```
~/.claude/
├── bin/                    ← All executable scripts
├── skills/                 ← SKILL.md files for Claude
├── agents/                 ← Agent definition files
├── tcc-logs/               ← Task execution logs
│   └── YYYY-MM-DD/         ← Daily log directories
└── tier0.env               ← API keys for all Tier 0 models
```

**Dependencies:** Python 3.11+ · Bash · GitHub CLI (`gh`) · Ollama (local models)

---

## ❓ FAQ

**Q: Do I need all API keys?**
A: No. Each Tier 0 model is optional. Ollama (free local) works standalone.

**Q: Will this work on Linux/Windows?**
A: Bash scripts → Linux ✓. Windows needs WSL2. All Python scripts cross-platform.

**Q: How much does it cost to run?**
A: Groq + Gemini free tiers cover 90% of tasks. DeepSeek/GPT-4o-mini ~$1-5/month heavy use.

**Q: Can I add my own models?**
A: Yes — add to `tier0.env` + update model list in `tier0-blast`.

---

## 📋 CHANGELOG

| Version | Date | Changes |
|---|---|---|
| v1.2 | 2026-05-15 | Added ollama watchdog, hermes integration, daily sync |
| v1.1 | 2026-05-12 | MAE engine, TCC queue, Tier 0 router |
| v1.0 | 2026-05-10 | Initial release — core scripts + skills |

---

## 🔗 RELATED REPOS

| Repo | Relation |
|---|---|
| [mae-master-automation-engine](https://github.com/hmzainjamil/mae-master-automation-engine) | Orchestrates this system |
| [tcc-task-command-center](https://github.com/hmzainjamil/tcc-task-command-center) | Task queue for parallel execution |
| [tier0-llm-router](https://github.com/hmzainjamil/tier0-llm-router) | LLM routing layer |
| [hermes-ai-system](https://github.com/hmzainjamil/hermes-ai-system) | Local model orchestration |
| [claude-ai-system](https://github.com/hmzainjamil/claude-ai-system) | Master backup repo |


<div align="center">
Built by <a href="https://github.com/hmzainjamil">HMZ</a> · Part of the <a href="https://github.com/hmzainjamil/claude-ai-system">HMZ Claude AI System</a> · Zero Claude quota
</div>
