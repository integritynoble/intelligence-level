# Intelligence Level — the delegation agents, as binaries

Downloadable executables for each level of **Delegation Intelligence**, one binary per level, behind
one executor interface.

> **Status: empty.** No artifacts have been published yet. The build lane (`dl-dist`) is running; this
> repository is the target, not the result. Nothing here has been accepted by an independent locus,
> and until it has, no level claim below should be read as measured.

## What ships here

| binary | what it is | credentials needed |
|---|---|---|
| `dli-dl0` | one declared operation, executed. Deterministic, no strategy. | **none** |
| `dli-dl1` | short local plan over a known procedure | small/cheap model |
| `dli-dl2` | multi-step, persistent state, ordinary error recovery | yes |
| `dli-dl3` | strategy constructed from an outcome and constraints; replans | yes |
| `dli-accept` | acceptance checks — which themselves run at DL0 | **none** |
| `dli-bench` | episode runner and `F*(κ,h,p)` scorer | **none** |

Every binary answers the same three calls:

```sh
dli-dlN --capabilities          # JSON: level, plans?, llm?, cost model, declared limits
dli-dlN exec --contract c.json  # run a task contract, emit a merged episode record
dli-dlN --version               # version, build SHA, and the SHA of the tier it was built from
```

## Why the lower levels ship too

Higher DL does not make lower-DL components obsolete. Atomic reliable procedures belong in DL0/DL1
executors; complex planning belongs in higher-DL agents; and independent verification deliberately
uses simpler, constrained components — which reduces cost and limits error propagation.

The requirement that makes this real rather than decorative:

> **A DL0 step must not cost an LLM call.** `dli-dl0` and `dli-accept` must run correctly on a machine
> with no API key and no network. If they cannot, the tier is named rather than built — and a
> downloadable DL0 that phones home is worse than useless to whoever downloads it.

That property is checkable by anyone who downloads the binary, which is the point.

## Verifying what you download

Every release carries `SHA256SUMS` and a `MANIFEST.json` naming the source commit each binary was
built from. A binary nobody can trace to a commit is not a release.

```sh
sha256sum -c SHA256SUMS
./dli-dl0 --capabilities        # smoke test; should work offline
```

## What this is not

These binaries do not certify their own level. A level is measured by a locus that did not perform the
work, against tasks it did not author — and where that has not happened, the honest report is
`unknown` rather than a number.
