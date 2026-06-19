# Repository notes for AI coding assistants

This file contains concise, repo-specific guidance so an AI assistant can be immediately productive in this Next.js App Router dashboard starter.

- **Big picture**: This is a Next.js App Router (app/) example dashboard. UI components live under `app/ui`, pages/layouts under `app/`, and server-side data access lives in `app/lib` (see `app/lib/data.ts`, `definitions.ts`, `utils.ts`). The app uses Tailwind (`tailwind.config.ts`) and the `postgres` package for direct SQL access via `process.env.POSTGRES_URL`.

- **Data flow & boundaries**:
  - Server components (async) call `app/lib/data.ts` functions (e.g. `fetchFilteredInvoices`, `fetchCardData`, `fetchRevenue`) to access Postgres directly. See `app/ui/invoices/table.tsx` for an example server component calling `fetchFilteredInvoices`.
  - `app/lib/placeholder-data.ts` contains in-memory demo data used for learning chapters — useful when `POSTGRES_URL` is not set.
  - Types are declared in `app/lib/definitions.ts` and are relied on across `lib` and UI components.

- **Key files to inspect**:
  - `app/lib/data.ts` — primary SQL queries and DB connection: the `postgres` client is instantiated with `postgres(process.env.POSTGRES_URL!, { ssl: 'require' })`.
  - `app/lib/utils.ts` — helpers like `formatCurrency` (amounts are stored as cents) and pagination generation.
  - `app/ui/*` — reusable components and `app/ui/dashboard/*` for layout pieces.
  - `app/page.tsx`, `app/layout.tsx` — entry points for the App Router root.

- **Developer workflows / commands**:
  - Dev server: `npm run dev` (or `pnpm dev` if using pnpm). `package.json` uses `next dev --turbopack`.
  - Build: `npm run build` (`next build`). Start production server: `npm start` (`next start`).
  - Environment: copy `.env.example` → `.env.local` and set `POSTGRES_URL` before running DB-backed pages.

- **Project-specific conventions & patterns**:
  - Components under `app/ui` are split into small presentational pieces. Prefer server components for pages that call `app/lib` fetch functions (they are declared async). Example: `InvoicesTable` is `export default async function InvoicesTable(...)` and directly awaits `fetchFilteredInvoices`.
  - Amounts from DB are in cents. `formatCurrency` in `app/lib/utils.ts` divides by 100 and formats USD. When creating/updating invoices, convert dollars → cents before persisting.
  - Pagination: `ITEMS_PER_PAGE` is defined in `app/lib/data.ts` (6). Use `fetchInvoicesPages` and `generatePagination` together.
  - SQL safety: `postgres` tagged templates are used for parameterization — mirror that style when adding queries.

- **Integration points & external deps**:
  - Postgres via `postgres` npm package — connection string from `POSTGRES_URL` (SSL enforced in current code). Beware of local non-SSL databases: adjust `{ ssl: 'require' }` if needed for dev.
  - `next-auth` (present in deps) — authentication expects `AUTH_SECRET` and `AUTH_URL` env vars in `.env.example`.
  - `@heroicons/react`, `next/image`, and Tailwind utility classes are used pervasively.

- **When changing data access or moving code**:
  - Keep DB access in `app/lib` server-only code. Converting a server component into a client component requires moving DB calls to a server action/route or using the fetch API.
  - If you add new SQL endpoints, follow the existing pattern: create helper functions in `app/lib/data.ts` + types in `definitions.ts`, and use tagged template parameterization.

- **Examples (quick references)**:
  - Fetch invoices for a server component: `const invoices = await fetchFilteredInvoices(query, currentPage)` — see `app/ui/invoices/table.tsx`.
  - Format currency for display: `formatCurrency(invoice.amount)` — see `app/lib/utils.ts` and `app/ui/invoices/table.tsx`.
  - DB connection: `const sql = postgres(process.env.POSTGRES_URL!, { ssl: 'require' });` — modify cautiously in `app/lib/data.ts`.

- **Missing / not present in repo**:
  - There are no automated test scripts in `package.json` and no seed script present. `app/lib/placeholder-data.ts` provides demo data for local development.

If any of the above details are incomplete or you want me to include examples for a specific change (new API route, migration, or client->server refactor), tell me which area to expand and I'll iterate. 
