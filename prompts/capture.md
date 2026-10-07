---
description: Capture an issue — file a new card on the board, with or without prior context
---

# Capture an Issue

You are capturing a new issue — a new continent that will be added to the Magrathea planet. The name fits both situations this is used in: deliberately filing a new card in a planning session, *and* grabbing something mid-conversation before it gets lost.

## Two Ways This Gets Invoked

You'll be invoked in one of two situations. Detect which before doing anything else.

**A — Cold invocation.** The user said `/capture` (with or without arguments) at the start of a conversation, with no other context in play. You'll need to ask what kind of session it is and what the issue is about.

**B — In-flight invocation.** The user was already mid-conversation — explaining a bug, sketching a feature, exploring an idea — and reached for `/capture` to file it before it gets lost. The description is **already on the table**. Asking "what does this issue need to accomplish?" would make them repeat themselves, which is what we're trying to fix.

**How to detect in-flight:**
- `$ARGUMENTS` contains a description ("add a dark mode toggle", "the cart bug we just talked about", etc.), OR
- The immediately preceding conversation clearly describes a concrete, scoped piece of work that the user just said should become an issue

When in doubt, treat the invocation as in-flight if there's *any* concrete material to work from. The cost of skipping a question the user already answered is high. The cost of confirming a generated title is low.

## 1. Set Mode

- **If in-flight (B):** assume `start working` mode. Skip the "multiple or start working?" question entirely. Do not ask it. Do not ask "what does this issue need to accomplish?" either — you already have the material.
- **If cold (A):** ask:

  > "Are you planning to create **multiple issues** (planning session), or do you want to **start working** on this one right away?"

  - **Multiple issues** — after each issue is created, ask "Describe the next issue, or say `done` to finish."
  - **Start working** — after creating the issue, automatically continue into the `/pickup` workflow (from Step 2: Load Context onward)

## 2. Gather Issue Details

- **If in-flight (B):** skip. The description is the `$ARGUMENTS` value and/or the prior conversation. Move to Step 3.
- **If cold (A):** ask: **What does this issue need to accomplish?** (description of the work, context, goals). Wait for their response.

## 3. Load Project Context

Before generating anything, read:

1. `magrathea/MEMORY.md` — project-specific conventions (if present)
2. `magrathea/core.md` — the molten core, cross-issue knowledge (if present)

This ensures the new issue title, slug, and initial state reflect established project conventions and avoids duplicating work already done.

## 4. Generate Title and Slug

Based on the description (whether asked-for or in-flight):
- Generate a **concise title** (3-5 words)
- Generate a **slug** using naming conventions:
  - Feature work: `feat-{feature-name}` (e.g., `feat-login-oauth`)
  - Bug fixes: `fix-{bug-description}` (e.g., `fix-cart-persist`)
  - Refactoring: `refactor-{component}` (e.g., `refactor-api-client`)
  - Investigations: `investigation-{topic}` (e.g., `investigation-performance`)
  - Standard issues: `issue-{number}` (e.g., `issue-42`)

Show the user the generated **title** and **slug**, and ask for confirmation: "Is this good, or should I adjust?"

Wait for their response and adjust if needed. **This confirmation is required even on in-flight invocations** — it's the only checkpoint where the user can correct your interpretation before a directory exists.

## 5. Determine Issue Number

- Scan `magrathea/` directory for existing directories matching the pattern `^\d{3}-` (e.g., `001-`, `002-`, etc.)
- Find the highest number N
- Assign the next number as (N+1), zero-padded to 3 digits (e.g., if highest is `006-`, use `007-`)
- Construct full issue directory name: `{NNN}-{slug}` (e.g., `007-feat-new-feature`)
- Use this as `{ISSUE}`

## 6. Check the Board Index

Before creating anything, check that `magrathea/board.md` exists. If not, stop and tell the user:

```
magrathea/board.md is missing — the kanban index hasn't been built yet.
Run /board-redraw first, then re-run this command.
```

The board and the filesystem must stay in sync. We don't create issues we can't index.

## 7. Create Directory Structure

Create the following structure for `magrathea/{ISSUE}/`:

```
magrathea/{ISSUE}/
  planning/          (create with .gitkeep)
  research/          (create with .gitkeep)
  state.md           (create with initial content)
```

The new issue lands on the board in the `Todo` lane. Valid status values: `Todo`, `In Progress`, `In Review`, `Complete`, `Closed`.

Create `magrathea/{ISSUE}/state.md` with this content (replacing {ISSUE} with the full directory name, {TITLE} with the confirmed human-readable title, and {DATE} with today's date):

```markdown
# {TITLE}

**Issue**: {ISSUE}
**Started**: {DATE}
**Status**: Todo

## Current Focus

(Nothing yet)

## Tasks

(No tasks yet)

## Progress Log

- Session started
```

If `magrathea/core.md` does not exist, create it as an empty file — the molten core grows as issues complete.

## 8. Append to board.md

After the directory is created and `state.md` is written, append the new issue's row to the `## Todo` group in `magrathea/board.md`:

- The row format is: `| {NNN} | {Title} | {Started} | {NNN-slug} |`
- If the Todo group currently reads `(none)`, replace it with the table header and the new row.
- If the Todo group already has a table, insert the new row sorted by number ascending (new issues will usually go at the bottom since they have the highest number).

## 9. Confirm to User

Output a brief summary:

```
✓ Created issue: {ISSUE} (Todo)
```

Then branch:

- **In-flight mode (B):** the user was mid-thought — return them to the thread. Don't run the full "Load Context" summary. A short "Issue {ISSUE} captured. Want to keep going with what we were doing, or switch to working on it?" is enough. Let them choose.
- **Multiple issues mode (cold A, multiple):** ask "Describe the next issue, or say `done` to finish." Repeat from Step 2 for each additional issue. When done, list all created issues and remind the user to invoke `/pickup {number}` to start working.
- **Start working mode (cold A, single):** automatically continue as if the user invoked `/pickup {ISSUE}` — proceed from the Load Context step, loading `magrathea/core.md`, the new `state.md`, and outputting the context summary before asking "What would you like to do?"

---

**Tip**: Invoke `/pickup` anytime to resume work on any issue — `state.md` will be automatically loaded, and the card will move to `In Progress` if it was in `Todo`.
