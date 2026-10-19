# 📕 DOKUMEN 4: SYSTEM DESIGN DOCUMENT (SDD), API CONTRACTS, SECURITY GUARDS & 3RD-PARTY INTEGRATIONS

* **Nama Projek:** NewTKK
* **Kode Dokumen:** `DOC-04-SYSTEM-DESIGN-API`
* **Lokasi File:** `docs/04-SYSTEM-DESIGN-API-AND-INTEGRATIONS.md`
* **Versi:** `2.1.0-FINAL` *(True All-In Bundled Pricing + Brand-Borne Mode 3 Shipping + Zero Vehicle Tiers)*
* **Tanggal Efektif:** 20 Oktober 2026
* **Referensi Induk:**
  * `docs/01-MASTER-PRD-NEWTKK.md`
  * `docs/02-FUNCTIONAL-SPECS-4-PORTALS.md`
  * `docs/03-DATABASE-ARCHITECTURE-AND-LEDGER-ERD.md`
* **Target Pembaca:** System Architect, Backend/Frontend/Mobile Engineers, Security Auditors, DevOps, & AI Coding Agents (Claude Code, Cursor, Antigravity, OpenCode)

---

## 1. Arsitektur Monorepo Turborepo & Konfigurasi Compiler (Baseline Oktober 2026)

NewTKK menggunakan arsitektur **Modular Monorepo berbasis Turborepo v2.x dan pnpm v10.x (Catalogs)** agar seluruh aplikasi web (4 portal), aplikasi *mobile* masa depan, *backend API*, skema database, dan kontrak validasi berbagi *Single Source of Truth* yang 100% *type-safe*.

### 1.1. Struktur Pohon Direktori Monorepo (`@newtkk/*`)
```text
newtkk-monorepo/
├── AGENTS.md                                  # Master AI Guardrails & Architectural Rules (identik dengan CLAUDE.md)
├── CLAUDE.md
├── docs/                                      # 5 Dokumen Blueprint Utama Projek (v2.1.0-FINAL)
│   ├── 01-MASTER-PRD-NEWTKK.md
│   ├── 02-FUNCTIONAL-SPECS-4-PORTALS.md
│   ├── 03-DATABASE-ARCHITECTURE-AND-LEDGER-ERD.md
│   ├── 04-SYSTEM-DESIGN-API-AND-INTEGRATIONS.md
│   └── 05-UI-UX-SPECS-AND-PHASE2-ROADMAP.md
├── apps/
│   ├── api-server/                            # Backend Headless Engine (NestJS v12 Native ESM + BullMQ Workers)
│   ├── web-customer/                          # Portal 1: Customer Storefront PWA (Next.js v16.4 + Turbopack)
│   ├── web-brand/                             # Portal 2: Official Brand Dashboard (Next.js v16.4)
│   ├── web-workshop/                          # Portal 3: Workshop Partner Operational Web (Next.js v16.4)
│   └── web-admin/                             # Portal 4: Super Admin Backoffice (Next.js v16.4)
├── packages/
│   ├── contracts/                             # @newtkk/contracts (Zod v4.6.4 + ts-rest / OpenAPI 3.1 Schemas)
│   ├── db/                                    # @newtkk/db (Drizzle ORM v0.45.4/v1.0 + PostgreSQL 18.6 + Seeder)
│   ├── ui/                                    # @newtkk/ui (Shared Tailwind v4 + shadcn/ui + React 19.3 Components)
│   └── shared-utils/                          # @newtkk/shared-utils (SLA Business-Hours Calc, Currency, Crypto, Plate Normalizer)
├── docker/
│   └── docker-compose.infra.yml               # PostgreSQL 18.6 + PostGIS 3.5, Redis 7.4/Valkey, Typesense v27
├── pnpm-workspace.yaml                        # Definisi workspace & pnpm catalogs terpusat
├── turbo.json                                 # Pipeline build, typecheck (tsgo), lint, & test
└── tsconfig.base.json                         # Konfigurasi TypeScript v7.0.2 (Native Go Compiler)
```

### 1.2. Standar `pnpm-workspace.yaml` (pnpm v10 Catalogs) & Kompatibilitas TypeScript v7.0.2
```yaml
# Lokasi File: pnpm-workspace.yaml
packages:
  - "apps/*"
  - "packages/*"

catalog:
  typescript: "^7.0.2"
  "@typescript/typescript6": "^6.0.2"
  zod: "^4.6.4"
  drizzle-orm: "0.45.4"
  drizzle-kit: "^0.31.0"
  "@nestjs/core": "^12.0.0"
  "@nestjs/common": "^12.0.0"
  "@nestjs/platform-fastify": "^12.0.0"
  next: "^16.4.0"
  react: "^19.3.0"
  react-dom: "^19.3.0"
  "@tanstack/react-query": "^5.70.0"
  bullmq: "^5.81.0"
  ioredis: "^5.6.0"
  sharp: "^0.34.0"
  typesense: "^2.0.0"
```

