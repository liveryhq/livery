# Where each statement in the ethos page comes from

Status: written 2026-11-22 with the first draft of `docs/ethos.md`. It is
not a plan and rules nothing. It exists so Willem can check each statement
of the draft against its source, and it is deleted or kept as he decides
once the page is accepted.

## The prompt (Willem)

> To know the latter you need to extract the ethos from whereever you can
> in this repo, and document it in an ethos.md document, to be included in
> the docs after thorough review (by me as well). Willem, 2026-11-22

## How it was gathered

Read on 2026-11-22 at main `eea997acb`:

- every note in `notes/` (the three largest and the 38 smaller ones by
  three reading agents, the merge queue and affected plans by me);
- the published pages under `packages/*/docs/` and
  `packages/extensions/*/docs/`, `README.md`, `docs/index.md`;
- the guidance fragments the workshop ships
  (`packages/workshop/src/livery/workshop/content/fragments/`) and
  `CLAUDE.project.md`;
- the three plan skills;
- the tests and lints that hold a convention;
- the agent's memory of Willem's rulings in chat, which records rulings
  that no note carries. A statement resting on memory alone is marked
  "chat" below, and is the weakest kind of source here.

Line numbers were taken by the reading agents. One agent checked 53 of its
cited lines against the files. Any quote printed elsewhere gets its line
re-read first.

## Basis, by kind

- **W**: Willem's own words, quoted in a note.
- **R**: a ruling of Willem's as a note's decision record states it.
- **C**: a ground-truth contract that recurs across plans.
- **D**: stated in a published page or a shipped fragment.
- **M**: held by a test, lint or hook.
- **chat**: a ruling in chat, recorded only in the agent's memory.
- **I**: my inference from several of the above. These need Willem's eye
  most.

## The statements

Paths are under `notes/` unless they start with `packages/` or `tests/`.

### What livery is for

| Statement | Basis | Source |
| --- | --- | --- |
| The workshop is what a project uses; livery is the repository | R | `20260903-workshop-plan.md:818`, `:98` |
| A complete environment in minutes | R | `20260901-workshop-plan.md:68`; W `20260930-empty-shell-plan.md:64` |
| Improvements arrive through package updates; the project keeps its own files | R, D | `20260903-workshop-plan.md:44`; `packages/workshop/docs/index.md:27` |
| Built for many projects over many years, judged at that size | W, chat | `20260905-extensible-gate-plan.md:87`; chat 2026-09-26 ("used for years and years by me, for hundreds of projects"); chat 2026-10-04 ("Stop thinking small") |
| Each part is useful alone | W, C | `20260930-empty-shell-plan.md:32`, `:168`; `20260925-strongroom-redesign-plan.md:106`; `20261006-toolenv-plan.md:116` |
| People and agents equally | C | `20260901-workshop-plan.md:92`; `20260831-workshop-plan.md:630` |
| An agent that coordinates many streams is a user in its own right, and keeps no build knowledge in its context | chat | chat 2026-11-22 ("you as orchestrator of multiple sessions have access to coordinate many different streams of development quickly and efficiently, without having to have a large part of your context wasted on in-depth build knowledge") |
| Something a person and an agent enjoy using | chat | chat 2026-11-22 ("it should be something you would enjoy using, as far as that is possible") |

### A verdict that can be trusted

