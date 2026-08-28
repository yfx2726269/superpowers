# Framework Skill Revision — Implementation Plan

> **For agentic workers:** This is a meta-plan that edits the framework skills
> themselves. Execute it **inline** in the current session (do not dispatch
> subagents per task — the very behavior under revision). Steps use checkbox
> (`- [ ]`) syntax for tracking.

**Goal:** Revise three places in the local 二开 branch's superpowers skills to
shift task sizing, subagent session model, and model-selection wording toward
functional-block boundaries, session reuse, and user-side config ownership.

**Architecture:** Pure documentation edits to two SKILL.md files. No code,
no tests. Each task is a localized text replacement with exact before/after
captured below; verification is re-read + `git diff` review, not automated
tests.

**Tech Stack:** Markdown skill files (superpowers framework).

**Spec:** This plan is the spec — derived from the user's three-entry revision
request plus the clarifications answered during planning:

- **Q1 (task_revive):** Do NOT name the tool. Use generic wording
  ("reuse the same session across segments when the runtime supports session
  resumption").
- **Q2 (wave == task):** A "wave" == one task (functional block). The user
  confirmed this during planning ("每个 task 是一波"). Consequence: the existing
  **per-task** review + fix loop already satisfies 2b's "波内 findings 聚合一次修复"
  intent — **§4's loop structure is NOT restructured**. The only §4 change
  needed is to strip wording that contradicts accepted edits 2a/3b (see Task 3).
- **Q3 (ripple):** All cross-references that contradict the new wording must be
  rewritten for consistency (see Task 6).

## Global Constraints

- Language: keep English prose in the skill files (matching existing voice);
  do not translate skill body text to Chinese.
- Preserve the "your human partner" / "ledger" / "ruling" vocabulary already in
  the files — only the targeted clauses change.
- Do not alter any other behavioral content outside the listed line ranges.
- `task_revive` is an omo runtime tool; per Q1 it is **not named** in the skill.
- These edits target a **local 二开 branch** only; they intentionally diverge
  from upstream superpowers philosophy and are not intended for upstream PR.
- **per-task == 波内**: keep the per-task loop as-is; do not rewrite §4/§5 into
  an "aggregated-once" model.

---

### Task 1: writing-plans — Task Right-Sizing + Bite-Sized Task Granularity

**Files:**
- Modify: `superpowers/skills/writing-plans/SKILL.md:36-52`

**Edit 1b — Task Right-Sizing (replace lines 36-43):**

Old:
```
## Task Right-Sizing

A task is the smallest unit that carries its own test cycle and is worth a
fresh reviewer's gate. When drawing task boundaries: fold setup,
configuration, scaffolding, and documentation steps into the task whose
deliverable needs them; split only where a reviewer could meaningfully
reject one task while approving its neighbor. Each task ends with an
independently testable deliverable.
```

New:
```
## Task Right-Sizing

A task is a functionally complete block that can be verified independently
and committed independently; the review gate acts on the functional block.
When drawing task boundaries: fold setup, configuration, scaffolding, and
documentation steps into the task whose deliverable needs them; split only
where a reviewer could meaningfully reject one task while approving its
neighbor. Each task ends with an independently testable deliverable.
```

**Edit 1a — Bite-Sized Task Granularity (replace the bold line + keep list, lines 45-52):**

Old:
```
## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step
```

New:
```
## Bite-Sized Task Granularity

**A step is a line in the implementation checklist; it is not a dispatch or
review boundary.** Tasks are divided along complete functional boundaries —
not by time estimate, story-point count, or file count.

- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step
```

- [ ] **Step 1: Apply both edits** (use Edit tool with the exact old/new above)
- [ ] **Step 2: Verify** — re-read lines 36-52; confirm no "2-5 minutes" and no
  "worth a fresh reviewer's gate" remain. Run `git diff --stat` to confirm only
  this file changed.

---

### Task 2: subagent-driven-development — Session model (2a)

**Files:**
- Modify: `superpowers/skills/subagent-driven-development/SKILL.md:8`, `:12`, `:65`, `:278-279`

**Edit 2a-1 — description (line 8):**

Old:
```
Execute plan by dispatching a fresh implementer subagent per task, a task review (spec compliance + code quality) after each, and a broad whole-branch review at the end.
```
New:
```
Execute plan by running one implementation session per functional block, a task review (spec compliance + code quality) after each, and a broad whole-branch review at the end.
```

**Edit 2a-2 — core principle (line 12):**

Old:
```
**Core principle:** Fresh subagent per task + task review (spec + quality) + broad final review = high quality, fast iteration
```
New:
```
**Core principle:** One implementation session per functional block (the same session may be reused to continue the block across segments when the runtime supports session resumption) + task review (spec + quality) + broad final review = high quality, fast iteration. A fresh subagent is reserved for unrelated new work only.
```

**Edit 2a-3 — vs Executing Plans bullet (line 65):**

Old:
```
- Fresh subagent per task (no context pollution)
```
New:
```
- One session per functional block (reusable across segments)
```

**Edit 2a-4 — §1 Dispatch the implementer (lines 278-279):**

Old:
```
A fresh subagent needs its task, the interfaces it touches, and the global
constraints. Nothing else.
```
New:
```
The implementing session for a functional block needs that block's task, the
interfaces it touches, and the global constraints. Nothing else. A fresh
subagent is used only when the next work is unrelated to what this block did.
```

- [ ] **Step 1: Apply the four edits** (exact old/new above)
- [ ] **Step 2: Verify** — re-read lines 8, 12, 65, 278-279; no "Fresh subagent
  per task" remains in these spots.

---

### Task 3: subagent-driven-development — Keep per-task loop, strip model/fresh escalation (2b, clarified)

**Clarification (from planning):** The user confirmed "per-task" == "波内"
(one task == one wave). Therefore the existing per-task review + fix loop
already satisfies 2b's "波内 findings 聚合一次修复" intent — **no structural
restructure of §4/§5 is needed.** The only change required is to remove wording
that contradicts accepted edits 2a (fresh only for unrelated work) and 3b
(Model Selection deleted): the rounds 4-5 "fresh implementer on a more capable
model (per Model Selection)" language. The 5-round loop and the breaker at
round 5 are kept as-is.

**Files:**
- Modify: `superpowers/skills/subagent-driven-development/SKILL.md:389-394` (§4 rounds 4-5)
- Modify: `superpowers/skills/subagent-driven-development/SKILL.md:85` (process diagram label)

**Edit 2b — §4 rounds 4-5 (lines 389-394):**

Old:
```
**Rounds 4-5 — dispatch a fresh implementer on a more capable model** (per
Model Selection), with the brief path, the report-file path, the open
findings, and this framing: "A prior implementer attempted this task
[N] times; you own it now. Read the report file for what was tried." A loop
that survives three resumes usually means the implementer cannot see its
own problem — fresh eyes and a capability bump in one move.
```

New:
```
**Rounds 4-5 — resume the same implementing session** (or a replacement
session if it cannot be revived), with the brief path, the report-file path,
the open findings, and this framing: "A prior implementer attempted this task
[N] times; you own it now. Read the report file for what was tried." A loop
that survives three resumes usually means the implementer cannot see its
own problem — fresh eyes in one move.
```

**Edit 2b-diagram — process diagram label (line 85):**

Old:
```
"Fix round R of 5: R≤3 resume implementer; R≥4 fresh implementer, more capable model" [shape=box];
```
New:
```
"Fix round R of 5: R≤3 resume implementer; R≥4 replacement session (no model escalation)" [shape=box];
```

- [ ] **Step 1: Apply the §4 rounds 4-5 edit** (exact old/new above).
- [ ] **Step 2: Apply the diagram label edit** (line 85).
- [ ] **Step 3: Verify** — `grep -n "more capable model\|per Model Selection" subagent-driven-development/SKILL.md`
  returns no hits (the only remaining "Model Selection" references are the
  section title itself and the Review-tasks line added in Task 5).

---

### Task 4: subagent-driven-development — Reviewer reuse clause (2c)

**Files:**
- Modify: `superpowers/skills/subagent-driven-development/SKILL.md` — insert after line 322 (`both are needed.`)

**Edit 2c — new clause inside §3 Review the task:**

Insert after the paragraph ending `... both are needed.` (line 322):
```
**Reviewer session reuse:** Reuse one oracle reviewer session across functional
blocks. Review is read-only with a small pollution surface; the benefit is
cross-block consistency. The reviewer and the implementer must be different
seats — the implementer never reviews its own block.
```

- [ ] **Step 1: Insert the clause** at the stated location.
- [ ] **Step 2: Verify** — re-read §3; the clause appears once, directly after
  "both are needed."

---

### Task 5: subagent-driven-development — Model Selection (3a + 3b)

**Files:**
- Modify: `superpowers/skills/subagent-driven-development/SKILL.md:192-227` (whole Model Selection section)
- Modify: `superpowers/skills/subagent-driven-development/SKILL.md:204-207` (Review tasks — inside the section above)

**Edit 3a — Review tasks (lines 204-207), replaced as part of the section rewrite:**

Old:
```
**Review tasks**: choose the model with the same judgment, scaled to the
diff's size, complexity, and risk. A small mechanical diff does not need the
most capable model; a subtle concurrency change does. Scoped re-reviews of
small fix diffs take a cheap-to-mid tier.
```

**Edit 3b — entire Model Selection section (lines 192-227) becomes:**

Old (whole section, lines 192-227):
```
## Model Selection

Use the least powerful model that can handle each role to conserve cost and increase speed.

**Mechanical implementation tasks** (isolated functions, clear specs, 1-2 files): use a fast, cheap model. Most implementation tasks are mechanical when the plan is well-specified.

**Integration and judgment tasks** (multi-file coordination, pattern matching, debugging): use a standard model.

**Architecture and design tasks**: use the most capable available model.
The final whole-branch review is one of these — dispatch it on the most
capable available model, not the session default.

**Review tasks**: choose the model with the same judgment, scaled to the
diff's size, complexity, and risk. A small mechanical diff does not need the
most capable model; a subtle concurrency change does. Scoped re-reviews of
small fix diffs take a cheap-to-mid tier.

**Fix-loop escalation (rounds 4-5)**: use a model at least one tier above
the implementer that got stuck.

**Always specify the model explicitly when dispatching a subagent.** An
omitted model inherits your session's model — often the most capable and
most expensive — which silently defeats this section.

**Turn count beats token price.** Wall-clock and context cost scale with how
many turns a subagent takes, and the cheapest models routinely take 2-3× the
turns on multi-step work — costing more overall. Use a mid-tier model as the
floor for reviewers and for implementers working from prose descriptions.
When the task's plan text contains the complete code to write, the
implementation is transcription plus testing: use the cheapest tier for
that implementer. Single-file mechanical fixes also take the cheapest tier.

**Task complexity signals (implementation tasks):**
- Touches 1-2 files with a complete spec → cheap model
- Touches multiple files with integration concerns → standard model
- Requires design judgment or broad codebase understanding → most capable model
```

New (entire section, lines 192-207 after removal of 209-222):
```
## Model Selection

Model selection is decided by user-side configuration. The dispatcher does
not assume it has model parameters available when dispatching a subagent.

**Review tasks**: fixed to the oracle seat. The dispatcher does not select a
model for review — review is the oracle seat's responsibility, reused across
functional blocks.
```

- [ ] **Step 1: Replace the whole Model Selection section** (lines 192-227) with
  the New block above (section header + config statement + Review tasks line).
- [ ] **Step 2: Verify** — `grep -n "least powerful\|most capable\|cheap model\|tier above\|specify the model" subagent-driven-development/SKILL.md`
  returns no hits (these phrases are gone). The word "oracle" now appears in
  both §3 (2c) and Model Selection (3a).

---

### Task 6: Consistency ripple edits (all cross-references)

**Files:**
- Modify: `superpowers/skills/writing-plans/SKILL.md:61`
- Modify: `superpowers/skills/subagent-driven-development/SKILL.md:8` (done in Task 2), `:65` (done in Task 2), `:278-279` (done in Task 2), `:460`, §3 reviewer terminology, Example Workflow `:512-577`

**R1 — writing-plans plan-header template (line 61):**

Old:
```
> **For agentic workers:** Implement this plan task-by-task with `superpowers:executing-plans` (inline). Under the orchestrator (multi-agent) mode you may instead use `superpowers:subagent-driven-development` to dispatch a fresh subagent per task. A build agent in single-execution mode always stays inline. Steps use checkbox (`- [ ]`) syntax for tracking.
```
New:
```
> **For agentic workers:** Implement this plan task-by-task with `superpowers:executing-plans` (inline). Under the orchestrator (multi-agent) mode you may instead use `superpowers:subagent-driven-development` to run one implementation session per functional block. A build agent in single-execution mode always stays inline. Steps use checkbox (`- [ ]`) syntax for tracking.
```

**R7 — Final Review dispatch line (line 460):**

Old:
```
Dispatch on the most capable available model (see Model Selection), using
superpowers:code-review's
[code-reviewer.md](../code-review/code-reviewer.md).
```
New:
```
Dispatch the oracle seat (the fixed reviewer, per Model Selection), using
superpowers:code-review's
[code-reviewer.md](../code-review/code-reviewer.md).
```

**R8 — §3 reviewer terminology:** replace "task reviewer" / "task reviewer-prompt.md"
with "the oracle reviewer (reused across blocks)" / "oracle-reviewer-prompt.md"
where it reads as the per-task reviewer. (Keep template filenames that actually
exist on disk as-is if the file is shared; only change the prose label.)
Concretely update the lead sentence of §3 (line 318) and the two bullets that
say "dispatch the task reviewer" (lines 298, 332) to say "dispatch the oracle
reviewer".

**R9 — Example Workflow (lines 512-577):** light-touch sync — change the two
`[Dispatch task reviewer with the printed path]` narration lines to
`[Dispatch the oracle reviewer with the printed path]` and the `Task reviewer:`
output lines to `Oracle reviewer:` so the illustrative transcript matches the
new terminology. (Cosmetic; safe to skip if you prefer minimal churn.)

- [ ] **Step 1: Apply R1** (writing-plans L61).
- [ ] **Step 2: Apply R7** (Final Review L460).
- [ ] **Step 3: Apply R8** (§3 reviewer terminology, lines 298/318/332).
- [ ] **Step 4: Apply R9** (Example Workflow) — optional, confirm with user.
- [ ] **Step 5: Verify** — `grep -rn "fresh subagent per task\|Fresh subagent per task\|most capable available model" superpowers/skills/` returns no hits outside this plan.

---

## Self-Review

**1. Spec coverage:**
- 1a ✅ Task 1 (Bite-Sized) — time metric removed, step redefined.
- 1b ✅ Task 1 (Task Right-Sizing) — task = independently verifiable/committable block, gate on block.
- 2a ✅ Task 2 — one session per block, reusable across segments, fresh only for unrelated.
- 2b ✅ Task 3 — per-task loop KEPT (per-task == 波内); only the rounds 4-5 "fresh / more capable model (per Model Selection)" wording stripped to align with 2a/3b.
- 2c ✅ Task 4 — oracle reviewer reused across blocks, different seat from implementer.
- 3a ✅ Task 5 — Review tasks fixed to oracle seat.
- 3b ✅ Task 5 — whole Model Selection section → user-side config statement.
- Ripple ✅ Task 6 — all contradicting references rewritten.

**2. Placeholder scan:** No TBD/TODO/"implement later". Every edit carries exact
old/new text.

**3. Type consistency:** N/A — documentation only. Terminology is consistent:
"functional block" == "task" (per Q2), "oracle reviewer" == "oracle seat"
(per 2c/3a).

## Execution Handoff

After you confirm, execute inline (this is a meta-plan; do not use
subagent-driven-development to edit subagent-driven-development). Commit
logically per file when done; the human partner handles branch/PR.
