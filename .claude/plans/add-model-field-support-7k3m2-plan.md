# Add Model Field Support to Slash Commands - Implementation Plan

**Status**: READY FOR IMPLEMENTATION
**Mode**: informational
**Created**: 2026-01-15

## Summary

Add `model:` field support to all slash commands in `essentials/commands/*.md` to allow explicit model specification (haiku, sonnet, opus) in command frontmatter. This enables cost-optimized execution where loop commands can default to cheaper/faster models (haiku) while complex creator commands remain on opus.

## Files

### Files to Edit
- `essentials/commands/beads-loop.md`
- `essentials/commands/implement-loop.md`
- `essentials/commands/spec-loop.md`
- `essentials/commands/cancel-beads.md`
- `essentials/commands/cancel-implement.md`
- `essentials/commands/cancel-spec-loop.md`
- `essentials/commands/plan-creator.md`
- `essentials/commands/proposal-creator.md`
- `essentials/commands/prompt-creator.md`
- `essentials/commands/bug-plan-creator.md`
- `essentials/commands/code-quality-plan-creator.md`
- `essentials/commands/document-creator.md`
- `essentials/commands/codemap-creator.md`
- `essentials/commands/mr-description-creator.md`
- `essentials/commands/beads-creator.md`
- `.claude/templates/COMMAND_TEMPLATE.md`

### Files to Create
None

---

## Code Context

### Current Command Frontmatter Structure

**Location**: `essentials/commands/*.md` (lines 1-10 typically)

Current commands use this frontmatter pattern (example from `beads-loop.md`):
```yaml
---
allowed-tools: Bash, Read, TodoWrite, Glob, Grep, AskUserQuestion, Task, TaskOutput
argument-hint: "[--step|--auto] [--label <label>] [--max-iterations N]"
description: "Execute beads iteratively until all tasks complete"
hide-from-slash-command-tool: "true"
---
```

**No `model` field exists in any current commands.**

### Agent Frontmatter Pattern (Reference)

**Location**: `essentials/agents/*.md` (lines 1-25)

Agents already have model field:
```yaml
---
name: plan-creator-default
description: |
  Creates comprehensive architectural plans...
model: opus
color: green
---
```

**All agents use `model: opus`** - this is the alias form.

### Command Template Structure

**Location**: `.claude/templates/COMMAND_TEMPLATE.md` (lines 9-16)

Current template shows:
```markdown
---
allowed-tools: <Tool1, Tool2, ...>
argument-hint: "<argument pattern>"
description: <One-line description>
context: fork  # Optional
---
```

**Template does NOT include `model` field.**

---

## External Context

### Claude Code Model Field Support

Based on the user's research and Claude Code documentation:

1. **Slash commands support `model` field in frontmatter** - confirmed supported
2. **Model aliases supported**: `haiku`, `sonnet`, `opus` (short form works)
3. **Full model strings also work**: `claude-haiku-4-5-20251001`, `claude-sonnet-4-20250514`, `claude-opus-4-5-20251101`
4. **If omitted**: Model inherits from conversation context

### Recommendation: Use Aliases

Aliases are preferred because:
- Shorter and more readable
- Version-independent (auto-updates with Claude Code)
- Consistent with agent pattern already in use
- Example: `model: haiku` vs `model: claude-haiku-4-5-20251001`

---

## Risk Analysis

### Technical Risks

| Risk | Likelihood | Impact | Mitigation Strategy |
|------|------------|--------|---------------------|
| Model field not recognized by Claude Code | Low | High | Already confirmed working via documentation |
| Alias format not supported | Low | Medium | Use full model string as fallback if needed |
| Breaking existing command behavior | Low | Medium | Model field is additive, won't break existing |

### Integration Risks

| Risk | Likelihood | Impact | Mitigation Strategy |
|------|------------|--------|---------------------|
| Inconsistent model choices across commands | Low | Low | Define clear policy in plan |
| Users confused by different models | Low | Low | Document model choices in README |

### Rollback Strategy

**Trigger Conditions**: If model field causes command failures

**Rollback Steps**:
1. Remove `model:` line from affected command frontmatter
2. Git revert if needed: `git revert <commit>`
3. Verify commands work with inherited model

**Overall Risk Level**: Low

---

## Architectural Narrative

### Task

