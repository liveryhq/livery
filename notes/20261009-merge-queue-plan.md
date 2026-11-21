# The merge queue: every commit on main is a proved tree

Status: rewritten 2026-11-21 as one queue on every forge; not started,
no issue filed. Phase 1 checks the forge facts the design rests on, and
its result decides whether phases 2 to 5 go ahead as written.

## The prompt (Willem)

> What kind of merge queue setup would give livery the most benefit?
> Willem, 2026-10-09

> We discussed implementing a cross forge merge queue before. can we
> dig up those plans and see what we can do in this space? Willem,
> 2026-11-20, after running a manual merge train on humain-scanner

> yes that is the level of uniformity I aspire to. Willem, 2026-11-21,
> on the model under "The design"

## What exists

Read in the code on 2026-11-21.

- `fm submit --armed` opens a squash pull request and arms the
  forge's auto-merge. A pull request lands on its own green, onto
  whatever main is at that moment. Branch protection on main does not
  require an up-to-date head.
  `packages/workshop/docs/development-patterns.md` states this under
  "What the forge does with several green pull requests".
- `fm submit.arm` and `fm submit.disarm` set and remove auto-merge,
  by number or on the current branch.
- The gate point runs on `pull_request` with every runner, the docs
  job and the gate job, the one required context. The gate job stamps
  the tree a green run proved. A narrowed run's row names the tree its
  delta was taken from, and is a proof when that tree is proved
  (`livery.workshop._gate_record`, `leg_plan`). The record keys trees,
  not commits.
- The merge point runs on `push` to main. A push whose tree a run
  already proved runs no leg; one whose tree no run proved narrows
  from the nearest proved tree.
- `livery.forge.PullRequests` has `open`, `get`, `list`,
  `find_by_head_sha`, `update_title`, `update_body`, `close`,
  `merge_now` (squash, with the title as the subject), `arm`,
  `disarm`, `is_armed`, `merge_hold`, `comment` and `add_labels`. It
  has no `remove_labels`, no `update_branch`, no draft field and no
  draft setter. `livery.forge.Checks` reads a commit's verdict and
  writes none.
- `livery.forge._merge_state` maps each forge's refusal to merge a
  draft to one state, `draft`: GitHub's `mergeable_state`, GitLab's
  `draft_status`, Gitea's "work in progress" message.
- `livery.forge.RepoConfig.labels` declares the labels governance
  creates; `ci:every-runner` is one, and the gate workflow already
  runs on `labeled` and picks its matrix in `fm ci.run`.
- `livery.forge.testing.FakeForge` with `Faults`, and one conformance
  suite run against the fake and recorded cassettes of all three
  backends.
- The loop, `fm ci.e2e --scenario=<name>`, runs on the local Gitea.
- The repository is owned by the `liveryhq` organization, so GitHub
  offers it the native merge queue. Gitea has no queue. gitlab-ce has
  no merge trains.

Measured from `refs/workshop-origin/metrics` on 2026-10-09, 300 runs
between 2026-10-03 and 2026-10-09:

| What | Value |
| --- | --- |
| Merges to main per day, last 21 days | median about 20, peak 37 |
| Pull request run, wall | p50 11.2 min, p90 15.6 min |
| Push run on main, wall | p50 5.1 min, p90 15.0 min |

A push run's p90 is a full leg re-run: main had moved under the pull
request, so the squash's tree was never proved, and a conflict between
two green pull requests is found on main after it landed.

## The reference: the train on humain-scanner

humain-scanner, a sibling repository, ran a manual merge train from
2026-11-09: trains 1 to 450 in eleven days. An orchestrator session
squashed approved pull requests one by one onto
`chore/merge-train-N`, ran one full gate on that tree, fast-forwarded
the base to it, and closed each pull request with a comment naming its
commit. One gate covered the whole train, so a red train named no pull
request: it was closed and rebuilt from the base without the pull
request its failing tests pointed at, chosen by the orchestrator. Its
scripts are outside that repository, in
`/Users/willem/humain-orchestrator/train-tools/`. Two of its rules
carry over: the queue records each entry's head and a moved head voids
the entry; landing and pushing the next candidate are separate steps.
Its red rule does not carry over as written: with every candidate run,
the red candidate names its entry and no choice or search is needed.

## Ground-truth contracts (do not violate)

1. **One queue on every forge.** What a person types, the shape of
   main, which trees are proved and what the record says are the same
   on GitHub, Gitea and GitLab. A forge capability changes who does
   the work, never what is promised.
