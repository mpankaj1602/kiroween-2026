# Requirements Document

## Introduction

Last Will.exe is a digital-legacy dead man's switch. An owner signs in with an email magic link, writes vault
entries (personal letters, account instructions, wishes, contacts), and configures a check-in schedule. The vault
contents are encrypted in the browser with AES-GCM. The encryption key is split using Shamir's secret sharing and
each trusted contact receives one share by email. If the owner stops checking in, a grace-period window elapses,
and the vault ciphertext is released to contacts, who each enter their share in the browser; when k shares are
combined the vault decrypts locally.

This document covers three tightly coupled slices:

1. **Authentication** — email magic-link sign-in via Resend.
2. **Vault management** — creating, editing, and encrypting entries; disclaimer enforcement; input warnings.
3. **Trusted contacts and secret sharing** — share generation, distribution, and threshold configuration.
4. **Check-in scheduler** — owner heartbeat, reminder cadence, grace period.
5. **Release flow** — ciphertext release, share submission, browser-side decryption.
6. **Demo mode** — time-compressed cycle for a ~1-minute end-to-end demonstration.

Out of scope: AI executor feature; spooky UI theme.

---

## Glossary

- **Owner**: the authenticated user who creates and manages the vault.
- **App**: the Last Will.exe web application (React frontend + Hono/Cloudflare Worker backend).
- **Vault**: the collection of all encrypted entries belonging to one owner.
- **Entry**: a single vault record (letter, account instruction, wish, or contact note) composed of a title, body
  text, and metadata.
- **Encrypted_Share**: an AES-GCM ciphertext stored in D1 per Trusted_Contact, produced by encrypting the plaintext Share with the Vault_Key in the browser; allows the Owner to re-send a share without re-splitting, while keeping the plaintext Share off the server.
- **Ciphertext**: an AES-GCM encrypted blob stored on the server; the server never holds the plaintext.
- **Vault_Key**: a 256-bit AES-GCM symmetric key generated in the browser and used to encrypt all entries.
- **Share**: one fragment of the Vault_Key produced by Shamir's secret sharing.
- **Threshold (k)**: the minimum number of shares required to reconstruct the Vault_Key.
- **Trusted_Contact**: a person designated by the Owner to hold one Share and participate in the release flow.
- **Check_In**: an action the Owner performs periodically to signal they are still alive.
- **Check_In_Interval**: the period (in days, or in demo-mode seconds) between required Check_Ins.
- **Grace_Period**: a window after the final missed-check-in reminder during which no release occurs; gives the
  Owner a last chance to check in.
- **Release_Event**: the server transition that makes the ciphertext available to Trusted_Contacts.
- **State_Machine**: the server-side finite automaton governing the check-in lifecycle.
- **Demo_Mode**: an operational mode that scales all time intervals from days to seconds.
- **Magic_Link**: a single-use, time-limited URL sent by email to authenticate the Owner.
- **PBKDF2_Salt**: a 16-byte cryptographically random value generated once per vault and stored alongside the wrapped Vault_Key on the server.
- **Shamir_Library**: the `shamir-secret-sharing` npm package used for all share operations.
- **Web_Crypto**: the browser's native `window.crypto.subtle` API used for all AES-GCM operations.
- **Wrapping_Key**: an AES-KW key derived from the Owner's passphrase via PBKDF2-SHA256; used solely to wrap and unwrap the Vault_Key.
- **Resend**: the third-party email delivery service used for magic links, check-in reminders, and share delivery.
- **D1**: Cloudflare D1 SQLite database used to persist accounts, vault blobs, check-in state, and contact records.
- **Cron**: Cloudflare Cron Trigger that fires the check-in evaluation job on a fixed schedule.
- **AI_Flag**: a boolean property on each Entry indicating whether the Owner has explicitly marked it as shareable
  with the AI executor.

---

## Requirements

### Requirement 1: Email Magic-Link Authentication

**User Story:** As an Owner, I want to sign in with an email magic link, so that I do not need to create or
remember a password.

#### Acceptance Criteria

1. WHEN the Owner submits a valid email address on the sign-in page, THE App SHALL send a Magic_Link to that
   address via Resend within 10 seconds.
