# Chapter 7 — Gates at the Threshold

*Draft status: author draft; human verification pending. Outputs are real
transcripts; the blocked commits in the listings are genuine hook refusals.*

## Discipline that does not depend on remembering

Six chapters of discipline share one weakness: they live in the operator's
conduct, and conduct is what session-bound operators cannot carry between
sessions. Volume two met the same weakness with schema — constraints that
outlive their authors — and git's version of schema is the hook: a script
the repository runs at defined moments of the ledger's life, empowered to
refuse. A pre-commit hook runs before an entry is composed; a commit-msg
hook reads the claim before it is accepted; their server-side cousins
guard the shared remote itself. Together they are the mechanism by which a
project's standards stop being chapter 2's advice and become chapter 3's
CHECK constraints — enforced at the threshold, on every seat, including
the seats that never read the advice. The publisher of this series runs
its whole press on the pattern: every manuscript passes a mechanical gate
(structure, citations, code execution) before any human judgment is
spent, and the gate is public so authors run it themselves first. That
two-step — self-run gate, then authoritative gate — is exactly the
architecture this chapter builds at repository scale.

## The pre-commit gate

The first threshold guards entry composition, and its demonstration is
the register's favorite kind — a mistake refused at the cheapest possible
moment:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
cat > .git/hooks/pre-commit <<'PEOF'
#!/bin/sh
if git diff --staged | grep -qE "^\+.*(TODO-BEFORE-SHIP|DO NOT COMMIT)"; then
  echo "pre-commit: staged changes contain a do-not-ship marker" >&2
  exit 1
fi
PEOF
chmod +x .git/hooks/pre-commit
echo "code with TODO-BEFORE-SHIP marker" > f.txt
git add -A
git commit -qm "try to ship it" 2>&1
echo "commit exit: $?"
sed -i "s/ with TODO-BEFORE-SHIP marker//" f.txt
git add -A && git commit -qm "ship it clean" && git log --oneline
```

```output
pre-commit: staged changes contain a do-not-ship marker
commit exit: 1
6d3aeaa ship it clean
```

The contract is volume one's in miniature: the hook is a shot; exit zero
admits the commit, nonzero refuses it, and stderr carries the reason the
refused operator will read in its transcript. What belongs behind this
threshold follows from where it sits — *instant and mechanical*: marker
scans like the demo's, formatting and lint on the staged files, secrets
detection (the last line of chapter 2's defense, and the one that pays
for every other hook the day it fires), fast unit checks on the touched
component. Three composition rules keep the gate an asset. It checks the
*staged* content (`diff --staged`, the entry being judged), never the
working tree — the v2/v3 distinction from chapter 2, which naive hooks
get wrong and then refuse commits for changes that were never in them.
It is *fast* — a threshold crossed dozens of times a session amortizes
seconds, not minutes; the expensive checks have their own gate below.
And it is *deterministic*, because volume one's rule about flaky
predicates applies with a cultural corollary: a gate that refuses
randomly teaches every seat the bypass habit, after which it guards
nothing.

A fleet's standing pre-commit suite, cataloged once as a starting kit:
the do-not-ship marker scan (above); a secrets scan tuned to the
credential shapes the fleet actually holds (key prefixes, token
formats — and tuned *tight*, because this is the one gate whose false
negatives are catastrophic and whose false positives are merely
annoying, the opposite weighting from every other check); a size guard
refusing files above a threshold (chapter 1's artifact boundary,
enforced mechanically — the estate database or build output that
wandered toward the ledger meets the gate instead of the review); and
validity checks for everything the repository holds as config-as-code
(the JSON that must parse, the YAML that must load — volume two's
validate-then-swap, relocated to the threshold where the swap is a
commit). Each is a line or three of shell around a tool the fleet
already runs; together they close the accident classes chapters 1 and
2 could only warn about.

## The message contract

The second threshold reads the claim, and it is where chapter 2's
machine-checkable subset becomes machinery:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
cat > .git/hooks/commit-msg <<'PEOF'
#!/bin/sh
subject=$(head -1 "$1")
[ ${#subject} -le 72 ] || { echo "commit-msg: subject exceeds 72 chars" >&2; exit 1; }
grep -q "^Ledger-Op: " "$1" || { echo "commit-msg: missing Ledger-Op trailer" >&2; exit 1; }
PEOF
chmod +x .git/hooks/commit-msg
echo x > f; git add -A
git commit -qm "quick fix" 2>&1; echo "without trailer: $?"
git commit -qm "raise retry budget" -m "Ledger-Op: retry-budget:2026-08" && echo "with trailer: accepted"
```

