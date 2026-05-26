---
name: codex-brief-generator
description: Use when the user wants Hermes to generate a self-contained task brief/prompt for Codex CLI, especially when Codex will be run manually in Windows or WSL for long-running coding, data, research, or local-application tasks.
version: 1.0.0
author: Hermes Agent
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [codex, prompt-generation, task-brief, windows, wsl, coding-agent]
    related_skills: [codex, hermes-agent, writing-plans, test-driven-development]
---

# Codex Brief Generator

## Overview

Use this skill to turn a user's request into a high-quality, self-contained instruction prompt for Codex CLI. The target use case is a two-agent workflow:

- **Hermes** acts as the research lead, product/engineering planner, reviewer, and prompt author.
- **Codex CLI** acts as the local execution agent that edits files, runs commands, tests code, and reports results.
- **The user** may manually copy the generated brief into Windows Codex or WSL Codex, especially for long-running tasks or tasks requiring local Windows applications.

The output must be self-contained and usable by Codex without relying on any prior chat state. Write the brief as a standalone document with enough background, paths, scope, constraints, validation steps, and final-report requirements.

## When to Use

Use this skill when the user asks for:

- A prompt to paste into Codex CLI.
- A Windows Codex task brief generated from a Hermes conversation.
- A long-running coding/data task where Hermes should not directly run Codex.
- A repo modification plan that Codex will execute locally.
- A data-processing or econometrics pipeline task that needs reproducible scripts and tests.
- A prompt that includes task background, implementation plan, guardrails, validation commands, and final report format.

Do **not** use this skill when:

- The user wants Hermes to execute the task directly with terminal/file tools.
- The task is a one-line quick command that does not need a structured brief.
- The user asks for general explanation rather than a copy-paste Codex instruction.
- Critical project paths, input files, or desired outputs are completely unknown and cannot be inferred. In that case ask for the missing fields, or produce a clearly marked template with placeholders.

## Core Principle

A Codex brief must be **self-contained, bounded, testable, and handoff-safe**.

Good Codex briefs answer:

1. What is the project and why does the task matter?
2. Where should Codex work?
3. What exactly should Codex change?
4. What must Codex not change?
5. How should Codex verify the work?
6. What should Codex report back to the user?
7. What should Codex do if assumptions fail?

Avoid vague references such as:

- “as discussed above”
- “use the previous plan”
- “fix the issue”
- “make it better”

Replace them with explicit facts and constraints.

## Output Contract

When the user requests a Codex prompt, output **only the copy-pasteable Codex brief** unless the user also asks for explanation. If explanation is needed, put it before or after the brief under a short Chinese heading, but keep the brief itself clean.

Use this wrapper:

```markdown
下面是可以直接复制给 Codex 的提示词：

```text
<Codex brief here>
```
```

If the user wants English, write the Codex brief in English. If not specified, prefer English for the brief when Codex is likely to work on code, because code agents and repository conventions often perform better with English technical instructions. Add Chinese commentary outside the brief only if useful.

## Standard Codex Brief Template

Use the following structure. Omit sections only when clearly irrelevant.

