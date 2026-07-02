---
name: spec
description: Use this skill to draft a written specification (design doc, RFC, or feature spec) BEFORE any code is written. The spec phase is strictly read-only — research the codebase freely, but the only file the agent may write or edit is the spec markdown itself. No source code, no config, no other artifacts. The skill presents 2–3 design approaches in conversation for the user to choose between, then writes a draft to docs/specs/YYYY-MM-DD-<slug>.md and asks the user to mark it up with inline `////` comments, after which the agent re-reads and integrates the comments. Once the user signs off, a fresh-eyes pass surfaces any high-level design decisions worth a second look — clarity, completeness, right-sizing, UX, and reversibility — and quietly fixes low-level defects, before the skill ends. This skill is primarily user-invoked via /spec — only auto-trigger on unambiguous explicit requests like "write a spec for X", "draft a design doc for Y", or "write up an RFC". Do NOT auto-trigger on general exploratory phrasing ("let's plan X", "think through Y", "before we start coding") — the user prefers to invoke this skill explicitly when they want it.
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

Seven phases. Don't skip them — even the fast ones serve a purpose.

### 1. Understand the request

Read the user's prompt carefully. The headline question to answer before doing anything else:

> **What is the smallest interesting version of this feature?**

If you can't name it in one sentence after the user's first message, the request isn't ready yet — ask 3–5 focused questions until you can. Scope creep is the dominant failure mode in spec-writing, and the smallest-interesting-version question is the main defense against it. A spec that ships a smaller feature than the user originally described is usually better than one that captures everything they said.

Also pin down: what is *explicitly out of scope*? The Non-goals section starts forming here.

### 2. Research the codebase

Read the relevant existing code, docs, and configuration. Ground the spec in what's actually there — file paths, function names, current data model, current invariants — so the proposal slots into reality rather than floating in space.

If the project has a `CLAUDE.md` or similar conventions file, read it. If `docs/CONCEPT.md`, `docs/ARCHITECTURE.md`, or other top-level docs exist, read the relevant ones. Spawn parallel reads when there are several independent files to look at.

Pay particular attention to **existing patterns the new feature could reuse**. The most valuable thing the spec can say is "this reuses X" — the second most valuable is "this deliberately diverges from X because Y." Both depend on knowing what X is.

### 3. Offer options upfront, in conversation

This is the most important step. Before writing anything to disk, present 2–3 candidate approaches in chat. Each one needs:

- A one-line name (e.g., "Approach A: server-authoritative, client speculatively renders").
- The shape of the change in 2–4 sentences.
- The main tradeoff vs the others — performance, complexity, future flexibility, blast radius, etc.
- A recommendation, with reasoning.

Then ask the user which to proceed with. The user may pick one wholesale, mix elements, or send you back for a different option set. **Don't draft the full spec until they've chosen.**

A real fork diverges on the load-bearing decision (server-authoritative vs client-authoritative, single-table vs separate tables). A fake fork diverges on naming or syntax — those waste the user's attention. If the only honest options share 90% of the design, say so and confirm before drafting.

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

#### Length calibration

Most specs are **50–200 lines**. Past 300 lines, you're either tackling something genuinely large or you're padding — and the model's bias is strongly toward padding. A reviewer who can't hold the whole spec in their head can't catch architectural problems in it. Resist the urge to include every type definition, every route signature, every migration step. The spec is the *thinking*, not the *implementation*.

If you find yourself approaching 300 lines, ask: is this one spec, or three? Splitting a large feature into multiple smaller specs is usually right.

#### Template

Use this as a starting skeleton. Adapt section names and depth — don't pad with empty sections, and add new ones only when they earn their keep.

