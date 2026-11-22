# Review: the merge queue plan and the affected plan

Status: a review, written 2026-11-22 at main `eea997acb`. It is not a plan
and it rules nothing. It judges `notes/20261009-merge-queue-plan.md` and
`notes/20261012-what-a-change-affects.md` before either is built, and lists
what Willem would need to rule and what each plan would need changed.

## The prompt (Willem)

> I don't want to progress development of livery until the merge queue plan
> and the affected plan are implemented. But before I start that, i want a
> thorough review of their quality, performance implications, edge cases,
> problems, style issue, and fitting with the livery ethos. Willem,
> 2026-11-22

> the inititial implementation of affected only needs to support python on
> a per test file level, but I want to be able to support other languages
> in the future if possible, so that should be taken into account. Willem,
> 2026-11-22

> another lens to view this through: Does implementing this allow for other
> features I ahven't considered yet? Willem, 2026-11-22

## The answer in short

Neither plan can be built as written. Both designs are sound in what they
aim for, and both rest on facts about the code and the forges that are not
true today.

- **The merge queue plan** has 4 wrong statements under "What exists" and 9
  gaps that stop the design from working: who the queue runs as, which
  point a candidate's run is, how the required check is satisfied, where
  the queue's order and recorded heads are kept, and how a release gets
  through it. Each has a fix inside the ruled model. None reverses a
  ruling.
- **The affected plan** rests its whole proof on "the nightly's full run".
  The nightly runs 4 tests on one platform. Its proof cannot attribute
  reads for about half the test files. Measured for this review, its static
  closure selects a mean 24.2% of the suite on imports alone and 71.6% with
  the plan's own further rules, against 44.0% for today's package grain.
  The plan's 26.5% came from a selector the plan rules out. Its phases 1
  and 2 are sound and independent of that.
- **Both plans share three unbuilt things**: a proof record that chains in
  CI, a coverage record that can hold a candidate's or a test file's
  measurement, and metrics rows that record a verdict. Building those once,
  first, removes most of the cross-plan risk.
- **Both plans were written against a CI model that an unmerged design
  record proposes to replace**, and neither mentions it.

The order I recommend is at the end. The ethos the plans are judged against
is `docs/ethos.md` in this branch, with its sources in
`notes/20261122-ethos-sources.md`.

## How this was checked, and its limits

- I read both plans in full, the agent memory of Willem's rulings, the
  shard note, the Rust note, and two unmerged notes (below).
- Seven reading agents worked in parallel on 2026-11-22: one checked each
  plan's claims against the code, one measured the merge rate and the CI
  store, one measured what a static closure selects, one read the forges'
  documentation and Gitea's source, and three gathered the ethos.
- Code findings carry a file and line. Path prefixes: `W` is
  `packages/workshop/src/livery/workshop`, `F` is
  `packages/forge/src/livery/forge`, `G` is `packages/git/src/livery/git`,
  `C` is `packages/extensions/coverage/src/livery/extensions/coverage`,
  `PY` and `PT` are the python and pytest extensions' source directories.
- **Not verified live.** No forge behaviour was tested. Every forge fact is
  from documentation or source read that day, with the confidence the
  reader gave it. The live settings of `liveryhq/livery` were not read. I
  did not re-read every cited line myself, so a line gets re-read before
  anyone acts on it.

## What both plans depend on and do not say

### X1. An unmerged design record changes the CI model under both plans

`notes/20261023-core-vocabulary.md` and
`notes/20261021-forge-identities.md` exist only as untracked files in the
`docs-forge-identities` worktree
(`~/.local/share/footman/worktrees/livery/docs-forge-identities`). That
worktree's remote still names `willemkokke/livery`, so `fm start` cannot
ask the forge about it and keeps it. Nothing else protects the two files.

The vocabulary record carries rulings of Willem's from 2026-10-21 to
2026-10-23: `nightly` is renamed `scheduled`, a CI-host backend axis joins
`livery.forge`, an extension declares its own credential, jobs are derived
from tasks. It proposes that points become triggers with an `access` of
`minimal`, `write` or `admin`, and that code from a proposed change gets
`minimal` access only. It says of these two plans:

- "The merge queue (deferred): its advance runs from the gate's verdict,
  untrusted under this design, and it needs a trigger for GitHub's
  `merge_group` event; both are revisited when it returns."
- "Reshapes destination phase 16: its phase 1 gives a CI job a `reads`
  spelling; with derived jobs it targets tasks, or follows the CI model."
- A merge queue is listed as verification policy of a workflow, and the
  workflow is proposed to move out of the core into an extension.

The merge queue plan was rewritten on 2026-11-21 against points, jobs and a
gate job that posts statuses. The affected plan says "the nightly point"
throughout. Neither mentions the record.

**To rule:** build both plans on today's CI model, or after the rename and
the CI model. If today's model: the queue's policy lives in its own module,
as `_shard.py` does, so a later move is a move.

### X2. The queue needs an identity that no CI job has today

An event caused by a job's own token starts no workflow, on all three
forges:

