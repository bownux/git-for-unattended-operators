# Chapter 6 — History Is Append-Only (For You)

*Draft status: author draft; human verification pending. Outputs are real
transcripts; the push rejection in the two-operator listing is a genuine
refusal between real repositories.*

## The covenant

Chapter 1 proved the store cannot be silently rewritten — content is
identity, history is a hash chain — and immediately named the soft spot: the
*names* pointing into the store move freely, and moving a shared name
backward or sideways is how git's strongest guarantee gets converted into
its worst accident. This chapter is the covenant that guards the soft spot,
and its statement is one sentence with two clauses: **published history is
append-only; private history is yours to reshape until the moment it is
published.** The second clause was chapter 2's repair window. The first is
this chapter, and for the register's operators it carries the force of
volume two's append-and-complete rule with higher stakes, because the
ledger being protected is not one lineage's estate but every colleague's
foundation. A force-push that rewrites a shared branch does not merely
lose work; it *forges the record* — commits colleagues hold, cite, and
build on cease to be part of the official past — and it does so with the
authority of the shared remote, which is why fleets configure the
capability away (protected branches, `receive.denyNonFastForwards`) rather
than trusting each operator's restraint. The register's posture: the
operator never force-pushes shared branches, treats the *ability* to do so
as a configuration bug to report, and reserves `--force-with-lease` — the
guarded form that refuses if the remote moved since last look — for the
one legitimate venue: its *own* unmerged task branch after a sanctioned
reshaping pass, where the only history rewritten is history nobody else
has built on.

## The public undo

The covenant would be unbearable without a compliant undo, and the ledger
has always had one — the entry that records the *reversal* of an earlier
entry:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
printf "retries = 3\n" > app.conf; git add -A; git commit -qm "baseline"
printf "retries = 9000\n" > app.conf; git commit -qam "raise retries aggressively"
git revert --no-edit HEAD >/dev/null
cat app.conf
git log --oneline
```

```output
retries = 3
69b2794 Revert "raise retries aggressively"
2298e6a raise retries aggressively
b088612 baseline
```

The file is back to baseline and the history holds *three* entries: the
mistake, standing; the reversal, explicit; the ledger, intact. Volume one's
reversibility ladder put "the undo that carries its own record" on the top
rung, and `revert` is exactly that — the anti-commit computed and applied
as a new commit, safe on published history because it appends. The
register's operational notes: revert the *newest* first when unwinding a
sequence (reverts apply cleanly in reverse order); reverting a merge needs
`-m 1` to name which parent's line survives, and un-reverting a reverted
merge holds enough subtlety that the operator reads the documentation's
own essay before attempting it; and the revert's message — auto-generated
naming the target — earns a body stating *why* the reversal, because
"Revert X" answers what while the incident that demanded it answers why,
and chapter 2's message discipline does not pause for emergencies. Under
incident pressure the decision tree is short: revert now (reversible,
auditable, fast — the register's default), fix forward only when the
revert itself would break consumers who adapted, and either way the
estate's ledger records the operation with the commit hashes as evidence.

## The black box recorder

Private history's freedom needs a safety net, because "yours to reshape"
includes "yours to destroy by accident" — the mistaken `reset --hard`, the
amend that vaporized a version, the branch deleted with work aboard. The
net is the reflog: a per-repository, private journal of *every place each
ref has pointed*, written automatically, consulted almost never until the
day it is the only thing that matters:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo keep > a.txt; git add -A; git commit -qm "work worth keeping"
echo alsokeep > b.txt; git add -A; git commit -qm "later work, about to be lost"
git reset -q --hard HEAD~1
echo "after the mistaken reset: $(git log --oneline | wc -l) commit(s); b.txt exists: $([ -e b.txt ] && echo yes || echo no)"
git reflog --format="%h %gs" | head -3
git reset -q --hard HEAD@{1}
echo "after recovery:           $(git log --oneline | wc -l) commit(s); b.txt exists: $([ -e b.txt ] && echo yes || echo no)"
```