2. WHEN the Owner clicks a valid, unexpired Magic_Link, THE App SHALL create an authenticated session and redirect
   the Owner to the vault editor.
3. IF a Magic_Link has already been used or is older than 15 minutes, THEN THE App SHALL reject it and display the
   message "This link has expired or already been used. Please request a new one."
4. THE App SHALL generate each Magic_Link token with at least 128 bits of cryptographic randomness.
5. THE App SHALL store Magic_Link tokens hashed (SHA-256) in D1 and SHALL NOT store the plaintext token.
6. WHILE an Owner session is active, THE App SHALL include a session token in every authenticated request and
   SHALL reject requests that omit or present an invalid session token with HTTP 401.

---

### Requirement 2: Disclaimer Enforcement

**User Story:** As an Owner, I want to see a clear disclaimer about what this app is and is not before I store
anything, so that I understand its limitations.

#### Acceptance Criteria

1. WHEN the Owner completes sign-in for the first time, THE App SHALL display the full DISCLAIMER.md content
   before the vault editor is shown, and SHALL require the Owner to click an explicit acknowledgement button
   before proceeding.
2. WHEN the Owner opens the vault editor in any session, THE App SHALL display a persistent, non-dismissible
   disclaimer summary in the page footer.
3. WHEN the Owner attempts to save a vault entry for the first time in a session, THE App SHALL display a
   non-blocking reminder that links to the full disclaimer before the save is confirmed.
4. THE App SHALL NOT require the Owner to re-read the full disclaimer on every session after the initial
   acknowledgement; the footer summary satisfies subsequent sessions.

---

### Requirement 3: Vault Entry Management

**User Story:** As an Owner, I want to create, edit, and delete vault entries of several types, so that I can
leave complete instructions and messages for my family.

#### Acceptance Criteria

1. THE App SHALL support the following entry types: Personal Letter, Account Instructions, Personal Wish, and
   Contact Note.
2. WHEN the Owner saves an entry, THE App SHALL encrypt the entry title and body using Web_Crypto AES-GCM with
   the Vault_Key before transmitting any data to the server.
3. THE App SHALL transmit only Ciphertext and entry metadata (entry ID, type label, creation timestamp,
   AI_Flag) to the server; plaintext SHALL NOT leave the browser.
4. WHEN the Owner updates an existing entry, THE App SHALL re-encrypt the full entry with the current Vault_Key
   and replace the stored Ciphertext in D1.
5. WHEN the Owner deletes an entry, THE App SHALL remove the corresponding Ciphertext from D1 and confirm
   deletion to the Owner.
6. THE App SHALL support a minimum of 50 entries per vault.
7. THE App SHALL allow the Owner to set the AI_Flag to true on any entry, and SHALL display a visual indicator
   on entries where AI_Flag is true.

---

### Requirement 4: Input Warning for Sensitive Data

**User Story:** As an Owner, I want an inline warning when I type data that looks sensitive, so that I am reminded
this app is not a secure credential store.

#### Acceptance Criteria

1. WHILE the Owner is typing in any vault entry text field, THE App SHALL evaluate the current field value
   against six pattern classes: passwords/passphrases (high-entropy strings or common password-manager export
   format), PINs (4–8 standalone digits), 2FA or backup codes (6–8 digit TOTP codes or one-time recovery code
   blocks), payment card numbers (Luhn-valid 13–19 digit strings), government ID numbers (SSN `XXX-XX-XXXX`,
   passport `[A-Z]{1,2}[0-9]{6,9}`, and equivalent common patterns), and cryptocurrency seed phrases (12 or 24
   space-separated BIP-39 words).
2. WHEN a vault entry field value matches any of the six pattern classes, THE App SHALL display the inline,
   non-blocking warning: "This looks like [a password / a PIN / a 2FA code / a card number / an ID number / a
   seed phrase]. Last Will.exe is not a secure credential store — see the disclaimer before saving."
3. THE App SHALL display the warning without blocking the Owner from continuing to type or saving the entry.
4. WHEN the field value no longer matches any pattern class, THE App SHALL remove the warning.
5. THE App SHALL evaluate field values client-side only; pattern-matching data SHALL NOT be sent to the server.

