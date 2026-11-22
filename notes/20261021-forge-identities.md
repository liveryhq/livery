# Forge identities: who acts, with which credential, on every forge

Status: written 2026-10-21. Nothing is built. Willem ruled the shape,
`FORGE_TOKEN` as the bot's name on every forge, and GitLab Free's
receipt tags by deploy key. On 2026-10-23 phases 1 to 3 were set to be
replaced by the CI model in `notes/20261023-core-vocabulary.md`, once
the rulings it reopens are decided; phases 4 to 10 are restated on top
of it then. Nothing here starts before that.

## The prompt (Willem)

> Is this a github only issue? Ideally I want a solution that works
> everywhere the same way, and happens to solve this issue on github.
> Willem, 2026-10-19

> As we're creating generic developer tools, I think most of these
> setups should be possible. Our job is to provide the setup
> instructions and to make it look uniform to the code base and across
> the forges. Willem, 2026-10-20

> I'd like people to be able to create new repositories, and delete the
> ones they've created but not others. I had this working in hse under
> self hosted gitea. Willem, 2026-10-20

> I am all for good security and separation of concerns, [...] but are
> we really saying that all this is for security? Willem, 2026-10-21

The intent: the code names who acts (a role), never which credential
supplies it; each forge supplies the roles in its own way, set up by
our verbs from instructions we ship; a job holds only the roles it
declares; and the one step that holds the bot's credential while
proposing a change runs no code the workspace installs.

## What exists

- **The failure.** Nightly run 39200159228 (2026-10-19, 04:40 UTC):
  `fm ci.balance --submit` committed a third ubuntu shard on
  `chore/balance-20261019` and the push was refused: "refusing to
  allow a GitHub App to create or update workflow
  `.github/workflows/ci.yml` without `workflows` permission". It then
  printed "run `fm submit` again once the forge answers", which is
  wrong for a refusal. The rest of the nightly was green.
- **The same fault waits in the refresh.** `toolroom.refresh --submit`
  runs in the same job every two weeks. The rendered workflows pin uv
  from the lock (`livery.workshop._entry.locked_uv_version`), so a
  refresh that moves uv is refused the same way; one that does not
  opens a pull request with the job token, which starts no run. The
  day's-branch logic exists twice: `livery.workshop._balance.submit`
  and `livery.toolroom.bench._tasks.submit_refresh`.
- **Token names.** `livery.workshop._tokens`: `FORGE_TOKEN` and
  `FORGE_ADMIN_TOKEN`, each read host-qualified first
  (`FORGE_TOKEN__GITHUB_COM`) then bare. `admin_token` falls back to
  the everyday token everywhere, CI included; in CI it is inert only
  because the govern job is given no `FORGE_TOKEN`.