```output
after the mistaken reset: 1 commit(s); b.txt exists: no
d0b519e reset: moving to HEAD~1
ab02515 commit: later work, about to be lost
d0b519e commit (initial): work worth keeping
after recovery:           2 commit(s); b.txt exists: yes
```

The "lost" commit was never lost — chapter 1's amend demonstration
already showed that rewriting abandons rather than destroys — and the
reflog is the index of the abandoned: every entry a former position, with
the operation that moved it (`reset: moving to…`, `commit:`, the
narrative of the repository's recent life), addressable by the `HEAD@{n}`
syntax the recovery uses. The register's facts to hold: the reflog is
*local and private* (it does not push, it does not clone — a fresh clone
starts an empty one, which is one more reason inherited repositories get
briefings, not assumptions); it expires (defaults measured in weeks — a
recovery deferred is a recovery forfeited); and it covers *ref
movements*, so work that was never committed was never in its
jurisdiction — the register's oldest rule, commit at observable stages,
is also the rule that keeps everything inside the recorder's reach. For
the operator, the reflog converts the scariest moments of repository
life into volume one's calm protocol: stop, read the recorder
(`reflog` with a format, bounded), identify the last good position, move
back with intent, and record in the session ledger what was recovered
and how it was endangered.

## The horizon: how long abandoned means recoverable

The recorder's expiry deserves its own honest accounting, because "nothing
is ever really lost" is folklore with a clock on it. Abandoned objects —
the amended-away commits, the reset-away work — persist in the store as
*unreachable* objects until garbage collection actually removes them, and
gc's own rules keep them well past the reflog entries that name them:
reflog entries expire (order of ninety days for reachable history, thirty
for the unreachable, tunable), and unreachable objects get a grace period
beyond that before pruning. The operational reading has two halves. For
recovery: inside the horizon, everything this chapter promised holds, and
even a *deleted branch* — whose own reflog dies with it — remains
recoverable through HEAD's reflog (which recorded every checkout and
commit on it) or, past the reflogs entirely, through `fsck --lost-found`,
the deep sweep that inventories every unreachable commit in the store and
parks them for inspection: slower, nameless, but exhaustive — the
recovery ladder's last rung before "restore from a colleague's clone."
For hygiene: the horizon means private mistakes genuinely do fade — the
repository does not archive every keystroke forever, which is by design
and fine — so anything worth *guaranteed* survival gets the only
guarantee the system offers: reachability. A ref pointing at it — a
branch, a tag, volume two's estate holding the hash of an artifact
commit — exempts an object from every expiry. The rule compresses to
one register sentence: the recorder buys you weeks; the ledger buys you
forever; know which one you are trusting before you need either. And the
session-bound corollary, since this reader's weeks pass without
continuity of memory: a recovery *deferred to a future session* is a
recovery delegated to the estate — the endangered hash goes into the
ledger now, this session, with the reason, or the future session that
finally has time will have a recorder with nothing left to say.

## The lease, precisely

Because `--force-with-lease` is this chapter's one sanctioned crossing of
a published boundary, its mechanics deserve the precision the register
gives every dangerous instrument. The lease is compare-and-swap — volume
two's optimistic concurrency, applied to a ref: "move `origin/task` to my
new history *only if* it still points where I last saw it." Plain
`--force` is the unconditional write, and the difference is exactly the
lost-update demonstration from volume two's opening chapter: between an
operator's last fetch and its forced push, a colleague (or a CI bot, or
yesterday's own forgotten session) may have pushed to the task branch,
and the unconditional force erases that work without ever learning it
existed, while the lease refuses and reports. The caveat that keeps the
lease honest is its reference point: bare `--force-with-lease` compares
against the local remote-tracking ref — *your last fetch's knowledge* —
so a fetch that happened moments before the push makes the lease
current, while a stale tracking ref makes the lease a guarantee about
last Tuesday. The ritual, therefore: fetch, verify the branch state is
the one your reshaping consumed (the counts, chapter 3), then push with
the lease, in one composed sequence — and if even that window feels
wide, the explicit form (`--force-with-lease=task:<sha>`) pins the
expectation to an exact hash, closing it entirely. An operator that
force-pushes its own task branch through that ritual has rewritten
nothing anyone held; an operator that skips the fetch has a lease
against its own ignorance — validity theater, in volume one's phrase,
and the reason the ritual is stated here rather than left to be derived.