| Statement | Basis | Source |
| --- | --- | --- |
| One command, exit code is the verdict, CI runs the same | C, D | `20261002-extensions-plan.md:180`, `:186`; `packages/footman/docs/design.md:196` |
| Nothing narrowed or skipped without a printed line | C | `20261002-extensions-plan.md:211`; `20260904-kind-hierarchy-plan.md:67` |
| Failure reasons verbatim; listings complete or raise | C, D | `20260830-forge-bootstrap.md:138`, `:410`; `packages/forge/docs/protocol.md:22`, `:34` |
| A claim is proved in the same work; no postponed proof | C, chat | `20260927-releases-as-a-target.md:82`; chat 2026-09-26 ("Why would we defer it?") |
| A proved result is not proved again; speed from combining proofs | W, R | `20260912-daily-loop-plan.md:408`, `:381`; `20260907-ci-quality-plan.md:1824` |
| Refusals and fallbacks first; a guard that cannot fire | W, D | `20260831-workshop-plan.md:525`; `20260912-daily-loop-plan.md:628`; rules fragment `:28` |
| Proved by running it; acceptance by command; the loop | R, D | `20260904-release-train-postmortem.md:92`; `20260906-phase-4-post-mortem.md:13`; create-plan skill `:27` |
| Rules are held by machines; allowances only shrink | M, R | `packages/workshop/src/livery/workshop/_packages.py:494`; `packages/extensions/python/src/livery/extensions/python/_reaches.py:66`; `tests/test_namespaces.py:628`; `20261006-workshop-destination.md:1019` |
| "A rule that only a person holds is a candidate for a check" (intro) | I | pattern across `20260912-daily-loop-plan.md:683` and the postmortem rules; no note states it as a rule |

### Work that does not wait

| Statement | Basis | Source |
| --- | --- | --- |
| Nothing on the merge path waits on a person; a token-reached service is allowed | R | `20260904-kind-hierarchy-plan.md:594` (the amended wording); rules fragment `:19` |
| Verbs safe to run twice; probe before acting | C, D | `20260830-forge-bootstrap.md:143`; `packages/forge/docs/protocol.md:26` |
| A refusal teaches; a deliberate stop exits 0 | C, W | `20260901-workshop-plan.md:85`; `20260831-workshop-plan.md:637`; `packages/footman/docs/design.md:91` |
| CI decides what runs, never the author | R | `20261017-shard-the-check-legs.md:561` |

### One of everything

| Statement | Basis | Source |
| --- | --- | --- |
| One way; a second implementation is a defect | W, C | `20260910-state-store-plan.md:3`; `20260901-workshop-plan.md:125`; `20261006-workshop-destination.md:69` |
| Same change in two places becomes one function | chat | chat 2026-09-27 ("a seam is usually the right answer") |
| One channel between people and sessions, across accounts and computers | chat | chat 2026-11-22 ("We need to formalise communication between sessions and humans on different accounts and on different computers ... we would not use inter agent communication on the same machine unless we formalise that too"). The tools do not provide this yet: the statement is an aim |
| Ranking: less code, simpler code, one way, one concern per module | R | `20261006-workshop-destination.md:69` |
| Everything through `fm`; a missing verb gets an issue | W, C | `20260831-workshop-plan.md:544`; rules fragment `:33` |
| CI files decide nothing | W, C | `20260907-ci-quality-plan.md:232`; `20260914-ci-declared-and-dispatched.md:17` |
| Final form now; no clean-up pass | R | `20260830-forge-bootstrap.md:381`; `20261011-git-library.md:499` |
| No compatibility, migration or fallback before outside users exist | R | `20260925-strongroom-redesign.md:556`; `20260901-workshop-plan.md:1573`; `20261006-workshop-destination.md:3518` |
| The default is the right value, not the convenient one | R | `20260925-strongroom-redesign.md:563` |
| Taste is an option, correctness is not | R, D | `20260901-workshop-plan.md:1237`, `:229`; `packages/footman/docs/design.md:487` |

### A small core