Add `model:` field to frontmatter of all slash commands in `essentials/commands/*.md` to enable explicit model selection. This allows:
- Loop commands (`beads-loop`, `implement-loop`, `spec-loop`) to use `haiku` for cost-effective iteration
- Cancel commands to use `haiku` (lightweight operations)
- Creator commands to remain on `opus` for complex reasoning
- Template updated to document the pattern

### Architecture

The essentials-claude-code plugin has two types of prompt files:

1. **Commands** (`essentials/commands/*.md`) - Slash command definitions
   - Frontmatter defines: `allowed-tools`, `argument-hint`, `description`, optional `context`, optional `hide-from-slash-command-tool`
   - Body defines: Instructions, workflow, error handling

2. **Agents** (`essentials/agents/*.md`) - Subagent definitions
   - Frontmatter defines: `name`, `description`, `model`, `color`
   - Body defines: Core principles, phases, rules

**Current Gap**: Commands lack `model` field while agents have it.

### Selected Context

| File | Purpose | Model Assignment |
|------|---------|------------------|
| `beads-loop.md` | Iterate on beads until complete | haiku (loop) |
| `implement-loop.md` | Iterate on plan until exit criteria pass | haiku (loop) |
| `spec-loop.md` | Iterate on spec tasks until complete | haiku (loop) |
| `cancel-beads.md` | Cancel beads loop | haiku (lightweight) |
| `cancel-implement.md` | Cancel implement loop | haiku (lightweight) |
| `cancel-spec-loop.md` | Cancel spec loop | haiku (lightweight) |
| `plan-creator.md` | Create architectural plans | opus (complex reasoning) |
| `proposal-creator.md` | Convert plans to OpenSpec | opus (complex reasoning) |
| `prompt-creator.md` | Create quality prompts | opus (complex reasoning) |
| `bug-plan-creator.md` | Create bug investigation plans | opus (complex reasoning) |
| `code-quality-plan-creator.md` | Create code quality plans | opus (complex reasoning) |
| `document-creator.md` | Generate DEVGUIDE.md | opus (complex reasoning) |
| `codemap-creator.md` | Generate code maps | opus (complex reasoning) |
| `mr-description-creator.md` | Generate MR/PR descriptions | opus (complex reasoning) |
| `beads-creator.md` | Convert specs to beads | opus (complex reasoning) |

### Relationships

```
Commands (add model field here)
    └── Invoke Agents (already have model field)
         └── Agents use their own model setting

The command's model field affects the command orchestrator.
The agent's model field affects the subagent execution.
Both are independent settings.
```

### Implementation Notes

**Pattern to follow** (from agents):
```yaml
model: opus
```

**Insert position**: After `description:` line in frontmatter, before optional fields like `context:` or `hide-from-slash-command-tool:`.

**Model assignment logic**:
- Loops = `haiku` (iterative, many calls, benefit from speed/cost)
- Cancels = `haiku` (simple state management)
- Creators = `opus` (complex reasoning, architectural decisions)

### Ambiguities

**Resolved**:
- Q: Use aliases or full strings? A: Use aliases (shorter, version-independent)
- Q: What model for loops? A: haiku (cost-effective for iteration)
- Q: What model for creators? A: opus (maintains current quality)

### Requirements

1. Add `model: haiku` to all loop commands (`*-loop.md`)
2. Add `model: haiku` to all cancel commands (`cancel-*.md`)
3. Add `model: opus` to all creator commands (`*-creator.md`)
4. Update `COMMAND_TEMPLATE.md` to include `model:` field with documentation
5. Maintain consistent YAML formatting in all frontmatter blocks
6. Preserve all existing frontmatter fields unchanged

### Constraints

- Use `model: haiku` or `model: opus` alias format (not full model strings)
- Insert `model:` line after `description:` for consistency
- Do NOT modify command body content
- Do NOT modify allowed-tools, argument-hint, or other existing fields
- YAML frontmatter must remain valid (proper `---` delimiters)

### Stakeholders

**Primary**:
- Plugin users who will see different models for different commands
- Claude Code system that parses frontmatter

**Secondary**:
- Future maintainers who will follow the template pattern

---

## Implementation Plan

### essentials/commands/beads-loop.md [edit]