## Knowing which side of the boundary you stand on

The covenant's two clauses meet at a question every reshaping session
must answer first: *is this history published?* — and the register
answers it with queries, not memory, because memory is what
session-bound operators do not have. The containment checks:
`branch -r --contains HEAD` names every remote branch that already
holds the commit about to be reshaped (any answer at all means the
boundary is behind you); `rev-list --count @{upstream}..HEAD` counts
the commits that exist only locally — the reshapeable surplus — while
its inverse counts what the remote holds that you lack; and for the
subtler case of *shared-but-unmerged* (a colleague fetched your task
branch even though no integration happened), the fleet's convention
carries what no query can — which is why chapter 5's dispatch
discipline treats task branches as single-owner by naming, making "who
else might hold this?" answerable from the branch name itself. The
pre-reshape ritual assembles in three lines: fetch (fresh knowledge),
containment check (which side of the boundary), count check (what
exactly is in scope) — and only then the autosquash pass, whose scope
the counts just defined. The habit's cost is seconds; what it prevents
is the covenant's only innocent violation mode — the operator that
rewrote published history *believing* it private, whose sincerity will
comfort nobody rebuilding on Monday.

## The refusal that saves the fleet

The covenant's enforcement at the remote is a refusal every operator
meets weekly, and its correct reading is the difference between fleets
that converge and fleets that clobber. Two real operators, one real
shared remote:

```bash
mkdir work && cd work
git init -q --bare -b main origin.git
git clone -q origin.git op-a 2>/dev/null && ( cd op-a && git config user.email a@example.invalid && git config user.name op-a && echo base > f && git add -A && git commit -qm base && git push -q origin main 2>/dev/null && git branch -qu origin/main )
git clone -q origin.git op-b 2>/dev/null
( cd op-a && echo "colleague progress" >> f && git commit -qam "advance the work" && git push -q 2>/dev/null )
( cd op-b && git config user.email b@example.invalid && git config user.name op-b && echo "stale change" > g && git add -A && git commit -qm "work from a stale view" && git push origin main 2>&1 | grep -E "rejected|fast-forward" | head -2 )
( cd op-b && git fetch -q && git merge -q --no-edit origin/main && git push -q origin main 2>/dev/null && echo "after fetch and merge: push accepted" )
git -C origin.git log --oneline main | head -4
```

```output
 ! [rejected]        main -> main (fetch first)
hint: Updates were rejected because the remote contains work that you do not
after fetch and merge: push accepted
252aa35 Merge remote-tracking branch 'origin/main'
b0b0056 work from a stale view
9a536b6 advance the work
67d744e base
```

Operator B pushed from a stale view; the remote refused — *"the remote
contains work that you do not"* — and the resolution was never force but
integration: fetch the colleague's progress, merge it (chapter 5's
protocol, judgment and all), push the union. The final log holds
everything: A's advance, B's work, and the merge that joined them —
against the alternative timeline where `--force` "fixed" the rejection
by erasing A's commit from the shared record. The non-fast-forward
refusal is the exact analog of volume two's BUSY: coordination working,
addressed to you, meaning *someone else did legitimate work; reconcile
before publishing*. Its diagnostic reading follows the same taxonomy —
routine when the fleet is active (fetch, integrate, retry); suspicious
only when it contradicts the topology (a refusal on a branch only you
own means another seat is writing where it should not — a finding for
the fleet briefing, not a bigger hammer). And the recovery story when
someone *has* forced a shared branch completes the chapter's arc: the
distributed design means every clone that held the erased commits still
holds them — chapter 1's full-replica inheritance as the fleet's
collective reflog — so the response is volume one's incident protocol
(stop, inventory who holds what, re-push the erased work, then fix the
configuration that allowed the erasure), and the uncomfortable
truth-telling afterward belongs in the record, because a forged ledger
quietly repaired is a ledger nobody should trust twice.

## The unprotected zone