2. **Every commit on main is a tree a run proved before it landed.**
   A red run names its pull request.
3. **Main is linear, one squash commit per pull request, its subject
   the pull request's title.** The changelog and the release train
   read it. The queue rewrites no title and no history.
4. **A pull request that lands reads merged on its forge's own page,
   and the forge makes and signs the commit.** No token the workshop
   holds can move protected main.
5. **The gate's record keeps its shape and its readers' rules.** The
   queue adds rows of today's shape. A row is a proof only when its
   chain reaches a proved tree.
6. **Nothing on the merge path waits on a person.** A queue step that
   cannot act prints why and exits; the next event runs it again.
7. **Every queue verb is idempotent.** Re-running the advance step is
   the recovery procedure, and a second run changes nothing.
8. **No logic in YAML.** The events are emitted plumbing; the
   decisions live in `fm submit`, the advance step and `fm ci.run`.
9. **The forge is spoken to through `livery.forge` alone**, and a
   feature a forge lacks is probed with `supports()`, never declined
   by a constant.
10. **Every runner runs on what lands.** The ruling of 2026-10-13
    moves from the pull request's run to the candidate's run.
11. **A draft is never queued.**
12. **Fallbacks before happy paths.** A lost arm, a lost update, a
    red candidate, a moved head, a tree mismatch, a conflicting
    rebuild and an empty queue are tested before the green path,
    through fault modes on the fake and the loop's scenario.
13. **The loop never runs from main.**

## The design

### The model

A queue is an ordered list of pull requests. Each entry has a
candidate: main's tip, then every entry ahead of it, then itself, each
as one squash commit made as the forge would make it. Two settings
describe every queue:

- **group size:** how many entries land on main at once;
- **which candidates run:** every one, or only the last of a group.

The baseline runs every candidate, in parallel up to a concurrency
the workspace sets, and lands each entry once its candidate and every
candidate ahead of it are green. Running only a group's last candidate
is batching, a setting described below.

### What every forge promises

1. `fm submit --armed` queues the pull request and prints its
   position.
2. Main stays linear, one squash per pull request.
3. Every candidate runs, so every commit on main is proved.
4. A red entry leaves the queue with its verdict; the entries behind
   it are rebuilt without it.
5. Each run writes a row of today's shape; main's push run finds its
   tree proved and runs no leg.
6. Each candidate's run selects through `fm ci.run`.
7. `livery.forge` answers whether a pull request landed the same way
   on every backend. `fm status` prints the queue.

### Membership

Membership is the label `queued`. `fm submit --armed` applies it; the
order is the time the label was applied, then the pull request number.

- **A draft is never queued.** `fm submit --armed` on a draft marks it
  ready, then queues it. Marking a queued pull request draft removes
  it from the queue. `fm submit` without `--armed` opens a draft, so
  every forge's page shows "not queued" in its own words. The forge
  refuses to merge a draft, which guards against a merge around the
  queue.
- **A push to a queued pull request voids its entry.** The queue
  records each entry's head. A head it did not record removes the
  label, and the entries behind are rebuilt without it.
- **A stacked child stays unqueued while its parent is open**, as it
  stays unarmed today.

`livery.forge.PullRequest` gains `draft: bool`, and its `title` never
carries a draft prefix: GitLab and Gitea mark a draft by a title
prefix, and each backend strips and writes it, so the title means the
same on every forge.

### Candidates and their runs

The advance step builds each candidate as a branch under `queue/`,
squashing each entry's change onto the one before it, and pushes it.
A push to `queue/` starts the gate point's run on that candidate.

**Candidate k measures from candidate k-1's tree.** Its run gates only
entry k's change, and its row names k-1's tree as its base. By the
record's chain rule the row is a proof once k-1's is, so the queue
lands entry k only when the chain from main's tip to k is green. Each
entry then pays for its own change alone, whatever its place in the
queue. An entry that touches a module most tests reach selects nearly
everything, and only its own candidate pays for that.

### Pull request runs

A pull request run runs on every push, except while the pull request
is queued; the candidate's run covers it then.

- Opened and queued at once, the usual `fm submit --armed`, a pull
  request gets no pull request run. Its candidate's run is its only
  run.
- Queued while its pull request run is going, that run is cancelled.
- Dropped after a red candidate, removing the label starts a pull
  request run on its own head. Its verdict says whether the failure is
  the change's own or comes from the entries ahead.