| Statement | Basis | Source |
| --- | --- | --- |
| The core knows no language, ecosystem or product | W, R, M | `20261002-extensions-plan.md:30`; `20261006-workshop-destination.md:45`; `_packages.py:1109` |
| Dependencies point downward; no reach into privates | C, R, M | `20261002-extensions-plan.md:2134`; `_reaches.py:66` |
| An extension requires only what it cannot work without | R | `20261006-workshop-destination.md:64` |
| What is not listed costs nothing; nothing activates by being installed | W, C | `20260930-empty-shell-plan.md:36`, `:154`; `20260905-extensible-gate-plan.md:1541` |
| As few third-party dependencies as possible; standard library only | W, D, M | `20261002-extensions-plan.md:39`; `packages/footman/docs/design.md:50`; `packages/footman/tests/test_invariants.py:43` |
| Declarations are data; facts in the contract, opinions in code | C, R | `20261006-workshop-destination.md:127`, `:1914`; `20261002-extensions-plan.md:213` |
| Each tool's own files are the truth; no unstable tool text | R, chat | `20260905-extensible-gate-plan.md:1775`; `20261006-workshop-destination.md:3440`; chat 2026-10-08 |
| livery's opinions are one layer; no special case | W, R | `20260930-empty-shell-plan.md:69`; `20260905-extensible-gate-plan.md:1749`; `20261006-workshop-destination.md:3506` |

### Every forge

| Statement | Basis | Source |
| --- | --- | --- |
| No tie to one forge; private and self-hosted first class | W | `20260831-workshop-plan.md:697`; `musings.md:72` |
| The same typing and the same promise on every forge | W | `20261009-merge-queue-plan.md:16` ("that is the level of uniformity I aspire to"); `20260926-ci-profile.md:30` |
| The union of features; probed per server; declined by name | R, D | `20260901-workshop-plan.md:1500`; `packages/forge/docs/protocol.md:30` |

### Built for scale and for years

| Statement | Basis | Source |
| --- | --- | --- |
| Not postponed for lack of users; only for sequencing or a named risk | R | `20260904-kind-hierarchy-plan.md:456`, `:733`; `20260905-extensible-gate-plan.md:1677` |
| No grammar for a case nobody can describe | R | `20261006-workshop-destination.md:271`, `:424`; `20260830-development-workflows.md:37` |
| Speed judged for the largest workspace; cost follows what changed | chat, C | chat 2026-10-04; `20261002-extensions-plan.md:263`, `:1742` |
| Measure, then decide; corrections recorded | R | `20260907-ci-quality-plan.md:241`; `20260925-strongroom-redesign-plan.md:109`; `20261006-workshop-destination.md:2786` |
| Shared resources spent with care; expensive settings opt-in | W, chat | `20261017-shard-the-check-legs.md:22`; chat 2026-10-17 ("I don't want to be part of the cause they have to make drastic changes in their free plan") |
| Tools pinned, verified, never hand-installed; records are data | W, C, D | `20261006-toolenv-plan.md:15`, `:105`; `20260925-direct-downloads.md:102`; `packages/workshop/docs/index.md:1265` |
| Pull request code never beside a credential; admin token for the settings job alone | R, chat | `20260914-ci-declared-and-dispatched.md:146`; `20261006-workshop-destination.md:2160`; chat 2026-10-11 |

### How the work is done

| Statement | Basis | Source |
| --- | --- | --- |
| Small phases, gate-green, toward a designed destination | C, R | `20260830-forge-bootstrap.md:149`; `20261002-extensions-plan.md:2180` |
| Hard-to-reverse choices are ruled first; cheap ones are taken and reported | C, chat | `20260901-workshop-plan.md:132`; `20260930-local-loop-plan.md:600`; chat 2026-10-07 |
| Say what is not covered; unmet acceptance stays open; temporaries are listed | D, R | voice fragment `:75`; `20260904-release-train-postmortem.md:216`; `20261006-workshop-destination.md:3184` |
| A rule may be broken for a while if recorded and its reversal checked | R | `20261006-workshop-destination.md:2635`, `:3184` |
| A shared fault gets an issue the same day; a small finding is fixed in place | R, I | `20260904-release-train-postmortem.md:225`; `20260907-ci-quality-plan.md:1561`; `20260912-daily-loop-plan.md:405`. The one sentence that joins the two is mine |
| Issues a piece of work raised are solved before the next piece | R, chat | `20260914-the-tool-record-and-its-index.md:944`; chat 2026-09-26 |
| Documentation states what is true now, in plain language | D | standards fragment `:31`, `:52`; voice fragment `:34` |
| livery uses what it builds | C | `20260831-workshop-plan.md:52`; `20260830-forge-bootstrap.md:161` |