This chapter's safety nets share one prerequisite that must be said in
warning type: they catch *committed* work. The reflog records ref
movements; the store holds objects; and uncommitted changes are neither
— which makes the small family of commands that overwrite the working
tree the only genuinely unrecoverable destroyers in daily git, and the
place volume one's blast-radius doctrine applies at full strength.
`restore <file>` (and its ancestor spelling `checkout -- <file>`)
replaces the working copy with the committed version — correct when
discarding is the intent, fatal when the working copy was the only
home of an hour's work; `reset --hard` does it tree-wide; `clean`
deletes untracked files that no git mechanism has ever seen. The
register's handling is exactly its handling of `rm`: proof-of-intent
before dispatch (the `diff` of what is about to be discarded, read —
discarding unread changes is deleting a file unlisted), `clean -n`
always rehearsed before `clean -f` (the dry-run exists; volume one's
doctrine requires it), and the structural cure outranking all
vigilance: chapter 2's commit cadence keeps the unprotected zone
minutes wide, because work that commits at observable stages has, at
any instant, almost nothing standing outside the nets. The zone
cannot be closed — a working tree is by design the one place git lets
state exist without history — but a fleet whose habits keep it narrow
has converted this warning from a hazard into a footnote, which is
where every hazard in this series is sent to live.

## Tombstones for finished work