> **Aturan Krusial untuk AI Coding Agent (Next.js v16.4 & TypeScript v7.0.2):**
> 1. Pada **Next.js v16.4**, penamaan file *middleware* lama (`src/middleware.ts`) telah digantikan secara resmi oleh **`src/proxy.ts`** dengan fungsi ekspor `export function proxy(request: NextRequest)`. Dilarang membuat file `middleware.ts`.
> 2. Pada **TypeScript v7.0.2**, pemeriksaan tipe di `turbo.json` dieksekusi oleh *native Go compiler (Corsa)*. Untuk plugin yang masih membutuhkan JS Compiler API lama, gunakan paket pendamping `@typescript/typescript6@^6.0.2`.

---

## 2. Arsitektur Internal Backend (`apps/api-server` — NestJS v12 Modular DDD)

Backend dijalankan di atas **NestJS v12 (Native ESM)** menggunakan adapter HTTP berkinerja tinggi (`@nestjs/platform-fastify`) dan dibagi ke dalam **9 Modul Domain Terisolasi**:

```text
apps/api-server/src/
├── modules/
│   ├── auth/                  # JWT Rotation, WA Passwordless OTP, Google OAuth, Signed Magic Links, Mechanic PIN
│   ├── catalog-fitment/       # 10 Dual-Label Categories, All-In Products, Clone Fitment, int4range GiST, Typesense Sync
│   ├── partner-network/       # Brands, Warehouses, Workshops, Branches (PostGIS), M:N Affiliations, Service Capabilities
│   ├── checkout-shipping/     # Cart Scoping, Mode 2/3 Free Shipping & Free Install Logic, Biteship Mode 1 Quote, Unpaid Guard
│   ├── order-state-machine/   # Transisi Status Ketat, Brand 1-Click Fitment Check, Outbox Event Producer
│   ├── workshop-execution/    # Magic Link Slot Confirm, Inbound Check, QR Check-in, Sharp Live-Camera QC, 14d Forfeit, B2B Report
│   ├── warranty-reviews/      # Digital Warranty Passport, 1x Ownership Transfer, Single-Window 3-Way Claims, Dual-Rating
│   ├── escrow-ledger/         # Double-Entry Accounting Engine, Midtrans Snap/Core Webhook, Two-Phase Iris Payouts (Brand & Refund)
│   └── notifications-sla/     # Business-Hours SLA Calculator (WIB/WITA/WIT), WA Cloud API + Email/SMS Failover
└── common/
    ├── guards/                # JwtAuthGuard, RbacPermissionsGuard, TenantScopeGuard, MechanicPinScopeGuard
    ├── interceptors/          # ZodSerializerInterceptor (Anti-Bypass Output Stripping), IdempotencyInterceptor
    └── outbox/                # Transactional Outbox Relay Worker (PostgreSQL -> BullMQ & Typesense)
```

---

## 3. Keamanan API, `TenantScopeGuard`, Anti-Hoarding & Rate Limiting

### 3.1. Pencegahan Kebocoran Data Antar-Mitra (`TenantScopeGuard` — Anti-IDOR)
Setiap rute pada `/api/v1/brand/*` dan `/api/v1/workshop/*` dilindungi oleh `TenantScopeGuard`:
1. Ketika staf *Brand* melakukan *login*, JWT atau sesi Redis memuat `tenantBrandId` dan `assignedWarehouseId`.
2. Ketika staf/mekanik *Bengkel* melakukan *login*, sesi memuat `tenantWorkshopId` dan daftar `allowedBranchIds`.
3. Controller **dilarang** menerima `brand_id` atau `workshop_branch_id` mentah dari *body/query* tanpa memvalidasinya terhadap konteks `request.tenantContext`. Seluruh *query* Drizzle ORM wajib menyertakan klausa `and(eq(orders.id, orderId), eq(orders.brandId, req.tenant.brandId))`.

### 3.2. Proteksi Anti-Hoarding Kuota Bengkel & Stok Terakhir
Untuk mencegah kompetitor atau bot mengunci seluruh kuota sesi bengkel dan stok barang dengan membuat pesanan gantung (`PENDING_PAYMENT`):
1. **Batas Pesanan Belum Dibayar (`MAX_CONCURRENT_UNPAID_ORDERS = 2`):** Sebelum membuat pesanan baru, `CheckoutService` menghitung `COUNT(*) FROM orders WHERE customer_user_id = $1 AND status = 'PENDING_PAYMENT'`. Jika $\ge 2$, permintaan ditolak dengan kode `429 TOO_MANY_UNPAID_ORDERS`.
2. **Dynamic Last-Slot / Last-Stock Expiry (15 Menit):**
   * Normalnya, `payment_expires_at` adalah `NOW() + 60 minutes`.
   * Apabila sisa kuota sesi bengkel (`max_capacity - booked_count`) tinggal **1 slot terakhir** ATAU sisa stok produk yang dipesan tinggal **1 unit terakhir**, sistem secara otomatis memperpendek `payment_expires_at` menjadi **`NOW() + 15 minutes`**.

### 3.3. Matriks *Sliding-Window Rate Limiting* (Redis + `@nestjs/throttler`)

