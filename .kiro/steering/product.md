# Product: Last Will.exe

A digital-legacy dead man's switch entered in the **Frankenstein** category of the [Kiroween 2026 hackathon](https://kiroween.devpost.com/) (submission deadline: Dec 4, 2026, 2:00 PM PT).

## What it does

Users check in on a schedule. If check-ins stop, encrypted instructions — password-manager emergency-access instructions, personal letters, notes — are released to named recipients using Shamir's secret sharing. An AI "executor" then guides the family through closing accounts and carrying out the user's wishes.

**Privacy rule:** the AI executor never receives decrypted vault contents. It works only from estate-plan entries the owner has explicitly marked as shareable with AI. See `data-policy.md` for the full rule.

## Core value proposition

- **Spooky angle**: "Dead man's switch" is native Halloween material; stitching old-fashioned printed letters and recorded voice messages together with modern cryptography, heartbeat scheduling and LLM agents satisfies the Frankenstein brief.
- **Centrepiece of the pitch**: the AI executor that guides the family, not the cryptography itself.
- **Target audience**: anyone who wants to protect their family from account lockout and unclear instructions after death; especially relevant for solo-dwellers and digital-first people.

## Timeline (today: Nov 27, ~7 days remaining)

| By | Goal |
|---|---|
| Dec 1 | Core features complete (vault, check-in, release flow, AI executor) |
| Dec 2 | Polish: UI, edge cases, disclaimer flow, input warnings |
| Dec 3 | Demo video recorded, write-up drafted |
| Dec 4 AM PT | Submit |

## Hackathon constraints

- Solo build.
- Must showcase Kiro features heavily in the write-up: specs, hooks, steering, MCP, property-based tests (PBT) on secret sharing, optionally Kiro Crew for the check-in flow.
- Judging criteria (20% each): Potential Impact, Technological Implementation (depth of Kiro use), Design, Innovation/Idea, Presentation.
- Deliverables: live app URL, public GitHub repo with OSI licence and `.kiro/` committed, <3-min YouTube demo, write-up on Kiro usage.

## Key risks

- **Not novel as a concept** — mitigate by leading with the AI executor in every demo and write-up.
- **Trust in cryptography** — encrypt client-side, explain where shares are stored, implement grace periods and trusted-contact confirmation before release.
- **Demo timing** — build a time-compressed demo mode so a release can be shown within 3 minutes.
