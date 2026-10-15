 <div align="center">

# Hey, I'm Tanvir 👋

Software engineer

<br>

<a href="https://contribcard.cncf.io/TanvirTian" target="_blank" rel="noopener noreferrer">
  <img src="https://img.shields.io/badge/CNCF-Contributor-0086FF?style=for-the-badge&logo=cncf&logoColor=white" alt="CNCF Contributor">
</a>

<br>

Go · Python · Docker · PostgreSQL · Redis · Linux

</div>

---

## Open-source contributions

| Project | Repo stars | Merged PRs | Focus |
|---|:-:|:-:|---|
| [**ORAS**](https://github.com/oras-project/oras) | ![](https://img.shields.io/github/stars/oras-project/oras?style=flat-square) | **14** | OCI, security, concurrency, filesystem safety, CLI, CI |
| [**ORAS-Go**](https://github.com/oras-project/oras-go) | ![](https://img.shields.io/github/stars/oras-project/oras-go?style=flat-square) | **2** | API correctness, compatibility, credentials |
| [**Docker CLI**](https://github.com/docker/cli) | ![](https://img.shields.io/github/stars/docker/cli?style=flat-square) | **1** | Build system, error handling |
| [**Vinix**](https://github.com/vlang/vinix) | ![](https://img.shields.io/github/stars/vlang/vinix?style=flat-square) | **2** | AArch64, QEMU, Linux |
| [**go-criu**](https://github.com/checkpoint-restore/go-criu) | ![](https://img.shields.io/github/stars/checkpoint-restore/go-criu?style=flat-square) | **1** | Documentation, code quality |

---

## Security, Reliability & Data Integrity

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2222">#2222</a></b> · Limit manifest config fetch size</summary>

<br>

`oras manifest fetch-config` buffered the entire config blob in memory with no size cap. A malicious or compromised registry could serve an arbitrarily large blob and exhaust client memory. In my repro against a local registry, a 1 GiB+ crafted blob pushed RSS to about 2 GiB before digest verification was even reached.

**Fix:** Enforced a 4 MiB config-size limit, rejecting oversized configs before downloading the full blob.

**Follow-up:** The first version also rejected `--output - --pretty`, which had worked before. I restored it in [#2225](https://github.com/oras-project/oras/pull/2225) with bounded buffering.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2218">#2218</a></b> · Detect credentials in multiple JSON values</summary>

<br>

Credential redaction in debug tracing stopped scanning after the first JSON value, so credentials appearing later in a response could leak into debug output.

**Fix:** Scan every JSON value and hide any credentials found, so they don't appear in debug logs.

**Tests:** Regression cases with credentials placed after the first value.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2194">#2194</a></b> · Clean up partial pull output on failure</summary>

<br>

A failed multi-platform `oras pull` could leave partially written files behind after an output-path collision. Because manifests are pulled concurrently, which file survived depended on timing.

**Fix:** `pullCleanup` tracks output paths before writing, distinguishes pre-existing files from files created by the current run, and removes only what the failed operation created. Newly created parent directories are cleaned up too, and warnings go through the logger.

**Tests:** Cleanup, path-tracking and duplicate-file unit tests, plus a CLI-level repro against a local registry.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/issues/2190">#2190</a></b> · <code>pull</code> leaves corrupted files after digest failure <i>(issue only)</i></summary>

<br>

`oras pull` correctly detected a digest mismatch but left the corrupted blob on disk. I built a deterministic repro with a local HTTP proxy that corrupts blob responses in transit, which pinned the bug to the failed-verification path rather than the artifact. Same class of bug as #2194.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2183">#2183</a></b> · Fix concurrency in <code>Tagged</code></summary>

<br>

`Tagged.Tags()` sorted its internal slice under only a read lock and returned that slice directly, so concurrent callers hit a data race and could mutate shared state.

**Fix:** Use a write lock when sorting and return a copy of the slice.
</details>

---

## CLI Correctness & Developer Experience

<details>
<summary><b>Docker CLI <a href="https://github.com/docker/cli/pull/7373">#7373</a></b> · Handle remote build context download failures</summary>

<br>

Fixed `docker build` to report errors when downloading a remote build context fails instead of continuing without properly surfacing the failure.

**Tests:** Added a regression test to ensure the error is reported and the build does not continue.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2237">#2237</a></b> · Validate extra tags in manifest commands</summary>

<br>

Improved extra-tag validation in manifest push and index create/update commands so invalid extra tags are rejected during command pre-run validation.

**Fix:** Validate extra tags after option parsing.

**Tests:** Added regression tests for invalid option parsing and extra-tag validation.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2230">#2230</a></b> · Fix empty path for digest references</summary>

<br>

Fixed an empty-path error when `oras pull --oci-layout` uses a digest reference without a file path.

**Tests:** Regression tests for empty paths with digest references.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2205">#2205</a></b> · Fix format validation</summary>

<br>

Fixed inconsistent `--format` validation by checking whether a format is supported before applying format-specific handling. This prevents misleading errors from `oras discover --format text` and an unnecessary deprecation warning from `oras repo tags --format`.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2203">#2203</a></b> · Fix manifest fetch format error</summary>

<br>

`manifest fetch` reported the wrong `--format` value in an error message: it used `opts.Template` instead of `opts.FormatFlag`.

**Fix:** Use the correct format flag in the error path.

**Tests:** Added regression coverage for combined output and format flags.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2177">#2177</a></b> · Only infer platform from OCI image configs</summary>

<br>

ORAS inferred platform metadata from any config blob whose JSON contained `os` and `architecture` fields, whatever its media type, so non-image artifacts could get incorrect platform metadata.

**Fix:** Only infer platform for `application/vnd.oci.image.config.v1+json`.

**Tests:** Reproduced with a non-image artifact; regression tests for valid and invalid config types.

</details>

---

## Compatibility & Maintenance

<details>
<summary><b>ORAS-Go <a href="https://github.com/oras-project/oras-go/pull/1555">#1555</a></b> · Migrate directory annotation keys</summary>

<br>

Moved directory annotation keys to the new namespace while keeping older artifacts compatible. Added validation for conflicting annotations that could make v2 and v3 handle the same artifact differently, updated the migration docs, and covered the edge cases with tests.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2225">#2225</a></b> · Restore <code>--output - --pretty</code> with bounded buffering</summary>

<br>

Follow-up to #2222. The first fix over-restricted `oras manifest fetch-config --output - --pretty`, which pretty-printing needs buffered in memory.

**Fix:** Route that combination through the existing size-limited fetch path. Configs over 4 MiB are still rejected, and `--output -` without `--pretty` still streams directly.

**Tests:** Command-level coverage for pretty output, and an oversized-config test proving the limit from #2222 still holds.

</details>

<details>
<summary><b>ORAS-Go <a href="https://github.com/oras-project/oras-go/pull/1482">#1482</a></b> · Make credential lookup case-insensitive</summary>

<br>

Credential lookup failed when hostname casing differed between storing and retrieving (`localhost:5020` vs `LOCALHOST:5020`).

**Fix:** Case-insensitive address matching across the relevant API functions, with path case-sensitivity preserved and exact matches taking precedence.

**Verified with:** Targeted and full-suite tests, the race detector, and an independent repro through the public Go API.

</details>
<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2234">#2234</a></b> · Update dependencies</summary>

<br>

Updated `oras.land/oras-go/v2` from `v2.6.2` to `v2.6.3` and `golang.org/x/sync` to `v0.24.0`. Resolved dependency conflicts with upstream `main` while preserving newer dependency versions.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2214">#2214</a></b> · Fix coverage report generation</summary>

<br>

Fixed `make covhtml` trying to open a coverage report that had not been generated. Updated the target to generate the report first.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2207">#2207</a></b> · Update <code>golang.org/x/crypto</code></summary>

<br>

Updated the `golang.org/x/crypto` dependency.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2199">#2199</a></b> · Replace deprecated publishing action</summary>

<br>

Replaced the unmaintained `canonical/action-publish` GitHub Action with direct Snapcraft CLI usage in the Snap release workflow, preserving credentials and release-channel behavior.

</details>

<details>
<summary><b>Vinix <a href="https://github.com/vlang/vinix/pull/223">#223</a></b> · Fix AArch64 QEMU temporary-file template</summary>

<br>

Fixed a missing `Xs` template in an AArch64 QEMU `mktemp` call.

</details>

<details>
<summary><b>Vinix <a href="https://github.com/vlang/vinix/pull/221">#221</a></b> · Disable unsupported musl backtrace support</summary>

<br>

Disabled backtrace support for the musl desktop build where it did not apply.

</details>

<details>
<summary><b>go-criu <a href="https://github.com/checkpoint-restore/go-criu/pull/278">#278</a></b> · Fix typos and grammar</summary>

<br>

Corrected typos and grammatical issues in the project documentation.

</details>

---

<div align="center">

> "You miss every shot you never take."

### 📫 Let's Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/tanvirtian)

</div>
