# Probes

Concrete measurements that produced findings. Adapt the tooling; the questions transfer.

Run everything from the repository root. Prefer counting over reading: a ratio is arguable, an impression is not.

**The commands assume a JavaScript, Prisma and git stack because that is where they were developed. The questions are stack-independent.** Translate the command, keep the question. Where a language or ORM has no equivalent of a probe, say so in the limits section rather than skipping it silently.

## Scale, first

Everything else is calibrated against this, and the numbers are often themselves the finding.

```bash
find . -path ./node_modules -prune -o -name '*.ts*' -print | wc -l
git rev-list --count HEAD
git log --format='%ad' --date=short | sort -u | wc -l          # active days
git log --format='%ad' --date=short | sort | uniq -c | sort -rn | head   # peak velocity
git show --stat --format="" $(git rev-list --max-parents=0 HEAD) | tail -1   # size of commit one
```

A very large initial commit means the bulk of the system arrived without review history. Say so.

## How much must be held in view

Produce this as a table, not a remark. It is the evidence for the inheritance section, and a number people grasp immediately.

```bash
for d in "the data model:prisma/schema.prisma" "business logic:lib" \
         "screens and actions:app" "interface components:components"; do
  label=${d%%:*}; path=${d##*:}
  lines=$(find $path -type f \( -name '*.ts' -o -name '*.tsx' -o -name '*.prisma' \) \
          -not -path '*/node_modules/*' 2>/dev/null | xargs cat 2>/dev/null | wc -l)
  bytes=$(find $path -type f \( -name '*.ts' -o -name '*.tsx' -o -name '*.prisma' \) \
          -not -path '*/node_modules/*' 2>/dev/null | xargs cat 2>/dev/null | wc -c)
  printf "%-24s %8s lines  ~%6.0fk tokens\n" "$label" "$lines" "$(echo "$bytes/3600" | bc -l)"
done
```

Divide bytes by roughly 3.6 for a token estimate. **The row that matters is the data model**, because deciding whether a change is safe usually means knowing how tables relate. If the schema alone consumes most of a working context, then anyone changing code either loads the whole model and has no room for the code, or works without sight of the relationships. The second is what produces hand-written joins and unenforced references, and it applies to people and tools alike.

## Platform alignment

Establish what the system actually chose, then compare it against the standard and direction from intake question 12. Without that answer this probe produces a list of technologies and no finding.

```bash
# data layer: engine, version, and anything engine-specific
grep -n 'provider' prisma/schema.prisma drizzle.config.* 2>/dev/null
grep -rn 'mysql\|mariadb\|postgres\|sqlite\|mssql' docker-compose.yml .env.example 2>/dev/null
grep -rin 'ONLY_FULL_GROUP_BY\|LIMIT 18446744\|::jsonb\|ILIKE\|RETURNING' \
  --include='*.ts' --include='*.sql' . 2>/dev/null | head   # engine-locked SQL

# hosting and runtime model
ls Dockerfile* docker-compose*.yml *.tf *.yaml 2>/dev/null
grep -rn 'output:' next.config.* 2>/dev/null                # standalone vs server
grep -rn 'localhost\|/home/\|/var/www\|C:\\\\' --include='*.ts' --include='*.mjs' . 2>/dev/null | head

# what pins it to one long-lived machine
grep -rn 'writeFile\|createWriteStream\|os.tmpdir\|process.cwd' lib app --include='*.ts' | head
grep -rn 'let [a-zA-Z]* = \(false\|null\|new Map\)' lib app --include='*.ts' | head   # in-process state
```

Report each choice in a table with three columns: what this system uses, what the organisation's standard is, and the classification. Matches, diverges, or regresses.

Two things decide the severity, and neither is the technology itself.

**Is it a regression or merely a divergence?** A sound choice that differs from the standard costs a permanent second thing to operate. A choice the organisation has already migrated away from costs that, plus the reversal of completed work, plus having the original argument again. The second is materially worse and should be labelled as such.

