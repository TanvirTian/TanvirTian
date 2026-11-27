 <div align="center">

# Hey, I'm Tanvir 👋

Software engineer

<br>

<a href="https://contribcard.cncf.io/TanvirTian" target="_blank" rel="noopener noreferrer">
  <img src="https://img.shields.io/badge/CNCF-Contributor-0086FF?style=for-the-badge&logo=cncf&logoColor=white" alt="CNCF Contributor">
</a>

<br>

Go · Python · Docker · PostgreSQL · Redis · Linux

<img src="koi/koi.gif" width="256" alt="GitHub contribution koi">

</div>

---

## Open-source contributions

| Project | Repo stars | Merged PRs | Focus |
|---|:-:|:-:|---|
| [**ORAS**](https://github.com/oras-project/oras) | **2.5k** | **24** | OCI, security, concurrency, filesystem safety, CLI, CI |
| [**ORAS-Go**](https://github.com/oras-project/oras-go) | **301** | **6** | API correctness, compatibility, credentials |
| [**Docker CLI**](https://github.com/docker/cli) | **6.1k** | **1** | Build system, error handling |
| [**Vinix**](https://github.com/vlang/vinix) | **2.7k** | **2** | AArch64, QEMU, Linux |
| [**go-criu**](https://github.com/checkpoint-restore/go-criu) | **257** | **1** | Documentation, code quality |
| [**Moby**](https://github.com/moby/moby) | **72.3k** | **1** | Documentation |
| [**pip**](https://github.com/pypa/pip) | **10.3k** | **1** | CLI correctness, argument validation, testing |
| [**gVisor**](https://github.com/google/gvisor) | **20.1k** | **1** | Documentation |

---

**Want to see more?** Check out [all my merged pull requests](https://github.com/pulls?q=is%3Apr+is%3Amerged+author%3Atanvirtian).

## Security, Reliability & Data Integrity

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2222">#2222</a></b> · Limit manifest config fetch size</summary>

<br>

`oras manifest fetch-config` buffered the entire config blob in memory without a size limit. A malicious registry could serve an arbitrarily large blob and exhaust client memory.

**Fix:** Enforced a 4 MiB config-size limit, rejecting oversized configs before downloading the full blob.

**Follow-up:** Restored `--output - --pretty` support with bounded buffering in [#2225](https://github.com/oras-project/oras/pull/2225).

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2218">#2218</a></b> · Detect credentials in multiple JSON values</summary>

<br>

Credential redaction stopped scanning after the first JSON value, potentially exposing credentials in later values in debug logs.

**Fix:** Scan every JSON value and redact credentials wherever they appear.

**Tests:** Added regression tests for credentials appearing after the first value.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2194">#2194</a></b> · Clean up partial pull output on failure</summary>

<br>

A failed multi-platform `oras pull` could leave partially written files behind after an output-path collision. Concurrent pulls also made the result dependent on timing.

**Fix:** Track output paths, distinguish pre-existing files from newly created files, and clean up only files and directories created by the failed operation.

**Tests:** Added cleanup, path-tracking, and duplicate-file tests, plus a CLI-level reproduction.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2183">#2183</a></b> · Fix concurrency in <code>Tagged</code></summary>

<br>

`Tagged.Tags()` modified its internal slice while holding only a read lock and returned the shared slice directly.

**Fix:** Use a write lock when sorting and return a copy to prevent concurrent access and external mutation issues.

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
<summary><b>PIP <a href="https://github.com/pypa/pip/pull/14391">#14391</a></b> · Reject unexpected positional arguments</summary>

<br>

Several pip commands silently ignored extra positional arguments, which could make users think their input had been accepted.

**Fix:** Made `pip check`, `pip completion`, `pip debug`, `pip freeze`, `pip inspect`, and `pip list` reject unexpected arguments with clear errors.

**Tests:** Added regression coverage for all six commands.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2378">#2378</a></b> · Validate OCI layout tags</summary>

<br>

Invalid bare tags could be accepted when pushing artifacts to OCI layouts.

**Fix:** Validate bare tags for `--oci-layout` while preserving raw reference names for `--oci-layout-path`.

**Tests:** Added regression coverage for invalid tags, registry ports, uppercase references, and special characters.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2353">#2353</a></b> · Fix repository tag filtering in OCI layouts</summary>

<br>

`oras repo tags --oci-layout-path` could show tags from other repositories when given a tagged reference. It also failed to find existing manifests with repository-scoped digest references.

**Fix:** Filter tags by repository for both tagged and digest references.

**Tests:** Added unit and end-to-end tests.

</details>

<details>
<summary><b>ORAS <a href="https://github.com/oras-project/oras/pull/2304">#2304</a></b> · Require digest references for blob commands</summary>

<br>

`oras blob fetch` could return an entire manifest instead of a blob, while `oras blob delete` could accidentally delete a manifest when given a tag in an OCI image layout.

**Fix:** Require digest references for both commands and return clear errors when tags are supplied.

**Tests:** Added unit and end-to-end regression tests.

</details>

<details>
<summary><b>ORAS <a href="PR_LINK">#PR_NUMBER</a></b> · Avoid creating OCI layouts during deletion</summary>

<br>

Deleting a blob or manifest from a missing or invalid OCI layout could unexpectedly create the layout.

**Fix:** Validate the existing OCI layout before deletion without creating or modifying files when the layout is missing or invalid.

**Tests:** Added coverage for missing remote CA files, invalid layouts, and successful blob and manifest deletion.

</details>

---

## Compatibility & Maintenance

<details>
<summary><b>ORAS-Go <a href="https://github.com/oras-project/oras-go/pull/1667">#1667</a></b> · Fix authentication after HTTPS upgrades</summary>

<br>

Authentication could fail when a registry redirected a request from HTTP to HTTPS on the same host because the scheme change affected origin validation.

**Fix:** Added same-host HTTPS-upgrade authentication handling while preserving safeguards against unsafe redirect targets.

**Tests:** Added coverage for Basic and Bearer authentication, redirect boundaries, credential-cache behavior, and unsafe HTTPS upgrades.

</details>

<details>
<summary><b>ORAS-Go <a href="https://github.com/oras-project/oras-go/pull/1482">#1482</a></b> · Make credential lookup case-insensitive</summary>

<br>

Credential lookup failed when hostname casing differed between storing and retrieving credentials.

**Fix:** Added case-insensitive address matching while preserving path case-sensitivity and exact-match precedence.

**Verified with:** Targeted and full-suite tests, the race detector, and an independent reproduction through the public Go API.

</details>

<details>
<summary><b>ORAS-Go <a href="https://github.com/oras-project/oras-go/pull/1555">#1555</a></b> · Migrate directory annotation keys</summary>

<br>

Moved directory annotation keys to the new namespace while keeping older artifacts compatible.

**Fix:** Added validation for conflicting annotations and updated migration documentation.

**Tests:** Added coverage for migration and compatibility edge cases.

</details>

---

<div align="center">

> "You miss every shot you never take."

### 📫 Let's Connect

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=for-the-badge&logo=linkedin)](https://linkedin.com/in/tanvirtian)

</div>
