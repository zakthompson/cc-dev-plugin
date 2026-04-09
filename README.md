# cc-dev-plugin

Feature development workflow for Claude Code, forked from the official `feature-dev` plugin with key changes:

- **Markdown handoff files**: Every phase writes output to `.claude/handoff/` so work survives context loss or interrupts
- **Opus for reasoning, Sonnet for execution**: Exploration, architecture, and review agents run on Opus; the code-builder agent runs on Sonnet following Opus-written plans
- **Recovery flow**: Detects existing handoff files and offers to resume from where work left off

## Usage

```
/dev <feature description>
```

## Workflow

| Phase | What happens | Handoff file |
|-------|-------------|--------------|
| 1. Discovery | Clarify requirements | `DISCOVERY.md` |
| 2. Exploration | Agents analyze codebase (Opus) | `EXPLORATION.md` |
| 3. Questions | Resolve ambiguities | `QUESTIONS.md` |
| 4. Architecture | Design approaches (Opus) | `ARCHITECTURE.md` |
| 5. Implementation | Build the feature (Sonnet) | `IMPLEMENTATION.md` |
| 6. Review | Quality check (Opus) | `REVIEW.md` |
| 7. Summary | Document results | `SUMMARY.md` |

All handoff files are written to `.claude/handoff/` (gitignored).

## Agents

| Agent | Model | Role |
|-------|-------|------|
| code-explorer | Opus | Traces codebase features and architecture |
| code-architect | Opus | Designs implementation blueprints |
| code-builder | Sonnet | Executes architect's plan |
| code-reviewer | Opus | Reviews for bugs, quality, conventions |

## Installation

```bash
claude --plugin-dir /path/to/cc-dev-plugin
```

Or add to your project's `.claude/plugins.json`.
