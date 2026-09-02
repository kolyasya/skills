---
name: pr-creator
description: >
  Create clear, focused GitHub pull requests with structured descriptions, proper labels, and assignee. Use when preparing, drafting, or submitting a PR. Trigger on: "create pr", "make a pr", "draft pr", "open pr", "submit pr", "open pull request", "pr-creator".
---

# Pull Request Creator

Create clear, focused GitHub pull requests (PRs) that transfer context in 30–60 seconds. Enforce concise descriptions, verified testing notes, dynamic label assignment, and self-review.

## Core Principles

- **Speed to Context**: Reviewers must understand the problem, the solution, and verification steps before reading the diff.
- **Why First**: Explain intent and necessity before implementation details.
- **Mental Model over Diffs**: Describe architectural and behavioral changes, not line-by-line file edits.
- **Atomic Scope**: One logical purpose per PR. Keep changes small. If a change exceeds size thresholds (> 500 lines changed or > 10 modified files), justify the large scope. If a change includes unrelated refactoring or dependency bumps, recommend splitting it.
- **Self-Review Pass**: Catch leftover debug statements, unintentional file changes, and formatting errors before requesting review.

---

## Workflow

### Step 1: Analyze Changes & Context

1. **Check Current Branch and Status**:
   ```bash
   git status
   git branch --show-current
   ```
2. **Determine Target Base Branch**:
   - Default to `main` (or `staging` / repository-defined base branch).
   - If the target branch is not the default production branch (e.g., `staging`), reflect it in the PR title.
3. **Inspect Diff and Change Size**:
   ```bash
   git log -n 5 --oneline
   git diff origin/<base-branch>...HEAD --stat
   ```
   - Calculate total changed lines and modified files.
   - Exclude lockfiles (e.g., `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`) and build artifacts from this count.
   - Flag as a **Large PR** if the change exceeds **500 lines changed** or **10 modified files**.
4. **Identify UI Changes**:
   - Check if frontend assets, styles, or components were modified.
   - If UI changes exist, prompt or remind to attach screenshots or screen recordings.

---

### Step 2: Research Repository Labels, Assignee & Task Link

1. **Fetch Available Labels**:
   ```bash
   gh label list --limit 50
   ```
   *(Or use GitHub MCP if configured).*
2. **Select Matching Labels**:
   - Match existing repository labels (e.g., `bug`, `enhancement`, `documentation`, `area/auth`).
   - **Never create or modify repository labels.** If no label matches, proceed without adding one.
3. **Assign Current User**:
   - Set the current user as assignee by default (`--assignee "@me"` in `gh`).
4. **Resolve Task Link**:
   - Look for a Trello card URL or ticket key in the current session context (conversation, branch name, commit messages, or open editor files).
   - If not found, call the Trello MCP `get_my_cards` tool to retrieve recent cards assigned to the user, then match against branch name or session topic.
   - If MCP returns a clear match, record the card URL. If the match is ambiguous or MCP is unavailable, ask the user: *"Do you have a Trello card or task link for this PR?"* Accept a URL or `none`.
   - Store the resolved URL (or absence) for use in the PR body.

---

### Step 3: Draft PR Title and Description

Apply the **`info-style-writing`** skill to draft all PR text. If the skill is available in the environment, load and follow its rules strictly:

- **Facts over adjectives**: Replace evaluative terms (*"cleaner"*, *"faster"*, *"better"*) with concrete mechanisms and numbers (*"reduces query count from N to 1"*).
- **Cut garbage words**: Remove filler phrases (*"in order to"*, *"due to the fact that"*, *"it is important to note"*).
- **Active voice & strong verbs**: State directly what the system or user does.
- **One idea per sentence**: Split overloaded explanations.
- **Reader value**: Focus strictly on what the reviewer needs to understand the change.

#### 1. PR Title Rules
- Use imperative mood: *"Add pagination to user activity list"* (not *"Added pagination"* or *"Pagination changes"*).
- Stand alone without opening the PR.
- If target branch is not the main/default branch (e.g. `staging`), append the target: `Add user activity pagination (staging)`.
- If an issue or ticket key exists, prefix or reference standard identifiers (e.g., `[JIRA-123] Add user activity pagination`).

#### 2. PR Body Template

```markdown
## Why
<!-- 1–3 sentences: What problem does this solve? Why is this change necessary now? -->

## What Changed
<!-- High-level mental model of the solution and key behavioral changes. -->

<!-- Optional: Small diagram/schema if it clarifies architecture or data flow. Avoid large, complex schemas. -->

## Why This PR Is Large
<!-- Required only if > 500 lines changed or > 10 files modified (excluding lockfiles and build artifacts). Explain why this change could not be split. -->

## Testing
<!-- Specific manual verification steps or automated test coverage. -->

## Review Focus
<!-- Optional: Call out high-risk paths, database migrations, or non-obvious trade-offs. Omit if not applicable. -->

## Task
<!-- Link to the originating card or ticket. Example: https://trello.com/c/abc123 — omit this section if no task link was resolved. -->
```

#### 3. Section Guidelines

- **Why**: 1–3 sentences stating the root problem and justification. Do not copy-paste full ticket requirements.
- **What Changed**: Summarize system behavior changes. Focus on architecture, data flow, and user impact.
- **Diagram / Schema (Optional)**: Include a short Mermaid diagram or ASCII flow **only** if it briefly explains a non-obvious state transition or architecture flow. Never include massive schemas.
- **Why This PR Is Large**:
  - Include this section **only** if real changes exceed 500 lines or 10 files. Omit for standard PRs.
  - Explain why the PR cannot be split into smaller units (e.g., mechanical refactor, broad migration, or tightly coupled components).
  - **Clarification Rule**: If the reason is clear from the session context or git log, draft it directly. If the reason is not known, ask the user directly in chat before generating the description.
- **Testing**:
  - State concrete test cases (e.g., *"Added unit tests for token expiration. Manually tested edge cases with 0, 1, and 100 items."*).
  - **If testing is unclear from context/discussion**: Omit the `## Testing` section entirely and notify the user in the final summary.
- **Review Focus**: Include only when specific risks exist (e.g., database migrations, heavy queries, critical security paths).
- **Task**: Include when a task link was resolved in Step 2. Omit when no link was found and the user confirmed `none`.

#### 4. Completion Criterion
All drafted text must pass the `info-style-writing` self-check (reader value first, active voice, zero fluff, verified facts) before presenting or submitting.

---

### Step 4: Self-Review & PR Submission

1. **Verify Uncommitted & Stray Changes**:
   - Ensure debug logs (`console.log`), temporary files, or unrelated config edits are removed.
2. **Determine PR Readiness**:
   - **Approved / Explicit Request**: If the user explicitly asked to create the PR, prepare standard creation.
   - **Unsure Context**: If uncertain whether the change is ready or approved, create as a **draft PR** (`--draft`).
3. **Execute PR Creation via `gh` CLI**:
   ```bash
   gh pr create \
     --base "<base-branch>" \
     --title "<PR Title>" \
     --body "<PR Body>" \
     --assignee "@me" \
     --label "<label1>,<label2>" \
     [--draft]
   ```
4. **Summary & User Notification**:
   - Output the PR URL.
   - If created as a `draft`, inform the user that a draft PR is open and needs review before marking ready.
   - If the `Testing` section was omitted due to missing context, notify the user to add testing notes.
   - If UI changes were detected, remind the user to drag-and-drop a visual preview into the PR description.