```markdown
# <Feature name>

<One-paragraph summary: what this feature is, why it exists, and the
chosen approach in one breath.>

## Key decisions

<The load-bearing choices a reviewer should check before approving.
This is the section a code reviewer reads first — surface them here,
don't bury them in Design. Cover both kinds:

- **Code-shape choices** — which existing pattern to reuse, where a
  check lives, which abstraction owns a piece of state.
- **Tech-stack choices** — which package or library to add, which
  dependency to drop, which framework / runtime / build tool to
  pull in. A wrong dependency is just as load-bearing as a wrong
  pattern.

Each entry is one short bullet tagged with how it relates to the
existing codebase:

- `(reuses)`   — adopts an existing pattern as-is
- `(extends)`  — reuses with deliberate additions or modifications
- `(new)`      — introduces a pattern not previously in the codebase
- `(diverges)` — goes against an existing convention, deliberately
- `(breaking)` — changes a contract other code depends on

Aim for 4–8 entries. More than 10 usually means you're listing
implementation details instead of decisions.>

- **<Short decision name>** (tag). <One- or two-sentence description
  of what was decided and why, especially the relationship to
  existing code.>

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

- **Q:** <Specific, narrow, answerable question.> **Default:** <Your
  best-guess answer. Stating a default lets the user accept by
  silence rather than having to type a reply for every question.>

## Alternatives considered

<Only if approaches were discussed and rejected. Two or three lines
each is plenty — the point is not to re-litigate, it's to record why
this path was taken.>
```

#### Why "Key decisions" matters

A reviewer's job is to catch architectural mistakes — the spec inventing a new pattern when an existing one would do, or diverging from a convention without good reason. With the load-bearing choices buried in Design, that work means reading the whole spec. With them surfaced at the top, tagged by their relationship to the existing codebase, a reviewer can scan the `(new)` and `(diverges)` bullets in under a minute and spend the rest of their attention on the genuinely novel parts.

This is the section that does the most work per line of any in the spec. Treat it as required, not optional.

#### Why Open Questions take defaults

Stating `Default: X` after every question lets the user accept by silence. Without defaults, every question is a blocker — the user has to type something on each one before the spec moves forward. With defaults, the spec is shippable as-is, and the user only weighs in on the questions where the default is wrong. The default also does double duty in phase 5: it becomes the pre-selected `(Recommended)` option when the questions are put to the user via AskUserQuestion, so accepting it is one click and overriding it is one pick.

#### A worked example (right-sized)

A complete spec for a small feature, demonstrating the shape and density that "right-sized" means in practice. ~55 lines, all sections present, none padded.

````markdown
# Skip inactive users in the daily digest

Add a guard to the daily-digest job so users who haven't logged in
for 30 days don't receive the email. The cutoff is checked at send
time against `users.lastLoginAt`; users below the threshold are
silently skipped.

## Key decisions

- **Where the check lives** (extends). Runs inside
  `DigestJob.shouldSend(user)` next to the existing unsubscribe
  check, not at the recipient-query level. Reason: keeps all
  "should this user receive a digest" logic in one place.
- **Threshold value** (new). 30 days, hardcoded as
  `INACTIVE_DAYS = 30` in `digest.ts`. Not a config — we don't
  expect tuning, and a constant is easier to grep.
- **Send-log behavior** (diverges). Skipped users get no row in
  `digest_sends`. Existing convention logs every attempted send;
  we diverge because "didn't try" and "tried and suppressed" are
  meaningfully different states.
- **`date-fns` dependency** (reuses). The 30-day arithmetic uses
  `differenceInDays` from `date-fns`, already a project dep.
  Considered raw `Date` math; rejected for readability.

## Goals

- Stop emailing users who have effectively churned.
- Have the skip be observable in metrics (daily count of
  inactive-skipped users).

## Non-goals

- Re-engagement campaigns for churned users.
- A configurable threshold per workspace.

## Design

`DigestJob.run()` already iterates eligible users and calls
`shouldSend(user)`. Add a guard at the top of `shouldSend`:

```ts
const inactive =
  user.lastLoginAt == null ||
  differenceInDays(Date.now(), user.lastLoginAt) > INACTIVE_DAYS;
if (inactive) {
  metrics.inc("digest.skipped_inactive");
  return false;
}
```

Users with `lastLoginAt = NULL` (admin-created, never logged in)
are treated as inactive and skipped.

## Open questions

- **Q:** Should re-activation flip eligibility same-day, or wait
  for the next daily run? **Default:** wait for the next run — the
  user already gets a login-confirmation email, so a same-day
  digest would be noise.

## Alternatives considered

- **Filter at the recipient query** (`WHERE lastLoginAt > ...`).
  Cheaper but scatters the eligibility logic across the query and
  `shouldSend`. Rejected — one place beats two.
