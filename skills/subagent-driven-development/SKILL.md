---
name: subagent-driven-development
description: "Execute an implementation plan under the orchestrator by dispatching implementer sessions. Provides the mechanics — plan-scoped workspace and ledger, task briefs, review packages, dispatch discipline, one review per functional block."
---

# Subagent-Driven Development

Execute a plan by dispatching implementer sessions and reviewing each block's diff. This skill supplies the **mechanics** — workspace, ledger, task briefs, review packages, dispatch discipline. The **policy** it serves lives in your human partner's AGENTS.md: one task is one independently verifiable, committable functional block; reuse sessions for related work, start fresh when the context would be polluted (your judgment per dispatch); one review per block with findings fixed in one aggregated fix wave, and one final whole-branch review; the reviewer and the implementer are different seats; review is fixed to the oracle seat; model selection is decided by user-side configuration — the dispatcher does not assume it has model parameters to set.

**Why subagents:** You delegate tasks to specialized agents with isolated context. By precisely crafting their instructions and context, you ensure they stay focused and succeed at their task. They should never inherit your session's context or history — you construct exactly what they need. This also preserves your own context for coordination work.

**Mode requirement:** Only use this skill under the **orchestrator (multi-agent) mode**, where you dispatch implementer sessions. A **build agent running in single-execution mode** must NOT use this skill — it never dispatches subagents itself; that is the orchestrator's job. In single-execution mode, implement the plan inline with `superpowers:executing-plans` instead.

**Narration:** between tool calls, narrate at most one short line — the
ledger and the tool results carry the record.

**Continuous execution:** Do not pause to check in with your human partner between tasks. Execute all tasks from the plan without stopping. The only reasons to stop are the ones named below, or all tasks complete. "Should I continue?" prompts and progress summaries waste their time — they asked you to execute the plan, so execute it.

**Rulings within the plan, not stalls.** Conflicts, ambiguities,
and plan defects — rule on them against the approved plan: the spec is
the binding authority, the plan is its argument, and your judgment
settles what neither answers. Record every decision in the ledger as
`Ruling: <what you decided> — <why> — <what it costs if wrong>`, and keep
going. A wrong ruling costs rework your human partner can see and undo; a
session parked on a question costs their whole day and buys nothing.

Two boundaries sit outside your ruling authority. A substantive change
beyond the approved plan — new scope, changed interfaces, different
architecture — goes back to your human partner and waits for approval
before you execute it. And a conflict the approved plan cannot decide,
where every resolution is a guess, stops for a question, not a ruling:
guessing there is a ruling made in secret.

These stop you, and only these: an irreversible or destructive
operation; a security-sensitive action; a side effect outside this checkout
that norms say you ask about first (e.g. touching another project or a
shared resource); a substantive change outside the approved plan; and a
conflict the approved plan cannot decide. For those, stop and ask.

## When to Use

```dot
digraph when_to_use {
    "Have implementation plan?" [shape=diamond];
    "Tasks mostly independent?" [shape=diamond];
    "Stay in this session?" [shape=diamond];
    "subagent-driven-development" [shape=box];
    "executing-plans" [shape=box];
    "Manual execution or brainstorm first" [shape=box];

    "Have implementation plan?" -> "Tasks mostly independent?" [label="yes"];
    "Have implementation plan?" -> "Manual execution or brainstorm first" [label="no"];
    "Tasks mostly independent?" -> "Stay in this session?" [label="yes"];
    "Tasks mostly independent?" -> "Manual execution or brainstorm first" [label="no - tightly coupled"];
    "Stay in this session?" -> "subagent-driven-development" [label="yes"];
    "Stay in this session?" -> "executing-plans" [label="no - parallel session"];
}
```

