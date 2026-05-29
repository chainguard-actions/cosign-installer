# Hardening Report: sigstore--cosign-installer/v4.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sigstore--cosign-installer/v4.1.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The Linux/macOS step maps `inputs.install-dir` to the `input_install_dir` env var and then writes it directly to `$GITHUB_PATH` via `envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled `install-dir` input containing newline characters could inject arbitrary entries into GITHUB_PATH, potentially hijacking subsequent PATH lookups.

Locations:

- `action.yml:232`

### github-env-injection (severity: high)

The Windows step maps `inputs.install-dir` to the `input_install_dir` env var and then writes it to `$env:GITHUB_PATH` via PowerShell `Out-File` without any newline sanitization. An attacker-controlled `install-dir` input containing newline characters could inject arbitrary entries into GITHUB_PATH on Windows runners.

Locations:

- `action.yml:239`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection findings in action.yml:

1. Linux/macOS step: Replaced the single-line `envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` with a multi-line run block that sanitizes the value using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_PATH.

2. Windows step: Added `$safe_install_dir = $install_dir -replace '[\r\n]', ''` to strip newlines before writing to $GITHUB_PATH via `Add-Content`, replacing the unsafe `echo "${install_dir}" | Out-File` pattern.

Both fixes prevent attacker-controlled `install-dir` input values containing newline characters from injecting arbitrary entries into GITHUB_PATH.