**Is it configuration or architecture?** Ask what would have to change. An engine swap before real data exists is small and worth doing immediately. An application whose file handling, background work and session model all assume one machine with local disk is architected around the choice, and the migration cost belongs in the estimate.

The probes above answer the second question directly. Local filesystem writes, in-process mutable state and absolute host paths are the three signals that a hosting model is baked in rather than configured, and they are the same signals that determine whether the system can run as more than one instance.

Engine-locked SQL matters for the same reason. Raw queries using one engine's dialect turn a configuration change into a rewrite of every raw query, so count them before estimating.

## Migration readiness

Where years of history must come in from an existing system, this is frequently the largest line item and the one found last. Two separate questions, and conflating them is the usual error.

**One: is there import machinery?**

```bash
grep -ocE '^model [A-Za-z]*(Import|Staging|Stg|Raw|Landing|Ext)[A-Za-z]*' <schema>
grep -oiE '^model [A-Za-z]*(Mapping|Alias|Canon|Xref|Lookup|Legacy)[A-Za-z]*' <schema>
grep -cE '^\s+(sourceSystem|sourceRecordId|legacyId|externalId|sourceRef)\s' <schema>
grep -n -A20 '^model ImportBatch' <schema> | grep -iE 'rowsAccepted|rowsRejected|status|validation'
```

**Two, and this is the one that matters: was the target designed against the actual source?** Machinery proves somebody anticipated an import. It does not prove anybody looked at what is coming.

The tells that the source was **not** examined:

- **Generic staging.** A landing table holding `rawJson` and `mappedJson` accepts anything and defers the mapping. Flexible, and a sign the shape was unknown when it was written.
- **Mapping tables with no rows and no seed.** The structure exists, the content was never specified.
- **Enumerations that look invented.** Status and type values that read as designed from first principles rather than derived from what the source actually contains.
- **The import path never exercised.** Cross-reference against per-module iteration counts. A migration path with one commit was never run against anything real.
- **No profiling artefact anywhere.** No row counts, distinct-value lists, null-rate notes or sample extracts in the repository or the documentation.

The tells that it **was**: staging columns mirroring a specific source system's fields, a documented field-level mapping, enumerations whose values match the source's, and handling for named quirks of that source.

**If the source was not examined, say what that means in the estimate**, because it changes the shape of the work rather than its size. The migration becomes a discovery exercise before a transformation one: profile the source, write the field-level mapping, decide what happens to source fields with no destination and to required destination fields with no source, then reconcile totals and handle the residue.

Three structural mismatches to check for explicitly, because each is a redesign rather than a mapping:

```bash
# does the target hold enough to represent the source's granularity?
grep -cE '^model ' <schema>              # target entity count, against the source's
grep -nE '@id|@@id|@unique|@@unique' <schema> | head    # identifier strategy
```

- **Granularity.** One row per transaction against one per line, one contact per organisation against many. A mismatch here means restructuring, not translating.
- **Identifiers.** Composite natural keys in the source against generated integers in the target means every relationship is rebuilt during load, in dependency order, with a lookup held throughout.
- **Parallel running.** If both systems operate at once, a one-off migration becomes an ongoing bidirectional sync with conflict rules. That is a different commitment and it is rarely costed.

## What is worth taking

Absorbing into an existing platform needs a different measurement from repairing in place: not what is broken, but what is worth moving. Produce the split explicitly.

```bash
# commodity versus differentiating, by module
ls lib/ | while read m; do
  n=$(git log --oneline -- "lib/$m" | wc -l)
  printf "%4d  lib/%s\n" "$n" "$m"
done | sort -rn
```