**vs. Executing Plans (parallel session):**
- Same session (no context switch)
- Reuse a session for related work; start fresh when the context would be polluted (agent's judgment per dispatch)
- Review after each functional block (spec compliance + code quality), broad review at the end
- Faster iteration (no human-in-loop between tasks)

## The Process

```dot
digraph process {
    rankdir=TB;

    "Setup: workspace, ledger, read plan, pre-flight scan" [shape=box];
    "More functional blocks?" [shape=diamond];
    "Dispatch implementer (./implementer-prompt.md)" [shape=box];
    "Handle report (DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED)" [shape=box];
    "Review package, dispatch the task reviewer (./task-reviewer-prompt.md)" [shape=box];
    "Review clean?" [shape=diamond];
    "ONE aggregated fix wave, then one scoped re-review (./re-review-prompt.md)" [shape=box];
    "Adjudicate residuals: park with rulings, or rule and carry" [shape=box];
    "Ledger completion; next block" [shape=box];
    "Final whole-branch review (oracle seat)" [shape=box];
    "ONE final fix wave + one scoped re-review" [shape=box];
    "Delete workspace; report rulings and completion" [shape=box style=filled fillcolor=lightgreen];

    "Setup: workspace, ledger, read plan, pre-flight scan" -> "More functional blocks?";
    "More functional blocks?" -> "Dispatch implementer (./implementer-prompt.md)" [label="yes"];
    "More functional blocks?" -> "Final whole-branch review (oracle seat)" [label="no"];
    "Dispatch implementer (./implementer-prompt.md)" -> "Handle report (DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED)";
    "Handle report (DONE / DONE_WITH_CONCERNS / NEEDS_CONTEXT / BLOCKED)" -> "Review package, dispatch the task reviewer (./task-reviewer-prompt.md)";
    "Review package, dispatch the task reviewer (./task-reviewer-prompt.md)" -> "Review clean?";
    "Review clean?" -> "Ledger completion; next block" [label="yes"];
    "Review clean?" -> "ONE aggregated fix wave, then one scoped re-review (./re-review-prompt.md)" [label="no"];
    "ONE aggregated fix wave, then one scoped re-review (./re-review-prompt.md)" -> "Adjudicate residuals: park with rulings, or rule and carry";
    "Adjudicate residuals: park with rulings, or rule and carry" -> "Ledger completion; next block";
    "Ledger completion; next block" -> "More functional blocks?";
    "Final whole-branch review (oracle seat)" -> "ONE final fix wave + one scoped re-review";
    "ONE final fix wave + one scoped re-review" -> "Delete workspace; report rulings and completion";
}
```

## Setup

Worktree/branch/merge/push/PR operations are done by the human partner,
not by the agent.

Conversation memory does not survive compaction. In real sessions,
controllers that lost their place have re-dispatched entire completed task
sequences — the single most expensive failure observed. Track progress in
a ledger file, not only in todos.

- Each plan owns a workspace: at skill start, run this skill's
  `scripts/sdd-workspace PLAN_FILE` — it prints the plan's git-ignored
  directory (`<repo-root>/.superpowers/sdd/<plan-basename>/`), home to
  every artifact for THIS plan: ledger, briefs, reports, review packages.
  Another plan's directory is never yours to read or write.
- Check for this plan's ledger at `<workspace>/progress.md`. If its first
  line names your plan file, tasks with a `Task <N>: complete` line are DONE
  — do not re-dispatch them; resume at the first task without one. A task
  whose last line is a fix wave entry is mid-loop: re-run the scoped
  re-review for the commits it names, then adjudicate the residuals. A
  ledger whose first line names a different plan
  file — or a stray ledger at the old flat path `.superpowers/sdd/progress.md`
  — is another plan's progress: leave it in place and start your own, fresh.
- Create the ledger with its identity as the first line:
  `# SDD ledger — plan: <plan file path>`.
- The ledger is your recovery map: the commits it names exist in git even
  when your context no longer remembers creating them. After compaction,
  trust the ledger and `git log` over your own recollection.
- `git clean -fdx` will destroy the workspace (it's git-ignored scratch); if
  that happens, recover from `git log`.

Read the plan once, note its context and Global Constraints, and create a
todo per task. If the plan names a Spec, read that too: the spec is the
authority the plan argues from, and conflicts inside the plan resolve
against it. A plan with no reachable spec gets a ledger note saying so —
rulings made without one are provisional.

Before dispatching Task 1, scan the plan once for conflicts, writing down
what you checked as you check it:

- tasks that contradict each other or the plan's Global Constraints
- anything the plan explicitly mandates that the review rubric treats as a
  defect (a test that asserts nothing, verbatim duplication of a logic block)

The scan's output is a table, not a verdict. One row for every pair of tasks
that share a file or an interface: the two tasks, what one produces against
what the other consumes, and what you found. One row for every task: whether
its own text agrees with itself — the tests it specifies against the code it
specifies, the files it creates against the files it later touches. "The scan
is clean" without those rows is not a scan you ran.

Write the table to the ledger. Rule on everything you find before execution
begins — each finding against the plan text that mandates it — and record
each ruling in the ledger. If the scan is clean, proceed without comment.
Rule on each conflict it surfaces — the spec is the binding authority, the
plan is its argument — record the ruling beside its row, and dispatch
Task 1. The review and fix wave remain the net for conflicts that only
emerge from implementation.

## Model Selection

Model selection is decided by user-side configuration. The dispatcher does
not assume it has model parameters available when dispatching a subagent.

**Review tasks**: fixed to the oracle seat. The dispatcher does not select a
model for review — review is the oracle seat's responsibility, reused across
functional blocks.

## The Task Loop

**Batch small same-shape work.** When the plan lists several tasks that are
each a small, independent edit of the same kind — the same one-line fix,
constant change, or field addition repeated across files — do not dispatch
one subagent per task. Compose ONE dispatch brief listing every file and
its change, send the whole batch to a single subagent, and review its diff
as one unit. Reserve one-dispatch-per-task for work that needs its own
judgment, its own tests, or its own review surface.

Everything you paste into a dispatch prompt — and everything a subagent
prints back — stays resident in your context for the rest of the session
and is re-read on every later turn. Hand artifacts over as files.

**Waiting on dispatched subagents:** never poll a wait interface with
short timeouts, and never sit in one silent, open-ended wait either.
While you have local work — ledger updates, packaging the next review,
reading reports — keep working; child results arrive on their own.
When you are genuinely idle, wait in bounded stretches (five to ten
minutes, where your platform allows), and between stretches post one
line of status and reconcile your live children: list them, and chase
any that finished without reporting. A bounded stretch keeps nearly
all of a long wait's efficiency while guaranteeing a stuck or lost
child is noticed within minutes, not at the end of the session.

### 1. Dispatch the implementer

Record BASE (`git rev-parse HEAD`) before dispatching — the review package
and fix-wave diff need it.

- **Task brief:** before dispatching an implementer, run this skill's
  `scripts/task-brief PLAN_FILE N` — it extracts the task's full text to a
  uniquely named file and prints the path. Compose the dispatch so the
  brief stays the single source of
  requirements. Your dispatch should contain: (1) one line on where this
  task fits in the project; (2) the brief path, introduced as "read this
  first — it is your requirements, with the exact values to use verbatim";
  (3) interfaces and decisions from earlier tasks that the brief cannot
  know; (4) your resolution of any ambiguity you noticed in the brief;
  (5) the report-file path and report contract. Exact values (numbers,
  magic strings, signatures, test cases) appear only in the brief. Never
  make a subagent read the whole plan file.
- **Report file:** name the implementer's report file after the brief
  (brief `…/task-N-brief.md` → report `…/task-N-report.md`) and put it in
  the dispatch prompt. The implementer writes the full report there and
  returns only status, commits, a one-line test summary, and concerns.
- A dispatch prompt describes one task, not the session's history. Do not
  paste accumulated prior-task summaries ("state after Tasks 1-3") into
  later dispatches — a real session's dispatch hit 42k chars of which 99%
  was pasted history. The implementer needs its task, the interfaces it touches, and the global constraints. Nothing else. Whether to continue the same session or start a new one is the agent's judgment, made from how related the next work is and whether its context would be polluted.
- The dispatch carries the no-subagents contract (it is in the
  implementer template): the implementer never dispatches subagents —
  not helpers, and never a reviewer. Review arrives from you, after the
  report. In real sessions, every reviewer a worker spawned duplicated
  the task review the controller dispatched anyway — a full extra
  review seat per task.
- If an earlier task parked a finding in the area this task touches, carry
  a pointer to that ledger entry in the dispatch.
- Record the implementer's agent identity from the dispatch result —
  the fix wave resumes this session.
- Never run implementers in parallel within one plan — shared checkout and
  review gate. Parallel dispatch is reserved for work whose dependency
  analysis shows it is disjoint, with no shared state.

Template: [implementer-prompt.md](implementer-prompt.md)

### 2. Handle the report

Implementer subagents report one of four statuses. Handle each appropriately:

**DONE:** Generate the review package (`scripts/review-package PLAN_FILE BASE HEAD`, from this skill's directory — it prints the unique file path it wrote; BASE is the commit you recorded before dispatching the implementer — never `HEAD~1`, which silently drops all but the last commit of a multi-commit task), then dispatch the task reviewer with the printed path.

**DONE_WITH_CONCERNS:** The implementer completed the work but flagged doubts. Read the concerns before proceeding. If the concerns are about correctness or scope, address them before review. If they're observations (e.g., "this file is getting large"), note them and proceed to review.

**NEEDS_CONTEXT:** The implementer needs information that wasn't provided. Provide the missing context and re-dispatch.

**BLOCKED:** The implementer cannot complete the task. Assess the blocker:
1. If it's a context problem, provide more context and re-dispatch
2. If the task requires more reasoning, re-dispatch resuming the same implementing session (or a replacement session if it cannot be revived)
3. If the task is too large, break it into smaller pieces
4. If the plan itself is wrong, rule on the correction, ledger it, and re-dispatch with the ruling carried in the dispatch

**Never** ignore an escalation or retry without changes. If the implementer said it's stuck, something needs to change.

If the implementer asks questions — before starting or mid-task — answer
clearly and completely, provide additional context if needed, and don't
rush it into implementation.

### 3. Review the task

Per-block reviews are block-scoped gates. The broad review happens once, at the
final whole-branch review. Never skip the review, and never accept a
report missing either verdict — spec compliance AND task quality are both
required. Implementer self-review never replaces the task review; both are
needed.

**Reviewer session reuse:** Reuse one task-reviewer session across functional
blocks. Review is read-only with a small pollution surface; the benefit is
cross-block consistency. The reviewer and the implementer must be different
seats — the implementer never reviews its own block.

- Hand the reviewer its diff as a file: run this skill's
  `scripts/review-package PLAN_FILE BASE HEAD` and pass the reviewer the file path
  it prints (or, without bash: `git log --oneline`, `git diff --stat`,
  and `git diff -U10` for the range, redirected to one uniquely named
  file). The output never enters your own context, and the reviewer sees
  the commit list, stat summary, and full diff with context in one Read
  call. Use the BASE you recorded before dispatching the implementer —
  never `HEAD~1`, which silently truncates multi-commit tasks. Never
  dispatch the task reviewer without a diff file.
- **Reviewer inputs:** the task reviewer gets three paths — the same brief
  file, the report file, and the review package — plus the global
  constraints that bind the task.
- The global-constraints block you hand the reviewer is its attention
  lens. Copy the binding requirements verbatim from the plan's Global
  Constraints section or the spec: exact values, exact formats, and the
  stated relationships between components ("same layout as X", "matches
  Y"). The reviewer's template already carries the process rules (YAGNI,
  test hygiene, review method) — the constraints block is for what THIS
  project's spec demands.
- Do not add open-ended directives like "check all uses" or "run race tests
  if useful" without a concrete, task-specific reason
- Do not ask a reviewer to re-run tests the implementer already ran on the
  same code — the implementer's report carries the test evidence
- Do not pre-judge findings for the reviewer — never instruct a reviewer to
  ignore or not flag a specific issue. If you believe a finding would be a
  false positive, let the reviewer raise it and adjudicate it in the fix
  wave. If the prompt you are writing contains "do not flag," "don't treat X
  as a defect," "at most Minor," or "the plan chose" — stop: you are
  pre-judging, usually to spare yourself a fix wave.
The task reviewer may report "⚠️ Cannot verify from diff" items — requirements
that live in unchanged code or span tasks. These do not block the rest of the
review, but you must resolve each one yourself before marking the task
complete: you hold the plan and cross-task context the reviewer
lacks. If you confirm an item is a real gap, treat it as a failed spec
review — it enters the fix wave with the other findings.

Template: [task-reviewer-prompt.md](task-reviewer-prompt.md)

### 4. The fix wave

The wave triggers when the review reports spec ❌, any Critical or
Important finding, or a ⚠️ item you confirmed as a real gap.

Two kinds of finding never enter it:

- **Minor findings** go to the progress ledger as you go
  (`Task <N>: minor (deferred): <one-liner>`), and the final
  whole-branch review reads that list to triage which must be fixed
  before merge. A roll-up nobody reads is a silent discard.
- **A finding labeled plan-mandated** — or any finding that conflicts with
  what the plan's text requires — is yours to rule on: weigh the finding
  against the plan text, decide with the spec as the binding authority,
  and ledger the ruling before you act on it. Do not dismiss the finding
  because the plan mandates it, and do not dispatch a fix that contradicts
  the plan without a recorded ruling.

Everything else gets **one aggregated fix wave**: resume the same
implementing session (or a replacement session if it cannot be revived)
with the COMPLETE open findings list — not one fixer per finding.
Per-finding fixers each rebuild context and re-run suites. The implementer
fixes, re-runs the tests covering the amended code, appends its fix report
to the same report file, and returns the short contract. Before
re-reviewing, confirm the fix report contains the covering tests, the
command run, and the output; then run `scripts/review-package PLAN_FILE
FIX_BASE HEAD` (FIX_BASE is the head the previous review saw) and dispatch
[re-review-prompt.md](re-review-prompt.md) with the findings list, the
brief, the report file, and the printed diff path. The re-reviewer verdicts
each finding ADDRESSED or NOT ADDRESSED and flags new breakage in the fix
diff only. Name the covering test files in the fix message — a one-line
fix does not need the whole suite.

New Critical/Important breakage the re-review flags in the fix diff joins
the findings being adjudicated — park it with a ruling (it surfaces in your
final report) or rule-and-carry it like any residual.

**Adjudicate residuals; there is no second wave.** When the re-review
still leaves findings open:

- **The reviewer is wrong, or the point is contestable:** park it —
  `Task <N>: parked — <finding> — Ruling: <why the code stands>`. The final
  review sees both sides.
- **Real, but nothing downstream builds on it:** park it the same way, with
  a ruling that says it's real and deferred.
- **Real and load-bearing** — a later task builds on it, or it reveals a
  plan defect: rule on the smallest change that unblocks the dependent work,
  ledger it as `Task <N>: Ruling: <finding> — <what you decided and why>`,
  and carry it into the next task's dispatch. Only a defect that leaves
  every path forward a guess stops you for your human partner.

Every adjudication is a ledger entry — a silent discard is forbidden.
After the wave, append to the ledger:
`Task <N>: fix wave (<X> addressed, <Y> open — <finding one-liners>; commits <a7>..<b7>)`

Never fix findings yourself in the controller session — your context stays
clean for coordination, and controller fixes skip review.

### 5. Complete the task

When the review comes back clean — or every open finding is parked with a
ruling — append the completion line to the ledger in the same
message as your other bookkeeping:

- `Task <N>: complete (commits <base7>..<head7>, review clean)`
- `Task <N>: complete (commits <base7>..<head7>, <K> parked)` after
  adjudicated residuals

Then mark the todo complete and move on. Never move to the next task while
the review has open Critical/Important issues that are neither fixed nor
parked-with-ruling.

## Final Review

The final whole-branch review gets a package too: run
`scripts/review-package PLAN_FILE MERGE_BASE HEAD` (MERGE_BASE = the commit the
branch started from) and include the
printed path in the final review dispatch, so the final reviewer reads
one file instead of re-deriving the branch diff with git commands. Dispatch
the oracle seat (the fixed reviewer, per Model Selection), using
superpowers:code-review's
[code-reviewer.md](../code-review/code-reviewer.md). Point it at
the ledger's deferred-minor and parked lines so it can triage which must be
fixed before merge.

If the final whole-branch review returns findings, dispatch ONE fix subagent
with the complete findings list — not one fixer per finding.
Per-finding fixers each rebuild context and re-run suites; a real
session's final-review fix wave cost more than all its tasks combined.
Then run exactly one scoped re-review of the fix wave
(`scripts/review-package PLAN_FILE FIX_BASE HEAD` over the fix range,
[re-review-prompt.md](re-review-prompt.md)).
Adjudicate any residual findings as in the task loop's fix wave: park with
rulings, or rule on the load-bearing ones and ledger what you decided. Only
the classes at the top of this skill stop you here. There is no second fix
wave — residual load-bearing findings surface to your human partner in your
final report.

## Finish

Before you delete anything, collect every ledger line containing `Ruling:` —
preflight rulings, parked findings, fix-wave adjudications, all of them — into
your final message under "Rulings I made", in the order you made them, each
with what it costs if wrong. The list is exhaustive: if the ledger holds a
ruling, the list holds it. That list is the only place the decisions you
took on your human partner's behalf reach them — they read it and rework
whatever you got wrong. A ruling that dies with the workspace was a decision
made in secret.

When the final whole-branch review is clean,
delete this plan's workspace (`rm -rf <workspace>`) — the git history is
the record now. Sibling directories belong to other plans; leave them
alone.

Report completion to your human partner; merge/push/PR are done by the
human partner.

## Common Rationalizations

| Excuse | Reality |
|--------|---------|
| "Close enough on spec compliance" | Reviewer found spec gaps = not done. Fix or park with a ruling — those are the only exits. |
| "I'll fix it myself, dispatching is overhead" | Controller fixes pollute your context and skip review. Resume the implementer. |
| "The reviewer will just find something new anyway" | Scoped re-reviews verify fixes; they cannot wander. New findings on untouched code go to the ledger, not the wave. |
| "This finding is obviously wrong, I'll drop it" | Every ruling is a ledger entry. Silent discards are forbidden. |
| "The fix was small, skip the re-review" | Unreviewed fixes are how regressions land. Every fix wave ends with a scoped re-review. |
| "Reviews slow the loop down" | Execution without review is just unverified churn. Review is the loop's brakes and steering. |
| "Ledger bookkeeping is overhead" | The ledger is what survives compaction. Controllers without one have re-dispatched entire completed task sequences. |
| "The implementer spawned its own reviewer — free extra assurance" | It's a duplicate seat reviewing the same diff; the task review is the gate. A worker-spawned reviewer is a defect to flag, not rigor. |

## Example Workflow

```
You: I'm using Subagent-Driven Development to execute this plan.

[Setup: work in the current checkout]
[Read plan file once: docs/superpowers/plans/feature-plan.md]
[Resolve workspace: scripts/sdd-workspace docs/superpowers/plans/feature-plan.md — no ledger inside, fresh start]
[Create todos for all tasks]

Task 1: Hook installation script

[Run task-brief for Task 1; dispatch implementer with brief + report paths + context]

Implementer: "Before I begin - should the hook be installed at user or system level?"

You: "User level (~/.config/superpowers/hooks/)"

Implementer: [Later]
  - Implemented install-hook command
  - Added tests, 5/5 passing
  - Self-review: Found I missed --force flag, added it
  - Committed

[Run review-package PLAN_FILE BASE HEAD; dispatch the task reviewer with the printed path]
Task reviewer: Spec ✅ - all requirements met, nothing extra.
  Strengths: Good test coverage, clean. Issues: None. Task quality: Approved.

[Ledger: Task 1: complete (commits a1b2c3d..d4e5f6a, review clean)]

Task 2: Recovery modes

[Run task-brief for Task 2; dispatch implementer with brief + report paths + context]

Implementer: [No questions]
  - Added verify/repair modes
  - 8/8 tests passing
  - Committed

[Run review-package PLAN_FILE BASE HEAD; dispatch the task reviewer with the printed path]
Task reviewer: Spec ❌:
  - Missing: Progress reporting (spec says "report every 100 items")
  Issues (Important): Magic number (100)

[Fix wave: resume the implementer with both findings]
Implementer: Added progress reporting, extracted PROGRESS_INTERVAL constant.
  Re-ran test/recovery.test.js — 10/10 passing. Fix report appended.

[Run review-package PLAN_FILE FIX_BASE HEAD; dispatch scoped re-review]
Re-reviewer: Missing progress reporting — ADDRESSED (src/recovery.js:41).
  Magic number — ADDRESSED (src/recovery.js:7). New breakage: none.
  Verdict: all findings addressed.

[Ledger: Task 2: fix wave (2 addressed, 0 open; commits d4e5f6a..b7c8d9e)]
[Ledger: Task 2: complete (commits d4e5f6a..b7c8d9e, review clean)]

...

[After all tasks]
[Run review-package PLAN_FILE MERGE_BASE HEAD; dispatch the oracle seat (the fixed reviewer)]
Final reviewer: All requirements met. Deferred minors triaged: none block merge.

[Delete this plan's workspace — the record now lives in git]

Done! Report completion to your human partner; merge/push/PR are done by the human partner.
```
