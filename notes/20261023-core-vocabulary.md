# The core's vocabulary: triggers, trust, access, and what moves out

Status: a design record, written 2026-10-23 from the discussion of
2026-10-21 to 2026-10-23. It is not a plan and builds nothing. It
proposes a direction, records Willem's rulings inside it, and lists the
recorded rulings it would reopen, each for Willem to confirm or reject.
The plans it leads to are listed at the end, in the order proposed. It
supersedes phases 1 to 3 of `notes/20261021-forge-identities.md` once
the reopened rulings are decided.

## The prompt (Willem)

> I think there is a way to flatten/decompose trigger and jobs role
> into a single set of steps that you can contribute a task to. And the
> token given to task is decided purely on the step that it is. Willem,
> 2026-10-21

> the word wheels should not exist in this contract, that's a python
> specific thing the core should know nothing about. and the amount of
> edge cases (pages grant, index token, the propose thing still feels
> warty) Willem, 2026-10-21

> I think I'm really looking for a slightly different level of
> vocabulary, on which we can build our current setup ( protected main,
> pr only, builds built and published on release pr merged to main ).
> Not everyone will want to run their project that way, but it's the
> way I currently like to work. Willem, 2026-10-21

> I think we should have a CI backend render separation within forge,
> so we can support things like jenkins etcetera in the future if we
> want, and that means that the extension's contribution to ci needs to
> support that too. Willem, 2026-10-22

> do we still need explicit seeds? can we not express our needs as part
> of the fragment engine? one less system to maintain. Willem,
> 2026-10-22

The intent: the core speaks a neutral vocabulary in which any way of
working can be expressed; Willem's way of working is one composition
of it, shipped as an extension; every forge-specific and
ecosystem-specific fact lives in the backend or extension that owns it;
and the core keeps fewer systems than today.

## Where it started

The nightly's `fm ci.balance --submit` was refused its push on
2026-10-19 (run 39200159228): the job token may not change a workflow
file. The identity plan (`notes/20261021-forge-identities.md`) answered
it with roles on jobs. Willem found the result too complex: points,
jobs, roles, grants, copied grants for jobs a schedule entry creates,
the pages grant, the index token and a separate send job, each its own
case. The cause: the core describes one workflow's policy (gate, merge,
release, the train) instead of mechanisms, and knows ecosystems (wheels)
and forge services (pages) it should not.

## The design

### Triggers, trust and access

The core describes CI in three terms.

- **Trigger**: what starts a run.
- **Trust**: where the code being run came from. A trigger's code is
  untrusted when it comes from a proposed change, and trusted when it
  has landed on a protected ref.
- **Access**: what a task may affect, which decides the credentials it
  receives.
  - `minimal`: the run's own token. It reads the repository and the
    forge, and writes only the run's own records (state store refs,
    rows, traces). An omitted `access` is `minimal`.
  - `write`: the bot, and every credential an extension declares for
    `write`. It sends things out: uploads, deploys, pull requests back
    to the repository, receipt tags.
  - `admin`: the repository's settings: protection, secrets, the
    receipt-tag rule.

GitHub's "write" means pushing to the repository; this `write` also
covers sending to a package index or a site host.

Untrusted code gets `minimal` access only. So code from a pull request
never runs next to a credential.

The four triggers today's points become:

| Today's point | Trigger | Starts on | Trust | Access its tasks may have |
| --- | --- | --- | --- | --- |
| `gate` | `pull-request` | a pull request; a dispatch | untrusted | minimal |
| `merge` | `merge` | a change landing on main | trusted | minimal, write, admin |
| `nightly` | `scheduled` | the clock | trusted | minimal, write |
| `release` | `release` | the dispatch the merge sends | trusted | minimal, write |

`nightly` is renamed `scheduled` (Willem, 2026-10-21), and
`[[ci.schedule]]` becomes `[[ci.tasks]]`, since it holds every
trigger's tasks.

### Tasks, jobs and the CI manifest

A task names its trigger and, when it needs more than `minimal`, its
access; its matrix (runners, Pythons, shards) comes with it. Jobs are
derived: the render groups a trigger's tasks by access and matrix. The
contract names no job, and `[[ci.point]]`, the schedule's `job` key and
`once` go; the toolroom bench's `tool-hosts` point becomes a scheduled
task with its own runners.

