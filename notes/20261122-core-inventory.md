# What the core holds, and which hooks a workflow extension would need

Status: an inventory, taken 2026-11-22 at main `eea997acb`. It is not a
plan and rules nothing. It answers one question for the single plan that
is to replace the merge queue plan, the affected plan, the core vocabulary
record and the identities plan.

## The prompt (Willem)

> if we can design it like that, yes. First I'd need us to look into what
> would be left of the core and if we have all the extension hooks we need
> to shift such a large part of functionality out. Willem, 2026-11-22, on
> the whole workflow leaving the core

## How it was taken

A reading agent classified every module of
`packages/workshop/src/livery/workshop/` by what it does, counted lines,
traced imports between the parts with `ast`, and read the extension schema
in `contract.toml` and the mount code. "Raw" is `wc -l`. "Code" leaves out
comments, blank lines and docstrings. The bucket of a module is the
agent's judgement where its name does not say. I did not re-read each
cited line. `W` is the workshop's source directory, `F` is footman's.

## What the core holds today

103 Python files, 58,408 raw lines, 34,808 code lines, 112 verbs.

| Part | Modules | Raw lines | Share |
| --- | --- | --- | --- |
| The engine: contracts, declarations, mounting extensions, phases and steps, queries, fragments, sync, drift, the tool store | 37 | 22,584 | 38.7% |
| The CI model: points, the workflow render, run context, CI verbs, metrics, traces, fold, shards, balance, speed | 15 | 11,209 | 19.2% |
| The development workflow: start, submit and arming, integrate, verdict and status, the reserved-branch engine, updates, governance, commit | 16 | 6,467 | 11.1% |
| Development tooling: the loop, devenv, the scale bench, doctor | 5 | 6,073 | 10.4% |
| The release train | 7 | 3,398 | 5.8% |
| Environment, run, clean, test support, store reading | 14 | 3,460 | 5.9% |
| The gate and its proof: check, the affected graph, the gate record, the verified record | 6 | 2,959 | 5.1% |
| Birth: new project, seeds, templates | 2 | 1,693 | 2.9% |
| The issue verbs | part of 1 | 565 | 1.0% |

The largest modules: `_e2e.py` 3,816, `_declaration.py` 2,318,
`_extensions.py` 2,227, `_points.py` 2,107, `_tools.py` 1,926,
`_steps.py` 1,907, `_release_driver.py` 1,801, `_submit.py` 1,736.

## What would be left

| If these leave | Remains, raw lines | Share of today |
| --- | --- | --- |
| The workflow, the release train and the issue verbs | 47,978 | 82.1% |
| The same, plus development tooling and the workflow render | 40,562 | 69.4% |

The workflow, the release train and the issue verbs are 10,430 lines and
32 verbs. Moving them takes 18% out of the core. The engine and the CI
model are 58% together and stay under every proposal so far.

## The bill of the move

The cost, stated as cost.

- **46 private names of the core that the moving parts read** become
  public. 13 further names they read are public already. By target: 34 in
  the engine, 14 in the CI model, 7 in the gate. The release driver alone
  reads 26.
- **34 names of the moving parts that staying modules read**, 27 of them
  private. The base cannot depend on its own extension, so each of these is
  turned around or the name stays in the core. This is the difference from
  the 19 extensions that left before: nothing in the base read those.
  - `_forge_lane.py` (the repository on its forge) has 8 staying importers,
    `_tokens.py` has 4. Both are access to the forge and belong in the
    core.
  - `_verdict.py` holds the watcher that follows a run and the classifier
    of where a pull request stands. `_ci_tasks` and the loop import the
    watcher. The module splits.
  - `_balance.py` calls `start_here`, `commit_flow` and `submit_flow` to
    send its proposal. `_new_project.py` calls `assert_configuration` and
    opens the setup pull request. Each needs a public way to send a change.
  - The loop imports from submit, the release driver and the dev release.
- **Coupling by name**: `_points.py` names five workflow and release tasks
  in its builtin schedule, and declares the `release` point and the
  `merge.govern` job. The prefix `workflow/` is a literal in eleven places.
- **Tests**: 27 of the workshop's 110 test files import a moving name:
  620 test functions, 17,665 lines. 8 test files outside the workshop
  import private names of moving modules.
- **What the extensions read today**: no extension source imports a private
  name of the workshop. The allowance list has 2 entries, both the
  workshop reaching into `livery.forge` for the loop.

## The hooks

What a workflow extension needs from the core, against what exists.

