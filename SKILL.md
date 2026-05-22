---
name: spec
description: Use this skill to draft a written specification (design doc, RFC, or feature spec) BEFORE any code is written. The spec phase is strictly read-only — research the codebase freely, but the only file the agent may write or edit is the spec markdown itself. No source code, no config, no other artifacts. The skill presents 2–3 design approaches in conversation for the user to choose between, then writes a draft to docs/specs/YYYY-MM-DD-<slug>.md and asks the user to mark it up with inline `////` comments, after which the agent re-reads and integrates the comments. This skill is primarily user-invoked via /spec — only auto-trigger on unambiguous explicit requests like "write a spec for X", "draft a design doc for Y", or "write up an RFC". Do NOT auto-trigger on general exploratory phrasing ("let's plan X", "think through Y", "before we start coding") — the user prefers to invoke this skill explicitly when they want it.
---

# Spec

This skill turns an exploratory feature idea into a written specification the user can review, edit, and approve *before* any code changes happen. It is the thinking phase, captured on disk.

## Why a spec phase exists

Writing the design down before coding does three things at once:

- It forces the choices to be made consciously rather than discovered mid-implementation.
- It produces a reviewable artifact — prose is much faster to read than a diff.
- It leaves a permanent record of *why* the system is shaped the way it is, which code alone can't capture.

The point is not bureaucracy. It's that revising prose is cheaper than revising a half-finished branch.

## Hard rule: read-only

During the spec phase, the only file the agent may create or modify is the spec file itself (the markdown under `docs/specs/`).

- Read, Grep, Glob, find, git log/blame/diff, ls, cat — all fine.
- Any read-only Bash command (npm view, package inspection, fetch, `date`, etc.) — fine.
- Creating, editing, or overwriting the one spec file under `docs/specs/` — fine.
- Editing, creating, or deleting any other file — not allowed. No source code, no config, no tests, no scratch files.
- Commands with side effects (npm install, git commit, code generators, migrations, anything that mutates state) — not allowed.

If a question can only be answered by trying something, write the question into the spec's Open Questions section and let the user run it. Don't violate the rule to "just check."

## The workflow

Six phases. Don't skip them — even the fast ones serve a purpose.

### 1. Understand the request

Read the user's prompt carefully. If the scope is unclear, ask 3–5 focused questions before doing anything else. Aim to leave the conversation knowing: *what is the smallest interesting version of this feature, and what is explicitly out of scope?*

### 2. Research the codebase

Read the relevant existing code, docs, and configuration. Ground the spec in what's actually there — file paths, function names, current data model, current invariants — so the proposal slots into reality rather than floating in space.

If the project has a `CLAUDE.md` or similar conventions file, read it. If `docs/CONCEPT.md`, `docs/ARCHITECTURE.md`, or other top-level docs exist, read the relevant ones. Spawn parallel reads when there are several independent files to look at.

### 3. Offer options upfront, in conversation

This is the most important step. Before writing anything to disk, present 2–3 candidate approaches in chat. Each one needs:

- A one-line name (e.g., "Approach A: server-authoritative, client speculatively renders").
- The shape of the change in 2–4 sentences.
- The main tradeoff vs the others — performance, complexity, future flexibility, blast radius, etc.
- A recommendation, with reasoning.

Then ask the user which to proceed with. The user may pick one wholesale, mix elements, or send you back for a different option set. **Don't draft the full spec until they've chosen.**

If the request is so well-defined that there really is only one reasonable approach, say so plainly, justify it in a sentence or two, and confirm before drafting. Don't manufacture fake options to pad a list — a weak alternative wastes the user's attention.

### 4. Draft the spec

Once an approach is chosen, write the draft to:

```
<repo-root>/docs/specs/YYYY-MM-DD-<slug>.md
```

- `YYYY-MM-DD` is today's date. Use `date +%Y-%m-%d` if unsure; the system date in your context is also fine.
- `<slug>` is short, kebab-case, descriptive: `treasure-loot-tables`, `path-capacity-upgrades`, `session-rotation`. Aim for 2–4 words.
- Create `docs/specs/` if it doesn't exist.
- If the project's existing conventions clearly call for a different location (e.g., a `CLAUDE.md` says specs live in `requirements/`, or the user explicitly names a path), follow that instead.

Use this template as a starting skeleton. Adapt the section names and depth to fit the spec at hand — don't pad with empty sections, and don't be afraid to add new ones (data model, migration, UI, edge cases) when they earn their keep.

