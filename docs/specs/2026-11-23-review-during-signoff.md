# Sign-off covers the whole spec, and the review runs while you read

Today `/spec` writes the Outcome section and the Implementation strategy only
after sign-off. So the user never approves either one, even though whoever
builds the spec is graded against Outcome and follows the strategy. The
fresh-eyes review also starts only after sign-off. Over the last month that
meant 174 waits with a median of 5.6 minutes, about 17 hours in total.

This change makes five moves:

1. Both sections go into every draft and stay current on each pass.
2. The review starts in the background when sign-off is asked, not after it's
   given. A wrapper skill's sign-off picker at the draft counts as asking.
3. The reviewer runs on Fable, the fastest setup measured.
4. Findings the user already settled in the conversation are noted, not asked
   again.
5. The model list in the strategy guidance is brought up to the current models.

## Outcome

**What you get:**

- The draft you mark up already contains Outcome and Implementation strategy,
  so your sign-off covers them.
- With a wrapper skill that offers sign-off at the draft (spec-role today, the
  coming design skill later), the fresh-eyes review runs while you read the
  draft. If it finishes before you click and has nothing for you to decide,
  `/spec` is done the moment you sign off.
- In plain `/spec` it starts at the sign-off question. That saves less, because
  you've usually just read the draft by then.
- The review runs on Fable, the fastest setup measured: a median 2.8 minutes,
  against 6.9 on Opus 5.5.
- The review no longer asks again about things you already settled in the
  conversation. It notes them in a line instead.
- The strategy guidance names the current models and prices: Opus 5.5,
  Sonnet 5.5, Haiku 5.5 and Fable 5.2.

**How to verify:**

- Run `/spec` on a small request with no open questions.
  - The first draft has Outcome at the top and Implementation strategy at the
    bottom.
  - The handoff recap ends with the strategy headline.
  - No reviewer starts at the handoff.
- Say you're happy with the draft. A background reviewer starts in the same
  turn as the sign-off question.
- With spec-role loaded, a background reviewer starts in the same turn as its
  draft picker ("Sign off on it as it stands, or mark it up first?").
- Wait until the reviewer has finished with no decisions, then sign off. `/spec`
  ends with no further wait.
- Answer "Not yet — more changes" while a review is running. That review is
  stopped, and a new one starts with the next sign-off question.
- The reviewer's transcript shows `claude-fable-5-2` (or a later Fable).
- In a spec where you rejected a reviewer suggestion, Alternatives considered
  records it. A finding that only repeats an earlier answer of yours shows up
  as a one-line note, not a question.
- `grep -nE "(Opus|Sonnet|Fable) 5(\.[^0-9]|[^.0-9]|\.?$)|Haiku 4\.5|phase [89]" SKILL.md`
  finds nothing, and neither does `grep -niE "phase [89]|nine phases" SKILL.md`.
  The first pattern skips "Opus 5.5" and still catches "Opus 5.".

## Key decisions

- **Outcome and strategy are in every draft** (diverges). This reverses
  today's "written after sign-off" rule (SKILL.md:183-187, 357-365 and the
  mistakes list). Every integration pass brings both back in line with the
  Design. Re-deriving 6–12 lines per pass costs less than an unapproved
  acceptance contract.
- **The review starts with the sign-off question** (extends). It starts in the
  same turn as the question, in the background, on the text being signed. It
  does not start at the draft handoff, so no review is spent on a draft that
  then gets marked up.
