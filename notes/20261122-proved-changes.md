# Proved changes: one record of what is proved, one CI model, one queue, and the workflow as an extension

Status: proposed 2026-11-22, written overnight on Willem's word to start
it. Nothing is built. No part starts before Willem rules the open items
that part names. Once he rules, this note replaces the notes listed under
"What this plan takes over".

## The prompt (Willem)

> I don't want to progress development of livery until the merge queue plan
> and the affected plan are implemented. Willem, 2026-11-22

> I'm perfectly happy for the entire CI model to change to accomodate this
> as smoothly and itegrated and consistent as possible. I want one way to
> do things, and by default they should be fast, efficient, and as optimal
> as possible for the common case. Willem, 2026-11-22

> we're making a single plan that closes a few open directions I think.
> Willem, 2026-11-22

> I am making this for people as well as AI's, so you also have to look at
> it that you as orchestrator of multiple sessions have access to
> coordinate many different streams of development quickly and
> efficiently, without having to have a large part of your context wasted
> on in-depth build knowledge. Willem, 2026-11-22

> do the meaurements, and let them influence the single plan that you
> should also start. Willem, 2026-11-22

The intent: a change is proved once, by what it reads, and lands through
one queue on every forge. A person and an agent hand a change over with
one verb and read one line per stream. The core gets smaller while this is
built, not larger.

## What this plan does, in eight lines

1. **One record.** A verdict per unit, kept with what the unit read. A
   unit runs again when a file its last green run read has changed.
2. **A test file is a unit.** Its reads are recorded while it runs, on
   every CI run. Nothing is declared by hand that a machine can observe.
3. **One CI model.** Triggers start runs, a task has an access, jobs are
   derived. Code from a proposed change never runs beside a credential.
4. **A bot identity.** It is what queues, labels, posts and merges, so its
   acts start runs.
5. **One queue on every forge.** `fm submit --armed` hands a change over.
   The queue proves it on top of what is ahead and lands it.
6. **The workflow is an extension**, listed by default. The queue is born
   in it, and start, submit and the release train move into it.
7. **The read and the notice.** One verb shows every stream of every
   account. One verb waits until something needs you.
8. **Small things on the same lines**: `Mined-At` becomes `Changelog-At`,
   the metrics row gains a verdict, a full run of every test is scheduled.

## What this plan takes over

| Note | Taken over | Left with that note |
| --- | --- | --- |
| `20261009-merge-queue-plan.md` | All of it | Batching, which this plan drops |
| `20261012-what-a-change-affects.md` | All of it. The static closure of a test file is replaced by the recorded footprint | Phase 2's sync and build as consumers of one engine (the destination plan's phase 16) |
| `20261023-core-vocabulary.md` | Its plans 1, 2 and 7: the rename, triggers and access with derived jobs, the workflow extension | The CI manifest with the narrow mount, the CI-host axis in `livery.forge`, destinations, composition, delivery |
| `20261021-forge-identities.md` | Roles on tasks, the bot by a stored token, proposals built then sent | GitLab's run role, receipt tags by identity, GitHub Apps, people's repositories, the organisation role |

The evidence is in three notes of 2026-11-22: the review
(`20261122-review-of-the-merge-queue-and-affected-plans.md`), the core
inventory (`20261122-core-inventory.md`) and the sessions record
(`20261122-sessions.md`).