- **Soft-delete inactive users.** Out of scope; we may still need
  to email them for account purposes (password expiry, etc).
````

Note what this example *doesn't* have: no Implementation Order section, no Testing section, no Migration section, no Deployment section. Those belong in the build phase — they're not design decisions.

### 5. Hand off for inline review

Once the draft is on disk, send one handoff message in two parts: a short recap of what the spec proposes, then how to mark it up. The recap is the substance of the message; the markup instructions are a brief procedural close. So the recap is the last thing the user reads about the *proposal itself* before they go mark up the file — it lets them sanity-check the design without opening it and see which parts deserve scrutiny.

**Lead the recap with "The short version of what it proposes:"** Its main content is the spec's **Key decisions** section rendered as *consequences* — the same load-bearing choices, restated as what changes and what to watch rather than as tagged decisions. Keep it to 3–5 bullets, not a table of contents. Aim for this shape:

- The load-bearing change — the single source of truth or the central move, and what cascades from it.
- The concrete edits or new pieces, with real file paths and identifiers.
- What deliberately *stays the same* — the non-change a reviewer might otherwise worry about.
- Any downstream consequence worth watching (a perf, throughput, or blast-radius note).

If you can't compress the spec into 3–5 bullets like these, the Key decisions section is usually the culprit — either missing the load-bearing choice or padded with implementation detail. So the recap doubles as a cheap check on that section.

Then, in one line, note that Open Questions remain and that you'll put each to the user next with its default pre-selected — so accepting is a click, not homework. A worked example of the whole recap:

> *"The short version of what it proposes:*
> - *One source of truth — bump `WELLS_PER_STRIP` and `SAMPLES_PER_STRIP` 8→12; scheduler durations, queue/tray math, and the operator-card pills all cascade.*
> - *Two manual edits — the lone hardcoded `repeat(8, …)` at `continuous.css:1082`, and extending the SVG strip template `#STICK_1` with `HOLE_9..12` (concrete coords in the spec) plus a longer body rect.*
> - *No structural scheduler change — it's parameterized; the real consequence is longer loading (80→120 s) and reagent-add (20→30 s) steps, flagged as a throughput thing to watch.*
> - *Tray stays 24 samples → now 2 strips/tray instead of 3.*
>
> *Two open questions remain (the longer arm-work; tray = 2 strips) — I'll put each to you next with its default pre-selected, so a click accepts or you can override."*

**Then the markup instructions:**

> *"Draft is at `docs/specs/<file>.md`. If you want changes, add inline comments anywhere in the file by prefixing a line with `////` — e.g., `//// this section should also cover the migration case`. You can also rewrite or restructure as you like. **Reply when you've added your comments, or just tell me you're happy with the draft as-is.**"*

**If the spec has Open Questions, drive them with the AskUserQuestion tool — don't leave them to be noticed in the file.** Right after the handoff message, ask them: one AskUserQuestion call, one question per Open Question, with the spec's `Default: X` as the first option tagged `(Recommended)` and the realistic alternatives as the other options. The user accepts a default in one click, picks an alternative, or types their own via Other. This is the defaults philosophy made literal — silence becomes a click, but the question is surfaced instead of buried. The picker is also how you "stop and wait" here: you're waiting on their answers.

- AskUserQuestion takes at most four questions per call. More than four open questions at handoff is usually a smell — ask the four that most affect the design and fold the rest into the next pass.
- A question with no real alternative to its default isn't open. Resolve it in the draft instead of asking.
- The picker doesn't stop the user from also marking up the file. When their answers come back, re-read the spec for `////` comments too, and integrate both.

**If the spec has no Open Questions, skip the picker and just stop and wait** for the user's reply. Either way, don't keep working on the spec until the user signals back — pestering them or pre-emptively revising defeats the handoff.

When the user answers the AskUserQuestion prompt, or signals they've added comments (typical phrasings: "I've added comments", "take another pass", "look at the file again"):

1. Re-read the spec from disk.
2. Find every line beginning with `////`. Treat each as user feedback attached to its surrounding context. Integrate the feedback into the relevant section, then **remove the `////` line itself** — the marker is review scaffolding, not part of the final document.
3. Integrate their Open Question answers — from the AskUserQuestion picker, or answered inline (replacing the default, rewriting the question, or deleting it outright) — into the design, and remove the resolved questions.
4. **If the user has rewritten or restructured parts of the document themselves, respect their edits.** Their words are the source of truth — your job is to make the rest of the document consistent with what they wrote, not to revert it to your version. This is the single most important rule of the integration pass.
5. Surface any new questions the changes expose, either as fresh Open Questions in the document or in chat.

