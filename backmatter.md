# Back Matter

## Glossary

- **annotated tag** — a tag that is itself an object: tagger, date, message, hashed and chained; the form for every named moment other parties consume.
- **append-only covenant** — published history only appends; private history reshapes freely; never confuse the two.
- **archive tag** — an annotated tag under a reserved namespace preserving a dead branch's story (with its reason) after the branch is deleted.
- **author / committer** — who created a change versus who entered it into this history; each carries its own date, and rebases move only the committer's.
- **bisect run** — binary search over history conducted unattended by a predicate command; exit 0 good, 1–127 bad, 125 skip.
- **bundle** — a single file carrying real history, accepted by clone and fetch as a remote; the repository's transport for environments no forge reaches.
- **cherry-pick** — applying an existing commit's change elsewhere as a new commit (new hash); a copy with citation (`-x`), never a move.
- **commit template** — a message scaffold installed via `commit.template`; the blank that asks the right questions before any gate judges the answers.
- **content addressing** — every object named by the hash of its content; identical content bears identical names everywhere.
- **detached HEAD** — standing on a commit with no branch underneath; ordinary for reads and probes, and named before any work is committed there.
- **fast-forward** — a merge that merely advances the branch pointer, minting no integration entry; `--no-ff` records the moment instead.
- **first-parent** — walking only an integration branch's own spine, treating each merged branch as one step; bisection and log at merge altitude.
- **five answers** — what was asked, what was done, what was not, how verified, how undone: the handoff contract a proposal's description owes.
- **hook** — a script the repository runs at a threshold, empowered to refuse; client hooks are self-discipline, server hooks are authority.
- **idempotency trailer** — the `Ledger-Op:` message trailer joining a commit to the estate operation that produced it.
- **index (staging area)** — the assembly bench for the next entry; snapshots content at `add` time, accepts patches, previewed by `diff --staged`.
- **lease** — `--force-with-lease`: compare-and-swap on a ref, refusing if the remote moved past your last knowledge; only as fresh as your last fetch.
- **no-run marking** — a listing executed by the author but excluded from the gate's per-book execution budget; fragments, by contrast, never run.
- **non-fast-forward refusal** — the remote's report that it holds work you lack; answered by fetch and integration, never by force.
- **notes** — mutable annotations attached beside commits after the fact; afterknowledge in the append-only system's own grammar.
- **pathspec** — staging and querying by explicit path or pattern; the precision instrument that keeps two truths out of one commit.
- **pickaxe (`-S`)** — history filtered to commits where a string's occurrence count changed; the birth-and-death query for any content.
- **porcelain / plumbing** — git's own names for its human-facing and machine-facing layers; machine formats (`--porcelain`, `--format`) are the parseable contract.
- **pull request** — a protocol, not a product: a branch, a base, and a proposal document with enough context to judge integration.
- **reflog** — the private, expiring journal of every place each ref has pointed; the black-box recorder for private history's accidents.
- **replayable hunt** — a bisection resumed across sessions via `bisect log` and `bisect replay`; the hunt's state as two durable artifacts.
- **request-pull** — git's original proposal generator: base, fetch location, endpoint, shortlog, diffstat — the skeleton every forge PR decorates.
- **rerere** — recorded conflict resolutions replayed on recurrence; automation of a judgment that must have been recorded the first time.
- **shallow clone** — history truncated at a depth; correct for read-only consumers, wrong wherever archaeology, bisection, or inheritance briefings run.
- **sparse checkout** — a worktree scoped to a subtree; least-privilege for the working surface while full history remains beneath.
- **squash** — collapsing a branch's entries into one at integration; linear history purchased with commit-level truths.
- **stacked proposals** — dependent tasks shipped as a chain of PRs, each reviewing its own increment; carried through rebases by `--update-refs`.
- **tombstone** — the recorded reason a thing ended, left where its absence will be noticed.
- **trailer** — a `Key: value` line at a message's end; the commit's machine-parseable provenance block.
- **unprotected zone** — uncommitted changes: outside every net; kept narrow by commit cadence, entered only with `diff` read and dry runs rehearsed.
- **worktree** — an additional working tree and checked-out branch over one shared object store; the seat-per-task instrument of fleet parallelism.

## References

