# Hardening Report: sigstore--cosign-installer/v4.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sigstore--cosign-installer/v4.1.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The Linux/macOS step writes the attacker-controllable `inputs.install-dir` value (via env var `input_install_dir`) to `$GITHUB_PATH` using `envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious value containing newlines could inject arbitrary entries into the runner's PATH.

Locations:

- `action.yml:282`

### github-env-injection (severity: high)

The Windows (pwsh) step writes the attacker-controllable `inputs.install-dir` value (via env var `input_install_dir`) to `$env:GITHUB_PATH` using `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH` without the required sanitization step. A malicious value containing newlines could inject arbitrary entries into the runner's PATH.

Locations:

- `action.yml:289`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection findings in action.yml:

1. Linux/macOS step (line ~282): Replaced `envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` with a sanitized version that pipes through `tr -d '\n\r'` before writing to $GITHUB_PATH, preventing newline injection.

2. Windows (pwsh) step (line ~289): Replaced `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH` with a sanitized version that uses PowerShell's `-replace '[\r\n]', ''` to strip newlines before writing to $GITHUB_PATH, preventing newline injection.

