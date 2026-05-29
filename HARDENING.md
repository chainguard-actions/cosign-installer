# Hardening Report: sigstore--cosign-installer/v4.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sigstore--cosign-installer/v4.1.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Two steps write the attacker-controlled input `inputs.install-dir` to `$GITHUB_PATH` via an env var (`input_install_dir`) without the required sanitization (`printf '%s' ... | tr -d '\n\r'`) before the write.

1. Linux/macOS step (line 228): `run: envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` — the value from `inputs.install-dir` is expanded by `envsubst` and appended directly to GITHUB_PATH with no newline stripping.

2. Windows step (line 236): `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append` — the value from `inputs.install-dir` is expanded and written to GITHUB_PATH with no sanitization.

An attacker who controls the `install-dir` input can inject newlines to add arbitrary entries to GITHUB_PATH, potentially hijacking PATH-based binary lookups in subsequent steps.

Locations:

- `action.yml:228`
- `action.yml:236`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two GITHUB_PATH injection vulnerabilities in action.yml:

1. Linux/macOS step (was line 228): Changed from a single-line `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` to a multi-line block that pipes envsubst output through `tr -d '\n\r'` to strip newlines before writing to GITHUB_PATH via `printf '%s\n'`.

2. Windows step (was line 236): Changed from `echo "${install_dir}" | Out-File ...` to first sanitize the expanded value using PowerShell's `-replace '[\r\n]', ''` regex replacement, then write the sanitized value to GITHUB_PATH.

Both fixes prevent an attacker who controls the `inputs.install-dir` input from injecting newlines to add arbitrary entries to GITHUB_PATH.

