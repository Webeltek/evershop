# Copilot / AI agent instructions — EverShop

Purpose: Quickly orient an AI coding agent to be effective in this repository by highlighting architecture, developer workflows, conventions, and common pitfalls specific to EverShop.

## Quick start (useful commands)
- Docker quick start: `curl -sSL https://raw.githubusercontent.com/evershopcommerce/evershop/main/docker-compose.yml > docker-compose.yml && docker compose up -d`
- Local development (edit → build → run):
  - `npm run compile` (swc compile) or `npm run compile:tsc` (tsc + copy static files)
  - `npm run dev` (runs compiled `./packages/evershop/dist/bin/dev`)
- Run server: `npm run start` (requires compiled `dist`)
- Seed sample data: `npm run seed:all`
- Build for release: `npm run build` (uses `dist/bin/build`)
- Tests: `npm run test` (note: **compile before running tests**; jest runs tests in `dist`)
- Lint & fix: `npm run lint`

## Big-picture architecture
- Monorepo with yarn/npm workspaces: top-level `packages/*` (main product in `packages/evershop`) and `extensions/*`.
- Core structure (packages/evershop): modules–based design: `src/modules/<module>/` typically contains `api/`, `pages/`, `services/`, `graphql/`, and `tests/`.
- Single codebase provides server, admin UI and storefront UI. GraphQL is first-class (see `modules/graphql/services/buildSchema.js`).
- Commands and CLI are exposed via `bin/evershop` (see `packages/evershop/dist/bin` and `package.json` `bin` and scripts).

## Key developer workflows & gotchas for AI
- Important: **Jest is configured to run tests from `dist`** (see `jest.config.js`). Always run `npm run compile` (or `prepack`) before `npm run test` or running dev servers that depend on `dist`.
- Do **not** edit anything in `/dist` directly. Change `src/*` and then compile.
- The repo uses ESM/NodeNext (`"type": "module"` + tsconfig `module: NodeNext`). Use ESM-style imports and be mindful of extensions/paths in compiled output.
- Many scripts expect the compiled CLI in `packages/evershop/dist/bin/*` — ensure `dist` is up-to-date for dev and tests.
- Use `npm run lint` to auto-fix lintable issues; Husky hooks are installed via `npm run prepare`.

## Pages & routing conventions (very important)
- Pages are located in `src/modules/<module>/pages/` and split between `admin/` and `frontStore/`.
- Route metadata sits next to page components in `route.json` and (optionally) `payloadSchema.json` for API endpoints.
  - Example: `src/modules/customer/pages/frontStore/resetPasswordPage/route.json`
- Filenames use bracket syntax to specify params/middleware/response types. Examples:
  - `pages/frontStore/product/[loadProduct]loadProductImage.js` — middleware/loaders named in filename
  - `pages/global/[response]errorHandler.ts` — response handler pattern
- When adding a page endpoint that accepts JSON, include `payloadSchema.json` for AJV validation (project uses `ajv`).

## GraphQL & services
- Resolver and type files: `src/modules/<module>/graphql/types/...` (e.g. `modules/oms/graphql/types/BestSeller/BestSeller.admin.resolvers.js`).
- Schema builder runs at bootstrap (`modules/graphql/services/buildSchema.js`); adding types there typically gets picked up by the system.
- Business logic and DB queries live in `src/modules/<module>/services/` (many helpers are exported from `index.ts` in services). Use existing `get*BaseQuery` functions as canonical examples.

## Database & seeds
- Uses Postgres (`pg`). `packages/postgres-query-builder` is used internally.
- Seed images and CSVs live under `/seed/` and `/translations/`. Use `npm run seed:all` to populate test/demo data.

## Packaging & public API
- `packages/evershop/package.json` declares `exports` mapping to `dist` — if you add a public helper/library, add appropriate export entries.
- Prepack step uses `tsc` to produce `dist/types` and copies static assets (`.graphql`, `.scss`, `.json`).

## Tooling and conventions
- TypeScript config: `tsconfig.json` uses `NodeNext`, emits `.d.ts` into `dist/types`.
- ESLint + Prettier used for style; Tailwind/Tailwind Loader present for CSS.
- Webpack config aliases `react`/`react-dom` to root `node_modules` to avoid duplicate React versions — preserve this alias when adding new packages or extensions.

## Tests & CI notes
- Jest runs unit tests from `dist` (see `jest.config.js` `testMatch`).
- Some tests require `ALLOW_CONFIG_MUTATIONS=true` (set by `npm run test`).
- E2E tests use Cypress (see `devDependencies`).

## Examples (where to look when implementing)
- Adding a new admin grid page: follow examples in `modules/customer/pages/admin/customerGrid/` and add a `route.json` + component JS/TSX.
- Adding a GraphQL type + resolvers: check `modules/catalog/graphql/types/*` and `modules/oms/graphql/types/*`.
- Registering defaults: many modules expose `registerDefault*` in `services/` (e.g., `registerDefaultProductCollectionFilters.js`) — use these to hook defaults.

## For AI agents (short, concrete rules) ✅
- Edit `src/*` only; run `npm run compile` before `npm run dev` / `npm run test`.
- Prefer adding unit tests alongside your feature in `src/` then compile to verify they land in `dist/tests/...`.
- Keep import style ESM and respect tsconfig `paths` and aliases.
- When changing public APIs, update `packages/evershop/package.json` `exports` if consumers need the API.
- Do not add logic in `dist/` or change compiled artifacts directly.

---
If you'd like, I can: (1) add a short checklist for PRs, (2) extract concrete example snippets for page/middleware filenames, or (3) add common debugging commands (process, logs, and inspectable runtime files). Which would you prefer? 
