# 🤖 NEWTKK — MASTER AI AGENT INSTRUCTIONS & ARCHITECTURAL GUARDRAILS (`AGENTS.md` / `CLAUDE.md`)

* **Project Name:** NewTKK (O2O Official Automotive Aftermarket, Spareparts & Affiliated Workshop Marketplace)
* **Architecture Paradigm:** Modular Monorepo (Turborepo v2.x + pnpm v10 Catalogs), 100% End-to-End Type-Safe, Headless API-First
* **Version:** `2.1.0-FINAL` (True All-In Bundled Pricing + Brand-Borne Mode 3 Workshop Shipping + Zero Vehicle Tiers)
* **Baseline Date:** October 20, 2026

---

## 1. MANDATORY DOCUMENTATION ROUTING (READ BEFORE CODING)

You MUST treat the `docs/` directory as the immutable **Single Source of Truth**. Never guess business rules, state transitions, fee formulas, or database columns. Before writing or modifying code for any feature, read the corresponding blueprint document:

1. **`docs/01-MASTER-PRD-NEWTKK.md` (Business Rules & State Machine):**
   * Read this for: **True All-In Pricing Model** (Product price includes FREE professional installation at Affiliated Workshops, zero vehicle complexity tiers), 3 Fulfillment Modes (Mode 1 Direct Home paid shipping by customer + *Warranty Void Waiver*; Mode 2 Ready at Workshop free install & free shipping; Mode 3 Ship-to-Workshop free install & **free shipping for customer, 100% shipping cost borne by Official Brand**), 10 Dual-Label Root Categories, Master Order State Machine, **16 Enterprise Edge-Case Scenarios**, Single-Window Digital Warranty, and Net Settlement Math Formulas.
2. **`docs/02-FUNCTIONAL-SPECS-4-PORTALS.md` (Portals, RBAC & Acceptance Criteria):**
   * Read this for: 10 Granular RBAC Roles across 4 Portals (`web-customer`, `web-brand`, `web-workshop`, `web-admin`), Multi-Warehouse/Multi-Branch rules, Instant O2O Workshop Preview (All-In Free Install & Free Workshop Shipping badges), 1-Click Fitment Check, Mechanic 4-Digit PIN Mode, B2B Workshop Reconciliation Report, and Acceptance Criteria (`[AC-*]`).
3. **`docs/03-DATABASE-ARCHITECTURE-AND-LEDGER-ERD.md` (43 Production Tables & Double-Entry Ledger):**
   * Read this for: Exact PostgreSQL 18.6 + PostGIS 3.5 DDL, Drizzle ORM schemas (`@newtkk/db`), `uuidv7()`, `int4range` GiST indexes, `workshop_branch_capabilities`, Immutable Transaction Snapshots, `order_item_units` serialization, Zero-Sum Double-Entry Accounting matrices, and Master Seeder datasets.
4. **`docs/04-SYSTEM-DESIGN-API-AND-INTEGRATIONS.md` (API Contracts, Security & 3rd-Party Integrations):**
   * Read this for: `@newtkk/contracts` (Zod v4.6.4), REST API endpoints, Anti-Bypass Output Stripping, Anti-Hoarding guards, Two-Phase Midtrans Iris Payouts, Biteship Mode 3 Brand-Borne Shipping & Volumetric Undercharge Guard, Server-Side `sharp` Watermarking to Cloudflare R2, Transactional Outbox, and BullMQ Business-Hours SLA workers.
5. **`docs/05-UI-UX-SPECS-AND-PHASE2-ROADMAP.md` (Routes, Wireframes & Sprint Plan):**
   * Read this for: Next.js 16.4 App Router file trees, High-Contrast Workshop UI tokens (`@newtkk/ui`), 7 Critical Component Wireframe Specs, Checkout Conversion Safeguards, and the 6-Sprint Execution Order.

---

## 2. STRICT TECH STACK & VERSION GUARDRAILS (OCTOBER 2026 BASELINE)

Do NOT use deprecated patterns from older framework versions. Enforce these exact specifications:

