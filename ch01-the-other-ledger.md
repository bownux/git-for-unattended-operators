# Chapter 1 — The Other Ledger

*Draft status: author draft; human verification pending. Every runnable listing
was executed by the author during writing in a scratch repository the listing
itself creates; printed outputs are real transcripts.*

## The ledger you already carry

The previous volume of this series closed on an operator that had learned to
remain: one estate file, verified, searchable, waiting for whoever wakes up
next. This volume begins with an admission that book's reader may have already
muttered: for one enormous class of operator work, such a ledger has existed
all along, installed on effectively every machine, holding history with
guarantees the estate had to build deliberately — append-only records,
tamper-evident identity, provenance on every entry, inheritance across
generations of workers who never meet. It is the version control system. A git
repository is an append-only, content-addressed, cryptographically chained
history store, and every operator that touches code — which is, increasingly,
every operator — already writes to one daily.

And writes to it badly, which is this book's reason to exist. The practicing
supervisor of agents knows the symptoms by heart: commits that bundle ten
unrelated truths into one unreviewable diff; messages that say "fix" or "update
files" and describe nothing; force-pushes that vaporize a colleague's evening;
merge conflicts "resolved" by keeping whichever side the operator was holding;
repositories treated as file dumps with a save button. None of this is
stupidity. It is the register mismatch this series has met twice before: git's
literature and culture assume an interactive human — staging hunks by
keystroke, resolving merges in a tool that paints three panes, rewriting
history in the editor that `rebase -i` throws open, developing taste through
years of fumbling at a prompt. The session-bound operator has none of that
available. It composes commands blind, one shot at a time, exactly as the first
volume taught — and nobody has written down what good git practice *is* in that
register. The interactive tradition's answer to every one of the symptoms above
is a habit this reader cannot form the interactive way. This book forms them
the other way: as composition-time discipline, demonstrated by listings that
run, in the register the reader actually inhabits.

The frame that organizes everything: **the repository is a ledger the operator
shares with people.** The estate of the previous volume was private
infrastructure — the operator's own tables, its own conventions, inherited by
its own lineage. The repository is the same *kind* of object — an append-only
record of deliberate changes, with provenance — but its readers include humans
with opinions, colleagues with in-flight work, reviewers whose trust the
operator's commits must earn. Every discipline in this book is one of the
estate's disciplines meeting that audience: the commit is a ledger entry others
must be able to review; history is evidence others must be able to trust;
concurrency is not two processes on one file but two *minds* on one project.
Volume one taught the operator to act well alone; volume two, to remember well
alone; this volume is where its work stops being alone.

## The fork, one more time

Volume one opened on `isatty` — the system call at which every tool chooses
its human face or its machine face. Git is that fork's most complete citizen,
so complete that it donated the vocabulary this series has used for three
books: the project's own documentation divides its commands into *porcelain*
(the polished interface for humans) and *plumbing* (the pipes underneath,
built for programs), and its maintainers coined the terms. The operator's
first orientation is knowing which face it is talking to:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo x > tracked; git add -A; git commit -qm base
echo y >> tracked; echo z > untracked
git status --porcelain
echo "---human display for contrast:"
git -c color.status=false status | head -4
```

```output
 M tracked
