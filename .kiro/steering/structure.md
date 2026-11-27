# Project structure

## Current layout

```
kiroweeen-2026/
├── .kiro/
│   └── steering/          # Persistent AI assistant rules (never gitignore)
│       ├── product.md
│       ├── tech.md
│       ├── structure.md
│       └── data-policy.md
├── research/
│   ├── 01-hackathon-brief.md   # Rules, dates, prizes, judging criteria
│   ├── 02-idea-research.md     # 2025 data, 12 scored ideas, decision rationale
│   └── data/                   # Gitignored — local only
│       ├── agent-scoring-brief.md
│       └── kiroween-2025-gallery.json
├── DISCLAIMER.md          # User-facing disclaimer; shown during onboarding
├── ROADMAP.md             # Deferred features requiring security or legal review
├── README.md
└── .gitignore
```

## Planned layout (once build starts)

```
kiroweeen-2026/
├── .kiro/
│   ├── steering/          # This folder — always committed
│   ├── specs/             # Requirements, design docs, task lists
│   └── hooks/             # Automation hooks
├── src/
│   ├── worker/            # Hono API + Cloudflare Worker entry point
│   ├── client/            # React + Vite + Tailwind frontend
│   └── shared/            # Types and utilities shared between worker and client
├── test/                  # Vitest unit and property-based tests
├── public/                # Static assets
├── research/              # Read-only reference material — do not modify during build
├── DISCLAIMER.md
├── ROADMAP.md
├── README.md
├── wrangler.toml          # Cloudflare Workers + D1 + Cron config
├── vite.config.ts
├── tsconfig.json
└── package.json
```

## Conventions

- **`.kiro/` is always committed.** It is part of the judging checklist; never add it to `.gitignore`.
- **`research/data/` is gitignored and local-only.** It contains large scraped data files not needed in the repo.
- **`research/` text files are read-only reference.** Do not edit them during the build phase.
- **Specs before code.** New features get a spec file in `.kiro/specs/` before implementation begins.
- **Kiro Crew app (if built) lives in a separate GitHub repo.** The Kiro Crew prize requires a separate repo link; do not add a `crew/` subfolder here.
- **Secrets never in source.** API keys and tokens go in Wrangler secrets or `.dev.vars`, never in committed files.
- **DISCLAIMER.md and ROADMAP.md are project-level documents.** They are referenced by `data-policy.md`; keep them accurate as the app evolves.