---

### Requirement 5: Vault Key Generation and Storage

**User Story:** As an Owner, I want my vault key generated and held only in my browser, so that the server can
never read my vault without my contacts' shares.

#### Acceptance Criteria

1. WHEN the Owner creates a vault for the first time, THE App SHALL generate a Vault_Key using Web_Crypto
   `crypto.subtle.generateKey` with algorithm AES-GCM and key length 256 bits.
2. THE App SHALL NOT transmit the plaintext Vault_Key to the server at any time.
3. WHEN the Owner sets a vault passphrase, THE App SHALL derive a wrapping key using Web_Crypto `crypto.subtle.deriveKey` with PBKDF2-SHA256, a randomly generated 16-byte salt, and at least 600,000 iterations, and SHALL use that wrapping key to wrap the Vault_Key with AES-KW; only the wrapped Vault_Key and the salt SHALL be stored on the server; the plaintext passphrase and the plaintext Vault_Key SHALL NOT be transmitted or stored server-side.
4. WHEN the Owner's session ends, THE App SHALL discard the in-memory Vault_Key.
5. WHEN the Owner opens a new session, THE App SHALL retrieve the wrapped Vault_Key and salt from the server, prompt the Owner for their vault passphrase, derive the wrapping key with PBKDF2-SHA256 (same salt, same iteration count), and unwrap the Vault_Key in the browser using Web_Crypto; the Vault_Key SHALL be held in session memory only and discarded when the session ends.
6. IF the PBKDF2 unwrap fails (wrong passphrase), THE App SHALL display "Incorrect passphrase. Please try again." and SHALL NOT expose any partial key material.

---

### Requirement 6: Trusted Contacts and Shamir Share Distribution

**User Story:** As an Owner, I want to designate 3–5 trusted contacts and distribute a Shamir share to each, so
that my vault can only be opened when enough of them cooperate.

#### Acceptance Criteria

1. THE App SHALL require the Owner to designate between 3 and 5 Trusted_Contacts, each identified by a unique
   email address.
2. THE App SHALL require the Owner to set a Threshold k where 2 ≤ k ≤ n and n is the number of
   Trusted_Contacts.
3. WHEN the Owner confirms the contact list and threshold, THE App SHALL use the Shamir_Library to split the
   Vault_Key into n shares with threshold k.
4. THE App SHALL send each Trusted_Contact exactly one Share via Resend with instructions explaining what it is
   and how to use it during a release.
5. THE App SHALL store a record per contact in D1 containing: email address, a delivery-status flag, and the Share encrypted with the Vault_Key using AES-GCM (Encrypted_Share); it SHALL NOT store the plaintext Share; the server can never read the Share because it never holds the Vault_Key.
6. IF Resend returns a non-2xx status for a share delivery email, THEN THE App SHALL mark that contact's
   delivery-status as failed and display an alert to the Owner with the affected email address.
7. THE App SHALL allow the Owner to re-send a share email to a specific contact without regenerating all shares; to do so, THE App SHALL retrieve the contact's Encrypted_Share from D1, decrypt it in the browser using the in-session Vault_Key, and send the plaintext Share to the contact via Resend; the plaintext Share SHALL NOT be sent to or stored on the server during this operation.
8. WHEN the Owner changes the contact list or the Threshold k, THE App SHALL generate a new Vault_Key, re-encrypt every existing vault entry with the new key, re-split the new key into n shares using the Shamir_Library, re-wrap the new Vault_Key with the Owner's passphrase-derived wrapping key, store the updated wrapped Vault_Key on the server, and email each contact their new share; previously issued shares for removed or changed contacts SHALL become cryptographically useless.

---

### Requirement 7: Owner Vault Unlock via Passphrase

**User Story:** As an Owner, I want the vault key unwrapped in my browser from my passphrase, so that I can access my vault in a new session without depending on Trusted Contacts.

#### Acceptance Criteria