| Kelompok Endpoint | Batas Request (`Limit / TTL`) | Kunci Identifikasi (`Tracker Key`) | Tindakan Saat Terlampaui (`Penalty`) |
| :--- | :--- | :--- | :--- |
| **Login, Request OTP WA, & Verifikasi OTP** | Maks **5 request / 15 menit** | `ip_address + phone_e164` | Blokir sementara selama **30 menit** (`429 Too Many Requests`). |
| **Verifikasi 4-Digit PIN Mekanik & 6-Digit PIN Booking** | Maks **5 percobaan gagal / 15 menit** | `branch_id + ip_address` | Bekukan sesi PIN mekanik selama **30 menit** + kirim alert ke Owner Bengkel. |
| **Kalkulasi Ongkir Biteship (`/shipping/quotes`)** | Maks **15 request / 1 menit** | `user_id` (atau `ip_address`) | Mengembalikan *cached quote* dari Redis jika parameter identik. |
| **Pembuatan Checkout (`/checkout`)** | Maks **5 request / 1 menit** | `user_id` | Tolak dengan `429` + wajib `Idempotency-Key` di header. |
| **Eksekusi Payout Midtrans Iris (`/wallet/payouts`)** | Maks **3 request / 10 menit** | `owner_user_id` | Tolak dengan `429` + wajib OTP WhatsApp 2FA. |

---

## 4. Kontrak Validasi Zod v4.6.4 & *Anti-Bypass Output Stripping* (`@newtkk/contracts`)

Untuk menegakkan **Skenario 15 (Anti-Bypass)** secara mutlak hingga ke level *Browser Network Tab* sekaligus mengunci jaminan **Gratis Jasa Pasang (`Rp 0`) & Gratis Ongkir ke Bengkel (`Rp 0`)**, respons API untuk pratinjau bengkel sebelum pembayaran menggunakan skema **Zod v4.6.4 (`z.compile()`)**:

```typescript
// Lokasi File: packages/contracts/src/workshop-preview.contract.ts
import { z } from 'zod';

/**
 * 1. Skema Publik Sebelum Pembayaran (Dipakai di PDP & Halaman Checkout)
 * PENTING: TIDAK BOLEH memiliki field fullStreetAddress atau emergencyPhone!
 * Menjamin installationFeeForCustomerIdr = 0 dan shippingFeeToWorkshopForCustomerIdr = 0.
 */
export const PublicWorkshopPreviewSchema = z.object({
  branchId: z.string().uuid(),
  branchDisplayName: z.string(),
  cityName: z.string(),
  districtName: z.string(), // Hanya menampilkan Kecamatan (Contoh: "Kec. Serpong")
  distanceKm: z.number().nonnegative(),
  ratingAvg: z.number().min(0).max(5),
  reviewCount: z.number().int().nonnegative(),
  facilityPhotos: z.array(z.string().url()).max(5),
  fulfillmentModeAvailable: z.enum(['MODE_2_READY_AT_WORKSHOP', 'MODE_3_SHIP_TO_WORKSHOP']),
  localStockAvailableQty: z.number().int().nonnegative(),
  estimatedCourierLeadTimeDays: z.number().int().nonnegative(), // 0 untuk Mode 2, >0 untuk Mode 3
  earliestBookableDateIso: z.string(), // Hasil rumus T_min = H + D_prod + D_ship + 1
  // Jaminan Mutlak Model True All-In O2O bagi Konsumen:
  installationFeeForCustomerIdr: z.literal(0),
  shippingFeeToWorkshopForCustomerIdr: z.literal(0),
  includedConsumables: z.array(z.string()),
  excludedScope: z.array(z.string()),
});

/**
 * 2. Skema Terkunci Pasca-Pembayaran (Hanya dikembalikan jika order.status >= PAID_AWAITING_VALIDATION)
 */
export const UnlockedWorkshopBookingPassSchema = PublicWorkshopPreviewSchema.extend({
  bookingCode: z.string(),
  fullStreetAddress: z.string(),
  emergencyPhone: z.string(),
  googleMapsNavigationUrl: z.string().url(),
  scheduledDate: z.string(),
  scheduledSession: z.enum(['MORNING', 'AFTERNOON', 'EVENING']),
  checkinQrPayload: z.string(),
  checkin6DigitPin: z.string().length(6),
  driverSharePassUrl: z.string().url(),
});

// Kompilasi AOT Zod v4.6.4 untuk serialisasi super cepat di Interceptor NestJS
export const validatePublicWorkshopPreview = z.compile(PublicWorkshopPreviewSchema);
export const validateUnlockedBookingPass = z.compile(UnlockedWorkshopBookingPassSchema);
```

---

## 5. Spesifikasi Lengkap Endpoint REST API (Lintas 4 Portal & Webhook)

Seluruh endpoint memiliki prefiks `/api/v1` dan mengembalikan amplop JSON terstandarisasi:
`{ "success": boolean, "data": T, "meta"?: PaginationMeta, "error"?: { "code": string, "message": string, "details"?: any } }`.

### 5.1. Endpoint Portal 1: Customer Storefront (`/api/v1/storefront/*` & `/api/v1/customer/*`)