Chapter 5's lifecycle deletes integrated branches, and the covenant adds
the pattern for the exceptions — work a fleet wants findable forever
without keeping branch namespace cluttered: the archive tag. A branch
whose story matters after death (the abandoned approach whose reasoning
future sessions will want, the release line no longer maintained, the
experiment that answered its question negatively) gets an annotated tag
under a reserved namespace — `archive/session-95-linear-retry`, message
stating why it ended (volume two's reason column, once more) — and then
the branch itself dies on schedule. The mechanics lean on facts already
established: the tag holds the commits reachable forever (the horizon
section's only guarantee), annotated tags carry their own provenance,
and the namespace keeps `branch` listings clean while `tag -l
'archive/*'` remains one query — the graveyard with an index, which is
precisely what volume one's quarantine pattern prescribed for state too
meaningful to delete and too dead to keep underfoot. The reading side
completes it: chapter 3's briefing treats a rich archive namespace as
signal (this fleet finishes its stories), and the searcher who wonders
"was linear retry ever tried?" finds the tombstone, its reason, and the
full branch behind it — institutional memory of the negative result,
which every research tradition knows is the memory most often lost and
most expensive to lose.

## The copy that is not a move

One instrument lives so near the covenant's edge that operators misuse
it in both directions: `cherry-pick`, which applies an existing commit's
*change* elsewhere as a *new commit* — same diff, same message by
default, different parent, therefore (chapter 1's arithmetic) a
different hash. Misreading one: treating the pick as a move — the
original still stands, and a fleet that picks a fix to a release branch
while believing it relocated has two copies whose divergence nobody
owns. Misreading two: panic at the duplicate — the same change under
two hashes looks like history confusion until the mechanics are held.
In practice the eventual merge usually resolves quietly, because both
sides carry *identical content* and content-level merging has nothing
to fight over; and where duplicates must be reasoned about before
that, the patch-identity instruments exist for the purpose — rebase
skips already-applied duplicates by patch-id, and `log --cherry-mark`
annotates a range's commits as equivalent-or-not across branches —
though the graph reads strangely to chapter 3's queries in the
interim. The register's rules
make the instrument boring, which is the goal. Cherry-pick *records
its lineage*: `-x` appends the `(cherry picked from commit …)` line,
turning the copy into a citation — provenance across branches, the
trailer discipline's cousin, and non-negotiable for backports, where
the whole point is that a future reader can join the release branch's
fix to its mainline original. Its legitimate genres are few and named:
the backport to a maintenance branch (the canonical case), the
hotfix promoted ahead of its branch's integration, the salvage of one
good entry from an abandoned branch (chapter 5's triage). And its
anti-genre is the one the covenant exists to prevent at scale:
pick-based workflows that *copy* work between long-lived branches
instead of merging it, manufacturing parallel histories of the same
truths whose reconciliation is everyone's eventual unpaid debt. Copy
with citation, for the named genres, and let integration integrate —
the pick is a scalpel, and fleets that use it as a conveyor rediscover
why the merge exists.

## History operations are ledger operations

Everything this chapter does *to* the shared ledger belongs *in* the
private one, and the join closes the trilogy's bookkeeping. A revert, a
sanctioned reshape, a reflog recovery, a non-fast-forward reconciliation
— each is a world-action in volume two's exact sense, and each lands in
the estate with the currency this chapter mints: hashes. The revert's row
carries the reverted and reverting commits; the reshape's row carries the
before-and-after branch tips (the before hash being, note, the reshaped
history's only durable name once the reflog horizon passes — the estate
outlives the recorder, which is the previous section's rule applied to
the operator's own paper trail); the recovery's row carries what was
endangered, by which operation, and where it was restored. The dividend
arrives at review and incident time: when chapter 8's reviewer asks "this
branch's history looks rewritten — sanctioned?", the answer is a ledger
row with a timestamp and a reason rather than a shrug; when a fleet
postmortem asks who force-pushed what and when, the honest seats have
receipts and the gap in receipts localizes the question. And the
worked-incident narrative assembles all of it: the Monday force-push
discovered (fleet briefing counts contradict the remote), the response
ledgered step by step — inventory of holders, the erased commits' hashes
recovered from a colleague's clone, the re-push, the config fix — and
the journal entry written for the searcher who, a year later, types
`MATCH 'force push'` and inherits the whole case instead of the
folklore version. The covenant protects the shared record; the estate
proves the covenant was kept; and an operator holding both has what
this series has been assembling from its first chapter — an account of
its conduct that does not depend on anyone's trust in its word. The
same join runs the other direction with equal force: volume two's
ledger rows have carried commit hashes since its chapter 2 taught the
`Ledger-Op` convention, and this chapter is where those hashes acquire
their guarantee — an estate that cites `81305e9` cites something the
covenant promises will still mean `81305e9`, verbatim, for as long as
the shared history stands. Cross-referenced records are only as strong
as the weaker register; the covenant makes both strong.

## Reshaping, sanctioned

The covenant's private clause deserves its operating limits stated as
positively as its public clause was stated prohibitively, because
reshaping is not a guilty pleasure — it is how chapter 2's published
stages get made from checkpoint-grade drafts. The sanctioned pass, run
on a task branch before first publication (or after, with
`--force-with-lease`, while the branch remains unmerged and unshared by
convention): `rebase --autosquash` folding the fixups (the scripted
sequence editor from chapter 2), message rewording where claims
sharpened, and — where the branch's base drifted — `rebase` onto
current main so the eventual integration diff is honest (chapter 5's
drift counsel; a rebase is also where `rerere` pays out). The register's
guards around the pass: it runs with a clean tree and a fresh
calibration read of `status` and the branch counts; it never crosses
the publication boundary without the lease guard, and never crosses a
*merge* it shares with anyone under any guard; and its result gets the
same four-question read as any inherited entry before pushing, because
a reshaping pass is an author reviewing its own ledger, and chapter 2's
standards do not soften for self-review. Where rewriting is demanded on
*published* history — the committed secret, the license violation, the
legal removal — the operator recognizes the situation as the
coordinated surgery it is: every holder must participate (the erased
content lives in every clone and every reflog until they act), the
secret is rotated regardless (history rewriting is not revocation —
chapter 2 said it first), and the operation belongs to the supervisor's
authority with the fleet's tooling (`filter-repo`-class instruments),
not to any session's initiative. The covenant, finally, is what makes
the whole trilogy's economics work: because published history only
appends, everything built on it — the estates citing hashes, the
bisections framing ranges, the reviews trusting diffs — builds on rock;
and because private history reshapes freely, the rock is made of
considered entries rather than keystroke archaeology. Append in public,
reshape in private, and never confuse the two — the fleet's whole
version-control ethics, in twelve words. What remains is enforcement
that does not depend on every seat's memory of this chapter, which is
what hooks are for, and where the next chapter begins.
