# 📙 DOKUMEN 3: DATABASE ARCHITECTURE (ERD), DOUBLE-ENTRY LEDGER & MASTER SEEDER SPECIFICATION

* **Nama Projek:** NewTKK
* **Kode Dokumen:** `DOC-03-DATABASE-ERD`
* **Lokasi File:** `docs/03-DATABASE-ARCHITECTURE-AND-LEDGER-ERD.md`
* **Package Monorepo Target:** `packages/db` (`@newtkk/db`)
* **Versi:** `2.1.0-FINAL` *(42 Production Tables — All-In Bundled Pricing, Brand-Borne Mode 3 Shipping & Zero Tiers)*
* **Tanggal Efektif:** 20 Oktober 2026
* **Database Engine:** PostgreSQL v18.6 + PostGIS v3.5+ + Drizzle ORM v0.45.4 (v1.0 Ready)
* **Referensi Induk:** `docs/01-MASTER-PRD-NEWTKK.md` & `docs/02-FUNCTIONAL-SPECS-4-PORTALS.md`
* **Target Pembaca:** Database Administrator, Backend Architect, Full-Stack Engineers, Finance System Auditors, & AI Coding Agents (Claude Code, Cursor, Antigravity, OpenCode)

---

## 1. Standar Konvensi Database, Ekstensi Wajib & 11 Pilar Pengaman

Seluruh skema di dalam paket `@newtkk/db` wajib mematuhi aturan dasar PostgreSQL 18.6 berikut tanpa pengecualian:

### 1.1. Ekstensi PostgreSQL 18.6 Wajib
```sql
CREATE EXTENSION IF NOT EXISTS "postgis";      -- Kalkulasi geospasial radius bengkel & gudang (GEOGRAPHY)
CREATE EXTENSION IF NOT EXISTS "btree_gist";   -- Menggabungkan UUID & int4range dalam satu GiST Index
CREATE EXTENSION IF NOT EXISTS "pg_trgm";      -- Pencarian teks trigram cepat untuk plat nomor & nama part
CREATE EXTENSION IF NOT EXISTS "pgcrypto";     -- Hashing kriptografi untuk PIN & token
```

### 1.2. Standar Tipe Data & Penamaan Kolom
1. **Primary Key (`id`):** Wajib menggunakan fungsi bawaan PostgreSQL 18 **`UUID DEFAULT uuidv7()`** (Time-ordered UUID yang menjaga lokalitas halaman *B-Tree Index* dan menghilangkan fragmentasi *insert*).
2. **Satuan Mata Uang (`*_idr`):** Seluruh kolom nominal uang wajib menggunakan **`BIGINT NOT NULL`** dalam satuan Rupiah penuh (tanpa desimal sen). Penggunaan `FLOAT`, `REAL`, atau `DOUBLE PRECISION` untuk uang **dilarang keras**.
3. **Persentase Komisi (`*_rate`):** Menggunakan **`NUMERIC(6, 4)`** (contoh: `0.0800` untuk 8%).
4. **Waktu & Zona Waktu (`*_at`):** Seluruh *timestamp* wajib menggunakan **`TIMESTAMPTZ`** (disimpan dalam UTC). Setiap entitas fisik (Gudang Brand & Cabang Bengkel) wajib menyimpan kolom **`iana_timezone VARCHAR(64)`** (`'Asia/Jakarta' | 'Asia/Makassar' | 'Asia/Jayapura'`).
5. **Soft Delete & Partial Unique Index:** Tabel master menggunakan kolom `deleted_at TIMESTAMPTZ`. Setiap constraint unik pada tabel yang memiliki `deleted_at` wajib didefinisikan sebagai **Partial Unique Index (`WHERE deleted_at IS NULL`)** agar kode SKU atau slug lama dapat digunakan kembali setelah dihapus.

### 1.3. Ringkasan 11 Pilar Pengaman Database NewTKK (`v2.1.0-FINAL`)
1. **Immutable Transaction Snapshots:** Harga All-In SKU, diskon, % komisi Brand, beban ongkir (`customer_paid_shipping_fee_idr` vs `brand_borne_shipping_fee_idr`), dimensi kemasan, dan syarat garansi dibekukan pada baris `orders` & `order_items` saat *checkout*.
2. **Database-Enforced Free Workshop Shipping:** Constraint `chk_free_workshop_shipping_for_customer` di level kernel PostgreSQL menjamin bahwa untuk Mode 2 dan Mode 3, `customer_paid_shipping_fee_idr` **wajib bernilai `0`**.
3. **Native `int4range` + GiST Index:** Pemetaan tahun kecocokan mobil menggunakan `int4range` (contoh: `[2016, 2026]`) dengan indeks GiST (< 1ms).
4. **Chassis-Separated Vehicle Models:** Memisahkan kode sasis (`chassis_code`) di level model untuk mencegah tabrakan tahun transisi (misal *Kijang Innova Reborn AN140 2022* vs *Kijang Innova Zenix AG10 2022*).
5. **Unit-Level Physical Serialization (`order_item_units`):** Pembelian `quantity > 1` (misal 4 ban) dipecah menjadi baris per unit fisik untuk melacak Nomor Seri, bukti DOA, *partial refund*, dan klaim garansi per unit.
6. **Atomic Capacity & Inventory CHECK Constraints:** Kuota sesi harian bengkel dan stok gudang/bengkel dijaga dengan `CHECK (available >= 0)` di level kernel PostgreSQL.
7. **Zero-Sum Double-Entry Accounting Ledger:** Setiap mutasi dana wajib memiliki jurnal berpasangan di mana $\sum \text{DEBIT} = \sum \text{CREDIT}$.
8. **Hybrid Ledger Concurrency Policy:** Akun dompet Brand dan Konsumen menggunakan `STRICT_ROW_LOCK` (`SELECT ... FOR UPDATE`), sedangkan akun kas/kliring sistem global menggunakan `ATOMIC_APPEND_ONLY`.
9. **Decoupled Booking & Reroute Logs:** Memisahkan kontrak komersial (`orders`) dari eksekusi lapangan (`installation_bookings`) dan melacak riwayat perpindahan bengkel darurat (`booking_schedule_and_reroute_logs`).
10. **Idempotent Webhook Inbox & Outbox Relay:** Menangkal *webhook* ganda dari Midtrans, Midtrans Iris, & Biteship melalui `webhook_events_inbox`, serta menjamin pengiriman job BullMQ/Typesense melalui `outbox_events`.
11. **Normalized Vehicle Plate Indexing:** Menyimpan `plate_number_display` (`"B 1234 XYZ"`) dan `plate_number_normalized` (`"B1234XYZ"`) untuk validasi garansi lintas waktu.

---

## 2. Definisi Tipe Enumerasi Global (`PostgreSQL Enums`)