```output
commit-msg: missing Ledger-Op trailer
without trailer: 1
with trailer: accepted
```

The hook receives the message file's path as its argument, reads and may
even rewrite it, and refuses what fails the contract — here the two
clauses a fleet can actually enforce: subject length (the summary-row
budget) and the presence of the provenance trailer that joins commits to
volume two's ledger. The boundary the register draws is between *form*
and *quality*: a hook verifies the trailer exists, the subject fits, the
message is not empty — it cannot verify the body answers "why", and
attempts to lint prose quality produce gates that refuse good messages
and admit hollow ones that game the pattern. Form is the machine's
threshold; quality is chapter 8's, where a human reads. Keeping each
gate to what it can actually judge is the same division of labor the
press's pipeline runs — mechanical adequacy at pass one, judgment at
review — and mixing them fails in the same direction at both scales.

## Policy that travels

The hooks above live in `.git/hooks/` — per-clone, unversioned, gone in
the next worktree — which is correct as a *security* default (a cloned
repository must not execute arbitrary scripts on arrival) and useless as
*fleet policy*. The bridge is one config key:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
mkdir -p hooks
cat > hooks/pre-commit <<'PEOF'
#!/bin/sh
echo "fleet hook ran" >&2
PEOF
chmod +x hooks/pre-commit
git config core.hooksPath hooks
git add -A; git commit -qm "policy travels with the repo" 2>&1 | head -1
git log --oneline
```

```output
fleet hook ran
e9283eb policy travels with the repo
```

The hooks directory is now *in* the repository — versioned, reviewed,
inherited by every clone — and one line of the open ritual
(`core.hooksPath hooks`, a git 2.9 key; a seat older than that ignores the
setting *silently* and runs ungated, which is precisely the drift the seat
audit below exists to catch) arms it per seat. That deliberate second step is
the security default preserved: policy travels automatically, *execution*
of policy remains each seat's explicit opt-in, made in the same ritual
that sets identity and pager discipline (chapter 1), and documented in
the repository so chapter 3's inheritance briefing finds it. The fleet
dividend compounds through everything this series builds: hook changes
get commits with claims and review like any policy change; a seat's
briefing can verify it runs the same gates as every other seat (`config
core.hooksPath` is a query); and the gate scripts themselves are written
to volume one's shot standards — bounded, deterministic, stderr for
reasons — because a hook is exactly a shot that other shots must survive.

## Refusals worth reading

A gate's stderr is its entire interface, and the difference between a
respected gate and a bypassed one is often nothing but the quality of its
refusals. The register's contract for a refusal message mirrors volume
one's contract for any failed shot's transcript — it must let the refused
operator act *without investigation*: name the check that failed, point
at the evidence (the file and line, the offending subject, the pattern
matched — the demo's marker hook prints the class; a production version
prints `f.txt:1` beside it), state the fix or where the fix is
documented, and, where a sanctioned bypass exists for its edge cases, say
so and say how — the audit hook this chapter's inheritance section
recounts did exactly that, and its self-aware refusal is what turned an
edge case into a two-minute resolution instead of an afternoon. The
anti-patterns are the transcript sins of the whole series: the silent
refusal (exit 1, no output — a calm face on a closed door); the
screaming refusal (three hundred lines of linter dump burying the one
actionable line — bound the output, summarize, point at the full log as
an artifact); and the moralizing refusal (a lecture where a file and
line were wanted — gates enforce, documentation persuades, and a hook
that confuses the two does neither well). One structural habit serves
all of it: hooks emit their refusals through a shared helper that
formats name, evidence, fix, and bypass-status uniformly, so every gate
in the fleet refuses in the same dialect and every seat learns to read
refusals once.

## Gates are shots: test them like shots

A gate that is wrong is worse than no gate — a false-refuser trains
bypassing, a false-admitter launders the very defects it advertises
catching — and the register's answer is the one it gives every predicate:
two-point calibration, mechanized. Chapter 4 verified bisect predicates
at a known-good and known-bad commit before trusting the hunt; hooks get
the identical treatment as *fixtures in the repository*: for each gate, a
pair of staged-state fixtures (one that must pass, one that must be
refused, each a tiny script that constructs the state in a scratch
worktree and runs the hook against it), executed by CI on every change
to the hooks directory (a path filter on the hooks dir is the trigger; the
fixtures run in seconds) — the gates gating themselves, with the
authoritative runner as their own second gate. Volume two's kill-testing
instinct extends the suite where a hook does more than read: a hook
that writes (commit-msg rewriting a message, a hook maintaining a
changelog) gets the interrupted-run test, because a half-rewritten
message file is the same corruption class as a half-written estate row.
And the suite's fixtures double as the gate's documentation-by-example:
the must-refuse fixture *is* the precise statement of what the fleet
has decided not to admit, reviewable in the same pull request as the
gate that enforces it. A fleet whose gates carry their own fixtures can
change policy with confidence and read policy from the tests — schema
and migration discipline, applied to the thresholds themselves.

## The bypass, and what client gates really are

Every client-side threshold has a documented door around it, and the
register teaches the door rather than pretending it away:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
cat > .git/hooks/pre-commit <<'PEOF'
#!/bin/sh
echo "gate would have run" >&2; exit 1
PEOF
chmod +x .git/hooks/pre-commit
echo x > f; git add -A
git commit -qm "blocked" 2>&1
echo "gated commit exit: $?"
git commit -qm "bypassed" --no-verify && echo "bypass: accepted"
```

