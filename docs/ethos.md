# Ethos

The values that livery's design decisions return to. Use this page to judge a
proposal: a design that breaks one of these needs a stated reason, and the
reason is recorded with the decision.

Where a machine holds a rule, the rule names the check. A rule that only a
person holds is weaker, and is a candidate for a check.

## What livery is for

livery is the repository where the workshop and the packages beside it are
developed. The workshop is what a project uses. A project that depends on the
workshop gets a complete development environment in minutes: one gate, one
set of workflow verbs, pinned tools, CI, releases and a documentation site.
The project then receives every improvement through a package update, and
keeps its own name and its own files.

livery is built for many projects over many years. It is judged as the base
of hundreds of repositories, and never by its own size or by the projects
that exist today.

Each part is useful alone:

- footman runs tasks. It imports no workshop and reads no `workshop.toml`.
- toolroom gives a typed handle to a command-line tool and keeps pinned tools
  in one store per machine.
- forge is one interface to GitHub, Gitea and GitLab.
- strongroom stores bytes by their digest. It knows no tool and no package.
- The workshop composes them into a development workflow, and extensions add
  each language and each tool.

Every verb serves a person and a coding agent equally. Neither needs
knowledge the command does not print.

An agent that coordinates many streams of work at once is a user in its own
right. It hands a finished change over with one verb. It reads one line per
stream, and that line says what to do next. It keeps no knowledge of how
anything is built in its own context, because the tools hold that knowledge.

The tools should be something a person and an agent enjoy using.

## A verdict that can be trusted

**One command is the gate, and its exit code is the verdict.** `fm check`
runs every check a workspace lists. CI runs the same command. Two checkouts
of one commit with the same listed extensions get the same verdict. Nothing
installed on the machine, and nothing in its environment, changes the
verdict.

**Nothing is narrowed, skipped or left out without a printed line.** A gate
that runs less prints what it left out and why. A check that does not apply
to a package says that it skipped. It never passes without running. A
failure prints the failing tool's own words. No failure is reduced to true or
false, and a listing is complete or it raises.

**A claim is proved by a machine, in the same work that makes the claim.** A
declared floor is built against. A declared input is checked against what the
code read. A check that proves a claim is never postponed. It can be made
cheaper, and it cannot be replaced by a weaker one.

**A proved result is not proved again.** The gate records the tree it proved.
A later run gates only what changed since the nearest proved tree. Speed
comes from combining proofs. It never comes from dropping one.

**Refusals and fallbacks are tested before the normal path.** A broken normal
path shows itself in daily use. A broken fallback stays hidden until the day
it is the only path left. A fallback has a test that forces it. A guard that
cannot fire is worse than no guard, because it reads as protection.

**A feature is proved by running it.** Every acceptance line names the
command that proves it, and that command is run against real data before the
work is called done. The loop, `fm ci.e2e`, creates a new project on a local
forge and runs its workflows there, so a release is rehearsed before a
release depends on it.

**Rules are held by machines.** The layering checks refuse an import that
points upward and a reach into another package's private names. Tests pin
each package's public names. Lints judge the docstrings and the links the
site publishes. A drift check refuses a generated file that was edited by
hand. An exception to a rule is a listed allowance with a reason, and the
check refuses an allowance that nothing uses any more, so the list only
shrinks.

## Work that does not wait

**Nothing on the merge path waits on a person.** A step that cannot act
prints why and exits. A service reached with a configured token is allowed.
A question to a person is not.

**Every workflow verb is safe to run twice.** Running a verb again is the
recovery procedure. A verb reads the state before it acts, so a second run
changes nothing. A verb that cannot promise this is not shipped.

**A refusal teaches.** It names the thing that stopped it, states what was
expected, and gives the next action. Where there are several ways forward it
lists each and says when each applies. A deliberate stop is not an error: a
pull request that waits for a review exits 0, and its text does not read as a
failure.

**CI decides what to run.** Running the right checks never depends on the
author knowing what the change touches, or on a label the author must
remember. A narrower run is acceptable only when CI decides it from data, and
runs everything when the data cannot decide.

## One of everything

**One way to do each thing.** One concept has one spelling and one
implementation. A second implementation of an act is a defect, because the
two drift apart. When the same change is needed in two places, the shared
part becomes one function that both call. Options are ranked in this order:
less code, simpler code, one way to do each thing, one concern per module.

**One channel between people and sessions.** People and agent sessions work
on different accounts and different computers. They coordinate through the
forge and the repository, with the same verbs for everyone. Two sessions on
one computer use that same channel. A faster local route may carry the same
messages, and never different ones.

**Everything goes through `fm`.** A job with a verb goes through the verb,
because the verb carries the guards a raw call skips. A job no verb covers
marks a missing verb, and gets an issue. CI files only call `fm`. They carry
events, matrices and secrets, and they decide nothing, because a decision
written in a CI file is the one part that cannot be run on a desk.

**The final form, now.** Types, public names and docstrings are written in
their final form the first time. There is no clean-up pass later. Before a
package has users outside this repository it carries no compatibility code,
no migration code and no fallback for an older version. A removed key meets
the ordinary refusal of an unknown key. A default lasts for the life of the
project, so the default is the right value and not the convenient one.

