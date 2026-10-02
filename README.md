# tier0-llm-router

Standalone scripts for parallel model calls, response comparison, and prompt-cache injection. The repository does not contain the Python package, CLI, setup script, hooks, or configuration template described in its previous README.

| Status | Evidence |
|---|---|
| Source reviewed | 2026-10-02; eight tracked scripts on `main` |
| Tests and evaluation | No test suite or reproducible evaluation is included |
| Packaging | No dependency manifest or installer is included |
| License | No license file or declared license in repository metadata |

## What is here

| Script | Source-backed behavior |
|---|---|
| `llm-burst` | Python async fan-out to configured providers plus local Ollama; ranks responses with a heuristic and prints a selected response. Optional provider selection uses `--models`. |
| `tier0-burst` | Separate Python model-comparison CLI with fast, full, cloud, and local modes; ranks outputs with its own heuristic. |
| `tier0-blast` | Bash fan-out to configured providers and a machine-specific GPT4All path. Waits for jobs and prints collected responses. Its usage comment mentions first/all/best, but implementation shown collects all results. |
| `tier0-blast-async` | Background calls to local Ollama and configured cloud endpoints; writes timestamped results under `/tmp/tier0-cache`. |
| `tier0-cache-inject` | Prints recent cached response text for use as prompt context. |
| `tier0-check` | Checks selected local models and configured key presence; can start `ollama serve` and writes `~/.claude/tier0-status.json`. |
| `tier0-prompt-inject` | Starts the async blast in the background and emits provider-status/context text for a Claude Code prompt hook. |
| `llm-burst-run` | Wrapper for a different installed path, `~/.claude/bin/llm-burst`; it does not invoke this checkout directly. |

## Requirements and setup

These scripts are personal-machine integrations, not a self-contained install. They expect Bash, Python 3, and in places `curl`; Python scripts import `aiohttp`, and optional local inference paths expect Ollama or GPT4All files. No pinned dependency list, supported platform matrix, or setup command is provided.

The scripts load keys from environment variables and/or `~/.claude/tier0.env`. Some wrappers also source `~/.zshrc`. Review a script and its paths before running it. Do not run `tier0-check` under the assumption that it is read-only.

For the checkout-local Python entry point, after installing `aiohttp` by a method appropriate for your environment:

```sh
python3 ./llm-burst --help
python3 ./llm-burst --models ollama "your prompt"
```

The second command selects the local Ollama caller in `llm-burst`; it still requires a running local Ollama service and the script's expected model. This has not been runtime-tested here. Other scripts are coupled to paths under `~/.claude/bin`; inspect and adapt those paths before use.

## Provider and data flow

`llm-burst` can send the full prompt to every selected, configured cloud provider concurrently. It also calls local Ollama. Its default model selection may include cloud callers when credentials are configured; it is not an enforced local-first fallback ladder. `tier0-blast` and `tier0-blast-async` also fan prompts out to configured cloud endpoints. Provider availability, model names, account access, pricing, and retention are external and are not verified by this repository.

The response score is a source-code heuristic based on length, prompt-word overlap, simple formatting/actionability patterns, and refusal/error text. It is not a quality, truth, safety, or task-success evaluation. A higher score does not establish a better answer.

## Limits and known operational details

- Several scripts assume macOS-specific GPT4All paths or Claude Code directories.
- Cache files and prompt-hook inputs may contain submitted task text. Async results are written to `/tmp/tier0-cache`; this project does not define a retention or deletion policy.
- `tier0-check` may launch an Ollama server in the background and writes status under the user's home directory.
- `tier0-blast` says it supports modes in comments, but source currently waits for all launched processes and prints all collected results.
- `llm-burst-run` and hook scripts require matching files under `~/.claude/bin`; this repository alone does not install them.
- No release history, test instructions, changelog, or support policy is supplied.

## Documentation

- [Security and data handling](SECURITY.md)

## Provenance and license

Repository metadata identifies `hmzainjamil/tier0-llm-router` as public, with default branch `main`. No license is declared in the repository metadata or tracked tree reviewed for this change. Obtain maintainer guidance before redistributing or reusing the scripts.

## Maintainer

[hmzainjamil](https://github.com/hmzainjamil)