```sql
-- 1. Peran Pengguna (10 RBAC Roles)
CREATE TYPE user_role_enum AS ENUM (
  'CUSTOMER_GUEST',
  'CUSTOMER_MEMBER',
  'BRAND_OWNER',
  'BRAND_CATALOG_ADMIN',
  'BRAND_ORDER_ADMIN',
  'WORKSHOP_OWNER',
  'WORKSHOP_FRONT_DESK',
  'WORKSHOP_MECHANIC',
  'SUPER_ADMIN',
  'OPS_AND_MEDIATOR',
  'FINANCE_AUDITOR'
);

-- 2. Status Verifikasi KYC Mitra
CREATE TYPE kyc_status_enum AS ENUM (
  'DRAFT',
  'PENDING_KYC_VERIFICATION',
  'VERIFIED_ACTIVE',
  'REJECTED_NEEDS_REVISION',
  'SUSPENDED'
);

-- 3. Status Afiliasi Brand - Bengkel (M:N)
CREATE TYPE affiliation_status_enum AS ENUM (
  'PENDING_BRAND_APPROVAL',
  'PENDING_WORKSHOP_APPROVAL',
  'ACTIVE',
  'SUSPENDED',
  'REVOKED'
);

-- 4. Tipe Kecocokan Produk & Kebijakan Pemenuhan
CREATE TYPE fitment_type_enum AS ENUM ('UNIVERSAL_FIT', 'SPEC_BASED', 'SPECIFIC_FIT');
CREATE TYPE fulfillment_policy_enum AS ENUM ('ALLOW_ALL', 'MANDATORY_WORKSHOP_ONLY');
CREATE TYPE availability_type_enum AS ENUM ('READY_STOCK', 'PRE_ORDER_CUSTOM');
CREATE TYPE warranty_type_enum AS ENUM ('NONE', 'AUTHENTICITY_AND_DOA', 'FUNCTIONAL_PERIODIC');

-- 5. Mode Pemenuhan Transaksi & Status Master Order State Machine
CREATE TYPE fulfillment_mode_enum AS ENUM (
  'MODE_1_DIRECT_HOME',
  'MODE_2_READY_AT_WORKSHOP',
  'MODE_3_SHIP_TO_WORKSHOP'
);

CREATE TYPE order_status_enum AS ENUM (
  'PENDING_PAYMENT',
  'PAYMENT_EXPIRED',
  'PAID_AWAITING_VALIDATION',
  'RESCHEDULE_PROPOSED',
  'PROCESSING_SHIPMENT',
  'IN_TRANSIT_TO_WORKSHOP',
  'IN_TRANSIT_TO_CUSTOMER',
  'INBOUND_DAMAGE_REPORTED',
  'READY_AT_WORKSHOP_FOR_INSTALL',
  'CUSTOMER_CHECKED_IN',
  'INSTALLATION_IN_PROGRESS',
  'ABORTED_BY_VEHICLE_CONDITION',
  'INCIDENT_UNDER_MEDIATION',
  'AWAITING_CUSTOMER_HANDOVER',
  'DELIVERED_TO_CUSTOMER',
  'STORAGE_EXPIRED_FORFEITED',
  'COMPLETED_AND_SETTLED',
  'PARTIALLY_COMPLETED',
  'CANCELLED_FULL_REFUND',
  'CANCELLED_WITH_PENALTY'
);

-- 6. Status Granular Unit Fisik & Jadwal Booking
CREATE TYPE item_unit_status_enum AS ENUM (
  'ALLOCATED',
  'IN_TRANSIT',
  'READY_AT_WORKSHOP',
  'INSTALLED',
  'DELIVERED_NO_INSTALL',
  'DOA_DEFECT_REPORTED',
  'ABORTED_RETURNED_TO_CUSTOMER',
  'FORFEITED_NO_SHOW',
  'CANCELLED_REFUNDED'
);

CREATE TYPE booking_session_enum AS ENUM ('MORNING', 'AFTERNOON', 'EVENING');
CREATE TYPE old_parts_handling_enum AS ENUM ('PENDING', 'RETURNED_TO_TRUNK', 'DISPOSED_WITH_CONSENT');

-- 7. Status Garansi & Klaim Satu Pintu
CREATE TYPE warranty_status_enum AS ENUM (
  'ACTIVE',
  'EXPIRED',
  'VOID_WAIVER_ACCEPTED',
  'VOID_DUE_TO_NO_SHOW',
  'VOID_EXTERNAL_TAMPERING',
  'TRANSFERRED_TO_NEW_OWNER'
);

CREATE TYPE warranty_claim_status_enum AS ENUM (
  'CLAIM_SUBMITTED',
  'INSPECTION_SCHEDULED',
  'UNDER_WORKSHOP_DIAGNOSIS',
  'RESOLVED_BY_WORKSHOP_REWORK',
  'INTERNAL_3WAY_TRIAGE',
  'REPLACEMENT_DISPATCHED',
  'CLAIM_REPLACEMENT_INSTALLED',
  'CLAIM_REJECTED_WITH_PROOF'
);

-- 8. Buku Besar Keuangan (Double-Entry Ledger) & Iris Payout
CREATE TYPE ledger_account_type_enum AS ENUM ('ASSET', 'LIABILITY', 'EQUITY', 'REVENUE', 'EXPENSE');
CREATE TYPE ledger_concurrency_mode_enum AS ENUM ('STRICT_ROW_LOCK', 'ATOMIC_APPEND_ONLY');
CREATE TYPE ledger_entry_direction_enum AS ENUM ('DEBIT', 'CREDIT');
CREATE TYPE payout_status_enum AS ENUM ('QUEUED', 'PROCESSING_IRIS', 'COMPLETED', 'FAILED', 'REJECTED');
```

---

## 3. Spesifikasi DDL Lengkap 45 Tabel Produksi (7 Domain Modular)

### DOMAIN 1: Identity, RBAC & Multi-Tenant Partners (8 Tabel)

