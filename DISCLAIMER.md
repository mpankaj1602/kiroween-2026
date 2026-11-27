# Disclaimer

**Last Will.exe is a hackathon prototype. It is not a legal, financial, estate-planning or security product.**

## What this app is not

- Not a substitute for a legally valid will, trust, power of attorney or any other estate-planning document.
- Not a licensed fiduciary, attorney, financial adviser or executor.
- Not a certified secure password manager, secrets vault or credential store.

## What you should store in this app

Last Will.exe is designed for instructions and wishes that help your family act after you are gone — not for secrets that could harm you if exposed today. Good things to store:

- Personal letters to family, friends or colleagues.
- A list of your online accounts with what to do with each (close, memorialise, transfer or keep).
- Where important physical documents are kept (will, deeds, insurance policies, passport).
- Helpful contacts: your lawyer, accountant, insurer, financial adviser.
- Personal wishes: funeral preferences, organ donation, what to do with pets or collections.
- Where your password manager's emergency-access document is kept (e.g. “my 1Password Emergency Kit PDF is in the fireproof box in the study”, “Bitwarden Emergency Access is set up with my partner’s email”) — tell your family *where to find* the document, never note its contents here, because that document contains your Secret Key.

## Data you must not store in this app

The following must **never** be entered into the vault, regardless of encryption:

- Passwords or passphrases of any kind (active, archived or "hint" forms).
- PINs, security-question answers, 2FA codes or backup/recovery codes.
- Full payment card numbers or CVVs.
- Government-issued ID numbers (SSN, passport number, driver's licence number, tax ID, etc.).
- Cryptocurrency seed phrases, private keys or recovery codes.
- Medical records or health information.
- Sensitive personal data belonging to other people, unless you have their explicit consent.
- Original signed wills or legally binding documents (store the physical original with a solicitor or in a safe; note its location here instead).

For credential storage, use a dedicated, audited secrets manager (e.g. 1Password, Bitwarden, hardware wallet) and use that manager's own emergency-access feature to give your family access.

## Encryption caveats

Vault contents are encrypted in the browser with AES-GCM before leaving your device. The encryption key is split with Shamir's secret sharing and distributed to recipients of your choosing. However:

- The app has not undergone a professional cryptographic audit.
- Key management, share delivery and the release flow involve attack surfaces that have not been independently reviewed.
- Items marked as shareable with the AI executor are sent in plaintext to a third-party LLM API (Groq or Cloudflare Workers AI). Do not mark sensitive data shareable with AI.

## No guarantees

The app is provided as-is, with no warranty of fitness for any purpose. The authors accept no liability for data loss, misdelivery, premature or failed release, or any harm arising from use of this software.

## Legal advice

Consult a qualified estate attorney in your jurisdiction before making any decisions about digital legacy, asset transfer or end-of-life planning.