- An author who wants a verdict on every platform before queueing
  pushes with `fm submit`, waits for the pull request run, and queues
  with `fm submit.arm`.

### The required check

Each forge requires the `gate` context on a pull request's head
before it merges. The candidate ran on another commit, so the gate job
posts the `gate` status on each entry's head when the chain to it is
green, naming the candidate run that proved it.
`livery.forge.Checks` gains the write.

### Landing

When the chain to entry k is green, the advance step calls the
forge's own squash merge for each entry up to k, in queue order, with
the pull request's title as the subject. The gate record keys trees,
so each merge's commit id differs from the candidate's and its tree
does not: the advance step compares main's tree after each merge with
the candidate's tree at that entry.

- Equal: the push run finds the tree proved and runs no leg.
- Different, or main moved by a merge around the queue: nothing is
  undone. The push run narrows and runs its legs, which is today's
  behaviour, and the advance step rebuilds the candidates behind from
  the new tip.

The forge marks each pull request merged, makes and signs the commit,
closes the issue the branch names and deletes the branch, as a merge
does today. Where the forge needs the head brought onto the base
before a squash (GitLab's fast-forward method), the advance step calls
`update_branch` first; the tree it yields is the candidate's.

### Red

Candidate k is candidate k-1 plus entry k. When k-1 is green and k is
red, entry k is the only change added to a green tree, so the run
names the entry and no search is needed. This holds for a failure that
needs an entry ahead of k too: the tree without k was green, so k is
the entry to drop, and its own pull request run, started when it
leaves, says whether the failure is the change's alone.

A red candidate's entry leaves the queue: the label is removed, the
verdict is commented on the pull request, and the candidates behind
are rebuilt from the last green one. A rebuild that conflicts drops
that entry the same way and names the conflict. An empty queue prints
`queue empty` and exits 0.

A platform-only failure in entry k voids the runs of every candidate
behind it, since each contains k. The local gate before `fm submit`
covers one platform. Open item 5 measures how often this happens.

### GitHub's native queue

`livery.forge.Capability` gains `merge_queue`. GitHub answers true
when a ruleset with a `merge_queue` rule targets the default branch;
Gitea and GitLab CE answer false after the probe. With the capability,
GitHub builds the candidates, starts their runs through the
`merge_group` event and lands the entries; `fm ci.run` on that event
selects and stamps as on a `queue/` push, and the advance step builds
nothing. What it adds over the workshop's queue: a ruleset that makes
the queue the only way into main, and the forge's own queue page.

### Batching

A workspace may declare that only the last candidate of a group runs,
for a group of up to N entries. One run then proves N entries; the
intermediate commits on main are covered as a group and their rows say
so. A red group names no entry, so it runs every candidate of the
group in parallel, the baseline mode, each selecting only its own
change from the candidate ahead. That takes one round of wall time and
finds every red entry of the group at once. A bisect takes log2(N)
rounds one after another and a further search for each further red
entry, so it pays only for groups large enough that N runs cost more
than those rounds (open item 9). Batching is off by default. A forge
whose native queue cannot do it refuses it by name at
`fm workflow.configure`.

### What it costs and does not cover

- A run per pull request, as today, started at queueing instead of at
  opening.
- A red entry costs every candidate behind it a run.
- Main's tree is re-proved by the push run when a forge's squash
  yields another tree; no worse than today.
- A merge made around the queue on GitHub without the native queue,
  or on Gitea or GitLab, is caught after it lands, by the tree check.

## Phases

### Phase 1: the forge facts, by conformance

Deliverables:

- `livery.forge.PullRequests.update_branch(number)` (GitHub
  `PUT pulls/{n}/update-branch`, Gitea `POST pulls/{index}/update`,
  GitLab `PUT merge_requests/{iid}/rebase`), answering the head after
  the update and raising `livery.forge.ForgeError` naming a conflict.
- `remove_labels`, `PullRequest.draft`, `mark_draft` and
  `mark_ready`, with the title free of the draft prefix on every
  backend.
- A status write on `livery.forge.Checks`.
- Fake faults `Faults.lose_update` and `Faults.lose_label`.
- Conformance scenarios, recorded on all three forges:
  - `squash-tree`: squashes made by the workshop, then the forge's
    squash merges in order, yield equal trees;
  - `posted-status`: a status posted on a head satisfies the required
    check, including beside a cancelled job of the same name;
  - `checked-push`: a push of a commit with green required checks to
    protected main is accepted, and one without is refused;
  - `draft`: open, mark ready, mark draft, and the merge refused while
    draft;
  - `update-branch`: a behind head is updated, a conflicting one
    raises naming the conflict;
  - `queue-events`: which events each forge delivers for a label
    change, a draft change and a push to `queue/`.
- `packages/forge/docs/protocol.md` gains the verbs;
  `packages/forge/docs/quirks.md` gains what each forge does
  differently.

Tests, refusals first. If `squash-tree` fails on a forge, the plan
stops for Willem before phase 2.

**Acceptance**

- `fm forge.dev.up`, then `fm forge.fixtures.record --scenario=<name>`
  for each scenario records the cassette sets, and
  `fm forge.conformance --scenario=<name>` replays green on the fake
  and all three backends.
- `fm check` green.

### Phase 2: the queue in the workshop

Deliverables:

- The `queued` label, declared through `RepoConfig.labels`.
- `fm submit --armed` labels and prints the position; `fm submit`
  opens a draft.
- The advance step (open item 1 names it): builds the `queue/`
  candidates, lands, drops red entries, rebuilds, and runs from the
  merge point, from a candidate run's verdict and from the pull
  request events.
- `fm ci.run`: a candidate measures from the candidate ahead; a queued
  pull request's own run skips; the gate job posts `gate` on each head
  whose chain is green.
- `fm status` prints the queue.
- Docs: `packages/workshop/docs/development-patterns.md`, "What the
  forge does with several green pull requests" rewritten to the
  queue.

Tests, refusals and fallbacks first: a lost update re-done, a lost
label re-applied, a moved head voids its entry, a draft refused, a red
candidate dropped and the ones behind rebuilt, a conflicting rebuild
dropped, a tree mismatch falls back to the push run's legs, an empty
queue, the advance step twice changes nothing; then the green path.

**Acceptance**

- With one pull request queued, `fm submit --armed` from a second
  branch prints `queued behind 1 pull request`.
- The advance step run twice prints the same queue, and the second run
  changes nothing on the forge.
- `fm check` green.

### Phase 3: the loop proves the queue

Deliverables: scenario `queue` in `livery.workshop._e2e`, in the `all`
set and runnable by name. On the local Gitea: three branches queued,
the third carrying a failing test. Pinned lines: the first two
candidates run in parallel and land in order, each pull request reads
merged, the third is dropped naming its verdict, its own pull request
run starts, the queue reads empty, and every push run's gate prints
`proved by run N` and runs no leg.

**Acceptance**

- `fm ci.e2e --scenario=queue` passes with the pinned lines in the
  pass's log.
- `fm store.show metrics` shows the pass's push runs with no leg.
- `fm check` green.

### Phase 4: GitHub's native queue

Deliverables: the `merge_queue` capability and its probe;
`RepoConfig.merge_queue`, declared by `[ci] merge-queue = true`, and
governance writing the ruleset `workshop-merge-queue`; the gate
point's `merge_group` event; `fm ci.run` on that event; the advance
step building nothing where the capability is present. Conformance
scenario `merge-queue` recorded on a scratch repository owned by an
organization.

**Acceptance**

- `fm forge.fixtures.record --backend=github --scenario=merge-queue`
  records and `fm forge.conformance --backend=github
  --scenario=merge-queue` replays green.
- On `liveryhq/livery`, after `fm workflow.configure` with
  `merge-queue = true`, the next `fm submit --armed` enters the queue,
  the pull request merges by it, and the push run's gate prints
  `proved by run N` with no leg.
- `fm check` green.

### Phase 5: batching

Deliverables: `[ci] queue-group = N` in `workshop.toml`, the group's
row naming its entries, the parallel run of every candidate of a red
group, and the refusal on a forge whose queue cannot batch. A loop
scenario `queue-group`, with a red group of three whose second entry
fails.

**Acceptance**

- `fm ci.e2e --scenario=queue-group` passes with one run for a green
  group of three, and for the red group one round of three candidate
  runs that drops the second entry and lands the other two.
- `fm check` green.

## Temporary, replaced by

| Temporary | Replaced by |
| --- | --- |
| Candidates select at package grain | per-file selection, `notes/20261012-what-a-change-affects.md` phase 11, switched on for candidates and pull requests together |

## Decision record

- 2026-10-09: Willem asked which merge queue setup gives livery the
  most benefit, and after the answer asked for it as a future plan.
  The plan records the answer: a serial queue the workshop drives on
  every forge first, GitHub's native queue second as a probed
  capability. Not started; no issue filed.
- 2026-10-09: the facts under "What exists" were read in the code
  and measured from `refs/workshop-origin/metrics` (300 runs,
  2026-10-03 to 2026-10-09) and `origin/main` (21 days) that day.
- 2026-10-09, instead of asking: only the head of the queue is
  armed, membership is a label, and the order is the pull request
  number. One path on every forge, because GitLab CE cannot block an
  outdated merge. The verb is `fm queue.advance` and the label
  `queued`; both names are open item 6.
- 2026-10-09, instead of asking: the `queue` scenario joins the `all`
  set and not `develop`, which stays the fast default; it is also
  runnable by name.

- 2026-10-13, Willem: "if enabling the github merge queue would make
  this faster then yes, I'm all for it." Read the same day: GitHub
  does not give a personal account's repository a merge queue, so
  phase 4 needs the repository under an organization first; open
  item 7 asks which way to go.

- 2026-10-13, Willem, on open item 1: the pull request point keeps
  every runner. "We did that, and then we had issues with incomplete
  coverage, not to mention that windows and macos are much closer in
  speed now we've optimised our git usage, so it wouldn't save that
  much." Pull requests have run every runner since 2026-10-12 (#1499).

- 2026-10-13, Willem: "No merge queue for now until we can do it
  properly, too much of a distraction now." The plan waits; open item
  7 stays open for when it is taken up.

- 2026-11-18: the repository moved to the `liveryhq` organization,
  which closes open item 7 of 2026-10-13: GitHub offers it the native
  queue.
- 2026-11-20, Willem reopened the plan after the train on
  humain-scanner (the prompt). He named the cross-forge queue as the
  goal, and said he will turn GitHub's native queue on.
- 2026-11-21, Willem, "yes that is the level of uniformity I aspire
  to", on the model the plan now describes: one queue on every forge,
  every candidate run, landing through each forge's own squash merge
  so each pull request reads merged and the forge signs, GitHub's
  native queue as a capability on top, batching as a setting off by
  default. This replaces the head-only arm of 2026-10-09.
- 2026-11-21, Willem confirmed that the candidate's run replaces the
  pull request's run, so the every-runner ruling of 2026-10-13 moves
  to the candidate (contract 10); and asked that pull request runs
  still run on a pull request's pushes while it is not queued.
- 2026-11-21, Willem agreed that drafts are a queue signal on every
  forge: a draft is never queued (contract 11), `livery.forge` reads
  and sets it uniformly, and `fm submit` without `--armed` opens a
  draft.
- 2026-11-21, Willem: candidates select per test file once the
  affected plan's selection lands, which answers that plan's open
  item 5 with yes. Instead of asking: candidate k measures from
  candidate k-1, so every entry pays the single-change rate.
- 2026-11-21, Willem asked why a red entry is never bisected. With
  every candidate run, the red candidate names its entry, so no search
  is needed; the note's "never bisected" came from humain-scanner,
  where one gate covered a whole train. A red batched group runs every
  candidate of the group in parallel, replacing one at a time, and a
  bisect for large groups is open item 9.

## Open

1. Willem: the names: the advance step's verb, the label `queued`
   and the branch prefix `queue/`.
2. Willem: the order against `notes/20261012-what-a-change-affects.md`.
   Recommendation: phases 1 to 3 here first on package-grain
   narrowing, since that plan's open items 1 to 4 are still unruled.
3. Agent, phase 2: the default concurrency of candidate runs, from
   the runners `[ci] runners` names.
4. Agent, phase 2: `leg_plan` accepting a candidate's base that is
   still running, and the coverage union on chained narrowed legs.
5. Agent, then Willem: how often a pull request goes red on Linux or
   Windows after a green local gate, read from the metrics store
   before phase 2; whether a rule asks platform changes to run
   `fm submit` and its pull request run before queueing.
6. Agent, phase 2: the permissions the gate job's token needs on each
   forge to label, post statuses and merge another pull request; the
   step names any it lacks.
7. Agent, phase 4: whether GitHub's queue runs one candidate per
   entry, which sets its runner cost; whether it needs
   `block_on_outdated` off.
8. Agent, phase 4: GitLab Premium's merge trains as the same
   capability, tested as a decline on gitlab-ce until a Premium
   instance gives a cassette.
9. Agent, phase 5: the group size above which a red group is bisected
   instead of run as every candidate in parallel, from the runner
   minutes and the wall time each costs; none below it.