| Method & Path | Role Guard | Deskripsi Fungsional & Validasi Inti |
| :--- | :--- | :--- |
| `GET /storefront/categories/tree` | `Public` | Mengambil hierarki 10 Kategori Utama (`ui_short_label`, `full_display_name`, L2, L3, & skema JSONB). Dicache di Redis/Next.js Cache Components. |
| `GET /storefront/vehicles/hierarchy` | `Public` | Mengambil daftar `Make -> Model (beserta chassis_code) -> Year` untuk filter kecocokan suku cadang. |
| `GET /storefront/products/search` | `Public` | Pencarian instan via Typesense dengan parameter `q`, `category_slug`, `make_id`, `model_id`, `year`, `fitment_type`, `sort`, `page`. |
| `GET /storefront/products/:slug/o2o-preview` | `Public` | Mengambil detail SKU (Harga All-In) + daftar Bengkel Rekanan terdekat menggunakan query PostGIS `ST_DWithin` & `ST_DistanceSphere`, diserialisasi dengan `PublicWorkshopPreviewSchema` (Anti-Bypass). |
| `POST /customer/cart/items` | `CUSTOMER_MEMBER` | Menambah/mengubah item keranjang; otomatis mengelompokkan per `brand_id` dan mengecek aturan dominasi `MANDATORY_WORKSHOP_ONLY`. |
| `POST /customer/shipping/home-quotes` | `CUSTOMER_MEMBER` | **Khusus Mode 1 (Kirim ke Rumah):** Memanggil Biteship API berdasarkan `packed_dimensions` SKU menuju Alamat Rumah Konsumen. Menyimpan tarif di Redis selama 15 menit dan mengembalikan `quote_id` bertanda tangan. |
| `GET /customer/workshops/:branchId/calendar` | `CUSTOMER_MEMBER` | Untuk Mode 2 mengembalikan kalender mulai $H+0/H+1$. Untuk Mode 3 mengecek estimasi hari tempuh kurir dari Gudang Brand ke Bengkel, menghitung tanggal minimum $T_{\text{min}} = H + D_{\text{prod}} + D_{\text{ship}} + 1$, serta mengembalikan status kuota sesi (`MORNING`, `AFTERNOON`, `EVENING`) dengan `customerShippingFeeIdr: 0`. |
| `POST /customer/checkout` | `CUSTOMER_MEMBER` | **Zero-Trust Checkout:** Menerima `brand_id`, `fulfillment_mode`, `quote_id` (jika Mode 1), `workshop_branch_id` + `scheduled_date` + `scheduled_session` + deklarasi kendaraan (`plate_number`) (jika Mode 2/3), & `warranty_waiver_accepted` (jika Mode 1). Untuk Mode 2 & Mode 3, server mengunci `customer_paid_shipping_fee_idr = 0`, membuat *Immutable Snapshot*, dan mengembalikan `midtrans_snap_token`. |
| `GET /customer/orders/:orderId/booking-pass` | `CUSTOMER_MEMBER` | Hanya berhasil jika `status >= PAID_AWAITING_VALIDATION`. Membuka alamat lengkap bengkel, QR Code Check-in, PIN 6-Digit, & tautan *Driver Pass*. |
| `GET /public/driver-pass/:shareToken` | `Public (Token)` | Halaman khusus Supir/Perwakilan (Skenario 10): menampilkan peta lokasi bengkel, jadwal, plat nomor, & QR Check-in tanpa menampilkan harga atau akses akun. |
| `POST /customer/orders/:orderId/reschedule` | `CUSTOMER_MEMBER` | Mengubah jadwal kedatangan (memvalidasi `customer_reschedule_count < 2` dan tanggal baru $\le$ `storage_14day_expires_at`). |
| `POST /customer/orders/:orderId/confirm-handover` | `CUSTOMER_MEMBER` | Konfirmasi jarak jauh oleh pemilik akun setelah memeriksa foto QC bengkel -> memicu penyelesaian `COMPLETED_AND_SETTLED`, pencairan *Escrow* ($N_{\text{brand}}$) ke Official Brand, & penerbitan *Digital Warranty Passport*. |
| `POST /customer/orders/:orderId/report-incident` | `CUSTOMER_MEMBER` | Melaporkan insiden kerusakan mobil saat instalasi (Skenario 11) -> mengubah status ke `INCIDENT_UNDER_MEDIATION`. |
| `POST /customer/warranties/:id/claims` | `CUSTOMER_MEMBER` | Mengajukan klaim garansi satu pintu (unggah foto/video kendala & pilih jadwal inspeksi di bengkel rekanan). |
| `POST /customer/warranties/:id/transfer` | `CUSTOMER_MEMBER` | Memindahkan kepemilikan *Digital Warranty Passport* ke pembeli mobil baru (maksimal 1x transfer, wajib OTP pemilik lama). |
| `POST /customer/refund-wallet/withdraw` | `CUSTOMER_MEMBER` | Menarik saldo `Customer Refundable Balance` (hasil *refund* VA/QRIS) secara instan ke rekening bank konsumen via Midtrans Iris API. |

---

### 5.2. Endpoint Portal 2: Official Brand Portal (`/api/v1/brand/*`)

