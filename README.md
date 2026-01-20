<div align="center">

# GET SHIT DONE (Multi-Model Fork)

**A cost-optimized fork of GSD with multi-model orchestration.**

**Opus plans. GLM implements. Codex reviews.**

[![Original](https://img.shields.io/badge/original-glittercowboy%2Fget--shit--done-blue?style=for-the-badge)](https://github.com/glittercowboy/get-shit-done)
[![Fork](https://img.shields.io/badge/fork-RicherTunes%2Fget--shit--done-green?style=for-the-badge)](https://github.com/RicherTunes/get-shit-done)

</div>

---

## What's Different

This fork adds **multi-model execution** to reduce costs while maintaining quality:

| Original GSD | This Fork |
|--------------|-----------|
| All Claude (Opus/Sonnet) | Claude orchestrates, GLM implements, Codex reviews |
| ~$1.00 per feature | ~$0.15 per feature |
| Single provider | Multi-provider resilience |

### Model Hierarchy

| Role | Model | Provider | Purpose |
|------|-------|----------|---------|
| **Orchestrator** | Opus 4.5 | Claude Code | Planning, decisions, commits |
| **Implementer** | GLM 4.7 | Z.AI (OpenCode CLI) | Writing code |
| **Reviewer** | GPT-5.2 | Codex CLI | Code review |
| **Fallback** | Gemini 2.5 | Google API | Alternative provider |

---

## Prerequisites

Before using this fork, install and authenticate the CLIs:

### 1. OpenCode CLI (Z.AI)

```bash
npm install -g @anthropic/opencode
opencode auth zai-coding-plan
```

### 2. Codex CLI (OpenAI)

```bash
npm install -g @openai/codex
codex auth
```

### 3. GetShitDone CLI (Multi-Model Tool)

```bash
git clone https://github.com/RicherTunes/getshitdone.git ~/Projects/getshitdone
cd ~/Projects/getshitdone
npm install
npm link
```

Verify setup:
```bash
gsd doctor
```

---

## Installation

```bash
# Clone this fork
git clone https://github.com/RicherTunes/get-shit-done.git
cd get-shit-done

# Install to Claude Code
node bin/install.js --global
```

Or use npx with the original and manually copy the multi-model files:
```bash
npx get-shit-done-cc
# Then copy agents/gsd-executor.md and get-shit-done/references/multi-model-execution.md
```

---

## How Multi-Model Execution Works

### During `/gsd:execute-phase`

1. **Orchestrator (Opus)** reads the plan and breaks it into tasks
2. **For each simple task:**
   - Creates detailed implementation spec
   - Delegates to **GLM 4.7** via OpenCode CLI
   - GLM creates/modifies files
   - Captures changes via `git diff`
   - Sends to **Codex GPT-5.2** for review
   - If APPROVED: commits with multi-model attribution
   - If FEEDBACK: refines and retries (max 3x)
3. **For complex tasks:** Opus executes directly

### Delegation Decision

| Task Type | Model | Rationale |
|-----------|-------|-----------|
| Simple implementations | GLM 4.7 | Fast, cheap, good at code |
| Bug fixes | GLM 4.7 | Mechanical fixes |
| Refactoring | GLM 4.7 | Pattern transformations |
| Architecture decisions | Opus 4.5 | Strategic thinking |
| Complex integrations | Opus 4.5 | Full context needed |

### Commit Attribution

```
feat(08-02): implement user authentication

- Add login endpoint
- Add JWT token generation
- Add password hashing

Co-Authored-By: GLM 4.7 <noreply@z.ai>
Co-Authored-By: Codex GPT-5.2 <noreply@openai.com>
Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>
```

---

## Configuration

### Per-Project Model Config

Create `.gsd/models.json` in your project:

```json
{
  "glm": {
    "model": "zai-coding-plan/glm-4.7",
    "timeout": 300000,
    "fallback": "zai-coding-plan/glm-4.7-flash"
  },
  "codex": {
    "model": "gpt-5.2-codex",
    "timeout": 60000,
    "sandbox": "read-only"
  }
}
```

### Environment Variables

```bash
# Optional: Gemini fallback
export GEMINI_API_KEY="your-key"
```

---

## New Commands

This fork adds:

| Command | Purpose |
|---------|---------|
| `/gsd:delegate` | Manually delegate task to multi-model flow |

---

## Troubleshooting

### OpenCode not found

```bash
# Check installation
which opencode

# Check available models
opencode models | grep zai-coding-plan
```

### Codex authentication failed

```bash
# Re-authenticate
codex auth

# Verify
codex whoami
```

### GetShitDone CLI errors

```bash
# Health check
gsd doctor

# Clear sessions
rm .gsd/sessions.json
```

---

## Credits

- **Original GSD** by [TÂCHES / glittercowboy](https://github.com/glittercowboy/get-shit-done)
- **Multi-model fork** by [RicherTunes](https://github.com/RicherTunes/get-shit-done)
- **GetShitDone CLI** - Multi-model orchestration tool

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

<div align="center">

**Save 85% on AI costs. Same quality.**

</div>
