---
name: spec
description: Use this skill to draft a written specification (design doc, RFC, or feature spec) BEFORE any code is written. The spec phase is strictly read-only — research the codebase freely, but the only file the agent may write or edit is the spec markdown itself. No source code, no config, no other artifacts. The skill presents 2–3 design approaches in conversation for the user to choose between, then writes a draft to docs/specs/YYYY-MM-DD-<slug>.md and asks the user to mark it up with inline `////` comments, after which the agent re-reads and integrates the comments. Sign-off is an explicit gate: the agent asks for it directly and never infers it from answered questions or partial praise. Every draft carries the spec's Outcome section (short bullets: what you get, and how to verify it) and an Implementation strategy section (single agent, multi-agent, or ultracode, plus how many agents and which model, with a one-clause reason for each). Both are kept current on every pass, so the user signs off on them too. When sign-off is asked, a fresh-eyes review starts in the background on that exact text. It surfaces high-level design decisions worth a second look (clarity, completeness, right-sizing, UX, reversibility), quietly fixes low-level defects, and notes rather than re-asks anything the user already settled. The skill stops once that review is handled. This skill is primarily user-invoked via /spec — only auto-trigger on unambiguous explicit requests like "write a spec for X", "draft a design doc for Y", or "write up an RFC". Do NOT auto-trigger on general exploratory phrasing ("let's plan X", "think through Y", "before we start coding") — the user prefers to invoke this skill explicitly when they want it.
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

## Before starting

What a spec contains, and how its fresh-eyes review runs, live outside this file: in `format.md` and `review.md`, in the `spec-format` folder beside this skill's own folder (normally `~/.claude/skills/spec-format/`; this skill's folder is the base directory Claude Code gave when it loaded the skill). Below, `format.md` and `review.md` mean those two files.

Before anything else, check that both are there with `test -f` (normally `test -f ~/.claude/skills/spec-format/format.md && test -f ~/.claude/skills/spec-format/review.md`). If either is missing, say so in one line and stop:

> *"/spec needs spec-format, which is missing or incomplete. Install it with `/skill-manager:add https://github.com/IngvarKofoed/claude-code-spec-format.git --dir spec-format`, then run /spec again."*

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

Once an approach is chosen, read `format.md` right before drafting, and follow it on this pass and every later one. It sets where the spec is written and its name (the path and slug), how long it runs, the template, the worked example, and how to call the Implementation strategy. Every draft carries the Outcome section and the Implementation strategy section from the start, so the user signs off on both.

### 5. Hand off for inline review

Once the draft is on disk, send one handoff message in two parts: a short recap of what the spec proposes, then how to mark it up. The recap is the substance of the message; the markup instructions are a brief procedural close. So the recap is the last thing the user reads about the *proposal itself* before they go mark up the file — it lets them sanity-check the design without opening it and see which parts deserve scrutiny.

**Lead the recap with "The short version of what it proposes:"** Its main content is the spec's **Key decisions** section rendered as *consequences* — the same load-bearing choices, restated as what changes and what to watch rather than as tagged decisions. Keep it to 3–5 bullets, not a table of contents. Aim for this shape:

- The load-bearing change — the single source of truth or the central move, and what cascades from it.
- The concrete edits or new pieces, with real file paths and identifiers.
- What deliberately *stays the same* — the non-change a reviewer might otherwise worry about.
- Any downstream consequence worth watching (a perf, throughput, or blast-radius note).

If you can't compress the spec into 3–5 bullets like these, the Key decisions section is usually the culprit — either missing the load-bearing choice or padded with implementation detail. So the recap doubles as a cheap check on that section.

Close the recap with the Implementation strategy's headline in one line, so the user sees the call without reopening the file. It's part of what they sign off on.

Then, in one line, note that Open Questions remain and that you'll put each to the user next with its default pre-selected — so accepting is a click, not homework. A worked example of the whole recap:

> *"The short version of what it proposes:*
> - *One source of truth — bump `WELLS_PER_STRIP` and `SAMPLES_PER_STRIP` 8→12; scheduler durations, queue/tray math, and the operator-card pills all cascade.*
> - *Two manual edits — the lone hardcoded `repeat(8, …)` at `continuous.css:1082`, and extending the SVG strip template `#STICK_1` with `HOLE_9..12` (concrete coords in the spec) plus a longer body rect.*
> - *No structural scheduler change — it's parameterized; the real consequence is longer loading (80→120 s) and reagent-add (20→30 s) steps, flagged as a throughput thing to watch.*
> - *Tray stays 24 samples → now 2 strips/tray instead of 3.*
>
> *Implementation strategy: **single agent on Opus 5.5** — one constant drives the cascade, and both manual edits sit on the same strip layout.*
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
6. Bring Outcome and the Implementation strategy back in line with whatever moved. If that changes an Outcome bullet or the strategy headline, say so in one line before the next sign-off question, so the user never signs a change they didn't see.

If the user instead says they're happy with the draft as-is, skip the integration pass and move to phase 6.