```sql
-- Tabel 1: users (Akun Induk Seluruh Pengguna)
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  email VARCHAR(255),
  phone_e164 VARCHAR(20) NOT NULL, -- Format: +62812...
  full_name VARCHAR(150) NOT NULL,
  password_hash TEXT,
  primary_role user_role_enum NOT NULL DEFAULT 'CUSTOMER_MEMBER',
  google_oauth_id VARCHAR(128),
  is_phone_verified BOOLEAN NOT NULL DEFAULT FALSE,
  is_email_verified BOOLEAN NOT NULL DEFAULT FALSE,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  last_login_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ
);
CREATE UNIQUE INDEX idx_users_phone_unique ON users (phone_e164) WHERE deleted_at IS NULL;
CREATE UNIQUE INDEX idx_users_email_unique ON users (LOWER(email)) WHERE email IS NOT NULL AND deleted_at IS NULL;

-- Tabel 2: auth_sessions (Refresh Token Rotation & Signed Magic Links)
CREATE TABLE auth_sessions (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  token_hash VARCHAR(128) NOT NULL UNIQUE,
  session_type VARCHAR(32) NOT NULL DEFAULT 'STANDARD_JWT', -- 'STANDARD_JWT' | 'WA_MAGIC_LINK' | 'MECHANIC_PIN_SESSION'
  scoped_order_id UUID, -- Terisi jika session_type = 'WA_MAGIC_LINK'
  ip_address INET,
  user_agent TEXT,
  expires_at TIMESTAMPTZ NOT NULL,
  revoked_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_auth_sessions_user ON auth_sessions (user_id, expires_at);

-- Tabel 3: brands (Entitas Induk Official Brand Partner)
CREATE TABLE brands (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  owner_user_id UUID NOT NULL REFERENCES users(id),
  slug VARCHAR(120) NOT NULL,
  brand_name VARCHAR(150) NOT NULL,
  legal_company_name VARCHAR(200) NOT NULL,
  nib_number VARCHAR(64),
  npwp_number VARCHAR(64),
  distributor_appointment_doc_url TEXT,
  logo_url TEXT,
  banner_url TEXT,
  kyc_status kyc_status_enum NOT NULL DEFAULT 'PENDING_KYC_VERIFICATION',
  custom_commission_rate NUMERIC(6,4), -- NULL = Gunakan DEFAULT_BRAND_COMMISSION_RATE
  bank_code VARCHAR(32),               -- Kode Bank Midtrans Iris (misal: 'bca', 'mandiri')
  bank_account_number VARCHAR(64),
  bank_account_holder_name VARCHAR(150),
  iris_beneficiary_alias VARCHAR(64),
  rating_avg NUMERIC(3,2) NOT NULL DEFAULT 0.00,
  review_count INTEGER NOT NULL DEFAULT 0,
  onboarded_by_admin_id UUID REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ
);
CREATE UNIQUE INDEX idx_brands_slug_unique ON brands (slug) WHERE deleted_at IS NULL;

-- Tabel 4: brand_warehouses (Multi-Gudang Pengiriman Brand dengan PostGIS)
CREATE TABLE brand_warehouses (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  brand_id UUID NOT NULL REFERENCES brands(id) ON DELETE CASCADE,
  warehouse_name VARCHAR(120) NOT NULL,
  contact_person VARCHAR(120) NOT NULL,
  contact_phone VARCHAR(20) NOT NULL,
  province_name VARCHAR(100) NOT NULL,
  city_name VARCHAR(100) NOT NULL,
  district_name VARCHAR(100) NOT NULL,
  postal_code VARCHAR(10) NOT NULL,
  biteship_area_id VARCHAR(64) NOT NULL,
  full_address TEXT NOT NULL,
  location GEOGRAPHY(Point, 4326) NOT NULL,
  iana_timezone VARCHAR(64) NOT NULL DEFAULT 'Asia/Jakarta',
  is_primary_warehouse BOOLEAN NOT NULL DEFAULT FALSE,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_brand_warehouses_geo ON brand_warehouses USING GIST (location);

-- Tabel 5: brand_staff_members (Sub-Akun RBAC Brand)
CREATE TABLE brand_staff_members (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  brand_id UUID NOT NULL REFERENCES brands(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  assigned_role user_role_enum NOT NULL CHECK (assigned_role IN ('BRAND_OWNER', 'BRAND_CATALOG_ADMIN', 'BRAND_ORDER_ADMIN')),
  assigned_warehouse_id UUID REFERENCES brand_warehouses(id), -- NULL = Akses semua gudang
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (brand_id, user_id)
);

-- Tabel 6: workshops (Entitas Induk Bengkel Rekanan - Pure Operational Partner)
CREATE TABLE workshops (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  owner_user_id UUID NOT NULL REFERENCES users(id),
  slug VARCHAR(120) NOT NULL,
  workshop_group_name VARCHAR(150) NOT NULL,
  legal_entity_name VARCHAR(200),
  nib_number VARCHAR(64),
  npwp_number VARCHAR(64),
  kyc_status kyc_status_enum NOT NULL DEFAULT 'PENDING_KYC_VERIFICATION',
  onboarded_by_admin_id UUID REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ
);
CREATE UNIQUE INDEX idx_workshops_slug_unique ON workshops (slug) WHERE deleted_at IS NULL;

-- Tabel 7: workshop_branches (Titik Cabang Fisik Bengkel dengan PostGIS & Anti-Bypass)
CREATE TABLE workshop_branches (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  workshop_id UUID NOT NULL REFERENCES workshops(id) ON DELETE CASCADE,
  branch_code VARCHAR(32) NOT NULL UNIQUE,
  branch_display_name VARCHAR(150) NOT NULL,
  province_name VARCHAR(100) NOT NULL,
  city_name VARCHAR(100) NOT NULL,
  district_name VARCHAR(100) NOT NULL, -- Ditampilkan sebelum PAID (Anti-Bypass)
  postal_code VARCHAR(10) NOT NULL,
  biteship_area_id VARCHAR(64) NOT NULL,
  full_street_address TEXT NOT NULL,   -- Disembunyikan di API publik sebelum status PAID
  emergency_phone VARCHAR(20) NOT NULL, -- Disembunyikan sebelum status PAID
  location GEOGRAPHY(Point, 4326) NOT NULL,
  iana_timezone VARCHAR(64) NOT NULL DEFAULT 'Asia/Jakarta',
  stall_count INTEGER NOT NULL DEFAULT 2,
  default_morning_capacity INTEGER NOT NULL DEFAULT 3,
  default_afternoon_capacity INTEGER NOT NULL DEFAULT 3,
  default_evening_capacity INTEGER NOT NULL DEFAULT 2,
  facility_photos_jsonb JSONB NOT NULL DEFAULT '[]'::jsonb,
  rating_avg NUMERIC(3,2) NOT NULL DEFAULT 0.00,
  review_count INTEGER NOT NULL DEFAULT 0,
  sla_breach_count INTEGER NOT NULL DEFAULT 0,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_workshop_branches_geo ON workshop_branches USING GIST (location);
CREATE INDEX idx_workshop_branches_city ON workshop_branches (city_name, is_active);

-- Tabel 8: workshop_staff_and_mechanics (Kasir/SA & PIN 4-Digit Mekanik)
CREATE TABLE workshop_staff_and_mechanics (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  workshop_branch_id UUID NOT NULL REFERENCES workshop_branches(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE SET NULL,
  staff_name VARCHAR(120) NOT NULL,
  staff_role user_role_enum NOT NULL CHECK (staff_role IN ('WORKSHOP_OWNER', 'WORKSHOP_FRONT_DESK', 'WORKSHOP_MECHANIC')),
  mechanic_pin_hash VARCHAR(128), -- Hash dari 4-Digit PIN khusus lantai bengkel
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### DOMAIN 2: Affiliation, Master Service Types, Capabilities & Slot Quotas (5 Tabel)

```sql
-- Tabel 9: master_service_types (Standar Jenis Pekerjaan & Cakupan Bahan Bantu All-In)
CREATE TABLE master_service_types (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  service_type_code VARCHAR(64) NOT NULL UNIQUE, -- Contoh: 'SVC_BRAKE_PAD_AXLE', 'SVC_SUSP_COILOVER'
  service_name VARCHAR(150) NOT NULL,
  root_category_code VARCHAR(16) NOT NULL,       -- 'CAT-01' s/d 'CAT-10'
  estimated_duration_minutes INTEGER NOT NULL DEFAULT 60,
  included_consumables_jsonb JSONB NOT NULL DEFAULT '[]'::jsonb, -- Cakupan All-In Gratis (Skenario 8)
  excluded_scope_jsonb JSONB NOT NULL DEFAULT '[]'::jsonb,       -- Di luar paket (misal: Spooring 3D)
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 10: brand_workshop_affiliations (Relasi Many-to-Many Brand & Cabang Bengkel)
CREATE TABLE brand_workshop_affiliations (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  brand_id UUID NOT NULL REFERENCES brands(id) ON DELETE CASCADE,
  workshop_branch_id UUID NOT NULL REFERENCES workshop_branches(id) ON DELETE CASCADE,
  status affiliation_status_enum NOT NULL DEFAULT 'PENDING_WORKSHOP_APPROVAL',
  authorized_category_codes_jsonb JSONB NOT NULL DEFAULT '["*"]'::jsonb,
  initiated_by VARCHAR(16) NOT NULL CHECK (initiated_by IN ('BRAND', 'WORKSHOP', 'ADMIN')),
  approved_at TIMESTAMPTZ,
  suspended_reason TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (brand_id, workshop_branch_id)
);
CREATE INDEX idx_affiliations_lookup ON brand_workshop_affiliations (brand_id, status, workshop_branch_id);

-- Tabel 11: workshop_branch_capabilities (Daftar Keahlian Layanan per Cabang Bengkel - Tanpa Tier Harga)
CREATE TABLE workshop_branch_capabilities (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  workshop_branch_id UUID NOT NULL REFERENCES workshop_branches(id) ON DELETE CASCADE,
  service_type_id UUID NOT NULL REFERENCES master_service_types(id) ON DELETE RESTRICT,
  custom_duration_minutes INTEGER,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (workshop_branch_id, service_type_id)
);
CREATE INDEX idx_workshop_capabilities_lookup ON workshop_branch_capabilities (workshop_branch_id, is_active, service_type_id);

-- Tabel 12: workshop_operating_hours (Jadwal Mingguan & Jam Cut-off Same-Day Cabang)
CREATE TABLE workshop_operating_hours (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  workshop_branch_id UUID NOT NULL REFERENCES workshop_branches(id) ON DELETE CASCADE,
  day_of_week SMALLINT NOT NULL CHECK (day_of_week BETWEEN 0 AND 6), -- 0=Minggu .. 6=Sabtu
  is_closed BOOLEAN NOT NULL DEFAULT FALSE,
  open_time TIME NOT NULL DEFAULT '08:30:00',
  close_time TIME NOT NULL DEFAULT '17:30:00',
  same_day_cutoff_time TIME NOT NULL DEFAULT '13:00:00',
  UNIQUE (workshop_branch_id, day_of_week)
);

-- Tabel 13: workshop_daily_session_quotas (Atomic Slot Counter Anti Double-Booking)
CREATE TABLE workshop_daily_session_quotas (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  workshop_branch_id UUID NOT NULL REFERENCES workshop_branches(id) ON DELETE CASCADE,
  booking_date DATE NOT NULL,
  session_window booking_session_enum NOT NULL,
  max_capacity INTEGER NOT NULL DEFAULT 3,
  booked_count INTEGER NOT NULL DEFAULT 0,
  is_blackout_closed BOOLEAN NOT NULL DEFAULT FALSE,
  CONSTRAINT chk_session_quota_bounds CHECK (booked_count >= 0 AND booked_count <= max_capacity),
  UNIQUE (workshop_branch_id, booking_date, session_window)
);
```

---

### DOMAIN 3: Master Vehicle Taxonomy & User Garage (5 Tabel — Tanpa Complexity Tier)

```sql
-- Tabel 14: vehicle_makes (Merek Kendaraan)
CREATE TABLE vehicle_makes (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  slug VARCHAR(64) NOT NULL UNIQUE,
  make_name VARCHAR(100) NOT NULL UNIQUE, -- Contoh: 'Toyota', 'Honda', 'BMW', 'BYD'
  country_origin VARCHAR(64) NOT NULL,    -- 'JAPAN', 'GERMANY', 'CHINA', 'KOREA', 'USA'
  logo_url TEXT,
  is_popular BOOLEAN NOT NULL DEFAULT FALSE,
  display_order SMALLINT NOT NULL DEFAULT 100
);

-- Tabel 15: vehicle_models (Model Kendaraan Dipisah Kode Sasis untuk Akurasi Fitment)
CREATE TABLE vehicle_models (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  make_id UUID NOT NULL REFERENCES vehicle_makes(id) ON DELETE RESTRICT,
  slug VARCHAR(120) NOT NULL UNIQUE,
  model_display_name VARCHAR(150) NOT NULL, -- Contoh: 'Kijang Innova Reborn (AN140)' vs 'Kijang Innova Zenix (AG10)'
  commercial_group_name VARCHAR(100) NOT NULL, -- Contoh: 'Kijang Innova'
  chassis_code VARCHAR(64),                 -- Contoh: 'AN140', 'AG10', 'GK5', 'FE1', 'G20'
  vehicle_body_type VARCHAR(64) NOT NULL,   -- 'MPV', 'SUV', 'HATCHBACK', 'SEDAN', 'PICKUP_4X4'
  powertrain_type VARCHAR(32) NOT NULL DEFAULT 'ICE', -- 'ICE', 'HEV', 'PHEV', 'BEV'
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_vehicle_models_make ON vehicle_models (make_id, is_active);

-- Tabel 16: vehicle_generations (Rentang Tahun int4range & Spesifikasi OEM)
CREATE TABLE vehicle_generations (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  model_id UUID NOT NULL REFERENCES vehicle_models(id) ON DELETE CASCADE,
  generation_name VARCHAR(150) NOT NULL,      -- Contoh: 'Innova Reborn Facelift 2.4 Diesel (2GD-FTV)'
  production_year_range int4range NOT NULL,   -- Contoh: '[2016,2026]' (Inklusif)
  engine_code VARCHAR(64),                    -- Contoh: '2GD-FTV', 'M20A-FXS', 'L15B'
  oem_technical_specs_jsonb JSONB NOT NULL DEFAULT '{}'::jsonb, -- PCD, bore, tire_oem, oil_viscosity, head_bulb_socket
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_vehicle_generations_range ON vehicle_generations USING GIST (model_id, production_year_range);

-- Tabel 17: crowdsourced_vehicle_submissions (Antrean Kurasi Mobil Langka Input Manual)
CREATE TABLE crowdsourced_vehicle_submissions (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  submitted_by_user_id UUID NOT NULL REFERENCES users(id),
  source_order_id UUID,
  raw_make_input VARCHAR(100) NOT NULL,
  raw_model_input VARCHAR(150) NOT NULL,
  raw_year_input SMALLINT NOT NULL,
  raw_variant_notes TEXT,
  review_status VARCHAR(32) NOT NULL DEFAULT 'PENDING_MASTER_VEHICLE_REVIEW', -- 'PENDING...' | 'MERGED' | 'PROMOTED'
  resolved_model_id UUID REFERENCES vehicle_models(id),
  reviewed_by_admin_id UUID REFERENCES users(id),
  reviewed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 18: user_garage_vehicles (Garasi Digital Konsumen)
CREATE TABLE user_garage_vehicles (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  vehicle_model_id UUID REFERENCES vehicle_models(id),
  vehicle_generation_id UUID REFERENCES vehicle_generations(id),
  manufacturing_year SMALLINT NOT NULL,
  plate_number_display VARCHAR(24) NOT NULL,    -- Contoh: 'B 1234 XYZ'
  plate_number_normalized VARCHAR(20) NOT NULL, -- Contoh: 'B1234XYZ' (Tanpa spasi, huruf kapital)
  vin_number VARCHAR(32),
  variant_transmission_notes VARCHAR(150),
  is_primary_vehicle BOOLEAN NOT NULL DEFAULT FALSE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_user_garage_plate ON user_garage_vehicles (plate_number_normalized);
```

---

### DOMAIN 4: 3-Level Dynamic Catalog, Fitment & Inventory (6 Tabel)

```sql
-- Tabel 19: categories (3-Level Dynamic Tree + Dual-Label Taxonomy + JSONB Schema)
CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  parent_id UUID REFERENCES categories(id) ON DELETE RESTRICT,
  category_code VARCHAR(32) NOT NULL UNIQUE,   -- 'CAT-01', 'CAT-01-01', 'CAT-01-01-01'
  level SMALLINT NOT NULL CHECK (level BETWEEN 1 AND 3),
  slug VARCHAR(120) NOT NULL UNIQUE,
  ui_short_label VARCHAR(16) NOT NULL,         -- Maks 14-16 kar untuk Mobile Grid ('Kaki-Kaki', 'Pengereman')
  full_display_name VARCHAR(150) NOT NULL,     -- Nama lengkap SEO ('Suspensi, Kaki-Kaki & Kemudi')
  subtitle_keywords VARCHAR(255),
  icon_name VARCHAR(64),
  default_fulfillment_policy fulfillment_policy_enum NOT NULL DEFAULT 'ALLOW_ALL',
  default_warranty_type warranty_type_enum NOT NULL DEFAULT 'FUNCTIONAL_PERIODIC',
  default_service_type_id UUID REFERENCES master_service_types(id),
  attribute_schema_template_jsonb JSONB NOT NULL DEFAULT '[]'::jsonb,
  display_order SMALLINT NOT NULL DEFAULT 100,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 20: products (Master SKU Official Brand — Harga All-In Sudah Termasuk Jasa Pasang)
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  brand_id UUID NOT NULL REFERENCES brands(id) ON DELETE RESTRICT,
  category_id UUID NOT NULL REFERENCES categories(id) ON DELETE RESTRICT,
  service_type_id UUID REFERENCES master_service_types(id),
  sku_code VARCHAR(100) NOT NULL,
  oem_part_number VARCHAR(100),
  slug VARCHAR(200) NOT NULL,
  product_name VARCHAR(255) NOT NULL,
  description_markdown TEXT NOT NULL,
  fitment_type fitment_type_enum NOT NULL DEFAULT 'SPECIFIC_FIT',
  fulfillment_policy fulfillment_policy_enum NOT NULL DEFAULT 'ALLOW_ALL',
  availability_type availability_type_enum NOT NULL DEFAULT 'READY_STOCK',
  production_lead_time_days SMALLINT NOT NULL DEFAULT 0 CHECK (production_lead_time_days BETWEEN 0 AND 30),
  catalog_price_idr BIGINT NOT NULL CHECK (catalog_price_idr > 0), -- Harga All-In (Termasuk Gratis Pasang)
  brand_discount_price_idr BIGINT CHECK (brand_discount_price_idr IS NULL OR brand_discount_price_idr < catalog_price_idr),
  -- Dimensi Setelah Dikemas (Packed Dimensions - Skenario 13)
  packed_weight_grams INTEGER NOT NULL CHECK (packed_weight_grams > 0),
  packed_length_cm NUMERIC(6,2) NOT NULL CHECK (packed_length_cm > 0),
  packed_width_cm NUMERIC(6,2) NOT NULL CHECK (packed_width_cm > 0),
  packed_height_cm NUMERIC(6,2) NOT NULL CHECK (packed_height_cm > 0),
  -- Konfigurasi Garansi Resmi Produk
  warranty_type warranty_type_enum NOT NULL DEFAULT 'FUNCTIONAL_PERIODIC',
  warranty_duration_months SMALLINT NOT NULL DEFAULT 12 CHECK (warranty_duration_months >= 0),
  warranty_terms_summary TEXT,
  -- Spesifikasi Teknis Dinamis sesuai Kategori
  technical_attributes_jsonb JSONB NOT NULL DEFAULT '{}'::jsonb,
  rating_avg NUMERIC(3,2) NOT NULL DEFAULT 0.00,
  review_count INTEGER NOT NULL DEFAULT 0,
  sold_count INTEGER NOT NULL DEFAULT 0,
  is_active BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  deleted_at TIMESTAMPTZ
);
CREATE UNIQUE INDEX idx_products_brand_sku_unique ON products (brand_id, sku_code) WHERE deleted_at IS NULL;
CREATE UNIQUE INDEX idx_products_slug_unique ON products (slug) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_category_active ON products (category_id, is_active) WHERE deleted_at IS NULL;
CREATE INDEX idx_products_tech_attrs_gin ON products USING GIN (technical_attributes_jsonb);

-- Tabel 21: product_media (Foto & Video Produk)
CREATE TABLE product_media (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  media_type VARCHAR(16) NOT NULL DEFAULT 'IMAGE',
  r2_object_key TEXT NOT NULL,
  public_cdn_url TEXT NOT NULL,
  display_order SMALLINT NOT NULL DEFAULT 0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 22: product_vehicle_fitments (Matriks Kecocokan int4range + GiST Index)
CREATE TABLE product_vehicle_fitments (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  vehicle_model_id UUID NOT NULL REFERENCES vehicle_models(id) ON DELETE CASCADE,
  vehicle_generation_id UUID REFERENCES vehicle_generations(id) ON DELETE SET NULL,
  production_year_range int4range NOT NULL,
  fitment_position_notes VARCHAR(120),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_product_fitments_gist ON product_vehicle_fitments USING GIST (vehicle_model_id, production_year_range);
CREATE INDEX idx_product_fitments_product ON product_vehicle_fitments (product_id);

-- Tabel 23: brand_warehouse_inventories (Stok Gudang Brand dengan Atomic Constraint)
CREATE TABLE brand_warehouse_inventories (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  warehouse_id UUID NOT NULL REFERENCES brand_warehouses(id) ON DELETE CASCADE,
  stock_on_hand INTEGER NOT NULL DEFAULT 0,
  stock_reserved INTEGER NOT NULL DEFAULT 0,
  low_stock_threshold INTEGER NOT NULL DEFAULT 3,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  CONSTRAINT chk_brand_stock_non_negative CHECK (stock_on_hand >= 0 AND stock_reserved >= 0 AND (stock_on_hand - stock_reserved) >= 0),
  UNIQUE (product_id, warehouse_id)
);

-- Tabel 24: workshop_branch_inventories (Stok Lokal di Bengkel untuk Mode 2 Instant Booking)
CREATE TABLE workshop_branch_inventories (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  workshop_branch_id UUID NOT NULL REFERENCES workshop_branches(id) ON DELETE CASCADE,
  stock_on_hand INTEGER NOT NULL DEFAULT 0,
  stock_reserved INTEGER NOT NULL DEFAULT 0,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  CONSTRAINT chk_workshop_stock_non_negative CHECK (stock_on_hand >= 0 AND stock_reserved >= 0 AND (stock_on_hand - stock_reserved) >= 0),
  UNIQUE (product_id, workshop_branch_id)
);
```

---

### DOMAIN 5: Carts, Orders, Unit Serialization, O2O Bookings & QC Logs (9 Tabel)

```sql
-- Tabel 25 & 26: carts & cart_items
CREATE TABLE carts (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
  selected_garage_vehicle_id UUID REFERENCES user_garage_vehicles(id) ON DELETE SET NULL,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE cart_items (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  cart_id UUID NOT NULL REFERENCES carts(id) ON DELETE CASCADE,
  brand_id UUID NOT NULL REFERENCES brands(id) ON DELETE CASCADE,
  product_id UUID NOT NULL REFERENCES products(id) ON DELETE CASCADE,
  quantity SMALLINT NOT NULL DEFAULT 1 CHECK (quantity > 0),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (cart_id, product_id)
);

-- Tabel 27: orders (Kontrak Komersial Induk — All-In Pricing & Brand-Borne Mode 3 Shipping)
CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  order_number VARCHAR(32) NOT NULL UNIQUE, -- Contoh: 'NTKK-20261020-99412'
  customer_user_id UUID NOT NULL REFERENCES users(id),
  brand_id UUID NOT NULL REFERENCES brands(id),
  origin_warehouse_id UUID REFERENCES brand_warehouses(id),
  workshop_branch_id UUID REFERENCES workshop_branches(id), -- NULL jika Mode 1 (Direct Home)
  fulfillment_mode fulfillment_mode_enum NOT NULL,
  status order_status_enum NOT NULL DEFAULT 'PENDING_PAYMENT',
  -- Deklarasi Kendaraan Saat Checkout (Murni untuk Fitment & Garansi)
  declared_vehicle_model_id UUID REFERENCES vehicle_models(id),
  declared_vehicle_make_name VARCHAR(100),
  declared_vehicle_model_name VARCHAR(150),
  declared_vehicle_year SMALLINT,
  declared_vehicle_variant_notes VARCHAR(150),
  plate_number_display VARCHAR(24),
  plate_number_normalized VARCHAR(20),
  -- Validasi Fitment oleh Official Brand (SLA 4 Jam Kerja)
  brand_fitment_status VARCHAR(32) NOT NULL DEFAULT 'PENDING', -- 'PENDING' | 'APPROVED' | 'MISMATCH_FLAGGED' | 'REJECTED'
  brand_fitment_checked_by UUID REFERENCES users(id),
  brand_fitment_checked_at TIMESTAMPTZ,
  brand_fitment_sla_deadline_at TIMESTAMPTZ,
  -- Audit Persetujuan Garansi Hangus (Khusus Mode 1 - Kirim ke Rumah)
  warranty_waiver_accepted BOOLEAN NOT NULL DEFAULT FALSE,
  warranty_waiver_accepted_at TIMESTAMPTZ,
  warranty_waiver_text_snapshot TEXT,
  customer_checkout_ip INET,
  -- Snapshot Keuangan All-In (Seluruhnya BIGINT Rupiah Murni sesuai Rumus Bab 5.2 Dokumen 1)
  subtotal_products_gross_idr BIGINT NOT NULL,               -- P_gross (Sudah termasuk fasilitas gratis pasang)
  brand_voucher_discount_idr BIGINT NOT NULL DEFAULT 0,      -- D_brand
  subtotal_products_net_idr BIGINT NOT NULL,                 -- P_net = P_gross - D_brand
  customer_paid_shipping_fee_idr BIGINT NOT NULL DEFAULT 0,  -- L_cust (Wajib 0 pada Mode 2 & Mode 3; > 0 hanya Mode 1)
  actual_courier_shipping_cost_idr BIGINT NOT NULL DEFAULT 0,-- Tagihan aktual kurir dari Biteship
  brand_borne_shipping_fee_idr BIGINT NOT NULL DEFAULT 0,    -- L_brand_ship (100% ongkir Mode 3 ke bengkel + undercharge Mode 1)
  platform_service_fee_idr BIGINT NOT NULL DEFAULT 5000,     -- F_platform
  platform_promo_discount_idr BIGINT NOT NULL DEFAULT 0,     -- D_platform
  wallet_balance_used_idr BIGINT NOT NULL DEFAULT 0,
  grand_total_customer_pay_idr BIGINT NOT NULL,              -- T_customer_pay = P_net + L_cust + F_platform - D_platform
  -- Snapshot Komisi & Net Settlement Final
  brand_commission_amount_idr BIGINT NOT NULL DEFAULT 0,     -- C_brand = round(P_net * R_brand)
  net_brand_settlement_idr BIGINT NOT NULL DEFAULT 0,        -- N_brand = P_net - C_brand - L_brand_ship
  net_platform_revenue_idr BIGINT NOT NULL DEFAULT 0,        -- N_platform = C_brand + F_platform - D_platform
  -- Integrasi Midtrans Pay-In
  midtrans_snap_token VARCHAR(128),
  midtrans_transaction_id VARCHAR(128),
  midtrans_payment_type VARCHAR(64),
  payment_expires_at TIMESTAMPTZ NOT NULL,
  paid_at TIMESTAMPTZ,
  completed_at TIMESTAMPTZ,
  cancelled_at TIMESTAMPTZ,
  cancellation_reason_code VARCHAR(64),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  -- Constraint Kernel: Mode 2 & Mode 3 tidak boleh menagih ongkir ke konsumen!
  CONSTRAINT chk_free_workshop_shipping_for_customer CHECK (
    fulfillment_mode = 'MODE_1_DIRECT_HOME' OR customer_paid_shipping_fee_idr = 0
  )
);
CREATE INDEX idx_orders_customer ON orders (customer_user_id, created_at DESC);
CREATE INDEX idx_orders_brand_status ON orders (brand_id, status, created_at DESC);
CREATE INDEX idx_orders_workshop_status ON orders (workshop_branch_id, status, created_at DESC);

-- Tabel 28: order_items (Snapshot Baris SKU Harga All-In)
CREATE TABLE order_items (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id UUID NOT NULL REFERENCES products(id),
  service_type_id UUID REFERENCES master_service_types(id),
  sku_code_snapshot VARCHAR(100) NOT NULL,
  product_name_snapshot VARCHAR(255) NOT NULL,
  quantity SMALLINT NOT NULL CHECK (quantity > 0),
  unit_catalog_price_snapshot_idr BIGINT NOT NULL,
  unit_net_price_snapshot_idr BIGINT NOT NULL,
  brand_commission_rate_snapshot NUMERIC(6,4) NOT NULL,
  packed_dimensions_snapshot_jsonb JSONB NOT NULL,
  warranty_type_snapshot warranty_type_enum NOT NULL,
  warranty_duration_months_snapshot SMALLINT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_order_items_order ON order_items (order_id);

-- Tabel 29: order_item_units (Normalisasi Unit Fisik untuk Serial Number, DOA & Klaim per Unit)
CREATE TABLE order_item_units (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  order_item_id UUID NOT NULL REFERENCES order_items(id) ON DELETE CASCADE,
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  unit_sequence_number SMALLINT NOT NULL, -- 1, 2, 3, 4 (misal untuk 4 ban)
  serial_number_scanned VARCHAR(128),
  serial_number_photo_url TEXT,
  unit_status item_unit_status_enum NOT NULL DEFAULT 'ALLOCATED',
  doa_defect_video_url TEXT,
  doa_defect_notes TEXT,
  refunded_unit_amount_idr BIGINT NOT NULL DEFAULT 0,
  installed_at TIMESTAMPTZ,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (order_item_id, unit_sequence_number)
);
CREATE INDEX idx_order_item_units_serial ON order_item_units (serial_number_scanned) WHERE serial_number_scanned IS NOT NULL;

-- Tabel 30: order_shipments (Pengiriman Kurir Biteship, Volumetrik & Inbound Verification)
CREATE TABLE order_shipments (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  shipment_direction VARCHAR(32) NOT NULL DEFAULT 'OUTBOUND_INITIAL', -- 'OUTBOUND_INITIAL' | 'DOA_REPLACEMENT' | 'RETURN_TO_BRAND'
  shipping_cost_bearer VARCHAR(16) NOT NULL DEFAULT 'BRAND',          -- 'BRAND' (Mode 3 & DOA) | 'CUSTOMER' (Mode 1)
  biteship_order_id VARCHAR(128),
  courier_company VARCHAR(64) NOT NULL,
  courier_service_type VARCHAR(64) NOT NULL,
  waybill_number VARCHAR(128),
  declared_chargeable_weight_kg NUMERIC(8,2) NOT NULL,
  actual_courier_measured_weight_kg NUMERIC(8,2),
  estimated_shipping_cost_idr BIGINT NOT NULL,
  actual_billed_shipping_cost_idr BIGINT,
  -- Bukti Penerimaan Paket di Bengkel (SOP Inbound Check - Skenario 2)
  inbound_received_by_staff_id UUID REFERENCES workshop_staff_and_mechanics(id),
  inbound_package_condition VARCHAR(32), -- 'SEALED_GOOD' | 'DAMAGED_REJECTED'
  inbound_seal_photo_url TEXT,
  inbound_received_at TIMESTAMPTZ,
  shipped_at TIMESTAMPTZ,
  delivered_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 31: installation_bookings (Eksekusi Operasional O2O, Driver Pass & 14-Day SLA)
CREATE TABLE installation_bookings (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  order_id UUID NOT NULL UNIQUE REFERENCES orders(id) ON DELETE CASCADE,
  booking_code VARCHAR(24) NOT NULL UNIQUE, -- Contoh: 'BKG-882910'
  workshop_branch_id UUID NOT NULL REFERENCES workshop_branches(id),
  scheduled_date DATE NOT NULL,
  scheduled_session booking_session_enum NOT NULL,
  workshop_schedule_confirmed BOOLEAN NOT NULL DEFAULT FALSE,
  workshop_confirmed_by_staff_id UUID REFERENCES workshop_staff_and_mechanics(id),
  workshop_confirmed_at TIMESTAMPTZ,
  workshop_confirm_sla_deadline_at TIMESTAMPTZ,
  -- Keamanan Check-in, Driver Pass & Remote Handover (Skenario 10)
  checkin_qr_token_hash VARCHAR(128) NOT NULL UNIQUE,
  checkin_6digit_pin VARCHAR(6) NOT NULL,
  completion_6digit_pin VARCHAR(6) NOT NULL,
  driver_share_pass_token VARCHAR(128) NOT NULL UNIQUE,
  -- Timer Penyimpanan 14 Hari & Kuota Reschedule (Skenario 3)
  customer_reschedule_count SMALLINT NOT NULL DEFAULT 0 CHECK (customer_reschedule_count BETWEEN 0 AND 2),
  package_arrived_at_workshop_at TIMESTAMPTZ,
  storage_14day_expires_at TIMESTAMPTZ,
  -- Eksekusi Pengerjaan di Bengkel
  assigned_mechanic_id UUID REFERENCES workshop_staff_and_mechanics(id),
  checked_in_at TIMESTAMPTZ,
  installation_started_at TIMESTAMPTZ,
  qc_submitted_at TIMESTAMPTZ,
  handover_auto_complete_at TIMESTAMPTZ, -- qc_submitted_at + 48 hours
  old_parts_handling old_parts_handling_enum NOT NULL DEFAULT 'PENDING',
  old_parts_trunk_photo_url TEXT,
  mechanic_technical_notes TEXT,
  flagged_gallery_upload BOOLEAN NOT NULL DEFAULT FALSE,
  aborted_vehicle_condition_photos_jsonb JSONB DEFAULT '[]'::jsonb,
  customer_approved_abort_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_bookings_branch_date ON installation_bookings (workshop_branch_id, scheduled_date, scheduled_session);
CREATE INDEX idx_bookings_storage_expiry ON installation_bookings (storage_14day_expires_at) WHERE checked_in_at IS NULL;

-- Tabel 32: booking_qc_and_handover_logs (Bukti Foto Kondisi Awal & Akhir Ber-Watermark)
CREATE TABLE booking_qc_and_handover_logs (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  booking_id UUID NOT NULL REFERENCES installation_bookings(id) ON DELETE CASCADE,
  uploaded_by_staff_id UUID NOT NULL REFERENCES workshop_staff_and_mechanics(id),
  stage_type VARCHAR(32) NOT NULL CHECK (stage_type IN ('PRE_INSTALL_CONDITION', 'POST_INSTALL_QC', 'INCIDENT_EVIDENCE')),
  photo_category VARCHAR(64) NOT NULL,
  r2_object_key TEXT NOT NULL,
  watermarked_cdn_url TEXT NOT NULL,
  capture_source VARCHAR(16) NOT NULL DEFAULT 'LIVE_CAMERA',
  watermark_metadata_jsonb JSONB NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 33 & 34: booking_schedule_and_reroute_logs & order_state_transition_logs
CREATE TABLE booking_schedule_and_reroute_logs (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  booking_id UUID NOT NULL REFERENCES installation_bookings(id) ON DELETE CASCADE,
  change_type VARCHAR(32) NOT NULL, -- 'CUSTOMER_RESCHEDULE' | 'WORKSHOP_PROPOSAL' | 'EMERGENCY_WORKSHOP_REROUTE'
  previous_branch_id UUID REFERENCES workshop_branches(id),
  new_branch_id UUID REFERENCES workshop_branches(id),
  previous_date DATE,
  new_date DATE,
  previous_session booking_session_enum,
  new_session booking_session_enum,
  initiated_by_user_id UUID REFERENCES users(id),
  reason_notes TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE order_state_transition_logs (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  order_id UUID NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  from_status order_status_enum,
  to_status order_status_enum NOT NULL,
  triggered_by_user_id UUID REFERENCES users(id),
  triggered_by_system_actor VARCHAR(64),
  reason_code VARCHAR(64),
  metadata_jsonb JSONB DEFAULT '{}'::jsonb,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_order_state_logs_order ON order_state_transition_logs (order_id, created_at);
```

---

### DOMAIN 6: Digital Warranty, 3-Way Claims & Dual-Reviews (5 Tabel)

```sql
-- Tabel 35: digital_warranties (Digital Warranty Passport per Unit Fisik)
CREATE TABLE digital_warranties (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  certificate_number VARCHAR(32) NOT NULL UNIQUE,
  order_id UUID NOT NULL REFERENCES orders(id),
  order_item_unit_id UUID NOT NULL UNIQUE REFERENCES order_item_units(id),
  current_owner_user_id UUID NOT NULL REFERENCES users(id),
  brand_id UUID NOT NULL REFERENCES brands(id),
  installing_workshop_branch_id UUID NOT NULL REFERENCES workshop_branches(id),
  product_id UUID NOT NULL REFERENCES products(id),
  serial_number VARCHAR(128) NOT NULL,
  plate_number_normalized VARCHAR(20) NOT NULL,
  plate_number_display VARCHAR(24) NOT NULL,
  vehicle_snapshot_name VARCHAR(200) NOT NULL,
  warranty_type warranty_type_enum NOT NULL,
  status warranty_status_enum NOT NULL DEFAULT 'ACTIVE',
  activated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  installation_warranty_expires_at TIMESTAMPTZ NOT NULL, -- activated_at + 30 days
  product_warranty_expires_at TIMESTAMPTZ NOT NULL,      -- activated_at + N months
  ownership_transfer_count SMALLINT NOT NULL DEFAULT 0 CHECK (ownership_transfer_count BETWEEN 0 AND 1),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_warranties_owner ON digital_warranties (current_owner_user_id, status);
CREATE INDEX idx_warranties_plate_serial ON digital_warranties (plate_number_normalized, serial_number);

-- Tabel 36: warranty_ownership_transfers (Pindah Tangan Garansi Saat Mobil Dijual)
CREATE TABLE warranty_ownership_transfers (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  warranty_id UUID NOT NULL UNIQUE REFERENCES digital_warranties(id) ON DELETE CASCADE,
  previous_owner_user_id UUID NOT NULL REFERENCES users(id),
  new_owner_email VARCHAR(255) NOT NULL,
  new_owner_phone_e164 VARCHAR(20) NOT NULL,
  new_owner_user_id UUID REFERENCES users(id),
  claim_invitation_token_hash VARCHAR(128) NOT NULL UNIQUE,
  transfer_status VARCHAR(32) NOT NULL DEFAULT 'PENDING_NEW_OWNER_CLAIM',
  transferred_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 37: warranty_claims (Tiket Klaim Garansi Satu Pintu & Mediasi 3 Arah)
CREATE TABLE warranty_claims (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  claim_number VARCHAR(32) NOT NULL UNIQUE,
  warranty_id UUID NOT NULL REFERENCES digital_warranties(id),
  claimant_user_id UUID NOT NULL REFERENCES users(id),
  inspecting_workshop_branch_id UUID NOT NULL REFERENCES workshop_branches(id),
  brand_id UUID NOT NULL REFERENCES brands(id),
  status warranty_claim_status_enum NOT NULL DEFAULT 'CLAIM_SUBMITTED',
  customer_symptom_description TEXT NOT NULL,
  customer_evidence_media_jsonb JSONB NOT NULL DEFAULT '[]'::jsonb,
  inspection_scheduled_date DATE,
  inspection_session booking_session_enum,
  workshop_diagnosis_category VARCHAR(32),
  workshop_diagnosis_notes TEXT,
  workshop_diagnosis_media_jsonb JSONB DEFAULT '[]'::jsonb,
  diagnosed_at TIMESTAMPTZ,
  final_liability_party VARCHAR(16), -- 'BRAND' | 'WORKSHOP' | 'CUSTOMER_REJECTED'
  resolved_by_mediator_admin_id UUID REFERENCES users(id),
  resolution_notes TEXT,
  replacement_shipment_id UUID REFERENCES order_shipments(id),
  resolved_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 38: warranty_claim_messages (Ruang Diskusi Privat 3 Arah vs Pesan ke Konsumen)
CREATE TABLE warranty_claim_messages (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  claim_id UUID NOT NULL REFERENCES warranty_claims(id) ON DELETE CASCADE,
  sender_user_id UUID NOT NULL REFERENCES users(id),
  sender_role user_role_enum NOT NULL,
  visibility_scope VARCHAR(24) NOT NULL DEFAULT 'INTERNAL_3WAY_ONLY',
  message_body TEXT NOT NULL,
  attachment_urls_jsonb JSONB DEFAULT '[]'::jsonb,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 39: dual_reviews (Penilaian Terpisah: Produk Brand vs Layanan Bengkel)
CREATE TABLE dual_reviews (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  order_id UUID NOT NULL UNIQUE REFERENCES orders(id) ON DELETE CASCADE,
  customer_user_id UUID NOT NULL REFERENCES users(id),
  brand_id UUID NOT NULL REFERENCES brands(id),
  workshop_branch_id UUID REFERENCES workshop_branches(id),
  product_rating SMALLINT NOT NULL CHECK (product_rating BETWEEN 1 AND 5),
  product_review_text TEXT,
  product_review_photos_jsonb JSONB DEFAULT '[]'::jsonb,
  workshop_rating SMALLINT CHECK (workshop_rating IS NULL OR (workshop_rating BETWEEN 1 AND 5)),
  workshop_quick_tags_jsonb JSONB DEFAULT '[]'::jsonb,
  workshop_review_text TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### DOMAIN 7: Double-Entry Escrow Ledger, Iris Payouts, Webhooks, Outbox & Configs (6 Tabel)

```sql
-- Tabel 40: ledger_accounts (Bagan Akun Buku Besar dengan Hybrid Concurrency Policy)
CREATE TABLE ledger_accounts (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  account_code VARCHAR(100) NOT NULL UNIQUE, -- Contoh: 'SYS_ESCROW_HOLDING', 'BRAND_AVL_0192...', 'CUST_REFUND_AVL_0192...'
  account_name VARCHAR(150) NOT NULL,
  account_type ledger_account_type_enum NOT NULL,
  owner_entity_type VARCHAR(24) NOT NULL,    -- 'SYSTEM' | 'BRAND' | 'CUSTOMER'
  owner_entity_id UUID,
  wallet_bucket VARCHAR(32) NOT NULL,        -- 'ESCROW_HOLDING' | 'AVAILABLE_BALANCE' | 'PAYOUT_IN_FLIGHT' | 'REFUND_WALLET' | 'REVENUE'
  concurrency_mode ledger_concurrency_mode_enum NOT NULL DEFAULT 'STRICT_ROW_LOCK',
  current_balance_idr BIGINT NOT NULL DEFAULT 0,
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  CONSTRAINT chk_wallet_non_negative CHECK (
    owner_entity_type = 'SYSTEM' OR current_balance_idr >= 0
  )
);
CREATE INDEX idx_ledger_accounts_owner ON ledger_accounts (owner_entity_type, owner_entity_id);

-- Tabel 41: ledger_journals (Header Transaksi Akuntansi Berpasangan - Immutable)
CREATE TABLE ledger_journals (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  journal_number VARCHAR(40) NOT NULL UNIQUE,
  idempotency_key VARCHAR(128) NOT NULL UNIQUE,
  journal_event_type VARCHAR(64) NOT NULL, -- 'ORDER_PAID_ESCROW', 'ORDER_SETTLED_ALL_IN', 'IRIS_PAYOUT'
  reference_order_id UUID REFERENCES orders(id),
  reference_payout_id UUID,
  description TEXT NOT NULL,
  total_debit_idr BIGINT NOT NULL CHECK (total_debit_idr > 0),
  total_credit_idr BIGINT NOT NULL CHECK (total_credit_idr > 0),
  CONSTRAINT chk_journal_zero_sum_balance CHECK (total_debit_idr = total_credit_idr),
  created_by_user_id UUID REFERENCES users(id),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 42: ledger_entries (Baris Mutasi Debit & Kredit - Append-Only)
CREATE TABLE ledger_entries (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  journal_id UUID NOT NULL REFERENCES ledger_journals(id) ON DELETE RESTRICT,
  account_id UUID NOT NULL REFERENCES ledger_accounts(id) ON DELETE RESTRICT,
  direction ledger_entry_direction_enum NOT NULL,
  amount_idr BIGINT NOT NULL CHECK (amount_idr > 0),
  running_balance_snapshot_idr BIGINT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_ledger_entries_account_time ON ledger_entries (account_id, created_at DESC);
CREATE INDEX idx_ledger_entries_journal ON ledger_entries (journal_id);

-- Tabel 43: payout_and_refund_disbursements (Pencairan Dana Brand & Refund VA via Midtrans Iris API)
CREATE TABLE payout_and_refund_disbursements (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  disbursement_number VARCHAR(32) NOT NULL UNIQUE,
  idempotency_key VARCHAR(128) NOT NULL UNIQUE,
  source_ledger_account_id UUID NOT NULL REFERENCES ledger_accounts(id),
  recipient_type VARCHAR(16) NOT NULL CHECK (recipient_type IN ('BRAND', 'CUSTOMER_REFUND')),
  recipient_entity_id UUID NOT NULL,
  requested_by_user_id UUID NOT NULL REFERENCES users(id),
  approved_by_user_id UUID REFERENCES users(id),
  gross_amount_idr BIGINT NOT NULL CHECK (gross_amount_idr >= 10000),
  bank_transfer_fee_idr BIGINT NOT NULL DEFAULT 0,
  net_transfer_amount_idr BIGINT NOT NULL CHECK (net_transfer_amount_idr > 0),
  bank_code VARCHAR(32) NOT NULL,
  bank_account_number VARCHAR(64) NOT NULL,
  bank_account_holder_name VARCHAR(150) NOT NULL,
  midtrans_iris_reference_no VARCHAR(128) UNIQUE,
  status payout_status_enum NOT NULL DEFAULT 'QUEUED',
  failure_reason TEXT,
  completed_at TIMESTAMPTZ,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tabel 44 & 45: webhook_events_inbox, outbox_events, platform_system_configs & national_holidays
CREATE TABLE webhook_events_inbox (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  provider_name VARCHAR(32) NOT NULL,
  external_event_id VARCHAR(128) NOT NULL,
  event_type VARCHAR(64) NOT NULL,
  raw_payload_jsonb JSONB NOT NULL,
  signature_verified BOOLEAN NOT NULL DEFAULT TRUE,
  processing_status VARCHAR(24) NOT NULL DEFAULT 'PROCESSED',
  error_log TEXT,
  processed_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE (provider_name, external_event_id)
);

CREATE TABLE outbox_events (
  id UUID PRIMARY KEY DEFAULT uuidv7(),
  aggregate_type VARCHAR(64) NOT NULL,
  aggregate_id UUID NOT NULL,
  event_name VARCHAR(100) NOT NULL,
  payload_jsonb JSONB NOT NULL,
  status VARCHAR(24) NOT NULL DEFAULT 'PENDING',
  retry_count SMALLINT NOT NULL DEFAULT 0,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  published_at TIMESTAMPTZ
);
CREATE INDEX idx_outbox_pending ON outbox_events (created_at ASC) WHERE status = 'PENDING';

CREATE TABLE platform_system_configs (
  config_key VARCHAR(64) PRIMARY KEY,
  config_value TEXT NOT NULL,
  value_type VARCHAR(16) NOT NULL CHECK (value_type IN ('INTEGER', 'DECIMAL', 'BIGINT', 'BOOLEAN', 'JSON')),
  description TEXT NOT NULL,
  updated_by_admin_id UUID REFERENCES users(id),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE national_holidays (
  holiday_date DATE PRIMARY KEY,
  holiday_name VARCHAR(150) NOT NULL,
  is_national_day_off BOOLEAN NOT NULL DEFAULT TRUE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## 4. Matriks Jurnal Akuntansi Berpasangan (`v2.1.0-FINAL` — All-In & Mode 3 Shipping Borne by Brand)

### 4.1. Saat Pembayaran Konsumen Lunas pada Mode 2 atau Mode 3 (`ORDER_PAID_ESCROW`)
Misal: Harga Produk Bersih All-In ($P_{\text{net}}$) = **Rp 4.000.000**, Jasa Pasang = **Rp 0**, Ongkir ke Bengkel bagi Konsumen ($L_{\text{cust}}$) = **Rp 0**, Biaya Layanan Aplikasi ($F_{\text{platform}}$) = **Rp 5.000**. Total dibayar konsumen ($T_{\text{customer\_pay}}$) = **Rp 4.005.000**.

| Akun Buku Besar (`account_code`) | Tipe Akun | Arah (`direction`) | Nominal (`amount_idr`) | Keterangan |
| :--- | :--- | :---: | ---: | :--- |
| `SYS_MIDTRANS_CLEARING` | `ASSET` | **DEBIT** | `4005000` | Kas masuk di rekening penampungan Midtrans NewTKK |
| `SYS_ESCROW_HOLDING` | `LIABILITY` | **CREDIT** | `4005000` | Kewajiban dana tertahan di Escrow atas pesanan `order_id` |

### 4.2. Saat Pesanan Mode 3 Selesai Normal (`COMPLETED_AND_SETTLED`)
Misal: Komisi Brand $R_{\text{brand}} = 10\%$ ($C_{\text{brand}} = \mathbf{400.000}$), dan biaya pengiriman kurir Biteship dari Gudang Brand ke Bengkel Rekanan yang ditanggung oleh Brand ($L_{\text{brand\_ship}}$) adalah **Rp 80.000**:
* Hak Bersih Official Brand ($N_{\text{brand}}$) = $4.000.000 - 400.000 - 80.000 = \mathbf{3.520.000}$.
* Kewajiban Bayar Logistik Biteship ($L_{\text{brand\_ship}}$) = $\mathbf{80.000}$.
* Pendapatan Platform NewTKK ($N_{\text{platform}}$) = $400.000 + 5.000 = \mathbf{405.000}$.

| Akun Buku Besar (`account_code`) | Tipe Akun | Arah (`direction`) | Nominal (`amount_idr`) | Keterangan |
| :--- | :--- | :---: | ---: | :--- |
| `SYS_ESCROW_HOLDING` | `LIABILITY` | **DEBIT** | `4005000` | Pelepasan seluruh dana Escrow pesanan |
| `BRAND_AVL_{brand_id}` | `LIABILITY` | **CREDIT** | `3520000` | Masuk ke *Available Balance* Official Brand ($N_{\text{brand}}$) |
| `SYS_LOGISTICS_PAYABLE` | `LIABILITY` | **CREDIT** | `80000` | Alokasi pembayaran tagihan kurir Mode 3 yang ditanggung Brand |
| `SYS_REVENUE_COMMISSION` | `REVENUE` | **CREDIT** | `400000` | Pendapatan Komisi Produk NewTKK ($C_{\text{brand}}$) |
| `SYS_REVENUE_SERVICE_FEE` | `REVENUE` | **CREDIT** | `5000` | Pendapatan Biaya Layanan Aplikasi ($F_{\text{platform}}$) |

*(Catatan: Untuk **Mode 2 - Stok Ready di Bengkel**, nilai $L_{\text{brand\_ship}} = 0$, sehingga `BRAND_AVL_{brand_id}` menerima penuh `3600000`).*

---

## 5. Master Seeder Dataset (`packages/db/src/seed-data.ts` — Tanpa Complexity Tier)

```typescript
// Lokasi File: packages/db/src/seed-data.ts

export const SEED_SYSTEM_CONFIGS = [
  { configKey: 'PAYMENT_EXPIRY_MINUTES', configValue: '60', valueType: 'INTEGER', description: 'Batas waktu tunggu pembayaran Midtrans normal (menit)' },
  { configKey: 'LAST_SLOT_PAYMENT_EXPIRY_MINUTES', configValue: '15', valueType: 'INTEGER', description: 'Batas waktu bayar jika sisa kuota bengkel/stok tinggal 1 unit' },
  { configKey: 'MAX_CONCURRENT_UNPAID_ORDERS', configValue: '2', valueType: 'INTEGER', description: 'Batas maksimal pesanan belum dibayar per akun konsumen' },
  { configKey: 'SLA_BRAND_FITMENT_CHECK_HOURS', configValue: '4', valueType: 'INTEGER', description: 'SLA jam kerja Brand validasi kecocokan mobil' },
  { configKey: 'SLA_WORKSHOP_SCHEDULE_CONFIRM_HOURS', configValue: '8', valueType: 'INTEGER', description: 'SLA jam kerja Bengkel konfirmasi jadwal' },
  { configKey: 'SLA_INBOUND_PACKAGE_CHECK_HOURS', configValue: '24', valueType: 'INTEGER', description: 'Batas jam Bengkel lapor kerusakan paket kurir' },
  { configKey: 'BOOKING_CALENDAR_BUFFER_DAYS', configValue: '1', valueType: 'INTEGER', description: 'Hari buffer tambahan pada kalkulasi kalender Mode 3' },
  { configKey: 'MAX_CUSTOMER_RESCHEDULE_COUNT', configValue: '2', valueType: 'INTEGER', description: 'Maksimal konsumen ubah jadwal kedatangan' },
  { configKey: 'WORKSHOP_STORAGE_MAX_DAYS', configValue: '14', valueType: 'INTEGER', description: 'Batas hari simpan barang di bengkel sebelum Forfeit' },
  { configKey: 'CANCELLATION_HANDLING_FEE_RATE', configValue: '0.0500', valueType: 'DECIMAL', description: 'Potongan biaya admin pembatalan setelah barang tiba di bengkel' },
  { configKey: 'HANDOVER_AUTO_COMPLETE_HOURS', configValue: '48', valueType: 'INTEGER', description: 'Waktu tunggu otomatis selesai setelah QC diunggah (2x24 jam)' },
  { configKey: 'DEFAULT_INSTALLATION_WARRANTY_DAYS', configValue: '30', valueType: 'INTEGER', description: 'Masa garansi pemasangan standar oleh Bengkel Rekanan' },
  { configKey: 'MAX_WARRANTY_OWNERSHIP_TRANSFERS', configValue: '1', valueType: 'INTEGER', description: 'Maksimal pindah tangan garansi saat mobil dijual' },
  { configKey: 'DEFAULT_PLATFORM_SERVICE_FEE_IDR', configValue: '5000', valueType: 'BIGINT', description: 'Biaya layanan aplikasi per transaksi (Rp)' },
  { configKey: 'DEFAULT_BRAND_COMMISSION_RATE', configValue: '0.0800', valueType: 'DECIMAL', description: 'Komisi standar produk Official Brand (8%)' },
  { configKey: 'MINIMUM_PAYOUT_AMOUNT_IDR', configValue: '100000', valueType: 'BIGINT', description: 'Nominal minimum penarikan dana Midtrans Iris (Rp)' },
] as const;

export const SEED_MASTER_VEHICLES_INDONESIA = [
  {
    make: 'Honda', country: 'JAPAN',
    modelDisplayName: 'Brio Satya / RS (DD1)', commercialGroup: 'Brio', chassisCode: 'DD1',
    bodyType: 'HATCHBACK', powertrain: 'ICE',
    generations: [
      { name: 'Gen 1 (2012-2018) 1.2L i-VTEC', yearRange: '[2012,2018]', engineCode: 'L12B', specs: { pcd: '4x100', oilLiter: 3.6, oilSae: '0W-20' } },
      { name: 'Gen 2 All New (2018-2026) 1.2L i-VTEC', yearRange: '[2018,2026]', engineCode: 'L12B', specs: { pcd: '4x100', oilLiter: 3.6, oilSae: '0W-20' } }
    ]
  },
  {
    make: 'Toyota', country: 'JAPAN',
    modelDisplayName: 'Avanza / Veloz RWD (F600/F650)', commercialGroup: 'Avanza', chassisCode: 'F650-RWD',
    bodyType: 'MPV', powertrain: 'ICE',
    generations: [
      { name: 'Avanza / Grand New Avanza Penggerak Belakang (2012-2021)', yearRange: '[2012,2021]', engineCode: '1NR-VE / 2NR-VE', specs: { pcd: '4x114.3', oilLiter: 3.5, oilSae: '0W-20' } }
    ]
  },
  {
    make: 'Toyota', country: 'JAPAN',
    modelDisplayName: 'All New Avanza / Veloz FWD (W100/W101)', commercialGroup: 'Avanza', chassisCode: 'W100-FWD',
    bodyType: 'MPV', powertrain: 'ICE',
    generations: [
      { name: 'All New Avanza & Veloz Penggerak Depan CVT (2021-2026)', yearRange: '[2021,2026]', engineCode: '2NR-VE', specs: { pcd: '5x100', oilLiter: 3.5, oilSae: '0W-20' } }
    ]
  },
  {
    make: 'Toyota', country: 'JAPAN',
    modelDisplayName: 'Kijang Innova Reborn (AN140)', commercialGroup: 'Kijang Innova', chassisCode: 'AN140',
    bodyType: 'MPV', powertrain: 'ICE',
    generations: [
      { name: 'Innova Reborn 2.4 Diesel (2GD-FTV) & 2.0 Bensin (2015-2026)', yearRange: '[2015,2026]', engineCode: '2GD-FTV / 1TR-FE', specs: { pcd: '5x114.3', oilLiter: 7.5, oilSae: '5W-30' } }
    ]
  },
  {
    make: 'Toyota', country: 'JAPAN',
    modelDisplayName: 'Kijang Innova Zenix (AG10)', commercialGroup: 'Kijang Innova', chassisCode: 'AG10',
    bodyType: 'MPV', powertrain: 'HEV',
    generations: [
      { name: 'Innova Zenix 2.0 Hybrid & Gasoline TNGA-C (2022-2026)', yearRange: '[2022,2026]', engineCode: 'M20A-FXS / M20A-FKS', specs: { pcd: '5x114.3', oilLiter: 4.3, oilSae: '0W-16' } }
    ]
  },
  {
    make: 'Mitsubishi', country: 'JAPAN',
    modelDisplayName: 'Pajero Sport (KR/KS)', commercialGroup: 'Pajero Sport', chassisCode: 'KR-KS',
    bodyType: 'SUV', powertrain: 'ICE',
    generations: [
      { name: 'All New Pajero Sport Dakar 2.4 MIVEC (4N15) (2016-2026)', yearRange: '[2016,2026]', engineCode: '4N15', specs: { pcd: '6x139.7', oilLiter: 8.4, oilSae: '5W-30' } }
    ]
  },
  {
    make: 'BMW', country: 'GERMANY',
    modelDisplayName: 'Seri 3 (G20) 320i / 330i', commercialGroup: '3 Series', chassisCode: 'G20',
    bodyType: 'SEDAN', powertrain: 'ICE',
    generations: [
      { name: 'BMW G20 320i / 330i B48 TwinPower Turbo (2019-2026)', yearRange: '[2019,2026]', engineCode: 'B48B20', specs: { pcd: '5x112', oilLiter: 5.25, oilSae: '0W-30' } }
    ]
  },
  {
    make: 'BYD', country: 'CHINA',
    modelDisplayName: 'Seal / Atto 3 / M6 EV', commercialGroup: 'BYD EV Series', chassisCode: 'BYD-E3',
    bodyType: 'SEDAN', powertrain: 'BEV',
    generations: [
      { name: 'BYD Blade Battery Series (2024-2026)', yearRange: '[2024,2026]', engineCode: 'BYD-PMSM', specs: { pcd: '5x114.3' } }
    ]
  }
] as const;
```