- **Grants are set in code.** `livery.workshop._points.Job` carries
  `token` (`job`, `repository`, `secret`, `admin`, none), `writes`,
  `pushes`, `dispatches`, `environment` and `publishes_index`. GitHub's
  `_grants` turns `environment`, `writes` and `dispatches` into
  `id-token`, `contents` and `actions` write; `_call_env` turns `token`
  into the call's `FORGE_TOKEN` or `FORGE_ADMIN_TOKEN`, with `||
  secrets.GITHUB_TOKEN` for `repository`; `_checkout_step` puts
  `secrets.FORGE_TOKEN || github.token` (GitHub) or
  `secrets.FORGE_TOKEN` (Gitea) on a pushing job's checkout. On GitLab
  a writing or pushing job runs `git remote set-url` with
  `GITLAB_PUSH_TOKEN`, a project access token, because the job token
  cannot push; the loop mints it (`livery.workshop._e2e._mint_push_token`).
  The nightly job is `token="repository", writes=True` without
  `pushes`, so its push rode the checkout's job token.
- **The index token has a forge branch.** `_call_env` reads
  `PYPI_TOKEN` on GitHub and `UV_PUBLISH_TOKEN` elsewhere.
- **Extension jobs.** `[ci.jobs.<point>.<name>] token` accepts `"job"`
  alone. #1280 records four ways a contributed job could get more and
  recommends (d): privilege stays with the base's jobs.
- **Receipt tags** (`_protect_tags` in each backend): GitHub's ruleset
  blocks deletion, update and non-fast-forward, and leaves creation
  open to anyone with write; Gitea whitelists `whoami()` of the
  identity that governs, which is the admin; GitLab sets
  `create_access_level` 40, Maintainers.
- **Every privileged job enters the workspace.** The nightly, publish
  and govern jobs run `setup.sh` and then `fm ci.run`, which loads
  `tasks.py` and mounts every extension, so their code runs beside the
  job's tokens.
- **`livery.forge` imports the standard library alone** at import
  time (`CLAUDE.project.md`); its one optional extra, PyNaCl for
  GitHub secrets, loads lazily.
- **Governance** asserts `livery.forge.RepoConfig`, including
  `secrets`, `variables` and `protected_tag_patterns`.
- **Desk.** `fm env.set ... --scope=shared` writes the shared env every
  checkout reads; on GitHub the lookup falls back to `gh auth token`;
  doctor reports the admin ladder (`livery.workshop._doctor`).
- **e2e accounts.** `LIVERY_E2E_GITHUB_TOKEN` and
  `LIVERY_E2E_GITHUB_OWNER` are stored on this repository and nothing
  reads them (`notes/20261017-github-lane.md`).
- **The local GitLab** runs `gitlab/gitlab-ce:latest`, unpinned
  (`livery.forge._dev`, `compose.yaml`).
- **Related issues.** #1546: the refresh reads GitHub's API
  unauthenticated in CI. #1240: the nightly's trace push refused with
  403; run 39200159228 pushed its trace (377 KiB), so it may no longer
  reproduce.

## Forge facts this plan relies on

Each fact a phase builds on is proved by a conformance scenario before
code depends on it. Facts from documentation alone are marked.

| Fact | Forge | Source | Proved in |
| --- | --- | --- | --- |
| The job token cannot push a commit that changes `.github/workflows/`, and nothing it creates starts a run, dispatch aside | GitHub | run 39200159228; the nightly's own note | phase 3 |
| The job token pushes to its own project when `ci_push_repository_for_job_token_allowed` is on; generally available in 18.4 (2025-09-18; the current release is 19.5, 2026-10-15); its pushes start no pipeline; it has the rights of the user who started the job | GitLab | docs | phase 4, custom refs included (the docs do not say) |
| Protected tags name users or groups on Premium and Ultimate only; Free names roles or a deploy key | GitLab | docs, deploy key tier not stated | phase 5, on the local CE |
| A ruleset's bypass list takes roles, teams, Apps and Dependabot, not users | GitHub | docs | phase 5 |
| An App's private key belongs to the App, and the manifest flow returns it at creation; an installation token lasts one hour and a mint can narrow its permissions and repositories; an App's API commits are signed | GitHub | docs | phase 8 |
| A commit can be made from file contents through the API: GraphQL `createCommitOnBranch`, the commits API's `actions`, the multi-file contents API | GitHub, GitLab, Gitea | docs | phase 3 |
| A member creates repositories when the organisation allows it; the creator gets admin; a member with admin deletes when "allow members to delete or transfer repositories" is on | GitHub | docs, creator admin from secondary sources | phase 9 |
| A team with "Create repositories" creates; the creator gets admin and may delete | Gitea | proved in hse (`hse-devkit` `_gitea_access`) | phase 9 |
| The group's minimum role to create projects admits Developers; deleting needs Owner; the creator keeps their group role | GitLab | docs, creator role unconfirmed | phase 9 |

## Ground-truth contracts (do not violate)

1. **The code names roles, never sources.** A role reads one name on
   every forge: the bot `FORGE_TOKEN`, admin `FORGE_ADMIN_TOKEN`,
   org `FORGE_ORG_TOKEN`; run is the job's own token and is never
   stored. Where a credential comes from is the backend's and the
   render's business.
2. **At the desk a role falls back to the everyday token; in CI no role
   falls back.** A job missing a role it declares refuses before its
   first entry, naming the role, the secret and the command that sets
   it up.
3. **A job holds the roles it declares and no others, and only the
   base declares them.** No extension, no contract key and no schedule
   entry grants a role (#1280, approach (d)).
4. **The step that proposes a change runs `livery.forge` and Python
   alone.** No `tasks.py`, no extension, no `fm`, no package the
   workspace installs.
5. **No secret is written in `workshop.toml`.** Secret names are
   configuration with defaults; values live in the forge's secret
   store or the desk's shared env.
6. **Receipt tags are created by the bot identity alone, and moved or
   deleted by nobody.** Protection names an identity, never a role
   level.
7. **Uniform or refused.** An operation means the same on every forge;
   where a forge or tier cannot do it, the capability refuses naming
   the forge, the tier and what is missing.
8. **Every forge touch goes through `livery.forge`, every job through
   `fm`.** The proposing step is the one job with no `fm`, by contract
   4; it calls `livery.forge` directly.

## The design

### Roles

| Role | Does | Holds it in CI |
| --- | --- | --- |
| `run` | the run's own writes: state store refs, rows, traces; reads of the forge's API | every job |
| `bot` | proposes changes: pushes branches, commits, opens and arms pull requests; cuts receipt tags; files issues; its acts start runs | the send job and the publish job |
| `admin` | governs the repository: settings, protection, secrets, the receipt-tag rule | the govern job |
| `org` | creates repositories in one organisation and deletes the ones it created | the fleet's birth and the scratch birth |

The roles sit on two axes. `run`, `bot` and `admin` are powers over
this repository, in that order. `org` is a different scope: it is not
above `admin` and does not fall back to it.

### Sources and names

A role's source is a stored token on every forge, or on GitHub an App.
The contract says which, and names the repository's secret; the
defaults need no table at all.

```toml
[forge]
kind = "github"
owner = "willemkokke"