```output
gate would have run
gated commit exit: 1
bypass: accepted
```

`--no-verify` steps past pre-commit and commit-msg unconditionally, and
its existence defines what client hooks *are*: self-discipline
infrastructure, not authority. Authority lives where bypass does not
reach — the server side, where `pre-receive` and `update` hooks (or the
forge's protected-branch rules, their managed descendants) judge every
push with no bypass flag in the protocol — and a fleet that needs a rule
*enforced* rather than *encouraged* puts it there, which is chapter 6's
configuration counsel generalized: the covenant's guarantees belong in
the layer no seat can decline. The register's ethics for the door
itself: bypass is an emergency instrument (the hook is broken, the fix
is the commit that repairs it, the incident cannot wait), every use is
ledgered in the estate with the reason, and a *pattern* of bypasses in
the fleet's history is a finding about the gate — too slow, too flaky,
wrongly scoped — routed to the gate's own review rather than absorbed
as culture. Gates earn compliance by deserving it; the bypass log is
where the fleet learns whether they do. The session-bound reading of
the same ethics: an unattended operator that meets a refusing gate
mid-task treats the refusal as volume one taught it to treat any
refusal — read, diagnose, fix or escalate — and *never* reaches for
the bypass merely because no human is present to ask; the absence of
supervision is precisely when self-imposed gates matter most, which
is the reason a press run by machines built its own gate before it
accepted its first manuscript.

## Surviving gates you did not write

The inheritance side arrives with sharper stakes here than anywhere in
this book, because hooks are *executable policy*: an inherited
repository's threshold scripts run with the operator's own authority, at
moments the operator triggers. The briefing obligations follow. Discover
(`config core.hooksPath`, the hooks directory listing) and *read* the
gates before the first commit — volume two's untrusted-file protocol,
applied to scripts that will run unbidden; a hook that fetches remote
content or writes outside the repository is a finding before it is a
gate. Expect the register's classic traps in other people's gates:
hooks that assume a terminal (volume one's isatty fork — a hook that
pages or prompts hangs the unattended commit exactly like any other
interactive assumption), hooks without timeouts, hooks that are slow
enough to train bypassing. And expect the *edge cases* the authors
never met, because this book's own production met one: a global
pre-push audit hook that errored on a freshly created repository — no
base ref to diff against — and blocked every initial push, with its own
error text offering the bypass as the sanctioned path. The protocol
that handled it generalizes: read the refusal (it named its own
confusion), confirm the hook's *intent* was inapplicable rather than
violated (an audit of changes cannot audit a repository with no
before), take the sanctioned bypass, and record the incident — after
which the durable fix (teach the hook about baseless repositories)
becomes a contribution to the gate rather than folklore about avoiding
it. Gates you did not write get the same reading as any inherited
constraint in this series: understood before obeyed, obeyed before
bypassed, and improved instead of quietly routed around. The defensive
wrapper for the whole class costs one line and volume one supplies it:
the operator's commits run with the hook chain under `timeout` at the
harness level where possible, and gates the fleet *writes* wrap their
own expensive steps in bounds internally — because a hook is a shot
running inside another shot, and unbounded nesting is how a one-second
commit becomes a mystery hang with two layers of silence to excavate.
The environmental note completes the survival kit: hooks inherit
git's process environment, not a login shell's — the stripped-env
lessons of both prior volumes apply, and a hook that works at one
seat and fails at another has usually lost a PATH entry or a variable
the author's shell exported invisibly, diagnosable in minutes by the
operator that read volume one and in afternoons by the one that did
not.

## The estate at the threshold

The hook points come in two temperaments, and the second completes a
join this trilogy has been preparing for two volumes. The `pre-*` hooks
refuse — they are gates, everything above. The `post-*` hooks
(`post-commit`, `post-merge`, `post-checkout`) run *after* the moment,
cannot refuse anything, and are therefore not gates at all but
*recorders* — the repository offering to narrate its own life to
whoever is listening. For this book's reader, the listener has a name:
a `post-commit` hook that appends the new entry's hash, subject, and
`Ledger-Op` trailer to volume two's estate makes the ledger-to-ledger
join automatic — every commit self-reports into the operational record,
and the estate's "what did run N commit?" query stops depending on any
session's diligence. The same wiring runs the gate direction with more
power than any text check: a `commit-msg` hook that does not merely
verify the `Ledger-Op` trailer's *format* but queries the estate for
the operation's *existence* — refusing commits that cite ledger
operations never opened — enforces intent-then-outcome across both
records at once, volume two's two-generals discipline with a mechanical
guarantor. The register's cautions keep the wiring sane: recorders must
be fast and unfailing (a post-commit that can error, errors *after*
the commit — it logs its own failures to the estate's dead-letter file
rather than confusing the seat); gates that consult the estate inherit
the estate's availability (the hook degrades to format-checking with a
warning when the database is unreachable, because a broken join must
not freeze the fleet); and both directions honor the boundary chapter 1
drew — the estate stays out of the shared repository even as the hooks
that write it travel in the hooks directory, configuration pointing
each seat at its own lineage's file. Wired so, the trilogy's records
close their loop: the shared ledger gates on the private one, the
private one transcribes the shared one, and an operator's whole
account — conduct, memory, and collaboration — audits as one system.

