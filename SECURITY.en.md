> Translation of the canonical Chinese document [`SECURITY.md`](SECURITY.md). If they differ, the Chinese version governs.

# Security Policy

## Supported scope

This repository contains documentation and templates only; it provides no runtime. Security fixes target only the latest version of the default branch.

## Reporting a vulnerability

Do not report security issues in a public issue. Use GitHub private vulnerability reporting:

1. Open the repository's **Security** tab.
2. Select **Report a vulnerability**.
3. Describe the affected files, impact, and reproduction method.

Please include:

- Affected files or sections
- Possible leak paths or data impact
- Reproduction steps or a minimal example
- Suggested fix direction (optional)

## Response

The maintainer confirms the scope after receiving a report and states a handling plan. Do not disclose details publicly before a fix is complete.

## Security focus of this repository

- Whether documents contain credentials, private keys, connection strings, or local absolute paths
- Whether example commands could induce dangerous operations
- Whether external links point to trustworthy sources
- Whether the review process can be bypassed by untrusted text injection
- Whether new dependencies or CI permissions are necessary