[forge.bot]
source = "app"            # "token" when absent
app-id = 1234567          # public; the App's key is the secret
secret = "FORGE_APP_KEY"  # the default for an App; "FORGE_TOKEN" for a token

[forge.admin]
source = "app"
app-id = 1234568
secret = "FORGE_ADMIN_APP_KEY"
```

- The render hands a job the role's name mapped from the named secret:
  `FORGE_TOKEN: ${{ secrets.<secret> }}`, or the App's key, which `fm
  ci.run` mints into `FORGE_TOKEN` per entry, narrowed to the role's
  permissions and this repository.
- One App per role: an App's key can mint every permission the App
  has, so an App that holds admin would put admin in every bot job.
- A table that acts on another organisation names its owner and its
  secret, and the job reads `FORGE_ORG_TOKEN`:

  ```toml
  [fleet]
  owner = "livery-examples"
  secret = "FLEET_TOKEN"
  ```

- The index token follows the same pattern: the index's table names
  its secret, and the job reads `UV_PUBLISH_TOKEN` on every forge.
- GitLab Free's bot carries a deploy key beside its token, for receipt
  tags: `[forge.bot] deploy-key-secret`, `FORGE_DEPLOY_KEY` by default.

Every scenario, from a solo developer's personal account to a GitLab
group, is these keys with different values; `fm identity.setup` writes
them (phase 7).

### Fallbacks

At the desk one person holds one set of rights, which the forge
enforces whichever token is used, so every role falls back to the
everyday token: admin as today, org directly (a person's admin token
is often scoped to one repository). In CI a fallback hides a missing
credential until the forge refuses after the work is done, which is
the failure that started this plan; so in CI the lookup never falls
back, and `ci.run` checks each declared role before the first entry.

### The run role

`run` is the job token. GitHub and Gitea job tokens push already; on
GitLab, governance turns on `ci_push_repository_for_job_token_allowed`,
and the render's `git remote set-url` and `GITLAB_PUSH_TOKEN` go. A
GitLab older than 18.4 refuses at governance, naming its version and
the setting. On GitHub, `ci.run` hands its children the run's token as
`GH_TOKEN` when none is set, for reads of GitHub's API (#1546); on
another forge it never does, since a Gitea token must not reach
GitHub's API host. No checkout carries a credential: a push takes the
role's credential for that one command through `livery.git`, each
backend supplying the header's form.

### Proposals: build, then send

A task that proposes a change (`ci.balance --submit`, `toolroom.refresh
--submit`, the fleet's `workflow.update --armed`) builds it with `run`
in the job it runs in, gate included, and writes a proposal file: the
base commit, the branch, the title, the body, whether to arm, and the
changed files' contents. The send job, after it, holds `bot` and runs
`python -I -m livery.forge.propose <file>` with `livery-forge` alone:
from the workspace's own source when `livery-forge` is a member, as
here, else the version the lock pins, from the index. It makes the
commit through the forge's API, opens or reuses the pull request, and
arms it when the file says so. A branch that already holds the same
changes answers "already proposed"; one holding other changes refuses
naming it. One seam in the workshop writes the file for every task, so
the day's-branch logic exists once.

Measured 2026-10-19: a git bundle of the balance change is 842 bytes
and builds in 0.02 s, so the handover's weight is the second job's
entry (11 to 28 s, measured 2026-10-11), not the payload. The API
commit was chosen over the bundle because the send step then needs no
checkout and no git, and on GitHub the App's commit is signed.

### Receipt tags

Governance asks the bot credential who it is and names that identity
as the only creator of `packages/*/v*` and `packages/*/*/v*`:

- GitHub: a ruleset restricting creation, its bypass list the bot's
  App. With a token source the bypass can only be a role, admin, so a
  person with admin can still cut a receipt by hand; `identity.status`
  says so.
- Gitea: the whitelist names the bot, not `whoami()`.
- GitLab Premium and Ultimate: the bot user. GitLab Free: a deploy key
  (ruled 2026-10-21), registered by governance, its private half held
  as the bot's `deploy-key-secret` by the publish job, which pushes
  receipts over SSH with it.

An admin can change the protection on every forge; protection by
identity makes a hand-cut receipt a recorded change of a setting.

### People's repositories

A person creates repositories and deletes the ones they created, and
no others. The forge enforces it where it can: a GitHub organisation
(members create, creators get admin, members with admin delete), a
Gitea organisation (a team with "Create repositories", as hse). GitLab
gives a creator their group role and deleting needs Owner, so each
person owns a subgroup, `<group>/<their-name>`, created by a group
owner through `fm identity.setup person <name>`; moving a project into
the shared group is a transfer. `fm repo.create` and `fm repo.delete`
are the verbs on every forge. A person's default owner for
`fm new.project` is a desk setting, since it differs per person.

### Setup and status

Each backend declares, per role and source, as data: the permissions
or scopes, where the credential is created (with the permissions
prefilled where the forge allows), and how it is checked. Three
readers:

- `fm identity.setup <role> [--source=token|app] [--scope=repo|org]`,
  attended: prints the steps, takes a pasted token as a secret, checks
  it, and stores it through `Repository.configure(RepoConfig(secrets=...))`
  at repository or organisation scope. With `--source=app` it runs the
  manifest flow: a localhost page sends the manifest, GitHub returns a
  code, the exchange returns the App's ID and key, the key is stored
  and the ID written into `workshop.toml`, and the verb waits for the
  installation. Re-running is the recovery. `fm new.project` runs it
  when a person is there and prints it when nobody is.
- `fm identity.status`, also in doctor: per role, what supplies it at
  the desk and in CI, whether its permissions suffice, its expiry date
  and a warning before it, whether an App is installed here, and
  hse's capabilities row ("can start new projects"), worded as what
  the person can do, never as a flag.
- The site: one page per role, generated from the same data.

### What this does not cover

- **Publish and govern trust the workspace.** They keep entering it
  with their credentials; a compromised extension or dependency can
  read them there. Giving them the send job's shape is larger work
  (publish needs the built wheels and uv, govern the contract read)
  and waits on a named risk.
- **Merged code.** Whoever merges can change the render. The attacker
  this design addresses is a compromised dependency or extension.
- **Tokens with no stored secret** (OIDC exchanged for a forge token):
  on GitHub that needs a service we would host.
- **An App's server features**: webhooks, check-run buttons, a merge
  queue.
- **Unattended rotation.** Status warns; a person rotates. GitLab could
  rotate access tokens through its API; not designed.

## Phases

Each phase lands gate-green and mergeable alone. Exceptional paths are
tested before happy paths.

### Phase 1: roles on jobs, the presence check, and the run token

Deliverables: `Job.roles` replaces `token` and `pushes`; the render
hands a job the names of the roles it declares and no others, with no
`||` fallback; no rendered checkout carries a credential; `livery.git`
pushes with a credential for one command; `ci.run` checks each declared
role before the first entry; the admin lookup does not fall back in
CI; on GitHub `ci.run` hands children the run's token as `GH_TOKEN`
when none is set (#1546); a refused push names the refusal and the
role. The nightly declares `run` and `bot` until phase 3 (temporary),
so the balancer and the refresh work once `FORGE_TOKEN` is set (open
item 6). Acceptance, refusals first:

- `test_a_job_missing_a_declared_role_refuses_before_its_first_entry_naming_the_secret`.
- `test_the_admin_lookup_does_not_fall_back_to_the_everyday_token_in_ci`.
- `test_a_refused_push_names_the_refusal_and_never_asks_for_a_retry`.
- `test_no_rendered_checkout_carries_a_credential`, over the goldens of
  all three forges.
- `test_a_github_run_hands_its_token_to_children_as_gh_token_and_no_other_forge_does`.
- `fm sync` renders this repository's workflows with no `||
  github.token` and no checkout `token:`; `fm check` exits 0.
- `fm ci.e2e --scenario=points --fresh` is green.
- After `FORGE_TOKEN` is set: `fm ci.dispatch --point=nightly` opens
  the day's balance pull request, its run starts, and `fm ci.status
  --point=nightly` exits 0.

### Phase 2: secrets named by purpose

Deliverables: `[forge.bot]` and `[forge.admin]` with `secret`; the
index's `secret`, and `_call_env`'s `PYPI_TOKEN` branch goes;
`FORGE_ORG_TOKEN` with the desk fallback to `FORGE_TOKEN`; a table
acting on another organisation names `owner` and `secret`; the e2e
accounts move to that shape and `LIVERY_E2E_GITHUB_*` go. Acceptance:

- `test_a_secret_name_the_forge_refuses_is_refused_naming_the_rule`
  (GitHub refuses names starting `GITHUB_`).
- `test_the_org_role_at_the_desk_falls_back_to_the_everyday_token_not_the_admin_token`.
- `test_the_org_role_in_ci_never_falls_back`.
- `test_the_index_token_reads_one_name_on_every_forge`, over the goldens.
- `fm check` exits 0; `fm ci.e2e --scenario=release --fresh` is green.

### Phase 3: proposals, built then sent

Deliverables: `livery.forge`'s `propose` operation on the three
backends and the fake, with its conformance scenario (a change to a
workflow file included); the proposal file and its one writer in the
workshop, used by `ci.balance --submit` and `toolroom.refresh
--submit`; the send job at the nightly point, which runs `python -I -m
livery.forge.propose` with `livery-forge` alone; the nightly job drops
`bot`. Acceptance, refusals first:

- `test_a_branch_holding_other_changes_refuses_naming_the_branch`;
  `test_a_branch_holding_the_same_changes_answers_already_proposed`.
- `test_the_send_step_imports_nothing_but_the_standard_library_and_livery_forge`,
  read from `sys.modules` in a subprocess.
- `fm forge.conformance --live --scenario=propose` passes on gitea and
  gitlab; `fm forge.fixtures.record` records GitHub's from the e2e
  organisation, and replay passes in `fm check`.
- `fm ci.e2e --scenario=points --fresh` proves the send job on Gitea,
  and `fm ci.e2e --forge=gitlab --scenario=points --fresh` on GitLab.
- The next hosted nightly opens the balance pull request through the
  send job; `fm ci.status --point=nightly` exits 0.

### Phase 4: GitLab's run role

Deliverables: `RepoConfig` gains the job-token push setting, asserted by
governance on GitLab (the other forges' job tokens push already); the
GitLab render's `git remote set-url`, `GITLAB_PUSH_TOKEN` and
`_mint_push_token` go; a server older than 18.4 refuses at governance;
the local GitLab image is pinned, amd64 and arm64. Acceptance:

- `test_governance_on_a_gitlab_older_than_18_4_refuses_naming_the_version_and_the_setting`.
- `fm forge.conformance --live --backend=gitlab --scenario=job-token-push`:
  a job token started by a Developer pushes `refs/workshop/*` and is
  refused on a protected tag and on main.
- `fm ci.e2e --forge=gitlab --scenario=release --fresh` is green with no
  push token provisioned.

### Phase 5: receipt tags by identity

Deliverables: `RepoConfig` names the receipt creator as an identity;
GitHub's ruleset restricts creation with the bot's App as its bypass,
or the admin role with a token source; Gitea's whitelist names the
bot; GitLab names the bot user on Premium and a deploy key on Free,
registered by governance, the publish job pushing receipts over SSH
with it. Acceptance, refusals first:

- `fm forge.conformance --live --scenario=receipt-by-identity` on each
  backend: the admin's token, a job token and a person's token are
  refused a receipt; the bot creates one.
- On the local CE: the deploy key creates a protected tag
  (`--backend=gitlab`); if it cannot, the phase stops for Willem.
- `test_the_status_names_a_role_level_receipt_rule_as_admitting_people`.
- `fm ci.e2e --scenario=release --fresh` on Gitea and on GitLab cuts
  its receipts.

### Phase 6: identity status

Deliverables: the per-backend setup data (permissions, creation place,
checks); `fm identity.status` and its doctor rows; hse's capabilities
row ported. Acceptance, refusals first:

- `test_a_token_without_a_required_permission_is_named_with_the_permission`.
- `test_an_expiry_within_the_warning_window_is_named_with_its_date`.
- `test_a_forge_answering_403_for_the_user_is_not_read_as_a_missing_scope`
  (hse's distinction).
- `fm identity.status` on this repository exits 0 and names each role's
  source and the jobs that hold it.

### Phase 7: identity setup with stored tokens

Deliverables: `fm identity.setup` for the token source, at repository
and organisation scope; `fm new.project` calls it; the site's pages
from the same data. Acceptance:

- `test_setup_refuses_a_token_that_fails_its_check_and_stores_nothing`.
- `test_setup_without_a_person_prints_the_command_and_changes_nothing`.
- Live on the local Gitea and GitLab: `fm identity.setup bot` stores the
  secret and `fm identity.status` reads it.
- `fm docs.build` renders one page per role.

### Phase 8: GitHub Apps

Deliverables: minting in the token lookup (open item 4), narrowed per
role; the manifest flow in `identity.setup --source=app`; `[forge.bot]`
and `[forge.admin]` accept `source = "app"`. Acceptance:

- `test_a_missing_installation_refuses_naming_the_install_page`.
- `test_a_mint_asks_only_for_the_roles_permissions`.
- Live in the e2e organisation: the manifest flow creates an App,
  `fm identity.status` names it, and a send job commits through it.
- This repository moves from Willem's token to its Apps; the nightly
  is green.

### Phase 9: people's repositories

Deliverables: `fm repo.create` and `fm repo.delete`;
`fm identity.setup person <name>` for GitLab's subgroup per person;
the capabilities row says what a person may create and delete.
Acceptance, refusals first:

- `fm forge.conformance --live --scenario=own-repositories` on each
  backend: person A creates, person B is refused deleting A's, A
  deletes it.
- `test_delete_of_a_repository_another_person_created_refuses_naming_the_creator`.

### Phase 10: the organisation role in CI

Deliverables: the fleet's birth and the scratch birth hold `org` and
nothing else; the jobs read `FORGE_ORG_TOKEN` from the tables' named
secrets. Built with `notes/20261017-github-lane.md` phases 2 and 4.
Acceptance: that note's acceptances, plus
`test_a_job_holding_org_holds_no_other_stored_role`.

## Temporary, replaced by

| Temporary | Replaced by |
| --- | --- |
| the nightly job holds `bot` | the send job (phase 3) |
| Willem's fine-grained token as `FORGE_TOKEN` on this repository | its Apps (phase 8) |
| `gitlab/gitlab-ce:latest` on the desk | a pinned image (phase 4) |
| `GITLAB_PUSH_TOKEN` in GitLab's render and the loop | the job token's push setting (phase 4) |
| receipt tags created by anyone with write on GitHub, Maintainers on GitLab | protection by identity (phase 5) |

## Decision record

- 2026-10-19, Willem: yes to the shape; `FORGE_TOKEN` is the bot's
  credential on every forge.
- 2026-10-19, mine: the handover is an API commit, not a git bundle.
  The bundle measured 842 bytes; the cost of either is the second job.
- 2026-10-20, Willem: every setup should be possible, with our setup
  instructions and a uniform look in the code and across forges.
- 2026-10-20, Willem: "lane" is meaningless. "author" is rejected too
  (mine): it reads as a verb and git's commits already carry an
  `author` field. `bot` is the working name (open item 1).
- 2026-10-20, Willem: people create repositories and delete the ones
  they created, not others, as hse did on Gitea.
- 2026-10-21, mine: `FORGE_RUN_TOKEN` is not needed; GitLab's job token
  pushes from 18.4, a version GitLab stopped patching about ten months
  ago.
- 2026-10-21, mine: secrets are named by purpose in the table that
  needs them, not qualified by host and owner; `org` does not fall back
  to `admin`, and no role falls back in CI.
- 2026-10-21, Willem: GitLab Free's receipt tags by deploy key, option
  (a).
- 2026-10-21, Willem questioned whether the separation is for security.
  Recorded: it is for function first (the forges require a second
  identity), mistakes second, and malice only where the privileged step
  runs no workspace code; so the send step runs `livery.forge` alone,
  and publish and govern are recorded as trusting the workspace.
- 2026-10-21, mine: #1280 is decided as (d), with the condition that
  the base's privileged step runs no extension code; recorded in
  `notes/20261006-workshop-destination.md` when phase 3 lands.
- 2026-10-21, Willem: "write up a plan unifying all your proposals".

## Open

1. **The role's name: `bot` or `contributor`.** Owner: Willem. `bot`,
   recommended: on all three forges the identity is called a bot, and
   the desk never names a role. `contributor` is only a noun, but
   GitHub uses it for people.
2. **The tables' place: `[forge.bot]` or `[identity.bot]`.** Owner:
   Willem. `[forge.*]`, recommended: beside the forge it belongs to.
   `[identity.*]` matches the verbs.
3. **The App's ID in the contract.** Owner: Willem. Recommended: it is
   an identifier, not a secret, and the render then needs no variable.
4. **Where an App's token is minted.** Owner: Willem. (a) In the token
   lookup, `cryptography` as a lazy optional extra of `livery-forge`,
   recommended: one path for every consumer, minted again past the
   hour, tested. (b) GitHub's pinned action in the render: no
   dependency, minted once at job start, the rule in YAML. Either puts
   third-party code in the send step.
5. **GitLab's shape for people's repositories.** Owner: Willem. (a) A
   subgroup per person, recommended: the forge enforces it. Not
   covered: the repositories are not side by side in one group. (b) A
   governance job that records creators and refuses others: one flat
   group, but our rule, not the forge's, and each act waits on a run.
6. **The interim token.** Owner: Willem. A fine-grained token on
   `willemkokke/livery` with Contents, Pull requests, Workflows and
   Issues write, set as `FORGE_TOKEN`, so phase 1 fixes the nightly.
7. **Organisation secrets on GitHub's free plan.** Owner: me. Whether
   they reach private repositories; checked before phase 7.
8. **#1240.** Owner: me. Run 39200159228 pushed its trace; confirm on
   the next nightlies and close it with the evidence.