**Taste is an option. Correctness is not.** A choice of taste is a setting
with a default. A choice that other mechanisms depend on is fixed and
documented as fixed: main is linear with one squash commit per pull request,
a release tag is cut only after the index confirms the package, and verbs
are idempotent. A setting whose right value can be known is not offered.

## A small core

**The core knows no language, no ecosystem and no product.** It composes
extensions and offers them a public interface. Python, C++, coverage, the
documentation site and each tool are extensions. A layering check refuses an
import from the core into an extension.

**Dependencies point downward.** A lower package knows nothing about the
packages that use it. No package imports another distribution's private
names: a name that is needed elsewhere is made public where it lives. An
extension requires only what it cannot work without, and cooperates with
another extension only when a workspace lists both.

**What a project does not list costs it nothing.** Listing is the only way
to turn an extension on. Nothing activates by being installed. An extension
a workspace does not list is not imported, installs no tool and adds no time.

**As few third-party dependencies as possible.** footman, forge and
strongroom import only the standard library. A package that every project
depends on must not change what those projects depend on.

**Declarations are data.** An extension says what it provides in a file that
is read and judged without running the extension. Facts about a workspace
live in `workshop.toml`. Opinions live in an extension's code.

**Each tool's own files are the truth.** The workshop uses a tool's own
configuration format, its own composition rules and its own lock. It keeps
no second copy of a fact that a native file already states. It reads a
tool's exit code or its structured output, and never text the tool has not
marked stable.

**livery's own opinions are one layer.** The workshop's rules and livery's
house style are separate. Four type checkers and this page's voice rules are
livery's choices. A project that wants other choices lists other extensions,
and never forks a template. The livery repository is a workspace like any
other and gets no special treatment.

## Every forge

**No project is tied to one forge.** GitHub, Gitea and GitLab are served
through one interface, and self-hosted and private forges are served as
fully as public ones.

**What a person types, and what is promised, is the same on every forge.** A
forge's own feature can do the work where the forge has one. It changes who
does the work and never what is promised.

**A difference is a named capability.** The interface targets everything the
forges can do between them, paid features included. Whether one server
grants a feature is asked of that server. A feature a server lacks is
declined by name, and is never imitated.

## Built for scale and for years

**A capability is not postponed because no project uses it yet.** For a base
that projects will be built on, having no users is the state before it
exists. Work is postponed for one of two reasons: something it depends on is
not built, or a named risk. The opposite error is avoided too: no grammar,
key or verb is added for a case that nobody can describe.

**Speed is judged for the largest workspace.** A cost is measured per unit
and then scaled to thousands of packages in several languages, many
worktrees and three operating systems. A command costs what changed, and
never the number of packages.

**Measure, then decide.** A decision about speed, size or a default rests on
a number, its sample and its date. An estimate that a measurement corrects is
corrected in the record, with the cause.

**Shared resources are spent with care.** The defaults a project inherits
must not use up a free CI plan without the project seeing it. An expensive
setting is opt-in. A tool is downloaded once per version per machine.

**The supply chain is narrow and checked.** Tools are pinned by version and
digest, and are installed by `fm sync`, never by hand. A tool record is
data: it holds no script and no hook. Code from a pull request never runs
beside a credential. The admin token reaches the one job that applies the
repository's settings, and that job runs no code from the change.

## How the work is done

**Small steps toward a designed destination.** A plan states the invariants
no phase may break. Each phase lands alone, with the gate green, in a day or
two. A phase that would break an invariant stops for a person.

**A choice that is hard to reverse is decided before it is built.** The
project's owner rules on it. A choice that is cheap to reverse is taken,
recorded as taken, and reported, so it can be overruled.

**Say what is not covered.** An option is offered together with what it does
not cover. An acceptance line that was not met stays open in the plan until
its evidence exists. A temporary state is listed with what replaces it. A
rule may be broken for a while when that makes a change simpler, if the
break is recorded and its reversal is checked.

**Problems are fixed while they are fresh.** A fault that other places share
gets an issue the same day, with a body an outsider can read. A small
finding inside the current change is fixed in that change. The issues a
piece of work raised are solved before the next piece starts.

**Documentation states what is true now.** It is written in plain language,
with a number where a number exists and no adjective in its place. It never
tells the history of the code. Every docstring is published, so every
docstring is written for a reader. See the [development
patterns](packages/workshop/development-patterns.md) for the daily workflow,
and footman's [design page](packages/footman/design.md) and the forge's
[protocol rules](packages/forge/protocol.md) for these values applied to one
package.

**livery uses what it builds.** This repository is the workshop's first
workspace. Its releases, its CI and its documentation site run on the
workshop's own verbs.

## When two values disagree

- **Speed against proof.** Combine proofs, or move a slow proof to a later
  point and state the delay. Never drop a proof without a printed line.
- **One behaviour on every forge against a forge's own feature.** Keep the
  promise the same, and let the forge's feature do the work where a probe
  finds it.
- **Low CI cost against coverage.** Keep the floor and move the cost. A
  lower floor is a temporary state with a name, a reason and a date it ends.
- **Building ahead against waiting for a need.** Build the capability a
  project built on livery will need. Wait with a further construct until its
  first case exists.
- **A strict refusal against the ability to repair.** A refusal must never
  stop the command that repairs the fault.
- **A general rule against a simpler change.** The rule describes the
  finished product. A step on the way may break it, when the break is
  recorded and undone at the first chance.