| # | Need | State | What exists today |
| --- | --- | --- | --- |
| 1 | Contribute top-level verbs such as `submit` | Exists, unused | An extension's plugin mounts at the top level (`W/_extensions.py:816`). Every extension verb today sits in a group |
| 2 | Replace a verb of the core or another extension | Missing | The mount passes no `override`. A workspace's `tasks.py` can replace a verb; an extension cannot |
| 3 | Contribute repository settings: labels, merge methods, protection, required checks | Missing | `RepoConfig` is built from literals in `contract_config` (`W/_workflow_tasks.py:204`) |
| 4 | Contribute CI events or triggers | Missing | Four event names and the points are literals (`W/_points.py:62`, `:236`). The render writes the pull request types as literals |
| 5 | A CI task with more than the run's token | Partly | An extension's job takes `token = "job"` alone. It can join a builtin job that holds more, as nanobind joins `release.publish` |
| 6 | Contribute to `fm status` and the verdict | Missing | The states and exit codes are fixed in `W/_verdict.py`, which asks no query |
| 7 | Contribute a state store series; read the gate's records | Exists; partly | The `store.series` query. The local gate record and `_verified.record` are not public |
| 8 | Own a root contract table | Exists | `[contract.root.<table>]`, as `[docs]` and `[python]`. `[release]`, `[owners]`, `[issues]`, `[forge]` are the core's today |
| 9 | Run the gate before a commit or submit, by public name | Partly | `_quality.check` is not exported. `run_phase` is |
| 10 | Worktrees, the sweep, the janitor | Partly | The `footman.sweepers` entry point exists. Worktree creation is core code with no hook |
| 11 | Run before every verb | Exists | `pre_tasks` and `pre_task` |
| 12 | Contribute a scenario to the loop | Missing | `SCENARIOS` is a literal tuple (`W/_e2e.py:3547`) |
| 13 | Be named by the default project list and by templates | Exists | `STOCK` (`W/_new_project.py:194`) |
| 14 | Reserve a branch prefix | Missing | A constant and eleven literals |
| 15 | Contribute to `fm doctor` and the task reference | Exists | The `footman.doctors` entry point; the task reference reads every plugin |
| 16 | The commit convention and the version source | Partly | The changelog writer is a query. The grammar, the title check and the next version are core code |
| 17 | Hooks for coding agents | Partly | They live in the claude extension, which also holds the rule against pushing a conflicting branch |
| 18 | Place a job entry first, or after the stamp | Partly | A join's entries go after the job's own |
| 19 | A point with dispatch inputs; job abilities such as `environment`, `pushes`, `dispatches` | Missing | Fields of the builtin jobs, with no key in an extension's job |
| 20 | Issue hooks in submit, the sweep and verify | Missing | The closes link is inside submit; a red `release.verify` files an issue |
| 21 | The package lifecycle calls the release train uses | Partly | The phases `stamp`, `publish`, `verify` are declared; ten functions it calls are private |
| 22 | A way for another task to send a change | Missing | `ci.balance --submit` imports the flows; the bench calls the verb by name |
| 23 | The workflow's rules in a guidance fragment of its own | Exists | The base's `rules.workshop.md:26-108` holds them today |
| 24 | Follow a run without the classifier | Missing | The watcher and the classifier share `_verdict.py` |

Count: 6 exist, 8 exist in part, 10 are missing.

## How the missing hooks meet the single plan

Mine, not ruled.

- **4, 5 and 19** are the core vocabulary record's CI model: triggers,
  access per task, credentials an extension declares. The plan builds them
  for its own reasons.
- **3, 6, 12 and 14** are needed by the merge queue wherever it lives: it
  adds labels and a protection rule, a queue state to `fm status`, a loop
  scenario and the `queue/` prefix.
- **2, 22 and 24** are the hooks the workflow itself needs: an extension
  that defines `submit`, a public way to send a change, and the watcher
  apart from the classifier.
- **20** belongs with issues becoming optional, ruled 2026-10-22.

So nothing found says the design is impossible. The move is larger than
the 19 before it, because the base reads the workflow in 34 places.

## An observation outside the question

The loop, devenv and the scale bench are 10.4% of the core. `fm ci.e2e` is
registered only where the workshop's own tests directory exists
(`W/_e2e.py:3704`), so no project built on the workshop gets it. Willem
ruled on 2026-08-31 that the local forge verbs belong to this repository
alone, and they moved to `livery-forge`. The same reasoning applies here.
Moving the loop takes more out of the core than the issue verbs and the
release train together.

## Rulings the move would reopen

- `notes/20260930-empty-shell-plan.md:211`, "What the base keeps": "the
  issue and submit family, the reserved-branch workflow engine, the release
  train's phases".
- `notes/20261002-extensions-plan.md:75`: the base "owns ... the issue and
  submit family, the workflow engine and the release train's phases".
- `notes/20261017-shard-the-check-legs.md:518`: "CI's weight in the core is
  the design, not a debt: no extraction into extensions." This bears on the
  workflow render leaving, not on the workflow.
- `notes/20261006-workshop-destination.md:2048`: an extension's job "takes
  `job` alone". This bears on hook 5.