**Purpose**: Add haiku model for cost-effective loop iteration

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: haiku` line after `description:` (line 5)

**Implementation Details**:
- Insert new line: `model: haiku`
- Position: After line 4 (`description: "Execute beads iteratively..."`)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Bash, Read, TodoWrite, Glob, Grep, AskUserQuestion, Task, TaskOutput
argument-hint: "[--step|--auto] [--label <label>] [--max-iterations N]"
description: "Execute beads iteratively until all tasks complete"
hide-from-slash-command-tool: "true"
---

# AFTER (new):
---
allowed-tools: Bash, Read, TodoWrite, Glob, Grep, AskUserQuestion, Task, TaskOutput
argument-hint: "[--step|--auto] [--label <label>] [--max-iterations N]"
description: "Execute beads iteratively until all tasks complete"
model: haiku
hide-from-slash-command-tool: "true"
---
```

**Dependencies**: None
**Provides**: Pattern for other loop commands

---

### essentials/commands/implement-loop.md [edit]

**Purpose**: Add haiku model for cost-effective loop iteration

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: haiku` line after `description:` (line 5)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Bash, Read, Glob, Grep, Write, Edit, TodoWrite, AskUserQuestion, WebFetch
argument-hint: "<plan-path> [--step|--auto] [--max-iterations N]"
description: "Implement a plan with iterative loop until completion"
hide-from-slash-command-tool: "true"
---

# AFTER (new):
---
allowed-tools: Bash, Read, Glob, Grep, Write, Edit, TodoWrite, AskUserQuestion, WebFetch
argument-hint: "<plan-path> [--step|--auto] [--max-iterations N]"
description: "Implement a plan with iterative loop until completion"
model: haiku
hide-from-slash-command-tool: "true"
---
```

**Dependencies**: None
**Provides**: Consistent pattern with beads-loop

---

### essentials/commands/spec-loop.md [edit]

**Purpose**: Add haiku model for cost-effective loop iteration

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: haiku` line after `description:` (line 5)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Bash, Read, Glob, Grep, Write, Edit, TodoWrite, AskUserQuestion
argument-hint: "<change-id> [--step|--auto] [--max-iterations N]"
description: "Implement an OpenSpec change iteratively until all tasks complete"
hide-from-slash-command-tool: "true"
---

# AFTER (new):
---
allowed-tools: Bash, Read, Glob, Grep, Write, Edit, TodoWrite, AskUserQuestion
argument-hint: "<change-id> [--step|--auto] [--max-iterations N]"
description: "Implement an OpenSpec change iteratively until all tasks complete"
model: haiku
hide-from-slash-command-tool: "true"
---
```

**Dependencies**: None
**Provides**: Consistent pattern with other loops

---

### essentials/commands/cancel-beads.md [edit]

**Purpose**: Add haiku model for lightweight cancel operation

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: haiku` line after `description:` (line 4)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-5):
---
allowed-tools: Bash, Read, TodoWrite
argument-hint: ""
description: Cancel active beads loop
---

# AFTER (new):
---
allowed-tools: Bash, Read, TodoWrite
argument-hint: ""
description: Cancel active beads loop
model: haiku
---
```

**Dependencies**: None
**Provides**: Pattern for other cancel commands

---

### essentials/commands/cancel-implement.md [edit]

**Purpose**: Add haiku model for lightweight cancel operation

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: haiku` line after `description:` (line 4)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-5):
---
allowed-tools: Bash, Read, TodoWrite
argument-hint: ""
description: Cancel active implement loop
---

# AFTER (new):
---
allowed-tools: Bash, Read, TodoWrite
argument-hint: ""
description: Cancel active implement loop
model: haiku
---
```

**Dependencies**: None
**Provides**: Consistent pattern with cancel-beads

---

### essentials/commands/cancel-spec-loop.md [edit]

**Purpose**: Add haiku model for lightweight cancel operation

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: haiku` line after `description:` (line 4)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-5):
---
allowed-tools: Bash, Read, TodoWrite
argument-hint: ""
description: Cancel active spec loop
---

# AFTER (new):
---
allowed-tools: Bash, Read, TodoWrite
argument-hint: ""
description: Cancel active spec loop
model: haiku
---
```

**Dependencies**: None
**Provides**: Consistent pattern with other cancels

---

### essentials/commands/plan-creator.md [edit]