1. WHEN the Owner opens a new authenticated session, THE App SHALL prompt the Owner to enter their vault passphrase before the vault editor is shown.
2. WHEN the Owner submits their passphrase, THE App SHALL retrieve the wrapped Vault_Key and PBKDF2_Salt from the server, run PBKDF2-SHA256 with the retrieved salt and the configured iteration count, and attempt AES-KW unwrap of the Vault_Key in the browser.
3. WHEN the AES-KW unwrap succeeds, THE App SHALL hold the Vault_Key in browser session memory only and proceed to the vault editor.
4. IF the AES-KW unwrap fails, THE App SHALL display "Incorrect passphrase. Please try again." and SHALL NOT expose any partial key material.
5. THE App SHALL perform all passphrase derivation and key-unwrap operations in the browser; the passphrase and the plaintext Vault_Key SHALL NOT be sent to the server.

---

### Requirement 8: Check-In Scheduler Configuration

**User Story:** As an Owner, I want to set a check-in interval, so that the app knows how often to expect a
sign of life from me.

#### Acceptance Criteria

1. THE App SHALL allow the Owner to set a Check_In_Interval of 7, 14, 30, 60, or 90 days.
2. THE App SHALL record the Owner's chosen Check_In_Interval and the timestamp of the last successful
   Check_In in D1.
3. WHEN the Owner saves a Check_In_Interval, THE App SHALL send a confirmation email via Resend summarising
   the schedule and the first check-in due date.
4. WHERE Demo_Mode is enabled, THE App SHALL substitute one day with one second for all interval and
   grace-period calculations.

---

### Requirement 9: Check-In Action

**User Story:** As an Owner, I want to check in by clicking a link in a reminder email, so that the app knows
I am still alive.

#### Acceptance Criteria

1. WHEN the Owner clicks a valid, unexpired check-in link, THE App SHALL record the check-in timestamp in D1,
   reset the State_Machine to ACTIVE, and display a confirmation page.
2. THE App SHALL generate each check-in link token with at least 128 bits of cryptographic randomness and
   invalidate it after a single use.
3. IF a check-in link is older than the Check_In_Interval plus 48 hours, THEN THE App SHALL reject it and
   display "This check-in link has expired. No check-in was recorded."
4. THE App SHALL allow the Owner to check in from the authenticated vault editor at any time, without waiting
   for a reminder email, by clicking a "Check In Now" button.

---

### Requirement 10: Missed Check-In Detection and Reminder Flow

**User Story:** As an Owner, I want the app to send reminders before escalating to a release, so that
accidental triggers are prevented.

#### Acceptance Criteria

1. WHEN the Cron job fires and the Owner's last Check_In timestamp is older than their Check_In_Interval, THE
   App SHALL transition the State_Machine from ACTIVE to OVERDUE and send a first reminder email via Resend.
2. WHEN the Cron job fires and the State_Machine is OVERDUE and 48 hours have elapsed since the first reminder,
   THE App SHALL send a second reminder email and transition to FINAL_REMINDER.
3. WHEN the Cron job fires and the State_Machine is FINAL_REMINDER and 48 hours have elapsed since the second
   reminder, THE App SHALL transition to GRACE and record the Grace_Period start timestamp.
4. THE App SHALL set the Grace_Period duration equal to the Owner's Check_In_Interval (e.g. a 7-day interval
   gives a 7-day Grace_Period).
5. WHILE the State_Machine is in GRACE, THE App SHALL send one additional reminder email to the Owner per day.
6. WHEN the Owner performs a successful Check_In while the State_Machine is in OVERDUE, FINAL_REMINDER, or
   GRACE, THE App SHALL transition to ACTIVE and cancel any pending release.

---

### Requirement 11: Release Trigger

**User Story:** As an Owner, I want the release to happen automatically only after all reminders and the grace
period pass, so that the vault is never released prematurely.

#### Acceptance Criteria

1. WHEN the Cron job fires and the State_Machine is GRACE and the Grace_Period has fully elapsed, THE App
   SHALL transition to RELEASED and record the release timestamp.
2. THE App SHALL NOT transition to RELEASED before the Grace_Period start timestamp plus the Grace_Period
   duration has been reached.
3. WHEN the State_Machine transitions to RELEASED, THE App SHALL send a release notification email to every
   Trusted_Contact containing a unique, single-use release link.
4. IF Resend returns a non-2xx status for any release notification email, THEN THE App SHALL log the failure
   in D1 and retry delivery up to 3 times with exponential back-off before marking that contact's
   notification as permanently failed.
