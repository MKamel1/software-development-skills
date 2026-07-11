# Audit Metrics — executable recipes

Cheap, agent-runnable commands for each step of the `codebase-audit` procedure.
All are approximations — ranking signals, not exact scores. Prefer a real tool
(`scc`, `dependency-cruiser`, `madge`) when it's already installed; the fallbacks
below need only `git`, `grep`/`egrep`, `find`, `wc`, `sort`, `uniq`.

First, exclude generated/vendored paths from every pass (adjust per repo):
`node_modules`, `dist`, `build`, `vendor`, `.min.`, lockfiles, generated code.

## 0. Scope & baseline
```bash
git log -1 --format=%cd                      # last commit date
git log --reverse --format=%cd | head -1     # first commit date (age)
git shortlog -sn --no-merges | head          # contributors / bus factor
find . -type f -not -path '*/.git/*' | wc -l # file count
```
Language/size mix: `scc` or `cloc` if available, else glob by extension.

## 1. Module inventory
```bash
# LOC per top-level dir (swap wc for scc --by-file if available)
for d in */; do echo "$(git ls-files "$d" | xargs wc -l 2>/dev/null | tail -1 | awk '{print $1}') $d"; done | sort -rn
```

## 2. Churn (most-changed files, last 12 months)
```bash
git log --format=format: --name-only --since="12 months ago" \
  | grep -v '^$' | sort | uniq -c | sort -rn | head -50
```
Bug-prone proxy (weak signal — depends on commit-message discipline):
```bash
git log -i -E --grep="fix|bug|broken" --name-only --format='' --since="12 months ago" \
  | grep -v '^$' | sort | uniq -c | sort -rn | head -20
```

## 3. Complexity proxy
- **LOC per file** (default): `git ls-files | xargs wc -l | sort -rn | head -40`.
- **Decision-point count** (cyclomatic proxy = decisions + 1):
  `grep -cE '\b(if|for|while|case|catch)\b|&&|\|\|' <file>` per hot file.
- **`scc --by-file`** if installed — gives a real per-file complexity estimate cheaply.
- Caveat: none capture nesting depth. Sanity-check outliers by eye.

## 4. Hotspots = churn × complexity
No proprietary math — take the **intersection** of the top-churn list (step 2) and
the top-complexity list (step 3). Files in both quadrants are the priority set.
`relative churn = churn ÷ file LOC` normalizes so big files aren't auto-flagged.

## 5. Change coupling (co-change)
For each commit touching ≥2 files, tally file-pair co-occurrences:
```bash
git log --name-only --pretty=format:"--%H--" --since="12 months ago"
```
Group filenames between `--HASH--` markers, count pairs. Report a pair when
`shared_commits ÷ avg(total_commits_each) ≥ threshold` and each file clears a minimum
revision count. Thresholds differ by source — code-maat defaults are `min-revs 5`,
`min-coupling 30%`; Tornhill's *Software Design X-Rays* argues a stricter ~20-commit /
50% bar for reporting (see [behavioral-analysis.md](behavioral-analysis.md) for the
trade-off and the "sum of coupling" ranking). Pick a bar and state it. Flag pairs whose
files live in different modules/layers. O(n²) in pairs — restrict to a recency window
and/or the top-churn files first.

## 6. Dependency & layering
- **Import graph** (grep fallback): per file, `fan-out` = count of `import`/`require`/
  `from … import` statements resolving inside the repo; `fan-in` = count of other
  files importing this module's path/export (reverse grep). Misses dynamic imports,
  barrel/re-export files, and DI/reflection — expect noise; verify outliers by hand.
- **Cycles**: `npx madge --circular src/` (JS/TS) if available; else DFS for back-edges
  on the grep-built graph.
- **Layer violations**: `dependency-cruiser` (JS/TS) / `import-linter` (Python) if
  present; else grep each lower-layer dir for imports of higher-layer paths. Requires
  knowing the intended layers — if undocumented, state them as a hypothesis first.
- **Instability**: `I = Ce / (Ce + Ca)` per module (Ce = fan-out, Ca = fan-in),
  aggregated to module level. Flag a high-Ca (depended-upon) module that itself has
  high I (depends on volatile modules) — fragile by the Stable Dependencies Principle.

## Method provenance
Hotspots = churn × complexity, change coupling, and the "~1–2% of code drives most
maintenance cost" claim are from Adam Tornhill's behavioral code analysis
(*Software Design X-Rays* / *Your Code as a Crime Scene*; open-source tool: code-maat).
Instability/abstractness from Robert Martin's package metrics (operationalized by
JDepend). These recipes replicate the public method; a later phase may distill the
books for deeper calibration.
