# Split what a spec contains from how it is written

Today `/spec` holds two things in one 530-line file: what a spec contains
(the template, Outcome, Key decisions, the strategy guidance, the review's
lenses) and the flow that produces one (phases, pickers, sign-off, review
timing). feature-flow needs the first but runs its own flow, so it wraps
`/spec` and overrides it in five places, and `/spec` has grown rules only
for its wrappers. This change moves the "what" into two plain files in a new
repo, `spec-format`, that both flows read. `/spec` keeps only its flow.
feature-flow gets its own design flow and stops wrapping `/spec`.

## Outcome

**What you get:**

- One place for what a spec contains: `~/.claude/skills/spec-format/`
  with `format.md` and `review.md`. It's installed by skill-manager and isn't
  a skill, so it never appears in the skill list.
- `/spec` behaves exactly as today, with the same phases, questions and
  review. Its `SKILL.md` holds only the flow.
- `/feature-flow` designs on its own and, in a main checkout, no longer runs
  `/spec`. A change to its design steps touches only the feature-flow repo.
- spec-role keeps working unchanged.

**How to verify:**

- Run `/spec` on a small request in a test repo. You get options, then a
  draft with Outcome, Key decisions and strategy, then the open-questions
  picker, "Here's what you get", sign-off and a Fable review, as before.
- Run `/feature-flow` on a small request in a main checkout. No `spec` skill
  is launched in the session; design through sign-off and the build work as
  before.