5. THE App SHALL NOT expose the Ciphertext to any Trusted_Contact before the State_Machine is RELEASED.
6. WHERE Demo_Mode is enabled, THE App SHALL calculate Grace_Period elapsed time in seconds instead of days.

---

### Requirement 12: Trusted Contact Release Portal

**User Story:** As a Trusted Contact, I want to submit my share on a release page and receive the decrypted
vault, so that I can carry out the Owner's wishes.

#### Acceptance Criteria

1. WHEN a Trusted_Contact clicks a valid release link, THE App SHALL display the share submission page and
   SHALL validate that the State_Machine is RELEASED before serving the page.
2. IF the State_Machine is not RELEASED when a release link is accessed, THEN THE App SHALL return an error
   page stating "The vault has not been released."
3. THE App SHALL allow one coordinating Trusted_Contact to collect k shares in their own browser session; each of the other k−1 contacts SHALL convey their share to the coordinator by a channel of their choice (the app displays their share on screen for them to copy); the coordinator enters all k shares into the release portal, and the browser reconstructs the Vault_Key from those k shares.
4. WHEN k or more valid Shares have been submitted, THE App SHALL reconstruct the Vault_Key client-side using
   the Shamir_Library and decrypt all Ciphertext entries using Web_Crypto; plaintext SHALL NOT be sent to the
   server during reconstruction or decryption.
5. WHEN the Vault_Key is successfully reconstructed and all entries are decrypted, THE App SHALL display the
   vault contents to the submitting Trusted_Contacts.
6. THE App SHALL invalidate each release link after it has been used to submit a share, to prevent replay
   submission.

---

### Requirement 13: Demo Mode

**User Story:** As a presenter, I want a demo mode that compresses all time intervals so the full cycle runs in
about one minute, so that I can show the complete check-in-to-release flow in a 3-minute demo video.

#### Acceptance Criteria

1. THE App SHALL provide a Demo_Mode toggle accessible to the Owner in the account settings.
2. WHERE Demo_Mode is enabled, THE App SHALL route all outgoing emails (magic links, check-in reminders, share delivery emails, release notifications) to an in-app outbox displayed on screen instead of sending them via Resend; the outbox SHALL show the recipient address, subject, and full body of each email.
3. WHERE Demo_Mode is enabled, THE App SHALL substitute 1 second for 1 day in all interval and grace-period calculations, compressing a standard 7-day-interval cycle to approximately 25 seconds: 7-day interval (7 s) + first-reminder gap 2 days (2 s) + second-reminder gap 2 days (2 s) + grace period 7 days (7 s) = 18 s, plus Cron polling overhead, targeting under 60 seconds end-to-end.
4. WHERE Demo_Mode is enabled, THE App SHALL display a persistent, clearly visible banner reading "DEMO MODE —
   time is compressed" on every screen.
5. WHERE Demo_Mode is enabled, THE App SHALL allow the Cron-driven state transitions to fire via a manual
   "Advance Clock" button so a live demo does not depend on actual Cron timing.
6. WHERE Demo_Mode is enabled, THE App SHALL suppress the daily reminder emails during the GRACE state (Requirement 10.5) to keep the demo flow uncluttered.
7. THE App SHALL prevent Demo_Mode from being enabled on a vault that already holds live (non-demo) entries,
   and SHALL display a warning: "Demo mode cannot be enabled on a vault with existing entries."

---

### Requirement 14: Secret-Sharing Correctness (Property-Based Test Target)

**User Story:** As a developer, I want automated property-based tests for secret sharing, so that correctness
of the split/combine logic is verified across all valid inputs.

#### Acceptance Criteria

1. FOR ALL valid Vault_Key values and all valid (k, n) pairs where 2 ≤ k ≤ n ≤ 5, splitting the
   Vault_Key into n shares and then combining any k of those shares SHALL reconstruct a value equal to the
   original Vault_Key.
2. FOR ALL valid (k, n) pairs and any subset of fewer than k shares, the Shamir_Library SHALL NOT return a
   value equal to the original Vault_Key.