?? untracked
---human display for contrast:
On branch main
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
```

Two views of one working tree. The human display narrates — branch context,
hints about what to type next, prose an interactive learner needs. The
porcelain-format output (the flag name is a historical joke: a stable
machine format *for* scripts, named after the human layer) is two lines of
fixed-position code: `M` modified, `??` untracked, documented and promised
stable across versions — the exact contract volume one taught the reader to
demand before parsing anything. The operator's standing rules follow in one
breath. Parse only `--porcelain` formats (status, and its cousins across the
suite) or plumbing output (`rev-parse`, `cat-file`, `ls-files`), never the
human displays. Disarm the pager class of traps once per session —
`GIT_PAGER=cat` from volume one's environment preamble, `--no-pager` where
it matters — because `log`, `diff`, and `show` all page when they believe a
human is watching. And know that the editor traps (`commit` bare, `rebase
-i`, `merge` without `-m` on some paths) all have non-interactive doors,
which chapter 2 walks through deliberately. None of this is new discipline;
it is volume one's chapter 1, arriving at the tool where the stakes are
shared with other people.

Two parsing footnotes complete the orientation, both volume-one rules
arriving at git's door. Filenames are hostile input here as everywhere — a
path containing a newline shreds line-oriented parsing of any listing
output — and the machine formats all offer the cure the register expects:
a `-z` variant (`status --porcelain -z`, `ls-files -z`, `diff --name-only
-z`) that terminates records with NUL bytes no filename can contain;
operators that parse repository listings parse the `-z` forms, full stop.
And configuration is the open-ritual problem of volume two wearing git's
clothes: the sandbox listings above set `user.email` and `user.name`
per-repository because an operator's identity is *not ambient* — a
session-bound worker must never depend on whatever global config the
machine happens to carry, and a repository it initializes gets its
identity, its default branch, and any policy switches set explicitly, in
the same breath, by its own ritual. One function opens the estate; one
function initializes the repo; drift dies in both places the same way.

## The repository you inherit

The operator's usual first contact is not `init` but `clone`, and the
clone deserves a paragraph of appreciation in this book's terms, because
it is the estate-inheritance ceremony of volume two performed wholesale.
A clone is not a checkout of the latest files; it is the *entire ledger* —
every commit, every tree, every blob, the full chain back to the root —
transferred into a store the operator now holds locally. Every question
this book teaches — history queries, blame, bisection, diff against any
ancestor — runs against local disk at local speeds, no network, no
server's permission, no rate limit: the register's economics of cheap
reads, delivered by architecture. (The distributed-by-default design also
means every colleague's clone is a full replica — the ledger's backup
story is its adoption story, though chapter 6 will insist that replicas
protect against loss, not against confusion.) The inheritance reflex
transfers from volume two intact: a fresh clone gets a briefing before it
gets work — where is HEAD, what branches exist, how far does history go,
what do the last twenty subjects say about how this project writes its
ledger — and chapter 3 turns that briefing into queries. What a clone does
*not* carry is the other repository's configuration, hooks, or identity:
those are per-copy, which is exactly right (policy is the inheritor's
responsibility, not an infection), and exactly why the open ritual above
exists.

## Two ledgers, one operator

The reader arriving from volume two owns an estate; this volume hands them
a repository; the boundary between the two is worth drawing before habits
form, because each is ruinous in the other's role. The repository holds
what is *shared and versioned*: the code, the configuration-as-code, the
documentation — artifacts whose history is meaningful to other people and
whose every change deserves review-shaped scrutiny. The estate holds what
is *operational and lineage-private*: run registries, cursors, probe
outcomes, the working memory of sessions — records other people never
review line-by-line and whose write rate would make repository history
unreadable. The classic cross-contaminations follow from the definitions.
An estate database committed to a shared repository ships a binary blob
that diffs as noise, bloats every clone, and — the graver half — publishes
an operational record full of paths, hostnames, and (if chapter 4 of
volume two was ignored) worse; estates stay out of shared repos as firmly
as secrets do, and for overlapping reasons. Conversely, a repository used
as an estate — sessions committing scratch state, logs, and downloaded
artifacts because commit is the only save verb the operator knows —
converts the shared ledger into a midden with hashes. The overlap zone is
narrow and principled: *decisions* about the shared work (an ADR, a design
note) are repository material because colleagues must find them beside the
code; the *operational trace* of implementing them (which sessions, what
probes, what failed en route) is estate material, with the commit hash as
the foreign key binding the two — one identity, kept once, pointing both
ways.

## Content is identity

What makes the repository a *ledger* rather than a backup folder begins with
one design decision, demonstrable in four commands: every object in git is
named by the hash of its content.

```bash
mkdir work && cd work
printf "retries = 5\n" > service.conf
git init -q -b main
h1=$(git hash-object service.conf)
printf "retries = 5\n" > copy.conf
h2=$(git hash-object copy.conf)
printf "retries = 6\n" > service.conf
h3=$(git hash-object service.conf)
echo "original:        $h1"
echo "identical copy:  $h2"
echo "one byte moved:  $h3"
```

```output
original:        53d37c741becf6b5212e1c56ea94b5a38d1145fe
identical copy:  53d37c741becf6b5212e1c56ea94b5a38d1145fe
one byte moved:  2f8674c39111a6ad74a71adccc38829139d444ba
```

Identical content, identical name — a different file, a different directory, a
different machine, a different decade, and `retries = 5` under a trailing
newline is `53d37c74…` in every SHA-1 repository on earth (still the default
object format; repositories born under the newer SHA-256 format digest the
same universality with longer names), because the name *is* the content,
digested. One byte moved, and the name is unrecognizable. The reader who
worked through the previous volume has seen this instrument before: it is the
artifact index's content hash, promoted from a column the operator maintains
to the *addressing scheme of the entire store*. Two consequences matter
immediately. Storage deduplicates itself — a thousand commits containing the
same unchanged file all point at one blob, which is why chapter 5's worktrees
and branches cost nearly nothing. And identity becomes portable evidence: a
hash in a handoff message, a ledger row, or a review comment names *exactly
one* possible content, with no room for "which version did you mean?" — the
question that consumes the first ten minutes of every incident call conducted
without one.

## The chain

Objects compose upward, and the composition is the ledger's structure. A
commit is itself an object — hashed like any other — whose content names a
tree (the complete snapshot of the project, itself naming blobs), an author,
a committer, a message, and its *parent commit's hash*:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
printf "retries = 5\n" > service.conf
git add service.conf && git commit -qm "raise retries to 5"
c=$(git rev-parse HEAD)
echo "commit  $c"
git cat-file -p "$c" | head -3
t=$(git cat-file -p "$c" | awk "/^tree/ {print \$2}")
echo "--- tree $t"
git cat-file -p "$t"
```