1. gitglossary(7) — git's own vocabulary. https://git-scm.com/docs/gitglossary
2. Pro Git, 2nd ed. (Chacon & Straub) — the project's book; objects and internals chapters ground chapter 1. https://git-scm.com/book/en/v2
3. git-hash-object(1). https://git-scm.com/docs/git-hash-object
4. git-cat-file(1). https://git-scm.com/docs/git-cat-file
5. git-config(1) — identity, hooksPath, rerere, commit.template. https://git-scm.com/docs/git-config
6. git-status(1) — porcelain formats and their stability promise. https://git-scm.com/docs/git-status
7. git-add(1) — pathspecs; `apply --cached` interplay for index patching. https://git-scm.com/docs/git-add
8. git-commit(1) — message composition, `--amend`, `--allow-empty`, templates. https://git-scm.com/docs/git-commit
9. git-diff(1) — `--staged`, word diffs, `--color-moved`, three-dot semantics. https://git-scm.com/docs/git-diff
10. git-interpret-trailers(1) — the trailer block as parseable metadata. https://git-scm.com/docs/git-interpret-trailers
11. git-log(1) — bounds, formats, `-S`/`-G`, `-L`, `--follow`, dates. https://git-scm.com/docs/git-log
12. gitrevisions(7) — ranges: two-dot, three-dot, `@{upstream}`, `HEAD@{n}`. https://git-scm.com/docs/gitrevisions
13. git-blame(1) — `-w`, `-C`, `--ignore-rev` and the ignore-revs file. https://git-scm.com/docs/git-blame
14. git-shortlog(1) — contribution summaries. https://git-scm.com/docs/git-shortlog
15. git-tag(1) — lightweight vs annotated; verification. https://git-scm.com/docs/git-tag
16. git-notes(1) — attachable annotation refs. https://git-scm.com/docs/git-notes
17. git-bisect(1) — run, skip (125), terms, log and replay, first-parent. https://git-scm.com/docs/git-bisect
18. git-worktree(1) — add, list, remove, prune; one-branch-one-tree. https://git-scm.com/docs/git-worktree
19. git-branch(1) — formats, containment queries. https://git-scm.com/docs/git-branch
20. git-stash(1) — the shelf this book reads suspiciously. https://git-scm.com/docs/git-stash
21. git-sparse-checkout(1) — scoped working surfaces. https://git-scm.com/docs/git-sparse-checkout
22. git-merge(1) — fast-forward vs `--no-ff`; conflict mechanics. https://git-scm.com/docs/git-merge
23. git-rebase(1) — autosquash, `--update-refs`, upstream-rebase recovery. https://git-scm.com/docs/git-rebase
24. git-cherry-pick(1) — `-x` citation; patch identity. https://git-scm.com/docs/git-cherry-pick
25. git-revert(1) — the public undo; reverting merges (`-m`). https://git-scm.com/docs/git-revert
26. git-reflog(1) — the recorder and its expiries. https://git-scm.com/docs/git-reflog
27. git-restore(1) — the unprotected zone's principal instrument. https://git-scm.com/docs/git-restore
28. git-clean(1) — `-n` before `-f`, always. https://git-scm.com/docs/git-clean
29. git-push(1) — non-fast-forward refusals; `--force-with-lease` semantics. https://git-scm.com/docs/git-push
30. githooks(5) — every threshold, client and server. https://git-scm.com/docs/githooks
31. git-fsck(1) — object-store audit; `--lost-found`. https://git-scm.com/docs/git-fsck
32. git-maintenance(1) — scheduled repository upkeep. https://git-scm.com/docs/git-maintenance
33. git-request-pull(1) — the proposal's original instrument. https://git-scm.com/docs/git-request-pull
34. git-format-patch(1) — entries as mailable artifacts; cover letters. https://git-scm.com/docs/git-format-patch
35. git-bundle(1) — history as a transportable file. https://git-scm.com/docs/git-bundle
36. Linux kernel: Submitting Patches — the conventions commit-message craft descends from. https://www.kernel.org/doc/html/latest/process/submitting-patches.html
37. *Linux for Language Models*, O'AILLY Systems & Craft — trilogy volume one. https://oailly.com/read/rogerai-labs--linux-for-language-models/
38. git-mv(1) — renames as recorded moves (and their inference at read time). https://git-scm.com/docs/git-mv

## Feature floors

The git features this book leans on beyond the ancient core, with the
versions that introduced them, in one place: `git worktree` 2.5 (2015) ·
`--porcelain=v2` status 2.11 · `sparse-checkout` command 2.25 ·
`git maintenance` 2.30 · `bisect --first-parent` 2.29 · `restore`/`switch`
2.23 · SSH commit/tag signing 2.34 · `rebase --update-refs` 2.38 ·
`log -L` with `--no-patch` 2.42. Every floor is comfortably below any
currently maintained distribution's git; inherited machines check with one
`git --version`, and the techniques degrade gracefully (older spellings —
`checkout` for `restore`, manual stack rebasing for `--update-refs` — are
noted where the text teaches the modern form).

## A note on measured outputs

Outputs printed in this book's listings are real transcripts from the
authoring machine (Gentoo Linux, kernel 6.18.31-gentoo-dist), captured
2026-08-28 in scratch repositories under the publisher gate's environment.
Hashes shown are those runs' real hashes and will differ on re-execution
(commit objects digest their dates); refusal messages, statuses, and
behaviors are the reproducible claims. Listings using `git log -L` with
`--no-patch` require git 2.42 or later.
