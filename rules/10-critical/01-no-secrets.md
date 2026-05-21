---
weight: 1
category: critical
description: Never commit, expose, transmit, or log secrets, credentials, keys, tokens, or sensitive data
estimated_tokens: 425
---

# No Secrets Exposure — STRICT

**Highest priority. Never expose secrets, credentials, keys, tokens, or sensitive data.**

## Definition

Secrets: API keys, tokens, SSH/GPG private keys, passwords, OAuth secrets, certificate keys, connection strings with credentials, session tokens, webhook secrets.

Sensitive data: PII, PHI, internal network topology, IPs/hostnames of non-public systems, customer/financial data.

## Forbidden

- Never commit secrets to git
- Never echo secrets to stdout/stderr
- Never log secrets (logs are plaintext)
- Never include secrets in error messages or diagnostic dumps
- Never pass secrets as CLI arguments (visible in process lists)
- Use env vars or dedicated secret stores only

## Deny Rules Enforced (settings.json)

- `Read(**/.env*)`, `Read(**/*.pem)`, `Read(**/*.key)`, `Read(**/*.crt)`
- `Read(**/secrets/**)`, `Read(**/credentials/**)`
- `Read(~/.ssh/**)`, `Read(~/.config/gh/hosts.yml)`
- `Write(~/.ssh/**)`

## Pre-Commit Protection

- `.gitleaks.toml` scans every staged change
- Pre-commit hook blocks any commit with detected secrets
- `--no-verify` does not bypass secret scanning

## If Exposed

1. **Immediately revoke** the credential
1. **Purge from history** (git filter-branch / BFG); consider permanently compromised
1. **Audit access logs** for unauthorized use
1. **Write incident record** to `.agents/docs/error-handling/`
1. Never commit remediation with `--no-verify`

## False Positives

Add `.gitleaksignore` entry for specific file+line. Document why it's a false positive. Never globally disable rules.
