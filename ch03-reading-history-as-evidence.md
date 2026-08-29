# Chapter 3 — Reading History as Evidence

*Draft status: author draft; human verification pending. Outputs are real
transcripts from scratch repositories with deliberately planted histories.*

## The ledger answers questions

Volume two's refrain — state you cannot query is barely state at all — has been
aimed at this chapter since chapter 1 called the repository a ledger. A
repository the operator only writes to is a filing cabinet; the craft dividend
arrives when history becomes the *first place questions go*: when did this
setting change, who touched this line and under what claim, what happened to
this file before it wore this name, what does that branch hold that this one
lacks. Interactive developers answer such questions by scrolling — `log` in a
pager, eyes hunting — which is exactly the reading mode volume one retired.
The register's answer is that `git log` is a query language wearing a
pager's clothing: date bounds, path scopes, content predicates, line
tracers, and set arithmetic, every one of them composable into bounded
one-shot reads. This chapter is that query language, taught the way volume
one taught `journalctl` — bounds first, machine formats second, and the
expensive read spent only where cheap reads point.

The bounding discipline transfers without modification, because unbounded
`log` is unbounded `journalctl` with better compression. Every history
query in this book carries at least one of: a count (`-n 20`), a date fence
(`--since`, `--until` — taking the same English the journal took), a path
scope (`-- path/`, which also collapses noise better than any filter
applied after), or a range (the set arithmetic at this chapter's end). And
every query meant for parsing states its format explicitly — `--format`
with field tokens (`%h %s`, `%an`, `%aI` for strict ISO dates — chapter 3
of volume two smiling from the wings) rather than the default
human-shaped layout, with `-z` termination available where filenames may
appear. The reflex pair, then: *bound the question, declare the shape.*
Everything below assumes both.

## Formats built for the next command

Declaring the shape deserves its demonstration, because the default log
layout — hash line, author line, date line, blank line, indented message —
is a human display in exactly volume one's sense: pleasant to eyes,
hostile to `awk`. The `--format` tokens compose the machine face:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
for s in "initial config" "add request timeout" "drop timeout: upstream handles it now"; do echo "$s" >> log.txt; git add -A; git commit -qm "$s"; done
git log --format="%h|%aI|%an|%s"
```

```output
fff6b5d|2026-08-28T13:57:35-07:00|operator|drop timeout: upstream handles it now
87a9807|2026-08-28T13:57:35-07:00|operator|add request timeout
ec7ad7b|2026-08-28T13:57:35-07:00|operator|initial config
```

One row per commit, fields chosen and delimited by the operator, ready
for the cut and join and comparison volume one built its pipelines from.
The tokens worth memorizing are few: `%h`/`%H` short and full hash, `%s`
subject, `%b` body, `%an`/`%ae` author name and mail, `%aI` the author
date in strict ISO-8601 — the format volume two's estate speaks natively,
so history rows and ledger rows join on timestamps without conversion —
and `%(trailers:key=Ledger-Op)` lifting a named trailer straight into the
row, which turns chapter 2's provenance convention into a queryable column
(the selective trailer placeholders arrived in git 2.22 — on older seats,
chapter 2's `interpret-trailers` pipeline is the portable spelling of the
same query). Delimiter choice follows the usual paranoia — subjects may contain pipes;
`%x00` emits NUL for the fully hostile case, consumed as
`--format='%h%x00%s' | awk -F'\0' …` — and the same
`--format` vocabulary drives `show`, `branch --format`, and `for-each-ref`
— one shape language across every reading tool. The rule it all serves is
volume one's porcelain rule with a local sharpening: *the default log
layout is not an interface; the format string is.*

## The pickaxe: asking about content

The question histories get asked most — *when did this change, and by which
commit?* — has a dedicated instrument almost no interactive tutorial
teaches, because scrolling hides its necessity. The pickaxe, `-S`, filters
history to the commits where a given string's *occurrence count changed* —
the commits that introduced or removed it:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
printf "retries = 3\n" > app.conf; git add -A; git commit -qm "initial config"
printf "retries = 3\ntimeout = 30\n" > app.conf; git add -A; git commit -qm "add request timeout"
echo "unrelated" > notes.md; git add -A; git commit -qm "add notes"
printf "retries = 3\n" > app.conf; git add -A; git commit -qm "drop timeout: upstream handles it now"
git log --oneline -S "timeout = 30" -- app.conf
```

```output
94c6b39 drop timeout: upstream handles it now
8b92d86 add request timeout
```

Four commits of history, and the pickaxe returned exactly the two that
matter to the question "what is the story of `timeout = 30`?" — its birth
and its death, each carrying its claim (and the death's message answering
the *why* a bare diff never could — chapter 2's message discipline, paying
the reader back). The unrelated middle commit never surfaces. For an
operator diagnosing configuration drift — volume one's "it worked
yesterday" — this is the opening query: the setting's value is wrong *now*;
`-S` with the old value finds the commit that removed it, `-S` with the new
value finds the commit that planted it, and both arrive with authors,
dates, and reasons attached. The variant `-G` takes a regex and matches
commits whose diff *mentions* the pattern (not just occurrence-count
changes — it also catches lines that moved or changed around the pattern),
the broader net when the exact string is uncertain; the discipline of
preferring `-S` first is the register's exactness preference from volume
two's search chapter, transplanted.

## Line archaeology

Below file granularity lives the question blame half-answers and `-L`
answers properly: *what is the story of this line?*

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
printf "alpha\nbeta\ngamma\n" > f.txt; git add -A; git commit -qm "three lines"
printf "alpha\nBETA\ngamma\n" > f.txt; git add -A; git commit -qm "capitalize beta"
printf "alpha\nBETA!\ngamma\n" > f.txt; git add -A; git commit -qm "emphasize beta"
git log -L 2,2:f.txt --oneline --no-patch
```

```output
50be31b emphasize beta
3b59905 capitalize beta
5d1d48b three lines
```

Three commits, and line two's complete biography: created, capitalized,
emphasized — every commit that ever touched the line, in order, with
claims. `-L` takes line ranges (`-L 10,25:file`) and even function names
(`-L :funcname:file`, using language-aware heuristics), and it follows the
line through edits above it that shift its number — the bookkeeping that
makes manual diff-walking miserable, done by the engine. Its sibling
question — *where did this file come from?* — meets rename tracking:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo content > old-name.md; git add -A; git commit -qm "create doc"
echo more >> old-name.md; git add -A; git commit -qm "extend doc"
git mv old-name.md new-name.md; git commit -qm "rename doc"
echo "without --follow: $(git log --oneline -- new-name.md | wc -l) commits"
echo "with --follow:    $(git log --oneline --follow -- new-name.md | wc -l) commits"
```

```output
without --follow: 1 commits
with --follow:    3 commits
```

The default path query stops at the rename — one commit, a file
apparently born yesterday — while `--follow` walks through it to the true
origin. The trap shape is volume one's calm-face family: the unfollowed
query *succeeds*, returns plausible history, and silently amputates
everything before the rename; nothing warns. The habit: any history
question about a file older than its current name — and the operator
often cannot know — asks with `--follow`, and any *surprisingly short*
file history is re-asked with `--follow` before being believed.

## Blame, read correctly

`git blame` — every line annotated with the commit that last touched it —
is history's most famous query and its most misread. The misreading is in
the name: blame answers *last touch*, not *authorship of behavior*. The
line that broke production may be "blamed" on the reformatting commit
that re-indented it, the rename that moved its file, or the mechanical
sweep that changed a parameter name across forty files — while the mind
that wrote the logic lives three commits deeper. The register reads blame
as a *starting pointer*, never a verdict, and drives it with the flags
that strip mechanical noise: `-w` ignores whitespace-only touches; `-C`
traces lines copied or moved across files to their true origin; and
`--ignore-rev` (or a committed `blame.ignoreRevsFile` listing the
project's known reformatting commits — policy code, chapter 2's ignore
discipline for attribution) excludes named noise commits from
consideration entirely. When blame's answer survives those flags, the
next hop is `-L` on the implicated lines for the full biography, and
*then* the four-question read of the guilty entry. Attribution earned
that way holds up in the incident review; attribution read off bare
blame output regularly indicts the janitor.

## Ranges: history as sets

Multi-branch questions — what is on that branch, what will this merge
bring, how far have we diverged — are set questions, and git's range
syntax is set notation compact enough to misread, so the register learns
it once, precisely:

```bash
mkdir work && cd work
git init -q -b main; git config user.email op@example.invalid; git config user.name operator
echo base > f; git add -A; git commit -qm base
git checkout -qb feature; echo feat > g; git add -A; git commit -qm "feature work"
git checkout -q main; echo mainline > h; git add -A; git commit -qm "mainline work"
echo "feature has, main lacks (main..feature): $(git rev-list --count main..feature)"
echo "main has, feature lacks (feature..main): $(git rev-list --count feature..main)"
echo "either side since fork  (main...feature): $(git rev-list --count main...feature)"
```

```output
feature has, main lacks (main..feature): 1
main has, feature lacks (feature..main): 1
either side since fork  (main...feature): 2
```

Two dots, `A..B`: commits reachable from B and not from A — "what does B
have that A lacks", direction mattering, the workhorse for "what would
this merge bring" (`main..feature`) and "what am I behind" (`HEAD..origin/
main`). Three dots, `A...B`: the symmetric difference — everything on
either side since the fork point, the divergence overview. `rev-list
--count` turns any range into arithmetic (the estate's counters, for
history), and the ranges feed every history command uniformly: `log
main..feature` to read the incoming claims, `diff main..feature` to read
the incoming content. One asymmetry demands memorization because it
inverts the pattern: for `diff` — and only for diff — the three-dot form
does *not* mean symmetric difference; `diff A...B` shows B's changes
since the *merge-base* (the fork point), which is almost always what
review wants and almost never what the log-trained intuition expects.
The pair to internalize: *log likes two dots for direction, diff likes
three dots for review* — and an operator unsure at composition time
spends one `merge-base` query to see the fork point explicitly rather
than trusting punctuation it half-remembers.

## Two clocks per entry

Chapter 1 introduced the author/committer pair for identity; the pair
of *dates* matters to queries enough for its own caution. The author
date marks when the change was made; the committer date, when this
object entered this history — and chapter 6's instruments split them
routinely: a rebase rewrites committer dates wholesale while preserving
author dates, a cherry-pick likewise, so a branch reshaped Tuesday from
work written in May carries May's `%aI` and Tuesday's `%cI` on every
entry. The queries care because they default differently than intuition
expects: `log --since` and friends filter on *commit* date, so "what
was written in May" asked naively of reshaped history answers "nothing"
— the work is there, wearing Tuesday's commit dates — while
`--author-date-order` and explicit `%aI` formats reach the other clock.
The register's rule assigns each clock its questions: process
archaeology (when did this land, what entered last week, release
windows) reads committer dates, which is what the defaults serve;
provenance archaeology (when was this actually written, what was
concurrent with what) reads author dates, explicitly. And ledger
entries that join history to the estate record *both* when the
distinction could matter — one more two-line habit that costs nothing
on the day it is formed and settles an argument on the day it is
needed.

## The sweep, composed

The instruments combine into the register's standard multi-angle hunt, and
one worked narrative fixes the composition better than rules. The incident:
a service that retried politely last month now hammers its upstream;
somebody changed retry behavior; find the change and its reasoning. The
sweep opens cheap and wide: `log --oneline --since='6 weeks' -- config/`
— fifteen entries, subjects scanned, nothing obviously guilty (a subject
scan is a claims scan; chapter 2's discipline decides whether it can be
trusted, and here it cannot, since two subjects say "update settings").
Second angle, content predicate: `log -S 'backoff' --since='6 weeks'` —
two hits, one introducing `backoff = exponential`, one — the later —
removing it inside one of those "update settings" monoliths. Third angle,
scope the damage: `show --stat` on the guilty entry (eleven files — the
monolith buried the behavior change among renames), then `show <hash> --
config/retry.conf` for the one diff that matters. Fourth angle, the
biography: `-L` on the changed stanza confirms the exponential line lived
eight months and died without a stated reason — the body says nothing;
the claim-versus-evidence gap of chapter 2, met in the wild. Total cost:
four bounded reads, no scrolling, and the output of the sweep is not just
the culprit commit but the *case file* — hashes, dates, authorship, the
absence of justification — that volume one's handoff format and chapter
8's review response both consume directly. The general shape (wide cheap
scan → content predicate → scope → biography) transfers to every history
hunt; only the predicates change.

## Reading diffs at the right resolution

The diff itself — this chapter's most-consumed output — has resolution
controls the register should drive deliberately rather than accept at
line default. `--stat` first, always (the silhouette; chapter 2's
routine). For prose, config, and anything where a line is a paragraph,
`--word-diff` collapses the misleading full-line churn into the words
that changed — the difference between "this line changed" and "this
*value* changed", which for the config archaeology this book keeps
returning to is the whole question. For refactors, `--color-moved`
(with `--color-moved-ws=allow-indentation-change` for the re-indent
case) distinguishes *moved* code from added-and-removed code — the
reading that turns a terrifying 400-line diff into "one block moved,
three lines actually new", and the reviewer's honest answer to
mechanical changes chapter 8 will flag. `-w` ignores whitespace
outright where formatting noise drowns signal. And the same flags feed
the pipeline forms — the capture-mode operator reads `--word-diff=porcelain`
when parsing, per the porcelain rule. Resolution, like bounding, is
part of composing the read: the default line diff answers "what
changed" at one altitude, and an operator that never changes altitude
is doing archaeology with one lens — workable, and permanently slower
than the machine offering the zoom.

## Annotating after the fact: notes

One reading-adjacent instrument completes the evidence toolkit because
it solves a problem the append-only design otherwise leaves sharp: how
to attach information to a commit *after* it exists — the benchmark
result measured post-merge, the incident that later implicated it, the
"superseded by" pointer — without rewriting anything. `git notes`
maintains exactly this: a parallel, attachable annotation per object,
displayed alongside the commit in `log` but stored outside it, so the
hash chain stands unmodified while the fleet's afterknowledge
accumulates. The register's uses are the estate's margins made public:
`notes add -m 'bench: p95 82ms (run 4411)' <hash>` binds a measurement
to the exact content it measured; a notes ref per concern (`--ref
perf`, `--ref incidents`) keeps annotation streams separable; and
because notes are refs, they push and fetch deliberately (not by
default — a fleet that adopts them wires the sync into its open
ritual). The honest bounds mirror their design: notes are *mutable* —
that is their purpose — so they carry the estate's provenance
discipline (who noted, when, from what evidence, inside the note's
text) precisely because the chain does not carry it for them; and
anything whose integrity matters as much as its content belongs in a
signed tag or the estate proper, not a note. Rightly bounded, notes
answer the archaeologist's recurring wish — *the ledger should have
known this* — with the append-only system's own grammar: the entry
stands; the knowledge attaches beside it.

## Counting as evidence

Between reading entries and naming moments sits the aggregate layer —
history as statistics — and the register uses it the way volume two used
the run registry: calibration, not curiosity. `rev-list --count` has
appeared already as range arithmetic; its siblings answer standing
questions one number at a time. Freshness: `log -1 --format=%aI -- path/`
dates a subsystem's last real change, and a "stable" module untouched for
two years reads very differently from one changed weekly — staleness
pricing for code, volume two's `recorded_at` discipline applied to the
shared ledger. Churn: `log --since='3 months' --oneline -- path/ | wc -l`
ranks where change actually concentrates, which is where review attention,
test investment, and — chapter 4's interest — bug probability concentrate
too. People: `shortlog -sn --since='3 months' -- path/` names who really
owns a subsystem *now*, as against the archaeology bare blame implies.
None of these numbers proves anything alone — the honest-limits section
below applies to aggregates doubly, since curated history curates its
statistics — but as *priors* for where to look, whom to ask, and how
hard to verify, they are one-line queries that replace folklore with
arithmetic. An operator planning work in an unfamiliar repository spends
five such counts before its first edit; the counts are the briefing's
quantitative half. And like the registry's statistics, they compound
across sessions when recorded: a lineage that logs its pre-work counts
into the estate can later ask which priors predicted trouble and which
merely felt predictive — the operator calibrating its own calibration,
which volume two argued is the only kind that survives contact with
enough incidents to matter.

## Named moments

Hashes address history precisely and mean nothing; the ledger's readable
anchors are tags, and the reading operator meets them constantly enough to
know their two species apart. A lightweight tag is a bare name pointing at
a commit — a sticky note, no provenance, fine for private bookmarks. An
annotated tag is an *object*: it carries its own tagger identity, date,
and message, hashed and chained like everything else — a ledger entry
whose claim is "this commit is a named moment", which is why releases,
review baselines, and anything another party will reference use annotated
tags exclusively (this press's own pipeline tags each reviewed version of
a manuscript `v1`, `v2` — annotated moments in exactly this sense). The
reading queries: `tag -l 'v*' --format` lists names with the same token
language as everything else; `describe` inverts the lookup — given any
commit, it answers *where is this relative to the named moments*
(`v2-14-gf29483c`: fourteen commits past v2), the one-line orientation
that turns an arbitrary hash into a position humans can discuss; and a
tag's own claim is read with `show <tag> --no-patch`, tagger and message
included. The operator's writing rule mirrors the reading: moments worth
naming are named with `-a` and a message that says what the moment *is*
(volume two's reason column, again), because a bare `v2` whose meaning
lives in somebody's memory is the midden's naming scheme, reborn at the
ledger's front door. And because tags are the anchors other parties build
on, they inherit the publication boundary early: a pushed tag is a
promise; re-pointing one is chapter 6's sin in its most disruptive form,
since consumers cache tag meanings precisely because they are supposed
never to move.

## What history cannot tell you

The chapter closes on the ledger's honest limits, because evidence
misread as more than it is corrupts better decisions than ignorance does.
History records *committed outcomes*: it holds no trace of the approaches
tried and abandoned before the entry (the working tree's churn is
invisible), and — by this book's own chapter 2 counsel — the private
reshaping window means published history is a *curated* account of
process, deliberately cleaner than the work it records. That is a feature
for readers and a caveat for forensics: "the fix took one clean commit"
describes the ledger, not the afternoon. Second, the dates are
declarations, not measurements. Author and committer timestamps are
values the committing process asserts — settable by environment,
inherited oddly through rebases and cherry-picks — and the hash chain,
for all its tamper-evidence, proves only *relationships between contents*,
never wall-clock truth; an operator that needs trustworthy timing keeps
it in volume two's estate, whose clock discipline it controls, joined to
commits by hash. Third, blame and log answer about lines and files, not
*behavior*: the commit that broke the system may be the innocent-looking
enabler three weeks before the symptom, and no textual query proves
causation. Which is precisely the boundary where reading ends and
experiment begins: history as evidence says *what changed and what was
claimed*; only running the code says *what worked*. The next chapter
makes history runnable.

## The inheritance briefing

Chapter 1 promised that a fresh clone gets a briefing before it gets
work; this chapter can now write it. The queries, each bounded, each a
line or two, composing the repository's introduction shot: identity and
position — `remote -v`, current branch, `log -1 --format` for HEAD's
claim and date; the shape of recent history — `log --oneline -15` read as
a ledger-quality sample (are these single truths with real claims, or
"updates"? — the answer calibrates how much to trust every other query);
the cast — `shortlog -sn --since='3 months'` for who actually works here;
the live topology — `branch -a --format` with upstream tracking, plus
`main...origin/main` counts for local drift; the conventions — does
`log --format=%B -5` show trailers, does the tree carry the policy files
(ignore, attributes, hooks documentation) chapter 7 will look for; and
the standing risks — `status --porcelain` for inherited dirt, `stash
list` for a predecessor's abandoned intentions. Ten queries, one
transcript page, and the operator knows what it holds, who it works
beside, and which of this book's disciplines the project already
practices — the estate briefing of volume two, executed against the
ledger everyone shares. What the briefing cannot say is whether the
inherited history *works* — whether HEAD builds, whether the tests pass,
which commit broke what. Those are questions history answers only when
interrogated experimentally, and the next chapter automates the
interrogation.