```toml
[[ci.tasks]]
trigger = "scheduled"
task = "toolroom.verify-host"
every = "2w"
runners = ["ubuntu-latest", "ubuntu-24.04-arm", "macos-latest",
           "macos-15-intel", "windows-latest", "windows-11-arm"]
```

A task that proposes a change (the balancer, the refresh, the fleet's
`workflow.update`) runs with `minimal` access and writes the change as
an artifact; a task of the workflow with `write` access submits it. No
separate mechanism remains.

The render writes a committed, drift-checked CI manifest beside the
workflows: per trigger and access, the tasks, their arguments and
cadence, the extension each comes from and the credentials each
receives. A job's first step is a standard-library entry that reads it,
not `fm ci.run`, so no extension code loads before the credentials are
split. A task that holds a credential mounts only its own extension and
what that requires (the narrow mount); every other task runs with the
credential removed from its environment. The manifest must answer the
five facts the empty-shell plan's deferred manifest dispatch listed
(`notes/20260930-empty-shell-plan.md`, Open 1): `tasks.py` is arbitrary
Python, `pre_tasks` hooks edit the live tree, lifecycle hooks import on
every call, generated verbs are closures, and `requires_*` gates are
evaluated live; or say why a CI-only manifest avoids them.

### Credentials

A credential is declared by whoever needs it: the core declares the
forge's (`bot`, `admin`, `org`), an extension declares its own. The
declaration names the access it belongs to; the core hands it only to
the declaring extension's tasks with that access, on trusted code,
without an extra runner (ruled 2026-10-21).

```toml
[credentials.package-index]
doc = "uploads the release's distributions to the package index"
access = "write"
optional = true
reads-as = "UV_PUBLISH_TOKEN"
secret = "PYPI_TOKEN"
create = "https://pypi.org/manage/account/token/"
```

`optional` means "not used when the extension's own settings make it
unnecessary". Whether python uses trusted publishing is python's
setting, not the credential's, and it refuses on Gitea (ruled
2026-10-21):

```toml
[python]
trusted-publishing = true
```

