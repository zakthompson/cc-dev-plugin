---
description: Guided feature development with codebase understanding and architecture focus
argument-hint: Feature description
allowed-tools: Agent, Read, Write, Edit, Glob, Grep, Bash, TaskCreate, TaskUpdate, TaskList, TaskGet
---

# Feature Development

You are helping a developer implement a new feature. Follow a systematic approach: understand the codebase deeply, identify and ask about all underspecified details, design elegant architectures, then implement.

Every phase writes its output to a markdown handoff file in `.claude/handoff/`. This ensures any interrupt or context loss is recoverable.

## Core Principles

- **Ask clarifying questions**: Identify all ambiguities, edge cases, and underspecified behaviors. Ask specific, concrete questions rather than making assumptions. Wait for user answers before proceeding.
- **Understand before acting**: Read and comprehend existing code patterns first.
- **Read files identified by agents**: When launching agents, ask them to return lists of the most important files to read. After agents complete, read those files to build detailed context.
- **Simple and elegant**: Prioritize readable, maintainable, architecturally sound code.
- **Use TaskCreate/TaskUpdate**: Track all progress throughout.
- **Write handoff files**: Every phase writes its output to `.claude/handoff/` so work survives context loss.

## Recovery Protocol

**Before starting any work**, check if `.claude/handoff/` exists and contains files from a previous run:

1. Run `ls .claude/handoff/` to check for existing handoff files.
2. If handoff files exist, read them to understand prior progress.
3. Present a summary to the user: which phases completed, where work stopped.
4. Ask: "Would you like to resume from where we left off, or start fresh?"
5. If resuming, skip to the earliest incomplete phase using context from the handoff files.
6. If starting fresh, delete the existing handoff directory contents first.

---

## Phase 1: Discovery

**Goal**: Understand what needs to be built.

Initial request: $ARGUMENTS

**Actions**:
1. Create todo list with all 7 phases.
2. Create `.claude/handoff/` directory if it doesn't exist: `mkdir -p .claude/handoff`
3. If feature unclear, ask user for:
   - What problem are they solving?
   - What should the feature do?
   - Any constraints or requirements?
4. Summarize understanding and confirm with user.
5. Write `.claude/handoff/DISCOVERY.md` with:
   - Feature description
   - User requirements and constraints
   - Confirmed scope

---

## Phase 2: Codebase Exploration

**Goal**: Understand relevant existing code and patterns at both high and low levels.

**Actions**:
1. Launch 2-3 code-explorer agents in parallel. Each agent should:
   - Trace through the code comprehensively and focus on getting a comprehensive understanding of abstractions, architecture and flow of control
   - Target a different aspect of the codebase (similar features, high level understanding, architectural understanding, user experience, etc.)
   - Include a list of 5-10 key files to read
   - **Write findings to a structured format suitable for handoff**

   **Example agent prompts**:
   - "Find features similar to [feature] and trace through their implementation comprehensively. Return your findings as structured markdown."
   - "Map the architecture and abstractions for [feature area], tracing through the code comprehensively. Return your findings as structured markdown."
   - "Analyze the current implementation of [existing feature/area], tracing through the code comprehensively. Return your findings as structured markdown."

2. Once the agents return, read all files identified by agents to build deep understanding.
3. Present comprehensive summary of findings and patterns discovered.
4. Write `.claude/handoff/EXPLORATION.md` with:
   - Summary of all agent findings
   - Key files list (with brief description of why each matters)
   - Architecture patterns discovered
   - Relevant conventions and abstractions

---

## Phase 3: Clarifying Questions

**Goal**: Fill in gaps and resolve all ambiguities before designing.

**CRITICAL**: This is one of the most important phases. DO NOT SKIP.

**Actions**:
1. Review the codebase findings and original feature request.
2. Identify underspecified aspects: edge cases, error handling, integration points, scope boundaries, design preferences, backward compatibility, performance needs.
3. **Present all questions to the user in a clear, organized list.**
4. **Wait for answers before proceeding to architecture design.**
5. If the user says "whatever you think is best", provide your recommendation and get explicit confirmation.
6. Write `.claude/handoff/QUESTIONS.md` with:
   - Questions asked
   - User's answers
   - Resolved decisions

---

## Phase 4: Architecture Design

**Goal**: Design multiple implementation approaches with different trade-offs.

**Actions**:
1. Launch 2-3 code-architect agents in parallel with different focuses:
   - Minimal changes (smallest change, maximum reuse)
   - Clean architecture (maintainability, elegant abstractions)
   - Pragmatic balance (speed + quality)

   **Each agent should read `.claude/handoff/EXPLORATION.md` and `.claude/handoff/QUESTIONS.md`** to understand prior context without re-exploring the codebase.