* **Runtime & Package Manager:** Node.js **v24.x / v26 LTS** + **pnpm v10.x**. Always define shared dependency versions in `pnpm-workspace.yaml` under `catalog:` and reference them via `"catalog:"` in `package.json`.
* **TypeScript v7.0.2 (Native Go Compiler - Corsa):**
  * Enforce `"strict": true`, `"noUncheckedIndexedAccess": true`.
  * Keep `@typescript/typescript6@^6.0.2` installed alongside `typescript@^7.0.2` for plugins requiring the legacy JS Compiler API.
* **Next.js v16.4 + React 19.3 (4 Web Portals):**
  * **CRITICAL NEXT.JS 16.4 RULE:** Do **NOT** create `src/middleware.ts`. Next.js 16.4 replaces `middleware.ts` with **`src/proxy.ts`** exporting `export function proxy(request: NextRequest)`.
  * Use Turbopack as the default bundler and leverage Next.js 16.4 Cache Components where appropriate.
* **Backend API (`apps/api-server`):** **NestJS v12.x (Native ESM)** on `@nestjs/platform-fastify`.
* **Validation & Shared Contracts (`packages/contracts`):** **Zod v4.6.4**. Use `z.compile(Schema)` for high-throughput request/response validation and output stripping.
* **Database & ORM (`packages/db`):** **PostgreSQL v18.6** + **PostGIS v3.5+** + **Drizzle ORM v0.45.4 (v1.0 Ready)**.

---

## 3. TEN NON-NEGOTIABLE CODING, COMMERCIAL & SECURITY LAWS

Every line of code generated for NewTKK MUST obey these 10 laws:

### Law 1: Zero Floating-Point Money (`BIGINT` Rupiah Only)
* Never use `number` floats, `FLOAT`, or `DOUBLE` for currency.
* In PostgreSQL, all monetary columns (`*_idr`) are `BIGINT NOT NULL`. In TypeScript business logic, calculate money using `bigint` or safe integer arithmetic, and always verify $\sum \text{DEBIT} = \sum \text{CREDIT}$ before committing any `ledger_journals` transaction.

### Law 2: True All-In Bundled Pricing & Brand-Borne Mode 3 Shipping
* **Zero Vehicle Tiers & Free Installation:** There are NO vehicle complexity tiers (`TIER_1`–`TIER_4`) and NO installation fees charged to customers. Product catalog prices (`catalog_price_idr`) are **All-In** and include free installation at any Affiliated Workshop (`installationFeeForCustomerIdr = 0`). B2B workshop service compensation is settled outside the platform between Official Brands and Workshops.
* **Free Workshop Shipping for Customers (Mode 2 & Mode 3):** When a customer selects `MODE_2_READY_AT_WORKSHOP` or `MODE_3_SHIP_TO_WORKSHOP`, `customer_paid_shipping_fee_idr` MUST be **`0`**. For Mode 3, the courier shipping cost from the Brand Warehouse to the Affiliated Workshop (`brand_borne_shipping_fee_idr`) is **100% borne by the Official Brand** and deducted from the Brand's net settlement ($N_{\text{brand}} = P_{\text{net}} - C_{\text{brand}} - L_{\text{brand\_ship}}$).
* **Mode 1 (Direct Ship-to-Home):** Customer pays the same catalog price (zero discount), pays the shipping fee to their home (`customer_paid_shipping_fee_idr > 0`), and MUST accept the **Warranty Void Waiver**.

### Law 3: Native `uuidv7()` Primary Keys & Partial Unique Indexes
* Every table's primary key MUST use PostgreSQL 18 native `DEFAULT uuidv7()`.
* Every unique index on a table that has a `deleted_at` column MUST be a **Partial Unique Index** (`WHERE deleted_at IS NULL`) so soft-deleted SKUs/slugs never cause duplicate key errors on re-creation.

