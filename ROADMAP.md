# Roadmap

Features and decisions deferred from the hackathon build because they require stronger security design, legal review, or time beyond the Dec 4 deadline. Nothing on this list should be coded before the corresponding review is done.

## Security / cryptographic review needed

- **Shamir share delivery over email.** Current plan sends encrypted shares via Resend. A proper review would assess replay attacks, share interception in transit, and whether email is an acceptable channel for share custody.
- **Grace-period and false-trigger logic.** The rules for when a release is triggered (missed check-ins, reminder cadence, trusted-contact confirmation) need threat modelling before the thresholds are considered safe.
- **Key derivation from passphrase.** If a passphrase-derived key is ever added, the KDF parameters (Argon2id/scrypt work factors) need review for the target device range.
- **Share recombination in the browser.** Confirm that `shamir-secret-sharing` performs recombination in a way that leaves no plaintext secret in memory longer than necessary.

## Legal review needed

- **Jurisdiction-specific estate law.** Whether instructions released by the app have any legal standing (they almost certainly do not without a notarised will). The disclaimer must be reviewed by a qualified attorney before any non-prototype release.
- **Data-protection regulations.** Storing encrypted blobs linked to a person's identity may trigger GDPR, CCPA or similar obligations depending on deployment region.
- **Terms of service for email delivery.** Resend's ToS must be checked for use cases involving posthumous or automated bulk delivery.
- **LLM API data handling.** Even with zero-retention enabled on Groq, the legal implications of sending estate-related content to a third-party API need review before any production use.

## Features not built for the hackathon

- SMS / WhatsApp delivery of shares (cost and complexity beyond the free-tier scope).
- Hardware-security-key (WebAuthn / FIDO2) second factor for vault unlock.
- Printed share cards (PDF generation + QR code per share for offline custody).
- Multi-executor support (more than one AI executor conversation per release event).
- Audit log visible to the owner (what was accessed, when, by which recipient).
- Account deletion and vault wipe flow with certificate of deletion.
- Native mobile app (iOS / Android) for check-in notifications.
- Per-device share submission for release (each Trusted_Contact enters their own share on their own device; the browser aggregates shares without a single coordinator). Current implementation requires one coordinator to collect all k shares manually.