## Auditing the seats themselves

Gates judge entries; nothing yet judges the *seats* — and configuration
drift across a fleet is the quiet failure this whole chapter's
architecture rests on not happening. The open ritual (chapter 1) sets
each seat's identity, hooks path, pager discipline, and policy
switches; the audit question is whether it still holds everywhere, and
the register answers it as always: a standing query, not an assumption.
The fleet's expected configuration lives as a manifest beside the hooks
directory (policy code, versioned, reviewed), and a briefing-grade
check — `git config --list` filtered to the policy keys, compared
against the manifest — runs per seat at session open, reporting drift
the way volume two's briefing reports staleness: as a line that names
the seat, the key, and the divergence. The classic drifts it catches
are the ones that silently disarm chapters of this book: `hooksPath`
unset (a seat committing ungated), identity defaulted (chapter 1's
someone-else's-name accident), `rerere` off where the fleet assumed
shared behavior, the pager environment missing on a fresh host (the
hang, rediscovered). Config *cannot* be enforced client-side — a seat
is always sovereign over its own copy, which is the same truth the
bypass section told about hooks — so the audit's role is the honest
one: make drift visible within a session of its birth, route it to
the ritual that repairs it, and let the server-side gates remain the
floor beneath whatever a drifted seat manages to do meanwhile. Fleets
run on defaults verified, not defaults assumed; the seat audit is one
more place this series converts an assumption into a query.

## The repository's own health check

Volume two gave the estate a standing verification job; the repository —
equally a database, chapter 1 insisted — deserves the same custody, and
its instruments parallel one for one. The integrity audit is `git fsck`:
a full walk of the object store verifying every hash against its content
and every reference against reachability — the chain checking itself —
scheduled at estate-audit cadence for repositories a fleet depends on,
its findings (dangling objects are normal life per chapter 6's horizon;
*corrupt* objects are the alarm) triaged with the same severity split as
`integrity_check`'s. The maintenance layer is `git maintenance`: the
modern porcelain that schedules what folklore ran as ad-hoc `gc` —
object packing, reference packing, the commit-graph file whose absence
is why big repositories' chapter 3 queries crawl (`maintenance start`
registers the background schedule; fleets with their own scheduling run
`maintenance run --task=...` from volume one's timers instead, keeping
custody explicit). And the recovery insurance is chapter 6's own
teaching applied at repo scale: the fleet's shared remote is the
replica-of-record, every seat's clone is a working replica, and the
*bare metal* backup question reduces to whether the remote itself is
backed up — a hosting-layer concern the fleet verifies the way volume
two verified everything: not by trusting the vendor's page, but by the
periodic drill of cloning cold from the backup and running the briefing
against it. A repository that is fsck-clean, maintenance-scheduled, and
drill-restored is infrastructure; one that is merely "on the forge" is
a hope with an SLA — and the operator that keeps ledgers has no license
to keep them on hope.

