# Multi-Model Execution Reference

This fork extends GSD with multi-model orchestration for cost-effective development.

## Model Hierarchy

| Role | Model | Provider | Purpose | Cost |
|------|-------|----------|---------|------|
| **Orchestrator** | Opus 4.5 | Claude Code | Planning, decisions, commits | $$$ |
| **Implementer** | GLM 4.7 | Z.AI (OpenCode) | Writing code | $ |
| **Reviewer** | GPT-5.2 | OpenAI (Codex) | Code review | $$ |
| **Fallback** | Gemini 2.5 | Google | Alternative provider | $$ |

## Prerequisites

### 1. OpenCode CLI (Z.AI GLM)

```bash
# Install OpenCode
npm install -g @anthropic/opencode

# Authenticate with Z.AI
opencode auth zai-coding-plan
```

Verify: `opencode models | grep zai-coding-plan`

### 2. Codex CLI (OpenAI)

```bash
# Install Codex
npm install -g @openai/codex

# Authenticate
codex auth
```

Verify: `codex --version`

### 3. GetShitDone CLI (Multi-Model Tool)

```bash
# Clone and install
git clone https://github.com/RicherTunes/getshitdone.git
cd getshitdone
npm install
npm link  # Makes 'gsd' command available globally
```

Verify: `gsd doctor`

## Configuration

All configuration files live in `.gsd/` folder in your project root.

### Model Config (`.gsd/models.json`)

Override defaults per project:

```json
{
  "glm": {
    "model": "zai-coding-plan/glm-4.7",
    "timeout": 300000,
    "fallback": "zai-coding-plan/glm-4.7-flash"
  },
  "codex": {
    "model": "gpt-5.2-codex",
    "reasoningEffort": "high",
    "timeout": 60000,
    "sandbox": "read-only"
  }
}
```

#### Model Options Reference

| Provider | Option | Type | Default | Description |
|----------|--------|------|---------|-------------|
| `glm` | `cli` | string | `"opencode"` | CLI executable |
| `glm` | `model` | string | `"zai-coding-plan/glm-4.7"` | Model ID |
| `glm` | `timeout` | number | `300000` | Max execution time (ms) |
| `glm` | `maxRetries` | number | `1` | Retry attempts |
| `glm` | `fallback` | string | `"glm-4.7-flash"` | Fallback model |
| `codex` | `cli` | string | `"codex"` | CLI executable |
| `codex` | `model` | string | `"gpt-5.2-codex"` | Model ID |
| `codex` | `reasoningEffort` | string | `"high"` | `"low"` \| `"medium"` \| `"high"` |
| `codex` | `timeout` | number | `60000` | Max review time (ms) |
| `codex` | `maxRetries` | number | `2` | Retry attempts |
| `codex` | `sandbox` | string | `"read-only"` | Sandbox mode |

### Prompt Templates (`.gsd/prompts.json`)

Customize prompts sent to models:

```json
{
  "review": "Review the following changes for task: {{task}}\n\nChanges:\n{{changes}}\n\nIMPORTANT: Your response MUST start with exactly one of these verdicts on its own line:\n- APPROVED (if the changes are good)\n- FEEDBACK: <your feedback> (if there are issues)\n\nDo not include any text before the verdict.",

  "implementation": "=== INSTRUCTIONS ===\n- Implement the task following project conventions\n- Use ES modules (import/export) syntax\n- Include appropriate error handling\n- Keep code simple and readable",

  "projectContext": "=== PROJECT CONTEXT ===\n{{context}}",

  "implementationSpec": "=== IMPLEMENTATION SPEC ===\n{{spec}}",

  "taskSection": "=== TASK ===\n{{task}}",

  "feedbackSection": "=== FEEDBACK TO ADDRESS ===\n{{feedback}}",

  "errorFeedback": "Previous attempt failed: {{error}}. Please try again."
}
```

#### Template Variables

| Variable | Used In | Description |
|----------|---------|-------------|
| `{{task}}` | review, taskSection | The task description |
| `{{changes}}` | review | Git diff + new file contents |
| `{{context}}` | projectContext | Project files, package.json info |
| `{{spec}}` | implementationSpec | Detailed implementation spec |
| `{{feedback}}` | feedbackSection | Review feedback for retry |
| `{{error}}` | errorFeedback | Error message from failed attempt |

### Environment Variables

```bash
# Optional: For Gemini fallback
export GEMINI_API_KEY="your-key"
```

## Usage in GSD Workflow

### Delegation Pattern

When executing tasks, the orchestrator (Opus) can delegate to GLM:

```markdown
## Implementation Spec

Task: Create user authentication endpoint

Target: src/api/auth/login.ts

Requirements:
- Validate email/password
- Return JWT on success
- Use bcrypt for password comparison

Context:
- Project uses Express
- JWT library: jose
- Database: PostgreSQL with Prisma
```

### CLI Commands

```bash
# Health check
gsd doctor

# Execute task with multi-model flow
gsd do "Create login endpoint with JWT authentication"

# Direct delegation
opencode run -m zai-coding-plan/glm-4.7 "spec..."
```

### Review Flow

1. **GLM implements** → Creates/modifies files
2. **Git captures changes** → `git diff` + new files
3. **Codex reviews** → Returns APPROVED or FEEDBACK
4. **Loop until approved** → Max 3 iterations
5. **Commit on approval** → Atomic commit with attribution

## Cost Optimization

### When to Use Each Model

| Task Type | Primary Model | Why |
|-----------|---------------|-----|
| Architecture decisions | Opus 4.5 | Complex reasoning |
| Simple implementations | GLM 4.7 | Fast, cheap, good at code |
| Bug fixes | GLM 4.7 | Straightforward tasks |
| Code review | Codex GPT-5.2 | Thorough, catches issues |
| Planning | Opus 4.5 | Strategic thinking |
| Refactoring | GLM 4.7 | Mechanical transformations |

### Estimated Savings

For a typical feature implementation:

| Approach | Cost |
|----------|------|
| All Opus | $1.00 |
| Multi-model (GLM + Codex) | $0.15 |
| Savings | ~85% |

## Troubleshooting

### OpenCode Issues

```bash
# Check available models
opencode models

# Test GLM directly
opencode run -m zai-coding-plan/glm-4.7 "Say hello"
```

### Codex Issues

```bash
# Check authentication
codex whoami

# Test review
echo "APPROVED" | codex exec -m gpt-5.2-codex --sandbox read-only -
```

### Session Issues

```bash
# Clear sessions
rm .gsd/sessions.json

# Check session state
cat .gsd/sessions.json
```

## Attribution

This multi-model approach was developed for cost-effective AI development:

- Original GSD by [TÂCHES](https://github.com/glittercowboy/get-shit-done)
- Multi-model fork by [RicherTunes](https://github.com/RicherTunes/get-shit-done)
- GetShitDone CLI: Multi-model orchestration tool
