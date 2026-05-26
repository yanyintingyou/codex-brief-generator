# Codex Brief Generator

[简体中文](README.zh-CN.md) | English

A Hermes / Agent Skills compatible skill for generating self-contained task briefs that can be copied into OpenAI Codex CLI.

This skill is designed for a practical two-agent workflow: Hermes acts as the research lead and task planner, while Codex CLI acts as the local execution agent that edits files, runs commands, validates results, and reports back.

## Motivation

Long-running local coding and data tasks often do not fit neatly into a single remote agent conversation. In practice, there are three recurring problems:

1. **Context handoff is fragile.** Codex may not have access to the previous Hermes conversation, so vague prompts like “do what we discussed” are unsafe.
2. **Local execution is environment-sensitive.** Windows, WSL, Linux, and macOS use different paths, shells, package managers, and local applications.
3. **Successful agent work needs verification.** A good prompt should not only say what to build; it should also tell Codex what not to touch, how to validate the result, and how to report evidence.

`codex-brief-generator` solves this by teaching Hermes to generate a complete, bounded, testable Codex task brief. The generated brief includes background, objective, working directory, inputs, outputs, implementation scope, guardrails, validation commands, and final report requirements.

The immediate motivation for this skill was a workflow where Hermes runs as a persistent research and planning assistant, while Codex is run manually on a local Windows or WSL machine for long-running tasks, local repository edits, econometrics/data pipelines, or Windows-application-related work.

## What It Generates

The skill guides Hermes to produce prompts with this structure:

- Operating mode
- Background
- Objective
- Working directory and environment
- Inputs
- Required outputs
- Scope of work
- Constraints and guardrails
- Implementation guidance
- Validation commands
- Final report format

It also includes add-ons for:

- Data and econometrics pipelines
- Windows local application tasks
- Code modification / refactoring tasks
- Documentation tasks

## Repository Structure

```text
.
├── README.md
├── README.zh-CN.md
├── LICENSE
├── .gitignore
└── skills/
    └── codex-brief-generator/
        └── SKILL.md
```

`skills/codex-brief-generator/SKILL.md` is the canonical skill file.

## Installation in Hermes

After adding this repository as a skill tap:

```bash
hermes skills tap add yanyintingyou/codex-brief-generator
hermes skills install yanyintingyou/codex-brief-generator/codex-brief-generator --force
```

Then start a new Hermes session or explicitly load the skill:

```text
/skill codex-brief-generator
```

## Example Usage

Ask Hermes:

```text
Use codex-brief-generator to create a copy-pasteable prompt for Windows Codex.
Task: clean the raw CSV files, build a panel dataset, add validation checks, and report row counts at each stage.
Working directory: C:\Users\me\projects\fomc-capital-flow
```

Hermes should return a structured `# Codex Task Brief` that can be pasted directly into Codex CLI.

## Why Not Let Hermes Run Codex Directly?

Direct Hermes-to-Codex execution is useful for short and well-bounded repository tasks. But for long-running work, local Windows applications, or large data pipelines, manual handoff can be safer and more controllable:

- The user can run Codex in the exact local environment.
- Codex can access local files and applications naturally.
- Hermes can focus on planning, research context, guardrails, and review.
- The task brief remains auditable and reusable.

## Compatibility

This repository follows the Agent Skills layout expected by Hermes:

```text
skills/<skill-name>/SKILL.md
```

The skill is written as a plain Markdown instruction file with YAML frontmatter, so it can also be adapted for other agent runtimes that support skill-style instruction packs.

## License

MIT License.