### When two values disagree

| Statement | Basis | Source |
| --- | --- | --- |
| Speed against proof | R | `20260907-ci-quality-plan.md:1824`, `:1719`, `:1987` |
| One behaviour against a forge's own feature | W, R | `20261009-merge-queue-plan.md:93`; `20260927-releases-as-a-target.md:20` |
| CI cost against coverage | W, R | `20260831-workshop-plan.md:342`; `20260907-ci-quality-plan.md:405`; `20261017-shard-the-check-legs.md:156` |
| Building ahead against waiting for a need | I | the two sets of rulings above; the dividing line is my reading |
| A strict refusal against the ability to repair | R | `20261002-extensions-plan.md:1645`; `20261006-workshop-destination.md:2025`, `:3736` |
| A general rule against a simpler change | R | `20261006-workshop-destination.md:3184` |

## Left out of the page, and why

- **hse as the reference.** It is a working method for the port, and later
  rulings drop it as a constraint (`20260914-the-tool-record-and-its-index.md:649`).
- **"I don't want anyone to use it, but I don't want to develop in a private
  repo either."** Only in `musings.md:87`, which says it is never a plan.
- **Names are identifiers, nothing depends on the metaphor**
  (`musings.md:61`). A naming rule, already in the documentation standards.
- **Release rules** (tags as receipts, no development wheels on the index).
  They are rules of one mechanism. The page names them only as examples of
  fixed choices.
- **Who may merge, release or file issues without asking.** Those are
  standing permissions between Willem and an agent, not values.

## For Willem to rule

1. **Where the page lives.** `docs/ethos.md` becomes a top-level page of the
   site, after Home, with the label "ethos" taken from the file name. The
   other home is a repository fragment (`fragments/identity.ethos.md`),
   which the site renders under Development and which `CLAUDE.md` then
   loads into every agent session here. The first is built in this branch.
   The second does not cover a top-level page of its own, and adds the
   whole page to every session's context.
2. **Whether the page is livery's or the workshop's.** As written it is
   livery's own. Shipping it in the workshop's fragments would put it in
   every project built on the workshop, which the "one layer" value argues
   against.
3. **The five statements marked I.** Each is my wording of something no
   note says in one sentence.
4. **"Hard-to-reverse choices are ruled first."** It describes how Willem
   and agents work. Say whether a public page should carry it.
5. **Statements whose only source is chat.** Four rest on the agent's
   memory of a ruling and on no note: the size livery is judged at, the
   seam rule, unstable tool text, and the free-plan cost. Three more were
   said on 2026-11-22 and are quoted in the table: an agent that
   coordinates many streams, tools a person and an agent enjoy using, and
   one channel between people and sessions. If they stand, they are worth
   a line in a note's decision record.

## Stale text found while reading

Not fixed here; each is outside this change.

- `README.md:8` lists two packages. The repository has 8 packages and 19
  extensions.
- `docs/index.md:3` says the docs are "deliberately unrendered", and
  `docs/index.md:11` says "Nothing on the merge path waits on anything
  outside the repository", which the 2026-09-26 amendment replaced.
- `.claude/skills/create-plan/SKILL.md:30` points at `.claude/guidance/`,
  which does not exist. The guidance is in `.workshop/fragments/`.
- footman's messages and pages use em-dashes and metaphors that the voice
  and console rules forbid (for example
  `packages/footman/src/livery/footman/_context.py:4353`).
- `packages/toolroom-bench/README.md` and `packages/toolroom-store/README.md`
  are template placeholders.