If the user instead says they're happy with the draft as-is, skip the integration pass and move to phase 6.

### 6. Iterate to signed-off

Loop on phases 4 and 5 until the user says the spec is good — the spec file exists, the user is satisfied, and the Open Questions section is empty (or all questions have been answered). Once they've signed off, run one final review before ending (phase 7).

### 7. Fresh-eyes review

The spec is signed off — but the person who just wrote it is the worst-placed to judge it. By the time you've drafted and revised a spec, every ambiguity in it has already been resolved *in your head*; you read the words and see what you meant, not what they actually say. So before the skill ends, hand the finished spec to a reader who wasn't in the room.

Spawn a subagent as a fresh reviewer and give it **only the spec file path** — not this conversation, not the approach discussion, not the option set you narrowed down in phase 3. That blindness is the whole point: a reviewer carrying your context inherits your interpretation and trips on nothing. Let it read the referenced code and docs read-only — it needs them to judge whether the design reuses what's there or reinvents it — but keep the conversation out of its head.

The reviewer looks for six kinds of problem — all high-level, design-level concerns a fresh reader is uniquely placed to catch, and exactly the kind of thing the user of this skill should get a say in. They fall into three pairs: is it **clear** (ambiguity, completeness), is it **right-sized** (unnecessary complexity, goal-fit), and can you **live with it** (bad UX, reversibility).

- **Ambiguity** — a load-bearing part of the design that a competent implementer could reasonably read two ways and build differently. The test: *would two engineers implement this and both believe they followed the spec, yet ship incompatible things?* "This sentence could be tighter" is a nit; "it's unspecified whether the check runs before or after the write, and the data model differs depending" clears the bar.
- **Completeness** — the design covers the happy path but leaves a load-bearing case unsaid: failure, empty or malformed input, concurrency, or the existing data and clients at rollout. Where ambiguity is something said two ways, this is something not said at all — and the cold reader is the one most likely to notice, because the author already filled the gap in their head. The test: *is there a case a builder will hit and have to guess at, where guessing wrong changes the result?* Let the trivial omissions go; flag the ones that force a blind decision.
- **Unnecessary complexity** — machinery the spec introduces that isn't earning its keep: a new table, service, abstraction, or dependency where something simpler or already-present would do, or scope that has crept past the smallest interesting version (phase 1). The test: *can you name a materially simpler design that still meets the stated Goals?* If yes, that's the finding.
- **Goal-fit** — the design doesn't actually deliver a stated Goal, or spends effort on things no Goal asked for. The test: *walk each Goal and point at the part of the Design that satisfies it, then walk the Design and check each piece traces back to a Goal.* A Goal with nothing behind it is a gap; a slab of Design with no Goal behind it is either a missing Goal or scope to cut (which is really the complexity finding wearing a different hat).
- **Bad UX** — the experience of whoever *uses the thing being specced* (end user or operator) would be confusing, surprising, or unrecoverable as designed: a destructive action with no confirmation, a silent failure, an error with no path forward, a default that will surprise most users. This is the UX of the feature — not the UX of using this skill.
- **Reversibility** — the design commits to something expensive or impossible to undo: an irreversible migration, a breaking API or schema change, deleting data, a dependency that's hard to back out. This lens matters most here, at spec-time, because a one-way door costs almost nothing to reconsider now and a great deal once the branch exists. The test: *if this proves wrong after it ships, how hard is it to walk back?* The point isn't to veto it — it's to be sure the user chose the door knowingly, so it's always a decision, never a silent fix.

The reviewer only reads — it returns what it found, and any edits to the spec are yours to make. Handle each finding by one question: **is this a decision, or a fix?**

