# AI Malware Inspector

An AI-powered Android APK malware analysis platform for security researchers and SOC analysts.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/malware-inspector run dev` — run the frontend (port 23355)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string
- Required env: `GEMINI_API_KEY` — Google Gemini API key for AI threat reports

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite + Tailwind CSS + shadcn/ui + Recharts + Wouter
- API: Express 5 + Multer (file uploads)
- DB: PostgreSQL + Drizzle ORM
- AI: Google Gemini 2.5 Flash (threat intelligence reports)
- APK Parsing: adm-zip (ZIP extraction), custom manifest parser
- Validation: Zod (zod/v4), drizzle-zod
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/api-spec/openapi.yaml` — OpenAPI contract (source of truth)
- `lib/db/src/schema/analyses.ts` — analyses table schema
- `lib/db/src/schema/reports.ts` — AI threat reports table schema
- `artifacts/api-server/src/routes/analyses/index.ts` — all analysis, report, and dashboard routes
- `artifacts/api-server/src/lib/analyzer.ts` — static APK analysis engine
- `artifacts/api-server/src/lib/gemini.ts` — Gemini AI report generation
- `artifacts/api-server/uploads/` — uploaded APK files
- `artifacts/malware-inspector/src/pages/` — all frontend pages

## Architecture decisions

- APK files are analyzed server-side using adm-zip to extract AndroidManifest.xml and DEX files
- Analysis runs asynchronously after upload — frontend polls every 3 seconds while pending/processing
- AI report generation is on-demand (user triggers it) to avoid automatic Gemini API costs
- File deduplication via SHA-256 hash prevents re-analyzing identical APKs
- All risk scoring is deterministic (no AI) — AI is only used for narrative report generation
- Static analysis scans: dangerous permissions, suspicious URLs, hardcoded secrets, obfuscation, reflection, dynamic loading, native libraries

## Product

- Upload APK files via drag-and-drop
- Automatic static analysis: reverse engineering (manifest parsing) + security findings
- Risk score 0-100 with levels: Safe / Low Risk / Suspicious / High Risk / Malware
- On-demand AI threat intelligence reports via Gemini (executive summary, classification, recommendations)
- SOC dashboard: metrics, risk distribution charts, threat category breakdown, recent activity
- Analysis history with status tracking and filtering
- Threat reports library

## Gotchas

- Real APKs have binary-encoded AndroidManifest.xml (AXML format) — current parser handles text-format manifests. For production binary AXML parsing, use androguard (Python) or apktool.
- After schema changes: run `pnpm --filter @workspace/db run push` then restart API server
- After OpenAPI spec changes: run `pnpm --filter @workspace/api-spec run codegen` then `pnpm run typecheck:libs`
- Upload limit: 100MB per APK file

## User preferences

_Populate as you build — explicit user instructions worth remembering across sessions._

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