| Method & Path | Role Guard | Deskripsi Fungsional & Validasi Inti |
| :--- | :--- | :--- |
| `POST /brand/products` & `PUT /brand/products/:id` | `BRAND_OWNER`, `BRAND_CATALOG_ADMIN` | Membuat/mengubah SKU (Harga All-In), `packed_dimensions`, aturan garansi, atribut `JSONB`, dan memicu event `PRODUCT_UPSERTED` ke `outbox_events`. |
| `POST /brand/products/:id/clone-fitment` | `BRAND_OWNER`, `BRAND_CATALOG_ADMIN` | Menyalin seluruh baris `product_vehicle_fitments` (`int4range`) dari `source_product_id` ke SKU target dalam 1 transaksi. |
| `POST /brand/products/bulk-import` | `BRAND_OWNER`, `BRAND_CATALOG_ADMIN` | Mengunggah file CSV/Excel katalog & rentang tahun kecocokan mobil dengan *parser* validasi otomatis. |
| `POST /brand/orders/:orderId/fitment-decision` | `BRAND_OWNER`, `BRAND_ORDER_ADMIN` | **1-Klik Validasi Fitment (SLA 4 Jam Kerja):** Menerima aksi `APPROVE_FITMENT`, `FLAG_MISMATCH_SUGGEST_ALT`, atau `REJECT_FITMENT_FULL_REFUND`. |
| `POST /brand/orders/:orderId/dispatch-shipment` | `BRAND_OWNER`, `BRAND_ORDER_ADMIN` | Hanya aktif jika `brand_fitment_status = 'APPROVED'` DAN (untuk Mode 3) `workshop_schedule_confirmed = true`. Memesan *pickup* kurir ke Biteship API (dengan biaya pengiriman Mode 3 dicatat sebagai `brand_borne_shipping_fee_idr`) dan mengembalikan PDF Label Pengiriman (dilengkapi QR Inbound Bengkel). |
| `POST /brand/warranty-claims/:id/triage-decision` | `BRAND_OWNER`, `BRAND_ORDER_ADMIN` | Merespons tiket klaim garansi pada ruang `INTERNAL_3WAY_TRIAGE` dan memicu pengiriman unit pengganti garansi. |
| `POST /brand/wallet/payouts` | `BRAND_OWNER` | Meminta pencairan saldo `BRAND_AVAILABLE` ($N_{\text{brand}}$) ke rekening bank terdaftar via protokol *Two-Phase Midtrans Iris Payout* (Wajib OTP 2FA). |

---

### 5.3. Endpoint Portal 3: Affiliated Workshop Portal (`/api/v1/workshop/*` & `/api/v1/magic-links/*`)

| Method & Path | Role Guard | Deskripsi Fungsional & Validasi Inti |
| :--- | :--- | :--- |
| `GET /magic-links/:token/preview` | `Public (Signed)` | **100% Read-Only (Bebas Efek Samping):** Aman dari *crawler bot* WhatsApp Link Preview. Mengembalikan detail ringkas pesanan untuk ditampilkan di layar konfirmasi. |
| `POST /magic-links/:token/execute` | `Public (Signed)` | Mengeksekusi keputusan jadwal oleh Admin Bengkel (`ACCEPT_SCHEDULE` atau `PROPOSE_NEW_SLOTS`) lalu menandai token sebagai telah digunakan. |
| `PUT /workshop/branches/:id/capabilities` | `WORKSHOP_OWNER`, `WORKSHOP_FRONT_DESK` | Mengatur daftar kapabilitas jenis pekerjaan (`workshop_branch_capabilities`) dan kuota sesi harian cabang. |
| `POST /workshop/bookings/:id/inbound-verify` | `WORKSHOP_OWNER`, `WORKSHOP_FRONT_DESK` | **SOP Inbound Check (Mode 3):** Memindai QR paket yang tiba dari kurir, mengunggah foto segel paket (`SEALED_GOOD` atau `DAMAGED_REJECTED`), mengubah status ke `READY_AT_WORKSHOP_FOR_INSTALL`, dan memulai *timer* **14-Day Storage Expiry**. |
| `POST /workshop/mechanic-session/pin-auth` | `WORKSHOP_OWNER`, `WORKSHOP_FRONT_DESK` | Mengaktifkan mode layar terbatas `WORKSHOP_MECHANIC` menggunakan 4-Digit PIN Mekanik (menyembunyikan seluruh informasi harga & kontak konsumen). |
| `POST /workshop/bookings/:id/checkin` | `WORKSHOP_OWNER`, `WORKSHOP_FRONT_DESK`, `WORKSHOP_MECHANIC` | Memvalidasi QR Code / 6-Digit PIN Check-in dari HP Konsumen/Supir **DAN** mewajibkan unggahan minimal 2 foto `PRE_INSTALL_CONDITION` (kondisi mobil di *stall* & indikator *dashboard*). |
| `POST /workshop/bookings/:id/scan-serial` | `WORKSHOP_OWNER`, `WORKSHOP_FRONT_DESK`, `WORKSHOP_MECHANIC` | Memindai/menginput Nomor Seri untuk setiap baris unit fisik (`order_item_units`) dan mengubah status ke `INSTALLATION_IN_PROGRESS`. |
| `POST /workshop/bookings/:id/report-anomaly` | `WORKSHOP_OWNER`, `WORKSHOP_FRONT_DESK`, `WORKSHOP_MECHANIC` | Melaporkan kendala di lantai bengkel: `ABORTED_BY_VEHICLE_CONDITION` (Skenario 4 — wajib 2 foto kerusakan mobil konsumen), `REPORT_DOA_DEFECT` (Skenario 5 — wajib video cacat pabrik), atau `WORKSHOP_STOCK_DISCREPANCY` (Skenario 7). |
| `POST /workshop/bookings/:id/submit-qc` | `WORKSHOP_OWNER`, `WORKSHOP_FRONT_DESK`, `WORKSHOP_MECHANIC` | Mengunggah foto `POST_INSTALL_QC` (produk terpasang + plat nomor mobil) via *Server-Side Sharp Watermark Pipeline*, mengisi `old_parts_handling` (`RETURNED_TO_TRUNK` / `DISPOSED_WITH_CONSENT`), dan memulai timer *Auto-Complete* 48 jam. |
| `GET /workshop/reports/b2b-installations` | `WORKSHOP_OWNER` | Mengunduh laporan rekapitulasi pekerjaan pemasangan yang telah selesai (`COMPLETED_AND_SETTLED`) per *Official Brand* dalam format Excel/PDF untuk rekonsiliasi insentif B2B. |

