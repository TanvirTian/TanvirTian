Hey, I'm Tanvir 👋

I enjoy breaking things, fixing them, and occasionally figuring out why they broke in the first place.

Technologies

Go · Python · Docker · PostgreSQL · Redis · Linux

Still learning, still experimenting, still breaking things.

# Open Source Contributions

| Project | Merged PRs | Focus |
|---|---:|---|
| [ORAS](https://github.com/oras-project/oras) | 10 | OCI, security, concurrency, filesystem safety, CLI, CI |
| [oras-go](https://github.com/oras-project/oras-go)| 1 | Credential handling, API correctness |
| [Vinix](https://github.com/vlang/vinix) | 2 | AArch64/QEMU, musl/Linux |
| [go-criu](https://github.com/checkpoint-restore/go-criu) | 1 | Documentation / code quality |


---

## All contributions at a glance

| PR | Severity | Title | Area |
|---|---|---|---|
| [#2222](https://github.com/oras-project/oras/pull/2222) | 🔴 HIGH | Limit manifest config fetch size | Security / resource exhaustion |
| [#2218](https://github.com/oras-project/oras/pull/2218) | 🔴 HIGH | Detect credentials in multiple JSON values | Security / credential redaction |
| [#2194](https://github.com/oras-project/oras/pull/2194) | 🔴 HIGH | Clean up partial pull output on failure | Filesystem safety |
| [#2190](https://github.com/oras-project/oras/issues/2190) | 🔴 HIGH | `pull` leaves corrupted files after digest failure | Filesystem safety *(issue report)*|
| [#2225](https://github.com/oras-project/oras/pull/2225)        | 🟡 MEDIUM     | Restore `--output - --pretty` with bounded buffering | CLI / compatibility|
| [#2183](https://github.com/oras-project/oras/pull/2183) | 🟡 MEDIUM | Fix concurrency in `Tagged` | Go concurrency |
| [#2177](https://github.com/oras-project/oras/pull/2177) | 🟡 MEDIUM | Only infer platform from OCI image configs | OCI semantics |
| [#1482](https://github.com/oras-project/oras-go/pull/1482) | 🟡 MEDIUM | Make credential lookup case-insensitive | API correctness |
| [#2203](https://github.com/oras-project/oras/pull/2203) | 🟢 LOW | Correct manifest fetch format error | CLI correctness |
| [#2214](https://github.com/oras-project/oras/pull/2214) | 🟢 LOW | Fix coverage report generation | Dev tooling |
| [#2207](https://github.com/oras-project/oras/pull/2207) | 🟢 LOW | Update `golang.org/x/crypto` | Dependency maintenance |
| [#2199](https://github.com/oras-project/oras/pull/2199) | 🟢 LOW | Replace `action-publish` with Snapcraft CLI | CI / release infra |
| [#223](https://github.com/vlang/vinix/pull/223) | 🟢 LOW | Fix AArch64 QEMU `mktemp` template | Build environment |
| [#221](https://github.com/vlang/vinix/pull/221) | 🟢 LOW | Disable backtrace for musl desktop | Runtime config |
| [#278](https://github.com/checkpoint-restore/go-criu/pull/278) |⚪MAINTENANCE | Fix typos and grammar | Code quality |

---
## 🔴 HIGH-impact fixes

### [#2222: Limit manifest config fetch size](https://github.com/oras-project/oras/pull/2222)
`oras manifest fetch-config` used to buffer an entire config blob in memory with no size cap. I reproduced this against a local registry serving a 1 GiB+ crafted blob: RSS ballooned to around 2 GiB before the fetch even reached digest verification, which made it a remotely triggerable, unauthenticated DoS. The fix adds a 4 MiB limit and rejects oversized configs before downloading them. The PR went through several review rounds (10 commits, plus a Codecov coverage fix), and I proactively added validation to reject `--pretty` combined with `--output -`. Two smaller output-handling edge cases (special file paths and file permission mode) surfaced after merge and are being tracked in follow-up issues/PRs.

### [#2218: Detect credentials in multiple JSON values](https://github.com/oras-project/oras/pull/2218)
Credential redaction in debug tracing stopped scanning after the first JSON value, so credentials appearing later in a response could leak into debug output. I fixed the detection logic to keep scanning across every JSON value, and added regression tests covering credentials that appear after the first value.

### [#2194: Clean up partial pull output on failure](https://github.com/oras-project/oras/pull/2194)
A failed multi-platform `oras pull` could leave partially written files behind after an output-path collision. Since manifests are pulled concurrently, which file survived depended on timing. I implemented `pullCleanup`, which tracks output paths before writing, distinguishes pre-existing files from ones created by the current run, and removes only what the failed operation created (never pre-existing files). The maintainer review asked for two changes, cleaning up newly created parent dirs and routing cleanup warnings through the logger, and both were implemented. Coverage includes cleanup/path-tracking/duplicate-file tests plus a CLI-level reproduction against a local registry.

### [#2190: `pull` leaves corrupted files after digest failure](https://github.com/oras-project/oras/issues/2190) *(issue report, not a PR)*
I reported and reproduced a case where `oras pull` correctly detected a digest mismatch but left the corrupted blob on disk. To isolate it, I built a deterministic repro using a local HTTP proxy that corrupted blob responses in transit, which pinned the bug to the failed-verification path rather than the artifact itself. Fixed upstream. It's the same underlying class of bug as #2194: failed operations should not leave partial state behind.

---

## 🟡 MEDIUM-impact fixes

### [#2225: Restore `--output - --pretty` with bounded buffering](https://github.com/oras-project/oras/pull/2225)

The security fix in #2222 rejected `oras manifest fetch-config --output - --pretty`, even though the combination was previously supported. The issue was that pretty-printing requires the config to be buffered in memory.

I restored the previous behavior by routing the `--output - --pretty` case through the existing size-limited fetch path instead of the streaming path. Configs larger than the 4 MiB limit are still rejected, while `--output -` without `--pretty` continues to stream directly.

Added command-level regression coverage for pretty output and an oversized-config test proving that the 4 MiB memory limit from #2222 remains enforced.

### [#2183: Fix concurrency in `Tagged`](https://github.com/oras-project/oras/pull/2183)
`Tagged.Tags()` sorted its internal slice while holding only a read lock and returned the internal slice directly. That meant a data race under concurrent access, and callers could also mutate shared state. The fix sorts under an exclusive lock and returns a cloned slice. Verified with concurrent-access tests, slice-isolation tests, and the Go race detector.

### [#2177: Only infer platform from OCI image configs](https://github.com/oras-project/oras/pull/2177)
ORAS inferred platform metadata from any config blob whose JSON happened to contain `os`/`architecture` fields, regardless of the actual media type. So non-image artifacts with similarly shaped config could end up with incorrect platform metadata. I restricted inference to configs with media type `application/vnd.oci.image.config.v1+json`. Reproduced with a non-image artifact and added regression tests for both valid and invalid config types.

### [#1482 (oras-go): Make credential lookup case-insensitive](https://github.com/oras-project/oras-go/pull/1482)
Credential lookup failed whenever hostname casing differed between storing and retrieving (`localhost:5020` vs `LOCALHOST:5020`). I made credential-address matching case-insensitive across all the relevant API functions, while preserving path case-sensitivity and giving exact matches precedence. Verified with targeted and full-suite tests, the race detector, and an independent reproduction through the public Go API.

---

## 🟢 LOW-impact / maintenance

- **[#2203](https://github.com/oras-project/oras/pull/2203)**: `manifest fetch` reported the wrong `--format` value in an error message (it used `opts.Template` instead of `opts.FormatFlag`). Fixed the error path and added regression coverage.
- **[#2214](https://github.com/oras-project/oras/pull/2214)**: `make covhtml` tried to open a coverage report that was never generated. Fixed the target to generate it first.
- **[#2207](https://github.com/oras-project/oras/pull/2207)**: Routine dependency bump for `golang.org/x/crypto`.
- **[#2199](https://github.com/oras-project/oras/pull/2199)**: Replaced the unmaintained `canonical/action-publish` GitHub Action with direct Snapcraft CLI usage in the Snap release workflow, preserving credentials and release-channel behavior.
- **[Vinix #223](https://github.com/vlang/vinix/pull/223)**: Fixed a missing `Xs` template in an AArch64 QEMU `mktemp` call.
- **[Vinix #221](https://github.com/vlang/vinix/pull/221)**: Disabled backtrace support for the musl desktop build where it didn't apply.
