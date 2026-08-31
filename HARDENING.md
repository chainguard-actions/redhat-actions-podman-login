<!-- markdownlint-disable -->

# Hardening Report: redhat-actions--podman-login/v2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **redhat-actions--podman-login/v2.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags instead of full 40-character SHA commit digests. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the upstream repository is compromised.

.github/workflows/ci.yml:
  - uses: actions/checkout@v7 (lines 25, 35, 50)
  - uses: redhat-actions/common/bundle-verifier@v2 (line 39)
  - uses: redhat-actions/common/action-io-generator@v2 (line 53)

.github/workflows/example.yml:
  - uses: actions/checkout@v7 (lines 33, 57, 79)

.github/workflows/link_check.yml:
  - uses: actions/checkout@v7 (line 22)
  - uses: gaurav-nelson/github-action-markdown-link-check@v1 (line 23)

Locations:

- `.github/workflows/ci.yml:25`
- `.github/workflows/ci.yml:35`
- `.github/workflows/ci.yml:39`
- `.github/workflows/ci.yml:50`
- `.github/workflows/ci.yml:53`
- `.github/workflows/example.yml:33`
- `.github/workflows/example.yml:57`
- `.github/workflows/example.yml:79`
- `.github/workflows/link_check.yml:22`
- `.github/workflows/link_check.yml:23`

### script-injection (severity: high)

Sub-rule (a): The workflow example.yml directly interpolates `${{ env.IMAGE_PATH }}` inside run: shell command strings. Even though IMAGE_PATH is set in the workflow's top-level env block, any `${{ ... }}` expression is expanded by the GitHub Actions template engine before the shell ever sees the string, allowing an attacker who can influence the env context to inject arbitrary shell commands. The three offending steps are:
  - Line 48: `run: podman pull ${{ env.IMAGE_PATH }}`
  - Line 71: `run: buildah pull ${{ env.IMAGE_PATH }}`
  - Line 93: `run: docker pull ${{ env.IMAGE_PATH }}`
Fix: move IMAGE_PATH into an `env:` block on the step and reference it as the shell variable `"$IMAGE_PATH"` (double-quoted) instead of using the `${{ }}` expression directly in the run script.

Locations:

- `.github/workflows/example.yml:48`
- `.github/workflows/example.yml:71`
- `.github/workflows/example.yml:93`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Pinned all mutable action tag references to full 40-character SHA digests with tag comments preserved: actions/checkout@v7 → SHA 3d3c42e5aac5ba805825da76410c181273ba90b1 (6 occurrences across ci.yml, example.yml, link_check.yml), redhat-actions/common/bundle-verifier@v2 and redhat-actions/common/action-io-generator@v2 → SHA 19c680ff95a52ee905481b54fc08d5c47788600c (ci.yml), gaurav-nelson/github-action-markdown-link-check@v1 → SHA 5c5dfc0ac2e225883c0e5f03a85311ec2830d368 (link_check.yml). Fixed script injection in example.yml by moving ${{ env.IMAGE_PATH }} out of run: shell strings into step-level env: blocks and referencing as double-quoted shell variable "$IMAGE_PATH" in all three pull steps (podman, buildah, docker).