---

## 6. Integrasi Midtrans (Snap/Core Pay-In & Two-Phase Iris Payout Engine)

### 6.1. Verifikasi Webhook Pay-In Midtrans (`POST /api/v1/webhooks/midtrans/pay-in`)
1. **Verifikasi Kriptografi `SHA-512` Wajib:**
   Sistem menghitung tanda tangan lokal dan membandingkannya secara *constant-time* (`crypto.timingSafeEqual`) terhadap `payload.signature_key`:
   $$\text{Signature} = \text{SHA512}(\text{order\_id} + \text{status\_code} + \text{gross\_amount} + \text{MIDTRANS\_SERVER\_KEY})$$
2. **Idempotent Inbox Check:**
   Sistem melakukan `INSERT INTO webhook_events_inbox (provider_name, external_event_id, ...)` di mana `external_event_id = payload.transaction_id + '_' + payload.transaction_status`. Jika baris sudah ada (`ON CONFLICT DO NOTHING`), server langsung mengembalikan `200 OK` tanpa memproses ulang.
3. **Aturan State Guard:**
   Jika `transaction_status` adalah `settlement` atau `capture` (dengan `fraud_status == 'accept'`), sistem mengubah status pesanan dari `PENDING_PAYMENT` ke `PAID_AWAITING_VALIDATION` dan mencatat jurnal `ORDER_PAID_ESCROW` di buku besar.

### 6.2. Protokol *Two-Phase Midtrans Iris Payout* (Anti-Double Transfer saat Timeout)
Untuk menjamin kas NewTKK tidak pernah mengalami transfer ganda ketika koneksi ke Midtrans Iris mengalami *HTTP Timeout* saat mencairkan dana ke *Official Brand* atau *Customer Refund Wallet*:

```typescript
// Lokasi File: apps/api-server/src/modules/escrow-ledger/iris-payout.service.ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class IrisPayoutService {
  /**
   * FASE 1: Kunci Dana di Buku Besar Dalam Transaksi Database Lokal SEBELUM Panggil HTTP Eksternal
   */
  async initiateDisbursement(params: {
    recipientType: 'BRAND' | 'CUSTOMER_REFUND';
    entityId: string;
    requestedByUserId: string;
    amountIdr: bigint;
    idempotencyKey: string;
  }) {
    // 1. Di dalam db.transaction(SERIALIZABLE / SELECT FOR UPDATE):
    //    - Kunci baris ledger_accounts (STRICT_ROW_LOCK) milik Brand atau Customer Refund Wallet.
    //    - Pastikan current_balance_idr >= amountIdr.
    //    - Buat Jurnal: DEBIT Akun AVAILABLE_BALANCE -> CREDIT Akun PAYOUT_IN_FLIGHT.
    //    - Buat baris di tabel payout_and_refund_disbursements dengan status = 'QUEUED'
    //      dan simpan idempotencyKey deterministik.
    //    - Tulis job 'EXECUTE_IRIS_HTTP_PAYOUT' ke tabel outbox_events.
  }

  /**
   * FASE 2: Dieksekusi oleh BullMQ Worker dengan Header X-Idempotency-Key Deterministik
   */
  async executeIrisHttpCall(disbursementId: string) {
    const record = await this.findDisbursementById(disbursementId);
    if (record.status === 'COMPLETED') return;

    const response = await fetch(`${process.env.MIDTRANS_IRIS_BASE_URL}/api/v1/payouts`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'Authorization': `Basic ${Buffer.from(process.env.MIDTRANS_IRIS_APPROVER_KEY + ':').toString('base64')}`,
        // KUNCI PENGAMAN UTAMA: Jika BullMQ retry karena timeout, Iris menolak membuat transfer ke-2!
        'X-Idempotency-Key': record.idempotencyKey,
      },
      body: JSON.stringify({
        payouts: [{
          beneficiary_name: record.bankAccountHolderName,
          beneficiary_account: record.bankAccountNumber,
          beneficiary_bank: record.bankCode,
          amount: record.netTransferAmountIdr.toString(),
          notes: `NewTKK Payout ${record.disbursementNumber}`,
        }],
      }),
    });

    // PENTING (FASE 3): Jika terjadi Network Timeout (504 / ECONNRESET), JANGAN kembalikan saldo!
    // Biarkan status tetap 'PROCESSING_IRIS' dan jadwalkan job rekonsiliasi GET /payouts/:ref_no.
    // Saldo hanya boleh dikembalikan (REVERT JOURNAL) jika Iris mengonfirmasi status terminal 'FAILED' / 'REJECTED'.
  }
}
```

---

## 7. Integrasi Logistik Biteship: Mode 3 (Ditanggung Brand) vs Mode 1 (Ditanggung Konsumen)

