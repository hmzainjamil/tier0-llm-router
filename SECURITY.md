# Security and data handling

## Scope

This note describes risks visible in the tracked scripts. It is not a security certification or a complete threat model.

## Prompt data

Treat prompts and model responses as potentially sensitive. `llm-burst`, `tier0-blast`, and `tier0-blast-async` can send prompts to configured third-party endpoints. Review provider selection and environment configuration before use. Local Ollama calls stay on the configured local endpoint; this does not establish the security of the local machine or service.

The async script stores responses under `/tmp/tier0-cache`. The cache injector may print response text into a prompt context. This repository sets no retention, access-control, or deletion policy for those files.

## Credentials

The public `llm-burst` source contained a hard-coded credential fallback. This branch removes that fallback and requires `BYTEZ_API_KEY` from the environment for that provider to be used. A credential committed to public history must be treated as exposed: the maintainer should revoke or rotate the associated credential with its provider. Rewriting this branch does not remove the value from earlier commits or forks.

Keep provider keys out of source control and shell history. The scripts read keys from environment variables and, in some cases, `~/.claude/tier0.env`. Protect that file with appropriate local permissions.

## Local side effects

- `tier0-check` may start `ollama serve` and writes `~/.claude/tier0-status.json`.
- `tier0-blast-async` writes response files and a timestamp under `/tmp/tier0-cache`.
- `tier0-prompt-inject` starts background model calls and emits text intended for a prompt hook.
- Wrappers source user shell configuration or invoke paths under `~/.claude/bin`.

Review scripts before installing them into hooks or running them with sensitive prompts. The repository has no test suite or security test evidence.
