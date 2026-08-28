# Chapter 5 — Parallel Operators

*Draft status: author draft; human verification pending. Outputs are real
transcripts; the parallel work in the listings is performed by genuinely
separate working trees on one shared store.*

## Two minds, one project

Volume two's fifth chapter asked what happens when two operators share one
estate file, and the engine answered with locks and queues. This chapter asks
the version-control edition — a supervisor dispatches two agents against one
project, or a timer's maintenance session wakes while an interactive session
works — and the answer git's design wants is subtly different from the one
operators reach for untaught. The untaught reflexes are both wrong in
instructive ways. Sharing a *working tree* — two sessions editing one
checkout — recreates the lost-update chaos of volume two's midden: staged
files interleave, one session's checkout yanks the branch out from under the
other, and `status` reports a fiction assembled from both minds. Cloning per
session — a fresh full copy in every scratch directory — is safe and pays
for safety twice over: the whole object store duplicated per operator, and
the operators' work stranded in separate repositories whose exchange now
requires a network hop or path-remote gymnastics. The instrument built for
exactly this shape sits between: `git worktree` gives each operator its own
working tree and its own checked-out branch, all backed by *one shared
object store* — chapter 1's content-addressed ledger, which never needed
duplicating because it is append-only and hash-addressed, the two properties
that make sharing safe.

```bash
mkdir project && cd project
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo base > shared.conf; git add -A; git commit -qm "base config"
git worktree add -q -b task-retries ../op-a
git worktree add -q -b task-logging ../op-b
( cd ../op-a && sed -i "s/base/retries: 8/" shared.conf && git commit -qam "raise retries" )
( cd ../op-b && echo "log_level: debug" >> shared.conf && git commit -qam "enable debug logging" )
git worktree list
echo "--- one object store, three histories:"
git log --all --oneline
```

```output
/tmp/oailly-gate-6e95k55z/project 0fcb782 [main]
/tmp/oailly-gate-6e95k55z/op-a    0765a35 [task-retries]
/tmp/oailly-gate-6e95k55z/op-b    f626d3b [task-logging]
--- one object store, three histories:
0fcb782 base config
f626d3b enable debug logging
0765a35 raise retries
```

Both parallel operators edited *the same file* — the classic collision — and
nothing collided, because each holds its own tree and its own branch; the
divergence is not an accident to prevent but the recorded, mergeable state
of two minds mid-work, visible whole from any of the three trees (`--all`
reaches every branch through the shared store). The supervisor's dispatch
pattern falls straight out: one repository, one worktree per concurrent
task, each session told its directory and its branch — and volume one's
scratch discipline supplies the frame it slots into, with `worktree add`
replacing `mktemp -d` for exactly the work that must survive and merge.

## The economics of the shared store

What the worktree costs is worth one honest measurement, because the
per-session-clone reflex survives on vague fears of sharing:

```bash
mkdir project && cd project
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo x > f; git add -A; git commit -qm base
git worktree add -q -b t ../wt
echo "main tree git dir:     $(git rev-parse --git-dir)"
echo "worktree git-common:   $(git -C ../wt rev-parse --git-common-dir)"
echo "worktree size: $(du -s ../wt | cut -f1) KB   full clone would carry the whole store"
git worktree remove ../wt && echo "removed cleanly"
```

```output
main tree git dir:     .git
worktree git-common:   /tmp/oailly-gate-97nb2bh5/project/.git
worktree size: 8 KB   full clone would carry the whole store
removed cleanly
```

Eight kilobytes: the checked-out file plus pointers, with `rev-parse
--git-common-dir` showing where the actual store lives — back in the primary
repository, shared. On a real project the arithmetic is decisive: a
repository whose store runs to a gigabyte spawns worktrees at the cost of
the checkout alone, and every object any operator commits is instantly
reachable from every other tree without fetch, push, or copy — the exchange
problem the per-session clone created, dissolved. The store-level operations
consolidate the same way: one `fetch` refreshes every worktree's view of the
remotes, one maintenance pass (gc, repack) serves all, and volume two's
instincts about splitting high-rate state from shared state find nothing to
split — the store's append-only design already made concurrent object
writes safe, which is why git needed no WAL chapter.

