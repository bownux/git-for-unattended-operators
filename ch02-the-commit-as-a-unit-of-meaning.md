# Chapter 2 — The Commit as a Unit of Meaning

*Draft status: author draft; human verification pending. Outputs are real
transcripts from scratch repositories the listings build.*

## The entry, not the save

Every discipline in this chapter descends from one reframing: a commit is not
a save; it is a *ledger entry*. The save mentality — accumulated changes
flushed to safety whenever anxiety or a session boundary strikes — produces
exactly the commits that make supervisors distrust machine operators: forty
files, six unrelated intentions, a message that says "updates". The ledger
mentality asks of every commit the question volume two asked of every
transaction: *what single truth does this entry record?* — and the costs of
ignoring the question are concrete enough to enumerate, because each lands on
a different chapter of this book. A monolithic commit cannot be *reviewed*
well: the reviewer of chapter 8 must untangle which hunks serve which
intention, and review quality degrades toward skimming — the operator's
trust-earning surface, squandered. It cannot be *reverted* alone: chapter 6's
public undo works commit-wise, so the emergency rollback of the bad half drags
the good half with it. And it cannot be *blamed* precisely: chapter 4's
bisection identifies guilty commits, and a bisection that lands on a
six-intention monolith has answered "which commit?" while leaving "which
change?" — the question that actually matters — as manual archaeology. Review,
revert, bisect: three machines that consume commits, all of which run better
on small, single-truth entries. The operator does not shape commits to be
tidy. It shapes them because everything downstream eats what it commits.

The register makes the discipline easier than it is for humans, which is
worth saying plainly as encouragement. An interactive developer's working
tree accretes changes organically — exploration, side-fixes, drive-by
cleanups — and untangling them at commit time requires the hunk-level
staging this reader cannot use. A session-bound operator's changes are
*already* the output of deliberate, enumerated actions: volume one's
operators compose edits one intention at a time and verify each before the
next; volume two's ledger discipline records each world-action singly. The
commits this chapter wants are those same units, carried one step further
into the shared ledger. An operator that works in single truths and commits
in monoliths is throwing away structure it already had.

## The staging area is your transaction