No review starts at this handoff: the user may still mark the draft up, and a review of text that then changes is wasted. The exception is a wrapper skill that turns the handoff into a sign-off picker. That is a sign-off ask, covered in phase 6. *(for spec-role; remove when it's retired)*

### 6. Iterate to explicit sign-off

Loop on phases 4 and 5 until the spec is ready: comments integrated, the user out of change requests, and the Open Questions section empty (or every question answered).

Ready is not signed off. Sign-off is **explicit, or it hasn't happened** — the user saying yes to the whole spec, in so many words. When you believe the spec is ready — every question answered and folded in — ask for it directly, with **Here's what you get** inside the picker, so the user signs what they can see:

- the Outcome section's **What you get** and **How to verify** bullets, from the final text on disk;
- the Implementation strategy's headline;
- in one line, what changed since the user last saw the spec (their answers, their markup, your fixes), or "nothing changed".

Ask with one AskUserQuestion — *"Do you sign off on this spec?"* — with "Signed off" and "Not yet — more changes" as the options (an Other answer is a change request). Give every option the same `preview`: the block above, as short markdown lines, since it sits in a narrow box beside the options. The same preview on every option keeps it in view whichever one is highlighted.

- **The preview is where the block goes,** not chat text before the picker: that text may be off screen while the picker is open.
- **Keep the question to that one line.** A summary of the design in the question is not What you get, and doesn't replace the preview.
- **No picker available:** show the block in chat, then ask "do you sign off?" plainly.

Never put the sign-off question in the same picker as a question whose answer would still change the spec.

Never infer sign-off from:

- answering the last Open Question, or clicking through the defaults in the picker;
- praise aimed at a part of the spec ("nice", "that section looks good") — especially when it arrives alongside another change request;
- silence, or the user moving the conversation to something else.

#### Start the review as you ask

Sign-off is also when the fresh-eyes review (phase 7) starts — in the same turn, in the background, just before the question goes out. That way it runs while the user reads the final text instead of after they answer. Skip the launch only if a review is already running, or has finished, on the current text. Start it per `review.md`'s "Starting a review", and note the spec's hash as it starts, as that file's "When a review counts" says.

- **A wrapper's sign-off picker counts as asking.** A wrapper skill may offer sign-off itself — with its own options, each carrying the Here's-what-you-get preview, and never next to a question still open. Start the review in the same turn that picker is shown, under the same rules. No review starts at an Open Questions picker — the answers may still change the spec. *(for spec-role; remove when it's retired)*
- **Re-read on every answer.** Before acting on any answer, re-read the spec from disk and compare the hash, since the user may have edited it while reading. `////` lines mean "Not yet", whatever they clicked. Any other change means the sign-off stands on the new text, but the review is stale, so start a fresh one.
- **Stop a stale review.** Whether a review still vouches for the spec is `review.md`'s "When a review counts" — apply it whenever the spec changes or a review ends. When the running review goes stale, stop it (TaskStop, or ignore its result where you can't), say so in one line naming what changed — the user sees the stop, and an unexplained one looks like a failure — and start a fresh one at the next sign-off ask.
- **Hold early findings.** If a review returns before the user has answered (a plain-text ask, or a dismissed picker), hold its findings until the answer comes.
- **After "Signed off":** if the review is back, handle its findings (phase 7). If not, say in one line that it's still running and its findings come when it's done, then end the turn. Its completion notice resumes the work.
- **After "Not yet":** integrate the changes and keep iterating.

The explicit yes gates the end of the skill. Until it's given, keep iterating or ask again; don't slide forward. Even after it, the spec isn't done until phase 7 has run on the text the user signed.

### 7. Fresh-eyes review

The spec is ready to sign — but the person who just wrote it is the worst-placed to judge it, so the text the user is signing goes to a reader who wasn't in the room. The review starts in phase 6, in the background, the moment sign-off is asked. How it runs is in `review.md`:

- **Start it** per "Starting a review". The reviewer's brief holds two paths: the spec, and `review.md` with the instruction to follow its "For the reviewer" part. Nothing from this conversation goes in — not the approach discussion, not the option set from phase 3.
- **Apply "When a review counts"** whenever the spec changes or a review ends. It says which text a review vouches for, and what to do when one fails, returns nothing or is lost.
- **Handle its findings** per "Handling the findings", including its inline pass when subagents aren't available. When the user picks something on a decision, loop back through phase 4 (revise) and phase 5 (re-confirm) for just those points.

#### When the spec is done

The spec is done when a review has returned on the text the user signed and every decision it raised is resolved. Changes made to resolve decisions get the quick check from "Handling the findings", or a fresh review when an answer brought in something the review never saw ("When a review counts", Settling its decisions). Neither gets a new sign-off question.

Then **stop**. Do not begin implementing. The handoff to coding is a separate decision the user makes — they may want to sit on the spec, share it, schedule the work, or use a different session for the build phase. Offering "want me to start on it?" at the very end is fine; jumping straight in is not.

No closing offer comes before this point. That covers your own offer, and any handoff or build offer a wrapper skill adds. Until the review is handled, the spec may still change. *(for spec-role; remove when it's retired)*

## Common mistakes to avoid

Mistakes about what goes in the spec are in `format.md`, and mistakes in handling the review are in `review.md`. These are about the flow:

- **Drafting before the user has picked an approach.** The whole point of phase 3 is to fork the conversation early. Writing the spec first and asking later wastes a draft and biases the discussion.
- **Manufacturing options that aren't real.** If there's only one reasonable approach, say so. A weak Option B insults the user's time.
- **Editing source code "to verify."** If you need to confirm something works, write it as an Open Question and let the user check. Read-only means read-only.
- **Leaving Open Questions unresolved at sign-off.** The spec is "done" only when the section is empty (or removed entirely).
- **Inferring sign-off.** Answering the last Open Question, clicking picker defaults, or praising a section is not approval of the spec. Sign-off is the user saying yes to the whole document, explicitly — ask for it (phase 6), and don't treat the spec as done without it.
- **Starting to implement after sign-off.** The skill ends at the signed-off, reviewed spec (phase 7). Implementation is a separate, deliberate decision by the user.