After this plan, in the order Willem ruled on 2026-11-22: lodges and e2e
(the destination plan's phase 15), first. The sessions record's own plan
follows the queue.

## What exists

Read in the code and measured on 2026-11-22 at main `eea997acb`. The
review note has each line's file and line. `W` is
`packages/workshop/src/livery/workshop`.

- **Two records and no chain in CI.** The local gate record chains proved
  trees (`W/_gate_record.py:245`) and never leaves the machine. CI's
  verified record writes a row for a narrowed run only when its base
  already has a full row (`W/_verified.py:499`). Both are per package.
- **The coverage record is already by unit.** One row per suite, replaced
  in place, with a closure identity as its skip key
  (`packages/extensions/coverage/src/livery/extensions/coverage/_record.py:454`).
- **Two spellings for what a step reads**: `claims` and `inputs`. `reads`
  is a third key with another meaning, the context keys a step reads
  (`W/contract.toml:830`).
- **A test file is already the shard's item**, and `PhaseContext.tests`
  carries test files to pytest with the meaning "only this package's tests
  changed" (`W/_steps.py:719`).
- **The nightly runs marked tests only**, on ubuntu: 4 tests on
  2026-11-21 (`W/_pytest_points.py:28`).
- **Every push event is the merge point** (`W/_points.py:1891`). The four
  event names and the points are literals. An extension adds a job at an
  existing point and takes the run's own token alone.
- **The gate job uses the job's own token** (`.github/workflows/ci.yml:131`),
  and an event that token causes starts no run on any of the three forges.
- **`livery.forge.PullRequests`** has no `update_branch`, no
  `remove_labels`, no draft and no status write. `merge_now` takes no
  expected head. The fake has no commits or trees.
- **The workflow, the release train and the issue verbs** are 10,430 of
  the core's 58,408 lines. They read 59 names of the rest, 46 of them
  private, and the rest reads 34 names of theirs.
- **Footman takes `override` as a mount parameter only.** `RepoConfig` is
  built from literals (`W/_workflow_tasks.py:204`). The verdict's states
  are fixed (`W/_verdict.py`).

## Measured

All on 2026-11-22. The review note has the methods and their limits.

| What | Value |
| --- | --- |
| Suite selected per Python-only commit, package grain as today | 42.9% of test functions, 56.3% of CI seconds |
| The same, from a recorded footprint | 15.9% mean and 9.9% median; 24.6% of CI seconds |
| The same, a sound static closure after about 20 edits | 34% to 43% |
| One pytest process per test file against one session, on a Mac desk | 199.2 s against 195.3 s, plus 2.8% for the recording hook |
| The same on hosted runners, files started in list order | Windows +8.9%, ubuntu +20.7% and +24.9% |
| The same, the longest file started first | Windows +2.9%, ubuntu +3.8% |
| The same on a hosted macOS runner | Not resolved: +3.4% in one run, +13.4% in the next, in a job where the runner slowed by 33% |
| The recording hook on top of a process per file, hosted runners | Windows +3.6%, ubuntu +1.6% and +4.5%, macOS +12.5% and +13.9% |
| What one more pytest process costs | Median 1.06 s on Windows, 0.61 s on ubuntu, 0.52 s on macOS |
| The hook on Windows | Loaded in 332 of 332 pytest processes. 439 of 502 Python children kept it. The three platforms recorded the same 667 modules |
| Footprints that differ between two runs | 1 of 331, by one text read |
| A private repository on GitHub's Free plan | Takes a draft pull request. Refuses branch protection and rulesets with 403 |
| Entries ahead of a merge when it queues | 0 for 65% to 75%, mean 0.3 to 0.5, max 8 |
| A CI run that runs no leg | 5.9 min, about 20 runner-minutes |
| Pull requests pushed again after their first run | 27 of 117 |

## Ground-truth contracts (do not violate)

1. **`fm check` is the gate and its exit code is the verdict.** The same
   commit gets the same verdict on every machine.
2. **Only CI writes a proof others rely on.** A desk's rows stay on the
   desk. There is one kind of runner. A project that needs a platform
   proved is made buildable in CI.
3. **One record.** A verdict per unit, with what the unit read. "This tree
   is proved" is derived from it and stored nowhere.
4. **A unit runs again when a file its last green run read has changed,
   or when it has no green row.** A skipped test is not green. A run that
   did not finish is not red.
5. **Nothing is left out without a printed line.** Every unit that runs
   prints the file that made it run. Every unit that does not run can be
   asked why.
6. **What a machine can observe is not declared by hand.** A declaration
   is for what no observer sees.
7. **A property the old code enforces is pinned by a test before the code
   is replaced.** The gate record, the verified record, the stamp and the
   coverage union each have properties to name first.
8. **One queue on every forge.** What a person types, the shape of main
   and what is promised are the same on GitHub, Gitea and GitLab. A forge
   capability changes who does the work.
9. **Every unit of a tree is proved before the tree lands.** The recovery
   merge is the one exception, and it says so.
10. **Main is linear, one squash commit per pull request.** Its subject is
    the pull request's title and its body the pull request's body. The
    forge makes the commit. No credential the workshop holds pushes to
    protected main.
11. **Code from a proposed change runs with minimal access.** It never
    runs in a job that can post a status, merge, label or publish.
12. **Nothing on the merge path waits on a person or on a session.** A
    step that cannot act prints why and exits.
13. **Every verb is safe to run twice, and every write is conditional**:
    a merge names the head it expects, a push carries a lease, a record
    moves by compare-and-swap.
14. **Polling is the baseline.** A scheduled run brings the queue in
    line. An event only makes it sooner.
15. **No logic in YAML.** The forge is spoken to through `livery.forge`
    alone. A difference between forges is a probed capability, and a
    missing one refuses by name.
16. **The core names no workflow.** It imports no extension. The queue is
    born in the workflow extension and never lives in the core.
17. **One channel.** People and sessions coordinate through the forge and
    the repository, with `fm`. This plan builds no way for two sessions to
    talk that a second account on a second computer cannot use.
18. **One line per stream, with the next action in it.** A session that
    coordinates many streams needs no build knowledge to read it.
19. **CI decides what runs.** No narrowing depends on the author knowing
    what a change touches.
20. **What a project inherits by default costs little.** One candidate
    runs at a time unless a workspace says otherwise.
21. **A language arrives as an extension.** The record, the unit and the
    item name no language. An item whose extension has no observer is
    proved at package grain, as today.
22. **Fallbacks before happy paths.** Kebab-case keys. No migration code,
    and no code for a spelling this repository alone wrote. A note changes
    with the code it describes.

## The design

### The record

A **unit** is one thing that gives a verdict: a step over a package, or
one item of a test step. An **item** is what a test runner runs as one.
For pytest it is a test file, as in the shards.

A row holds the unit, the platform, the verdict, how many times each
verdict was seen, the run that wrote it, and what the unit read:

- **For a step**, what it read is what its declaration names: the files
  its `inputs` match, the tools it uses with their pinned versions, and
  its options. The row keeps a fingerprint of those. Nothing is observed.
- **For an item**, what it read is what was recorded while it ran: each
  file with its content id, the directories it listed, and the
  third-party distributions it imported. The row keeps the list, since
  the list is what a later tree is compared with.

A unit is proved on a tree when it has a green row whose reads are the
same on that tree. That needs no base tree, no chain and no search. A
pull request, a queue candidate, a push to main, a revert and a desk all
ask the same question.

What goes into every row's key besides files: the platform as the CI host
reports it, image version included, and the Python version.

**Verdicts.** `green`, `red`, `skipped` and `unfinished`. Two verdicts
under one set of reads mark the unit as flaky: the row counts both, and a
flaky unit is not proved by one green.

**Who writes.** CI. A run started by a pull request or a candidate writes
rows for its own branch. The trusted run on main copies the rows that hold
on main's tree into main's record, and runs nothing for them. A desk
writes rows that only the desk reads, and reads CI's.

**What is kept.** The newest few sets of reads per unit and platform, and
each open branch's own rows until a day after the branch is gone. That
bounds the record by the number of units, and a revert within those few
finds its rows. The count is open item 2.

**Where.** The state store, as a keyed family split over several refs, so
that legs writing at once do not contend on one. A list of reads is
stored once and shared by the rows that have the same one.

**Coverage.** A unit's coverage part is kept with its row and keyed the
same way. A process per item gives each item its own coverage data,
branches included, so the union needs no rule for lines that belong to no
test.

What the record does not cover:

- **Trust in a pull request's own run.** A run with the branch's code
  writes the branch's rows, as it stamps the tree today. A fork's token
  cannot write, so a fork's pull request proves nothing by itself.
- **A unit that fails only after another unit ran.** One process per item
  removes that for tests. For steps it is as today.

### What a step reads

One key, `inputs`, with two kinds of entry, each under its own name so
neither can be taken for the other:

```toml
[phases.lint.ruff]
inputs.categories = [
  { category = "source", suffixes = [".py", ".pyi"] },
  { category = "test", suffixes = [".py", ".pyi"], ignore = ["D1"] },
]

[phases.layering.declared-imports]
inputs.reads = ["tasks.py", "packages/**/*.py"]
inputs.ignores = ["**/*.md"]
```

`claims` goes. Its entries become `inputs.categories` unchanged, so the
suffixes and the rule codes a tool skips keep their place, and `ruff.toml`
is still rendered from them. `reads`, the context keys a step reads, keeps
its name and its meaning. This is open item 3.

### Recording what an item reads

The pytest extension records, for each test file, the first-party modules
its processes imported, the repository files they opened, the directories
they listed, and the third-party distributions they imported. A hook
loaded into every Python process does the recording, so a child that
inherits the environment is recorded too.

Each test file runs in a pytest process of its own, as many at once as a
session has workers. The extension starts the longest file first, by the
durations the shards are balanced by, and a file with no duration yet
before those. In list order the longest file starts near the end and runs
alone after every other file is done: 169 s on the Windows runner. Started
longest first, a process per file costs 2.9% on Windows and 3.8% on ubuntu
against one session. One session with a plugin that observes imports and
fixtures is not built: it keeps a value read once and held in memory out
of the row. Restoring modules between files in one process is ruled out.

The hook records an open that fails, too. 359 of the 2,169 paths recorded
are of files that do not exist, the same 359 on the three platforms. The
row keeps them as absent, so a file created at one makes the item run.

A recorded path has one spelling on every platform: relative to the
root, with forward slashes, in the case the disk has. On Windows the hook
drops a `\\?\` prefix, compares without case, and takes a junction for
a link.

An item that collects no test at the run's trigger writes no row.
`tests/test_editor_extensions.py` is one at every trigger but the
scheduled run.

What a recording must not count: the test runner's own search for its
configuration, which opens every `pyproject.toml` above a test file. The
runner's configuration files are named by the extension instead.

What a hook cannot see, and what covers each:

| Not seen | Covered by |
| --- | --- |
| A Python child whose environment lost the hook (12 test files) | The hook's own row says a child was started without it. The item is then proved at package grain, until the child keeps the environment |
| What a native program reads inside the repository (39 test files) | `[tests] reads` in the package's contract, the one hand declaration. The hook knows which items start which program where, and the row names an item that needs one |
| A check that a file exists; a file added to a listed directory | Listed directories are in the row. A path added or removed under one makes the item run |
| The installed environment between the lock and an import | The hook records each imported distribution. A lock change that moves it, or anything it depends on, makes the item run |
| A skipped test, another platform | A skip is its own verdict. Rows are per platform |
| An environment variable the item reads. `test_workshop_currency.py` waits on an editor on Windows without `GIT_EDITOR`, which every task sets | Not in the row. The extension gives each item's process one named environment: open item 14 |

### Selection, and how it is kept honest

A run asks, for every unit of its tree, whether the unit is proved. It
runs the ones that are not, and prints for each the file that changed.
`fm check` does the same on a desk.

- **A changed or new test file runs several times** before its first
  green row, at once and with different hash seeds, so a flaky test is
  found on the day it is written. The count is open item 5.
- **A scheduled run executes every unit on every platform**, proved or
  not. A unit that is green by its row and red in that run is an escape:
  the run is red, the row names the unit and what changed since its row,
  and the package it belongs to runs whole until the cause is declared or
  fixed. This run replaces the affected plan's nightly proof, its closure
  check and its bisect.
- **Selection is switched on per package**, by a person, after it ran in
  shadow: everything runs, and the record says what would have been left
  out.
- **The static closure is not built.** The row says why a unit ran. A new
  item has no row and runs. Three imports that widen every footprint are
  removed (`_pytest_layout.py:34`, `_templates.py:538`,
  `_lifecycle.py:212`), and the share of the suite a change selects is a
  measure the store keeps.

### The CI model

From the core vocabulary record, with the trust rule made a contract.

- **A trigger** starts a run: `pull-request`, `candidate`, `merge`,
  `scheduled`, `release`. `nightly` is renamed `scheduled`, with a one-off
  migration of the series that carry the name, as ruled on 2026-10-21.
- **Trust** follows the code: a pull request's and a candidate's code are
  untrusted, and code on main is trusted.
- **A task** names its trigger and its access: `minimal` (the run's own
  token, which writes the run's own records), `write` (the bot) or
  `admin`. Untrusted code gets `minimal` and no more.
- **Jobs are derived** from the tasks of a trigger, grouped by access and
  matrix. The contract names no job.
- **An extension contributes a trigger**: its name, its trust, and the
  forge events that start it in neutral words. The workflow extension
  contributes `candidate`, and the trusted `queue` trigger that runs main's
  code when a label, a draft state, a queued head or a candidate's verdict
  changes, and on a schedule.
- **A task that proposes a change** builds it with minimal access and
  writes it as a file. A task with write access sends it. The balancer and
  the tool refresh use that, and their pull requests enter the queue like
  any other.

Not in this plan: the committed CI manifest with the narrow mount, and
the CI-host axis in `livery.forge`. Until they exist, a task with write
access loads the workspace, as the nightly, publish and govern jobs do
today.

### Identities

Three roles, as the identities plan names them: `run` is the job's own
token and is never stored, `bot` is `FORGE_TOKEN`, `admin` is
`FORGE_ADMIN_TOKEN`. In CI no role falls back to another, and a task that
declares a role it lacks refuses before its first entry, naming the
secret and the command that sets it up.

The bot is a stored token in this plan, on every forge. What it needs is
listed per forge by the conformance scenarios of part E. A GitHub App is
left with the identities plan.

### What the forge package gains

- Verbs: `update_branch`, `remove_labels`, a draft read and its two
  setters, a status write, the time a label was applied, and a merge that
  names the head it expects.
- One question, "is this pull request queued", answered from the label or
  from a native queue. Every reader that asks today whether a pull request
  is armed asks this.
- One promise, "the forge refuses a merge the queue did not prove", held
  by each backend's own means and probed. The queue's status has a name no
  job has.
- Capabilities for drafts, the status write and a native queue, each
  refusing by name where a forge or a plan lacks it.
- The fake gets commits and trees, so a squash's tree can be compared.

### The queue

The model Willem agreed on 2026-11-21 stands: an ordered list of pull
requests, a candidate for each made of main, the entries ahead and itself,
landing through the forge's own squash merge. What changes:

- **Membership is the label `queue:armed`**, urgent entries carry
  `queue:urgent`, and the order is the time the label was applied. On a
  native queue the forge's own list is the membership.
- **A candidate is a branch under `queue/`.** Each of its commits names
  its entry and the head it was built from, so the candidates are the
  queue's record and the verb keeps no state.
- **A candidate is built as the forge merges**: with no merge attributes.
  A conflict is then found before a run is spent, and the entry is dropped
  with the conflict named.
- **A candidate's run asks the record.** It runs the units its tree has
  not proved. There is no "measure from the candidate ahead".
- **One candidate runs at a time by default.** The next starts when the
  one ahead has its verdict, and then finds those rows. More at once is a
  setting, and two candidates that do not overlap do not repeat each
  other's work.
- **A red entry leaves the queue** with one line: the units that failed
  and the file that made each run. A run that did not finish is run again
  and drops nothing. A unit already marked flaky is run again once.
- **Behind a red entry, the candidates are rebuilt** and find the rows
  they already wrote for everything the dropped entry did not touch.
- **Landing**: the trusted `queue` task posts the queue's status on the
  entry's head and calls the forge's squash merge with that head, the
  title and the body. Main's own run then finds every unit proved. A
  squash whose tree differs from the candidate's is no longer a special
  case: the units decide.
- **A stack**: an armed child waits in the queue until its parent has
  landed. Nobody comes back to arm it.
- **Urgent**: `fm submit --urgent` and `fm submit.arm --urgent` put an
  entry ahead of everything that has not started landing.
- **A release** is an urgent entry. Its squash carries the body with
  `Changelog-At`, the new name of `Mined-At`.
- **The recovery merge**: `fm submit.merge --bypass-queue <number>
  --reason=<text>`, for when the queue cannot land its own fix.
- **A draft is never queued.** `fm submit` without `--armed` opens a
  draft where the forge has drafts.
- **Shadow first.** Candidates are built and run beside today's
  auto-merge, and land nothing, until Willem switches it.
- **GitHub's native queue** is a capability on top, in the last phase of
  part F. It needs the queue's status on a pull request's head before the
  entry, a governed default for the squash's title and body, and
  `ALLGREEN`.

Dropped: batching. With a record by unit, a candidate already runs only
what its entry changed.

### The workflow extension

`livery-extensions-workflow`, listed by `fm new.project`'s stock list. It
is born holding the queue alone, written against public names. Then start,
submit, integrate, abandon, status and the repository's rules move into
it. The release train moves into `livery-extensions-release`, and the
issue verbs into `livery-extensions-issues`, which a repository without
issues does not list. The names and the split are open item 6.

What the core gains first, since the queue needs each wherever it lives:

| Hook | For |
| --- | --- |
| An extension contributes a trigger and a task with an access | `candidate`, the `queue` task |
| An extension contributes repository settings | The two labels, the required status, the protection of `queue/` |
| An extension contributes a state and its line to `fm status` | "queued, 2 ahead" |
| One list of reserved branch prefixes | `queue/` beside `workflow/` |
| An extension replaces a verb | `submit` in the extension while the core still has one |
| A public way for a task to send a change | The balancer, the refresh, a new project's setup |
| The code that follows a run, apart from the code that classifies a pull request | The CI verbs and the loop keep the first |

The 46 private names the workflow reads become public in the phase that
moves their reader, two pull requests each, as the 19 extensions left.
The 34 names the core reads from the workflow are turned around: access
to the forge (`_forge_lane`, `_tokens`) stays in the core.

### The read and the notice

- **`fm status --all`**: every open stream of every account, one line
  each: the pull request, the branch, the account, the state, the next
  action. `fm queue` is the queue's part of it. Both have a JSON form.
- **`fm status --watch --all`** exits when a stream of yours needs
  someone, and prints that line. The name of the verb that lists what
  waits on a person is open item 7.

This plan stops there. A request and its answer between sessions, the
address of a session, a series a person's session may write, and a
project's own ledger are designed in a record of their own after part F.

### What it does not cover

- Batching, and a bisect of a red group.
- The CI manifest, the narrow mount and the CI-host axis.
- GitHub Apps, receipt tags by identity, people's repositories.
- Sync and the development build as consumers of one engine.
- An observer for a language other than Python. Contract 21 keeps the
  place for it.
- A static graph of a test file's imports, and a forecast of which
  streams overlap.
- Refusing an unproved merge on a repository with no branch protection,
  such as a private GitHub repository on the Free plan. The queue works
  there unchanged. `fm queue` and `fm submit` warn that a merge around it
  cannot be refused.

## Parts and phases

Each phase lands alone, gate-green, in a day or two, with its refusals
tested first and this note updated in the same change. A phase's
acceptance names the command. Where a command does not exist yet, the
phase that makes it is named.

The parts and the sessions that build them
(`notes/20261122-sessions.md`): A, B and C by `gate`; D by `ci` with
`forge`; E by `forge`; F by `workflow`, with `footman` for one hook; G by
`workflow`. At most four at once. E can start with A.

### Part A: measure and pin

Nothing is replaced in this part.

**A1. The metrics row says what happened.** A verdict, an attempt, the
trigger, the entry and the base in every row. A row for a red run. The
windows of `metrics` and `verified` sized for candidate runs.

- `test_a_red_run_writes_a_row_with_its_verdict`, before the happy path.
- `fm store.show metrics` after a red run on a branch shows its row.
- `fm check` green.

**A2. A scheduled run of every test, on every platform.** Today's marks
keep their meaning. A test with no mark runs at this run too.

- `fm ci.dispatch --point=nightly --ref=<branch>` runs the whole suite on
  three platforms, and `fm store.show metrics` shows its test count.
- Does not cover: what the run costs a project on a free plan. It is on
  for this repository and a setting for others (open item 8).

**A3. The pins.** Tests only: each property of the gate record, the
verified record, the stamp, `leg_plan` and the coverage union that the
record by unit must keep, named in this note and pinned.

- The list of properties is added to this note under this phase.
- `fm test.pytest packages/workshop` green before and after.

**A4. Three imports.** Remove the imports at `_pytest_layout.py:34`,
`_templates.py:538` and `_lifecycle.py:212` from the paths that load
them into every test.

- A test that a pytest session of another package loads no module of the
  workshop's 57-module cycle.
- `fm check` green.

### Part B: the record by unit

Needs open items 1, 2 and 3.

**B1. One spelling.** `inputs` with `categories` and `reads`; `claims`
gone. Every declaration of the workshop and its extensions in it.

- A pinning test written first: for a fixture of changed paths, every
  step's selection and every job's skip are what they are today.
- `ruff.toml` renders byte for byte as before (`fm drift.check`).
- `fm check` green; `fm ci.e2e --scenario=release` green.

**B2. The row, written beside today's records.** Step units at package
grain. Every CI leg writes rows. Nothing reads them yet.

- `test_two_writers_of_one_ref_both_land`, and
  `test_a_skipped_step_writes_skipped_not_green`, first.
- `fm store.show proved` (new) lists a run's rows.
- Budget: the size and the fetch of the record on the scale fixture
  (`fm ci.scale`), recorded in this note.

**B3. Readers in shadow.** `fm check` and the CI legs compute what the
record would skip and print where it differs from today's decision. Today
still decides.

- `fm store.show proved --differences` (new) is empty over a week of this
  repository's runs, or each difference is explained in this note.

**B4. The record decides.** `fm check`, the CI legs and main's run read
the record. Main's run copies a branch's rows.

- A3's pins pass.
- `fm ci.e2e --scenario=verified-skip` green, asserting rows.
- A revert of the last commit on a branch runs no unit
  (`fm check` prints `nothing to run`).

**B5. The old records go.** The gate record, the verified record, the
stamp, `leg_plan` and the merge-base fallback are deleted.

- `git grep -n "leg_plan\|ci.verified.stamp"` finds nothing under
  `packages/`.
- `fm check` green; the loop's `develop` set green.

**B6. Coverage with the row.** A unit's coverage part is kept with its
row. The union reads the tree's rows.

- The coverage pins of A3 pass.
- Every package's figure equals its figure before the phase on one full
  run.

### Part C: test items

Needs open items 5 and 14, and part B through B4.

**C1. The item.** The pytest extension names its items. The shard's
`files` become `items` (#1820).

**C2. The recording.** A process per item, the longest started first,
with the hook in every Python process. A row holds the reads.

- `test_a_child_that_lost_the_hook_marks_its_item_unrecorded`, first.
- `test_an_item_that_collects_no_test_writes_no_row`.
- `test_an_open_of_a_missing_path_is_recorded_as_absent`.
- `test_a_recorded_path_has_one_spelling_on_every_platform`, with a
  drive letter in another case, a `\\?\` prefix and a junction.
- `test_a_file_with_no_duration_starts_before_the_longest`.
- `test_a_listed_directory_gaining_a_file_makes_the_item_run`.
- `fm store.show proved --item packages/forge/tests/test_gitea_pulls.py`
  lists what it read.
- A process per item costs under 5% of a leg's wall against one session,
  and the hook under 5% more, on each platform and with coverage on. Read
  from `fm ci.timings` over 10 runs of each: one hosted macOS runner
  slowed by 33% inside one job, so one run settles nothing there.

**C3. Item rows in shadow.** Everything runs. The record says what would
not have.

- A measure `selected` in the store: the share of items a run would have
  run. `fm store.show measures` (new) shows it per run.
- Over 30 merges the mean is under 30% of test seconds, or this note says
  why.

**C4. What the hook cannot see.** `[tests] reads`, the directory rule,
the distribution rule, the runner's configuration files.

- A scratch workspace whose test reads a file through git: with the file
  in `[tests] reads` a change to it selects the item, and without it the
  scheduled run reports an escape (a loop scenario).

**C5. Repeats and flaky units.** A changed or new item runs several
times. A unit with two verdicts is flagged.

- `test_a_unit_with_both_verdicts_is_not_proved_by_one_green`, first.
- A test that fails one run in two is caught in the pull request that
  adds it (a loop scenario).

**C6. The switch, per package.** `[tests] selection = "on"` in a
package's contract. The scheduled run audits.

- An escape makes the scheduled run red, names the unit and the changed
  file, and puts the package back to whole runs (a loop scenario).
- A one-line change in `packages/forge/src/livery/forge/_types.py`:
  `fm check` prints the items it runs and the file that selected each.

**C7. Shards follow the selection.** A leg starts the shards its
selected items need.

- A run that selects five items starts one shard per platform, read from
  `fm ci.timings`.

### Part D: the CI model and the bot

Needs open items 10 and 11.

**D1. The rename.** `nightly` to `scheduled`, points to triggers, and a
hidden task that migrates the series once.

**D2. Access on tasks, jobs derived.** `[[ci.tasks]]`, the three access
levels, the render. Every job of the workshop and its extensions
re-spelled.

- A pinning test first: the rendered workflows of all three forges do the
  same work as before, by the goldens.
- `test_a_task_on_an_untrusted_trigger_asking_for_write_refuses`.

**D3. The bot.** `FORGE_TOKEN` as a declared credential, the presence
check before a task's first entry, `fm credential.setup bot` and
`fm credential.status`.

- `test_a_task_missing_its_role_refuses_before_its_first_entry_naming_the_secret`.
- `fm credential.status` on this repository names what supplies each role.

**D4. Proposals, built then sent.** The balancer and the refresh write a
proposal. A task with write access sends it.

- The next scheduled run opens the day's balance pull request, and its
  run starts.

**D5. Contributed triggers.** An extension declares a trigger, its trust
and its events. The render writes them for each forge.

- `test_a_trigger_naming_an_event_a_forge_lacks_refuses_naming_both`.

### Part E: the forge's facts, by conformance

Can start with part A. If `squash-tree` or `queue-status` fails on a
forge, the plan stops for Willem before part F.

**E1. The fake has commits and trees.**

**E2. The verbs**, on three backends and the fake: `update_branch`,
`remove_labels`, draft, the status write, label times, a merge with an
expected head, `queued`.

**E3. The scenarios**, recorded on all three forges:

- `squash-tree`: a head behind main, a rename, a file with a CRLF
  attribute, two note changes that a union merge would join, an LFS
  pointer. Equal trees, or a conflict the candidate builder names too.
- `queue-status`: a status with a name no job has satisfies the
  protection, and a merge without it is refused.
- `bot-events`: which events start a run when the bot causes them, and
  that none starts when the job's token does.
- `draft`, `update-branch`, `label-time`, `expected-head`.
- `packages/forge/docs/quirks.md` gains what each forge does differently.

Acceptance for each: `fm forge.fixtures.record --scenario=<name>` records
and `fm forge.conformance --scenario=<name>` replays green on the fake and
three backends.

### Part F: the workflow extension and the queue

Needs parts B, D and E, and open items 6, 9 and 12.

**F1. The hooks.** The seven rows of the table under "The workflow
extension", each its own change. The verb override is footman's.

**F2. The extension is born, with the read.** `queued`, the two labels,
`fm queue`, the queue's line in `fm status`, `fm status --all`.

- With one pull request labelled by hand, `fm queue` lists it with its
  account and `fm status --all` shows the same line.

**F3. Candidates in shadow.** The `candidate` and `queue` triggers, the
builder, the candidate's run. Auto-merge still lands.

- For 30 merges, each landed tree's units were proved by its candidate's
  run, read from `fm store.show proved`. A difference stops the part.

**F4. Landing and red.** The status, the protection, the merge with an
expected head, the drop. Auto-merge goes. Armed pull requests are moved
to the label by the phase's own task.

Tests, each before the green path: a repository with no protection,
which lands through the queue and warns, two updates at once, a head pushed
between the check and the merge, a landing the forge refuses, a run that
did not finish, a conflict at the build, an empty queue, a second update
that changes nothing.

- `fm ci.e2e --scenario=queue` (new): three entries, the third with a
  failing test. Two land in order and read merged, the third is dropped
  with its line, and main's runs execute no unit.

**F5. Stacks, urgent entries, the recovery merge.**

**F6. A release through the queue.** `Changelog-At`. The release's checks
read the entry from the candidate.

- `fm ci.e2e --scenario=release` green through the queue.
- Land this phase while no release wave is in flight: the newest release
  squash carries the old line.

**F7. The rest of the workflow moves.** In this order, since a requirer
leaves before what it requires: the release train, the issue verbs, then
start, submit, integrate, abandon, status and the repository's rules.
Each is two pull requests: the names made public, then the move.

- `git grep -n "livery.workshop._submit"` finds nothing under
  `packages/workshop/src`.
- The core is under 48,500 lines (`wc -l`), from 58,408.

**F8. GitHub's native queue.**

- On `liveryhq/livery` after `fm workflow.configure`: an armed submit
  enters GitHub's queue and merges by it, and main's run executes no unit.

### Part G: the notice

**G1. `fm status --watch --all`**, and the verb that lists what waits on
a person.

- With a dropped entry of this account, the verb exits and prints its
  line; with none it waits.

## Temporary, replaced by

| Temporary | Replaced by |
| --- | --- |
| Rows written beside the gate record and the verified record | B5 |
| Step units at package grain for a test step | C6, per package |
| Auto-merge beside candidates in shadow | F4 |
| `submit` and `start` in the core while the queue is in the extension | F7 |
| A task with write access loads the workspace | The CI manifest and the narrow mount, in the vocabulary record's plan |
| The bot as a stored token on GitHub | An App, in the identities plan |

## Decision record

- 2026-11-22, Willem: one plan for the merge queue, what a change
  affects, the CI model and the identities. The CI model may change
  entirely. The identities work may be part of it.
- 2026-11-22, Willem: the record by unit "seems more livery like". That
  it replaces the gate record, the verified record and the stamp at once
  is a reason for it, on "the fewest number of concepts axis". Not ruled
  as a whole: open item 1.
- 2026-11-22, Willem: the workflow leaves the core "if we can design it
  like that", and 18% out of the core "is a signal that it is the right
  thing to do".
- 2026-11-22, Willem: no second kind of runner. "If that's required the
  bullet should just be bitten and the project should be made buildable."
  Contract 2.
- 2026-11-22, Willem: restoring modules between test files in one
  process, no. Measure a process per file on a Windows runner before
  choosing.
- 2026-11-22, Willem: a skipped test recorded as green "probably
  shouldn't happen". Contract 4.
- 2026-11-22, Willem: a changed test run several times has value, and
  the deterministic clock and the threading rules should cut flaky tests
  at the source.
- 2026-11-22, Willem: `Mined-At` becomes `Changelog-At`, "a very small
  element of the plan". A label was set aside, since it carries no sha.
- 2026-11-22, Willem: "formalise" means the read and the notice. No
  arbitration of branches or runners: the queue and CI decide those. The
  plan provides what a resource plugin stands on, and none of its logic.
- 2026-11-22, Willem: lodges and e2e is the first plan after this one.
- 2026-11-22, mine, from the measurements: selection reads the recorded
  footprint, and the static closure is not built. The footprint selects
  15.9% where a sound static closure ends at 34% to 43%, and recording
  costs 2% on a Mac.
- 2026-11-22, Willem, on a repository with no branch protection: "I
  don't think we should limit anything if branch protection is not
  available. They'll just have to be warned." Closes open item 9. Probed
  the same day on a private repository on GitHub's Free plan: protection
  and rulesets answer 403, and a draft pull request is accepted.
- 2026-11-22, mine, from the hosted runners: each test file runs in a
  process of its own, the longest started first. That costs 2.9% on
  Windows and 3.8% on ubuntu against one session, the hook loads in every
  pytest process on Windows, and a Python child there inherits it. Does
  not cover: macOS, which the hosted runner did not resolve, and coverage,
  which was off. Both are C2's last acceptance line.
- 2026-11-22, mine: one candidate at a time by default. Depth is 0 for
  most merges, and three macOS shards per run meet the runners' limit
  first.
- 2026-11-22, mine: batching is dropped, and the tree comparison after a
  landing is no longer a proof.
- 2026-11-22, mine: `inputs` with `categories` and `reads`, since `reads`
  is taken and a claim's suffixes and rule codes need a home.
- 2026-11-22, mine: the queue is born in the extension, so the core does
  not grow while the plan is built.

## Open

1. **The record by unit**, as under "The record". Willem. Everything in
   parts B, C and F rests on it.
2. **How many sets of reads a unit keeps.** Proposal: 3. Mine, at B2,
   from the scale fixture.
3. **The one spelling**: `inputs.categories` and `inputs.reads`. Willem,
   before B1. Does not cover: one kind of entry. There are two, each
   under its own name.
4. **What a process per item costs on macOS, and with coverage on.**
   The choice itself is made: see the decision record. A Mac desk
   measured 2%. A hosted macOS runner gave 3.4% and 13.4%. Coverage adds
   a start and a data file to every process and was not measured. Mine,
   at C2, from `fm ci.timings`.
5. **How many times a changed test file runs.** Proposal: 5, at once.
   Mine, at C5.
6. **The extensions' names and split**: workflow, release, issues.
   Willem, before F1. Three names to claim on the index.
7. **The name of the verb that lists what waits on a person.** Willem,
   before G1.
8. **The scheduled full run for a project on a free plan**: on or off by
   default, and how often. Willem, at A2.
9. Ruled on 2026-11-22: a repository with no branch protection gets the
   whole queue and a warning. See the decision record.
10. **The trigger names**: `pull-request`, `candidate`, `merge`,
    `scheduled`, `release`, and the trusted `queue`. Willem, before D1.
11. **The bot on this repository**: whose token, with which rights, until
    an App exists. Willem, before D3.
12. **Whether the queue is the only way to land once the workflow
    extension is listed.** Proposal: yes. A workspace that wants another
    way lists another workflow. Willem, before F4.
13. **A request between sessions, a session's address, a series a
    person's session may write.** Their own design record, after part F.
    Willem.
14. **The environment an item's process gets.** A row does not hold the
    variables an item read. Proposal: the task's environment, with the
    forge's run variables removed, so a row made at one trigger holds at
    another. A test that needs a run sets the variables itself, as the
    suite's tests do today. Mine, at C2. Does not cover: a variable a
    person sets at a desk. A desk's rows are read by that desk alone.
