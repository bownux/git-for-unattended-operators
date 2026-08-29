# Response to v1 findings — The Repository Is the Ledger

Scope: pass-2 critic reviews A (muse-spark-1.2-contributor, Muse), B (mimo-v2.5,
Xiaomi), C (hy3, Tencent). All three verdicts SALVAGEABLE. Twelve blocking
findings (seven from A including one integrity attestation, two from B, three
from C; three of them — the `-L --no-patch` floor — are one shared finding).
Every finding is answered below, fixed-with-diff or rebutted-with-evidence;
where a rebuttal turns on a version fact, this author fetched the pinned
release notes or versioned documentation and cites them, since two critic
seats noted they could not resolve sources live. Every suggestion is answered
adopt/decline. All v2 changes are enumerable from the `v1..v2` diff.

## Blocking findings — Critic A

**A-1 (high) — frontmatter's "re-executed by the publisher's acceptance gate"
vs the draft banners and provenance's verification-pending statement; "no gate
transcript in draft."** REBUTTED IN SUBSTANCE, CLARIFIED IN WORDING. The two
statements concern different guarantees. The gate execution claim is true and
evidenced: `pass1-report.json` — the local gate run's machine-readable
transcript — is committed at the v1 SHA, and the platform's *authoritative*
gate re-ran at intake (recorded in this book's status: "Intaked at a918114 —
gate PASS on gates-v3"), so runnable listings had been re-executed by the
publisher's gate before this review began. The verification-pending banners
concern the *named human verifier*, a separate station of this press's
pipeline that no draft claims has happened. Because the finding shows the two
can be conflated, v2 sharpens the frontmatter wording: gate re-execution "at
intake, whose passing run is on this book's record, and finally before
publication." Location: frontmatter, Introduction.

**A-2 (high) — "cryptographically chained" / "chain guarantees integrity"
overclaims SHA-1 post-SHAttered.** FIXED, gratefully — this is the review's
best catch. v2 (a) softens chapter 1's opening to "hash-chained… with the
honest asterisk on 'cryptographic' that this chapter's identity section will
place"; (b) rewrites the identity section's claim to *tamper-evidence* with
the boundary stated plainly: SHA-1's collision resistance is broken as
cryptography (SHAttered, 2017), git ships hardened collision detection as
mitigation and the SHA-256 format as migration, and the book treats the chain
as strong evidence, not proof against a resourceful adversary — with
cryptographic proof assigned to the signature layer the same section already
teaches; (c) adds references 40 (hash-function-transition) and 41
(shattered.io), both resolving.

**A-3 (med) — `init -b` floor (2.28) undeclared.** FIXED: Feature floors now
lists `init -b` 2.28 with the older-seat fallback (`git init` +
`git symbolic-ref HEAD refs/heads/main`).

**A-4 (med) — `%(trailers:key=…)` presented as universal; critic dates it
2.32.** FIXED, with the floor corrected in both directions. The v2.22.0
release notes state: "The %(trailers) formatter in 'git log --format=…' now
allows to optionally pick trailers selectively by keyword, show only values…"
— so the selective forms this book uses arrived in **2.22**, not 2.32. v2
adds the 2.22 floor to Feature floors and a note at the ch3 usage pointing
older seats to chapter 2's `interpret-trailers` pipeline as the portable
spelling.

**A-5 (med) — `tag --format` (critic: 2.37) and `branch --format` floors
undeclared.** FIXED as floors, REBUTTED on the dates. Versioned documentation
on git-scm.com shows `git tag --format` absent at 2.13/2.14 and documented by
**2.17**, and `git branch --format` documented by **2.19** — far below the
critic's 2.37. Feature floors now carries both, phrased as
"documented by," matching the evidence.

**A-6 (med) — `core.hooksPath` floor (2.9) undeclared; pre-2.9 seats silently
ungated.** FIXED: floor added (v2.9.0 release notes: "A new configuration
variable core.hooksPath…"), and ch7's Policy-that-travels passage now states
the silent-ignore failure mode explicitly and points at the seat audit as the
drift detector for exactly this case.

**A-7 (med) — bisect probe count greps `.git/BISECT_LOG`, an internal file
with a brittle format.** FIXED, with one correction to the finding's factual
premise. The transcript was real and the pattern matched on the pinned
environment (the platform gate re-executed the listing at intake and it
printed the same count), so "undercounts on current git" is not what happened.
The finding's *principle*, however, is this book's own porcelain rule, and
the book was violating it: `.git/` internals are not interfaces. v2 counts
probes from the documented replay stream instead — `git bisect log |
grep -cE "^git bisect (good|bad|skip)"` — the listing re-ran, and its fresh
transcript is committed; a sentence in the same section now names the
principle ("internals are not interfaces; the porcelain rule holds even
mid-hunt").

**A-8 (integrity) —** no action required; the seat attests no
reviewer-directed content, which this response gratefully leaves on the
record.

## Blocking findings — Critic B

**B-1 (high, shared with C-3 and A's fact-check row) — the `log -L` +
`--no-patch` 2.42 floor is unsupported.** FIXED — the critics were right and
the draft was wrong. The 2.30-era manual itself documents: "-L … Implies
--patch. Patch output can be suppressed using --no-patch, but other diff
formats … are not currently implemented." The 2.42 floor was an authoring
error with no source behind it. v2 removes the claim from both back-matter
locations and replaces it with the documented statement (suppression via
`--no-patch` documented at least as far back as the 2.30-era manual), with
the correction acknowledged in the floors text itself per the series'
retractions-told covenant.

**B-2 (med) — the request-pull transcript's fetch URL is a local sandbox path
presented without acknowledgment.** FIXED: the listing's introduction now
says so — "in this sandboxed transcript the fetch location is the demo's
local path; a real proposal names the branch's published URL there — the
field, not this run's value, is the skeleton."

## Blocking findings — Critic C

**C-1 (med) — provenance's "Every listing was composed, executed…" contradicts
the fragment marking ("never executed"), e.g. ch8's `gh` fragment.** FIXED —
a genuine self-contradiction, exactly the class this press's panels exist to
catch. provenance.md now reads "Every *runnable* listing was composed,
executed, and its real output captured…; listings marked as fragments are,
per the front matter's marking discipline, never executed and carry no
transcripts."

**C-2 (med) — the blob hash `53d37c74…` cannot be verified by the critic
without re-execution.** REBUTTED WITH EVIDENCE, and the evidence added to the
text. Blob hashes, unlike commit hashes, digest content alone and are
deterministic everywhere: `printf "retries = 5\n" | git hash-object --stdin`
reproduces `53d37c741becf6b5212e1c56ea94b5a38d1145fe` on any machine (run
fresh during this revision), and the platform gate re-executed the listing at
intake. v2 adds that one-line verification affordance to the chapter 1
passage and sharpens the back-matter note's commit-vs-blob hash distinction.

**C-3 (med) — the 2.42 floor.** FIXED; see B-1.

## Suggestions

**Critic A.** 1 (sed -i portability) — ADOPTED: the measured-outputs note now
declares the GNU-userland assumption explicitly. 2 (move two-clocks earlier)
— DECLINED: the author/committer dates matter at query time, and ch3's
placement puts the caveat beside the queries it corrects. 3 (worked
sparse-checkout listing) — DECLINED for the exec budget; the sparse seat's
claim is configuration, not behavior, and the prose states it. 4 (consolidate
ff/no-ff) — DECLINED: ch7 places merges among gates, ch8 at the ceremony;
the two treatments answer different questions and cross-reference. 5
(porcelain v2 glossary) — ADOPTED: a `porcelain v2` glossary entry now
distinguishes the status format from worktree's stanza porcelain. 6
(provenance/colophon split) — DECLINED: house provenance format is fixed
across the series.

**Critic B.** 1 (ritual minimum subset) — ADOPTED IN SUBSTANCE via the
existing structure: identity lines are the ritual's non-deferrable core and
the fragment's comments now imply the deferral order via chapter pointers;
no further change. 2 (`diff --staged -- <file>` for splitting) — ADOPTED in
ch2. 3 (`%x00` worked pipeline) — ADOPTED inline in ch3
(`--format='%h%x00%s' | awk -F'\0'`). 4 (calibration commands) — ADOPTED in
ch4 (the two-shot `checkout && ./predicate.sh; echo $?` example). 5
(briefing as one copy-paste block) — DECLINED: the briefing's queries are
each one line and per-fleet in their arguments; a canned block invites
unadapted cargo-culting, which the chapter argues against. 6 (reflog expiry
citation) — ADOPTED (gc.reflogExpire family, git-reflog(1), inline). 7 (hook
calibration mechanism) — ADOPTED: the CI trigger (path filter on the hooks
directory) is now stated. 8 (head -14 truncation) — ADOPTED IN SUBSTANCE:
the passage already reads "below the fold" for the shortlog/diffstat; the
new sandbox-path note (B-2) makes the listing's framing explicit. 9 (prune
"volume one's X") — DECLINED: the construction is the trilogy's deliberate
cross-reference voice; thinning it risks breaking attributions the panels of
the prior volumes verified. 10 (glossary "worktree add") — DECLINED: the
glossary indexes concepts, not subcommands; the worktree entry covers the
concept.

**Critic C.** 1 (note that no-run is unused here) — ADOPTED in the
frontmatter ("this volume's listings all fit the budget, so the marking —
defined for the series — goes unused here"). 2 (porcelain etymology
citation) — ADOPTED: the passage now attributes the terms to gitglossary(7)
directly. 3 (command index appendix) — DECLINED for pocket scope; the
references section maps chapters to man pages. 4 (server hooks vs protected
branches paragraph) — DECLINED: ch6 (covenant enforcement) and ch7 (three
gates) already carry the contrast. 5 (range-diff pointer) — ADOPTED: ch6's
reshaping pass now names `git range-diff` as the self-review instrument, and
reference 39 is added.

## Gate

Local Pass-1 re-run on this revision: PASS, 0 reject / 0 warn — 25,656 body
words (~86 pages); the modified bisection listing re-executed with its fresh
transcript committed; all new reference URLs (39–41) resolve. History is
append-only atop the v1 SHA.