### Law 4: Strict Anti-Bypass Output Stripping (Scenario 15)
* Public endpoints (`GET /storefront/products/:slug/o2o-preview`, Workshop Directory, and Cart Preview) MUST serialize responses through `PublicWorkshopPreviewSchema`.
* **NEVER** expose `full_street_address`, `emergency_phone`, or `checkin_6digit_pin` in any HTTP response until `order.status >= 'PAID_AWAITING_VALIDATION'`.

### Law 5: Zero-Trust Checkout & Immutable Snapshots
* `POST /api/v1/customer/checkout` MUST ignore any price or shipping fee numbers sent from the client browser.
* Fetch product prices from `products` and shipping quotes/lead-times from the signed Redis key `shipping_quote:{quote_id}`.
* Freeze all prices, commission rates, shipping allocations, and `packed_dimensions` into the `orders` and `order_items` snapshot columns at checkout creation.

### Law 6: Unit-Level Physical Serialization (`order_item_units`)
* Whenever an `order_items` row is created with `quantity = N`, immediately generate $N$ child rows in `order_item_units` (`unit_sequence_number: 1..N`).
* Serial number scanning, DOA defect reporting (Scenario 5), partial fulfillment/refunds (Scenario 9), and `digital_warranties` issuance MUST bind to `order_item_unit_id`, never just `order_item_id`.

### Law 7: Two-Phase Midtrans Iris Payouts & Idempotent Webhooks
* Never call `POST /iris/api/v1/payouts` without first locking the account row (`SELECT ... FOR UPDATE`), moving the balance to `PAYOUT_IN_FLIGHT`, and passing a deterministic `X-Idempotency-Key`.
* If an Iris HTTP call times out (`504` / `ECONNRESET`), **NEVER** automatically revert the ledger balance; wait for status reconciliation.
* All incoming webhooks (Midtrans Pay-In, Midtrans Iris, Biteship) MUST be deduplicated via `webhook_events_inbox (provider_name, external_event_id)` before state mutation.

### Law 8: Two-Step Read-Only GET for WhatsApp Signed Magic Links
* `GET /m/[magicToken]` and `GET /api/v1/magic-links/:token/preview` MUST be **100% Read-Only** and idempotent so WhatsApp/Meta link-preview crawlers do not prematurely consume tokens or trigger state changes.
* State mutation (`ACCEPT_SCHEDULE` / `PROPOSE_RESCHEDULE`) may only occur on an explicit interactive `POST` request.

### Law 9: Server-Authoritative `sharp` Watermarking & Mechanic Financial Blindness
* All workshop operational photos (`PRE_INSTALL_CONDITION`, `POST_INSTALL_QC`, `INBOUND_CHECK`, `INCIDENT_EVIDENCE`) MUST be processed server-side via `QcMediaWatermarkService` using `sharp` (burning server timestamp in local `iana_timezone`, `booking_code`, `plate_number`, and PostGIS GPS verification).
* All payloads returned to `WORKSHOP_MECHANIC` (4-Digit PIN session) MUST strip all Rupiah price fields.

### Law 10: Transactional Outbox for BullMQ & Typesense
* Never call `bullmq.add()` or `typesense.upsert()` directly inside a controller without durability. Write an event row to `outbox_events` inside the same PostgreSQL ACID transaction as your state change, and let `OutboxRelayWorker` dispatch it reliably.

---

## 4. COMMAND REFERENCE FOR AI AGENTS

```bash
# 1. Start local infrastructure (PostgreSQL 18.6 + PostGIS 3.5, Redis 7.4, Typesense v27)
docker compose -f docker/docker-compose.infra.yml up -d

# 2. Install workspace dependencies (using pnpm v10 catalog)
pnpm install

# 3. Generate & apply Drizzle ORM migrations + run Master Seeder (Categories, Vehicles, Configs)
pnpm --filter @newtkk/db db:generate
pnpm --filter @newtkk/db db:migrate
pnpm --filter @newtkk/db db:seed

# 4. Run ultra-fast monorepo typecheck (TypeScript 7.0.2 Go Compiler) & unit tests
pnpm turbo run typecheck test

# 5. Start all 4 Next.js 16.4 portals + NestJS 12 API server in development mode
pnpm turbo run dev
```
