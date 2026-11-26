# Idea research and category choice

Research done 2026-11-24. Method: scrape and analyse Kiroween 2025 (first edition) and the Code with Kiro
hackathon (2025, same sponsor), generate 3 ideas per category from the winner patterns, then score each idea
with an independent analyst agent per category using one shared rubric.

**Caveat:** win chances are estimates, not measurements. The category split is keyword-based (42% of 2025
entries could not be classified), and the agents' outside web findings below were not all re-verified.

## Decision

**Category: Frankenstein. Project: Last Will.exe**, a digital-legacy dead man's switch with an AI executor.
Runner-up: Crypt PWA (Skeleton Crew).

## Data: Kiroween 2025

744 gallery entries (https://kiroween-2025.devpost.com/project-gallery), 4,956 registered participants.
62 entries carry a "Winner" badge, but 51 of them are $100 blog or social prizes. Cash winners ≥ $1k:

| Prize | Project | Category | What it is | Team |
|---|---|---|---|---|
| Overall 1st $30k | [InteractionKit](https://devpost.com/software/kiro-interactionkit) | Skeleton Crew | Template for gesture-controlled games/apps with MediaPipe | 2 |
| Overall 2nd $20k | [UnitSet](https://devpost.com/software/unitset) | Frankenstein | Canvas + AI + code + live sandboxes for UI design | 1 |
| Overall 3rd $10k | [SpookBnB](https://devpost.com/software/spookbnb-when-dark-mode-gets-truly-dark) | Costume Contest | Rental site whose dark mode becomes a horror escape room | 1 |
| Best Resurrection $5k + Influencer $1k | [Faxi](https://devpost.com/software/faxi-16iecy) | Resurrection | Fax-to-internet bridge for offline seniors in Japan | 1 |
| Best Frankenstein $5k | [RumorNet](https://devpost.com/software/rumornet) | Frankenstein | Multilingual multi-agent misinformation detection | 1 |
| Best Skeleton Crew $5k + Most Creative $2.5k | [Impulse Kit](https://devpost.com/software/impulse-kit) | Skeleton Crew | Go/No-Go habit-breaking game kit, vanilla JS | 1 |
| Best Costume Contest $5k | [Seance](https://devpost.com/software/seance) | Costume Contest | Spooky book-style legends explorer | 1 |
| Best Startup $10k | [Splat-n-Treat](https://devpost.com/software/splat-n-treat) | — | "Notion for 3D Gaussian-splat worlds" | 1 |
| Influencer Choice $1k | [BlueKai](https://devpost.com/software/bluekai-bluesky-on-a-flip-phone) | Resurrection | BlueSky client for KaiOS flip phones | 1 |

Patterns:

1. 8 of 9 cash winners were solo; the 1st-place team had 2 people.
2. Winners pair a spooky hook with a specific audience and problem (offline seniors, flip-phone users,
   misinformation, habit-breaking, computer-vision developers).
3. Simple stacks can win (Impulse Kit is vanilla JS). Polish and story beat technical complexity.
4. Winning write-ups document Kiro features in depth (UnitSet mentions specs, steering, MCP and hooks repeatedly).
5. Crowded 2025 themes: COBOL/legacy migration, haunted portfolios and terminals, ELIZA, Yahoo Pipes, mixtapes,
   file-cleanup "exorcists", digital-footprint and cloud-identity audits, GitHub "graveyards".

Estimated category share of classifiable 2025 entries:

| Category | Share | 2025 results |
|---|---|---|
| Costume Contest | ~39% | 3rd overall + category prize |
| Resurrection | ~37% | Category + 2 influencer prizes; no overall placing |
| Frankenstein | ~15% | 2nd overall + category prize |
| Skeleton Crew | ~10% | 1st overall + category prize + Most Creative |

## Data: field size and the Code with Kiro hackathon

- Registered participants: Kiroween 2026 = **18,820** (checked 2026-11-24), Kiroween 2025 = 4,956 (744 entries,
  15% submitted), Code with Kiro 2025 = 9,948 (552 entries, 5.5% submitted).
- Projected 2026 entries ≈ 1,000–2,800, so 2025-based win chances were divided by ~2.5.
- Code with Kiro winners (https://kiro.devpost.com/project-gallery) skew towards practical tools for a named
  audience: self-hosted file upload, healthcare-worker burnout prediction, liver-disease lab tracker,
  garment-factory payroll, personalised kids' storybooks.

## All 12 ideas, scored

Scores assume a solo builder with Kiro and ~10 days, including deploy, video and write-up.
Chance = estimated probability of any cash prize ≥ $1k, adjusted for the 2026 field size.

| Category | Idea | What it is | Build days | Complexity /10 | Winnability /10 | Chance |
|---|---|---|---|---|---|---|
| Frankenstein | **Last Will.exe** | Dead man's switch: check-ins, secret-shared release, AI executor | 6 | 6 | **7** | ~4% |
| Frankenstein | Night Shift | Kiro Crew bots act as on-call for solo devs | 9 | 8 | 6 | ~3.5% |
| Frankenstein | SheetShop | Run a micro-business from a spreadsheet via SMS | 6 | 5 | 5 | ~2.5% |
| Skeleton Crew | **Crypt PWA** | Offline-first PWA template + NGO survey app + clinic queue app | 8.5 | 7 | 6 | ~4% |
| Skeleton Crew | Bone Yard | Template for building Kiro Crew bots | 9 | 7 | 4 | ~2.5% |
| Skeleton Crew | Coffin Kit | Turn any API into an MCP server + Kiro power | 8 | 6 | 3 | ~1.5% |
| Resurrection | **Knock Twice** | Morse code as one-switch typing for people with ALS | 6.5 | 5 | 6 | ~3% |
| Resurrection | Dead Line | AI answering machine that screens scam calls | 8.5 | 8 | 5 | ~2% |
| Resurrection | HyperHaunt | HyperCard rebuilt with AI | 9 | 7 | 3 | ~1% |
| Costume | **The Witching Hour** | Extension that makes sites spookier after bedtime | 6.5 | 6 | 6.5 | ~3% |
| Costume | Subscription Graveyard | Finds forgotten subscriptions, shown as zombies | 6 | 5 | 5 | ~2% |
| Costume | Exorcist Manor | Security audit shown as a haunted house | 8 | 7 | 4 | ~1.5% |

Expected judging scores (1–10) for the four category leaders:

| Idea | Impact | Kiro | Design | Innovation | Presentation |
|---|---|---|---|---|---|
| Last Will.exe | 7 | 8 | 8 | 6 | 8 |
| Crypt PWA | 7 | 7 | 7 | 6 | 7 |
| Knock Twice | 8 | 7 | 7 | 6 | 8 |
| The Witching Hour | 7 | 6 | 8 | 7 | 8 |

## Why Last Will.exe

- Highest winnability score of the 12 ideas and tied for the best chance, at only ~6 build days, which leaves
  ~4 days for polish, video and write-up. Crypt PWA has the same chance but needs ~8.5 days.
- No 2025 Kiroween entry was a dead man's switch. Nearest: TimeGrave (voice time capsule) and Mortis
  (afterlife chat); neither won.
- Frankenstein was ~15% of 2025 entries versus 37–39% for Resurrection and Costume, and produced 2nd overall.
- Death is the most Halloween-native theme; the emotional story helps Presentation and the Startup prize.
- Strong Kiro showcase: specs, hooks, steering, and property-based tests on secret sharing
  ("any k shares rebuild the secret, k−1 do not"). An optional Kiro Crew check-in app (~1.5 extra days)
  adds a route to the $1k Best Kiro Crew App prize.

Risks and mitigations:

- **Not new outside Kiroween.** Devpost already has Last Safe (https://devpost.com/software/last-safe),
  DeadVault, Lifeline and Passage; products include deadmansswitch.net, GoodTrust and Apple Digital Legacy;
  "Dead Man's Block" won a crypto hackathon
  (https://bitcoinmagazine.com/culture/hackathon-winner-can-help-pass-your-bitcoins-after-you-die).
  → Make the **AI executor** that guides the family the centrepiece of the product and the demo.
- **Trust in the cryptography.** → Encrypt client-side, explain where shares are stored, guard against false
  triggers (grace periods, multiple reminders, trusted-contact confirmation).
- **Demo timing.** → A time-compressed demo mode so a release happens within the 3-minute video.
- **Frankenstein fit.** → Stitch an old-fashioned channel (printed letter / recorded voice message) together with
  modern cryptography, scheduling and AI agents, and say so explicitly in the pitch.

## Rules check: building an idea that already exists elsewhere

Checked against the Official Rules (https://kiroween.devpost.com/rules) on 2026-11-26. No rule requires the
concept to be new. Similar existing products or hackathon projects do not make an entry ineligible.
What the rules do require:

- **New Projects Only**: all code built during the submission period (Oct 30 – Dec 4, 2026). Standard frameworks,
  libraries and starter templates are allowed; any other pre-existing code must be disclosed.
- **Original work and IP**: the submission must be our own work and must not infringe copyright, trademark,
  patent, privacy or other rights. → Do not copy code, text, UI designs, names or logos from Last Safe, GoodTrust,
  deadmansswitch.net or similar. Shamir's secret sharing (1979) and dead man's switches are long-established
  public techniques. No patent or trademark search has been done for the name "Last Will.exe".
- **Open source**: allowed if licences are respected and our software builds on it.
- **Video**: no third-party trademarks or copyrighted music without permission (don't show competitors' logos).
- **Third-party APIs**: use only services whose terms we comply with (email/SMS providers, AI APIs).

Similarity affects only the **Innovation** score (20%), which is why the AI executor must lead the pitch.

## Scoring rubric used by the agents

- Complexity: build days for a solo developer with Kiro (incl. deploy, video, write-up), 1–10 score, biggest risk.
- Winnability: 1–10 score, probability of any cash prize ≥ $1k, expected 1–10 score on each judging criterion,
  grounded in the 2025 base rate (~11 prizes ≥ $1k across 744 entries ≈ 1.5%), category density,
  originality against the 2025 gallery, and outside hackathon winners and products.
