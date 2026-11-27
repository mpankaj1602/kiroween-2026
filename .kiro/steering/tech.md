# Tech stack

**Status:** stack decided 2026-11-27. ~7 days to submission (Dec 4, 2026, 2:00 PM PT).

## Timeline

| By | Goal |
|---|---|
| Dec 1 | Core features complete |
| Dec 2 | Polish, edge cases, disclaimer flow, input warnings |
| Dec 3 | Demo video recorded, write-up drafted |
| Dec 4 AM PT | Submit |

## Constraints

- Solo builder.
- Must deploy to a live URL accessible to judges.
- Public GitHub repo (not GitLab/Bitbucket) with an OSI-detected open-source licence.
- `.kiro/` directory must be committed (never gitignored).
- Kiro IDE/CLI as the primary development tool, with up to 4,000 Kiro credits.

## Stack

**TypeScript everywhere.**

| Concern | Choice | Notes |
|---|---|---|
| Frontend | React + Vite + Tailwind | Served by the Cloudflare Worker |
| API | Hono | Runs on the same Cloudflare Worker |
| Hosting | Cloudflare Workers (free plan) | Deploy with `wrangler` |
| Database | Cloudflare D1 | Accounts, check-in state, encrypted vault blobs |
| Scheduler | Cloudflare Cron Triggers | Free plan: 5 triggers, 10 ms CPU per invocation — keep jobs light |
| Email | Resend (free tier) | 100 emails/day; no SMS; Telegram optional |
| Encryption | Web Crypto (AES-GCM), browser-side | Key split with `shamir-secret-sharing` (audited); do **not** use `secrets.js-grempe` |
| AI / LLM | Groq free tier (OpenAI-compatible, zero data retention) | Cloudflare Workers AI as fallback; active provider selected by environment variable |
| Tests | Vitest | `fast-check` for property-based tests |
| Secrets | Wrangler secrets or `.dev.vars` | Never committed |

**AI executor note:** the AI executor is a runtime feature of the app — it calls the LLM API from the Hono backend. It is **not** a Kiro custom agent. Kiro custom agents are for our development workflow only (for example, a crypto-review agent that checks the secret-sharing logic before a PR).

**AI data rule:** the executor receives only estate-plan entries the owner has explicitly marked as shareable with AI. It never receives decrypted vault contents in bulk. See `data-policy.md`.

**Bedrock is not used.** This AWS account has no free tier, and Kiro credits do not pay for runtime AI.

## Kiro features to use (judging requirement)

- **Specs**: write requirements → design → task specs before coding; include a Bugfix Spec.
- **Property-based tests (PBT)**: test the secret-sharing logic with `fast-check` (any k shares reconstruct the secret; k−1 do not).
- **Steering**: this file and companions in `.kiro/steering/`.
- **Hooks**: at minimum a file-save or task-completion hook.
- **MCP / Powers**: connect at least one external tool via MCP.
- **Custom agents**: development-workflow agents (e.g. crypto-review agent), not runtime app features.

## Common commands

```bash
# Install dependencies
npm install

# Run dev server (Workers + D1 locally)
npx wrangler dev

# Run tests
npm test

# Run tests with coverage
npm run test:coverage

# Build frontend
npm run build

# Deploy to Cloudflare
npx wrangler deploy

# Push a secret
npx wrangler secret put SECRET_NAME
```