## Three gates, one architecture

The chapter closes by placing its threshold in the full line of defense,
because misplacing checks across the line is how fleets get slow gates,
noisy CI, and exhausted reviewers at once. Client hooks judge what is
*instant and mechanical* — form, markers, secrets, the contracts a
second's work can verify — at the moment of composition, where the fix
costs least. Continuous integration judges what is *expensive and
mechanical* — the full build, the suite, the platform's authoritative
re-run of everything the client gates claimed (the press's own
architecture again: the author's local gate run is a courtesy; the CI
run is the record) — asynchronously, where minutes are affordable.
Review judges what is *judgment* — design, correctness the suite cannot
see, the quality of claims — and chapter 8 gives it the handoff it
deserves. Each layer trusts the ones before it and verifies anyway
(hooks lie when bypassed; CI is the check on that; review reads CI's
verdict rather than re-deriving it), which is volume two's trust ladder
built out of process instead of tables. The operator's contribution to
the architecture is the discipline this chapter mechanized: gates it
writes are fast, honest, and bounded; gates it inherits are read,
respected, and improved; and everything that can be judged by a machine
is judged before any human's attention — the fleet's scarcest resource,
and the subject of the final chapter — is spent. What reaches that
attention, and in what shape, is the last craft this book owes its
reader.