3. THE App SHALL encode Share values as hex strings for display and entry in the release portal, and FOR ALL valid Share values, hex-encoding then hex-decoding SHALL produce a value byte-for-byte equal to the original Share (round-trip property).

---

### Requirement 15: Encryption Round-Trip Correctness (Property-Based Test Target)

**User Story:** As a developer, I want automated property-based tests for encryption, so that no plaintext is
silently corrupted by the AES-GCM implementation.

#### Acceptance Criteria

1. FOR ALL valid plaintext strings and Vault_Key values, encrypting the plaintext with Web_Crypto AES-GCM and
   then decrypting the resulting Ciphertext with the same key SHALL produce a value equal to the original
   plaintext.
2. IF a Ciphertext is modified (any byte altered, added, or removed), THEN Web_Crypto AES-GCM decryption SHALL
   fail with an authentication error and SHALL NOT return any partial plaintext.

---

### Requirement 16: State Machine Safety (Property-Based Test Target)

**User Story:** As a developer, I want automated property-based tests for the check-in state machine, so that
the system can never release the vault before the grace period has fully elapsed.

#### Acceptance Criteria

1. FOR ALL valid sequences of Cron-tick and check-in events, the State_Machine SHALL NOT transition to
   RELEASED before the Grace_Period start timestamp plus the Grace_Period duration has been reached.
2. FOR ALL valid event sequences, the State_Machine SHALL transition through states only in the order ACTIVE →
   OVERDUE → FINAL_REMINDER → GRACE → RELEASED; backward transitions from RELEASED SHALL NOT occur.
3. WHEN a check-in event occurs in state OVERDUE, FINAL_REMINDER, or GRACE, the State_Machine SHALL
   transition to ACTIVE, and a subsequent run of the Cron job SHALL NOT immediately re-enter OVERDUE unless
   the new Check_In_Interval has also elapsed.

---

### Requirement 17: AI Executor Data Access Control

**User Story:** As an Owner, I want the AI executor to access only the entries I have explicitly marked as
shareable, so that sensitive vault contents are never sent to a third-party LLM in bulk.

#### Acceptance Criteria

1. THE App SHALL send to the LLM API only the plaintext of entries where AI_Flag is true; it SHALL NOT send
   the Vault_Key, any Share, or any Ciphertext to the LLM API.
2. THE App SHALL NOT send entries where AI_Flag is false to the LLM API, regardless of whether those entries
   have been decrypted in the current session.
3. THE App SHALL decrypt AI_Flag-true entries in the browser before sending them to the Hono backend, which
   SHALL forward only the plaintext content to the LLM API endpoint.
4. WHEN the Owner marks an entry with AI_Flag true, THE App SHALL display the inline notice: "This entry will
   be shared with an AI model when the executor runs."

---

### Requirement 18: Judge / Demo Access

**User Story:** As a hackathon judge, I want a one-click "Try the demo" button that creates a throwaway demo account with demo mode already enabled and no email address required, so that I can experience the full flow without waiting for an inbox.

#### Acceptance Criteria

1. THE App SHALL display a "Try the demo" button on the sign-in page alongside the magic-link form.
2. WHEN the Judge clicks "Try the demo", THE App SHALL create a throwaway demo account in D1 with a system-generated identifier (no email required), set Demo_Mode to enabled, and start an authenticated session in the browser.
3. THE App SHALL pre-populate the throwaway account with one sample vault entry of each type (Personal Letter, Account Instructions, Personal Wish, Contact Note) so the judge can see a fully configured vault without manual setup.
4. THE App SHALL display a persistent banner on the throwaway account reading "JUDGE DEMO — this account and all its data will be deleted after 2 hours."
5. WHEN 2 hours have elapsed since the throwaway account was created, THE App SHALL delete the account and all associated D1 records.
6. THE App SHALL allow the judge to progress through the full check-in → reminder → grace → release flow using the in-app outbox and the "Advance Clock" button without any real email interaction.
7. WHEN the throwaway demo account is created, THE App SHALL set a fixed vault passphrase of "demo-passphrase" and display it prominently on the judge demo screen with the label "Vault passphrase: demo-passphrase", so that the judge can unlock the vault without choosing or remembering a passphrase.
