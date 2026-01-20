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

## Usage Examples

### Automatic Multi-Model Flow (via `gsd` CLI)

The `gsd` CLI automatically orchestrates GLM for implementation and Codex for review:

```bash
# Health check - verify all CLIs are available
gsd doctor

# Simple task - auto-delegates to GLM, reviewed by Codex
gsd do "Add a validateEmail function to src/utils/validators.js"

# With implementation spec file for complex tasks
gsd do "Implement user registration" --spec registration-spec.md

# Specify target file explicitly
gsd do "Add logout endpoint" --target src/api/auth/logout.ts

# Add extra context for the implementer
gsd do "Fix the date formatting bug" --context "Use ISO 8601 format, timezone UTC"

# Force delegation even if task seems complex
gsd do "Refactor authentication module" --no-classify

# Check if a task would be delegated or kept in Opus
gsd classify "Add a simple utility function"
# → DELEGATE to GLM (80% confidence)

gsd classify "Design the database schema for multi-tenancy"
# → KEEP in Opus (80% confidence)
```

### Direct Model Usage (bypassing `gsd`)

For finer control, call the CLIs directly:

#### GLM (OpenCode) - Implementation

```bash
# Simple prompt
opencode run -m zai-coding-plan/glm-4.7 "Create a function that validates URLs"

# With stdin for longer prompts (recommended)
cat implementation-spec.md | opencode run -m zai-coding-plan/glm-4.7 -

# Use faster model for simple tasks
opencode run -m zai-coding-plan/glm-4.7-flash "Add console.log debugging"

# List available models
opencode models | grep zai
```

#### Codex (OpenAI) - Review & Analysis

```bash
# Code review with high reasoning
cat changes.diff | codex exec -m gpt-5.2-codex --sandbox read-only --reasoning-effort high -

# Quick review with lower reasoning
git diff | codex exec -m gpt-5.2-codex --sandbox read-only --reasoning-effort low -

# Architecture review
codex -m gpt-5.2-codex --sandbox read-only --reasoning-effort high \
  -p "Review the architecture of this project for scalability concerns"

# Security audit
codex -m gpt-5.2-codex --sandbox read-only --reasoning-effort high \
  -p "Audit this codebase for security vulnerabilities, focus on auth and input validation"
```

#### Claude (via Claude Code) - Planning & Orchestration

```bash
# Claude Code handles planning automatically when you run GSD commands
# These run inside Claude Code, not as standalone CLI:

/gsd:plan-phase      # Claude plans the phase
/gsd:execute-phase   # Claude orchestrates, delegates to GLM/Codex
/gsd:delegate        # Manually delegate current task to multi-model flow
```

### Example: Full Feature Implementation

```bash
# 1. Create implementation spec
cat > feature-spec.md << 'EOF'
## Task
Add user profile endpoint with avatar upload

## Target
src/api/users/profile.ts

## Requirements
- GET /api/users/:id/profile - return user profile
- PUT /api/users/:id/profile - update profile
- POST /api/users/:id/avatar - upload avatar (max 5MB, jpg/png)
- Store avatars in /uploads/avatars/

## Context
- Framework: Express with TypeScript
- Auth: JWT middleware already exists at src/middleware/auth.ts
- File upload: Use multer
- Validation: Use zod schemas
EOF

# 2. Run with gsd (GLM implements, Codex reviews)
gsd do "Implement user profile feature" --spec feature-spec.md

# 3. If approved, changes are ready to commit
git status
git add -A
git commit -m "feat: add user profile endpoint with avatar upload

Co-Authored-By: GLM 4.7 <noreply@z.ai>
Co-Authored-By: Codex GPT-5.2 <noreply@openai.com>"
```

### Example: Bug Fix Workflow

```bash
# 1. Classify the task
gsd classify "Fix the off-by-one error in pagination"
# → DELEGATE to GLM (mechanical fix)

# 2. Run the fix
gsd do "Fix the off-by-one error in pagination" \
  --target src/utils/pagination.ts \
  --context "Line 42 uses < instead of <=, causing last page to be skipped"

# 3. Review output and commit
```

### Example: Direct Codex Review

```bash
# Review recent changes before committing
git diff HEAD~1 > recent-changes.diff
cat recent-changes.diff | codex exec -m gpt-5.2-codex \
  --sandbox read-only \
  --reasoning-effort high \
  -

# Review specific file for issues
cat src/auth/login.ts | codex exec -m gpt-5.2-codex \
  --sandbox read-only \
  --reasoning-effort medium \
  -p "Review this authentication code for security issues"
```

## GSD Workflow Integration

### During `/gsd:execute-phase`

1. **Orchestrator (Opus)** reads the plan and breaks it into tasks
2. **For each task:**
   - Runs `gsd classify` to check complexity
   - If delegatable: runs `gsd do` with spec
   - If complex: Opus implements directly
3. **On approval:** Commits with multi-model attribution
4. **On feedback:** Refines and retries (max 3 iterations)

### Review Flow

```
┌─────────────────┐
│  gsd do "task"  │
└────────┬────────┘
         ▼
┌─────────────────┐
│  GLM implements │ ← Creates/modifies files
└────────┬────────┘
         ▼
┌─────────────────┐
│  git diff HEAD  │ ← Captures changes
└────────┬────────┘
         ▼
┌─────────────────┐
│  Codex reviews  │ ← APPROVED or FEEDBACK
└────────┬────────┘
         ▼
    ┌────┴────┐
    │         │
 APPROVED  FEEDBACK
    │         │
    ▼         ▼
  Done    Retry (max 3x)
```

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