- GitHub: a push, a label change or a merge made with `GITHUB_TOKEN`
  creates no run. `workflow_dispatch` and `repository_dispatch` are the
  exceptions. (GitHub's documentation, high confidence.)
- Gitea: every event whose doer is the Actions user is dropped
  (`services/actions/notifier_helper.go`, "avoiding triggering
  cyclically"). (Source read, high confidence.)
- GitLab: a job-token push triggers no pipeline, needs a setting that is
  off by default, and the job token cannot label, merge or post a status.

The gate job runs with the job's token today
(`.github/workflows/ci.yml:131`, `W/_points.py:269`), and declares no
`pull-requests`, `issues` or `statuses` permission (`ci.yml:103`). The code
already states the rule: "A pull request a scheduled task opens with the
job token starts no workflow" (`W/_points.py:334`).

So `fm queue --update`, run where the plan puts it, would push candidates
that start no run, remove labels that start no run, and merge pull requests
whose push to main starts no run. With no push run on main there is no
release dispatch, no pages deploy and no govern.

The plan's open item 6 asks which permissions the token needs. The question
is which identity the queue is, and that is the forge identities plan's
subject. **The merge queue has that plan, or a part of it, as a
prerequisite.**

### X3. CI's record has no chain rule

Both plans rest on "the record's chain rule". The merge queue plan: "By the
record's chain rule the row is a proof once k-1's is." The affected plan's
contract 1: "The chain rule is unchanged when the unit moves from the
package to the test file."

The chain rule exists in the local `gate-record` series only
(`W/_gate_record.py:245`), which "never leaves the machine. CI never reads
or writes it" (`W/_gate_record.py:28`). CI's `verified` series works
differently:

- A narrowed run writes a row only when its base tree already has a full
  row at stamp time, and the row it writes is itself `scope: full`
  (`W/_verified.py:384`). "Without a full row for the base, a narrowed run
  stamps nothing" (`W/_verified.py:32`, `:499`). Nothing stamps it later.
- `leg_plan` accepts as a base only a tree that already has a full row
  (`W/_gate_record.py:432`).
- The verified row and the stamp's argument are package-grained
  (`W/_quality.py:832`, `W/_verified.py:27`).

So candidate k, narrowed from a candidate k-1 that is still running, stamps
nothing, and nothing stamps it when k-1 goes green. A test-file selection
has no row shape to be recorded in.

**Suggested:** one piece of groundwork, before either plan: the proof rule
of CI's record, stated once for a tree, for a chain of trees and for a unit
finer than a package, with its pinning tests. This is also where the first
possibility under "What the two plans make possible" is decided.

### X4. The coverage record has no home for either plan's measurements

- A candidate's run is a push event. The union then targets main's record
  (`C/_union.py:510`) and declines because main's head differs
  (`C/_union.py:480`). Records exist for main and for pull request branches
  only (`C/_record.py:978`). So candidate k's legs measure every suite
  whose closure differs from main's record, which includes the suites of
  every entry ahead of it. "Each entry then pays for its own change alone"
  does not hold for the test step.
- After a candidate lands, main's push run finds the tree proved. The row
  names no branch, because the head ref is empty on a push
  (`W/_verified.py:466`), so its legs measure the changed suites again
  (`C/_leg.py:622`). The phase 3 acceptance "runs no leg" does not hold.
- The stored coverage row keeps contexts per line, and branches without a
  context (`PT/_coverage.py:395` against `:417`). It cannot say which test
  file took a branch. The affected plan says "the contexts and the ctrace
  core already make the split complete; the store's keying and union
  change". The row format changes too, or each test file is reduced again
  from raw data.

Both plans re-key the coverage record. Doing it once, for "a unit measured
on a tree that is not main's and not a pull request's", serves both.

### X5. The metrics rows cannot measure either plan

Read from `refs/workshop-origin/metrics` on 2026-11-22 (300 rows,
2026-10-14 to 2026-11-22):

- A row has no verdict. The gate job's state is `running` in all 148 pull
  request rows, because the row is written before the verdict.
- A red sharded leg writes no row at all: a red shard leaves no ref, the
  fold refuses, and the job stops before the collect. (From reading
  `W/_points.py` and `W/_fold.py`. Not observed, since a missing row leaves
  nothing to observe.)
- A re-run keeps its run id and overwrites its row. There is no attempt
  field.
- A row has no pull request number, no point and no base.
- The window is 300 rows over all events. It held 4 busy days.

The merge queue plan's open item 5 ("how often a pull request goes red on
Linux or Windows after a green local gate, read from the metrics store
before phase 2") cannot be answered: the store shows 2 red rows of 148,
both ubuntu-only, and it cannot see a macOS or Windows red since
2026-10-17. The affected plan's four measures need the same fields.

The verified series keeps 200 rows by count (`W/_verified.py:57`). Each
green candidate adds a row for a tree that may never be main's. After 200
such rows main's tip row is gone, and a push pays the full gate.

**Suggested:** add the verdict, the attempt, the point, the entry and the
base to the row, write a row for a red run, and size the windows for
candidate rows, before either plan needs the numbers.

## The merge queue plan

### Statements under "What exists" that are not true

| Plan says | What the code does | Where |
| --- | --- | --- |
| A narrowed run's row "is a proof when that tree is proved" | True of the local record only. See X3 | `W/_verified.py:32`, `:499` |
| A push whose tree a run proved "runs no leg" | The check jobs start (the matrix is static), each returns early with scope `verified`, and coverage can still run suites. Such a run costs 5.9 minutes of wall and about 20 runner-minutes | `W/_quality.py:682`; `C/_leg.py:622`; measured below |
| The gate workflow "picks its matrix in `fm ci.run`" | A GitHub expression in an emitted `legs` job picks it. This repository emits no such job | `W/_ci_generate.py:579` |
| "Each forge requires the `gate` context" | GitHub requires `gate`. Gitea requires `ci / gate (pull_request)`. GitLab has no required context, and its `Checks.status` reads the newest pipeline and no commit status | `W/_workflow_tasks.py:199`; `F/_gitlab.py:609`, `:1254` |
| "No verb writes main today" (contract 14) | `fm new.project` pushes `main` of the new repository. No verb writes this repository's main | `W/_new_project.py:533` |

The measurements in the table are from 2026-10-09. Current figures, sharded
since 2026-10-17: a pull request run p50 11.3 min and p90 17.0 (n=46); a
push run p50 7.2 and p90 18.6 (n=37). 74 of 113 push runs (65.5%) found
their tree proved. Two days exceed the stated peak of 37 merges: 62 on
2026-10-11 and 49 on 2026-10-17. Main had 8 commits in the 30 days to
2026-11-22.

### Gaps that stop the design from working

**Q1. Who the queue runs as, and what starts each step.** See X2. The
design needs to name the identity, and say that a candidate's run and a
dropped entry's pull request run are started by an explicit dispatch where
the forge starts nothing. One more forge fact bears on that: Gitea drops,
for pull requests, a status whose context ends in `(workflow_dispatch)`
when the Actions user made it.

**Q2. A candidate's run is the merge point today.** Every push event is
the merge point (`W/_points.py:1891`). A run on a `queue/` push would
dispatch a release, run the janitor and the govern job, and deploy the
pages (`W/_points.py:1331`, `ci.yml:141`, `:165`). `verify_points` refuses
two points in one workflow file that share an event (`W/_points.py:429`).
The plan needs a point of its own for a candidate, and says nothing about
one.

**Q3. Trust.** The plan says in one place that the gate job posts `gate`
on each entry's head, and in another that `fm queue --update` does. The
gate job of a candidate's run executes the candidate's code. A token that
can post `gate` and merge, held beside that code, lets a queued change land
any pull request. The 2026-10-11 ruling on the admin token has the same
reasoning: a job that runs the checkout's own code must not hold it. The
step that posts and merges has to run main's code.

**Q4. The posted status does not satisfy the required check as designed.**

- GitHub: "If a check and a commit status have the same name, both must
  pass." A pull request run that the queue cancelled leaves a cancelled
  `gate` check run, and the posted `gate` status beside it does not unblock
  the merge. A job skipped by `if:` counts as passed, so a skipped `gate`
  job would open a way around the queue. (High confidence.)
- Gitea: the required context is another string (table above).
- GitLab: with merge request pipelines present, only those count toward
  "Pipelines must succeed". A posted status can land in a separate
  `external` pipeline and be ignored. (Medium confidence.)
- `_refuse_unreported_contexts` builds the reported names from runs and
  jobs (`W/_submit.py:95`), so a posted status reads as a context CI never
  reports.

The fix inside the model: the queue's status has a name no job has, and
"the forge refuses a merge the queue did not prove" becomes a promise of
`livery.forge` that each backend holds by its own means, probed, with a
conformance scenario. Phase 1's `posted-status` scenario expects the
cancelled case to pass. On GitHub it will not.

**Q5. The queue's order and its recorded heads have no source.** The plan
says `fm queue --update` "keeps no state of its own", and also that "the
queue records each entry's head". `PullRequest.labels` is names only
(`F/_types.py:264`), and no label event is read. The order by label time
needs a new read on each forge (GitHub's issue events, Gitea's timeline,
GitLab's `resource_label_events`), and phase 1 lists none. A natural record
exists: each candidate commit carries its entry's number and head as
trailers, so the `queue/` refs are the record. The plan should say so.

**Q6. Two updates at once.** "Idempotent" does not cover two runs at the
same moment, and CI starts one per event. No verb is serialised today: the
only concurrency group is the pages job's (`ci.yml:174`). `merge_now` takes
no expected head (`F/_protocol.py:147`), and the GitHub and Gitea backends
send none, so a push between the queue's head check and its merge lands the
new head. Every write needs a condition: the merge with the recorded head
(all three APIs take one), the candidate push with a lease, and one update
at a time held by a compare-and-swap in the state store.

**Q7. A release cannot pass through the queue as designed.**

- `check-title` and `check-fresh` read the pull request payload and print
  "not a pull request" on another event (`W/_release_driver.py:1615`,
  `:1669`).
- `check-fresh` compares against `origin/<base>`, which lacks the entries
  ahead. The publish then refuses after the merge (`W/_publish.py:153`).
- The squash message must carry the pull request's body: `Mined-At` is
  read from it (`W/_publish.py:140`). The plan's landing passes the title
  only.

Contract 14 says a release goes through the queue. For that, a release
entry is urgent, the landing passes the body, and a candidate's run knows
its entry.

**Q8. Nothing runs the queue when no event arrives.** Contract 6 says "the
next event runs it again". GitLab starts no pipeline on a label or draft
change. Gitea has no draft events: a title change arrives as `edited`,
together with body changes. The emitter writes `opened, synchronize,
reopened, labeled` and no other type (`W/_ci_generate.py:1020`). A lost
event leaves the queue standing with no line printed anywhere. The
development workflows note already has the answer: "Polling, not webhooks,
is the v1 observation model everywhere". A scheduled update is the
baseline, and an event makes it faster.

**Q9. A red run and a run that did not finish read the same.**
`CheckStatus` has four states (`F/_types.py:70`). GitHub's backend reads
every completed conclusion other than success as failure, cancelled
included (`F/_github.py:1136`). A candidate whose run was cancelled by a
rebuild, lost its runner or timed out would be dropped as red, and at
depth d a wrong drop costs d runs. The design needs "ran and failed" apart
from "did not finish", and a stated answer for a flaky red.

### Edge cases the plan does not name

1. **`merge=union` and render receipts.** `notes/*.md` merge by union
   (`.gitattributes:14`). GitHub ignores user attributes on its merges
   (medium confidence, from a relayed support answer), and Gitea's conflict
   check ignores them (source read). Receipt conflicts are resolved on the
   client only (`W/_sync.py:57`). Two queued note changes build a clean
   candidate locally, and the forge then refuses the squash. 15.7% of the
   636 commits in the 60 days to 2026-11-22 were `docs:` changes to
   `notes/` alone. Build the candidate the way the forge merges, with no
   attributes, so a conflict is named before a run is spent.
2. **The forge refuses the landing for another reason**: a missing review,
   an open thread, or "not mergeable yet, try again later" (Gitea's 405).
   `merge_hold` exists. The plan covers a conflicting rebuild only.
3. **When a candidate is stale.** After entry 1 lands, candidate 2's parent
   is a commit main does not have, with an equal tree. Staleness judged by
   ancestry rebuilds every candidate at every landing. It has to be judged
   by tree.
4. **GitLab.** Governance writes no merge method (`F/_gitlab.py:576`), so
   the instance default applies, and merge commit plus squash puts two
   commits on main, against contract 3. `create_repo` removes the default
   branch protection (`F/_gitlab.py:390`), against contract 4. The rebase
   API is asynchronous and changes the head, which voids the entry by the
   plan's own rule and orphans a posted status.
5. **Drafts.** GitHub offers no drafts in private repositories on Free or
   Pro. Gitea has no draft events. `is_armed` is read as "will this land"
   in the verdict classifier (`W/_verdict.py:615`, green and not armed
   exits 11), the follow, the workflow engine and `disarm_before_push`. The
   plan lists none of these readers. One question on `livery.forge`, "is
   this pull request queued", would serve them all.
6. **`fm submit.merge` refuses a head behind main**
   (`W/_submit.py:1637`). `--bypass-queue` meets that refusal.
7. **An entry that changes the queue's own code or a workflow file.** The
   candidate's run uses the candidate's files, the update uses main's. On
   GitHub a job-token push that changes `.github/workflows/` is refused.
8. **A pull request closed, merged by hand or retargeted while queued; an
   entry whose squash is empty; an urgent entry that arrives while a
   landing is in progress.**
9. **`queue/` as a branch prefix.** No registry of reserved prefixes
   exists. `workflow/` is tested ad hoc in six places
   (`W/_workflow_state.py:29` and five more). A second reserved prefix is
   the moment for one function.
10. **The `checked-push` scenario.** A direct push of a commit with green
    checks is refused whenever "require a pull request" also applies, and
    contract 4 says no token moves protected main. The plan does not say
    what the scenario is for.

### GitHub's native queue against the promises (phase 4)

All from GitHub's documentation, read 2026-11-22.

- A pull request must pass its required checks on its own head before it
  can enter the queue. "Opened and queued at once, a pull request gets no
  pull request run" cannot hold there without a status on the head.
- The queue's squash message cannot be set per merge. It follows the
  repository's default, which governance does not write
  (`F/_github.py:489`). Contract 3 and `Mined-At` then depend on a setting
  nothing asserts.
- GitHub builds one group per entry and has no "run only the last of a
  group". Batching as the plan defines it cannot be expressed. The plan
  allows for that.
- `grouping_strategy` must be `ALLGREEN` for contract 2.
- Available for public repositories of an organization, and for private
  ones on Enterprise Cloud only.
- Membership would have two sources, the label and GitHub's queue. A
  person who clicks "Merge when ready" is queued with no label.
- `enqueuePullRequest` takes `jump` and `expectedHeadOid`, which answers
  open item 7. Who may jump outside Enterprise is not documented. A REST
  endpoint `merge-async` with `merge_action = merge_queue` was reported as
  generally available since 2026-09-30.

Willem said he will turn the native queue on for `liveryhq/livery`. Then
phases 2 and 3 are never run by this repository's own merges. Running the
workshop's queue here first is the "livery uses what it builds" proof.

### Performance

Measured 2026-11-22 from the commits on main and the metrics store.

| What | Value |
| --- | --- |
| Entries ahead of a merge when it queues, run of 11 min | 0: 75.2%, 1: 20.1%, 2: 3.1%, 3 or more: 1.6% (n=636, 60 days) |
| The same, run of 16 min | 0: 64.6%, 1: 25.8%, 2: 6.6%, 3 or more: 3.0%; mean 0.50; max 8 |
| A run that ran no leg | p50 5.9 min, about 20 runner-minutes (n=23, sharded) |
| What sets that floor | docs job 245 s, gate job 110 s, 11 jobs each paying setup |
| Wait for a macOS runner with 3 or more runs open | p50 55 s, p90 824 s (n=59) |
| Pull requests with 2 or more heads | 27 of 117 (23%) |
| A pull request run, runner-minutes, sharded | full p50 59.5, affected p50 51.9, prose only p50 19.2 |

The depth model assumes landings would not move under a queue, one run of
constant length, and no red.

What follows:

- **The gain** is the 34.5% of push runs that ran legs today (p50 14.3 min
  when narrowed), which the queue removes once X4 is fixed.
- **Every candidate in parallel** meets the macOS runners first. A run uses
  3 macOS shards. The default concurrency (open item 3) should be read from
  the wait measurement, and the data says 1 or 2.
- **`fm queue --update` must be one light job.** If it rides the workflow
  it rides today, each event costs the 6 minute floor, and a landing has 5
  to 8 events.
- **A red entry voids every candidate behind it.** At this repository's
  depth the cost is small. At a depth of 20 with a few percent red it grows
  with the square of the depth. The second possibility below removes most
  of it.
- **A pushed head voids its entry**, and 23% of pull requests were pushed
  again after their first run.
- **Label times** cost one request per entry per update.

### The merge queue plan against the ethos

- **Say what is not covered.** Contract 2 ("Every commit on main is a tree
  a run proved before it landed") has three exceptions the plan itself
  describes: a tree mismatch, a recovery merge, and a batched group's inner
  commits. The contract should name them.
- **CI decides what runs.** Open item 5 asks "whether a rule asks platform
  changes to run `fm submit` and its pull request run before queueing".
  That is the author knowing what the change touches, which the 2026-10-18
  ruling refuses. The optional form the plan already describes is fine.
- **Small phases.** Phase 2 holds the label, the draft default, the
  candidate builder, the lander, the red rule, the `fm ci.run` changes, the
  status, two verbs, urgent entries, the recovery merge and the docs. Its
  acceptance is two lines, and neither proves a landing. It needs to be
  five or six phases, with a shadow step among them: build and run
  candidates beside today's auto-merge, compare the trees, land nothing.
  The affected plan's contract 7 states that value and this plan does not
  use it.
- **Shared resources.** The plan does not say whether the queue is a
  setting or the only path, or what a project that upgrades the workshop
  gets. `[ci] merge-queue = true` means "GitHub's native queue" in phase 4,
  which reads as "no queue" when false.
- **One way.** Membership by label and by GitHub's queue is two truths.
  `--armed` means two different acts by forge.
- **A difference is a named capability.** Drafts, the status write, the
  branch update and the queue each need a `supports()` answer and a
  refusal by name. The plan probes the queue only.
- **Proved by running it, read from the outcome.** Phase 3 pins printed
  lines. The 2026-10-15 ruling prefers a store row or a run's conclusion,
  and the loop asserts rows today (`W/_e2e.py:2564`). `proved by run N` is
  the line of a narrowed step. A skipped gate prints "proved green by run".
- **The loop as it is.** Its default environment has one runner, so two
  candidates cannot run in parallel there. `_completed_run` fails on a red
  run. No schedule runs the `all` set. The fake forge has no commits or
  trees (`F/testing/_fake.py:636`), so `squash-tree` on the fake needs a
  tree model first.
- **Fakes and simulated time.** Willem asked for both on 2026-11-20. The
  order by label time needs a clock on the fake.
- **Arming is its own verb pair** (ruled 2026-10-17). The plan adds
  `--urgent` to `fm submit` alone. `fm submit.arm --urgent` is missing.

### Style of the merge queue plan

- Line 197 reads "the `fm queue --update` lands each entry", an article
  left from the rename. Lines 303 and 449 to 452 are broken mid-sentence.
- The 2026-10-09 and 2026-10-13 decision entries point at "open item 6"
  and "open item 7", which are other questions now. The house keeps closed
  items numbered so earlier references stay readable.
- "What every forge promises", item 3, says every candidate runs. Batching
  says otherwise in the same note.
- The plan has no "what this does not cover" under any phase.
- Words the standards ask to avoid, in a note that is exempt: "plumbing",
  "unstick", "pays for its own change".
- "Read in the code on 2026-11-21" heads a list with four wrong entries. A
  note states the current state or it is wrong.

## The affected plan

### Statements that are not true today

| Plan says | What the code does | Where |
| --- | --- | --- |
| "The nightly's full run is the proof's only full run" (contract 8) | The nightly runs pytest under `WORKSHOP_POINT=nightly`, which deselects every test not marked for it. The 2026-11-21 nightly ran 4 tests on the 3.14 leg. It runs on ubuntu only | `W/_pytest_points.py:28`, `:102`; `.github/workflows/nightly.yml:33` |
| `run_suites(selection=...)` passes the selection to pytest | It left on 2026-10-14. The pytest step asks the `python.test.runner` query | `PT/_steps.py:69`; `PY/_backend.py:303` |
| "The module graph reads that answer" | No module graph exists | no hit in any source tree |
| "The test-file phases fill the map; nothing downstream changes shape" | A package's presence in `PhaseContext.tests` means "only its tests changed". Package steps skip on it, the examples skip on it, the CI leg drops the map, and an empty tuple means the whole directory | `W/_steps.py:719`; `PT/_steps.py:152`; `W/_quality.py:346`; `PY/_backend.py:344` |
| The rule codes a tool skips "stay in its configuration fragment, where they are written today" | They are written in `claims[].ignore` and rendered into `ruff.toml` | ruff's `extension.toml:50`; `W/_steps.py:601` |
| "The invalidate-all list from the root attribution" | No list exists. The rule is the reverse: a path outside every package runs everything unless it is exempt or explained | `W/_graph.py:303`; `W/_root_attribution.py:117` |
| A closure "computed from the tree alone" resolves a third-party module to its distribution | That mapping comes from the installed environment. `uv.lock` holds no module names | `PY/_installed.py:93` |
| "The footprint is a second reading of data the full run already collects" | File opens are collected nowhere. Lines run in a child or at import time land in the empty context | `PT/_coverage.py:86` |

Two names are taken. `reads` is a step key that names the context keys a
step reads (`W/contract.toml:830`), used by the python, conan and nanobind
extensions. `footprint` is `fm ci.footprint` and
`livery.workshop._footprint`, the runner-minutes report.

The engine also asks a `reaches` query (`W/_graph.py:351`), which python
answers for pytest plugin modules. The plan does not mention it.

### Problems in the design

**A1. The proof has no run to stand on.** Phases 3, 4, 8 and 9 and the cost
argument of 2026-10-13 assume one full run a night. A full, metered run of
the whole suite has to be added: about 620 s of wall on ubuntu, 7,731
tests. It has to run on every platform the selection is used on, since 41
test files branch on the platform and the coverage union is judged over
three. "One nightly job audits, since the selection is
platform-independent" holds for the selection. It does not hold for the
footprint or for an escape. Phase 3's acceptance names a Windows leg the
nightly does not have.

**A2. The static closure misses this repository's own exports.** Every
large root `__init__.py` imports nothing at import time and serves its
public names through `__getattr__` and a table of strings (forge: 53 names
over 8 modules, workshop: 111 over 29). `from livery.forge import
Repository` reaches `_protocol.py` at run time and through no import
statement. Following imports, a closure stops at `__init__.py`. Source 5,
dotted string literals, then adds every module the table names, for every
test that imports any name from the root. This is the ruled convention for
every root, so a project that follows it has the same hole. A sound and
narrow rule exists: the `TYPE_CHECKING` imports in the root say which
module each name comes from, and python's `references` answer can resolve a
name through them. The plan names no such rule. The file its phase 5
acceptance uses imports forge this way.

The effect of that rule on what is selected is small: 24.2% without it,
23.8% with it (the measurement below). The hole is one of soundness, and
only if a closure follows runtime imports alone.

**A2b. Measured: the static closure selects far more than the plan's
figures, and with the plan's own rules more than today's package grain.**
The plan's 61.2% (package) and 26.5% (file) came from what tests executed.
Nobody had measured the closure the plan builds. Measured on 2026-11-22
with a script over the last 300 commits on main, each commit against its
own tree, for the 53 commits that change only Python files in packages,
weighted by test functions:

| Selector | Mean share of the suite | Median |
| --- | --- | --- |
| Package grain, as the engine does it today | 44.0% | 38.5% |
| File grain: every import statement and the conftest chain | 24.2% | 37.2% |
| The same, plus what the pytest plugins import (source 2) | 62.6% | 100% |
| The same, plus the mount rule and string literals (sources 3 and 5) | 71.6% | 100% |

- With imports alone, file grain selects 90% or more of what package grain
  selects in 28 of the 53 commits, and under 10% in 14, of which 8 change
  only test files.
- The median test file's closure holds 81.6% of its own package's source
  modules. The cause is imports inside function bodies that form cycles:
  3,861 first-party imports are inside functions against 1,974 at top
  level. One cycle holds 68 of the workshop's 103 modules, another 35 of
  footman's 47. A test that reaches one member reaches all of them.
- The pytest plugins and what they import reach 185 of 284 source modules.
  That edge is real: `livery.footman._pytest_plugin` has autouse fixtures
  that import footman's internals in every test of every package. With it,
  file grain selects more than package grain in 37 of the 53 commits.
- The script does not model children, data files, the lock or the
  invalidate-all list. A sound selector selects more than these figures.
- 122 of the 300 commits change a TOML or template file inside a package.
  Those need `[tests] reads` or take a coarser grain. 101 of 300 are
  decided by the import closure alone.
- The weights are test-function counts and the plan's were test time, so
  only the ratios compare: file over package is 0.72 static on the plan's
  own window, against 0.43 for the plan's figures.

So the selection phases, as designed, would cost eleven phases and select
about what the package grain selects today, or more. The finer grain is
more honest than today's: the plugin edge is real and today's engine
widens only for a plugin module's own file. The saving the plan promises
comes from what a test executes, which contract 2 rules out as a selector.

**A2c. What causes the width, and what moves it.** A second measurement
the same day, on the same 53 commits, after Willem asked whether the
figures can be improved. Shares by test functions, with the share of
linux test seconds in brackets (durations read from the store's
`durations` series).

| Selector | Mean share |
| --- | --- |
| Package grain, today | 44.0% [56.5%] |
| File grain, imports and conftest chain | 24.2% [32.6%] |
| Plus the plugin rule, a plugin's own file only (today's engine) | 31.0% [39.2%] |
| Plus the plugin rule, everything a plugin can import | 62.6% [62.9%] |
| The same after removing one import, `_pytest_layout.py:34` | 44.0% |
| The same after removing three imports | 38.8% |
| File grain, imports alone, after removing 17 imports inside functions | 20.0% [27.2%], median 10.3% |
| The plan's full rules, lazy roots resolved, 21 imports removed | 43.2% [49.3%] |
| The same without the mount rule | 33.7% [38.6%] |
| Import-time imports only (unsound, the floor) | 7.6% [8.5%] |

- Weighted by time, the static closure on imports selects 32.6% against
  the plan's 26.5% from executed tests, and package grain 56.5% against
  the plan's 61.2%. So on imports alone the static figure is close to the
  plan's.
- The jump to 62.6% is one line. `offenders()` in
  `packages/workshop/src/livery/workshop/_pytest_layout.py:34` imports
  `package_directories` from the workshop's root, which resolves into the
  57-module cycle. That line alone carries 26 points.
- One edge splits the workshop's cycle most: `wire_member()` in
  `_templates.py:538` imports `sync_tools` from `_tool_tasks`. It is worth
  1.9 to 3.0 points alone.
- The rest is spread thin. 1,389 import statements sit inside functions
  outside the tests, and the median source module is reached by 53% of
  test files through such imports against 1.5% at import time (workshop).
  After the best 17 to 20 edges no single edge helps. No short list of
  edits reaches the floor.
- A workshop-only change, 41 of the 53 commits, falls into two groups: 17
  select 0.1% to 8.2% at file grain, and 24 select 36% to 45%, within one
  point of package grain.
- Imports inside functions are the house's way to keep start-up fast and
  unused features free. A static reading must count each one for every
  test that reaches the module. What a test executed does not have that
  limit, which is what the footprint measurement checks.

**A2d. Measured: what a recorded footprint selects, and what recording it
costs.** A third measurement the same day. Every one of the 332 test files
ran in a pytest process of its own, twice, with a hook loaded into every
Python process that recorded the first-party modules it imported and the
repository files it opened. The commits were then replayed against those
footprints. Footprints exist for the tree at HEAD only, so changed paths
that no longer exist are skipped (32 of 200 for the Python-only commits),
and old commits are judged by today's reads.

| Commits | Package grain | Static, imports | Static, with plugin rule | Footprint |
| --- | --- | --- | --- | --- |
| Python files only (45), by test functions, mean / median | 42.9% / 45.8% | 25.3% / 35.3% | 51.6% / 35.3% | 15.9% / 9.9% |
| The same, by CI seconds | 56.3% / 62.2% | 33.8% / 22.8% | 52.1% / 22.8% | 24.6% / 21.6% |
| Also TOML or templates in packages (41) | 45.7% / 45.8% | 35.1% / 45.1% | 73.9% / 100% | 50.6% / 44.1% |

- On Python-only commits the footprint selects under 25% of the suite in
  38 of 45, and everything in 2. Those 2 change `livery.git` modules that
  a plugin loads into every pytest process.
- **Exact attribution costs 2% here.** One process per test file, 4 at
  once: 199.2 s of wall. One session over the same files with 4 workers:
  195.3 s. The hook adds 2.8% to a long file. A file with one test starts
  in 268 ms. The plan set this aside as too expensive without measuring
  it. Not measured on a CI runner, and Windows starts a process more
  slowly.
- **The footprint is stable.** Of 331 files run twice, the imported
  modules differ in none, and one file differs by a single text read.
- **The plugins load 33 source modules at run time.** The static plugin
  rule reaches 185. A source module is in a median 9.2% of footprints
  against 54.5% of static closures.
- **The static closure on imports misses 6,575 pairs of test file and
  module** that ran. 5,741 are behind the pytest plugin entry point, 411
  behind `importlib.import_module` in `_declaration.py:resolve`, where an
  extension names a step's code by a dotted string, and 286 ran in a child
  process only.
- **For TOML and template changes the footprint selects about what
  package grain does, and more in 13 of 41 commits.** 164 test files read
  all 19 `extension.toml` files and 163 read the workshop's
  `contract.toml`. Today's package graph does not have those edges: a
  change to one extension's declaration does not run the workshop tests
  that mount it.

What the hook does not see, each of which makes a footprint too small:

- a Python child whose environment lost the hook (12 test files);
- what a native program reads: 119 test files start one, git in 95;
- a check that a file exists, which opens nothing;
- a file added to a directory a test lists, since a file that did not
  exist is in no footprint;
- the installed environment, which stands between `uv.lock` or a
  `pyproject.toml` and what a test imports;
- a test that was skipped, and any platform but the one it ran on.

**What this changes.** Recording costs almost nothing, so every run can
record, and the row that proves a unit can hold what the unit read. The
record by unit and the selector are then one thing: a unit runs again when
a file its last green run read has changed. The proof is taken on every
run, not once a night. The static graph stays for a unit with no row yet,
for printing why a unit was selected, and for saying before work starts
whether two streams overlap. The plan's phases for declared data, string
literals and the invalidate-all list shrink to what the hook cannot see.

Two anomalies the run met. `test_workshop_sync.py:566` calls
`sync_workspace(ROOT)` on the real checkout and opens 28 tracked rendered
files for writing on every run, with unchanged content. And one file
failed once because its pytest base temporary directory vanished while
pytest of other repositories ran on the same computer.

Three ways forward, for Willem:

- (a) **Keep the static selector and narrow the code.** Report closure
  width as a measure, and cut the cycles and the plugins' imports. Does not
  cover: the saving depends on refactors of the workshop and footman that
  no plan holds, and a change to a module the plugins reach still selects
  everything.
- (b) **Select in CI from the recorded footprint, inside the static
  closure as the bound.** A test's verdict can change only when a file it
  read changes, so a complete footprint is a sound selector. It gives the
  plan's figures. Does not cover: a fresh clone and the local reflex, which
  keep the static answer; it makes A4 a question of soundness, so exact
  attribution is required; and it narrows contract 2 to the local reflex,
  which is a reversal of that contract for CI and should be ruled as one.
- (c) **Stop at the item graph.** Build phases 1, 2 and 5, keep package
  grain for selection, and use the graph for the features under "What the
  two plans make possible". Does not cover: the prompt's goal of only ever
  running affected tests.

Recommendation, changed after A2c and A2d: (b). The numbers the item
graph's replay was to give exist now. The footprint selects 15.9% where
the sound static selector ends near 34% to 43%, recording it costs 2%, and
it is stable. (a) still pays as a second step: the three imports of A2c
are cheap and narrow every selector.

The script and its output are in the session's scratch directory and are
not kept. The measurement is repeated by phase 5's own replay.

**A2e. Measured on hosted runners: a process per test file costs 3% to 4%
when the longest file starts first.** A fourth measurement the same day,
on GitHub's hosted runners for ubuntu, macOS and Windows. One job on each
ran all 332 test files, 7,773 tests, with as many processes at once as the
session has workers: 4, and 3 on macOS. The passes ran back to back in one
job: one session (A), a process per file in list order (B), the same with
the hook (C), one session again (A2), and a process per file with the
longest started first (B2). The table is the third push's run, the one
where every pass is valid on Windows.

| Wall, seconds | Windows | ubuntu | macOS |
| --- | --- | --- | --- |
| A, one session | 737.5 | 250.1 | 352.4 |
| A2, one session again | 683.3 | 231.9 | 426.3 |
| B, a process per file, list order | 773.7 | 291.0 | 441.4 |
| C, B with the hook | 801.5 | 304.1 | 496.7 |
| B2, a process per file, longest first | 730.8 | 250.3 | 480.3 |
| B against the mean of A and A2 | +8.9% | +20.7% | +13.4% |
| B2 against the mean of A and A2 | +2.9% | +3.8% | +23.4% |
| C against B | +3.6% | +4.5% | +12.5% |
| One more pytest process, median | 1.06 | 0.61 | 0.52 |
| A bare interpreter, median | 0.046 | 0.018 | 0.024 |
| pytest on a file with one test | 0.667 | 0.408 | 0.379 |

- **The order of the files costs more than starting the processes.** In
  list order `tests/test_workspace_provenance.py` starts near the end and
  runs alone after every other file is done: 169 s on Windows, 74 s on
  ubuntu. Started longest first, that time is gone.
- **A session has start-up of its own.** On Windows the workers of pass A
  spent 622 s outside any test, and the processes of pass B 466 s. On
  ubuntu it is 302 s against 256 s, and on macOS 182 s against 212 s. So
  on two of the three a process per file adds no process time. It loses
  where one long file has nothing to run beside.
- **macOS is not resolved.** The second push's run gave +3.4% for B. In
  the run of the table the summed time of the tests alone rose from 876 s
  in pass A to 1,164 s in the last pass, so the runner slowed by 33% while
  the job ran, and every later pass reads worse than it is. The Mac desk
  of A2d measured 2%.
- **The hook works on Windows.** It loaded in 332 of 332 pytest
  processes. Of 502 Python children, 439 kept it. The 63 that lost it are
  in 12 test files that build the child's environment themselves, the
  same 12 on every platform. Windows recorded the same 667 executed
  modules as ubuntu and macOS.
- **Paths needed work on Windows.** The hook compares through
  `os.path.normcase`, drops a `\\?\` prefix, turns backslashes into
  `/`, takes a junction for a link, and counts `os.spawn` and a direct
  `_winapi.CreateProcess` as child starts. After that, of 2,169 recorded
  paths none is absolute, none has another case than the disk, and none
  resolves outside the root.
- **359 recorded paths do not exist**, the same 359 on the three
  platforms. The hook records an open before it fails. That is what a
  row needs: a file created at one of them can change the verdict.

What this run does not cover:

- **Coverage was off.** It adds a start and a data file to every process,
  so the cost of a process here is a lower bound.
- **One runner ran every file.** A real macOS or Windows leg is one shard
  of 3 or 4.
- **No `fm` task started the processes**, so the task's environment was
  absent apart from `GIT_EDITOR=true`, set by hand.
- **Runner to runner the spread is large.** Pass A on ubuntu took 185.2 s
  in one run and 250.1 s in the next. Only ratios inside one job are
  compared, and the second A was 7% faster than the first on Windows and
  ubuntu.

Three anomalies the run met:

- `packages/workshop/tests/test_workshop_currency.py` waits on an editor
  on Windows when `GIT_EDITOR` is not set, and fails on ubuntu and macOS.
  Every `fm` task sets it, so the gate does not see it. In the second
  push's run pass A on Windows ended at the 60 minute limit for this
  reason.
- Two tests of `packages/workshop/tests/test_workshop_issue.py` fail on
  Windows outside a task with "no bash shell available on this machine".
- `tests/test_editor_extensions.py` collects no test at any point but the
  nightly, so its own pytest process exits 5. A session hides that.

**What this changes.** A process per item is chosen, and the observing
plugin is not built. The single plan,
`notes/20261122-proved-changes.md`, starts the longest file first, keeps
an absent path in the row, gives a row no entry for an item that collects
nothing, and has an open item for the environment an item's process gets.
The cost on macOS and with coverage on is that plan's open item 4.

The scripts, the hook as ported and each run's output are kept outside the
repository, on Willem's computer. The plan's phase C2 builds the hook in
the repository.

**A3. Sources 3 and 5 select more than today's package grain, against
contract 5.** No core package depends on an extension package. A change
under `packages/extensions/ruff/src` runs ruff's suite today and no
workshop suite (`W/_graph.py:285`). Source 3 makes every test file that
imports the mount code depend on every listed extension's sources. 52 of
110 workshop test files import it directly. Contract 5 says "a selected
test file is always one today's package selection runs", and phase 5 pins
it. The pin fails. It also names the dependents closure where a test's
closure reaches its package's dependencies. The finer graph has found an
edge the package graph lacks, so today's package selection can let an
extension change through without the workshop tests that mount it. The
contract should say what happens then: the package edge is added.

**A4. The footprint cannot attribute reads for about half the test files.**

- About 150 of 333 test files spawn a process. A child starts its own
  meter and nothing passes it the test's context. The plan counts "context
  passed into every child" as a cost of the per-test grain only. The file
  grain needs it, and phase 3's deliverables do not list it, while its
  first acceptance line needs it.
- 47 test files share a seed that is built once per worker session
  (`packages/workshop/tests/workshop_seeds.py:50`). The run uses
  `--dist=worksteal`, so the test that pays for the build changes between
  runs. Every other file's footprint lacks the build's reads. The in-process
  memos behave the same (`PY/_parse.py:43`).
- So the closure check can miss an undeclared read. It cannot raise a
  false one. "This finds the missing edge on the first nightly" does not
  hold for these, and the footprint of one tree differs between two runs.
- For code steps: they run as threads of one process
  (`W/_phases.py:746`), an audit hook is process-wide, and three layering
  steps share one parse, so only the first step opens a file. Phase 3 asks
  for a row for "a code step that reads a rendered file" and for "the hook
  records nothing outside a metered run (a test asserting the gate's own
  driver writes no row)". The code steps run in the gate's own driver.

The plan set aside one process per unit as too expensive for every run. On
the proof's run alone it is 333 pytest starts a night, and it gives exact
attribution. That option should be measured before phase 3 is designed.

**A5. The outcome auditor.**

- Each bisect step needs a tree's own environment: a sync, perhaps another
  lock. The stated cost, "the test's time times the logarithm of the
  window's merges", leaves that out, and it is the larger part.
- A night on which hundreds of tests fail for one cause starts hundreds of
  bisects. It needs a cap, and one bisect for the set.
- A test that fails in the suite and passes alone is recorded as flaky. It
  can be order-dependent, which is another finding with another fix.
- A failing test that the recomputed selection did select has no row kind.
- An escape leaves the missing edge in place. Until a person declares it,
  further escapes through it are possible while the selector stays on.
  Contract 7 makes switching on a person's act. Switching the affected
  package back to full runs could be automatic.

**A6. A removed file.** "The changed paths since the measured tree against
each test file's closure" names one closure. A deleted `conftest.py` or
data file is in the base tree's closure and not in the new one. Selection
needs the union of both.

**A7. Edges the six sources miss.**

- A distribution's own `pyproject.toml`: a changed entry point or
  dependency changes what an importing test sees.
- 99 `module:function` strings in tracked TOML files. Source 5 reads
  Python literals.
- A compiled extension module. A Python test that imports one has no
  source file to resolve to, and nanobind is in the workspace's extensions.
- The lock's transitive entries. A bump of a dependency's dependency has
  to select too.
- The coverage record's `ROOT_PINS` put `uv.lock` and the root
  `pyproject.toml` into every closure identity (`C/_record.py:107`), so
  any lock change re-measures every unit, whatever source 1's
  per-distribution edge says.

**A8. Coverage by test file is larger than stated.** Besides X4: 333 test
files against 28 suites is 11.9 times the rows per leg and base. The
empty-context arcs go today to every suite whose closure holds the file
(`PT/_coverage.py:88`), and contract 9 does not say which test file owns
them. A suite's closure identity is one git call over a few directories
(`C/_record.py:476`). A test file's is a hash over every blob in its
closure, 333 times. Today's record is 10.6 to 19.0 MB raw per leg. No
budget is stated.

**A9. Sharding is not mentioned.** The plan predates it. The packing is
over every test file of the workspace (`PT/_slice.py:95`) and the counts
are fixed in the contract. A selection of five files can fall into one
shard, and the other six shards each start a runner, pay 60 to 80 s of
setup and run nothing. The sharded and the unsharded paths also read "no
file for this package" differently (`PT/_steps.py:128` against
`PY/_backend.py:344`). The shard vocabulary has a word for the unit:
"item".

**A10. One spelling (open 1).** Under option (c) a category name and a
root-relative pattern share one key and nothing separates them by syntax:
`notes` is a category and `notes/` a directory, and `LICENSE` and
`CODEOWNERS` are patterns in use with no slash, star or dot. None of the
three options says where a claim's `suffixes` go, and 14 claims carry
them. A claim may name only the built-in categories today
(`W/_steps.py:350`).

**A11. The cost of the graph itself.** The parse memo is in the process
only, keyed by absolute path, size and mtime (`PY/_parse.py:43`). Every
`fm` process parses again, and a new worktree shares nothing. Parsing this
repository's 712 Python files takes 1.8 s in one process, with the file
cache warm. Contract 21's budgets cover sync. The selection has none. At
monorepo scale the graph needs a store keyed by blob id, which is the
design the 2026-10-04 ruling on composition at scale postponed.

**A12. The gain has a ceiling here.** On the slowest job of a full pull
request run, the test step is 496 s of 643 s (76%) and setup 58 s. A run
that executes no test still takes about 6 minutes, because the docs job and
the gate job set the floor. Selection can bring a pull request run from
11.3 minutes toward 6, and no further until the docs job narrows too. The
plan's own open 8 records that a day's merges selected 96.3% of the suite.

### Other languages

The plan's contract 11 is the right place for another language to arrive:
"An extension in a package's set supplies its file graph, as an answer to a
query." Much of what is needed is neutral already: the `references` and
`module-roots` queries (the cpp extension answers `module-roots`), the
`reaches` query, `[tests] reads`, the coverage line format with llvm-cov's
ranges, and the shard's item.

What the plan binds to Python, and what to change so the first
implementation is Python only and the second language is an extension:

1. **Phase 5 puts `livery.workshop._module_graph` in the core, reading
   python's `parsed_modules`.** The gate refuses that today:
   `layering.base-imports-no-extension` prints "the base reaches an
   extension through a registry it fills at mount, never by import". The
   core should own the unit, the closure arithmetic, the edge reasons and
   the pin. The python extension answers which files each source reaches,
   and the pytest extension adds the conftest chain and the plugin modules.
   Sources 4 and 6 are the core's.
2. **The unit is named "test file" everywhere**: `Scope.tests`,
   `fm graph.closure <test-file>`, `--tests`. A ctest test and a cargo test
   target are not files. The shard's word "item", owned by the extension
   that runs the tests, carries over.
3. **Contract 2, "computed from the tree alone", fits a language that can
   be parsed without building.** An include graph needs include paths,
   generated headers and resolved dependencies, so it needs at least a
   configure. Python's own third-party mapping already needs the
   environment. The contract can say what it protects instead: the closure
   is computed from the tree and the synced environment, never from a
   recorded run. The `compile-commands` query exists for the C++ case.
4. **The proof is Python's.** The audit hook and the coverage contexts
   observe a Python process. A compiler's dependency files prove a C++
   item's closure at every build, which is earlier and cheaper than a
   nightly. The neutral rule: an extension that supplies a file graph
   supplies the observer that proves it, and an item with no observer stays
   at package grain. Contract 13 implies this and no contract says it.
5. **`[tests] selection` is one switch for a workspace.** Contract 7 wants
   a new selector to run in shadow first. A second language added to a
   workspace that is already `on` then needs its own shadow period. The
   switch should be per package, or per answering extension, from the
   start.
6. **The invalidate-all entries "the Python version, the pytest plugins,
   the coverage configuration" are an extension's facts.** The core list
   holds the root contract, the lock and the tool pins, and an extension
   contributes its own.
7. **A package of two languages.** The Rust note already says a change to a
   `.rs` file of a binding's closure affects the Python package's tests,
   declared through categories. At item grain a Python test that imports a
   compiled module takes the package-grain answer, said in the same line,
   until the build's inputs are a unit the closure can point at. The plan's
   phase 2 makes the build a consumer of the engine, so that unit will
   exist.
8. **Coverage by item suits the other languages better than Python.**
   llvm-cov writes one profile per test binary. The record's schema should
   say "item" and hold no context or node id.

None of these makes the Python work larger. Items 1, 2, 5 and 6 are
decisions about names and homes that cost the same now and much more after
a second language.

### The affected plan against the ethos

- **A claim is proved by a machine.** The plan's intent states this well:
  "A selector that is a belief is not acceptable." A1 and A4 show the proof
  does not yet hold that line.
- **Measure, then decide.** The 61.2% and 26.5% figures were measured with
  a selector that contract 2 rules out, from a kind of run the nightly does
  not make. A2b is the measurement the plan lacked. Phase 5 is cheap and
  pure. It should come first among the selection phases, with a replay over
  history as its acceptance, and a stop for Willem on its numbers. The
  merge queue plan's phase 1 has that shape.
- **Small phases.** Phase 2 holds the engine's extraction, sync as a
  consumer, the development build as a consumer, one walk per sync and the
  budgets. It is several phases. Phase 10 is the heaviest and has the
  thinnest design.
- **The core knows no language.** See "Other languages", item 1.
- **Notes say the current state.** The status line says "written
  2026-10-12" and gives no state. The timing paragraph needs two other
  notes and three numbering schemes to read. The destination note's and the
  extensions plan's status lines are stale too. The 2026-10-11 ruling, that
  selection waits until the workshop stops changing wholesale, is still the
  recorded timing. Today's prompt changes it and belongs in the decision
  record.
- **One mechanism.** The extensions plan's open item 3 (a check's input
  digests, owner phase 15) is not carried into this plan, and it is the
  same idea as the first possibility below.

### Style of the affected plan

- The title is one sentence of 23 words.
- "The measures" says "Four rows" and lists three items. The fourth is the
  split of undeclared reads into steps and test files.
- "Pants' cost" names a tool without saying what it is. "The second net"
  is a metaphor.
- "Since 12e" and "phase 16" need another note to read.
- Phase 5's pin says "dependents closure" where the dependencies closure
  is meant.

## What the two plans make possible

Willem asked whether building these allows features not yet considered.
These are mine, not rulings. The first two bear on design choices to make
now.

1. **Proofs keyed by a unit's closure, on any tree.** The affected plan
   gives every item a closure identity. A verdict stored under the item and
   that identity holds on every tree where the closure is unchanged. A
   revert then runs nothing, since its closures were proved before. A
   candidate rebuilt after a red entry runs only what the dropped entry
   touched. A branch switch on a desk runs only what differs. The coverage
   record already keys its parts this way. This is "a proved tree runs
   nothing" stated for a unit, and X3 is where it is decided.
2. **Queue entries that do not chain.** Two queued changes whose influence
   sets do not meet cannot affect each other's tests. The queue then voids,
   behind a red entry, only the candidates that overlap it. With the first
   possibility this needs no new mechanism: the rebuilt candidates find
   their units proved. It is what keeps a queue usable at a depth of 20.
3. **Runners decided per change.** The 2026-10-18 ruling accepts a
   Linux-only pull request only when CI decides it from data. A footprint
   per item and per platform is that data: a change that reaches no item
   with platform-specific reads runs one platform. That is the cost
   reduction for a project on a free plan.
4. **Fewer shards for a small selection.** The matrix can follow the size
   of the selection instead of the contract's fixed count (A9).
5. **The affected tests after every edit.** `fm hooks.post-edit` exists. A
   closure index answers "which test files reach this file" at once. An
   agent, or a person's editor, runs those three files seconds after a
   save.
6. **A revert that proposes itself.** The auditor names the merge that
   brought an escape. The queue has an urgent lane. Together they can open
   and queue the revert of that merge.
7. **Build once.** A candidate's tree is main's tree after the landing. An
   artifact the candidate's run built (wheels, the site) can be keyed by
   tree and used by the release or the deploy without a second build.
   strongroom is the store for that.
8. **`fm bisect`.** With every commit on main proved, a bisect never meets
   a broken commit, and the auditor's bisect is the same code.
9. **Unattended changes at scale.** The tool refresh, the balancer and the
   update of every project built on the workshop can queue and land with
   nobody attending, in order, with no rebase loop. For many agent
   worktrees submitting at once, the queue removes the integrate step.
10. **Reports.** The reverse closure lists the source files no test
    reaches. The closure's size is a change's reach, which a review can
    show. The footprint is a record of what each verdict read, which the
    data lineage note asks for.
11. **A hold on the queue.** A label that stops landings while a release
    or an incident is handled. The plan's model takes it without a new
    mechanism.

## The order I recommend

Mine, for Willem to rule. Each step is small enough to land alone.

1. **Rule X1 and commit the two unmerged notes.** Everything below reads
   differently under each answer.
2. **Groundwork both plans need** (X3, X4, X5): the proof rule of CI's
   record, the metrics row's missing fields and a row for a red run, and a
   real full scheduled run on every platform. Each is useful with neither
   plan built.
3. **The merge queue's phase 1, widened.** It is cheap and it decides
   whether the model holds. Add to it: the event and token facts (X2), the
   label-time read, the merge with an expected head, the same-name status
   case, `merge=union`, and a tree model on the fake.
4. **The affected plan's phase 1**, once open 1 is ruled and the key has a
   free name. Then phase 2, split.
5. **The item graph** (the affected plan's phase 5, neutral as under
   "Other languages"), and the replay over history. Stop for Willem with
   the measured selectivity.
6. **The merge queue's phase 2 in parts**: membership and `fm queue`, then
   candidates in shadow, then landing and red, then urgent entries and the
   recovery merge. Then the loop scenario. Run it on this repository before
   the native queue.
7. **The affected plan's proof and selection phases**, redesigned for A1,
   A4 and A8.
8. **GitHub's native queue**, last.

What this does not cover: the forge identities plan's own order, which
step 3 depends on for the queue's identity.

## Answers so far (Willem)

- 2026-11-22, on items 1 and 2 below: "I'm perfectly happy for the entire
  CI model to change to accomodate this as smoothly and itegrated and
  consistent as possible. I want one way to do things, and by default they
  should be fast, efficient, and as optimal as possible for the common
  case." and "If the identities plan needs integrated as part of this that
  is acceptable". So neither plan is bound to today's points and jobs, and
  the queue's identity may come from the identities plan built as part of
  this work. Which CI model, and which identity, are still to be designed
  and ruled.
- 2026-11-22, a question, not a ruling: "Is the whole proven tree still
  needed if we only run affected tests? or is that part of calculating what
  is affected?" Answered in chat: a proved state is the base of every
  "affected" answer, and it does not have to be a whole tree. Item 6 below
  is that choice.

- 2026-11-22, a lean, not a ruling: "core vocabulary was in response to
  feel core was getting too big, and I still feel that. And it would still
  be very nice to have this as a separate extension somehow, although I
  firmly believe it should be a default one". Read as: the merge queue,
  and perhaps the workflow it belongs to, lives outside the core and is
  listed by default. Which of the two is asked in chat. It reopens in part
  the 2026-10-17 ruling that CI's weight in the core is the design.
- 2026-11-22: the release line `Mined-At` is renamed `Changelog-At`. Not
  built, and not scheduled.
- 2026-11-22: "we're making a single plan that closes a few open
  directions I think". One plan replaces the merge queue plan, the affected
  plan, the core vocabulary record and the identities plan. Its outline was
  given in chat. It is not written.
- 2026-11-22, a requirement for that plan: "I am making this for people as
  well as AI's, so you also have to look at it that you as orchestrator of
  multiple sessions have access to coordinate many different streams of
  development quickly and efficiently, without having to have a large part
  of your context wasted on in-depth build knowledge." and "it should be
  something you would enjoy using, as far as that is possible". Neither
  plan reviewed here was judged on this. What it asks of the design: one
  verb hands a change over, stacks included; one read shows every stream
  with its next verb; a verb waits until a stream needs someone, in place
  of polling; a dropped entry carries its reason in one line; the machine
  holds who works on what and how many heavy gates run at once; the
  affected engine says before work starts whether two streams overlap; and
  proofs are shared between worktrees. A measure: 56 of the 153 entries in
  the agent's memory of this project are operating traps (my count, by
  title). Each one the plan retires is context an agent gets back.

- 2026-11-22, marked by Willem as very important: "humain-scanner was the
  first project I worked on with another person that was also into ai
  assisted coding. We need to formalise communication between sessions and
  humans on different accounts and on different computers. If we were to
  follow my one way to do things paradigm strictly, it would mean we would
  not use inter agent communication on the same machine unless we formalise
  that too". Read as a contract for the single plan: every stream, queue
  entry, request and ruling is carried by the forge and the repository,
  through `fm`, so a session on another account and computer sees what a
  session beside it sees. What already fits: the queue keeps no local
  state; an issue's body is the work order; forge accounts say who wrote
  what. What does not: rulings held only in one agent's memory, a started
  stream that is invisible until its first push, `fm status` reading one
  branch, and the humain-scanner train's scripts living on one computer.

- 2026-11-22, how it worked on humain-scanner: "we informally had a setup
  where we both had sessions called orchestator, where mine was in charge
  project wise, and their on their machine. As we had shared resources to
  contend with as well, there would be requests and grants for that, and a
  ledger kept by the main orchestrator session". So the kinds of message
  that were needed in practice are a request for a shared resource, its
  grant, and a ledger of both, with one side holding the project's
  authority. The ledger lived in one session's context. Read against the
  ethos: the ledger belongs in a record every contender can read, a grant
  that follows a rule is given by the tool and needs no session to be
  alive, and "in charge" is the contract's `[owners]`. The merge queue is
  already such a ledger for one resource, the right to land on main. Which
  resources were contended, and how a request travelled between the two
  computers, is asked in chat.
- 2026-11-22, the two answers asked for above. The contended resource:
  "scanner hardware resource mostly." How a request travelled: "via issues
  and verbally relayed. we had to manually check what the other was up to
  or was requesting anything". So the resource was a physical device, not
  the main branch or a runner, and the forge already carried the requests.
  What was missing is the read and the notice: nothing showed either side
  what the other held or asked for, and nothing told them when a request
  arrived. A ledger for a device needs what a queue for main does not: an
  expiry for a holder whose session ended, and a way for a test or a task
  to say it needs the device.
- 2026-11-22, on whether the whole workflow leaves the core in the single
  plan: "if we can design it like that, yes. First I'd need us to look into
  what would be left of the core and if we have all the extension hooks we
  need to shift such a large part of functionality out." An inventory of
  the core and of the extension hooks was started the same day.
- 2026-11-22, on the record by unit: "in theory yes, but will need
  pros/cons. I also am still unsure about how you measured the numbers
  24/74, whether we can improve that and whether they are relative to per
  package narrowing already". The figures are 24.2% and 71.6%. Both are
  shares of the whole suite, as is the package grain's 44.0%. Two further
  measurements were started the same day: what causes the width of the
  static closure, and what a recorded footprint selects with one process
  per test file.
- 2026-11-22: the `Changelog-At` rename is "a very small element of the
  plan".
- 2026-11-22, on the record by unit replacing the gate record, the
  verified record and the stamp at once, which I had listed as a point
  against: "that could be considered a good thing on the one way to things,
  and the fewest number of concepts axis?" Yes. Three records and their
  rules becoming one is a reason for the change. The size of the swap is
  its cost, paid by pinning each property before the code is replaced.

- 2026-11-22, a lean on item 6: "The record by unit seems more livery
  like. How does the choice affect storage size and performance at large
  monorepo scale though?" Answered in chat with estimates, not
  measurements: the by-tree record costs one row per run, the by-unit
  record one row per unit a run executes. Kept as every fingerprint ever
  seen it grows with history and needs a store with eviction. Kept as the
  newest few fingerprints per unit plus each open branch's own rows it is
  bounded by the number of units. To be measured on the scale fixture.
- 2026-11-22, on the selection figures: "I don't like those numbers, it
  sounds like it'll be about even on average vs package grain? That doesn't
  seem right. How would the numbers look for something like pants2?" Pants
  infers a file's dependencies from its import statements, so it computes
  the closure this review measured. The figures are a property of this
  repository's import graph and of plugins loaded into every test session,
  not of the selector. Which code changes move them is being measured.

- 2026-11-22, after the core inventory: "18% is not a bad number. I did
  not hope for a large reduction, i actually quite like this and it's a
  signal that it is the right thing to do." The workflow leaving the core
  is wanted. On the loop: "I'm pretty sure moving devenv to a plugin, and
  building e2e test on top of it was part of a probably unfinished plan
  already?" It is: the destination note's phase 15, "lodges and e2e"
  (`notes/20261006-workshop-destination.md:1797`), with the distribution
  name `livery-lodge` claimed. Not built.
- 2026-11-22, on selection: "we would need to write our code in a way to
  be empathetic to per test file selection. Could there be a benefit that
  we require the public interface to pacakges be declared exactly?"
  Answered in chat: yes for static steps, which can depend on a
  dependency's interface in place of its source, and for tests written
  against a verified fake at the interface, as forge's are. Not for a test
  that executes the dependency's code.
- 2026-11-22, on flaky tests: "hopefully we can reduce flaky tests a lot
  with the deterministic clock and threading rules. Would there be value in
  making sure a test that has changed is at least run n number of times?
  could we even detect that?" Answered in chat: a changed or new test file
  is known from the diff and the record, running it several times before
  its first green row is cheap at one process per file, and two verdicts
  under one fingerprint show a flaky unit without any extra run.
- 2026-11-22, correcting my "expiry" for a device grant: "There was no
  expiry to grants. They were usually granted with a duration or a scope.
  There were electrical safety and performance implications as well, so a
  grant couldn't always be unilaterly revoked." So a ledger never takes a
  device back by itself. A grant past its duration is shown as overdue,
  and giving it back is the holder's act.
- 2026-11-22, on what "formalise" means: "yes, that is what I meant with
  formalise. There shouldn't be any need to arbitrate branches and runners
  when using livery directly. that should all be dealt with the queue, or
  even CI as it is now to be honest." So the single plan builds the read
  and the notice. It builds no ledger for branches or runners.
- 2026-11-22, the resources on humain-scanner: "There were 2 scanners
  accessible over the network, with sub resources such as workstations that
  can't capture and process at the same time, so a grant for one excludes
  the others. then each scanner would have up to 4 different electrical
  grants as well, one for each type of camera rail and one for the
  lighting. Not to mention the controller boards and camera raspberry pis
  per rig". A tree of resources with exclusions between them and grants
  that carry safety. My reading: that is a project's own domain and belongs
  in an extension of that project, on top of a request and answer the
  workshop carries.

- 2026-11-22, the order after the single plan: "lodges and e2e -> this
  should be the first plan we implement on top of the queues and granular
  testing implementation".
- 2026-11-22, on a resource ledger: "not specifically for the scanner no,
  but it should provide the infrastructure to build that plugin.
  humain-scanner does not use livery yet btw, but that is a goal for sure.
  A lot of the complexity and logic of what grants belong to eachother were
  handled by the orchestrator. it would not be easy to design something
  that is flexible and easy enough to do it generically I think". So the
  single plan builds what such a plugin stands on, and none of its logic:
  a request and its answer through the forge, the read and the notice, who
  asked and who may answer, and a shared record a plugin can own. The one
  piece of that which does not exist: a series on origin that a person's
  session may write. Today a remote series is written by CI alone.
- 2026-11-22, facts asked for and read the same day. Of about 10,200 child
  starts in the test suite, 9,434 are git (95 test files) and 41 are `fm`
  (14 test files, 18 of them `fm --json --list`). `fm` already runs in
  process: `livery.footman.testing` has a `Runner`, and 44 test files use
  it or `capture()`. The rendered files are compared before a write and
  left alone when equal (`_fragment_engine.py:628`). The receipt files are
  not: `write_rendered` (`:615`) and `_write_local` (`:766`) write on every
  sync, which is the 28 `.workshop-rendered` files the measurement saw.
  Willem: "we shouldn't rewrite them if they haven't changed".

- 2026-11-22, three answers on recording a footprint. Restoring modules
  between test files in one process: "agreed, no". Measuring the cost of
  one process per file on a Windows runner before choosing between that
  and an observing plugin: "fair!" On the receipt files written at every
  sync: "that should be fixed then". The fix is on the branch
  `fix/receipt-files-written-only-when-changed`.

## For Willem to rule

1. X1: today's CI model, or the vocabulary record's first.
2. The queue's identity (X2, Q1): a token of the project's, an App, or
   wait for the identities plan.
3. Whether the queue is the only path or a setting, and what a project
   that upgrades the workshop gets.
4. The affected plan's open 1, with the `suffixes` and the name clash in
   view, and a name for the key.
5. Whether the proof run is a full run on every platform every night
   (A1), and whether it may use one process per test file (A4).
6. Whether the first possibility, proofs keyed by a unit's closure, is in
   the groundwork's design or left out.
7. Whether `liveryhq/livery` runs the workshop's queue before the native
   one.
8. A2b: the static selector with narrower code, selection from the
   recorded footprint in CI, or the item graph without finer selection.
   Ruled after the item graph's replay.