**Purpose**: Add opus model for complex reasoning

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: opus` line after `description:` (line 5)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Glob, Grep, Read, Bash, Write, Edit, WebFetch, WebSearch
argument-hint: "<task description>"
description: Create a comprehensive architectural plan for any task - detailed enough to feed directly into /implement-loop or OpenSpec
context: fork
---

# AFTER (new):
---
allowed-tools: Glob, Grep, Read, Bash, Write, Edit, WebFetch, WebSearch
argument-hint: "<task description>"
description: Create a comprehensive architectural plan for any task - detailed enough to feed directly into /implement-loop or OpenSpec
model: opus
context: fork
---
```

**Dependencies**: None
**Provides**: Pattern for other creator commands

---

### essentials/commands/proposal-creator.md [edit]

**Purpose**: Add opus model for complex reasoning

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: opus` line after `description:` (line 5)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Task, TaskOutput, Bash, Read, Glob, Grep, Write, AskUserQuestion
argument-hint: "[plan-path | description]"
description: Convert plans, context, or descriptions into validated OpenSpec proposals (project)
context: fork
---

# AFTER (new):
---
allowed-tools: Task, TaskOutput, Bash, Read, Glob, Grep, Write, AskUserQuestion
argument-hint: "[plan-path | description]"
description: Convert plans, context, or descriptions into validated OpenSpec proposals (project)
model: opus
context: fork
---
```

**Dependencies**: None
**Provides**: Consistent pattern with plan-creator

---

### essentials/commands/prompt-creator.md [edit]

**Purpose**: Add opus model for complex reasoning

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: opus` line after `description:` (line 5)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Task, TaskOutput, Bash, Read, Glob, Grep, Write, AskUserQuestion
argument-hint: "<prompt description>"
description: Create high-quality prompts from any description (project)
context: fork
---

# AFTER (new):
---
allowed-tools: Task, TaskOutput, Bash, Read, Glob, Grep, Write, AskUserQuestion
argument-hint: "<prompt description>"
description: Create high-quality prompts from any description (project)
model: opus
context: fork
---
```

**Dependencies**: None
**Provides**: Consistent pattern with other creators

---

### essentials/commands/bug-plan-creator.md [edit]

**Purpose**: Add opus model for complex reasoning

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: opus` line after `description:` (line 5)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Glob, Grep, Read, Bash, Write, Edit, WebFetch, WebSearch
argument-hint: "<error-message-or-symptom> [additional context]"
description: Deep bug investigation with architectural fix plan generation - detailed enough to feed directly into /implement-loop or OpenSpec
context: fork
---

# AFTER (new):
---
allowed-tools: Glob, Grep, Read, Bash, Write, Edit, WebFetch, WebSearch
argument-hint: "<error-message-or-symptom> [additional context]"
description: Deep bug investigation with architectural fix plan generation - detailed enough to feed directly into /implement-loop or OpenSpec
model: opus
context: fork
---
```

**Dependencies**: None
**Provides**: Consistent pattern with plan-creator

---

### essentials/commands/code-quality-plan-creator.md [edit]

**Purpose**: Add opus model for complex reasoning

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: opus` line after `description:` (line 5)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Glob, Grep, Read, Bash, Write, Edit, WebFetch, WebSearch
argument-hint: "<file-or-directory> [--focus <area>]"
description: LSP-powered architectural code quality analysis - generates comprehensive improvement plans for /implement-loop or OpenSpec
context: fork
---

# AFTER (new):
---
allowed-tools: Glob, Grep, Read, Bash, Write, Edit, WebFetch, WebSearch
argument-hint: "<file-or-directory> [--focus <area>]"
description: LSP-powered architectural code quality analysis - generates comprehensive improvement plans for /implement-loop or OpenSpec
model: opus
context: fork
---
```

**Dependencies**: None
**Provides**: Consistent pattern with plan-creator

---

### essentials/commands/document-creator.md [edit]

**Purpose**: Add opus model for complex reasoning

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: opus` line after `description:` (line 5)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Task, TaskOutput
argument-hint: "[directory-or-files]"
description: Generate DEVGUIDE.md architectural documentation using LSP (project)
context: fork
---

# AFTER (new):
---
allowed-tools: Task, TaskOutput
argument-hint: "[directory-or-files]"
description: Generate DEVGUIDE.md architectural documentation using LSP (project)
model: opus
context: fork
---
```

**Dependencies**: None
**Provides**: Consistent pattern with other creators