```output
commit  d1691cd18fca26515f46f9c04aedd659d3f23631
tree 33ae5052e89a4da3e80758f7b755b7b6c3004566
author operator <op@example.invalid> 1787950654 -0700
committer operator <op@example.invalid> 1787950654 -0700
--- tree 33ae5052e89a4da3e80758f7b755b7b6c3004566
100644 blob 53d37c741becf6b5212e1c56ea94b5a38d1145fe	service.conf
```

Read the walk bottom-up and notice the returning guest: the tree's blob is `53d37c74…` — the *same hash* the
previous listing computed in a different repository, because content is
identity and this repository also holds `retries = 5`. Now read it top-down,
because the top-down reading is the ledger property. The commit's hash
digests its content; its content includes the tree's hash, which digests the
entire snapshot; and it includes the parent's hash, which digests *that*
commit's content, which includes *its* parent's hash — all the way to the
root. History is a hash chain: no commit can be altered, no file in any
historical snapshot touched, no message reworded, without changing every
descendant hash in the repository. The demonstration costs one amend:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo one > f; git add -A; git commit -qm "first"
echo two >> f; git add -A; git commit -qm "second"
before=$(git rev-parse HEAD)
git commit -q --amend -m "second, reworded"
after=$(git rev-parse HEAD)
echo "before amend: $before"
echo "after amend:  $after"
git cat-file -p HEAD | grep ^parent
```

```output
before amend: 34e181db1388dad700e6c1d4f7162a1024fb3e2c
after amend:  7ac79c6ff0881516e468ca4842df6b903e400ee8
parent 22031ae65102ea3925f3d1bfbc5452c79dcb80f0
```

The "same" commit, message adjusted, is a different commit — a new hash, a
new object, the old one not modified but *abandoned* (still in the store;
chapter 6 recovers such orphans with the reflog). Nothing was edited in
place, because nothing in this store can be edited in place; there is only
writing new history and moving names to point at it. The previous volume
built tamper-evidence as an optional far rung of its trust ladder — hash
chains for estates with adversaries. Git is that far rung as the *default
substrate*: the repository the operator was going to use anyway is already
append-only, already chained, already carrying provenance (author,
committer, timestamps — the estate's provenance block, natively) on every
entry. What remains is to write entries worthy of the ledger they land in,
which is chapter 2's whole subject — and to respect the one soft spot the
amend just demonstrated: the *store* cannot be silently rewritten, but the
*names* pointing into it can be moved, and moving them on shared history is
the sin chapter 6 exists to prevent.

## Names, and the state of having none

The chain's objects are immutable; everything mutable in a repository is
a *name* — a ref, a file containing a hash — and the operator's mental
model of the name layer prevents a famous class of confusion. Branches
are refs that move as you commit; tags are refs that should not move;
HEAD is the name of *where you stand* — usually an indirection ("HEAD is
`main`", so commits move main), but sometimes a direct hash, the state
called *detached HEAD*, whose scary reputation is pure interactive-era
folklore. Detachment means only "standing on a commit without a branch
underneath"; reads are perfectly ordinary there, and this book's own
instruments produce the state routinely — a bisection probe stands
detached at every midpoint, an archaeology session checks out a
historical hash to run something, a worktree opens at a tag. The single
real hazard is committing while detached: the new entries hang from no
name, becoming chapter 6's reflog-recoverable orphans the moment you
move away. The register's protocol is accordingly two lines: detached
reads need nothing; detached *work* gets a name first (`switch -c
rescue/whatever` — cheap, per chapter 5) or is deliberately disposable
and known so. The deeper habit the ref model installs is reading
`status`'s first line as load-bearing — attached and where, or detached
and why — because every subsequent judgment about what a commit will do
depends on which name, if any, is listening.

## Who wrote this: the identity fields

The commit object the walk exposed carried two name lines, and the
distinction between them is provenance machinery this book's reader is
unusually positioned to need. The *author* is who created the change; the
*committer* is who entered it into this ledger — distinct roles that
coincide in solo work and separate the moment work is relayed: a patch
written by one party and applied by another, a commit cherry-picked across
branches, a rebase (the committer updates; the author survives). For
machine operators the fields are the byline discipline this press itself
runs on, scaled down to a commit. The convention that has emerged for
agent work — and that this book's own production history uses — is
layered attribution: the commit's identity names the accountable lineage
(the operator identity the open ritual set), and `Co-Authored-By:`
trailers in the message body name the contributing minds, human and
machine, in a machine-parseable form the forges already aggregate.
Trailers generally — `Co-Authored-By:`, and any `Key: value` pair an
organization standardizes — are the commit's provenance block in volume
two's sense: structured metadata riding in the message, extractable by
`git interpret-trailers` and the log's trailer formats, no schema
migration required. The rule the register adds to the convention:
identity is *declared, not defaulted* — an operator that commits under
whatever `user.email` the machine had lying around has signed someone
else's name to its ledger entry, which is precisely the kind of accident
the open ritual exists to make impossible.

## Identity, cryptographically

The identity fields above are declarations, and chapter 3 will note what
declarations are worth: whatever the declarer's honesty backs. Where a
fleet needs identity *proven* — commits from outside contributors,
release tags consumed by strangers, regulatory environments — git layers
signatures over the chain: a commit or tag signed with a key (GPG
historically; SSH keys since git 2.34, which for operator fleets is the
practical arrival, since every seat already holds SSH identity for the
remote) carries a verifiable assertion that the keyholder made this
object, checked by `verify-commit`/`verify-tag` or `log
--show-signature` against the fleet's allowed-signers file. The
register's counsel is layered adoption matched to actual threat. The
chain itself (chapter 1's hashes) already guarantees *integrity* —
nobody can alter history unnoticed; signatures add *attribution* — this
exact lineage vouched for this exact object; and most fleets need
attribution enforced at exactly two places: annotated release tags
(the objects strangers consume) and the merge commits of protected
branches (the moments of publication — many forges sign these
server-side as a platform guarantee). Signing every workaday commit
from every agent seat adds key-management surface faster than it adds
trust, and the register prefers the honest middle: declared identity
plus trailers for the daily ledger, cryptographic attestation at the
boundaries where the audience stops being the fleet. What the operator
never does is confuse the layers — a green "verified" badge proves the
key signed it, the key's custody is volume-one key hygiene, and a
fleet that cannot say who holds a key has signatures without
attribution, ceremony without the property it was bought for.

## What this buys the operator

Assembled, the model pays out in the currencies this series prices. Identity
for evidence: volume two's ledger rows and volume one's handoff messages
gain a universal foreign key — the commit hash — that binds "what I did" to
an exact, verifiable, shared state of the world; the phrase *as of
`b0ec917`* is a complete provenance statement, resolvable by anyone holding
the repository. Time travel for diagnosis: every historical snapshot is
addressable and checkout-able, which chapter 4 weaponizes into automated
bisection. Cheap parallelism: branches and worktrees are pointers into the
shared store, so concurrent operators cost pointers, not copies (chapter 5).
And an audit surface the supervisor can trust structurally: the reviewer of
chapter 8 does not have to believe the operator's account of its changes,
because the diff *is* the change and the chain proves nobody retouched it.
The estate taught the operator that records with guarantees beat records
with hopes. The repository is where those guarantees come pre-installed —
and where the records are read by people whose trust is the operator's to
earn or squander.

## What this book claims, and what it refuses to claim

House rules, plain text, early. This book claims that non-interactive git
practice is a learnable craft with specific techniques — commit shaping,
history reading, automated bisection, worktree concurrency, hook gating,
PR-shaped handoff — and it demonstrates each with listings that run in
scratch repositories under the publisher's gate, on git and the standard
shell alone. It claims the interactive tradition's habits fail predictably
in this register, and shows the failures rather than asserting them. It
grounds engine claims in git's own documentation, cited in the back matter.

It refuses the adjacent territory. It does not cover forge platforms' APIs —
pull-request *protocol* is taught as craft; the platform-specific commands
appear only as labeled fragments. It does not teach git from zero; the
reader can clone, commit, branch, and merge, or should meet those verbs
elsewhere first. It does not adjudicate workflow religions (rebase versus
merge, trunk versus flow) beyond what the register's constraints actually
decide, and it says explicitly when a choice is taste. It does not cover
repository-scaling machinery — partial clone, submodule strategy, monorepo
tooling — beyond honest pointers. And it makes no claim that good commits
make good code: the ledger records the work; the work must still deserve
recording, which no version control system has ever supplied. What the
system supplies — a shared, chained, queryable memory of everything the
operator does among people — is exactly enough to be worth a book to an
operator that has one shot per command and colleagues on the other side of
the merge.