- Run `grep -n -i -E 'phase [0-9]|picker|wrapper|recap|AskUserQuestion|(^|[^a-z/])/spec\b|feature-flow|spec-role' ~/.claude/skills/spec-format/{format,review}.md`.
  It prints nothing. (`docs/specs/` doesn't match.)
- Move `~/.claude/skills/spec-format` aside, then start `/spec` and
  `/feature-flow <topic>`. Both stop at once with the install command. Move
  it back.
- In a spec-role session, run `/spec` on a small request with no open
  questions. When spec-role shows its sign-off picker at the draft, a Fable
  review starts in that same turn.

## Key decisions

- **Shared content as plain files, not a skill** (new). The `spec-format`
  repo has no `SKILL.md`, so Claude Code skips the folder, just as it skips
  `synced/` and `foss-pptx-generator-workspace/` today. Both flows read it
  as the `spec-format` folder beside their own skill folder (normally
  `~/.claude/skills/spec-format/`), so a moved config folder still works.
- **Its own repo, installed by skill-manager** (extends). It's installed and
  auto-updated like the other skills. `add` doesn't require a `SKILL.md`;
  only `discover` looks for one (`sync.js:807`). It sits apart from `/spec`
  by the user's choice: a session sent to read the format should find
  nothing else in that folder, and neither flow's rules are next to it.
- **The files stand alone** (new). They say what a spec contains and how a
  review sorts its findings. They never mention phases, pickers, wrappers or
  skill names. When things happen is each flow's business.
- **`/spec` keeps its phase numbers and its wrapper rules** (reuses).
  spec-role refers to phase 6's exact question and relies on "a wrapper's
  sign-off picker counts as asking". `/research` refers to phase 3 and "the
  strategy section". Those rules stay, marked *for spec-role; remove when
  it's retired*.
- **feature-flow owns its design flow, in `design.md`** (diverges). It no
  longer invokes `/spec`, except as today's fallback outside a main
  checkout. `SKILL.md` becomes a short router: modes, checks, then
  `design.md` or `build.md`.
- **A missing spec-format stops spec writing** (new). `/spec` checks at its
  start, feature-flow when design begins; each prints the skill-manager `add`
  command. There's no fallback copy, because a copy is exactly the drift
  this change removes.

## Goals

- A change to what a spec contains is made once and reaches both flows.
- A change to either flow touches only that flow's repo.
- `/spec` and spec-role behave as they do today.

## Non-goals

- Retiring spec-role and implementer-role. That's a follow-up, which also
  removes `/spec`'s wrapper rules.
- Changing what a spec contains. Text moves, reworded only where it named a
  phase or a picker.
- Sharing the early steps (understand, research, options) or the revising
  rules (`////` lines, user edits win, Outcome and strategy re-derived).
  Each flow keeps its own copy, and `format.md` stays strictly about what a
  spec contains.
- spec-review and `/research`, which stay as they are.

## Design

### The spec-format repo

```
~/.claude/skills/spec-format/     ← github.com/IngvarKofoed/claude-code-spec-format
├── format.md    what a spec contains
├── review.md    the fresh-eyes review: starting one, the lenses, when it counts, handling findings
└── README.md    for humans: what this is, that it isn't a skill, who reads it
```

It's installed with
`/skill-manager:add git@github.com:IngvarKofoed/claude-code-spec-format.git --dir spec-format`.

### What moves where

| From `/spec` `SKILL.md` today | Goes to |
|---|---|
| Where a spec lives and its name (phase 4, lines 71–80) | `format.md` |
| Length calibration, Template, why Outcome / Key decisions / defaults, worked example (82–296) | `format.md` |
| Revising a spec: `////` lines, user edits win, Outcome and strategy re-derived (phase 5 steps 1–6, 337–344) | stays in `/spec`; feature-flow's `design.md` carries its own copy |
| Calling the implementation strategy, with models and prices (418–506) | `format.md` |
| Style guidelines (507–513) | `format.md` |
| Phase 7's why a fresh reader, what it's given, Fable at default effort (381–387) | `review.md`, part "Starting a review" (read by the session) |
| The six lenses (389–396), plus what to return | `review.md`, part "For the reviewer" (read by the reviewer) |
| Which text a review vouches for: the hash, a change after it started makes it stale (except its own fixes and default-only answers), a failed review counts as none (373–374, 408) | `review.md`, part "When a review counts" |
| Sorting findings, rejected suggestions, quick check, inline fallback (398–406) | `review.md`, part "Handling the findings" |
| Common mistakes (515–530) | Split by topic. Format and review mistakes go to their file; flow mistakes (drafting before an approach is chosen, inferring sign-off, implementing after sign-off) stay in the flow |
| Everything else: read-only rule, phases 1–3, handoff and recap, sign-off, review timing, when done | stays in `/spec` |

### Standing alone

Every sentence that names a step of a flow is reworded to name the
condition instead. For example:

- "Re-derive it from the Design on every pass (phase 5)" becomes
  "Re-derive it whenever the Design changes".
- "Defaults become the pre-selected `(Recommended)` option when the
  questions are put to the user via AskUserQuestion" becomes "a default lets
  the reader accept by silence". How a default is offered moves to each
  flow.

The grep in How to verify enforces this.

### Reading the files

- **Before any spec work,** each flow checks for both files with `test -f`:
  `/spec` at its start, feature-flow when design begins (and when a resume
  finds the design unfinished). Resume into a build, ship and drop never
  need them. If one is missing, the flow says so in one line with the
  HTTPS `add` command and stops.
- **Right before drafting,** each flow reads `format.md` and follows it on
  every pass.
- **Starting the review,** the session reads `review.md`'s "Starting a
  review". The reviewer's brief then holds two paths, where today it holds
  one: the spec, and `review.md` with the instruction to follow its "For the
  reviewer" part. The reviewer stays blind to the conversation, since
  `review.md` carries none.
- **Whenever the spec changes or a review ends,** the session applies "When
  a review counts". Each flow still decides when to start a review, and
  stops a stale one with a one-line reason.
- **When findings arrive,** the session follows "Handling the findings".
- **The contract** between the files and the flows is the two file names and
  the part headings above. Renaming one is a change to both flows.

### /spec after the split

Roughly 200 lines. It keeps:

- why a spec phase exists, the read-only rule, and phases 1–7 with the same
  numbers;
- the recap and markup handoff, the open-questions picker, "Here's what you
  get", the sign-off question with today's wording and options, and the
  review timing (when to start one, when to stop a stale one);
- the wrapper lines (the handoff exception, "a wrapper's sign-off picker
  counts as asking", no wrapper offer before the review is handled), marked
  for spec-role.

Phases 4 and 7 shrink to "read `format.md`" and "start and handle the review
per `review.md`".

### feature-flow after the split

- **`SKILL.md`** is the router: the modes, the main-checkout checks (failing
  falls back to plain `/spec`, as today), then `design.md` (which checks for
  spec-format first) or `build.md`.
- **`design.md`** is its own flow:
  1. understand;
  2. research;
  3. options, then wait for a choice;
  4. read `format.md` (it sets the slug and path), note the ignored files,
     `EnterWorktree` with that slug, and draft;
  5. recap and markup, with open questions in their own picker (no sign-off
     option, no review), and its own copy of the revising rules;
  6. "Here's what you get", then **Sign off and implement (Recommended)** /
     **Sign off, build later** / **Not yet**, and the Fable review starts
     with that picker. **Not yet** goes back to step 5;
  7. after implement: wait for the review and handle it per `review.md`.
     Decisions go in one picker ending with **Then: Build now / Hold**, then
     `build.md`;
  8. after build later: handle the review as in step 7, then commit the
     spec and say how to resume.
- **Rules `design.md` carries its own copy of,** which today it inherits
  from `/spec` by "follow it fully": read-only until sign-off (apart from
  `EnterWorktree` and the spec file), "Here's what you get" and the recap
  shape, never inferring sign-off, re-reading the spec on every answer,
  and holding findings that arrive before the answer. Which text a review
  vouches for comes from `review.md`, not a copy.
- **Gone:** "invoke `/spec` and follow it", and every "where `/spec` says
  something different, this wins".
- **Other `/spec` mentions updated:** `build.md`'s Picking up ("continue the
  design with `/spec`" becomes `design.md`), the `SKILL.md` description, and
  the README, whose requirement becomes spec-format (plus spec, for the
  fallback).

### Rollout

1. Create and push spec-format, and install it.
2. Push `/spec`, then feature-flow.
3. In skill-manager's `RECOMMENDED_SKILLS.md`, `spec`'s section (lines
   52–63) says it needs spec-format and gives its `add` line; the other
   places that list `spec` (lines 25, 129, 181) add spec-format beside it.

Anyone with `/spec` installed through skill-manager, on another machine or
another person's, gets the new `/spec` on the next auto-update and sees the
stop with the install command until they add spec-format. That is the
intended failure. `/spec` used outside Claude Code (Claude.ai, with no local
spec-format) stops the same way.

## Alternatives considered

- **Revising rules in `format.md`.** That would give one shared copy, but
  `format.md` is kept to what a spec contains; each flow carries its own.
- **The stale-review rule copied into each flow.** Rejected: if the copies
  drifted, feature-flow could build on a text no review has read, so the
  rule lives once, in `review.md`.
- **spec-format as a skill.** It would be listed in every session, could
  trigger by itself, and nobody would type it.
- **`format.md` inside `/spec`'s folder.** It would sit next to `/spec`'s
  flow rules, where a session could wander into them.
- **A folder outside `skills/`.** skill-manager can't install or update it.
- **Retire spec-role in the same change.** That loses the two-session
  fallback before feature-flow's ship, drop and resume have run for real.
- **Keep feature-flow wrapping `/spec` and share only the format.** That
  keeps the overrides that started this.

## Implementation strategy

*Not part of the design — a starting point for whoever builds this.*

- **Multi-agent: the lead plus 1 chunk, in sequence.** The lead creates
  spec-format and cuts `/spec` down, since both come from the same text.
  Then a subagent writes feature-flow's `design.md` and trims its `SKILL.md`
  against the finished files.
- **Opus 5.5 for both.** Each is judgment about wording for an LLM reader,
  not transcription.