High iteration means used and refined; one commit means never opened. Then classify each surviving module by hand into commodity (contacts, files, auth, notes, calendars, generic admin) and differentiating (whatever encodes this business's own rules). The commodity half usually should not be ported at all, since a receiving platform either already has it or can buy it, and that split alone often removes more than half the work.

Then check whether the domain rules travel:

```bash
grep -rl 'export function\|export const' lib/<differentiating-module> | head
grep -rn '<differentiating concept>' app components | wc -l   # is the rule in one place or forty
```

A rule expressed in a named calculation module, a rules register, or a methodology document moves cleanly. The same rule spread across forty screens does not, and that distinction changes the estimate more than any count of defects.

## Is the domain knowledge recoverable

The question a team inheriting this has to answer before they can safely change anything, and the one most likely to come back better than feared.

```bash
find docs -name '*.md' | xargs wc -l | sort -n | tail -20
git log -1 --format='%ad' --date=short -- docs/          # is any of it maintained
grep -rliE 'methodology|glossary|business rule|register|decision record' docs/ | head
grep -rc 'Verified\|verified' <(git log --format='%B')   # do commit bodies carry evidence
```

Look for three things specifically, because their presence changes the handover risk materially:

- **A methodology or calculation document** that states a root cause, the corrected formula and a measured before-and-after. That is analysis rather than narrative and it means correctness is knowable without the author.
- **A glossary** with precise, non-generic definitions of the trade vocabulary. Generic definitions mean it was generated; specific ones mean somebody knew the domain.
- **A rules register extracted mechanically** from live configuration rather than written by hand, since that reflects the system as it actually runs.

Then check the other direction. A document a newcomer trusts on day one and which is **wrong** is worse than no document. Check the architecture overview against reality, check whether any document lists shipped modules as unbuilt, and read any quality report the documentation set produced about itself, since those often admit which claims were never verified.

## Claimed against actual

Read the repository's own account first, then test each claim. A capability is not present because a table, a screen or a setting exists.

```bash
ls docs/ && wc -l README.md docs/*.md
git log -1 --format='%ad' -- docs/ARCHITECTURE.md      # date a doc last changed
```

### Look beyond markdown

Non-technical authors write in Word, Excel and PowerPoint. Grepping `*.md` misses the requirements.

```bash
find . -path ./node_modules -prune -o -type f \
  \( -iname '*.docx' -o -iname '*.doc' -o -iname '*.pdf' -o -iname '*.xlsx' \
     -o -iname '*.xls' -o -iname '*.pptx' -o -iname '*.csv' -o -iname '*.odt' \) -print

git log --all --diff-filter=A --name-only --format="" \
  | grep -iE '\.(docx?|pdf|xlsx?|pptx?)$' | sort -u    # including deleted
```

Office formats are ZIP archives of XML, so the standard library reads them with no dependencies:

```python
import zipfile, re, html
z = zipfile.ZipFile(path)
# .docx  -> word/document.xml,          tags <w:t>
# .xlsx  -> xl/worksheets/sheet*.xml,   tags <t>  (plus xl/sharedStrings.xml if present)
# .pptx  -> ppt/slides/slide*.xml,      tags <a:t>
text = [html.unescape(t) for t in re.findall(r'<[aw]?:?t[^>]*>([^<]*)</', 
        z.read('word/document.xml').decode('utf8','ignore'))]
```

For `.xlsx`, list sheet names from `xl/workbook.xml` first; the useful sheet is rarely the first one. PDFs need `pdftotext` or a file-reading tool.

Three things to check:

- **Does a markdown mirror exist?** Often the same content is committed both ways. Compare counts before spending effort on the binaries.
- **Do the binaries hold something the markdown does not?** Frequently yes, and it is usually the business view: what each module is for and what problem it solves, as opposed to the technical register of routes and models. That is exactly what intake and the executive summary need.
- **Are there annotation columns?** Inventories built for stakeholder review often carry an empty "comments", "priority" or "owner" column. Filled, it is business context available nowhere else. Empty, it tells you the review it was prepared for never happened.

A file prefixed `~$` is a Word lock file, meaning the document was open when it was committed. Minor, but it indicates the document is actively edited rather than generated.

Documents frozen at the first commit are common. Documents contradicting themselves are the stronger finding: grep a roadmap for both "not built" and "Built" and check whether they cover the same stages.

For each claimed feature, walk the four steps. The gap is usually between 2 and 3.

```bash
grep -n "^model Thing" prisma/schema.prisma                       # 1. table exists
grep -rn "prisma\.thing\.\(create\|upsert\)" app lib scripts      # 2. anything writes it
grep -rln "MediaRecorder\|createReadStream\|SignJWT" app lib      # 3. the mechanism exists
find app/api -ipath '*thing*'                                     # 4. reachable
```

A model written only by a form action that receives a filename as a parameter is a register, not a capture system. A configuration table with admin screens and no library installed is administration over nothing.

## Supply chain and secrets

Two questions: what executes at install time, and where do credentials live.

**Do not run the install to find out.** Everything here reads the manifest and lockfile only. See `SAFE-CONDUCT.md`.

```bash
# packages that execute code at install time (reads the lockfile, installs nothing)
python3 - <<'EOF'
import json
d = json.load(open('package-lock.json'))
pkgs = d.get('packages', {})
hooked = [k for k, v in pkgs.items() if v.get('hasInstallScript')]
missing = [k for k, v in pkgs.items() if k and not v.get('integrity') and not v.get('link')]
print(len(hooked), 'packages run install scripts')
print(len(missing), 'of', len(pkgs), 'entries lack an integrity hash')
EOF

grep -n 'ignore-scripts' .npmrc package.json 2>/dev/null      # is execution disabled
grep -rn 'npm ci\|npm install' Dockerfile .github/workflows/ 2>/dev/null
grep -oE '"resolved": "https?://[^/]+' package-lock.json | sort | uniq -c | sort -rn
```

Lifecycle scripts are the delivery mechanism for the current generation of registry worms, which steal maintainer and cloud tokens at install time and republish themselves into further packages. Count them, and check whether anything disables them. `npm install` in a build is not reproducible against the lockfile; `npm ci` is.

A dependency resolved from somewhere other than the registry is not automatically wrong, since vendors do leave npm, but it sits outside every registry-based scanner and needs a named owner watching releases.

### Which defences exist, and are they still aligned

Look for the control before reporting its absence, and when one is present, check it still matches what is installed. A defence that has drifted is worse than none, because it is counted as protection.

```bash
# install-time defences
grep -rn 'minimumReleaseAge\|minimum-release-age\|cooldown' .npmrc package.json 2>/dev/null
grep -rn 'ignore-scripts\|ignoreScripts' .npmrc package.json 2>/dev/null
node -e "const p=require('./package.json');console.log(JSON.stringify({allowScripts:p.allowScripts,pnpm:p.pnpm,overrides:p.overrides,resolutions:p.resolutions},null,2))"
grep -n 'registry=' .npmrc 2>/dev/null                       # private proxy rather than the public registry

# automation
ls .github/dependabot.yml renovate.json .snyk .socket.yml 2>/dev/null
grep -rn 'audit\|sbom\|cyclonedx\|syft' .github/workflows/ 2>/dev/null

# secret hygiene tooling
ls .pre-commit-config.yaml .gitleaks.toml .secrets.baseline 2>/dev/null
grep -rn 'gitleaks\|trufflehog\|git-secrets\|detect-secrets' .github/workflows/ 2>/dev/null
```

The defences worth naming, roughly in order of value against the current threat:

- **A minimum release age**, refusing versions published within the last several days. This is the direct mitigation for registry worms, which are typically caught and pulled within hours to days, so a cooldown means you never install the compromised window.
- **An install-script allowlist**, so only named packages may execute at install time.
- **`ignore-scripts` by default**, with exceptions handled explicitly.
- **Overrides or resolutions** pinning transitive packages to safe versions.
- **A lockfile plus a frozen install** (`npm ci`) everywhere, including the container build.
- **A private registry or proxy**, which gives you a chokepoint and a quarantine.
- **Secret scanning in CI or pre-commit**, and a secrets manager rather than files on disk.
- **Provenance or attestation checking** where the ecosystem supports it.

**Then verify alignment.** Compare each control against the lockfile rather than accepting its presence:

```bash
python3 - <<'EOF'
import json
lock = json.load(open('package-lock.json')).get('packages', {})
allow = json.load(open('package.json')).get('allowScripts', {})
hooked = {k.split('node_modules/')[-1] for k, v in lock.items() if v.get('hasInstallScript')}
allowed = {a.rsplit('@', 1)[0] for a in allow}
print('runs scripts but not allowlisted:', sorted(hooked - allowed))
print('allowlisted but no longer runs scripts:', sorted(allowed - hooked))
for a, _ in allow.items():
    name, pin = a.rsplit('@', 1)
    got = next((v.get('version') for k, v in lock.items() if k.endswith('node_modules/' + name)), None)
    if got and got != pin:
        print(f'STALE PIN: {name} allowlisted at {pin}, installed {got}')
EOF
```

In one assessment this found a thoughtfully built allowlist that had come apart: a handful of packages pinned by exact version, but an `overrides` entry had bumped one of them past its pin, so the allowlist entry no longer matched the installed version, while a different package that does run install scripts was not listed at all. The control looked present in the manifest and covered neither of the things it was supposed to.

Report both halves. Controls that exist are a genuine strength and belong in the report; a control that has drifted is a finding in its own right, distinct from never having had one.

**Secrets, without reading them.** Confirm location and class only.

```bash
for f in .env .env.local .env.production; do
  [ -f "$f" ] && echo "PRESENT: $f ($(wc -l < "$f") lines)"
done
git log --all --diff-filter=A --name-only --format="" | grep -E '^\.env' | sort -u
grep -rlniE 'secretsmanager|vault|doppler|keyvault|1password' . \
  --include='*.ts' --include='*.yml' --include='*.tf' 2>/dev/null | head
```

Never `cat` these. The findings are: whether secrets sit in a file on disk rather than a managed store, whether one was ever committed, how many distinct secrets share a single file, and whether one key serves several purposes.

That last one matters more than it looks. A single signing key covering sessions, second-factor challenges and an external portal means rotating it logs everybody out simultaneously, which is why rotation gets deferred indefinitely and why the key stays valid long after it should have changed.

## Has anyone looked at the screens

Distinct from both "has this code ever run" and "does the calculation compute the right thing", and it catches what neither does. A screen can execute cleanly, contain arithmetically correct functions, and still show the wrong thing: the query filters on the wrong column, the label does not match the value beneath it, the form writes to a different field than the one it displays, the total sums a column nobody intended. Nothing static finds these. Somebody has to look at the screen with known data and check.

**Look for evidence of execution, not for a plan.** Test checklists, UAT packs and acceptance criteria are cheap to generate and frequently exist in quantity. That proves somebody anticipated testing.

```bash
find docs -iname '*test*' -o -iname '*uat*' -o -iname '*acceptance*' -o -iname '*smoke*'

for f in <the files found>; do
  echo "$f  last touched: $(git log -1 --format=%ad --date=short -- "$f")"
  echo "   ticked: $(grep -c '\[x\]' "$f")   unticked: $(grep -c '\[ \]' "$f")"
  grep -ciE 'tested by|date tested|result|pass/fail|sign.?off' "$f"
done
```

Three signals decide it:

- **Ticked against unticked.** A checklist with every box empty was written and never run. Zero ticked across several hundred items is unambiguous and needs no interpretation.
- **Last touched.** A checklist whose only commit is the initial one was produced with the system, not used on it.
- **Result columns.** A plan has steps. A record has a date, a tester and an outcome. Their absence tells you which one you are holding.

Also check for the artefacts a real pass leaves behind: screenshots, a defect log, an issue tracker with items raised during testing, release notes referencing verification.

**Report the count plainly**, because it is one of the few findings that cannot be argued with. Then state the consequence: nothing has confirmed that what the screens display matches what the data says, so any figure a user has seen is unverified.

### The minimum bar, when there are no automated tests

Automated tests covering forms, dashboards and displayed figures are the right answer, and where they are absent the recommendation should say so. But a full suite is months of work, and there is a floor beneath it worth naming, because it is achievable in days and the gap between it and nothing is enormous.

**Every function of the system, exercised once by a person, against known data, with the result written down.** For each screen:

- It loads without error, in each role that can reach it.
- Every form saves, and the saved value is correct when the record is reopened.
- Every displayed figure is traced to its source at least once, with data whose correct answer is known in advance.
- Filters, sorts and date ranges change the result in the direction they claim.
- Exports carry the same figures as the screen they came from.
- Destructive and state-changing actions do what their label says, and nothing else.

Recorded means a date, a name and an outcome per item. A pass nobody wrote down is not evidence, and it will be repeated from scratch the first time anyone asks whether something works.

State the reasoning in the report, because it is the argument that gets this funded: **generated code is plausible by construction, and plausible is not the same as correct.** The failure mode is not code that crashes, which is obvious, but code that runs and produces a confident wrong answer, which is invisible until somebody compares it against something they already know.

### Say plainly what a manual pass does not buy

Recommend the floor, then set expectations about it in the same breath, or it will be mistaken for a solution.

**A manual pass is a snapshot, not a safety net.** It establishes that the system was correct on one date, with one dataset, in one configuration. It says nothing about the system after the next change.

**Its cost recurs in full.** An automated test is paid for once and re-run for nothing, so its cost is roughly flat however often the code changes. A manual pass is paid for every time, so its cost multiplies by the number of releases, and grows again as the system does. Two hundred manual checks across forty releases is eight thousand checks. That arithmetic is what makes the floor a temporary position rather than a strategy.

**And the re-test decision cannot be made honestly.** After a change, somebody has to decide what to re-check. That needs the blast radius: which screens and figures this change could affect. In a codebase with declared relationships, adopted shared components and one implementation per rule, the blast radius is roughly knowable. In the systems that lack automated tests, it is usually the same systems where logic is duplicated across modules, abstractions exist but are not used, and relationships are not declared anywhere a tool can read. The blast radius is unknowable precisely where it matters most.

Faced with that, teams do one of three things: re-test everything, which is unaffordable after the first few rounds; re-test what they believe was touched, which is a guess dressed as a plan; or re-test nothing and find out from users. In practice the third, then the first after an incident, then the third again.

**So position it correctly in the recommendation.** A manual pass is a go-live gate. It buys the decision to go live and nothing after it. The ongoing answer is automated coverage, and the practical route is to characterise the highest-value paths in tests as each area is repaired, so the manual burden shrinks with every phase instead of repeating at every release.

## Run the calculations

**Grep finds absent mechanisms. It does not find a formula that computes the wrong thing.** That has to be read and executed, and it is often the single most consequential finding in an assessment, because a wrong number looks like a number and gets acted on.

Budget an hour. Do not skip it because nothing in the structural sweep pointed here; nothing will.

**Pick the calculations people act on.** Forecasts, projections, commission or bonus, budget phasing, weighted or probability-adjusted totals, unit costs, anything feeding an executive screen or a payment. Usually five to fifteen functions in total.

**For each, extract the formula and run it with realistic inputs.** Not the code's own tests, which do not exist. Copy the expression into a scratch script, feed it the values a real caller supplies, and look at what comes out.

```bash
# what does this function actually receive?
grep -rn "periodElapsed(\|computeForecast(\|weightedAmount(" app lib --include='*.ts'
# then reproduce it standalone with those inputs and sanity-check the output
node -e "const fy=2027; const start=Date.UTC(fy+1999,6,1); console.log(new Date(start).toISOString())"
```

The question is never "does this compile". It is **is this number plausible**. A fraction of a year should sit between 0 and 1 and move through the year. A run rate should be near the booked figure early and above it late. A commission should be a small share of revenue. Anything that is constant when it should vary, or an order of magnitude out, shows up in one line of output.

### The smells that make it findable

- **A magic numeric constant in date or period arithmetic.** `year + 1900`, `century + offset`, `slice(-2)`. These encode an assumption about two-digit versus four-digit years that the callers may not share.
- **A clamp or a floor.** `Math.max(x, 0.05)`, `Math.min(x, 1)`. A clamp exists to handle an edge case; if it fires on every ordinary input it is not handling an edge case, it is concealing a broken one. **Check what the value is before the clamp.**
- **A comment describing a different convention than the code uses.** A note describing a two-digit year convention above arithmetic that receives a four-digit one is the defect stated in plain language.
- **Division by something derived.** Anything of the form `total / elapsed` where `elapsed` is computed rather than counted.
- **A widely-called function with one commit.** Cross-reference the calculation list against per-file iteration counts. A formula used by six screens and never revisited since the initial drop was never checked against a real figure.
- **Silent fallbacks.** A calculation returning `0` or `null` on missing input, so a data gap becomes a confident wrong number rather than an error.

Report these with the reproduction, not the reasoning. Four lines of input and output settle it; a paragraph of explanation invites argument.

## Trace one operation through every layer

Individual absences understate the risk, because they compound on the same operation. Counting missing transactions and missing constraints separately does not show what happens when both are missing on the write that matters.

Pick three to five **routine business operations**, not edge cases. Merging two duplicate records, closing a period, cancelling an order, running the monthly import, reassigning ownership. Then follow one from the button to the database and ask what would catch a failure:

| Layer | Question |
|---|---|
| Transaction | Do the writes succeed or fail together? |
| Constraint | Would the database refuse an inconsistent result? |
| Error handling | Does a failure surface, or is it discarded? |
| Audit | Is there a record that the operation happened, and is it reliable? |
| Idempotency | What happens if it runs twice? |
| Reconciliation | Would anything later notice the result was wrong? |

Count how many layers would catch it. **Zero or one means the operation fails silently and permanently.** That is a materially different statement from six separate findings, and it is the version a non-engineer understands immediately.

The strongest form quotes the operation's own code next to its own claim. An operation described in a commit message as proving no reference is lost, whose implementation discards the errors on three of its ten writes, makes the point without any commentary.

## In-flight work and deploy collisions

```bash
grep -rln "MediaRecorder\|new Blob\|FormData" components app     # long client-side actions
grep -rn "IndexedDB\|localStorage\|sessionStorage" components    # any client-side durability
grep -rn "SIGTERM\|gracefulShutdown\|drain" app lib scripts docker
grep -rn "maintenanceMode" app lib | grep -v settings            # does the flag block, or only display
```

Read the catch around each upload. `catch { /* keep going */ }` means the user is shown success after the payload was lost. Check whether the file write and its database record share a transaction; they usually do not.

## Has it ever run?

Code that runs gets fixed. Code touched once either worked first time or was never executed.

```bash
# files never revisited since creation, sampled
git ls-files 'lib/*.ts' | shuf -n 200 | while read -r f; do
  [ "$(git log --oneline --follow -- "$f" | wc -l)" -le 1 ] && echo "$f"
done | wc -l

# per-module iteration: low counts are suspect
for d in lib/*/; do printf "%4d %s\n" "$(git log --oneline -- "$d" | wc -l)" "$d"; done | sort -n
```

Cross-check with tables never written to (no `create`/`upsert` anywhere, including nested writes), stub markers, and exported functions with no importers. Distinguish "definitely never runs" from "probably" from "unclear".

## Controls that report a state they do not implement

The highest-value probe in the set. For each setting, status field or control surface, find where it is **read**.

```bash
grep -rn "encryptAtRest" --include='*.ts' --include='*.prisma' .
# written by a toggle, read by nothing = decorative
```

Then read the function behind any "run"/"verify"/"test" button. Check it performs the action rather than recording that it did. Check whether a verification step inspects real output or its own metadata.

Enum-style placeholder values (`daily_placeholder`) are an admission in the schema.

## Constraints: never added, or removed?

Completely different remediation stories. Measure both ends.

```bash
git show $(git rev-list --max-parents=0 HEAD):prisma/schema.prisma > /tmp/t0.prisma
# then count relations vs FK-shaped columns in /tmp/t0.prisma and in HEAD
```

Then look for removal events, and for trend:

```bash
cat migrations/*/migration.sql | grep -c 'DROP FOREIGN KEY'
# bucket migrations into thirds; count constraints added per new table in each
```

A declining density is a finding about how the system grew. Zero drops refutes any "constraints were torn out" theory outright.

**What can refuse a write.** Count columns that are nullable or defaulted. If most cannot reject anything, the database enforces nothing and every invariant lives in application code.

## Shared abstractions

Do not ask whether abstractions exist. Ask the adoption ratio.

```bash
grep -rl "SharedTable" app components | wc -l     # uses it
grep -rl "<table" app components | wc -l          # reimplements it
```

Repeat for buttons, modals, cards, result shapes, formatting helpers, access-control helpers, export paths. A good abstraction used by a handful of files while most hand-roll their own is a stronger finding than an absent one, because it proves the problem is discovery rather than design. Report the ratio you measured.

Also count verbatim repetition of long styling strings, and files declaring their own local copy of a shared helper.

## Type-safety escapes

```bash
grep -ro 'as never\|as unknown as\|@ts-ignore\|: any' app lib | wc -l
git log -S "ignoreBuildErrors" --oneline --reverse    # when, and read the message
```

Distinguish a cast that follows validation from one that replaces it. Both look identical, which is the problem. Check whether disabling happened once with a stated reason or accumulated silently, and whether a compensating gate exists elsewhere before claiming there is none.

## Money and domain types

Check preconditions before applying the rule.

```bash
grep -rhoE '`(netValue|amount|total)` [A-Z]+' migrations/*/migration.sql | sort -u
```

`DOUBLE` holds integers exactly to 2^53; `FLOAT` only to 2^24, so revenue above ~16.7m drifts. If values are whole units and the code rounds at calculation points, there may be no precision error at all. The real finding is usually that one business value has two column types in different tables, and that no database rule enforces the intended one.

Also check: is currency modelled at all; are gross, net and tax separable; do date columns carry timezone; do required fields derive a period at read time.

## Error visibility

```bash
grep -roE '\.catch\(\(\)\s*=>' app lib | wc -l
grep -roE 'catch\s*(\([^)]*\))?\s*\{' app lib | wc -l
```

Then sample a dozen and date them with `git log -S` or `git log -L`. A swallow written with the code it wraps is a defensive habit. One added later, in a commit mentioning a crash, is a symptom being suppressed. The ratio is the finding.

Check whether the audit or logging path swallows its own failures, and whether anything captures the console in production.

## Scheduling and automation

A script named `cron:` is not scheduled.

```bash
find app/api/cron -name 'route.ts'      # what a scheduler can actually call
grep -rl "purge-audio" app lib scripts | grep -v 'scripts/purge-audio'   # inbound references
```

Zero inbound references plus no endpoint means it runs only when someone types it. Retention purges, snapshots and backups are the usual casualties.

## Multi-instance readiness

Enumerate every runtime filesystem write and every module-level mutable variable. Each is a single-instance assumption. Ask specifically: if an upload lands on instance A and the download is routed to instance B, what happens?

Check whether migrations run per-replica at boot, and whether the container image actually uses the build output it produces.

## Migration history

```bash
cat migrations/*/migration.sql | grep -cE '^\s*(DROP TABLE|DROP COLUMN|MODIFY)'
ls prisma/manual-sql/ 2>/dev/null     # hand-written patches are a strong signal
grep -rn "_prisma_migrations" prisma/manual-sql/
```

Scripts that write to the migration ledger by hand mean the ledger cannot be trusted to describe the live schema. **This is testable:** rows inserted by a script set start and finish timestamps in one statement, so they match exactly; a real migration updates the finish timestamp afterwards, so they differ.

Count scripts whose only purpose is repairing data. A cluster around one entity points at a missing uniqueness rule.

## Commit narrative

```bash
git log --format='%h|%ad|%s' --date=short | grep -v 'chore(release)'
git log --grep='workaround\|for now\|temporarily\|unblock\|placeholder' --oneline
```

Measure the fix-after-feature rate: scoped fixes landing within 24 hours of a feature commit in the same area. Read bodies, not just subjects. Report genuine root-cause work as prominently as the shortcuts; its presence changes the assessment of whether the system is salvageable.
