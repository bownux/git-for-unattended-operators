# Chapter 4 — Diagnosis by Bisection

*Draft status: author draft; human verification pending. The bisection in this
chapter's central listing is a real unattended run against a planted regression;
outputs are its true transcript.*

## Where reading ends

Chapter 3 closed at the boundary every history reader eventually hits: the
ledger says what changed and what was claimed; only running the code says what
*worked*. The question that lives past that boundary is the oldest in
operations — *it worked before and it does not work now; which change broke
it?* — and the interactive tradition answers it with an expert's intuition:
skim the log, suspect the likely commits, check out a few, test by hand,
narrow by feel. The register cannot use intuition-shaped tools, and here that
poverty becomes wealth, because git ships the systematic answer as a built-in,
and the systematic answer *wants* to be run by a machine. Bisection is binary
search over history: pick the midpoint between a known-good and a known-bad
commit, test it, discard the half the result exonerates, repeat. Each probe
halves the suspect range, so the arithmetic is logarithmic and worth feeling
once: twenty suspect commits need at most five probes; a thousand need ten; a
year of a busy repository — ten thousand commits — falls in fourteen. The
interactive expert's skimmed suspicion competes with that arithmetic only on
lucky days, and the operator that internalizes it stops dreading wide suspect
ranges at all: doubling the range costs one probe.

Manual bisection (`bisect start`, then `bisect good`/`bisect bad` verdicts by
hand, checkout by checkout) is the teaching form, and the register skips
straight past it to the form built for unattended use — `git bisect run`,
which takes a *predicate command* and conducts the entire search alone: check
out midpoint, run predicate, read its exit status as the verdict, move, and
repeat until the culprit is cornered. Diagnosis collapses into a
predicate-writing problem — and predicate-writing is a discipline this
series' reader has been practicing since volume one taught that exit codes
are the channel machines speak. This chapter is the craft of that collapse.

## The whole hunt, unattended

The demonstration plants a regression the way real ones happen — buried
mid-history among unrelated changes, with two commits along the way that
cannot be tested at all (their "build" is broken, as mid-refactor commits'
builds often are) — then hands the hunt to the machine:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
for i in $(seq 1 20); do
  if [ $i -eq 13 ]; then printf "threshold = 50\nlimit = 9000\n" > app.conf
  elif [ $i -lt 13 ]; then printf "threshold = 50\nlimit = 100\n" > app.conf
  fi
  if [ $i -eq 6 ] || [ $i -eq 7 ]; then touch BROKEN_BUILD; else rm -f BROKEN_BUILD; fi
  echo "change $i" >> notes.txt; git add -A; git commit -qm "change $i"
