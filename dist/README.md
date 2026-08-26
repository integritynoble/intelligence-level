# dli-dist — release artifacts

Version `0.4.0`, built from lane `dli/dl-dist` commit `202f61ebbd7820053d1eae485f26c17f72a79914`, packaging
`dl-brain` tier `f3099a550da1ef5a7b2ca76ed07f97fdcfe57feb`. All artifacts are the standard library only — no
dependency to install, nothing that reaches a network.

## What is here

| file | what it is | needs on the target |
|------|------------|---------------------|
| `dli_dist-0.4.0-py3-none-any.whl` | wheel; `pip install` gives you the six `dli-*` commands | Python ≥ 3.10 + pip |
| `dli.pyz` | single-file zipapp, all six binaries (multicall) | Python ≥ 3.10 (`/usr/bin/python3`) |
| `SHA256SUMS` | checksums for every other file here | — |
| `MANIFEST.json` | per-artifact provenance: sha256, size, lane commit, tier SHA, builder | — |

If a PyInstaller onefile was skipped, `MANIFEST.json` says so and why under
`skipped`.

## Verify before you run

```sh
sha256sum -c SHA256SUMS
```

Every line must print `OK`. `SHA256SUMS` does not list itself; every other file
in this directory appears in it.

## Smoke test

Wheel:

```sh
python3 -m venv /tmp/dli && /tmp/dli/bin/pip install ./dli_dist-0.4.0-py3-none-any.whl
/tmp/dli/bin/dli-dl0 --capabilities
```

Zipapp:

```sh
python3 dli.pyz dli-dl0 --capabilities
# or: ln -s dli.pyz dli-dl0 && ./dli-dl0 --capabilities
```

Both print a JSON capabilities document and exit 0.

## Exit-code convention (part of the contract)

A binary you script against returns:

| code | meaning |
|------|---------|
| `0` | success — the verb ran (for `exec`, an episode record was emitted on stdout) |
| `2` | usage error (no verb, unknown verb, unreadable/unparseable contract) |
| `3` | specification refusal — a contract defect or an un-provenanced kappa; the request was refused, not attempted |
| `4` | the record built would not validate — includes the F5/F6 self-certification refusal (a record naming its performer as its own verifier) |
| `5` | charter refusal — a benchmark row outside the set this lane may read |
| `6` | no model client — `DLI_MODEL_CLIENT` is unset, so a level above DL0 (`dli-dl1/2/3`) has nothing to construct what the contract left open. **Deliberately not `70`:** the binary is real, a precondition of the environment is missing. There is no model credential on this account, so `dli-dl1/2/3 exec` exits `6` here and no such run has ever been billed. |
| `7` | vendored tier absent — this build declares a tier SHA but does not carry the tier (`dli-dl1/2/3`). A correctly built artifact never returns this; if you see it, the build is broken. Also not `70`. |
| `64` | over-level refusal — the contract is shaped for a level above this binary (e.g. a DL3-shaped contract handed to `dli-dl1`); refused structurally, `failure_class: specification`, zero model calls. The level is a limit, not a label. |
| `70` | not implemented — the `dli-bench` stub, which intentionally does nothing. It is the only stub; `dli-dl1/2/3` are real. |

`dli-dl0` and `dli-accept` return `0/2/3/4/5`. `dli-dl1/2/3` add `6/7/64`, and —
once a model client is configured (there is none on this account) — the vendored
tier can additionally surface its own runtime-failure codes for a level that ran
and stumbled: `74` (the world refused — a path did not exist), `75` (execution),
and `70` for a *planning* failure. That last one reuses the same number this CLI
gives the `dli-bench` stub; the collision is a protocol-vs-charter conflict
recorded in `KNOWN_DEFECTS.md` rather than papered over, and it is unreachable on
an account with no model credential (`exec` stops at `6` first).

Note that `exec` reports **what was done**, never a verdict: a completed
`dli-accept` comparison exits `0` whether or not the bytes matched — the match is
in the evidence, not the exit code.

## Offline guarantee, and how to check it yourself

`dli-dl0` and `dli-accept` make no network call and need no credential. You do not
have to take that on faith — run them in an empty network namespace with the API
keys scrubbed:

```sh
env -u ANTHROPIC_API_KEY -u OPENAI_API_KEY unshare -Un python3 dli.pyz dli-dl0 --capabilities
```

It still exits 0. (`unshare -Un` needs no root on a box that already uid-maps you;
`--capabilities` and `exec` behave identically inside and outside the namespace,
because nothing here opens a socket.) The artifacts were also swept module-by-module
for any network/model import by `tools/check_pkg_no_network_imports.py`; see
`network_import_check` in `MANIFEST.json`.

`exec --contract` additionally reads the pinned benchmark file for kappa
provenance (`DLI_BENCH_PATH`); that is a local file read, not a network call.

**Nothing here is published.** A first push of this repository is the owner's
decision; these artifacts are committed to the lane branch and go no further.