```text
# Codex Task Brief

## 0. Operating Mode
You are Codex CLI running locally on <Windows/WSL/Linux/macOS>. Work inside the specified repository. Treat this brief as the complete task specification.

If any critical assumption is false, stop and report the mismatch instead of inventing data, paths, APIs, or requirements.

## 1. Background
<Project/research/product context. Explain enough for Codex to understand the purpose without needing the Hermes conversation.>

## 2. Objective
<One concise paragraph describing the final outcome.>

## 3. Working Directory and Environment
- Working directory: `<absolute or user-provided path>`
- OS/environment: `<Windows PowerShell / WSL Ubuntu / Linux / macOS>`
- Expected language/runtime: `<Python/Node/R/etc.>`
- Package manager, if known: `<uv/pip/npm/pnpm/conda/etc.>`
- Important local applications, if relevant: `<Chrome/Office/Zotero/Stata/RStudio/etc.>`

Before editing, inspect the repository structure and current git status.

## 4. Inputs
List all input files, folders, APIs, datasets, schemas, examples, or user-provided materials.

For each input, specify:
- Path or source
- Expected role
- Whether it is read-only
- Any known schema/columns/format

## 5. Required Outputs
List the exact files, scripts, generated artifacts, reports, logs, or command outputs that should exist after completion.

## 6. Scope of Work
Implement the following, in order:

1. <Step 1>
2. <Step 2>
3. <Step 3>

You may adjust implementation details after inspecting the repo, but do not expand the task beyond this scope.

## 7. Constraints and Guardrails
- Do not delete or overwrite raw data.
- Do not fabricate data, variables, regression results, credentials, or API responses.
- Do not introduce large dependencies unless necessary; justify any new dependency.
- Do not modify unrelated files.
- Keep changes minimal, readable, and reversible.
- Preserve existing public APIs unless explicitly asked to change them.
- If a required file/field/API is missing, stop and report what is missing.
- If running on Windows, use Windows-compatible paths and commands unless the repo clearly expects WSL/Linux.
- Never print secrets or API keys in logs or final output.

## 8. Implementation Guidance
Suggested approach:

1. Inspect repository structure and existing conventions.
2. Locate relevant files and tests.
3. Make a small implementation plan.
4. Edit files incrementally.
5. Run targeted tests after each major change.
6. Run final validation commands.
7. Summarize changes and remaining risks.

Prefer existing project conventions over inventing new architecture.

## 9. Validation Commands
Run the relevant commands and include outputs or summaries in the final report:

```bash
<command 1>
<command 2>
<command 3>
```

If a command is unavailable on the local system, explain why and run the closest valid alternative.

## 10. Final Report Format
At the end, report in this exact structure:

1. Summary of completed work
2. Files changed
3. Commands run and results
4. Tests/validation status
5. Assumptions made
6. Issues encountered or unresolved risks
7. Recommended next steps

Do not claim success unless validation was actually run or explicitly explain why it could not be run.
```

## Task-Type Add-ons

### Data / Econometrics Pipeline Add-on

Use this when the task involves CSV/Excel/PDF/data cleaning, panel construction, regressions, event studies, or empirical finance/econometrics.

Add to the brief:

```text
## Econometrics/Data Requirements
- Keep raw data immutable. Write cleaned outputs to a separate `processed/`, `output/`, or project-standard directory.
- Produce a data dictionary or schema summary for generated datasets.
- Log row counts at each major transformation step.
- Explicitly handle missing values and duplicates; do not silently drop observations.
- If constructing panel data, verify entity IDs, time index, duplicates, and balanced/unbalanced panel status.
- If running regressions, report model specification, fixed effects, clustered standard errors, sample restrictions, and exact dependent/independent variables.
- Save reproducible scripts instead of one-off notebook-only transformations unless the repo is notebook-first.
```

Suggested validation commands:

```bash
python -m pytest
python scripts/build_panel.py
python scripts/run_regressions.py
python scripts/validate_outputs.py
```

Only include commands that match the actual project.

### Windows Local Application Add-on

Use this when Codex will run on Windows and may need local apps such as Chrome, Office, Zotero, PDF tools, Stata, or RStudio.

Add to the brief:

```text
## Windows Local Application Notes
- You are running on Windows. Prefer PowerShell-compatible commands unless the repo clearly uses bash.
- Use Windows paths such as `C:\Users\...` where applicable.
- If interacting with GUI applications, first check whether a CLI/API automation route exists. Prefer CLI/API automation over manual GUI steps.
- Do not assume WSL paths like `/home/...` are directly available unless explicitly provided.
- If you need to open a browser or local app, explain what you opened and why in the final report.
```

### Repo Code Modification Add-on

Use this for feature implementation, bug fixing, refactoring, or PR preparation.

Add to the brief:

```text
## Code Quality Requirements
- Inspect existing style, naming, test layout, and dependency conventions before editing.
- Add or update tests for behavioral changes.
- Keep commits/changes logically grouped if you create commits.
- Avoid broad refactors unless needed for the task.
- If tests fail due to pre-existing unrelated issues, document evidence and isolate your changes.
```

### Documentation / README Add-on

Use this when Codex should write docs.

Add to the brief:

```text
## Documentation Requirements
- Keep documentation concise and accurate.
- Do not overclaim capabilities.
- Include exact commands and expected outputs when relevant.
- If bilingual docs are requested, keep English and Chinese versions synchronized.
```

## Hermes Pre-Brief Checklist

Before generating the Codex brief, collect or infer:

- [ ] Target environment: Windows Codex, WSL Codex, Linux Codex, or unspecified.
- [ ] Working directory or repository path.
- [ ] Task type: code, data/econometrics, docs, local app, debugging, review.
- [ ] Inputs and outputs.
- [ ] Hard constraints and forbidden actions.
- [ ] Validation commands.
- [ ] Whether Codex should modify files, only inspect, or produce a plan first.
- [ ] Whether the task may run long enough to require manual Codex execution.

If one or two fields are missing but not critical, insert placeholders like `<PROJECT_PATH>` and clearly mark them. If critical fields are missing, ask a targeted clarification question before writing the final brief.

## Brief Quality Rubric

Before sending the prompt to the user, self-check:

- **Self-contained**: Codex can understand the task without the Hermes chat.
- **Bounded**: Scope and forbidden actions are clear.
- **Executable**: It includes paths, commands, files, or placeholders.
- **Verifiable**: It tells Codex how to prove completion.
- **Safe**: It protects raw data, secrets, unrelated files, and user credentials.
- **Reportable**: It demands a structured final report with evidence.
- **Environment-aware**: Windows/WSL/Linux path and command assumptions are explicit.

If any item fails, revise before output.

## Suggested Hermes Response Pattern

When the user asks for a Codex prompt, respond with:

```markdown
下面是可以直接复制给 Codex 的提示词：

```text
# Codex Task Brief
...
```

我已经按以下标准自检：自包含、边界清楚、可执行、可验证、安全、最终报告格式明确。
```

If the user explicitly wants only the prompt, omit the self-check sentence outside the code block.

## Common Pitfalls

1. **Brief depends on hidden Hermes context.** Fix by restating all relevant facts.

2. **No working directory.** Codex needs to know where to work. If unknown, use `<PROJECT_PATH>` and tell the user to replace it.

3. **No validation commands.** Even if commands are uncertain, include a validation discovery instruction: inspect package files and run the closest test/lint/build command.

4. **Codex allowed to over-expand.** Always include “do not expand scope” and “do not modify unrelated files.”

5. **Raw data risk.** For research/data tasks, explicitly mark raw data read-only and require processed outputs elsewhere.

6. **Windows/WSL confusion.** State whether paths are Windows (`C:\...`) or WSL (`/home/...`). Do not mix without explanation.

7. **Fake success.** Require Codex to report commands actually run and distinguish passed, failed, and not run.

8. **Overly academic plan but weak execution detail.** Codex needs concrete file operations, tests, and artifacts more than high-level discussion.

9. **Too many optional branches.** If the prompt contains many “maybe” paths, Codex may wander. Give a preferred path and bounded fallback rules.

10. **Secrets in prompt.** Never include API keys, tokens, cookies, or private credentials. Use placeholders.

## Verification Checklist for Generated Briefs

- [ ] The brief starts with `# Codex Task Brief`.
- [ ] It states operating mode and environment.
- [ ] It includes working directory or a clear placeholder.
- [ ] It includes objective, inputs, outputs, scope, constraints, validation, and final report format.
- [ ] It tells Codex to stop and report if assumptions are false.
- [ ] It forbids fabricating data/results and exposing secrets.
- [ ] It distinguishes raw inputs from generated outputs when data is involved.
- [ ] It includes task-type add-ons when relevant.
- [ ] It is ready to paste into Codex without additional Hermes context.
