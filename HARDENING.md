# Hardening Report: sigstore--cosign-installer/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sigstore--cosign-installer/v4.1.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Two steps write the attacker-controlled `inputs.install-dir` value to `$GITHUB_PATH` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). The value is routed through the `input_install_dir` env var but is never sanitized before the write. On Linux/macOS (step 2): `envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"`. On Windows (step 3): `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH`. An attacker can supply a newline-containing value for `inputs.install-dir` to inject arbitrary entries into `GITHUB_PATH`, potentially hijacking subsequent tool lookups.

Locations:

- `action.yml:183`
- `action.yml:190`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection vulnerabilities in action.yml:

1. Linux/macOS step (line ~183): Changed from `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` to a multi-line run that sanitizes the value: `safe_install_dir=$(envsubst <<<"${input_install_dir}" | tr -d '\n\r')` followed by `printf '%s\n' "${safe_install_dir}" >> "$GITHUB_PATH"`. This strips any newline characters that an attacker could inject via the `inputs.install-dir` value.

2. Windows step (line ~190): Changed from `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append` to use `$safe_install_dir = $install_dir -replace '[\r\n]', ''` followed by `Add-Content -Path $env:GITHUB_PATH -Value $safe_install_dir -Encoding utf8`. This strips carriage returns and newlines before writing to $GITHUB_PATH.

Both fixes prevent an attacker from injecting arbitrary entries into GITHUB_PATH by supplying a newline-containing value for `inputs.install-dir`.

