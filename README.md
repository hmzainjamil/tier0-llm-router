# tier0-llm-router

Local-first LLM router: Ollama → DeepSeek → Gemini → Groq → GPT-4o-mini → Claude last resort. Zero cost for 90% of queries.

![Router](https://img.shields.io/badge/Router-Tier_0_First-blue?style=flat&labelColor=555) ![Ollama](https://img.shields.io/badge/Ollama-Local-green?style=flat&labelColor=555) ![Free](https://img.shields.io/badge/Cost-Near_Zero-brightgreen?style=flat&labelColor=555) ![License](https://img.shields.io/badge/License-MIT-yellow?style=flat&labelColor=555)

[Concepts](#-concepts) · [How It Works](#-how-it-works) · [Install](#-install) · [Usage](#-usage) · [Config](#-configuration) · [Tips](#-tips-and-tricks-12) · [Troubleshooting](#-troubleshooting) · [Architecture](#-architecture) · [Startups](#️-startups--businesses)

---

## 🧠 CONCEPTS

| Feature | Location | Description |
|---|---|---|
| Tier System | `config/tiers.yaml` | Tier 0 (free/local) → Tier 1 (cheap) → Tier 2 (Claude) |
| Provider Registry | `providers/registry.yaml` | All providers with base URLs, models, costs, rate limits |
| Smart Routing | `router/smart.py` | Routes by task type: code→deepseek, long→gemini, chat→llama |
| Fallback Chain | `router/fallback.py` | Tier 0 fails → try next Tier 0 → escalate to Tier 1 |
| Health Checker | `health/checker.py` | Continuous provider ping — removes degraded endpoints |
| Cost Calculator | `cost/calc.py` | Real-time cost estimate before routing decision |
| Request Logger | `logging/logger.py` | Every request logged: provider, model, tokens, latency, cost |
| Rate Limiter | `rate/limiter.py` | Per-provider rate limiting to prevent 429s |
| Model Selector | `router/model_select.py` | Task-aware model selection within chosen provider |
| Caching Layer | `cache/semantic.py` | Semantic dedup — same-meaning queries skip re-inference |
| Budget Enforcer | `budget/enforcer.py` | Hard stops when daily/monthly budget exceeded |
| Analytics | `analytics/dashboard.py` | Provider usage stats, cost breakdown, cache hit rate |

### 🔥 Hot

| Feature | Location | Description |
|---|---|---|
| Tier 0 First | `config/tiers.yaml` | 90% of queries handled free by local/free providers |
| Smart Routing | `router/smart.py` | deepseek-coder for code, gemini for long context, llama for chat |
| Fallback Chain | `router/fallback.py` | Never fails — always escalates until response found |
| Semantic Cache | `cache/semantic.py` | Identical-intent queries served from cache — 0 tokens |
| Claude Last Resort | `config/tiers.yaml` | Claude Sonnet only when user explicitly requests it |

---

## ⚙️ HOW IT WORKS

```
Request + Task Type
    │
    ▼
Smart Router
    ├── task_type = "code"     → deepseek-coder (Tier 0)
    ├── task_type = "long"     → gemini-flash (Tier 0, 1M ctx)
    ├── task_type = "chat"     → llama3.1:8b (Tier 0, local)
    ├── task_type = "reason"   → deepseek-r1:7b (Tier 0)
    └── task_type = "default"  → Try Tier 0 in order

Tier 0 Attempt:
    Ollama → (fail?) → DeepSeek → (fail?) → Gemini → (fail?) → Groq

    │ (all Tier 0 fail)
    ▼
Tier 1 Attempt:
    GPT-4o-mini → (fail?) → Kimi

    │ (Tier 1 fails OR user requests Claude)
    ▼
Tier 2:
    Claude Haiku → Claude Sonnet

Result:
    ├── Cache write (if cacheable)
    ├── Log (provider, model, tokens, latency, cost)
    └── Return response
```

---

## 🚀 INSTALL

```bash
git clone https://github.com/hmzainjamil/tier0-llm-router
cd tier0-llm-router

pip install -r requirements.txt

# Pull Ollama models
ollama pull llama3.1:8b
ollama pull deepseek-r1:7b
ollama pull deepseek-coder:6.7b
ollama pull nomic-embed-text

cp .env.example .env
# Fill: GROQ_API_KEY, GEMINI_API_KEY, DEEPSEEK_API_KEY, OPENAI_API_KEY, ANTHROPIC_API_KEY

# Test routing
python3 router/smart.py --test

# Check all provider health
python3 health/checker.py --check-all

# Start router API server
python3 server.py
```

---

## 📟 USAGE

```bash
# Route a query (auto-selects best Tier 0)
python3 router/smart.py --prompt "Write a Python function to sort a dict by value"

# Force specific tier
python3 router/smart.py --prompt "..." --tier 0
python3 router/smart.py --prompt "..." --tier 2  # Claude only

# Route by task type
python3 router/smart.py --prompt "..." --task-type code
python3 router/smart.py --prompt "..." --task-type reason
python3 router/smart.py --prompt "..." --task-type long

# Check cost estimate before routing
python3 cost/calc.py --prompt "Long analysis prompt..." --tier 0

# View analytics
python3 analytics/dashboard.py

# Via REST API (when server running)
curl -X POST http://localhost:8000/route \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is RAG?", "task_type": "chat"}'

# Python SDK
from tier0_router import Router
r = Router()
response = r.route("Explain transformers", task_type="chat")
```

---

## ⚙️ CONFIGURATION

| Variable | Default | Description |
|---|---|---|
| `TIER0_PROVIDERS` | `ollama,deepseek,groq,gemini` | Tier 0 provider order |
| `TIER1_PROVIDERS` | `openai,kimi` | Tier 1 provider order |
| `TIER2_PROVIDERS` | `anthropic` | Tier 2 (last resort) |
| `OLLAMA_BASE_URL` | `http://localhost:11434` | Local Ollama endpoint |
| `CACHE_ENABLED` | `true` | Enable semantic request caching |
| `CACHE_SIMILARITY_THRESHOLD` | `0.95` | Cosine similarity for cache hit |
| `HEALTH_CHECK_INTERVAL_S` | `60` | Provider health ping interval |
| `BUDGET_DAILY_USD` | `2.00` | Daily spend hard cap |
| `LOG_LEVEL` | `INFO` | Logging verbosity |
| `SERVER_PORT` | `8000` | REST API server port |

---

## 💡 TIPS AND TRICKS (12)

[Routing](#tips-routing) · [Models](#tips-models) · [Cache](#tips-cache) · [Budget](#tips-budget)

<a id="tips-routing"></a>■ **Smart Routing (3)**

| Tip | Source |
|---|---|
| Tag your prompts with task_type — router makes 3× better model choices | Smart router docs |
| `task_type=long` → Gemini Flash with 1M context window, free, handles books | Model guide |
| `task_type=code` → deepseek-coder beats GPT-4 on most coding benchmarks, free | HumanEval benchmarks |

<a id="tips-models"></a>■ **Model Selection (3)**

| Tip | Source |
|---|---|
| deepseek-r1:7b = best free local reasoning model as of 2025 | Model benchmarks |
| llama3.1:8b = best free local chat model — fast, capable, 128K context | Ollama docs |
| Groq llama-3.1-70b = free cloud option when local too slow | Groq docs |

<a id="tips-cache"></a>■ **Semantic Cache (3)**

| Tip | Source |
|---|---|
| `CACHE_SIMILARITY_THRESHOLD=0.90` — slightly looser threshold catches more near-duplicates | Cache tuning |
| Cache uses nomic-embed-text locally — zero cost for embedding | Embedding config |
| `python3 cache/stats.py` — check hit rate; target >30% for repetitive workloads | Cache analytics |

<a id="tips-budget"></a>■ **Budget Control (3)**

| Tip | Source |
|---|---|
| With Tier 0 routing, `BUDGET_DAILY_USD=2.00` is rarely hit — mostly Tier 0 handles it | Budget guide |
| `python3 analytics/dashboard.py` shows cost breakdown by provider — identify leaks | Analytics |
| Force Tier 0 for internal tasks: `--tier 0` — Claude budget preserved for user-facing output | Tier guide |

---

## 🔧 TROUBLESHOOTING

| Issue | Fix |
|---|---|
| Ollama not available | `brew services restart ollama` |
| All Tier 0 failing | `python3 health/checker.py --check-all` |
| Cache not hitting | Lower `CACHE_SIMILARITY_THRESHOLD` to 0.90 |
| Routing to wrong model | Check `config/tiers.yaml` provider order |
| Budget exceeded | `python3 budget/reset.py --daily` |
| Server not starting | Check port conflict: `lsof -i :8000` |
| nomic-embed-text missing | `ollama pull nomic-embed-text` |

---

## 📊 ARCHITECTURE

```
tier0-llm-router/
├── router/
│   ├── smart.py                # Task-aware routing
│   ├── fallback.py             # Tier escalation chain
│   └── model_select.py         # Model selection within tier
├── providers/
│   └── registry.yaml           # Provider definitions
├── config/
│   ├── tiers.yaml              # Tier assignments
│   └── task_types.yaml         # Task→tier mappings
├── health/
│   └── checker.py              # Provider health monitoring
├── cache/
│   └── semantic.py             # Semantic dedup cache
├── cost/
│   └── calc.py                 # Real-time cost estimation
├── budget/
│   └── enforcer.py             # Hard budget limits
├── rate/
│   └── limiter.py              # Per-provider rate limiting
├── logging/
│   └── logger.py               # Request/response logging
├── analytics/
│   └── dashboard.py            # Usage analytics
├── server.py                   # REST API server
├── requirements.txt
└── .env.example
```

---

## 📋 PROVIDER TIER TABLE

| Provider | Tier | Cost | Context | Best For |
|---|---|---|---|---|
| Ollama (local) | 0 | $0 | 128K | All tasks offline |
| DeepSeek API | 0 | $0.001/1M | 64K | Code, reasoning |
| Groq | 0 | Free (rate limited) | 128K | Fast inference |
| Gemini Flash | 0 | Free (15 RPM) | 1M | Long context |
| Mistral | 0 | Free tier | 32K | European data |
| GPT-4o-mini | 1 | $0.15/1M | 128K | General fallback |
| Kimi/Moonshot | 1 | $0.12/1M | 262K | Long context paid |
| Claude Haiku | 2 | $0.25/1M | 200K | Quality fallback |
| Claude Sonnet | 2 | $3/1M | 200K | Final output only |

---

## ☠️ STARTUPS / BUSINESSES

| This Repo / Feature | Replaced |
|---|---|
| Tier 0 routing | Paying Claude for every sub-task |
| Smart routing | Using one model for everything suboptimally |
| Fallback chain | App errors when single provider goes down |
| Semantic cache | Re-inferencing identical queries |
| Budget enforcer | Surprise monthly bill from runaway usage |
| Health checker | Manual provider status page checking |
| Analytics dashboard | No visibility into model usage patterns |
| Rate limiter | 429 errors from accidental burst usage |

---

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=hmzainjamil/tier0-llm-router&type=Date)](https://star-history.com/#hmzainjamil/tier0-llm-router&Date)

---
<div align="center">Built by <a href="https://github.com/hmzainjamil">HMZ</a> · Part of HMZ Claude AI System</div>

---

## 🔄 CONTRIBUTING

PRs welcome. Please include:
- Tests for new functionality
- Updated `config/providers.yaml` if adding providers
- Benchmark comparison for performance claims
- Documentation update in README

```bash
git checkout -b feature/my-feature
# make changes
python3 tests/run_all.py  # must pass
git push origin feature/my-feature
# open PR
```

---

## 📌 RELATED REPOS

| Repo | Purpose |
|---|---|
| [G0DM0D3](https://github.com/hmzainjamil/G0DM0D3) | Multi-model race + Liquid Response |
| [hermes-ai-system](https://github.com/hmzainjamil/hermes-ai-system) | Local agent with 30+ tools |
| [claude-ai-system-backup](https://github.com/hmzainjamil/claude-ai-system-backup) | Full system backup |
| [hmz-ai](https://github.com/hmzainjamil/hmz-ai) | Personal automation hub |
