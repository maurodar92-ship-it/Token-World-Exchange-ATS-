# Token World Exchange ATS (TWE)

**Signal Terminal** — a full-stack, non-production exchange-design and paper-trading sandbox.

TWE brings a sample token market, paper account, deterministic token-concept generator, private NFT collection planner, and architecture-planning workspace into one responsive application. It is a working software prototype, not a licensed exchange or an operational trading venue.

> **Sandbox boundary:** No real money is accepted or moved. Orders and balances are simulated. The application does not custody assets, connect wallets, mint NFTs, deploy or call smart contracts, publish token concepts, connect a live market, use a test chain, or contact a financial/token provider. Keep these capabilities disconnected.

## How TWE looks

TWE uses a dark, technical “Signal Terminal” visual language. The canvas is deep navy (`#07131f`), with raised blue-black panels, thin blue-gray separators and restrained cyan highlights (`#20d9ff`). Green (`#36d7a1`) and coral (`#ff7184`) indicate positive and negative sample movements. Space Grotesk is used for headings and general interface text; IBM Plex Mono is used for symbols, labels and numeric market data.

The public landing page pairs a large, quiet headline with an orbital market illustration, sample quote strip, capability cards and a prominent simulation disclosure. The authenticated shell uses a fixed desktop navigation rail, compact top bar, breadcrumb, “PAPER MODE” indicator and information-dense cards, tables, charts and order panels. On narrow screens the navigation becomes a collapsible menu and data panels reflow. A low-opacity, transparent TWE logo watermark sits behind the landing, public, authenticated, loading and not-found page backgrounds; it is decorative and does not intercept clicks.

The visual hierarchy favors clear labels and subdued surfaces over exchange-style flashing effects. Quote prices, depth, settlement language, balances, order fills and generated supply are marked or treated as illustrative simulation data.

## Pages and workspaces

- `/` — public landing page; signed-in users see the paper-account overview.
- `/markets` and `/markets/:symbol` — sample token catalogue, market detail, indicative depth and simulated trade tape.
- `/trade/:symbol` — paper order entry and trading workspace.
- `/portfolio` and `/orders` — simulated holdings, balances, order history and fills.
- `/token-generator` — private, deterministic token-design batches of up to 1,000 concepts per request, with a per-account saved limit. Each concept proposes a simulated supply up to one trillion units; it does not create or issue a token.
- `/nft-foundry` — private NFT collection-design drafts, up to 25 per batch and 1,000 saved drafts per account. Drafts are not minted NFTs and have no real contract or token IDs.
- `/readiness` — educational planning content, not legal advice or authorization to operate.
- `/architecture` — owner-scoped planning tools for asset classifications, proposed listing reviews, provider boundaries, ledger/reconciliation concepts, restrictive jurisdiction notes, evidence references, draft-only contract specifications, and release gates.

## Current sandbox behavior

The sample market and order book are illustrative. New paper accounts are initialized with a simulated USD balance and sample holdings. Market and limit orders update only paper balances, simulated orders and simulated fills. No order is routed to a live venue.

Token and NFT generators are deterministic from their stored seed. Concept batches are private to the signed-in owner. The Architecture workspace is also owner-scoped; user-entered evidence references are not fetched or verified. Its listing review has no live approval/listing state, and its contract specifications are draft-only.

These database tables and services are demonstration components. In particular, `account_balances`, `exchange_orders` and `exchange_trades` must never be reused as books for customer funds or real assets. A separate production-readiness checklist can guide future planning; it is not authorization to launch.

## Full-stack architecture

- **Frontend:** React 19, TypeScript, Vite, Tailwind CSS 4, Wouter routing, Lucide icons and TanStack Query/tRPC client.
- **Backend:** Node.js and Express, typed tRPC routers, Zod input validation, application services and server-side ownership checks.
- **Persistence:** MySQL with Drizzle ORM and checked-in migrations. The schema separates users, sample assets, paper balances/orders/trades, token-concept batches, NFT-design batches and architecture-planning records.
- **Tests:** Vitest and TypeScript checks.

### Repository map

```text
client/src/              React routes, UI components, pages and styles
client/public/           Public assets, including twe-watermark.png
drizzle/                 MySQL schema and migrations
server/                  tRPC routers and business services
server/_core/            Express runtime and Manus platform adapters
shared/                  Shared types, enums and deterministic generators
```

## Run the source locally

Use Node.js supported by the checked-in Vite version and pnpm 10.18.0. The original application expects a MySQL database and Manus authentication/platform services; these values are **not** included in the source package.

```bash
corepack enable
pnpm install --frozen-lockfile
pnpm check
pnpm test
pnpm build
```

To run the original Manus-integrated version, configure the required server environment through the appropriate secure environment manager. Core settings include `DATABASE_URL`, `MANUS_PROJECT_ID`, `MANUS_JWT_SECRET`, and `MANUS_OAUTH_API_URL`; platform API settings may also be required for the specific storage or API features used. Never commit `.env` files or credentials. Apply checked-in migrations with `pnpm db:migrate`, then start the development server with `pnpm dev` (the default port is 3000; `PORT` can override it).

## Use this source with Google AI Studio

Google AI Studio’s current Build-mode documentation describes importing a project from GitHub. It documents a React web frontend and Node.js server runtime, and describes Firebase Authentication/Firestore integration and GitHub synchronization as available options. This does **not** mean the Manus-bound application runs unchanged in AI Studio. [1]

1. Download and extract the accompanying `TWE_Google_AI_Studio_source.zip`.
2. Create a GitHub repository in your own account that your AI Studio account can access, and upload the extracted source. Keep the repository private where the import workflow permits; do not make it public just to bypass an access issue. Do not add `.env`, database passwords, OAuth secrets or private keys.
3. In Google AI Studio **Build** mode, choose **Add files (+) → Import from GitHub** and select that repository. The exact interface may change; follow the current [Google Build-mode documentation](https://ai.google.dev/gemini-api/docs/aistudio-build-mode).
4. Open [`GOOGLE_AI_STUDIO_PROMPT.md`](./GOOGLE_AI_STUDIO_PROMPT.md). Ask the Build agent to audit compatibility and propose a migration plan before it edits the project. Then have it adapt and test the application in small steps.
5. Replace the Manus OAuth/session adapter with an authentication system supported by the target app. A Google sign-in implementation commonly uses Firebase Authentication, but server-side identity verification and user-owned data access must still be implemented.
6. Choose persistence deliberately. The current app uses MySQL and Drizzle. Keep a separately hosted, network-reachable MySQL database only if you intend to operate it and can configure it securely; otherwise explicitly redesign and test the schema and transactional behavior for Firestore. Do not assume importing code migrates or provisions the current database.
7. Keep secrets server-side in the target platform’s secret manager. The concept generators are deterministic and do not require a Gemini API key.

The source ZIP omits Git history, dependencies, build output, credentials, and Manus-only project metadata. It includes the application source, lockfile, watermark asset, this README and the AI Studio prompt. It is an import starting point, not a one-click deployment or production conversion.

## Google AI Studio references

[1]: https://ai.google.dev/gemini-api/docs/aistudio-build-mode "Build apps in Google AI Studio"
