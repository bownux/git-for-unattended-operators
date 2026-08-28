# The Repository Is the Ledger — proposal and evidence map

**Working title:** The Repository Is the Ledger
**Subtitle:** Git for unattended operators
**Shelf:** SYSTEMS & CRAFT (deltas: none)
**Tier:** Pocket (target ~25,500 measured words, 8 chapters)
**Proposed book-id:** rogerai-labs--git-for-unattended-operators
**Status:** proposal + outline only; queued behind the one-in-pipeline rule
(Durable State for Ephemeral Minds, in review). Signature listings pre-tested
under gate conditions (bisect run, worktrees). Not yet written.
**Mascot request (draft):** weaver ant — workers that build by *linking their own
bodies into chains* so others can cross, then binding leaves into shared
structures: collaboration as physical version-joining. Worker-insect taxon per
the SYSTEMS & CRAFT reservation; third panel of the shelf's operator trilogy.

## The book-shaped hole

Git's literature — magisterial though some of it is — assumes an interactive
human: staging hunks by keystroke, resolving merges in a mergetool, rewriting
history in an editor that `rebase -i` opens, learning by fumbling at a prompt.
Meanwhile the fastest-growing population of git users cannot do any of that.
Session-bound operators — agents above all, CI and automation beside them —
commit enormous volumes of change through one-shot commands, and they are
visibly bad at it in ways daily practice now documents: monolithic commits that
bury ten truths in one diff, messages that describe nothing, force-pushes that
destroy colleagues' work, merge conflicts "resolved" by deletion, repositories
treated as file dumps rather than as the append-only, verifiable, *queryable*
history engine git actually is. No standard reference teaches git in the
non-interactive register — commits composed as ledger entries, history read as
evidence, `bisect run` as automated diagnosis, worktrees as the concurrency
model for parallel operators, hooks as acceptance gates, the PR as a handoff
message. The register's operators already own the disciplines these practices
need (this press's first volume taught the conduct, its second the memory);
git is where both meet other people's work, and the craft of that meeting is
book-shaped: commit shape constrains reviewability, history discipline
constrains diagnosis, branch topology constrains collaboration, and every
choice is enforced or squandered at composition time, one shot at a time.

## Reader

The developer supervising agents that commit to shared repositories — and, in
second person throughout, the operator itself: any session-bound worker whose
output is commits others must review, inherit, and trust. Assumes working git
vocabulary (clone, commit, branch, merge) and the register's basics; assumes no
git internals knowledge.

## Boundaries

Grounded in git's own documentation (git-scm.com and the man pages) and
runnable listings — every demonstration executes in scratch repositories built
by the listing itself, in the gate sandbox, with git and the standard shell
alone. The book does not cover forge-specific APIs (PR mechanics are taught as
protocol, with `gh`-style commands as labeled fragments), does not teach git
from zero, does not cover monorepo scaling machinery, and takes no position on
workflow religions beyond what the register's constraints actually decide.

## Chapter architecture and evidence plan

1. **The Other Ledger** — the thesis: a repository is an append-only,
   content-addressed, cryptographically-chained history store the operator
   already holds; the estate/ledger disciplines (volume two) mapped onto
   commits; why interactive git habits fail one-shot operators; porcelain vs
   plumbing as the isatty fork of volume one. Evidence: gitglossary,
   git-scm book's objects chapter; runnable object-inspection listings
   (`cat-file`, hashes as identity).
2. **The Commit as a Unit of Meaning** — one truth per commit; message
   discipline (subject as claim, body as why, trailers as metadata);
   composing commits non-interactively (`add -A` vs pathspec precision;
   `commit -m` multi-paragraph forms); the staging area as the operator's
   transaction. Evidence: git-commit(1), the kernel's submitting-patches
   conventions; worked commit-shape demonstrations.
3. **Reading History as Evidence** — log as query (`--since`, `-S`/`-G`
   pickaxe, `-L` line history, `--follow`); blame read correctly (last
   touch ≠ authorship of behavior); diff discipline (`--stat` first,
   bounded output always); range syntax (`..` vs `...`) demystified in the
   register's determinism terms. Evidence: git-log(1), gitrevisions(7);
   runnable history-mining listings on a scratch repo with planted history.
4. **Diagnosis by Bisection** — the chapter volume one's postmortems were
   missing: `bisect run` as the automated differential diagnosis (tested:
   finds the guilty commit unattended); writing bisect predicates (exit-code
   discipline from volume one, verbatim); `bisect skip`, untestable commits,
   and bounding the hunt. Evidence: git-bisect(1); the pre-tested worked
   bisection.
5. **Parallel Operators** — worktrees as the concurrency model (tested: two
   operators, two branches, one object store, no collisions); why agents
   share a repo not a working tree; branch hygiene for machines (naming as
   provenance, one task one branch); the stale-branch graveyard and its
   retention. Evidence: git-worktree(1), git-branch(1); runnable
   two-worktree demonstrations.
6. **History Is Append-Only (For You)** — the force-push covenant: rewriting
   shared history destroys colleagues' work and forges the record; amend and
   rebase scoped to the unpushed; `revert` as the public undo (the
   reversibility ladder of volume one, applied to history);
   `reflog` as the private black box recorder; recovery of "lost" commits.
   Evidence: git-rebase(1) "recovering from upstream rebase", git-revert(1),
   git-reflog(1); runnable revert-vs-reset and reflog-recovery
   demonstrations.
7. **Gates at the Threshold** — hooks as the repo's own acceptance gates
   (pre-commit for mechanical checks, commit-msg for message contracts;
   hooks the operator *writes* vs hooks it must *survive* — including the
   base-ref-less edge this press's own tooling hit, told honestly);
   `git diff --check`, attribute-driven policies; what belongs in a hook vs
   CI vs review. Evidence: githooks(5); runnable hook demonstrations in
   scratch repos.
8. **The Handoff Is a Pull Request** — the PR as volume one's handoff
   message with a diff attached: what was asked, done, not done, how
   verified, how to undo — mapped onto PR description conventions; review
   as the supervisor's trust interface; responding to review findings
   (the press's own fixed-with-diff-or-rebutted-with-evidence discipline,
   generalized); the merge as publication and the trail it leaves.
   Evidence: git-merge(1), git-request-pull(1) lineage, forge conventions
   as fragments; closes the trilogy's frame.

## Length and listing plan

8 chapters × ~3,200 measured words ≈ 25,500. Listing policy
`executable_plus_marked_fragments`; listings are bash driving git in scratch
repositories (git resolves in /usr/bin on both authoring machine and CI;
`git config user.*` set per-repo inside listings since the sandbox has no
identity). Exec budget target ≤ 38 with `no-run` overflow. All outputs real
transcripts. Measurement discipline per the previous two books: gate-counter
per chapter while drafting; listing-heavy chapters need ~1.6× drafting
instinct.

## Contamination note

Third panel of the trilogy: volume one taught acting, volume two remembering,
this volume versioning-with-others. Cross-references are pointers, never
restatements; the ledger concept is *mapped onto* commits here (the commit as
the ledger entry whose store is the repository), which is new ground — volume
two's ledger is a table the operator owns; this volume's is a history shared
with humans. The catalog-overlap gate is the enforcement.
