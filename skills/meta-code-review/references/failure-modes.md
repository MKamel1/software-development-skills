# AI-Code Failure Modes — by producer

The taxonomy `meta-code-review` routes on. Each mode lists who it's worst in and a
cheap detection heuristic. Sources: package-hallucination study (Spracklen et al.,
USENIX Security 2025); Copilot security studies (Pearce et al., "Asleep at the
Keyboard," IEEE S&P 2022; and the separate real-world study arXiv:2310.02059);
AI-vs-human PR analysis (CodeRabbit); SWE-agent failure taxonomies; and
code-hallucination surveys (arXiv:2511.00776, arXiv:2512.05239). Figures are
directional industry signal, not ground truth — verify before citing exact numbers.

| Failure mode | Worst in | Cheap detection |
|---|---|---|
| **Package / API hallucination** — nonexistent library, function, or parameter | weak AI (≈21.7% of packages vs ≈5.2% for commercial models) | resolve every new import against the lockfile/registry; grep used symbols against the package's real exports/types. Never trust "this symbol exists." |
| **Insecure patterns** — SQLi, hardcoded secrets, missing validation, weak crypto | all producers (~40% of Copilot scenarios vulnerable) | run SAST/secret-scan regardless; verify, don't accept "no security issues." |
| **Plausible-but-wrong logic** — syntactically valid, semantically wrong | strong AI (AI PRs ~1.7× more issues, correctness-skewed) | trace one concrete input through each changed branch by hand. |
| **Incomplete but looks done** — stubbed error handling, TODO wiring, skipped hard parts | weak + strong AI | diff acceptance criteria vs what's actually wired; grep `TODO|stub|pass|NotImplemented`; confirm new paths are called from somewhere. |
| **Over-engineering** — abstractions/factories/interfaces for one implementation | strong AI | count abstraction layers vs concrete callers; flag any 1-implementation seam. |
| **Under-engineering / missing edge cases** | AI + junior | enumerate the edge cases a senior expects (empty/null, concurrency, failure/retry, scale); check each. |
| **Fault mislocalization** — patches the symptom file, not the file that owns the bug | weak–mid AI | check sibling callers for the same bug class; a fix touching only the reported site is suspect. |
| **Silent constraint / spec violation** — skips a stated rule without flagging it | all AI, worse in long context | re-check the diff against the *original* spec line by line; don't trust the agent's own "all requirements met." |
| **Programming by coincidence** — passes the shown example, not the general rule | junior + AI | feed an input not in the prompt/example; ask "why does this work," not "does it." |
| **Fabrication** — invented numbers, library behavior, or citations presented as fact | weak AI (frontier under pressure too) | independently verify any "verified/tested" claim or cited figure. |
| **Quantization / numeric artifacts** — subtly wrong constants, off-by-one | small/quantized local models | extra scrutiny on numeric and boundary logic. |

## Producer-calibration model (where to spend review budget)

- **Weak/small AI** (local, quantized, small-tier): hallucination + fabrication +
  mislocalization + numeric artifacts dominate. Verify almost everything mechanically
  — imports, referenced symbols, constants — rather than trusting a prose walkthrough;
  confirm the patch applies, compiles, and runs.
- **Strong/frontier AI**: syntax/package errors rare; the risk shifts to semantic
  logic errors that look idiomatic, over-engineering, and getting the letter of the
  request right but the intent wrong (especially over long multi-file sessions where
  earlier constraints drift). Trace logic and spec-intent; challenge every abstraction's
  necessity; assume "looks production-quality" ≠ correct.
- **Junior developer**: architectural/maintainability blind spots, weak edge-case and
  error-handling coverage, programming-by-coincidence, over-literal rule-following.
  Rarely hallucinates APIs (a human hits an import error). Shift budget from
  fact-checking toward design/testing coaching — and credit genuine fresh-eyes catches
  (naming, unclear docs, overly clever code) that senior/AI reviewers overlook.

**If the producer is unknown**, hedge toward the union: run the weak-AI mechanical
checks *and* the strong-AI logic/intent checks, and state that you couldn't calibrate.

## Reviewing the review specifically

Beyond the code's failure modes, an AI-produced *review* has its own:
- **Citation drift** — a finding pointing at the wrong/nonexistent line, or claiming a
  symbol is missing when it's present. Open every cited location.
- **Severity miscalibration** — uniform confident tone across a one-line typo fix and a
  risky auth rewrite. Re-rate against real blast radius.
- **False-positive noise** vs **missed real bugs** — the recall/precision trade-off: no
  tool achieves both, so a review that claims to "catch everything" should raise
  suspicion of noise, not confidence. Check whether it picked a stance and held it.
- **Assertion without reasoning** — findings with no "why" correlate with lower
  reliability; a finding you can't reconstruct from the diff is unverified until you do.