- **A review always reads the text being signed** (new). Any edit to the spec
  while a review runs makes it stale. The agent stops it (TaskStop, or ignores
  its result where it can't stop it), and the next sign-off question starts a
  fresh one. This is rare: "Not yet — more changes" came up 2 times in 34
  sign-off questions last month. Two kinds of edit don't make it stale:
  - the agent applying that review's own fixes;
  - folding in Open Question answers that only accept the stated defaults,
    which the review already read.
- **A wrapper's draft-time sign-off picker counts as the sign-off question**
  (extends).
  - spec-role shows sign-off options at the draft today, and the coming design
    skill will too.
  - The review starts in the same turn that picker is shown. That's where 96 of
    102 sign-offs happened last month.
  - The same goes for a wrapper's picker that offers sign-off next to Open
    Questions. Those drafts were 25% of sign-offs but 3.9 of the 8.2 hours of
    waiting left, and every default was accepted in 21 of 35 of those pickers.
  - Nothing changes in the wrapper; `/spec` says when a picker counts. In plain
    `/spec` the Open Questions picker is not a sign-off ask, so no review
    starts there.
- **The reviewer runs on Fable** (new).
  - The review's Agent call passes `model: "fable"`. Today `/spec` names no
    model, so the review runs on whatever the session passes.
  - Last month's median review time by model: Fable 5.2 2.8 min (46 runs),
    Opus 5.5 6.9 min (102), Fable 5.1 7.5 min (26).
  - Effort isn't pinned, matching the measured runs.
  - Fable costs about twice as much per token as Opus.
- **Findings get a third outcome: already decided** (extends). Decisions go to
  the user and fixes get made, as today.
  - A finding that re-opens something the user already settled in this
    conversation isn't asked again. That covers an approach chosen in phase 3,
    an answered question, a markup comment, or a suggestion they rejected.
  - Instead, the agent makes sure the spec states the decision, and notes it in
    a line.
  - A reviewer suggestion the user rejects is recorded in Alternatives
    considered, so builders and later reviews don't raise it again.
  - Last month a quarter of review answers went against the recommended
    option, and several of those were "keep it as specced". The bar stays at
    critical/high, and there is still no re-review treadmill.
- **Nine phases become seven** (breaking, for references).
  - The old phases 7 (Outcome) and 9 (strategy) fold into phase 4 (draft), and
    the old phase 8 (review) becomes phase 7.
  - The strategy guidance moves out of the numbered workflow into a reference
    section, **Calling the implementation strategy**.
  - Phase 6, the sign-off question, keeps its number, so spec-role's references
    to it still hold.
- **The model list is updated** (extends).
  - **Opus 5.5** ($4/$20 per MTok) stays the implementation default.
  - **Sonnet 5.5** ($2/$10) is for delegated chunks.
  - **Haiku 5.5** ($0.10/$0.50, rising to $0.50/$2.50 past 100K-token prompts)
    is for mechanical transforms. It now has a 1M context, so the
    "much smaller context window" caveat goes.
  - **Fable 5.2** ($8/$40, twice Opus) is for the hardest long-horizon work.

## Goals

- The user signs off on everything a builder relies on: Design, Outcome and
  strategy.
- Less waiting: the review starts when sign-off is asked, including at a
  wrapper's draft picker, not after it's given, and it runs on the fastest
  setup measured.
- Fewer questions: the review never re-asks something the user already
  settled.
- The skill names models that exist.

## Non-goals

- The "Sign off and implement" option, worktrees and the background builder.
  Those belong to the coming design skill, which builds on this change.
- Changing the reviewer's six lenses or its critical/high bar.
- Making the answers to the review's decisions the go signal for a build. That
  belongs to the design skill.
- Editing spec-role or implementer-role in any way. They are due to be retired
  and stay untouched. Two rough edges remain while spec-role is in use:
  - Its "the strategy didn't exist when the user clicked" now makes it ask
    about ultracode when it doesn't need to.
  - It offers its handoff once the strategy section exists, which is now from
    the first draft. `/spec`'s new "no closing offer before the spec is done"
    rule (phase 7) holds that offer until after the review.

## Design

All edits are in `~/.claude/skills/spec/SKILL.md`, plus one line elsewhere.

### Phase 4: the draft

- The template's Outcome placeholder (SKILL.md:102) and Implementation strategy
  placeholder (:179) change from "Added in phase 7/9, after sign-off" to
  "Written with the draft; kept in sync on every pass".
- "Why Outcome is written last but placed first" (:183-187) becomes **"Why
  Outcome is in the draft"**.
  - The user signs off on it because the builder is graded against it.
  - It is re-derived from the Design on every pass, never recycled from Goals.
  - The verify bullets stay black-box.
- Length calibration (:88) keeps the exemption: Outcome and strategy don't count
  against the 50–200 line budget.
- The note after the worked example (:296) drops "appended after sign-off".

### Phase 5: handoff

- The recap's last line becomes the strategy headline, for example
  *"Implementation strategy: single agent on Opus 5.5 — four files on one code
  path."* This replaces the old phase 9 echo.
- The integration pass gets a sixth step: bring Outcome and Implementation
  strategy back in line with whatever moved. If that changes the strategy
  headline or an Outcome bullet, say so in one line before the next sign-off
  question, so the user doesn't sign a change they never saw.
- No review starts here, unless a wrapper turns the handoff into a sign-off
  picker (phase 6).

### Phase 6: sign-off

- The sign-off question stays as it is.
- In the same turn, just before asking, start the reviewer in the background,
  unless one is already running or has finished on the current text. Note the
  file's hash (`shasum`) as the review starts.
- On any answer to the sign-off question, re-read the spec first, because the
  user may have edited it while reading.
  - `////` lines mean "Not yet".
  - Any other change means the sign-off stands on the new text, but the review
    is stale: start a fresh one.
- **Stale means the text changed.** A review goes stale only when the user's
  edits or a "Not yet" pass actually change the spec. Then it is stopped
  (TaskStop, or its result ignored) and the next sign-off question starts a
  fresh one. The agent's own edits that apply that review's fixes never make
  it stale.
- If a review returns while no sign-off answer has come yet (a plain-text ask,
  or a dismissed picker), hold its findings until the answer.
- After "Signed off":
  - If the review has returned, handle it (phase 7).
  - If not, say in one line that it's still running and the findings come when
    it's done. Then end the turn; the completion notice resumes the work.
- After "Not yet": integrate the changes and loop.
- A wrapper skill's picker that offers sign-off at the draft counts as the
  sign-off question, including one that also carries Open Questions. Start the
  review in the same turn the picker is shown, under the same rules: stale on
  edits, re-read on the answer, findings held until it. Accepting only the
  defaults keeps the review valid.

### Phase 7: fresh-eyes review (formerly phase 8)

- The lenses move over unchanged.
- The reviewer is launched with `model: "fable"`.
- Findings now sort into three groups:
  - **Decisions** go to the user.
  - **Fixes** get made.
  - **Already decided** findings are stated in the spec and noted in a line,
    not asked. The reviewer doesn't see the conversation, so the main agent
    makes this call.
- A rejected reviewer suggestion gets one line in Alternatives considered.
- The opening changes from "the spec is signed off" to "the spec is ready to
  sign". The reviewer still gets only the spec path.
- The stale rule from Key decisions is stated once here.
- The inline fallback (no subagents) runs the pass right before the sign-off
  question, since it can't overlap with the user's reading.
- A review that fails, returns nothing, or is lost counts as no review, never
  as "nothing to decide". Say so, re-run it once, and if that fails too, fall
  back to the inline pass.
- **When the spec is done.** A review has returned on the text the user signed,
  and every decision it raised is resolved.
  - Resolving decisions keeps today's rule: revise, then re-confirm just the
    changed points (SKILL.md:384-387).
  - Those changes get a quick check that the flagged items are resolved, not a
    new review and not a new sign-off question.
- The phase ends with today's closing line: stop, and offering "want me to
  start on it?" is fine.
- **No closing offer before the spec is done.** Neither `/spec`'s offer nor a
  wrapper skill's handoff or build offer comes before this phase ends.
  spec-role offers its handoff once the strategy section exists, which is now
  from the first draft. This rule keeps that offer after the review without
  changing spec-role.

### Calling the implementation strategy (reference section)

The old phase 9 text moves here intact (Strategy, Agent count, Model, the
section shape and examples, "Call it in both directions"), with these changes:

- every model name in the file (`Opus 5` → `Opus 5.5`, `Sonnet 5` →
  `Sonnet 5.5`, `Haiku 4.5` → `Haiku 5.5`, `Fable 5` → `Fable 5.2`). That
  includes the worked example in phase 4 (SKILL.md:291) and the strategy
  examples and chat echo (:435, :450, :467, :474), not just the model list
  (:419-422);
- "As of writing" gets a date (2026-11);
- the Haiku context caveat goes;
- "Then stop" moves to phase 7.

### Cross-references

- **Frontmatter description:** it says Outcome, review and strategy all come
  "only after that explicit sign-off". It is rewritten to say the draft carries
  Outcome and strategy, and that the review runs in the background on text
  ready to sign.
- **Phase 6:** its "everything that follows — phases 7, 8, 9" line is updated.
- **Workflow intro (:34):** "Nine phases" becomes "Seven phases".
- **Common mistakes (:501, :503):** "Phase 8 earns its keep" becomes "Phase 7
  earns its keep", and "Phase 9 picks on the shape of the work" becomes "The
  strategy call picks on the shape of the work".
- **Common mistakes:**
  - "Writing the Outcome section or Implementation strategy before sign-off"
    becomes "Letting Outcome or the strategy go stale between passes".
  - The "Inferring sign-off" item's "don't start phases 7–9" becomes "don't
    treat the spec as done".
  - "Starting to implement after sign-off" keeps its point, with the new phase
    numbers.
- **Goal-fit lens:** its "Outcome bullets (phase 7)" becomes "(in the draft)".
- **`~/.claude/skills/research/SKILL.md:131`:** "`/spec` phase 8 calls
  ultracode" (already off by one) becomes "`/spec`'s strategy section calls
  ultracode".

## Alternatives considered

- **Also start the review at the draft handoff.** Last month the user signed
  off at the draft 96 times out of 102, so this would hide more of the wait.
  Rejected by the user for plain `/spec`: the review starts only when sign-off
  is asked. A wrapper's draft picker counts as asking.
- **Review during markup, on the first draft.** It overlaps more of the
  waiting, but your `////` changes would go unreviewed, and a review that never
  sees the final text can't vouch for it. Rejected.
- **Keep nine phases with 7 and 9 as stubs.** That keeps old references working,
  but leaves two phases that only say "see phase 4". Only one stale reference
  exists outside this file (research). Rejected.

## Implementation strategy

*Not part of the design — a starting point for whoever builds this.*

- **Single agent, Opus 5.5.** One file whose edits depend on each other
  (renumbering, cross-references, moved text), plus a one-line touch in
  `research/SKILL.md`. There's no chunk a subagent could take without
  re-reading the rest.