2. Review all approaches and form your opinion on which fits best for this specific task (consider: small fix vs large feature, urgency, complexity, team context).
3. Present to user: brief summary of each approach, trade-offs comparison, **your recommendation with reasoning**, concrete implementation differences.
4. **Ask user which approach they prefer.**
5. Write `.claude/handoff/ARCHITECTURE.md` with:
   - Chosen approach and rationale
   - Complete implementation blueprint
   - Specific files to create/modify with detailed change descriptions
   - Component responsibilities and interfaces
   - Data flow from entry points through transformations to outputs
   - Phased build sequence as a checklist
   - Critical details: error handling, state management, testing, performance

   **This file is the primary directive for the code-builder agent.** Write it as a clear, actionable implementation plan that a developer could follow step-by-step.

---

## Phase 5: Implementation

**Goal**: Build the feature.

**The user's selection of an architecture approach in Phase 4 constitutes approval to proceed with implementation. Do not ask again whether to begin building.**

**Actions**:
1. Launch one or more code-builder agents. Each agent:
   - Reads `.claude/handoff/ARCHITECTURE.md` as its primary directive
   - Reads `.claude/handoff/EXPLORATION.md` for codebase context
   - Implements the specified changes following the build sequence
   - Follows codebase conventions strictly
3. As implementation progresses, write/update `.claude/handoff/IMPLEMENTATION.md` with:
   - Completed steps (checked off from the build sequence)
   - Files created or modified
   - Any deviations from the plan and why
   - Remaining work if interrupted

---

## Phase 6: Quality Review Loop

**Goal**: Iteratively review, plan fixes, and rebuild until the implementation meets quality standards.

This phase is a loop. It continues until reviewers have no remaining issues to report. The loop proceeds as: **Review → Architect response → Build fixes → Review again**.

### Loop Step 1: Review

1. Launch 3 code-reviewer agents in parallel with different focuses:
   - Simplicity / DRY / elegance
   - Bugs / functional correctness
   - Project conventions / abstractions

   **Each reviewer should read `.claude/handoff/ARCHITECTURE.md`** to understand the intended design, so they can review against intent, not just style.
   **Each reviewer should also read `.claude/handoff/REVIEW.md`** if it exists from a prior iteration, so they understand what issues have already been raised and addressed (including issues the architect explicitly justified not fixing -- reviewers must accept these justifications and not re-flag them).

2. Consolidate findings across all reviewers.
3. Write/update `.claude/handoff/REVIEW.md` with:
   - Iteration number
   - All new findings organized by severity
   - History of prior iterations' findings and resolutions

4. **If reviewers found no new issues**: The loop is complete. It is expected and acceptable for reviewers to find no issues -- this is the successful exit condition. Proceed to Phase 7.

5. **If issues were found**: Continue to Loop Step 2.

### Loop Step 2: Architect Response

1. Launch a code-architect agent to review the findings in `.claude/handoff/REVIEW.md`.
   - The architect reads REVIEW.md, ARCHITECTURE.md, and EXPLORATION.md for full context.
   - For each finding, the architect decides: **fix it** (with a concrete plan) or **justify not fixing it** (with a clear rationale).
   - The architect must not dismiss issues without justification. Every "won't fix" must include reasoning that a reviewer would accept.

2. Write `.claude/handoff/REVIEW_RESPONSE.md` with:
   - Iteration number
   - For each finding: action (fix or won't fix) and rationale
   - Implementation plan for fixes (files to modify, specific changes)
   - Justifications for any deferred/declined items

### Loop Step 3: Build Fixes

1. Launch one or more code-builder agents to implement the fixes specified in `REVIEW_RESPONSE.md`.
   - Builders read REVIEW_RESPONSE.md as their primary directive for this iteration.
   - Builders also read ARCHITECTURE.md and EXPLORATION.md for context.
2. Update `.claude/handoff/IMPLEMENTATION.md` with the fixes applied.

3. Return to **Loop Step 1** for the next review iteration.

---

## Phase 7: Summary

**Goal**: Document what was accomplished.

**Actions**:
1. Mark all todos complete.
2. Write `.claude/handoff/SUMMARY.md` with:
   - What was built
   - Key decisions made
   - Files modified
   - Review iterations: how many rounds, key issues found and resolved
   - Suggested next steps
3. Present the summary to the user.
4. Clean up handoff files: `rm -rf .claude/handoff/`