### 7.1. Kalkulasi Chargeable Weight & Alokasi Beban Ongkir
1. Sistem menghitung berat tagihan (`chargeable_weight_grams`) berdasarkan nilai tertinggi antara Berat Aktual Kemasan vs. Berat Volumetrik:
   $$\text{Volumetric Weight (kg)} = \frac{P_{\text{packed}} \times L_{\text{packed}} \times T_{\text{packed}}}{\text{Divisor}} \quad (\text{Divisor} = 6000 \text{ Reguler}, \, 4000 \text{ Kargo})$$
2. **Alur Mode 3 (*Ship-to-Workshop* — Gratis Ongkir bagi Konsumen, Ditanggung Brand):**
   * Saat konsumen memilih bengkel Mode 3, server mengecek durasi pengiriman kurir dari Gudang Brand ke Cabang Bengkel ($D_{\text{ship}}$) untuk mengunci tanggal kalender minimum $T_{\text{min}} = H + D_{\text{prod}} + D_{\text{ship}} + 1$.
   * Pada *Checkout*, `customer_paid_shipping_fee_idr` dikunci bernilai **`0`**.
   * Saat Admin Brand mencetak label pengiriman dan memesan *pickup* kurir ke Biteship, biaya pengiriman dicatat pada `orders.brand_borne_shipping_fee_idr` ($L_{\text{brand\_ship}}$).
3. **Alur Mode 1 (*Direct Ship-to-Home* — Ongkir Dibayar Konsumen, Selisih Volumetrik Ditanggung Brand):**
   * Saat *Checkout* Mode 1, tarif dari Redis Quote Token ditagihkan kepada konsumen (`customer_paid_shipping_fee_idr` = $L_{\text{mode1\_est}}$).
   * Apabila pihak ekspedisi/Biteship mengirimkan *webhook* koreksi berat aktual di mana $L_{\text{mode1\_actual}} > L_{\text{mode1\_est}}$ akibat Brand salah menginput dimensi kemasan, maka selisih kurang bayar $\Delta L_{\text{undercharge}} = L_{\text{mode1\_actual}} - L_{\text{mode1\_est}}$ dimasukkan ke dalam `orders.brand_borne_shipping_fee_idr` dan dipotong otomatis dari pencairan bersih Brand ($N_{\text{brand}}$).

---

## 8. Pipeline Media Cloudflare R2 & *Server-Authoritative Sharp Watermarking*

Seluruh foto bukti operasional di bengkel (`PRE_INSTALL_CONDITION`, `POST_INSTALL_QC`, dan `INBOUND_CHECK`) wajib melewati *pipeline* verifikasi server berikut:

```typescript
// Lokasi File: apps/api-server/src/modules/workshop-execution/qc-media-watermark.service.ts
import { Injectable, BadRequestException } from '@nestjs/common';
import sharp from 'sharp';

@Injectable()
export class QcMediaWatermarkService {
  async processAndUploadQcPhoto(params: {
    fileBuffer: Buffer;
    bookingCode: string;
    plateNumber: string;
    stageType: 'PRE_INSTALL_CONDITION' | 'POST_INSTALL_QC' | 'INBOUND_CHECK';
    captureSource: 'LIVE_CAMERA' | 'GALLERY_FALLBACK';
    clientGpsLat: number;
    clientGpsLng: number;
    workshopBranchLat: number;
    workshopBranchLng: number;
    branchTimezone: string;
  }) {
    const metadata = await sharp(params.fileBuffer).metadata();
    if (!['jpeg', 'png', 'webp'].includes(metadata.format ?? '')) {
      throw new BadRequestException('Format file gambar tidak valid.');
    }

    const distanceMeters = this.calculateHaversineMeters(
      params.clientGpsLat, params.clientGpsLng,
      params.workshopBranchLat, params.workshopBranchLng
    );
    const gpsStatusFlag = distanceMeters <= 500 ? 'GPS_VERIFIED_AT_WORKSHOP' : `GPS_DEVIATION_${Math.round(distanceMeters)}M`;

    const serverTimeFormatted = new Intl.DateTimeFormat('id-ID', {
      timeZone: params.branchTimezone,
      dateStyle: 'medium',
      timeStyle: 'long',
    }).format(new Date());

    const targetWidth = 1280;
    const svgBanner = Buffer.from(`
      <svg width="${targetWidth}" height="90">
        <rect x="0" y="0" width="${targetWidth}" height="90" fill="rgba(0, 0, 0, 0.72)" />
        <text x="20" y="34" font-family="sans-serif" font-size="22" font-weight="bold" fill="#10B981">
          NEWTKK VERIFIED QC | BOOKING: ${params.bookingCode} | PLAT: ${params.plateNumber}
        </text>
        <text x="20" y="68" font-family="sans-serif" font-size="18" fill="#FFFFFF">
          STAGE: ${params.stageType} | SERVER TIME: ${serverTimeFormatted} | LOC: ${gpsStatusFlag} | SRC: ${params.captureSource}
        </text>
      </svg>
    `);

    const watermarkedWebpBuffer = await sharp(params.fileBuffer)
      .rotate()
      .resize({ width: targetWidth, withoutEnlargement: true })
      .composite([{ input: svgBanner, gravity: 'south' }])
      .webp({ quality: 82 })
      .toBuffer();

    return { watermarkedWebpBuffer, distanceMeters, gpsStatusFlag, serverTimeFormatted };
  }

  private calculateHaversineMeters(lat1: number, lon1: number, lat2: number, lon2: number): number {
    const R = 6371e3;
    const toRad = (deg: number) => (deg * Math.PI) / 180;
    const dLat = toRad(lat2 - lat1);
    const dLon = toRad(lon2 - lon1);
    const a = Math.sin(dLat / 2) ** 2 + Math.cos(toRad(lat1)) * Math.cos(toRad(lat2)) * Math.sin(dLon / 2) ** 2;
    return R * 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
  }
}
```