Git's staging area — the index — bewilders newcomers as pure ceremony:
why not commit the working tree directly? For this book's reader the answer
is immediate, because volume two built the same machinery under a different
name: the index is the *staged copy* in the atomic-swap pattern — the place
where the next entry is assembled, inspected, and made exactly right while
the working tree (the operator's live workspace) churns on undisturbed. The
semantics have one sharp edge that one-shot operators must know cold,
because it bites precisely when a session edits, stages, and edits again:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo v1 > f && git add -A && git commit -qm base
echo v2 > f
git add f
echo v3 > f
git status --porcelain
git commit -qm "advance to v2"
echo "committed content: $(git show HEAD:f)"
echo "working tree:      $(cat f)"
```

```output
MM f
committed content: v2
working tree:      v3
```

`git add` does not mark a *file* for committing; it snapshots the file's
content *at that moment* into the index. The later edit (v3) exists only in
the working tree; the commit faithfully recorded the staged v2; and the
porcelain status told the whole story in two characters — `MM`, staged
modification *and* unstaged modification, the two-column code whose first
column describes index-vs-HEAD and second column working-tree-vs-index. An
operator that reads `MM` and commits anyway is choosing to publish v2 while
holding v3, which is occasionally exactly right (the staged version was the
reviewed one) and more often a session about to be confused by its own
ledger. The composition rule that prevents the accident is the same
edit-then-verify rhythm as ever: stage, *then* read status porcelain, then
commit — never `add` in one breath and `commit` in a distant later one with
edits between.

The index also answers the operator's scoping instrument. `git add -A` is
the monolith machine: everything changed, everything staged, strays
included — the `rm $f` of this domain, correct only when "everything" is
genuinely one truth. The precise tool is the pathspec — staging by explicit
path or disciplined pattern — and with it, a working tree holding two
truths becomes two clean entries:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
printf "retries = 5\n" > service.conf
echo "notes on the outage" > incident.md
git add -A && git commit -qm "initial state"
sed -i "s/5/8/" service.conf
echo "root cause: dns" >> incident.md
git add service.conf
git commit -qm "raise retries to 8 for flaky upstream"
git add incident.md
git commit -qm "record outage root cause"
git log --oneline --stat | head -8
```

```output
9a47ba2 record outage root cause
 incident.md | 1 +
 1 file changed, 1 insertion(+)
322bbcb raise retries to 8 for flaky upstream
 service.conf | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)
fd86b0f initial state
 incident.md  | 1 +
```

One session, two intentions, two entries — each with a one-file stat a
reviewer absorbs at a glance, each revertable alone, each carrying its own
why. The log's `--stat` rendering *is* the payoff made visible: history
that reads as a ledger. The boundary test, adapted from volume two's
transaction rule: **stage together exactly what a successor must never
half-see** — the rename and every reference to the renamed thing, the
schema change and its migration entry, the fix and its test. And the
converse: changes that merely happened in the same session share no claim
on the same commit, however convenient `-A` makes their bundling.

## Splitting below the file: the index takes patches

One staging problem seems to demand the interactive tool this reader
cannot use: two truths tangled *inside one file* — the bug fix and the
drive-by rename sharing a function, where `add -p`'s keystroke-driven
hunk picking is the human answer. The register's answer is that the
index accepts *patches*, not just files, and patches are text an
operator can compose: `git diff` emits the file's full change; the
operator splits that diff — keeping the fix's hunks, dropping the
rename's — and `git apply --cached` stages exactly the edited patch,
leaving the working tree untouched and the remainder for the second
commit. The craft caveats: hunk headers carry line offsets, so the
operator splits at hunk boundaries (whole hunks kept or dropped — the
common case, since distinct truths rarely share a hunk) rather than
editing hunk interiors, and verifies the split with the chapter's
standing pair — `diff --staged` shows truth one, `diff` shows truth
two, both read before either commits. When the truths *do* share a
hunk, the honest fallback is simpler than patch surgery: edit the file
to contain only truth one (volume one's file disciplines), commit,
restore truth two, commit — the working tree as staging area, two
clean entries, no tool heroics. Both routes close the last gap between
this chapter's ideal and practice: there is no mixture the register
cannot separate into single truths; there are only mixtures whose
separation costs more than not creating them, which is what the
cadence section's advice was quietly pricing all along.

## What never enters the ledger

Staging discipline has a structural ally that decides most cases before any
session weighs them: the ignore policy. Chapter 1 drew the estate/repository
boundary in principle; `.gitignore` is where the boundary is *enforced*, and
the register treats it as policy code — versioned, reviewed, committed
first:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
printf "*.tmp\nestate.db*\n/artifacts/\n" > .gitignore
git add .gitignore && git commit -qm "ignore policy: scratch, estates, artifacts"
echo x > work.tmp; mkdir artifacts; echo y > artifacts/build.bin; echo z > estate.db
git status --porcelain
echo "--- why is estate.db invisible?"
git check-ignore -v estate.db work.tmp
```

```output
--- why is estate.db invisible?
.gitignore:2:estate.db*	estate.db
.gitignore:1:*.tmp	work.tmp
```

The porcelain status printed *nothing* — three fresh files, all invisible,
because the committed policy already classifies them: scratch stays
scratch, estates stay private, artifacts stay in the artifact store. That
silence is the demonstration: an operator whose ignore policy is right
commits with `-A` far more safely, because the sweep can only gather what
policy admits. And `check-ignore -v` is the accountability query — *which
rule, which line, decided this file's fate* — the first diagnostic when a
file refuses to stage or a stray appears where policy should have caught
it. Three composition rules complete the practice. The policy is written
for the *categories* this series already defined (scratch patterns, estate
files, build outputs, credentials), not accreted one annoyed filename at a
time. It ships in the repository's first commits, because a policy that
arrives after the strays is archaeology. And it is not a security control:
ignore prevents *accidental* staging only — a secret that does get
committed is published history, and the recovery is chapter 6's grim
exception (history rewriting coordinated with every holder, plus rotation
of the secret itself, because the chain remembers what the rewrite
removes from view). Cheap policy up front; no good options after.

The mirrored question — what *does* belong despite instinct — has a
register answer too. Generated files earn a commit exactly when colleagues
must review them or reproducibility requires them pinned: lockfiles yes
(they are the build's truth, and their diffs are review material);
compiled outputs and rendered artifacts no (they are derivable, they bloat
every clone, and their diffs are noise — volume two's artifact index is
their home). Empty commits — entries with a message and no diff — are
legitimate exactly where a ledger needs a marker whose evidence lives
elsewhere: a release point, a recorded decision; `--allow-empty` exists
because sometimes the claim *is* the content. Both rules are one principle
seen twice: the ledger records what its readers must weigh, and nothing
else.

## Renames are inferred, not recorded

One storage fact shapes commit composition enough to earn its place
here: git does not record renames. The ledger stores snapshots (chapter
1's trees); "renamed" is a *conclusion* tools draw at read time by
noticing a vanished path and an appeared path with sufficiently similar
content — which is why `log --follow` and `blame -C` exist as options
rather than defaults, and why their inference has a breaking point. A
rename combined with heavy edits in the same commit can drop below the
similarity threshold, at which point every reading tool sees an
unrelated deletion and creation: the file's history amputates (chapter
3's trap, now with its mechanism), blame restarts at zero, and review
displays a full-file replacement where a reviewer needed a diff. The
composition rule follows with unusual crispness: **rename in one
commit, edit in the next** — the move at near-100% similarity, trivially
inferred forever after, and the edit reviewed as the modest diff it is.
The same logic generalizes to every mechanical/semantic mixture (the
reformat-plus-fix, the move-plus-refactor): inference-dependent
readers, human and machine, survive the mechanical layer only when it
arrives pure. It is chapter 2's one-truth rule again, but with teeth
the style argument lacked — mix the truths here and the tooling itself
starts telling worse stories about your history, to everyone, for the
file's whole remaining life.

## Commit cadence: entries at observable stages

One question remains before message craft: *when*, during a long
autonomous session, should entries land? Volume one answered for
operations (make each stage's completion observable); the ledger version
is: **commit at every observable stage** — after each verified unit, not
at the session's end in one heap, not at anxiety intervals mid-thought.
The payoffs compound across this series' concerns. A session that dies
mid-task leaves a clean committed prefix plus a working tree holding
exactly the interrupted stage — volume one's retry doctrine (read the
evidence, resume at the proven point) gets its evidence from `status` and
`log` instead of forensics. Review inherits stages instead of heaps.
Bisection inherits fine-grained history. And the estate's run registry
gains its natural join: a session's registry row, its ledger operations,
and its commit range tell one story in three registers. The cadence has a
floor as well as a ceiling — commits *smaller* than an observable stage
(one per file touched, one per command run) shred meaning as surely as
monoliths bury it; the unit is the verified stage: the test now passing,
the config now valid, the subsystem now migrated. Where safety wants
snapshots faster than meaning accrues, the private-branch checkpoint
pattern from this chapter's close covers the gap: checkpoint freely,
reshape before sharing, publish stages.

## The message is the claim

If the diff is the entry's evidence, the message is its *claim* — the one
part of the ledger written purely for future readers, and the part machine
operators most reliably squander. The register's composition, demonstrated
whole and then dissected:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo x > f && git add -A
git commit -q -m "cap GPU power at 500W at boot" -m "PSU trips on transient spikes when both cards boost together; capping at boot prevents the trip window before the daemon applies profiles. Verified: vendor tool reads 500 after reboot." -m "Ledger-Op: gpu-power-cap:2026-08
Co-Authored-By: operator-session-93 <op@example.invalid>"
git log -1 --format="SUBJECT: %s%nBODY: %b" | head -6
echo "--- trailers, parsed:"
git log -1 --format=%B | git interpret-trailers --parse
```

```output
SUBJECT: cap GPU power at 500W at boot
BODY: PSU trips on transient spikes when both cards boost together; capping at boot prevents the trip window before the daemon applies profiles. Verified: vendor tool reads 500 after reboot.

Ledger-Op: gpu-power-cap:2026-08
Co-Authored-By: operator-session-93 <op@example.invalid>

--- trailers, parsed:
Ledger-Op: gpu-power-cap:2026-08
Co-Authored-By: operator-session-93 <op@example.invalid>
```

The mechanics first, since they are the register's whole reason this works
without an editor: repeated `-m` flags become paragraphs, so
subject-body-trailers composes in one shot, no `$EDITOR` trap, no here-doc
gymnastics required (though `git commit -F -` with a here-doc is the equal
citizen for messages built by tooling). The anatomy carries fifty years of
convention worth honoring because every tool downstream assumes it. The
*subject* is the claim compressed: imperative mood, capitalized, no period,
targeted under fifty characters and hard-capped by convention around
seventy-two, because `--oneline` views, forge UIs, and shortlog digests
show the subject alone — it is the entry's row in every summary the
supervisor will ever scan. The *body* answers the question the diff cannot:
*why* — the situation that demanded the change, the alternative rejected,
and (house discipline from volume one) the verification performed, stated
as evidence. What the body never does is narrate the diff — "changed X to
Y" restates what `show` displays authoritatively; the reviewer has the
diff, and needs the reasons. And the *trailers* are the provenance block:
machine-parseable `Key: value` lines at the message's end, extracted
cleanly by `interpret-trailers` as the transcript shows — attribution
(`Co-Authored-By`), issue linkage, and, for this book's reader, the key
that closes the loop with volume two: a `Ledger-Op:` trailer carrying the
estate's idempotency key binds the commit to the operation that produced
it, making "which session did this and what else did it do" a join instead
of an investigation.

## Rehearse the entry

Volume one's doctrine — rehearse anything you cannot take back — lands here
with unusual grace, because the staging design gives the rehearsal for
free. The staged entry can be read *exactly as it will be recorded* before
recording it:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo a > f && git add -A && git commit -qm base
echo b >> f && echo temp > scratch.tmp
git add f
git diff --staged --stat
git diff --stat
```

```output
 f | 1 +
 1 file changed, 1 insertion(+)
```

`diff --staged` answers "what will this commit contain?"; plain `diff`
answers "what am I leaving behind?" — here, nothing staged-but-unwanted
and nothing wanted-but-unstaged (the scratch file, untracked, correctly
appears in neither). That pair of reads, run before every commit, is the
proof-of-target discipline: the first is the entry's preview, the second
the check that no intended change was orphaned. The full-text form
(`diff --staged`, unabridged) is the actual rehearsal for consequential
entries — bounded, per volume one, with `--stat` first and the full diff
only at the size the stat justifies — and `git commit --dry-run` adds the
final formality, reporting what would be committed without committing.
An operator that reads its staged diff before committing catches, at the
cheapest possible moment, every accident this chapter has named: the
stray file `-A` swept in, the v3-vs-v2 surprise, the second truth hiding
in the first truth's entry. Thirty seconds of read against an immutable
entry in a shared ledger — volume one's economics have rarely priced
anything so lopsidedly.

## Wrong-sized anyway: the private repair window

Discipline notwithstanding, operators will sometimes commit and then see
the flaw — the typo in the subject, the file that belonged in the previous
entry, the truth that turned out to be two. The repair instruments exist
and are non-interactive; what bounds them is *audience*, and the bound is
absolute enough to state before the tools. A commit that has been pushed
to a shared branch is published history — other operators may already
hold it, build on it, cite its hash in their own ledgers — and repairing
it in place is chapter 6's cardinal sin, forgery-shaped even when
innocent. A commit that exists only locally is a *draft entry*, and
drafts are the operator's to reshape freely. Within that window: `commit
--amend` re-opens the newest entry (the chapter 1 demonstration showed
its mechanics — a new commit, the old abandoned), `--amend --no-edit`
folds a forgotten file into it, and deeper reshaping — combining fixup
commits into their targets across the last few entries — runs
non-interactively through the door volume one taught for every
editor-insisting tool: `rebase --autosquash` with the sequence editor
scripted (`GIT_SEQUENCE_EDITOR=:` accepts the generated plan verbatim),
consuming the `commit --fixup=<target>` entries the session dropped as it
noticed flaws. The pattern that keeps checkpoint anxiety and entry
discipline compatible: commit checkpoints freely on the private branch
while working — safety is cheap — then spend one reshaping pass before
the branch is shared, so what publishes is the ledger the work deserved.
The boundary, restated once because everything in chapter 6 hangs on it:
*reshape drafts, never publications.*

## Reading an entry like an operator

Composition is half the craft; the other half is consuming commits others
made — the inheritance problem again — and volume one's four-question
transcript routine adapts to the ledger entry nearly clause for clause.
First the *claim against the evidence*: does the subject describe what the
diff actually does? The disagreement cases are the diagnostic gold — a
subject narrower than its diff ("fix typo" touching four hundred lines)
flags either a careless bundler or a change hiding inside a trivial one,
and both readings demand the full diff before trust; a subject broader
than its diff flags work that was intended and not completed, the open
intent of volume two wearing git's clothes. Second the *shape*: the
`--stat` silhouette before any content — file count, spread across
subsystems, insert/delete balance — because shape anomalies (the
one-line fix touching thirty files; the "refactor, no behavior change"
that is 90% insertions) are cheaper to catch than content anomalies and
usually decisive about how deeply to read. Third the *provenance*: author,
committer, trailers — who claims this work, which operation produced it
(the `Ledger-Op` join, when the convention holds), and whether the
verification the body claims is stated as evidence ("tests pass") or as
hope ("should work") — volume one's evidence-theater detector, applied to
messages. Fourth the *absence check*: what the entry should contain and
does not — the test that should accompany the fix, the migration that
should accompany the schema change, the documentation the new flag owed —
because an entry's gaps, like a transcript's silences, are findings that
no amount of reading its contents will surface. The routine takes under a
minute against a well-shaped entry, longer against a monolith — which is
itself the economics of this chapter, experienced from the consumer's
side, and the fairest argument for imposing on one's own commits the
discipline one's own reviews will wish for.

The reading commands compose to the routine's rhythm, bounded per volume
one throughout. `git show --no-patch --format=fuller <hash>` serves
questions one and three in a dozen lines — full message, both identities,
both dates — without a byte of diff; `show --stat` adds the silhouette
for question two; and the full `show`, the expensive read, is spent only
on entries the cheaper reads flagged, with pathspec narrowing (`show
<hash> -- path/`) when only one file's role is in question. The pager
trap applies to all of them under interactive detection and to none of
them under capture — but the operator that sets `GIT_PAGER=cat` in its
preamble never has to remember which, which was volume one's argument
for preambles the day it made it.

The entry, then: one truth, staged precisely, previewed exactly, claimed
in a subject the summaries will carry, justified in a body the diff
cannot supply, attributed in trailers machines can parse, and repaired
only while it is still yours alone. Ledger entries of that shape are what
make the next chapter possible at all — because history worth reading is
made of commits that were written to be read, and reading history is the
operator's next superpower.
