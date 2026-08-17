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

**How much must be held in view.** Divide bytes by ~3.6 for a token estimate. Compare the data model alone against a working context window. If the schema does not fit alongside the code being changed, relationship-level mistakes are structurally likely, for people and tools alike.

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

In one assessment this found a thoughtfully built allowlist that had come apart: six packages pinned by exact version, but an `overrides` entry had bumped one of them past its pin, so the allowlist entry no longer matched the installed version, while a different package that does run install scripts was not listed at all. The control looked present in the manifest and covered neither of the things it was supposed to.

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
