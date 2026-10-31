# Corroba

![Next.js](https://img.shields.io/badge/Next.js-16-black)
![React](https://img.shields.io/badge/React-19-61DAFB)
![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6)
![Node](https://img.shields.io/badge/Node-20-339933)
![Status](https://img.shields.io/badge/Status-Hackathon%20prototype-informational)

**Turn hidden customer proof into your growth engine.**

Companies already hold their best marketing evidence, but it sits buried in reviews, support tickets, NPS surveys, sales calls and customer emails. Corroba finds those trust signals, scores and ranks them, then turns the strongest ones into audience ideas, content and a go-to-market playbook. It was built by a five-person team in one day at the GTM Hackathon in London, and it runs end to end with no API keys.

**Links:** [Demo Video](https://youtu.be/ALsHWDr2dMA) · [Team Repository](https://github.com/Joachimsyh/Corroba) · [Auto-GTM Generator Docs](docs/auto-gtm-generator.md)

## Contributors

| Contributor | Role / Feature Ownership |
| --- | --- |
| Anjila Gurung ([@Anj-13](https://github.com/Anj-13)) | Auto-GTM Generator: GTM page, API, feedback metrics and documentation |
| Faizan Alvi ([@alvi83252-dot](https://github.com/alvi83252-dot)) | Initial full-stack build, RAG demo-data pipeline, proof validation, Unify notifications, sponsor integrations, Vercel deployment, UX improvements and serverless migration |
| Andrii Kontovskyi ([@akontovskyi](https://github.com/akontovskyi)) | Local dev setup, Next.js 16 upgrade, Corroba rebrand, UnifyGTM Analytics integration and API hardening |
| Oleksandra Denysenko ([@Aalex2203](https://github.com/Aalex2203)) | Zero CRM sync and database schema |
| Joachim Sin ([@Joachimsyh](https://github.com/Joachimsyh)) | Team member; hosts the [team repository](https://github.com/Joachimsyh/Corroba) |

## Table of Contents

- [Project Overview](#project-overview)
- [My Role & Technical Contributions](#my-role--technical-contributions)
- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [System Architecture & Project Structure](#system-architecture--project-structure)
- [Getting Started](#getting-started)
- [Usage](#usage)
- [Testing & Quality](#testing--quality)
- [Challenges & Solutions](#challenges--solutions)
- [Results & What I Learned](#results--what-i-learned)
- [Future Improvements](#future-improvements)
- [License](#license)

## Project Overview

**Problem:** Go-to-market teams rarely know which customer outcomes are their strongest evidence, so proof like "we saved £40,000" stays unused while they write generic content.

**Solution:** Corroba extracts quotable claims from customer material, gives each one a Proof Score, and uses the top-ranked signals to drive targeting, content and outreach. Discovering proof comes first and generating content comes second.

**Audience / Context:** B2B founders and GTM teams. Built on 20 June 2026 at the [GTM Hackathon](https://gtmengineer.dev) in London, hosted by Cursor, GTMengineer.dev and Lightfern.

**Project Status:** Prototype. It is a working demo, not a production service: most state is held in memory and several partner integrations are simulated (see [which integrations are live](#integrations-live-vs-simulated)).

> The project was originally called **ProofLoop**. That name remains in internal identifiers such as the npm package names and the `proofloop` database.

## My Role & Technical Contributions

**Anjila Gurung ([@Anj-13](https://github.com/Anj-13)) — Developer, Auto-GTM Generator**

I built the Auto-GTM Generator, the feature that turns ranked proof into a usable go-to-market plan. My teammates built the rest of the product, as listed under [Contributors](#contributors).

### Core Contributions

**Playbook generation**
- Rewrote `generateGtmSystem()` so that playbooks are built from the top three trust signals instead of static demo content.
- Mapped signal types to an ideal customer profile, then generated positioning, a four-touch outreach sequence, content ideas, landing page angles and next actions around the selected proof quotes.

**GTM API and data model**
- Added the `POST /api/gtm/generate`, `POST /api/gtm/feedback` and `GET /api/gtm/metrics` endpoints, and extended `GET /api/gtm-playbooks` to return metrics.
- Added `FeedbackEntry` and `GtmMetrics` to the store, with `addPlaybook()`, `addFeedback()` and `getGtmMetrics()`. These were kept when the team later moved the backend into Next.js.

**GTM page (`/gtm`)**
- Rewrote the page with a metrics dashboard, a panel showing which signals were used, full playbook rendering, helpful / not helpful feedback on each playbook, and an empty state.

**Documentation**
- Wrote the [Auto-GTM Generator docs](docs/auto-gtm-generator.md), covering the data flow, API contracts and UI.

> **Contribution summary:** Built the Auto-GTM Generator for a five-person hackathon team: a full-stack TypeScript feature that turns a company's top three customer proof signals into a go-to-market playbook, with a feedback loop and usage metrics.

## Key Features

- **Proof discovery:** Upload PDF, CSV or TXT files, or paste text, and get extracted trust signals such as revenue savings, time savings, growth gains and risk reduction.
- **Proof Score and rankings:** Score each signal out of 100 on specificity, credibility, revenue impact, emotional impact and conversion potential, then rank them.
- **Proof validation:** Check a quote against social posts and mark it verified, partial or unverified. This uses a simulated dataset by default.
- **Audience expansion:** Given a strong quote, suggest the companies and ICPs it would resonate with most, using retrieval over demo conversations.
- **Auto-GTM generator:** Create a playbook from the top three signals in one click, then rate it.
- **Content and growth recommendations:** Generate proof-led content assets and a list of next actions ranked by impact and effort.
- **Proof CRM:** Track signals and assets, and sync them to Zero when configured.
- **Demo mode:** Explore every page with realistic sample data, without API keys or a database.

## Tech Stack

| Layer | Technologies | Purpose |
| --- | --- | --- |
| Frontend | Next.js 16 (App Router), React 19, Tailwind CSS v4, D3 | Dashboard pages, theming and charts |
| Backend | Next.js Route Handlers, TypeScript | 50 API route files running as serverless functions |
| AI / Retrieval | OpenAI `gpt-4o-mini`, Voyage AI `voyage-3`, in-memory vector store | Optional LLM extraction, embeddings and hybrid search |
| Database | PostgreSQL 16, Drizzle ORM, `postgres` | Optional audit log for Zero sync |
| Integrations | UnifyGTM, Zero, Faxxing, SCAILE, Lightfern, GTMengineer.dev | Hackathon partner APIs |
| Tools | Docker Compose, Vercel, Git, Cursor, Next.js DevTools MCP | Local services, deployment and development |

## System Architecture & Project Structure

Everything runs in one Next.js app. Pages call `/api/*` route handlers, which share logic in `src/lib/server/handlers.ts`. Each step uses a real provider when its key is set and falls back to local logic when it is not, so the demo never depends on an outside service.

```mermaid
flowchart LR
    A[Upload, paste or<br/>sample data] --> B[Signal extraction<br/>OpenAI or rule-based]
    R[(In-memory<br/>vector store)] -. context .-> B
    B --> C[Proof Score<br/>and rankings]
    C --> D[Validation]
    C --> E[Audience expansion]
    C --> F[Auto-GTM playbook]
    C --> G[Content and growth]
    C --> H[Proof CRM]
    H -. optional .-> I[Zero and Postgres audit log]
```

<details>
<summary>View project folder structure</summary>

```text
Corroba/
├── apps/web/
│   ├── drizzle/                # SQL migrations
│   ├── scripts/                # API route generator
│   └── src/
│       ├── app/                # Pages, plus api/ route handlers
│       ├── components/         # Shell, charts, command palette, UI primitives
│       └── lib/
│           ├── api.ts          # Frontend API client
│           └── server/
│               ├── ai/         # Extraction, scoring, embeddings, file parsing
│               ├── data/       # Demo data
│               ├── db/         # Drizzle schema, setup and seed
│               ├── integrations/ # Partner APIs and simulated fallbacks
│               ├── rag/        # Chunking, vector store, retrieval
│               ├── store/      # In-memory application state
│               └── handlers.ts # Shared route logic
├── demo-data/                  # Sample customer proof and guidance
├── docs/                       # Feature documentation
├── infra/                      # OpenTelemetry collector config
├── scripts/                    # Pre-dev dependency check
├── docker-compose.yml          # Local Postgres and optional services
└── README.md
```

</details>

### Integrations: Live vs Simulated

| Service | Without keys | With keys |
| --- | --- | --- |
| OpenAI | Rule-based extraction | LLM extraction when retrieval context is available |
| Voyage AI | Local hash embeddings | `voyage-3` embeddings |
| UnifyGTM Data API | Demo mode | Syncs proof to Unify records. Retrieval still uses demo conversations because Unify has no conversations API. |
| Unify Analytics | Simulated. Events are kept in memory and shown in the UI. | Sends intent events using a separate write key |
| Unify notifications | Simulated locally | Remains simulated; live sync uses the Data API |
| Zero | Sync reports as not configured | Syncs CRM records and logs results to Postgres |
| Faxxing | **Simulated.** The social posts and accounts are fictional sample data. | Calls the API at `FAXXING_API_URL` |
| SCAILE | Demo recommendations | Calls the API at `SCAILE_API_URL` |
| Lightfern | Signals keep their local scores | Sends quotes to the API at `LIGHTFERN_API_URL` for scoring |
| GTMengineer.dev | **Simulated.** Uses the local generator, as there was no public API at the event. | Calls the API at `GTMENGINEER_API_URL` |

### Notable Design Decisions

- **Key-gated fallbacks:** Every integration works without credentials. This let judges use the product within a minute, at the cost of a clear gap between demo and live behaviour.
- **Deterministic playbooks:** GTM output comes from mapped signal types, not an LLM, so it is consistent and needs no external calls. It is less tailored than generated text would be.
- **In-memory state:** Sources, signals, playbooks and feedback live in memory, which was fast to build. They reset on restart, and serverless instances do not share them. The Drizzle schema defines tables for this data, but today only the Zero sync log is written to Postgres.
- **One deployable:** The team replaced the separate Hono API with Next.js route handlers, so the app deploys as a single project with no external API URL.

## Getting Started

### Prerequisites

- Node.js 20 (see `.nvmrc`)
- Docker, only if you want the optional database

### 1. Clone and Install

```bash
git clone https://github.com/Anj-13/Corroba.git
cd Corroba
npm run install:all
```

### 2. Run

```bash
npm run dev
```

**Expected result:** The app runs at <http://localhost:5173> in demo mode. No `.env` file is needed for this.

### 3. Configure (Optional)

Add keys only for the integrations you want to use live.

```bash
cp .env.example apps/web/.env
```

> **Important:** Put the file in `apps/web/`, not the repository root. Next.js runs from `apps/web` and does not read a root `.env`.

### 4. Set Up the Database (Optional)

The database is only needed for the Zero sync log. Start Postgres:

```bash
docker compose up -d postgres
```

Set this value in `apps/web/.env`:

```env
DATABASE_URL=postgresql://proofloop:proofloop@localhost:5432/proofloop
```

Then create the tables and seed them:

```bash
npm run db:setup
```

`docker-compose.yml` also defines Typesense, Grafana and an OpenTelemetry collector. The app does not use them yet, so you can leave them off.

### Deployment

The repository is configured for Vercel. Set the Root Directory to `apps/web` (recommended) or the repository root, and add any keys as environment variables.

## Usage

1. Open **Proof Discovery**, then upload a file, paste customer text or pick a sample dataset.
2. Review the results in **Trust Signals** and **Proof Rankings**.
3. Open **GTM Playbooks** and select **Generate New GTM System**, then rate the result as helpful or not.
4. Explore **Proof Validation**, **Audience Expansion**, **Content Studio** and **Growth Recommendations** for the same proof.
5. Use **Settings** to check integration status and reset the demo data.

## Testing & Quality

- **Tests:** Automated tests are not yet implemented, and `npm run lint` is currently a placeholder.
- **Build check:** `npm run build` compiles the app and runs the TypeScript type-check.
- **Runtime safety:** API routes share a per-client rate limiter (10 requests per second by default), and a failed integration or database connection falls back to demo behaviour.

## Challenges & Solutions

| Challenge | Approach | Outcome |
| --- | --- | --- |
| GTMengineer.dev had no public API | Simulated the endpoint locally behind the same key-gated call | The playbook flow worked in the demo and can switch to a real API by config |
| UnifyGTM has no conversations API for retrieval | Kept a demo conversation corpus for retrieval and used the Data API for outbound sync | Audience expansion worked without changing the sync design |
| Unify uses separate credentials for data and analytics | Separated the Data API secret key from the Analytics write key in config | Each API gets the correct credential |
| The frontend and Hono API were set up to deploy separately | Moved the backend into Next.js route handlers | Frontend and API deploy as one Vercel project |
| Vercel did not detect Next.js in the monorepo | Added a root `next.config.js` that re-exports the app config, and pinned Node 20 | Vercel can build from the repository root or `apps/web` |

## Results & What I Learned

- **Outcome:** A working 13-page product and a [demo video](https://youtu.be/ALsHWDr2dMA), delivered by five people in one day.
- **Skills gained:** Designing a full-stack feature from data model to UI, API contract design, TypeScript and Next.js, shipping quickly on a shared codebase through pull requests, and writing technical documentation.

## Future Improvements

- [ ] Persist all application data in Postgres, so it survives restarts and works across serverless instances.
- [ ] Add authentication and real workspaces, as the app currently uses a single default workspace.
- [ ] Replace rule-based GTM generation with LLM generation grounded in the top signals, to make playbooks more specific.
- [ ] Use playbook feedback to influence future recommendations.
- [ ] Replace simulated validation with a real social data source, so that verification results can be trusted.
- [ ] Add automated tests and ESLint, then run them in CI.

## License

No license has been specified.
