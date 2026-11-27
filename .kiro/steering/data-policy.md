---
inclusion: always
---

# Data policy

This file governs what the app stores and how the AI executor accesses data. Read it before adding any new field, feature or AI integration.

## Reference documents

- **[DISCLAIMER.md](../../DISCLAIMER.md)** — the user-facing disclaimer shown during onboarding and before the first vault save. Keep it accurate as the app evolves.
- **[ROADMAP.md](../../ROADMAP.md)** — deferred features that require stronger security design or legal review. If a new feature raises a security or legal question that cannot be resolved in the current build, add it to ROADMAP.md and do not implement it.

## Must-not-store list

Never add a feature or a data field that invites users to store any of the following:

- Passwords or passphrases of any kind (active, archived or "hint" forms).
- PINs, security-question answers, 2FA codes or backup/recovery codes.
- Full payment card numbers or CVVs.
- Government-issued ID numbers (SSN, passport number, driver's licence number, tax ID, etc.).
- Cryptocurrency seed phrases, private keys or recovery codes.
- Medical records or health information.
- Sensitive personal data belonging to other people, unless they have given explicit consent.
- Original signed wills or legally binding documents.

If a proposed feature would naturally collect any of these (e.g. an "import from password manager" flow, a "card on file" field, a "store my 2FA backup codes" prompt), stop, add it to ROADMAP.md under "Security / cryptographic review needed", and do not write the code.

## Security and legal escalation rule

Anything that requires a cryptographic audit, jurisdiction-specific legal review, or a change to the data-retention or deletion model goes into ROADMAP.md, not into the codebase. When in doubt, escalate to ROADMAP.md.

## Disclaimer display rules

The app must show DISCLAIMER.md content (or a faithful summary of it):

1. During onboarding, before the user creates a vault.
2. Before the first vault save in any session.
3. As a persistent footer note on the vault editor screen.

## Input-warning rules

Warn the user (non-blocking, inline) when typed content in any free-text vault field matches any of these patterns:

- Looks like a password or passphrase (high-entropy string, or matches common password-manager export format).
- Looks like a PIN or numeric code (4–8 digit standalone number).
- Looks like a 2FA or backup code (6–8 digit TOTP code, or a block of one-time recovery codes).
- Looks like a payment card number (Luhn-valid 13–19 digit string).
- Looks like a government ID number (common patterns: SSN `XXX-XX-XXXX`, passport `[A-Z]{1,2}[0-9]{6,9}`, etc.).
- Looks like a cryptocurrency seed phrase (12 or 24 space-separated BIP-39 words).

The warning text should read: *"This looks like [a password / a PIN / a 2FA code / a card number / an ID number / a seed phrase]. Last Will.exe is not a secure credential store — see the disclaimer before saving."*

## AI executor data access

The AI executor calls an LLM API at runtime. It must only receive estate-plan entries that the owner has **explicitly marked as shareable with AI**. It must never receive:

- Decrypted vault contents in bulk.
- Any field the owner has not individually marked shareable.
- The encryption key, any Shamir share, or any derived key material.

This rule applies regardless of which LLM provider is active (Groq or Cloudflare Workers AI).