---

### essentials/commands/codemap-creator.md [edit]

**Purpose**: Add opus model for complex reasoning

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: opus` line after `description:` (line 5)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Task, TaskOutput
argument-hint: "[directory] [--ignore <patterns>]"
description: Generate comprehensive code maps with LSP for symbol tracking and reference verification (project)
context: fork
---

# AFTER (new):
---
allowed-tools: Task, TaskOutput
argument-hint: "[directory] [--ignore <patterns>]"
description: Generate comprehensive code maps with LSP for symbol tracking and reference verification (project)
model: opus
context: fork
---
```

**Dependencies**: None
**Provides**: Consistent pattern with other creators

---

### essentials/commands/mr-description-creator.md [edit]

**Purpose**: Add opus model for complex reasoning

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: opus` line after `description:` (line 5, after skills:)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-7):
---
allowed-tools: Task, TaskOutput, Bash, Read, Glob, Grep, AskUserQuestion
argument-hint: "[--template 'markdown']"
description: Generate and apply MR/PR description directly via gh or glab CLI (project)
skills: ["github-cli", "gitlab-cli"]
---

# AFTER (new):
---
allowed-tools: Task, TaskOutput, Bash, Read, Glob, Grep, AskUserQuestion
argument-hint: "[--template 'markdown']"
description: Generate and apply MR/PR description directly via gh or glab CLI (project)
model: opus
skills: ["github-cli", "gitlab-cli"]
---
```

**Dependencies**: None
**Provides**: Consistent pattern with other creators

---

### essentials/commands/beads-creator.md [edit]

**Purpose**: Add opus model for complex reasoning

**TOTAL CHANGES**: 1

**Changes**:
1. Add `model: opus` line after `description:` (line 5)

**Migration Pattern**:
```yaml
# BEFORE (current, lines 1-6):
---
allowed-tools: Task, TaskOutput, Bash, Read, Glob
argument-hint: "<spec-path> [spec-path-2] [spec-path-3] ..."
description: Convert OpenSpec specs to self-contained Beads (project)
---

# AFTER (new):
---
allowed-tools: Task, TaskOutput, Bash, Read, Glob
argument-hint: "<spec-path> [spec-path-2] [spec-path-3] ..."
description: Convert OpenSpec specs to self-contained Beads (project)
model: opus
---
```

**Dependencies**: None
**Provides**: Consistent pattern with other creators

---

### .claude/templates/COMMAND_TEMPLATE.md [edit]

**Purpose**: Update template to include model field documentation

**TOTAL CHANGES**: 2

**Changes**:
1. Add `model: <haiku|opus>` line to template frontmatter example (line 13)
2. Add `model:` documentation to Section Requirements (around line 189)

**Migration Pattern (Change 1 - frontmatter example)**:
```markdown
# BEFORE (lines 9-16):
```markdown
---
allowed-tools: <Tool1, Tool2, ...>
argument-hint: "<argument pattern>"
description: <One-line description>
context: fork  # Optional: runs command in isolated context (use for heavy exploration commands)
---

# AFTER (new):
```markdown
---
allowed-tools: <Tool1, Tool2, ...>
argument-hint: "<argument pattern>"
description: <One-line description>
model: <haiku|opus>  # Optional: haiku for loops/cancels, opus for creators
context: fork  # Optional: runs command in isolated context (use for heavy exploration commands)
---
```

**Migration Pattern (Change 2 - Section Requirements)**:
```markdown
# BEFORE (lines 184-189):
### Frontmatter (Required)
- `allowed-tools`: Comma-separated list of tools the command can use
- `argument-hint`: Pattern showing expected arguments (e.g., `<file1> [file2]`)
- `description`: Single-line description for help text
- Optional: `context: fork` for commands that spawn heavy background agents (isolates context from main session)
- Optional: `hide-from-slash-command-tool: "true"` for internal commands

# AFTER (new):
### Frontmatter (Required)
- `allowed-tools`: Comma-separated list of tools the command can use
- `argument-hint`: Pattern showing expected arguments (e.g., `<file1> [file2]`)
- `description`: Single-line description for help text
- Optional: `model: <haiku|opus>` for explicit model selection (haiku for loops/cancels, opus for creators)
- Optional: `context: fork` for commands that spawn heavy background agents (isolates context from main session)
- Optional: `hide-from-slash-command-tool: "true"` for internal commands
```

