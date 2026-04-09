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

**DO NOT START WITHOUT USER APPROVAL.**

**Actions**:
1. Wait for explicit user approval.
2. Launch one or more code-builder agents. Each agent:
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

## Phase 6: Quality Review

**Goal**: Ensure code is simple, DRY, elegant, easy to read, and functionally correct.

**Actions**:
1. Launch 3 code-reviewer agents in parallel with different focuses:
   - Simplicity / DRY / elegance
   - Bugs / functional correctness
   - Project conventions / abstractions

   **Each reviewer should read `.claude/handoff/ARCHITECTURE.md`** to understand the intended design, so they can review against intent, not just style.

2. Consolidate findings and identify highest severity issues that you recommend fixing.
3. Write `.claude/handoff/REVIEW.md` with:
   - All findings organized by severity
   - Recommended fixes
   - Issues deferred or accepted
4. **Present findings to user and ask what they want to do** (fix now, fix later, or proceed as-is).
5. Address issues based on user decision (launch code-builder agents if fixes are needed).

---

## Phase 7: Summary

**Goal**: Document what was accomplished.

**Actions**:
1. Mark all todos complete.
2. Write `.claude/handoff/SUMMARY.md` with:
   - What was built
   - Key decisions made
   - Files modified
   - Suggested next steps
3. Present the summary to the user.