done
cat > predicate.sh <<'PEOF'
#!/bin/sh
[ -e BROKEN_BUILD ] && exit 125          # untestable here: tell bisect to skip
grep -q "^limit = 100$" app.conf         # 0 = still good, 1 = regressed
PEOF
chmod +x predicate.sh
git bisect start HEAD HEAD~19 >/dev/null 2>&1
git bisect run ./predicate.sh >bisect.out 2>&1
grep -E "first bad commit" bisect.out | head -1
git log -1 --format="guilty entry: %h %s" "$(git rev-parse refs/bisect/bad)"
echo "probes spent: $(grep -cE "^# (good|bad|skip)" .git/BISECT_LOG) across 19 candidate commits"
```

```output
069375fa58dd242d49002a6fcf30dd5e70b1efc1 is the first bad commit
guilty entry: 069375f change 13
probes spent: 7 across 19 candidate commits
```

Change 13 — the commit that moved `limit` from 100 to 9000 — identified
exactly, unattended, in seven probes over nineteen candidates, *including*
navigating around the two untestable commits without human help. Read the
three moving parts the way the operator will reuse them. `bisect start HEAD
HEAD~19` declares the frame: bad here, good nineteen back — the two
assertions everything rests on, of which more below. The predicate is the
hunt's entire intelligence, and its contract is pure volume one: exit 0
declares this commit good, exit 1 through 127 declares it bad — *except*
125, the reserved status meaning "this commit cannot be judged; skip it",
which the predicate's first line spends on the broken-build marker. And the
wrap-up queries collect the verdict from where bisect leaves it: the
`refs/bisect/bad` ref names the culprit for machine consumption (no parsing
of the human transcript required), and the bisect log — itself a replayable
record, `git bisect log` emitting the whole session as commands — is the
hunt's ledger entry, ready for the estate. One footnote closes the frame:
`bisect reset` afterward returns the repository to where it stood, because
a bisect session leaves HEAD detached mid-history, and a session that
forgets the reset bequeaths its successor a repository lying about what it
was doing — volume two's unfinished-run inheritance, avoidable here by
making reset part of the ritual.

## Writing predicates: the whole craft

Everything rides on the predicate, and its craft is volume one's shot
discipline with a new consumer: not a transcript reader but the bisect
engine, probing dozens of times without supervision. Four properties
decide whether the hunt converges on truth.

*Correct polarity, verified first.* Before handing the predicate to `run`,
execute it once at the known-bad point and once at the known-good point
and confirm it says what the frame asserts — bad fails, good passes. The
failure mode this prevents is not subtle: a predicate inverted, or subtly
testing the wrong thing, does not err randomly — it conducts a flawless
binary search to a *confidently wrong* commit, and the operator inherits a
verdict with perfect form and no truth. Volume one's evidence-theater
detector ("what outcome would make this print differently?") applies to
predicates verbatim, and the two-point calibration is its mechanical form.

*Hermetic and bounded.* The predicate builds what it tests from the
checked-out tree alone, touches no shared state (a scratch directory per
probe — `mktemp` discipline — because probes run in sequence in one
working tree), and bounds itself in time (`timeout` from volume one; a
hung probe is a hung hunt) and in output (bisect keeps the transcript;
a chatty predicate buries the verdicts). Where the build is expensive,
the predicate caches by commit hash — content-addressing from chapter 1,
serving diagnosis.

*Deterministic — or made honest about not being.* A flaky predicate is
bisect poison: one wrong verdict sends the search into the wrong half,
and the final answer is an innocent commit — worse than no answer,
because it arrives with bisect's authority. The mitigations, in order:
fix the flake (best); run the probe N times inside the predicate and
verdict on the consensus (the vote pattern, volume two's verification
instincts); or, when flakiness cannot be tamed, drop to manual bisection
with human judgment on the ambiguous probes — the one place this chapter
concedes the register.

*Skip honestly, and read skips honestly.* Exit 125 is for genuinely
untestable states — broken builds, missing fixtures — and the earlier run
showed it working when the untestable zone lies away from the boundary.
The honest caveat from this book's own testing: when the culprit hides
*inside or adjacent to* a skipped stretch, bisect ends not with a verdict
but with a candidate set — "the first bad commit could be any of these" —
and that answer is correct behavior, not failure: the ledger contains
commits that cannot be judged, and the machine has narrowed truth to the
smallest set the evidence permits. The follow-up is manual: make one
candidate testable (patch the broken build in the working tree, test,
revert the patch) or judge by reading. An operator that reports the
candidate set *as* a candidate set, rather than picking one and calling
it the verdict, is practicing volume one's claims-sized-to-evidence at
the exact moment it is hardest.

## Framing the hunt

The frame — the good and bad endpoints — is the operator's other
contribution, and its craft is short but consequential. The *bad* end is
usually free: HEAD, or the deployment that alarmed. The *good* end wants
the nearest trustworthy anchor, and chapter 3 already built the finding
tools: the last release tag that verifiably worked (`describe` orients),
a date fence (`log -1 --before='2 weeks ago' --format=%H` for "whatever
we ran then"), or the estate's own records — volume two's run registry
saying which commit the last green deployment carried, joined by the
hash that chapter 2's trailers put in reach. Two temptations to resist:
framing narrow to save probes (the arithmetic says wide is cheap; a
wrongly-asserted good endpoint, believed because it saved three probes,
poisons the hunt the same way an inverted predicate does — when in
doubt, widen to certainty); and bisecting with a dirty working tree
(bisect refuses or entangles; stash or commit first — chapter 2's
cadence means there is usually nothing loose to entangle).

What to bisect generalizes past "the tests broke", and the register's
operators should carry the wider list: performance regressions (the
predicate measures and compares against a threshold — volume one's
instrumented probes, promoted to verdicts); configuration drift (this
chapter's demo *was* one); behavioral changes with no failing test yet
(the predicate is the reproduction script of the bug report); even
documentation rot (the predicate greps for the promise that vanished).
Anything a script can judge, history can be searched for — which is the
chapter's thesis run backward.

## A second hunt, in prose

The demo's predicate was a grep; the instrument's reach shows better in the
hunt operators actually dread — *it got slow*. The service's p95 latency
doubled somewhere in six weeks of commits; no test fails; nothing in the
log confesses. The predicate for this hunt is a measurement with a
verdict: start the service from the checked-out tree, warm it, fire the
benchmark volume one taught (bounded requests, `curl`'s timing variables
or the harness's equivalent), compare the measured figure against a
threshold, exit accordingly. Two craft points carry the whole case. The
threshold is *chosen from the endpoints*: measure at known-good and
known-bad first — 80 ms and 160 ms, say — and place the bar between them
at the point that separates the populations (120 ms), not at the spec's
wishful target; a threshold below the good end's natural variance
convicts innocents, and the two-point calibration that verified polarity
doubles as the variance check (run each endpoint thrice; if the spreads
overlap the bar, the predicate votes N runs and verdicts on the median —
determinism bought with repetition). And the measurement is *hermetic* in
the register's fullest sense: same machine, same load conditions, warmup
discarded, because a bisection whose probe conditions drift mid-hunt is
measuring the afternoon, not the commits. Framed at the last green
deploy's hash — read from volume two's run registry, which has been
recording exactly this anchor since its chapter 4 — the hunt runs
unattended through sixty commits in six probes, and the guilty entry
turns out to be the innocent-looking cache-key widening nobody suspected.
The moral is the chapter's thesis at full strength: intuition had no
suspect; arithmetic did not need one.

## Merge-heavy history: hunting at the right altitude

Real shared history is not the demo's clean line — it braids, and
bisection's default walks *every* commit, including the interior commits
of merged branches. That default is sometimes exactly wrong. In a
repository that integrates by merge (the PR shape chapter 8 assumes),
main's own history is a sequence of integration points, and the question
the incident actually asks is usually "*which merge* broke main?" — the
altitude at which the remedy (revert the merge, re-open the PR) also
operates. The instrument has a switch for the altitude:
`git bisect start --first-parent` walks only the first-parent chain —
main's own spine — treating each merged branch as one opaque step, which
both matches the question and slashes the candidate count (a busy main's
spine is dozens of merges where its full graph is thousands of commits).
The full-graph default earns its keep afterward, if the convicted merge
is large: a second, interior bisection framed *inside* the guilty branch
(good at its fork point, bad at its tip) descends from the merge verdict
to the individual entry. Two altitudes, two frames, same machinery —
and the operator that asks at merge altitude first is aligning the hunt
with how the ledger was actually written, which is chapter 2's shaping
discipline collecting one more dividend.

## The probe budget

Bisection's economics deserve one honest table-in-prose, because "it's
logarithmic" hides the term that dominates practice: the predicate's own
cost. Probes number log₂ of the frame — five for twenty commits,
fourteen for ten thousand — but each probe pays the full price of
checkout plus build plus test, and a twenty-minute build makes fourteen
probes an overnight affair. The levers, in the order the register pulls
them: cheapen the predicate (test the narrowest reproduction, not the
suite; build only the implicated component — the pathspec discipline
applied to compilation); cache by content (chapter 1's hashes mean a
probe's build outputs can be keyed by tree hash and reused when
bisection revisits nearby states — real hunts revisit more than
intuition expects); narrow the frame *honestly* (a trustworthy newer
good-anchor from the registry saves probes without risk; a guessed one
poisons the hunt — the earlier warning, restated as economics: one probe
costs minutes, a wrong frame costs the whole hunt); and parallelize only
with care (bisect itself is inherently sequential — each verdict decides
the next probe — but the *endpoints'* calibration runs and a suspected
handful of spot-checks can run concurrently in chapter 5's worktrees
before the formal hunt frames itself). And when the arithmetic still
lands the hunt at hours: that is what unattended means — the operator
dispatches the run, volume one's monitoring patterns watch it, and the
verdict is waiting in `refs/bisect/bad` when the next session opens.
The interactive expert cannot skim while asleep. The register can.

## Hunting forward: old and new

Bisection's vocabulary betrays its usual errand — good, bad, a
regression assumed — and hides its generality: the machinery finds *any*
boundary where a testable property flips, in either direction. The
built-in generalization is terms: `git bisect start --term-old=absent
--term-new=present` renames the poles, and the hunt now answers
questions the good/bad frame contorts: when did this behavior *appear*
(hunting a feature's birth, or an unwanted side effect's — "bad" would
be backward); when did this file's format change; when did performance
*improve* (finding the optimization worth backporting — the happy
hunt, and the terms keep the predicate's polarity readable). The
register's interest is partly cognitive hygiene: chapter 4's inverted-
predicate accident breeds precisely in frames where "good" must mean
"the thing I am hunting is present", and self-chosen terms
(`--term-new=fixed`, hunting the commit that silently fixed a bug
nobody claimed — a real genre: the fix worth understanding and
porting) let the predicate read as the question reads. Mechanically
nothing changes — same halving, same 125, same `run` — which is the
point: the instrument was never a regression tool; it is a boundary
finder over any property a shot can test, and the operator that
internalizes the general form reaches for it in half the
investigations where the specific form never came to mind.

## When the hunt is the wrong hunt

Three shapes of trouble wear a regression's face and defeat bisection from
outside it, and the operator's protocol names them before probes are
spent. The world moved: if the breakage came from data, environment, or a
dependency beyond the tree, every commit will test bad and the hunt
degenerates — which is why the calibration run at the *good* endpoint is
the hunt's true first probe: a known-good commit that now fails convicts
the world, not the history, and redirects the investigation to volume
one's territory (what changed on the machine) and volume two's (what the
estate recorded changing). Two culprits interacted: bisection finds *a*
boundary — the first commit where the predicate flips — and when the
symptom needs two changes to manifest, that boundary names only the
later accomplice; a verdict that survives the four-question read but
cannot explain the mechanism is the cue to re-frame (bisect again with
the convicted change held applied, hunting its partner). And the bug that
was always there: a hunt that cannot find a good endpoint because none
exists is not a regression hunt at all — "since forever" is a different
genre of investigation, and recognizing it after two widenings of the
frame, rather than after twenty, is the probe budget's cheapest saving.
All three are volume one's oldest lesson in new clothes: the instrument
is sound; the question must still be the right question.

## Hunts that outlive their sessions

A long hunt — the twenty-minute build times fourteen probes — will not
fit one session, and the instrument was built for exactly this reader
without knowing it: `git bisect log` emits the session's every assertion
as a replayable script, and `git bisect replay` reconstructs the hunt
from it — the interrupted bisection resumed by a successor that shares
nothing with its predecessor but the file. The session-bound protocol
writes itself from the parts: each probe's verdict is appended to the
saved log (an artifact in volume two's index, beside the predicate
script itself — the two files that *are* the hunt's state); the session
that runs out of budget ledgers the hunt as an open intent with the
log's path; and the successor's briefing surfaces it, replays, and
continues from probe eight as though the lineage had never blinked.
The same artifacts serve the fleet horizontally: a hunt's log and
predicate posted to the proposal thread let a colleague — human or
machine — replay the identical hunt to verify the conviction (chapter
7's two-point calibration, socialized), which converts "my bisection
says" from testimony into the reproducible claim this series requires
evidence to be. Nothing here is new machinery; it is the trilogy's
resumability doctrine — legible stages, durable state, briefings that
surface unfinished business — discovering that git had already built
its half.

## The control experiment, and working at the scene

Two post-verdict practices convert the conviction from probable to
proven and the fix from disruptive to parallel. The control: before
building anything on the verdict, run the predicate at the guilty
commit's *parent* — the one probe bisection's own economy usually
already spent, verified now deliberately — because "bad here, good one
step before" is the conviction's controlled experiment, and a parent
that also fails means the frame or predicate lied somewhere and the
verdict is an artifact (the inverted-predicate hazard, catchable one
last time for the price of one probe). The scene of the crime then
becomes a workplace without disturbing anything: chapter 5's worktrees
open the guilty commit and its parent side by side (`worktree add
../guilty <hash>`, detached — chapter 1's protocol for detached work
applies), where the diff between them is read at chapter 3's
resolutions, the failing behavior is reproduced live in one tree and
its absence confirmed in the other, and the fix is developed against
the *modern* branch in a third tree while both evidence trees stand —
diagnosis, evidence, and remedy proceeding in parallel with no
checkout thrash and no risk to anyone's working state. The pattern is
the trilogy's instruments composing exactly as designed: the hunt
found the moment, the worktrees hold the moment open for inspection,
and the ledger receives the case — which is where every hunt in this
chapter has been heading.

## After the verdict

Bisect ends where accountability begins, and the aftermath is assembled
from disciplines already on the shelf. The guilty entry gets chapter 2's
four-question read — claim against evidence, shape, provenance, absence
— because "which commit" was never the real question; *which change,
wanted by whom, for what reason* is, and a well-shaped ledger answers in
one read while a monolith (chapter 2's warning, now at collection time)
answers only after intra-commit archaeology. The remedy decision — fix
forward or revert — belongs to chapter 6's reversibility treatment, with
the register's default inherited from volume one's ladder: the revert,
being the reversible move, buys time under incident pressure that
fix-forward gambles. And the whole case — frame, predicate, probes,
verdict, remedy — lands in the estate as one operation: the bisect log
as artifact, the guilty hash in the outcome column, the journal entry
written for the future searcher who will someday hunt something similar
(volume two's promotion discipline; a lineage's second bisection of the
same subsystem should start from its first). Diagnosis, in this
register, is not an art the operator performs. It is a predicate the
operator writes, a frame the operator asserts honestly, and a machine
that does the rest — which frees the operator's judgment for the two
places no machine reaches: whether the predicate tests the truth, and
what to do about the commit it convicts.