Identity tokens (OIDC) are granted in every job with `write` access and
in no other: `id-token: write` on GitHub, `id_tokens:` on GitLab. An
extension declares the audience and the variable a tool reads
(`audience = "pypi"`, `reads-as = "PYPI_ID_TOKEN"`) on every forge,
though GitHub's render ignores them: an extension's declaration means
the same on every forge, and forge differences live in the render
(Willem, 2026-10-21: "the configuration being able to be cross forge
compatible").

One verb pair sets up and reports every credential, the core's and the
extensions': `fm credential.setup <name>` and `fm credential.status`,
reading each declaration for what it is, where to create it and
whether it is needed under the current settings.

### Two backend axes in `livery.forge`

`livery.forge` gets a second kind of backend (Willem, 2026-10-22):

| Axis | Answers | Backends |
| --- | --- | --- |
| repository host | the repository, review, protection, releases, site hosting | github, gitlab, gitea |
| CI host | how runs start, are written, get secrets and run | github-actions, gitlab-ci, gitea-actions |

The CI host defaults to the forge's own (`[ci] host`). The workshop
builds the neutral description (the manifest's content); each CI
backend renders it into its files and declares its capabilities, and a
missing capability refuses, naming host and capability. The render in
`livery.workshop._ci_generate`, its `ACTIONS` pin table and its
per-forge permissions move there. Where secrets are stored and the
`run` credential's source are CI-host facts; required checks stay with
the repository host.

Extension CI contributions are neutral by default (tasks, access,
artifacts, credentials, identity-token audiences, caches, runners);
host-keyed pieces (`[ci.hosts.github-actions.write]`: steps on pinned
actions and the permissions they need) are the exception, for forge
features no task can reach, and an extension names the hosts it
supports.

Kept possible, not built (Willem, 2026-10-22): Jenkins as a CI host,
with a minimum plugin list its backend checks through
`/pluginManager/api/json` and extensions exposing plugin features
through host-keyed contributions; Gerrit as a repository host, which
needs the review unit generalised from "a pull request of a branch" to
"one or more reviewable changes", and has no issues and no CI of its
own.

### Destinations

An artifact sent to a destination, with that destination's credential
and settings, is one building block: a site to a host, a package to an
index, a binary to a forge release, an image to a registry. A
destination extension contributes a task with `write` access, its
credential and its contract keys (composed into the schema, so the
editor completes them). docs builds the site and names a destination
(`[docs] publish`); it no longer knows which forges host sites.

`pages` becomes a destination extension implemented for GitHub (the
pages actions and grant) and GitLab (the `pages` job), refusing on
Gitea; the core render loses `_PAGES_GRANT`, `_PAGES_DEPLOY` and the
`pages` case in `_grants`. Other hosts (Netlify, a bucket, SSH) are
other destination extensions (Willem, 2026-10-22: "it should be easy to
use something other than pages on github too").

### Workflows

A workflow is a named composition of building blocks, shipped as an
extension:

1. branch roles and rules (which refs, how changes arrive, merge
   method, required checks, who may push), which define trust;
2. routes between branches (to trunk, promotions, back-merges,
   backports);
3. verification policy (what runs on proposed and on landed code, the
   verified skip, a merge queue);
4. the release trigger (a release pull request lands, a tag, every
   landing, a schedule, a dispatch);
5. the version source (conventional commits, change files, calendar);
6. the release record (receipt tags, forge releases, changelogs);
7. how automated changes are sent (pull request, armed or not, or a
   push);
8. review policy (approvals, code owners);
9. work tracking.

Willem's workflow, the one this repository runs, moves out of the core
into such an extension: the repository rules `workflow.configure`
asserts (squash only, auto-merge, the required context, approvals, the
receipt-tag rule); the gate as a merge requirement and the verified
skip (`ci.verified.stamp`); the release train (`workflow.release`, its
title and freshness checks, the merge's dispatch, the publish wave,
receipts, `release.verify`, `workflow.abort`); the development verbs
(`start`, `submit`, `submit.arm`, `submit.disarm`, `submit.merge`,
`integrate`, `abandon`, `status`); and the janitor's sweep of merged
worktrees. In common names it is scaled trunk-based development (or
GitHub Flow) with a release-pull-request train in the manner of
release-please and Changesets. The test of the blocks: express it and a
contrasting workflow (trunk with pushes, releases on tags) from the
blocks alone.

### Work tracking is optional

Repositories disable issues by choice (Willem, 2026-10-22), and today
nothing reads that a repository has none. Work tracking becomes an
extension: listed, the `issue.*` verbs exist, `start` accepts an issue
number or title, a merge closes its issue, and governance turns issues
on; not listed, those verbs do not exist and governance leaves the
setting alone. Its footman plugin defines `start` and `submit` marked
`@task(override=True)`, handles the issue and calls the base's task
(Willem, 2026-10-22). Footman takes `override` today only as a mount
parameter (`livery.footman._compose._merge_group`); a task-level flag
is a small footman change. A workspace's own `tasks.py` still wins.

### Composition: the requiring extension wins

Steps' `deletes` and `replaces` (used five times, all by nanobind) and
shipped files' `[replaces]` and `[deletes]` (used by no extension)
go. Instead:

- outputs are named by what they are (`python-distributions`,
  `conan-package`), not a shared `dist`;
- one provider per output per package: of two, the step of the
  extension that requires the other wins, and unrelated providers
  refuse, naming both;
- a step runs only when its inputs are provided; a read no extension of
  any set could provide refuses statically, and a step skipped for a
  missing provider shows in `fm explain`;
- a single-owner file follows the same rule.

### Delivery: one engine for kept, written-once and template files

Seeds (seven files) and package templates (four, 35 files) fold into the
fragment engine:

- a fragment can be written once: written when absent, recorded as
  delivered in the package's or the root's receipt, never written
  again, so a deleted file stays deleted;
- `when = "<answer>"` replaces files that render to whitespace;
- a template becomes a preset: a name, the extensions a package born
  from it lists, the answers it fixes, and only the files that belong
  to no extension; the starter files become the extensions' own
  written-once fragments, and tool configuration such as cmake's ignore
  lines becomes kept fragments;
- a birth writes the package's contract and syncs;
- an answer is read back from the file it leaves a trace in where there
  is one, and otherwise recorded in the receipt at birth (Willem,
  2026-10-22); a verb re-answers it. Receipts are per root and per
  package, so a package's answers are written by the branch that
  creates it and never conflict.

## Rulings to reopen

Each is a recorded ruling the direction reverses. Owner: Willem, one by
one.

1. **"'point' is the vocabulary (never 'stage', which GitLab's
   pipelines own)"**, 2026-09-07 (`notes/20260907-ci-quality-plan.md`).
   Proposed: points become triggers; "stage" stays unused.
2. **"The matrix shape belongs to the point, not the task"**,
   2026-09-07 (the same note). Proposed: a task brings its matrix.
3. **A job's identity is its point and name; a contributed point
   declares no permission, no secret and no environment;
   `[[ci.point]]`** (`notes/20260914-ci-declared-and-dispatched.md`,
   contracts 1 and 4, phase 5). Proposed: jobs are derived; contributed
   points become tasks.
4. **`once = true`**, 2026-10-11 (destination note). Proposed: a task's
   default matrix is the first runner and Python.
5. **`[ci.jobs.<point>.<name>]` stays until the review in #1534**,
   2026-10-12 (destination note, Open 5). Proposed: answered, jobs are
   derived; #1534 closes.
6. **An extension's CI job takes the run's own token alone (#1280)**,
   2026-10-07 (destination note). Proposed: an extension declares
   credentials for an access level, received by its own tasks on
   trusted code, kept from other extensions' code by the manifest's
   entry and the narrow mount. This is #1280's (a) with (c), not its
   recommended (d).
7. **Steps' `replaces` and `deletes`** ("one mechanism", 2026-10-10;
   nanobind's deletes, 2026-10-14; N2 and #1685, 2026-10-15), and
   **files', fragments' and seeds' `REPLACES` and `DELETES`**,
   2026-10-03 (extensions plan). Proposed: the requiring extension
   wins, derived from outputs.
8. **A seed is written at birth and "never compared or rewritten"**
   (extensions plan, "Seeds and birth"). Proposed: written-once
   fragments, delivered by any sync while absent and recorded.
9. **Templates used "at birth alone"**, 2026-10-14; **one verb per
   template**, 2026-10-16; **answers are not stored**, #1672,
   2026-10-15; **no persisted key as "a second truth beside the native
   manifest"**, 2026-10-14. Proposed: presets; answers read from files,
   else recorded in receipts as a record of the birth, not a setting.
10. **"What the base keeps: the issue and submit family, the
    reserved-branch workflow engine"**, 2026-09-30
    (`notes/20260930-empty-shell-plan.md`, repeated in the extensions
    plan). Proposed: they move into a workflow extension and a
    work-tracking extension.
11. **The development-workflows yardstick's assumptions**
    (`notes/20260830-development-workflows.md`): W1 and W3 assume
    protected main and pull requests only, W10 and W11 assume issues.
    Proposed: the yardstick judges the building blocks, and W1 to W12
    are restated per workflow.

## How it meets the plans already written

- **Merges with destination phase 13 and #810.** Phase 13's ecosystems
  as registrations, an open `livery.forge.RegistryKind` and the forge's
  `package_admin`, and `notes/20260927-releases-as-a-target.md`'s
  `[[registries.<kind>]]` with a store per entry, describe destinations
  twice. One plan.
- **Overlaps destination phase 15** (lodges, e2e, the admin protocol)
  with the forge's two axes and the identity plan's `org` role.
- **Reshapes destination phase 16** (`notes/20261012-what-a-change-affects.md`):
  its phase 1 gives a CI job a `reads` spelling; with derived jobs it
  targets tasks, or follows the CI model. Its `reads` sits close to the
  composition's "a step runs when its inputs are provided".
- **The GitHub lane** (`notes/20261017-github-lane.md`): its open items
  2 and 3 are the identity plan's App and `org` role; its scheduled
  `workflow.update` becomes a scheduled task; "one example per
  template" becomes one per preset; a red example's issue needs the
  work-tracking extension listed.
- **The merge queue** (deferred): its advance runs from the gate's
  verdict, untrusted under this design, and it needs a trigger for
  GitHub's `merge_group` event; both are revisited when it returns.
- **The Rust plan**: cargo's trusted publishing follows python's
  setting; the cargo registry ladder is a package-index destination;
  its loop seeds become written-once fragments. No conflict.
- **Ready work to order**: #1736 P2b selects tests by the point names,
  so it lands before the rename or after it, not across it; #1309 (an
  unmounted extension deletes its rendered files) is fixed before
  written-once fragments exist; #1729 (nanobind cooperates with conan)
  is the composition change's first case, and its wheels job becomes
  derived. Unaffected: the Rust plan's phase 9, destination 12b to 12d,
  #1843.

## The plans it leads to, in the order proposed

1. **The rename and the series migration.** `nightly` to `scheduled`,
   `[[ci.schedule]]` to `[[ci.tasks]]`, points to triggers (`gate` to
   `pull-request`); a one-off hidden task migrates the store series that
   carry the name (`durations/nightly-*` keys, three `metrics` rows,
   the `refs/workshop-trace/run/<run>/nightly-*` refs), run once after
   the rename merges and removed in the next change. First, before the
   identity work (Willem, 2026-10-21).
2. **The CI model and the forge split**: access per task, derived jobs,
   declared credentials, identity tokens, the manifest and the narrow
   mount, the CI-host axis, neutral and host-keyed contributions,
   `pages` as a destination extension. It absorbs the identity plan's
   phases 1 to 3.
3. **Identities**: `notes/20261021-forge-identities.md` phases 4 to 10,
   restated on top of 2.
4. **Destinations**: destination phase 13 and #810 as one plan.
5. **Composition**, with #1729.
6. **Delivery**, after #1309.
7. **Workflows**: the building blocks, Willem's workflow extension, the
   work-tracking extension and footman's task-level override, and the
   yardstick restated.

The order of 4 to 7 is Willem's; each depends on 2 only where it
touches CI.

## Housekeeping found on the way

Five status lines lag the record: the destination note still says
"Phase 11 started"; the extensions plan still says "the rest is not
started"; the Rust plan says nothing is built, though its phase 1
landed inside coverage's P4; the GitHub lane says nothing is built,
though phase 1's render half landed; the sharding plan lists phase 3 as
in review.

## Decision record

- 2026-10-21, Willem: `nightly` is renamed `scheduled`, with a one-off
  migration of the series it names, before the identity plan.
- 2026-10-21, Willem: an extension declares its own credential, marks
  it optional, binds it to the access level it is used at, and only its
  own tasks receive it, without a separate runner.
- 2026-10-21, Willem: trusted publishing is python's setting, refused
  on Gitea.
- 2026-10-21, Willem: an extension's declarations are valid on every
  forge; identity-token audiences are required everywhere.
- 2026-10-22, Willem: a CI-host backend axis inside `livery.forge`;
  extension CI contributions must support it.
- 2026-10-22, Willem: Jenkins and Gerrit stay possible and are not
  built.
- 2026-10-22, Willem: issues are optional; the verbs do not exist where
  work tracking is not listed; the extension overrides `start` with a
  task-level `override=True`.
- 2026-10-22, Willem: site hosting other than pages must be easy on
  GitHub; pages is an extension.
- 2026-10-22, Willem: seeds and templates fold into the fragment
  engine; answers with no file trace are recorded in committed
  receipts.
- 2026-10-23, Willem: write this record before the plans, and check
  the plans already written first (done above).
- 2026-10-23, Willem: no "occasion" and no "effect". Triggers alone
  start runs; `access` is `minimal`, `write` or `admin`, `minimal` when
  omitted.

## Open

1. **The rulings to reopen**, 1 to 11 above. Owner: Willem.
2. **The trigger values** `pull-request`, `merge` and `release` beside
   the ruled `scheduled`. Owner: Willem.
3. **The manifest's five facts**: answered or avoided, before plan 2's
   phase that writes it. Owner: me.
4. **The role name `bot`** (the identity plan's open 1). Owner: Willem.