**Dependencies**: None
**Provides**: Documentation for future command creation

---

## Testing Strategy

### Manual Verification Steps

1. [ ] Run `/beads-loop --step` and verify it uses haiku model
2. [ ] Run `/implement-loop <plan> --step` and verify it uses haiku model
3. [ ] Run `/plan-creator test task` and verify it uses opus model
4. [ ] Verify YAML frontmatter is valid in all modified files

### Validation Commands

```bash
# Check YAML frontmatter syntax (basic validation)
for f in essentials/commands/*.md; do
  head -20 "$f" | grep -E "^---$" | wc -l | grep -q "2" || echo "FAIL: $f"
done

# Verify model field exists in all commands
for f in essentials/commands/*.md; do
  grep -q "^model:" "$f" || echo "MISSING model: $f"
done

# Verify correct model assignments
grep -l "model: haiku" essentials/commands/*-loop.md essentials/commands/cancel-*.md
grep -l "model: opus" essentials/commands/*-creator.md
```

---

## Success Metrics

### Functional Success Criteria

- [ ] All 15 command files have `model:` field in frontmatter
- [ ] Loop commands (`*-loop.md`) have `model: haiku`
- [ ] Cancel commands (`cancel-*.md`) have `model: haiku`
- [ ] Creator commands (`*-creator.md`) have `model: opus`
- [ ] Template updated with `model:` documentation
- [ ] All YAML frontmatter remains valid

### Quality Metrics

| Metric | Target | How to Measure |
|--------|--------|----------------|
| Commands updated | 15 | Count files with model field |
| YAML validity | 100% | Parse test each file |

### Acceptance Checklist

- [ ] All commands have model field
- [ ] Model assignments follow the policy (haiku for loops/cancels, opus for creators)
- [ ] Template documentation updated
- [ ] No other content modified in command files

---

## Exit Criteria

Exit criteria for `/implement-loop` - these commands MUST pass before implementation is complete.

### Verification Script

```bash
# Count model fields (should be 15)
model_count=$(grep -l "^model:" essentials/commands/*.md | wc -l | tr -d ' ')
[ "$model_count" -eq 15 ] && echo "PASS: All 15 commands have model field" || echo "FAIL: Only $model_count commands have model field"

# Verify haiku for loops and cancels
haiku_count=$(grep -l "model: haiku" essentials/commands/*-loop.md essentials/commands/cancel-*.md 2>/dev/null | wc -l | tr -d ' ')
[ "$haiku_count" -eq 6 ] && echo "PASS: 6 files have model: haiku" || echo "FAIL: Only $haiku_count files have model: haiku"

# Verify opus for creators
opus_count=$(grep -l "model: opus" essentials/commands/*-creator.md 2>/dev/null | wc -l | tr -d ' ')
[ "$opus_count" -eq 9 ] && echo "PASS: 9 files have model: opus" || echo "FAIL: Only $opus_count files have model: opus"

# Verify template updated
grep -q "model:" .claude/templates/COMMAND_TEMPLATE.md && echo "PASS: Template updated" || echo "FAIL: Template not updated"
```

### Success Conditions

- [ ] All 15 commands have `model:` field (verified by count)
- [ ] 6 files have `model: haiku` (3 loops + 3 cancels)
- [ ] 9 files have `model: opus` (all creators)
- [ ] Template documentation includes `model:` field

---

## Post-Implementation Verification

### Automated Checks

```bash
# Run verification script from Exit Criteria section
# All 4 checks must PASS
```

### Manual Verification Steps

1. [ ] Review git diff for unintended changes
2. [ ] Verify only frontmatter modified, not command body
3. [ ] Test one loop command and one creator command manually

### Success Criteria Validation

| Requirement | How to Verify | Verified? |
|-------------|---------------|-----------|
| 15 model fields | grep -l "^model:" count | [ ] |
| Correct haiku assignments | grep "model: haiku" in loops/cancels | [ ] |
| Correct opus assignments | grep "model: opus" in creators | [ ] |
| Template updated | grep "model:" in template | [ ] |

### Rollback Decision Tree

If issues found:
1. Minor issues (formatting) -> Fix in follow-up commit
2. Model field not working -> Remove model lines via git revert
3. Breaking changes -> Full git revert of commit
