# Intelligence Level — the delegation agents, as binaries

Downloadable executables for each level of **Delegation Intelligence**, one binary per level, behind
one executor interface.

## What ships

| binary | what it is | needs credentials? |
|---|---|---|
| `dli-dl0` | one declared operation, executed. Deterministic, no strategy. | **no** |
| `dli-dl1` | short local plan over a known procedure | yes |
| `dli-dl2` | multi-step, persistent state, ordinary error recovery | yes |
| `dli-dl3` | strategy constructed from an outcome and constraints; replans | yes |
| `dli-accept` | acceptance checks — which themselves run at DL0 | **no** |
| `dli-bench` | episode runner and scorer | **not built** — exits 70 and says so |

Every binary answers the same three calls:

```sh
dli-dlN --capabilities          # JSON: level, plans, llm_required, network_required, cost model
dli-dlN exec --contract c.json  # run a task contract, emit an episode record
dli-dlN --version               # version, build SHA, and the tier SHA it was built from
```

Exit codes are part of the contract: `0` success, `2` usage, `3` specification refusal,
`4` self-certification refusal, `70` not implemented.

## Install

```sh
sha256sum -c SHA256SUMS          # verify first
python3 dli.pyz dli-dl0 --capabilities
# or
pip install dli_dist-0.4.0-py3-none-any.whl
```

The zipapp needs `python3` on the target. A PyInstaller onefile is **deliberately not shipped** —
it vendors CPython stdlib network clients, which would silently break the property below.

## The property that makes the tier real rather than named

> **A DL0 step must not cost an LLM call.** `dli-dl0` and `dli-accept` run correctly on a machine
> with **no API key and no network**.

Higher levels do not make lower ones obsolete: atomic reliable procedures belong in DL0/DL1
executors, complex planning in higher-DL agents, and independent verification deliberately uses the
simpler, constrained tier — *a verifier that plans is a verifier that can be persuaded*.

Check it yourself, offline:

```sh
unshare -Un env -u ANTHROPIC_API_KEY -u OPENAI_API_KEY python3 dli.pyz dli-dl0 --capabilities
# → {"level":"DL0","llm_required":false,"network_required":false,
#    "cost_model":{"llm_calls_per_invocation":0}, ...}
```

Each level also **refuses what is above it, before touching a model client** — an executor that
calls a model in order to decide to refuse has already attempted the contract.

## What has been measured, and what has not

These binaries were built alongside a benchmark harness and an independent acceptance locus. The
measurement, stated at its real strength — including two qualifications found **after** the first
publication of this page and recorded here rather than quietly dropped:

- **88 delegation episodes** — 4 independent agent implementations × 22 benchmark rows.
- **74 accepted at `alpha3`** — accepted by a locus that did not build the systems, against
  criteria it compared **byte for byte** rather than paraphrasing.
- **53 of 88 recorded `uncertain`**, not `pass`. Verdicts were issued only where the oracle actually
  ran; `uncertain` was never rounded up.
- **Seven κ-cells** reported as `F*(κ,h,p)` with Wilson intervals — never a single scalar. Sample
  sizes are **3 to 8 per cell**, so the intervals are wide and are reported as wide.
- **The exam was proven to discriminate** before it was trusted. An earlier version could be passed
  by reading the task card; that was found, declared a defect, and repaired, and four blind agents on
  byte-identical prompts then confirmed the repair 4/4.
- The verifier's **false-pass rate was measured** (not assumed) and reported decomposed, and one whole
  task family was **excluded** because its verifier passed every seeded defect.

### Two qualifications on the numbers above

**The criterion was vacuous on 10 of 22 rows when those episodes were accepted.** The benchmark's
`budget_cross` stratum ships one identical `success_criterion` across all 36 of its rows —
*"Declared in the criterion register before the work, outside the system's write set."* That is a
directive about where the criterion must live, not a condition anything can be tested against. So the
byte-identity check that gates `alpha3` was satisfied against a sentence stating no success
condition, on the rows that produced most of the cells and most of the acceptances. The register now
carries a real condition per row, derived from what the sealed oracle enforces, marked
`criterion_source: benchmark_authored` and distinguishable from `dataset_verbatim`, with the
declaration timestamped so it can be shown to precede the work. **The 74/88 figure was measured
before that fix and has not been re-measured.**

**Three of the seven cells differ only by a label.** The `budget_cross` stratum varies the
human-intervention budget (H0/H1/H2), but across all 22 rows **zero cognitive interventions were
raised** — so rows nominally at H1 and H2 were conducted at H0. Reporting them as separate cells
implies a separation that was never exercised.

Both were found by lanes reporting against their own passing results, which is the only reason they
are here.

> **No delegation level is certified by this repository.** These are accepted episodes on a small
> sample, with the qualifications above. A level is measured by a locus that did not perform the work,
> against tasks it did not author, at sample sizes these intervals do not yet support.

`dli-bench` is a declared stub. It exits 70 and says so rather than pretending.

## Verifying what you download

`SHA256SUMS` covers every file; `MANIFEST.json` records, per artifact, the source commit it was built
from and the tier commit it consumed. `build_sha` is a baked literal — the environment cannot forge
it through a variable.

One redaction: `MANIFEST.json`'s `builder.interpreter` was an absolute path on the build host and was
replaced at publish time; that file's checksum was regenerated. The artifact digests
(`dli.pyz`, the wheel) are unchanged and are exactly what the build produced.
