---
name: pr-creator
description: Create clear, focused GitHub pull requests with structured descriptions, proper labels, and assignee. Use when preparing, drafting, or submitting a PR. Trigger on: "create pr", "make a pr", "draft pr", "open pr", "submit pr", "open pull request", "pr-creator".
---

# Pull Request Creator

Create clear, focused GitHub pull requests (PRs) that transfer context in 30–60 seconds. Enforce concise descriptions, verified testing notes, dynamic label assignment, and self-review.

## Core Principles

- **Speed to Context**: Reviewers must understand the problem, the solution, and verification steps before reading the diff.
- **Why First**: Explain intent and necessity before implementation details.
- **Mental Model over Diffs**: Describe architectural and behavioral changes, not line-by-line file edits.
- **Atomic Scope**: One logical purpose per PR. If a change includes unrelated refactoring or dependency bumps, recommend splitting it.
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
3. **Inspect Diff and Commit History**:
   ```bash
   git log -n 5 --oneline
   git diff origin/<base-branch>...HEAD --stat
   ```
4. **Identify UI Changes**:
   - Check if frontend assets, styles, or components were modified.
   - If UI changes exist, prompt or remind to attach screenshots or screen recordings.

---

### Step 2: Research Repository Labels & Assignee

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

---

### Step 3: Draft PR Title and Description

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

## Testing
<!-- Specific manual verification steps or automated test coverage. -->

## Review Focus
<!-- Optional: Call out high-risk paths, database migrations, or non-obvious trade-offs. Omit if not applicable. -->
```

#### 3. Section Guidelines

- **Why**: 1–3 sentences stating the root problem and justification. Do not copy-paste full ticket requirements.
- **What Changed**: Summarize system behavior changes. Focus on architecture, data flow, and user impact.
- **Diagram / Schema (Optional)**: Include a short Mermaid diagram or ASCII flow **only** if it briefly explains a non-obvious state transition or architecture flow. Never include massive schemas.
- **Testing**:
  - State concrete test cases (e.g., *"Added unit tests for token expiration. Manually tested edge cases with 0, 1, and 100 items."*).
  - **If testing is unclear from context/discussion**: Omit the `## Testing` section entirely and notify the user in the final summary.
- **Review Focus**: Include only when specific risks exist (e.g., database migrations, heavy queries, critical security paths).

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
