---
name: lm-studio
description: Query a local LLM running in LM Studio for cheaper, faster responses on simple tasks — summarization, translation, rewrites, Q&A, brainstorming. Use instead of your own reasoning when the task is straightforward and latency/cost matters. Do NOT use for tasks requiring code execution, tool use, real-time data, or complex multi-step reasoning.
allowed-tools: Bash(ask:*)
---

# Local LLM via LM Studio

A local model is available at `host.docker.internal:1234` (LM Studio). Use it for tasks where a smaller, faster model is good enough.

## When to use

**Good fit (use local model):**
- Summarize text or documents
- Translate between languages
- Rewrite / rephrase content
- Answer factual questions from provided context
- Brainstorm ideas or lists
- Draft short messages or replies
- Classify or categorize text

**Not a good fit (use your own reasoning):**
- Writing or debugging code
- Tasks requiring tool calls (web search, file ops)
- Complex multi-step reasoning
- Real-time or up-to-date information
- Tasks where accuracy is critical

## How to call

The `ask` script is at `/home/node/.claude/skills/lm-studio/ask`.

```bash
# Basic usage
/home/node/.claude/skills/lm-studio/ask "Summarize this in 3 bullet points: ..."

# With stdin (for long text)
echo "long text here..." | /home/node/.claude/skills/lm-studio/ask

# Custom system prompt
/home/node/.claude/skills/lm-studio/ask --system "You are a professional translator." "Translate to Italian: Hello world"

# Explicit model (if multiple loaded)
/home/node/.claude/skills/lm-studio/ask --model "lmstudio-community/Meta-Llama-3.1-8B-Instruct-GGUF" "Your prompt"
```

## Check available models

```bash
curl -s http://host.docker.internal:1234/v1/models | python3 -c "import json,sys; [print(m['id']) for m in json.load(sys.stdin)['data']]"
```

## Error handling

If LM Studio is not running or no model is loaded, `ask` exits with a non-zero code and an error message on stderr. In that case, fall back to your own reasoning — never fail the user's request just because the local model is unavailable.

```bash
# Safe invocation with fallback
RESULT=$(/home/node/.claude/skills/lm-studio/ask "..." 2>/dev/null) || RESULT=""
if [[ -z "$RESULT" ]]; then
  # LM Studio unavailable — answer directly
  echo "LM Studio unavailable, answering directly..."
fi
```

## Environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `LM_STUDIO_HOST` | `host.docker.internal` | Host where LM Studio runs |
| `LM_STUDIO_PORT` | `1234` | LM Studio server port |