Precision about what is *not* shared completes the model, because the
division is exactly the working-state boundary this chapter opened on.
Each worktree privately owns its HEAD (which branch it stands on), its
index (chapter 2's staging transaction — two seats can stage
simultaneously without interleaving), and its tree-local metadata; the
store shares objects, branches, tags, remotes, and configuration. The
consequence operators should hold: anything *committed* anywhere is
instantly everyone's; anything staged-or-working is one seat's private
draft until it commits — the exact draft/publication line chapter 2 drew
inside one operator, now drawn between them, by the tool's own
architecture.

The safety rule the sharing does impose is the single-writer truth in new
clothing, enforced by the tool itself:

```bash
mkdir project && cd project
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo x > f; git add -A; git commit -qm base
git worktree add -q -b task ../wt-one
git worktree add ../wt-two task 2>&1 | head -2
```

```output
Preparing worktree (checking out 'task')
fatal: 'task' is already used by worktree at '/tmp/oailly-gate-z7udmi5d/wt-one'
```

One branch, one working tree — the refusal is git protecting the branch
ref from the two-minds-one-checkout chaos this chapter opened with, and
its reading follows volume two's BUSY discipline: this is coordination
working, not breaking. The operator's responses, in order of likelihood:
the second session wanted *its own* branch anyway (dispatch discipline —
one task, one branch — below); it wanted to *read* that branch, which
needs no checkout at all (`git -C anywhere show task:file`, `log task` —
chapter 3's queries run against any ref from any tree); or it genuinely
found a stale claim — a dead session's worktree still registered — which
is the inheritance problem, and the lifecycle section closes it.

## Branch hygiene for machine fleets

Parallelism multiplies branches, and branches named by machines rot into
namespace landfill faster than humans manage — `fix`, `fix2`, `temp`,
`agent-output-final` — unless naming is treated as what it is in this
series: provenance. The register's convention makes the branch name a
ledger header: *lineage/task-slug*, with the task slug stable enough to
join against volume two's registry (`session-94/raise-retry-budget`), so
that `branch --format` listings read as a work registry and any branch's
owner, purpose, and age are one query. The dispatch rule that keeps the
namespace meaningful: **one task, one branch, born at dispatch, dead at
integration** — branches are workspaces, not archives; the *ledger* is
the archive (chapter 1's boundary, applied to refs). And the aging rule
mirrors volume two's cursor staleness: a standing query for branches
whose last commit predates a threshold (`for-each-ref --sort=committerdate
--format` with a date cut) feeds the graveyard review — merged branches
deleted on integration by the workflow itself, unmerged stale ones
triaged with chapter 3's reading tools (what does it hold that main
lacks?) and either salvaged into the ledger or closed with a recorded
reason, volume one's quarantine discipline for the one kind of state
that never needed a graveyard directory, because deletion of a merged
branch deletes nothing the store does not keep.

## The synchronization cadence

Divergence economics got their numbers in the fleet briefing; the policy
they argue for deserves its own statement, because fleets fail here by
default rather than by decision. A task branch drifts from main at
main's velocity, and the cost of reconciling grows superlinearly — the
conflicts compound, and worse, they arrive *at integration time*, when
the work is done, the context is cold, and chapter 8's reviewer is
waiting. The cadence rule inverts the arrival: task branches synchronize
with main *early and often* — each session's open ritual includes the
fetch and the drift counts, and a branch more than a briefing-threshold
behind merges main in (or rebases onto it, per chapter 6's boundary:
rebase while private, merge once shared) *before* new work, so conflicts
surface one day's worth at a time, in warm context, resolved by the
mind that just created half of them. For stacked work, modern git
removes the classic tax: `rebase --update-refs` carries a whole stack
of dependent branches through one rebase, re-pointing each as its base
moves — the instrument that makes chapter 8's stacked proposals
practical for machine fleets rather than heroic. The judgment call the
cadence rule leaves open is deliberate: a branch hours from integration
may reasonably freeze and reconcile once at the end rather than chase a
busy main commit by commit — cadence is drift *management*, not drift
phobia, and the briefing counts exist exactly so the choice is made
looking at numbers instead of made by forgetting.

## The stash, read suspiciously

One instrument adjacent to this chapter earns a caution rather than a
recommendation. `git stash` shelves uncommitted changes into an anonymous
holding stack — the interactive human's "hold my drink" for a quick branch
switch — and everything that makes it convenient for humans makes it
hazardous for fleets. Stashes are unattached to any branch, unnamed by
default, invisible to every chapter 3 query that reads branches, and owned
by nobody the registry can name: state parked outside the ledger, which is
this series' definition of a midden. The register's substitute is already
on the shelf: the WIP commit on the task branch (chapter 2's checkpoint
pattern) parks the same state *inside* provenance — attached, attributed,
recoverable by the branch's name, cleaned by the same reshaping pass that
was coming anyway. The operator therefore writes stashes rarely (a
worktree per task removes the branch-switching motive entirely), and reads
inherited ones with the unfinished-business protocol: `stash list
--format='%gd %ci %gs'` inventories the stack with dates and origin
branches, `stash show -p stash@{n}` reads each as evidence, and each is
either salvaged into a commit on its proper branch or discarded with a
recorded reason. A repository whose stash stack is deep and old is telling
the briefing something about its operators' discipline — and volume two's
staleness pricing applies to every entry in it.

## The fleet and its remotes

Worktrees share one store, and the store's view of the outside world —
its remotes — is therefore fleet-wide state with fleet-wide discipline.
Fetching benefits first: one `fetch` (scheduled, volume one's timer
patterns) refreshes `origin/*` for every worktree at once, and the
register's operators fetch *before* framing any decision that depends on
the remote's state — a bisect frame, a merge, a review — because a stale
remote view is volume two's meaning-rot in its most actionable form: the
question "am I behind?" (`rev-list --count HEAD..origin/main`, chapter
3's arithmetic) is only as fresh as the last fetch, and the counts
belong in the session briefing. Pushing is per-branch and carries the
dispatch discipline outward: first push sets tracking (`push -u origin
session-94/raise-retry-budget`), after which status and the counts speak
the branch's divergence natively; and every push is preceded by the
behind-check, because pushing into a branch that moved produces the
non-fast-forward refusal — a coordination signal whose correct and
incorrect readings differ so consequentially that the next chapter
spends a section on it. What no fleet member does is push *shared*
integration branches as a side effect of its task: task branches are the
operators' to publish; main moves through the integration ceremony of
chapter 8, one authority at a time — the same one-writer-per-truth
instinct the worktree refusal enforced locally, applied at the remote.

## Composing repositories, avoided knowingly

Fleets eventually ask how repositories themselves compose — the shared
library, the vendored dependency, the platform repo the product repos
lean on — and the built-in answer, submodules, earns this book's most
explicit advisory: understand it, and reach for it last. A submodule
pins another repository at a hash inside a parent tree, which is
exactly right as a *concept* (content-addressed composition, chapter 1
approving) and operationally hostile to unattended work in practice:
clones arrive incomplete until a second command runs, `status` in the
parent goes ambiguous about child state, every briefing and gate in
this book needs submodule-aware variants, and the classic accident — a
parent commit pinning a child hash that exists only on some machine —
is a broken build with no local evidence, the calm face at
architecture scale. The register's preference order for the same
needs: a real dependency gets a *release artifact* and a lockfile (the
ecosystem's package manager is the instrument built for pinning);
code that must live in-tree gets vendored *as content* (a plain copy,
committed, with its origin and version in the ledger entry — chapter
2's provenance carrying what submodule metadata would have), or
`subtree`-merged where history import matters; and only the case that
truly needs live dual-repo development — rare, and staffed by seats
that will maintain the discipline — earns submodules, wired into the
open ritual (`clone --recurse-submodules`, update policy explicit) so
the sharp edges are at least institutional rather than per-seat
surprises. Composition is real; the advisory is only that the default
instrument for it should be the one whose failure modes the fleet's
existing disciplines already cover.

## Constrained seats: sparse and shallow

Two reduced forms of the working arrangement serve the fleet's edge
cases, and both are volume one's least-privilege instinct wearing git's
clothes. Sparse checkout scopes a worktree to a subtree
(`sparse-checkout set services/billing` after `worktree add`): the
operator dispatched against one component sees only that component,
which shrinks its blast radius (a pathspec mistake cannot stage what the
tree does not materialize), its noise floor (status and diff speak only
in-scope), and — for the enormous monorepos where this matters most —
its checkout cost. The seat still holds full history through the shared
store; only the *visible working surface* narrows, which is precisely
the shape a scoped task wants. Shallow clones (`clone --depth=1`)
narrow the other axis — history instead of surface — and belong to a
different niche: the read-only consumer (a CI-style build, a one-shot
analysis) that needs today's tree and no ledger. The register uses them
knowingly for that niche and refuses them everywhere this book's
techniques live, because a shallow store amputates exactly what the
techniques consume: chapter 3's archaeology stops at the horizon,
chapter 4's bisection cannot frame, and chapter 1's inheritance briefing
reads a history one commit deep. The rule of thumb the two forms share:
constrain the *surface* freely (sparse seats compose with everything),
constrain the *ledger* only for seats that will never ask it questions
— and when in doubt about which seat a task needs, the full worktree's
eight kilobytes were never the thing to economize.

## Rejoining: the merge as integration entry

Parallel work exists to converge, and the convergence primitive deserves
its plain demonstration before chapter 8 builds ceremony on it:

```bash
mkdir project && cd project
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo base > shared.conf; git add -A; git commit -qm "base config"
git worktree add -q -b task-a ../ra; git worktree add -q -b task-b ../rb
( cd ../ra && echo "retries: 8" > retries.conf && git add -A && git commit -qm "retries policy" )
( cd ../rb && echo "level: debug" > logging.conf && git add -A && git commit -qm "logging policy" )
git merge -q --no-edit task-a && git merge -q --no-edit task-b
ls *.conf; git log --oneline | head -4
```

```output
logging.conf
retries.conf
shared.conf
48b47e9 Merge branch 'task-b'
3c8b45c retries policy
e9dcb23 logging policy
0fcb782 base config
```

Two operators' work, integrated sequentially into main, both files
present, the merge itself an entry in the ledger (the second merge
recorded as such; the first fast-forwarded silently — the two integration
shapes chapter 8 weighs). What this chapter takes from the demo is the
register's framing of *conflict*, because conflict is where parallel
work's bill arrives. A merge conflict is not an error; it is the tool
reporting that two minds changed the same lines and no algorithm can rank
their intentions — a *finding*, in volume one's vocabulary, demanding
judgment. The non-interactive protocol: attempt the merge with the
combined diff already read (`diff main...task-a`, chapter 3's three-dot
review form — an operator that reads before merging predicts most
conflicts before creating them); on conflict, read the markers as
evidence (`diff` shows both sides annotated; `checkout --conflict=diff3`
adds the ancestor, and the three-way view is the whole story); resolve
by *decision, not deletion* — the resolution is a judgment about intent
that belongs in the merge commit's message (chapter 2: the body answers
why); and when the judgment exceeds the operator's authority — two
plausible intentions, no basis to rank them — the honest move is volume
one's escalation discipline: stop, record the conflict as a finding, and
hand the decision to the supervisor rather than guessing silently. The
worst resolution in the register is the quiet one: `checkout --ours` as
a reflex is deleting a colleague's intention without a hearing, and the
ledger will remember only that the merge "succeeded."

One conflict amenity deserves the fleet's attention because it converts
judgment already spent into judgment reused: `rerere` ("reuse recorded
resolution", enabled once per store) records each conflict's shape and
its resolution, and replays the resolution automatically the next time
the same conflict appears — which in fleet practice is constantly, since
a long-lived task branch merging a moving main re-meets its own
conflicts on every synchronization. The register adds one caution to the
convenience: a replayed resolution is a judgment applied without a fresh
hearing, so sessions note when rerere fired (its output says so) and the
first resolution's reasoning still lands in that integration entry's
message — the replay then inherits a recorded why, rather than becoming
automation of an undocumented decision, which no volume of this series
has been willing to bless.

## The reviewer's seat

One worktree pattern serves chapter 8 directly enough to install here:
review happens in its own tree. The reviewing operator — machine or
human-driven — adds a worktree at the proposal's branch
(`worktree add ../review-212 origin/session-95/harden-retries`,
detached or tracking), and the entire review toolkit runs there
without touching any working seat: the diff read at chapter 3's
resolutions, the build and tests run live (review that executes is
worth two reviews that squint), the suspicious behavior probed with
volume one's instruments — while the reviewer's own task, in its own
tree, stays exactly as it was. The economics repeat the chapter's
opening: the alternative reflexes are stashing or committing half-done
work to switch branches (state churn in the reviewer's seat, the exact
cost worktrees exist to delete) or reviewing from the diff alone
(fine for prose, thin for behavior). And the lifecycle rules apply
unchanged — the review tree is removed when the verdict posts, or
ledgered as open business if the review spans sessions — so `worktree
list` keeps telling the fleet's whole truth: every open review visible
beside every open task, each a named seat with an owner and an age,
which is what a fleet's work-in-flight was always supposed to look
like.

## The fleet briefing

The chapter's instruments compose into the supervisor's standing view —
the parallel-work edition of the briefings volumes one and two
institutionalized — and writing it out fixes the queries as a set. Seats:
`worktree list --porcelain` (the machine format, one stanza per tree)
answers what is checked out where, joined against liveness the way volume
two joined registry rows against processes — a worktree whose branch has
not moved in days and whose session the registry shows ended is inherited
unfinished business, triaged by the lifecycle protocol below. Work:
`for-each-ref refs/heads --sort=-committerdate --format='%(refname:short)
%(committerdate:iso) %(subject)'` is the task registry — every branch,
its owner-by-naming-convention, its freshness, its last claim — with the
staleness threshold marking candidates for the graveyard review.
Parked state: the stash inventory from the suspicious-reading section,
ideally empty. Divergence: the ahead/behind counts against origin for
main and for every active task branch — the numbers that say which work
is ready to integrate, which is drifting from a moving main (the earlier
a task branch merges main's progress, the smaller chapter 8's conflicts
— a fetch-and-count line in each session's own briefing makes the drift
visible daily), and whether anyone is sitting on unpushed work the fleet
cannot see. Six queries, one transcript page, and the answer to the
question every supervisor of parallel machines actually has — *what is
in flight, how stale, and what needs a decision* — read from the
repository itself rather than from the operators' self-reports. The
fleet briefing is also where this chapter's disciplines become
observable: seats named by convention, no anonymous stashes, no
immortal branches, divergence counted daily. A fleet whose briefing is
boring is a fleet whose habits are working — and a briefing that suddenly
grew interesting names, by line, which habit slipped and which seat
slipped it, which is all a supervisor ever needed monitoring to do.

## Lifecycle: worktrees end like sessions

Worktrees are session-shaped, and everything this series knows about
session ends applies. The clean end is `worktree remove` (shown above)
plus the branch's integration-or-triage — the workspace gone, the work
merged or accounted for. The unclean end — a session dies holding a
worktree — leaves the registration behind, and the successor meets it
exactly as volume two taught: `worktree list` is the run registry
(every tree, its branch, its staleness), a dead session's tree is
inspected before disposal (uncommitted changes in it are the dead
session's unfinished stage — read, salvage into a commit on its task
branch, or record the discard), and `worktree prune` clears
registrations whose directories are already gone. The estate closes the
loop: a dispatch pattern that records worktree births and deaths in the
registry gives the fleet's supervisor one query for "what is checked
out where, by whom, since when" — which is this chapter's whole subject,
reduced to the standing question it always was. Parallel operators,
then: one store because the ledger shares safely, one tree and one
branch per mind because working state does not, names that carry
provenance, merges that record judgment, and endings — clean or
inherited — that leave the fleet's workspace as legible as any single
operator's. What parallelism has not yet touched is the ledger's own
integrity across all these hands, and that is the next chapter's
covenant.