```markdown
# <Feature name>

<One-paragraph summary: what this feature is, why it exists, and the
chosen approach in one breath.>

## Goals

- <What this is trying to accomplish>

## Non-goals

- <What it explicitly is not trying to do — bounds the discussion>

## Design

<The meat. Structure is flexible. Use whatever subsections make the
design legible. Reference real file paths and identifiers where it
helps. Code snippets are welcome when they pin down a tricky bit, but
the spec is prose, not an implementation.>

## Open questions

- **Q:** <Specific, narrow, answerable question.>

  *(Leave a blank line for the user to type the answer in.)*

## Alternatives considered

<Only if approaches were discussed and rejected. Two or three lines
each is plenty — the point is not to re-litigate, it's to record why
this path was taken.>
```

### 5. Hand off for inline review

Once the draft is on disk, tell the user it's ready and explain how to mark it up. A short, concrete prompt works best — something like:

> *"Draft is at `docs/specs/<file>.md`. If you want changes, add inline comments anywhere in the file by prefixing a line with `////` — e.g., `//// this section should also cover the migration case`. You can also answer any Open Questions inline, or rewrite/restructure as you like. **Reply when you've added your comments, or just tell me you're happy with the draft as-is.**"*

Then **stop and wait**. Don't keep working on the spec until the user signals back. The whole point of the handoff is that the user is now driving — pestering them or pre-emptively revising defeats it.

When the user signals they've added comments (typical phrasings: "I've added comments", "take another pass", "look at the file again"):

1. Re-read the spec from disk.
2. Find every line beginning with `////`. Treat each as user feedback attached to its surrounding context.
3. Integrate the feedback into the relevant section, then **remove the `////` line itself** — the marker is review scaffolding, not part of the final document.
4. If they answered any Open Questions inline (under the question, or by rewriting it), integrate those answers into the design and remove the resolved questions.
5. If the user has rewritten or restructured parts of the document themselves, **respect their edits**. Their words are the source of truth — your job is to make the rest of the document consistent with what they wrote, not to revert it to your version.
6. Surface any new questions the changes expose, either as fresh Open Questions in the document or in chat.

If the user instead says they're happy with the draft as-is, skip the integration pass and move to phase 6.

The `## Open Questions` section in the template is still useful for unresolved sub-decisions you want to flag for the user — defaults, naming, edge-case behavior, exact thresholds. Use it when you have specific questions; the `////` mechanism is for everything else the user wants to comment on.

### 6. Iterate to signed-off

Loop on phases 4 and 5 until the user says the spec is good. The skill is finished when the spec file exists, the user is satisfied, and the Open Questions section is empty (or removed).

Then **stop**. Do not begin implementing. The handoff to coding is a separate decision the user makes — they may want to sit on the spec, share it, schedule the work, or use a different session for the build phase. Offering "want me to start on it?" at the very end is fine; jumping straight in is not.

## Style guidelines for the spec itself

- **Concrete over abstract.** "Adds a `carrierLimit: number` field on `Path`, default 1" beats "introduces capacity tracking on paths."
- **Reference real names.** File paths, function names, types — they make the spec auditable against the code.
- **Explain *why*, not just *what*.** A reader six months from now needs to know the reasoning, not just the rule. The why is the part the code alone can't capture.
- **Right-size the spec.** A small spec is a paragraph and a list. A big spec is a few pages with subsections. Don't pad. Don't include a section heading if there's nothing under it.
- **Match the project's existing voice.** If the project's docs are narrative prose, write narrative prose. If they're bullet-heavy, lean bulletier. Skim a sibling doc before writing.

## Common mistakes to avoid

- **Drafting before the user has picked an approach.** The whole point of phase 3 is to fork the conversation early. Writing the spec first and asking later wastes a draft and biases the discussion.
- **Manufacturing options that aren't real.** If there's only one reasonable approach, say so. A weak Option B insults the user's time.
- **Editing source code "to verify."** If you need to confirm something works, write it as an Open Question and let the user check. Read-only means read-only.
- **Skipping the Open Questions section.** It's the cheapest way to surface "I'm not sure about this" without blocking the draft. If you have any doubts, list them.
- **Leaving Open Questions unresolved at sign-off.** The spec is "done" only when the section is empty (or removed entirely).
- **Starting to implement after sign-off.** The skill ends at the signed-off spec. Implementation is a separate, deliberate decision by the user.