- **Decisions go to the user.** If resolving it means choosing between plausible alternatives — which of two designs was intended, whether scope has crept too far, which side of a UX tradeoff to take — a human has to make that call. These are the high-level items the review exists to surface. Bring them back as a short list: the category, the spot in the spec, what's at stake, and the options. Hold to critical/high here — the user just signed off, and interrupting that is earned only by something they'd genuinely want to decide before walking away; a pile of things to adjudicate trains them to skip the review. If nothing rises to that bar, say so in one line. Then let *them* choose what changes — reworking an approved design behind their back takes the decision out of their hands. When they pick something, loop back through phase 4 (revise) and phase 5 (re-confirm) for just those points.
- **Fixes just get made.** If the problem has one obviously-correct resolution and no real tradeoff — a snippet that contradicts a stated decision, a missing null guard, two sections that disagree, an off-by-one in an example — there's nothing for the user to decide. Correct it directly in the spec (the spec file is the one you're allowed to edit), then note what you tidied in a line or two so nothing changes silently. These fixes make the spec match the design the user already approved; they don't re-open it. Forcing the user to adjudicate a mechanical correction wastes the very attention the review is meant to protect.

The line between the two is simply whether more than one resolution is plausible. If a "fix" turns out to need a judgment call, it was never a fix — treat it as a decision and surface it. Either way, don't re-run the full review after every touch-up — it's a final gate, not a treadmill; a quick check that the flagged items are resolved is enough.

If subagents aren't available (e.g., Claude.ai), do the pass inline instead: re-read the spec cold from disk and apply the same split — surface the decisions, fix the mechanical defects. It's weaker — you can't truly un-know the conversation — but reading the written words fresh still catches more than skipping the pass.

Then **stop**. Do not begin implementing. The handoff to coding is a separate decision the user makes — they may want to sit on the spec, share it, schedule the work, or use a different session for the build phase. Offering "want me to start on it?" at the very end is fine; jumping straight in is not.

## Style guidelines for the spec itself

- **Concrete over abstract.** "Adds a `carrierLimit: number` field on `Path`, default 1" beats "introduces capacity tracking on paths."
- **Reference real names.** File paths, function names, types — they make the spec auditable against the code.
- **Explain *why*, not just *what*.** A reader six months from now needs to know the reasoning, not just the rule. The why is the part the code alone can't capture.
- **Right-size.** See the length calibration above. Don't include a section heading if there's nothing meaningful under it.
- **Match the project's existing voice.** If the project's docs are narrative prose, write narrative prose. If they're bullet-heavy, lean bulletier. Skim a sibling doc before writing.

## Common mistakes to avoid

- **Drafting before the user has picked an approach.** The whole point of phase 3 is to fork the conversation early. Writing the spec first and asking later wastes a draft and biases the discussion.
- **Manufacturing options that aren't real.** If there's only one reasonable approach, say so. A weak Option B insults the user's time.
- **Editing source code "to verify."** If you need to confirm something works, write it as an Open Question and let the user check. Read-only means read-only.
- **Treating implementation logistics as design.** Implementation order, test matrices, deployment runbooks, NuGet / dependency change lists — these belong in the build phase, not the spec. If you find yourself writing a "Testing" or "Implementation order" section, ask whether it's making any genuine *design* decision. If not, cut it.
- **Burying the architectural choices in Design.** A reviewer should be able to find the load-bearing decisions in the Key Decisions section without reading the rest. If the `(new)` and `(diverges)` bullets aren't there, you've made the reviewer's job much harder than it needs to be.
- **Skipping the Open Questions section.** It's the cheapest way to surface "I'm not sure about this" without blocking the draft. Use it generously, with `Default: X` for each entry.
- **Leaving Open Questions unresolved at sign-off.** The spec is "done" only when the section is empty (or removed entirely).
- **Bringing the user things that aren't decisions.** Phase 7 earns its keep by only surfacing genuine, critical/high design decisions — the calls a human actually needs to make. Mechanical defects with one right answer get fixed in the spec, not raised. Dumping fixable nits on someone who just signed off teaches them to ignore the review; if there's nothing to decide, say so in one line and stop.
- **Getting the decide/fix split backwards.** The two failure modes are opposite: making the user adjudicate a null-guard-grade fix, or quietly reworking the *design* under the cover of a "fix." The test is whether more than one resolution is plausible — if it is, it's a decision and belongs to the user.
- **Starting to implement after sign-off.** The skill ends at the signed-off, reviewed spec. Implementation is a separate, deliberate decision by the user.