---

## 9. Mesin Kalkulasi SLA Jam Kerja (*Business-Hours Aware*) & BullMQ Workers

| Nama Queue BullMQ | Nama Job & Trigger | Logika Eksekusi & Penanganan Kegagalan (`v2.1.0-FINAL`) |
| :--- | :--- | :--- |
| `q-order-lifecycle` | `EXPIRE_UNPAID_ORDER`<br>*(Delay: 60m atau 15m jika stok/slot terakhir)* | Mengecek apakah `orders.status === 'PENDING_PAYMENT'`. Jika ya, ubah menjadi `PAYMENT_EXPIRED`, kurangi `stock_reserved` di gudang/bengkel, dan kurangi `booked_count` pada `workshop_daily_session_quotas`. |
| `q-sla-monitors` | `CHECK_BRAND_FITMENT_SLA`<br>*(Delay: 4 jam kerja sesuai `BusinessHoursSlaCalculator`)* | Jika setelah 4 jam kerja Brand belum memvalidasi *fitment*, kirim *High-Priority Alert* ke WhatsApp Owner Brand & tim `OPS_AND_MEDIATOR` NewTKK. |
| `q-sla-monitors` | `CHECK_WORKSHOP_CONFIRM_SLA`<br>*(Delay: 8 jam kerja lokal bengkel)* | Jika Bengkel belum merespons konfirmasi jadwal, kirim pengingat darurat & tawarkan opsi *Reroute* ke bengkel rekanan terdekat kepada konsumen. |
| `q-storage-forfeit` | `STORAGE_REMINDER_AND_FORFEIT`<br>*(Dijadwalkan saat paket tiba di bengkel)* | • **Hari ke-3, ke-7, ke-12:** Kirim pengingat WhatsApp ke konsumen untuk segera datang/reschedule.<br>• **Hari ke-15 (00:01 Lokal):** Eksekusi otomatis Skenario 3 (`STORAGE_EXPIRED_FORFEITED`): cairkan 100% hak produk bersih ($N_{\text{brand}} = P_{\text{net}} - C_{\text{brand}} - L_{\text{brand\_ship}}$) ke saldo *Official Brand* & tandai garansi `VOID_DUE_TO_NO_SHOW`. |
| `q-handover-settlement` | `AUTO_COMPLETE_HANDOVER_48H`<br>*(Delay: 48 jam sejak QC diunggah)* | Jika status masih `AWAITING_CUSTOMER_HANDOVER` dan tidak ada laporan insiden, otomatis ubah status menjadi `COMPLETED_AND_SETTLED`, jalankan jurnal *Settlement* ($N_{\text{brand}}$ cair ke Brand), dan terbitkan *Digital Warranty Passport*. |
| `q-notifications` | `DISPATCH_MULTICHANNEL_NOTIF`<br>*(Dipicu oleh `OutboxRelayWorker`)* | Mengirim pesan via **WhatsApp Business Cloud API**. Memantau *delivery status* dalam 30 detik; jika gagal, otomatis mengeksekusi *Failover* ke **Email Transaksional & SMS**. |

---

## 10. Skema Indeks Pencarian Instan Typesense (`products_search_index`)

```json
{
  "name": "products_search_index",
  "fields": [
    { "name": "id", "type": "string" },
    { "name": "sku_code", "type": "string" },
    { "name": "oem_part_number", "type": "string", "optional": true },
    { "name": "product_name", "type": "string" },
    { "name": "brand_id", "type": "string", "facet": true },
    { "name": "brand_name", "type": "string", "facet": true },
    { "name": "root_category_code", "type": "string", "facet": true },
    { "name": "ui_short_label", "type": "string", "facet": true },
    { "name": "category_lvl3_slug", "type": "string", "facet": true },
    { "name": "fitment_type", "type": "string", "facet": true },
    { "name": "fulfillment_policy", "type": "string", "facet": true },
    { "name": "availability_type", "type": "string", "facet": true },
    { "name": "all_in_effective_price_idr", "type": "int64", "sort": true },
    { "name": "includes_free_installation", "type": "bool" },
    { "name": "includes_free_workshop_shipping", "type": "bool" },
    { "name": "warranty_duration_months", "type": "int32", "facet": true },
    { "name": "compatible_make_ids", "type": "string[]", "facet": true },
    { "name": "compatible_model_ids", "type": "string[]", "facet": true },
    { "name": "compatible_model_year_keys", "type": "string[]", "facet": true },
    { "name": "search_keywords_combined", "type": "string" },
    { "name": "rating_avg", "type": "float", "sort": true },
    { "name": "sold_count", "type": "int32", "sort": true },
    { "name": "is_active", "type": "bool" }
  ],
  "default_sorting_field": "sold_count"
}
```
