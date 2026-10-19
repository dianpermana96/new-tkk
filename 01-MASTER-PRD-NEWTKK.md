# 📘 DOKUMEN 1: MASTER PRD — BUSINESS LOGIC, OPERATIONAL SOP & END-TO-END STATE MACHINE

* **Nama Projek:** NewTKK
* **Kode Dokumen:** `DOC-01-MASTER-PRD`
* **Lokasi File:** `docs/01-MASTER-PRD-NEWTKK.md`
* **Versi:** `2.1.0-FINAL` *(True All-In Bundled Pricing + Brand-Borne Mode 3 Shipping + Zero Vehicle Tiers)*
* **Tanggal Efektif:** 20 Oktober 2026
* **Klasifikasi Sistem:** O2O (*Online-to-Offline*) Official Automotive Aftermarket, Spareparts & Affiliated Workshop Marketplace
* **Target Pembaca:** Stakeholder, Product Manager, System Architect, Full-Stack/Mobile Engineer, QA Engineer, & AI Coding Agents (Claude Code, Cursor, Antigravity, OpenCode)

---

## 1. Executive Summary & Value Proposition Matrix

### 1.1. Visi Produk & Model Komersial "True All-In O2O"
**NewTKK** adalah platform *marketplace* otomotif terintegrasi berbasis *Online-to-Offline* (O2O) yang menghubungkan **Pemilik Kendaraan (*Vehicle Owner*)** secara langsung dengan **Official Brand Partner (Prinsipal/Distributor Resmi Suku Cadang & Aftermarket)** serta jaringan **Bengkel Rekanan Terafiliasi (*Affiliated Workshops*)**.

NewTKK hadir untuk mengeliminasi tiga masalah struktural utama di pasar purnajual (*aftermarket*) otomotif Indonesia:
1. **Peredaran Barang Palsu/KW:** Keraguan konsumen saat membeli komponen secara daring tanpa jaminan keaslian dari pemegang merek resmi.
2. **Ketidakpastian Kecocokan (*Fitment Error*), Kejutan Biaya Pasang & Kualitas Instalasi:** Risiko salah membeli spesifikasi komponen, kejutan biaya jasa/ongkir kargo yang mahal, serta pemasangan yang tidak sesuai SOP sehingga merusak kendaraan atau menggugurkan garansi.
3. **Saling Lempar Tanggung Jawab (*Blame Game*) Saat Klaim Garansi:** Konflik klasik di mana *Brand* menyalahkan prosedur instalasi bengkel, sementara bengkel menyalahkan cacat pabrikasi produk.

**Tiga Pilar Model Harga "True All-In O2O" NewTKK:**
1. **Harga Produk Sudah Termasuk Gratis Jasa Pasang (`Rp 0` bagi Konsumen):** Seluruh harga katalog produk yang tertera di platform sudah mencakup fasilitas pemasangan profesional di jaringan Bengkel Rekanan Resmi tanpa biaya tambahan dan tanpa pembedaan kelas/tier kendaraan.
2. **Gratis Ongkos Kirim ke Bengkel Rekanan (`Rp 0` bagi Konsumen pada Mode 2 & Mode 3):** Apabila stok fisik dikirim dari Gudang *Official Brand* menuju Bengkel Rekanan pilihan konsumen (*Mode 3: Ship-to-Workshop*), **100% biaya logistik pengiriman ke bengkel ditanggung oleh Official Brand** (dipotong otomatis dari penyelesaian saldo *Brand*).
3. **Penyelesaian Insentif Bengkel secara B2B:** Seluruh hasil penjualan bersih produk dicairkan ke saldo *Official Brand*. Kompensasi jasa pemasangan antara *Official Brand* dan *Bengkel Rekanan* diselesaikan secara B2B sesuai kesepakatan kemitraan mereka di luar sistem pembayaran konsumen.

### 1.2. Matriks Nilai Ekosistem (*3-Sided Marketplace Value Proposition*)

| Aktor Ekosistem | Masalah Utama Saat Ini (*Pain Points*) | Solusi & Nilai Jual NewTKK (*Value Proposition*) |
| :--- | :--- | :--- |
| **Konsumen (*Vehicle Owner*)** | • Takut mendapat barang palsu/rekondisi.<br>• Bingung apakah suku cadang cocok dengan varian mobilnya.<br>• Terbebani ongkir kargo mahal & biaya jasa pasang yang tidak standar di bengkel.<br>• Dipingpong antara penjual dan bengkel saat barang bermasalah. | • **100% Official Brand Store:** Hanya menjual produk dari prinsipal/distributor resmi.<br>• **True All-In Price:** Harga produk **sudah termasuk Gratis Jasa Pasang + Gratis Ongkir ke Bengkel Rekanan Resmi**.<br>• **Double Fitment Protection:** Pencarian terpandu + verifikasi kecocokan manual oleh *Brand* sebelum barang dikirim.<br>• **Single-Window Digital Warranty:** Garansi produk dan garansi instalasi 30 hari menyatu dalam satu sertifikat digital dengan klaim satu pintu. |
| **Official Brand Partner** | • Reputasi merek rusak akibat ulah bengkel umum yang salah memasang produk.<br>• Sulit mengontrol harga eceran, kualitas instalasi, dan validitas klaim garansi. | • **Standarisasi Instalasi & QC:** Produk dipasang oleh bengkel terafiliasi dengan bukti foto QC ber-*watermark* & pemindaian nomor seri.<br>• **Proteksi Beban Garansi:** Pembelian produk tanpa instalasi di bengkel rekanan (Mode 1) otomatis membebaskan *Brand* dari kewajiban garansi.<br>• **Pencairan Terpusat:** Dana penjualan bersih produk langsung cair ke saldo *Official Brand*. |
| **Affiliated Workshop (*Bengkel Rekanan*)** | • Kehilangan pelanggan yang membeli *part* di *e-commerce* lalu bingung pasang di mana.<br>• Keterbatasan modal kerja dan ruang gudang untuk menyetok ribuan SKU dari berbagai *brand*. | • **Akuisisi Pelanggan Pasti (*Zero-CAC*):** Menerima pelanggan terjadwal dari ekosistem resmi *Official Brand*.<br>• **Tanpa Beban Modal Stok (*Ship-to-Workshop*):** Dapat melayani pemasangan ribuan SKU *Brand* dengan barang dikirim gratis langsung dari gudang pusat *Brand*.<br>• **Operasional Ringkas & Bukti Rekonsiliasi B2B:** Sistem mencatat seluruh bukti kerja QC & nomor seri sebagai dasar klaim B2B ke *Brand*. |

---

## 2. Spesifikasi Tech Stack & Standar Arsitektur (Baseline Oktober 2026)

Sistem NewTKK dibangun dengan pendekatan **AI-Agentic Native, End-to-End Type-Safe Monorepo, dan Headless API-First** agar siap melayani **4 Portal Web (MVP)** sekaligus ekspansi **Mobile App (iOS/Android)** tanpa duplikasi logika bisnis.

| Layer Arsitektur | Teknologi & Versi Terkunci (Okt 2026) | Aturan Implementasi & Alasan Arsitektural |
| :--- | :--- | :--- |
| **Monorepo & Package Manager** | **Turborepo v2.x** + **pnpm v10.x** *(dengan fitur `pnpm catalogs`)* | Seluruh aplikasi (`apps/*`) dan pustaka bersama (`packages/*`) berada dalam satu repositori. Versi dependensi dikunci terpusat di `pnpm-workspace.yaml` untuk mencegah *version drift*. |
| **Runtime & Bahasa** | **Node.js v24.x LTS** *(Siap naik ke **v26 LTS** akhir Okt 2026)* + **TypeScript v7.0.2** *(Native Go Compiler)* | Menggunakan `typescript@^7.0.2` untuk kompilasi dan *type-checking* super cepat, didampingi paket kompatibilitas `@typescript/typescript6@^6.0.2` untuk *linter/plugin* yang masih membutuhkan JS Compiler API. |
| **Shared Contracts & Validation** | **Zod v4.6.4** + **OpenAPI 3.1 / ts-rest** (`@newtkk/contracts`) | Memanfaatkan `z.compile()` dan `.validate()` pada Zod v4 untuk validasi skema berkecepatan tinggi. Kontrak API didefinisikan satu kali dan dikonsumsi bersama oleh Backend, Web, dan Mobile. |
| **Backend API Engine** | **NestJS v12.x** *(Native ESM)* | Arsitektur modular berbasis *Domain-Driven Design (DDD)*: memisahkan modul `Catalog`, `Fitment`, `OrderStateMachine`, `Booking`, `EscrowLedger`, dan `Warranty`. |
| **Database Utama & Geospasial** | **PostgreSQL v18.6** + **PostGIS v3.5+** | Memanfaatkan fungsi bawaan PostgreSQL 18 **`uuidv7()`** untuk seluruh *Primary Key* (terurut secara kronologis, optimal untuk *B-Tree index* dan *audit log*). **PostGIS** menangani kalkulasi jarak/radius bengkel terdekat secara instan di dalam database (biaya Rp 0 tanpa ketergantungan Google Distance Matrix API). |
| **ORM & Database Migration** | **Drizzle ORM v0.45.4** *(Kompatibel penuh menuju **v1.0**)* | Skema SQL deklaratif berbasis TypeScript (`@newtkk/db`), bebas *overhead* memori, mendukung transaksi ACID dengan `SELECT ... FOR UPDATE` pada buku besar keuangan. |
| **Frontend Web (4 Portal)** | **Next.js v16.4 (App Router, Turbopack)** + **React 19.3** + **Tailwind CSS v4** + **shadcn/ui** + **TanStack Query v5** | Menggunakan fitur standar Next.js 16.4 (`proxy.ts` menggantikan `middleware.ts`, *Cache Components*, dan *Next.js DevTools MCP* untuk observabilitas AI Agent). |
| **Queue, Cron & Caching** | **Redis v7.4 / Valkey** + **BullMQ v5.81+** | Menangani *background jobs* berbatas waktu (SLA): *payment expiry* (60 menit), SLA validasi *fitment* & jadwal, pengingat H-1, *auto-complete* (2x24 jam), dan eksekusi batas simpan (14 hari). |
| **Search Engine & Object Storage** | **Typesense v27+ (Self-Hosted)** + **Cloudflare R2 (S3-Compatible)** | **Typesense** untuk pencarian instan SKU & kombinasi *Make-Model-Year* (< 30ms). **Cloudflare R2** untuk penyimpanan foto produk & bukti QC bengkel dengan *Zero Egress Fee* (bebas biaya *bandwidth* keluar). |
| **Payment Gateway & Payout** | **Midtrans Snap/Core API** *(Pay-In)* + **Midtrans Iris API** *(Disbursement/Payout)* | Pembayaran konsumen masuk ke *Escrow Account* Midtrans NewTKK. Pembagian hak dana dikelola oleh **Internal Double-Entry Escrow Ledger** di PostgreSQL, dan pencairan ke rekening bank *Brand* (serta *Customer Refund*) dieksekusi melalui **Midtrans Iris API**. |
| **Logistik & Komunikasi** | **Biteship API** *(Reguler & Kargo)* + **WhatsApp Business Cloud API** | Biteship menangani ongkir multi-ekspedisi (termasuk kargo berat/volumetrik). WhatsApp Cloud API mengirimkan notifikasi transaksional beserta **Signed Magic Link** untuk aksi cepat bengkel. |

---

## 3. Arsitektur Entitas, Afiliasi Mitra & Master Data Kendaraan

### 3.1. Relasi Multi-Tenancy & Siklus Hidup Afiliasi ($M:N$)
Di dalam NewTKK, **Official Brand** dan **Affiliated Workshop (Bengkel Rekanan)** adalah dua entitas bisnis terpisah yang dihubungkan melalui tabel relasi Many-to-Many (`brand_workshop_affiliations`):
* Satu *Official Brand* dapat menunjuk banyak *Bengkel Rekanan* di seluruh Indonesia.
* Satu *Bengkel Rekanan* dapat berafiliasi dengan banyak *Official Brand* sekaligus (misal: Bengkel A menjadi *authorized installer* untuk Brand Suspensi X, Brand Pengereman Y, dan Brand Oli Z).

**Status Siklus Hidup Afiliasi (`Affiliation Status`):**
1. **`PENDING_BRAND_APPROVAL` / `PENDING_WORKSHOP_APPROVAL`:** Undangan kemitraan dikirim oleh salah satu pihak dan menunggu persetujuan pihak lainnya di portal masing-masing.
2. **`ACTIVE`:** Kedua pihak menyetujui kemitraan dan cabang bengkel telah mengaktifkan kapabilitas layanan (`workshop_branch_capabilities`) yang relevan. Bengkel resmi muncul sebagai lokasi pemasangan gratis saat konsumen membeli produk *Brand* terkait.
3. **`SUSPENDED`:** Dibekukan sementara (oleh *Brand*, Bengkel sendiri, atau Super Admin NewTKK karena pelanggaran SLA/QC). Bengkel tidak menerima *booking* baru, namun wajib menyelesaikan seluruh pesanan yang sudah berstatus `PAID` sebelumnya.
4. **`REVOKED`:** Kemitraan dihentikan secara permanen.

### 3.2. Pemetaan Kapabilitas Layanan Bengkel & Master Data Kendaraan (Tanpa Tier Harga)
1. **Master Layanan (`Service Type Catalog`):** Super Admin NewTKK mendefinisikan daftar kode jenis pekerjaan standar (misal: `SVC-SUSP-COILOVER`, `SVC-BRAKE-PAD-AXLE`, `SVC-OIL-ENGINE-CHANGE`, `SVC-ELEC-DASHCAM-2CH`) beserta estimasi durasi pengerjaan dan rincian bahan bantu standar (*All-In Consumables*).
2. **Pemetaan oleh Brand & Bengkel:**
   * Saat *Official Brand* membuat SKU produk, *Brand* memilih `service_type_code` yang dibutuhkan untuk memasang produk tersebut.
   * Masing-masing Cabang Bengkel Rekanan mencentang `service_type_code` mana saja yang mampu mereka kerjakan (`workshop_branch_capabilities`).
3. **Master Data Kendaraan Indonesia (2005–2026) — Murni untuk Fitment & Garansi:**
   * Sistem **tidak menggunakan klasifikasi Tier harga kendaraan**. Seluruh jenis mobil mendapatkan fasilitas gratis pasang yang sama sesuai harga katalog SKU.
   * Basis data `vehicle_makes`, `vehicle_models` (dipisah berdasarkan `chassis_code` untuk mencegah tabrakan tahun transisi seperti *Innova Reborn AN140* vs *Innova Zenix AG10*), dan `vehicle_generations` (`int4range`) dibangun dari kombinasi:
     1. Basis Data NJKB Kemendagri / Bapenda Samsat (Hingga Tahun Pajak 2026).
     2. Data Retail & Wholesale GAIKINDO (2000–2026), termasuk merek EV baru.
     3. Spesifikasi Fitment Teknis OEM (PCD roda, ukuran ban OEM, tipe soket bohlam, kapasitas liter oli).
     4. **Crowdsourced Fallback Queue (`PENDING_MASTER_VEHICLE_REVIEW`):** Antrean kurasi di *Admin Backoffice* jika konsumen menginput varian mobil CBU langka secara manual saat *checkout*.

---

## 4. Taksonomi Produk: *Dual-Label 10 Root Categories* & *3-Level Dynamic Tree*

Untuk menjamin tampilan **UI/UX Mobile tetap simetris dan bebas potongan teks (*truncation*)** sekaligus mempertahankan kekuatan **SEO pada halaman web**, setiap kategori utama di NewTKK menerapkan skema **Dual-Label Taxonomy** yang terdiri dari:
* **`ui_short_label` (Maksimal 14 Karakter):** Digunakan khusus pada *Mobile Icon Grid*, *Bottom Sheet*, *Navigation Bar*, dan *Filter Chips*.
* **`full_display_name`:** Digunakan pada *Page Header (H1)*, *Breadcrumb*, *Meta Title SEO*, dan *Desktop Mega Menu*.
* **Arsitektur 3-Level Dinamis:** **Level 1 (Root Category)** -> **Level 2 (Sub-Category)** -> **Level 3 (Leaf Product Type)**. Super Admin dapat menambah kategori/sub-kategori baru beserta skema atribut `JSONB`-nya langsung melalui *Admin Backoffice* tanpa perubahan kode (*zero-code taxonomy expansion*).

### Matriks Lengkap 10 Kategori Utama & Aturan Bisnisnya

| Kode L1 | `ui_short_label` *(Mobile UI)* | `full_display_name` *(Header & SEO)* | Sub-Kategori Level 2 (L2) | Contoh Tipe Produk Level 3 (L3) | Tipe Fitment & Skema Atribut `JSONB` | Aturan Fulfillment & Jenis Garansi Default |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CAT-01** | **Kaki-Kaki** | **Suspensi, Kaki-Kaki & Kemudi** | • 1.1. Shockbreaker & Coilover<br>• 1.2. Per & Lowering Kit<br>• 1.3. Komponen Understeel<br>• 1.4. Sistem Kemudi (Steering)<br>• 1.5. Chassis Bracing & Stabilizer | • Shockbreaker Set/Depan/Belakang, Coilover<br>• Lowering Spring, Lift Kit<br>• Tie Rod, Ball Joint, Link Stabilizer, Bushing, Lower Arm<br>• Rack Steer, EPS<br>• Strut Bar, Underbrace, Sway Bar | **`SPECIFIC_FIT`**<br>JSONB: `axle_position`, `damping_clicks`, `drop_mm`, `series_type` | • **Fulfillment:** `ALLOW_ALL`<br>• **Garansi:** `FUNCTIONAL_PERIODIC` (6–24 Bln)<br>• *Garansi produk hangus jika tanpa pasang di Bengkel Rekanan.* |
| **CAT-02** | **Pengereman** | **Sistem Pengereman (Brakes)** | • 2.1. Kampas Rem (Brake Pads)<br>• 2.2. Piringan Cakram (Rotors)<br>• 2.3. Big Brake Kit (BBK) & Kaliper<br>• 2.4. Hidrolik & Selang Rem | • Kampas Rem Depan/Belakang, Sepatu Rem<br>• Disc Rotor Standard, Slotted/Drilled Rotor<br>• Paket BBK (4-Pot/6-Pot)<br>• Master Rem, Selang Rem Braided, Minyak Rem | **`SPECIFIC_FIT`** *(Minyak Rem: `UNIVERSAL`)*<br>JSONB: `axle_position`, `rotor_dia_mm`, `material_compound`, `dot_spec` | • **Fulfillment:** `ALLOW_ALL` *(BBK: dapat diset `MANDATORY_WORKSHOP`)*<br>• **Garansi:**<br>- BBK/Rotor/Selang: `FUNCTIONAL_PERIODIC`<br>- Kampas/Minyak: `AUTHENTICITY_DOA` |
| **CAT-03** | **Velg & Ban** | **Velg, Ban & Aksesoris Roda** | • 3.1. Velg Mobil (Wheels)<br>• 3.2. Ban Mobil (Tires)<br>• 3.3. Ban Khusus EV & Run-Flat<br>• 3.4. Aksesoris & Mur Roda | • Velg Casting, Flow Forming, Forged<br>• Ban Harian (HT/Touring), UHP, AT/MT<br>• Ban EV (Acoustic Foam)<br>• Lug Nut, Center Ring, Spacer, Sensor TPMS | **`SPEC_BASED` & `SPECIFIC_FIT`**<br>JSONB Velg: `ring`, `width_jj`, `pcd`, `offset_et`, `center_bore`<br>JSONB Ban: `width`, `aspect_ratio`, `rim`, `load_speed_index` | • **Fulfillment:** `ALLOW_ALL`<br>• **Garansi:** `FUNCTIONAL_PERIODIC` (Garansi cacat struktur pabrik + Garansi Balancing/Instalasi Bengkel). |
| **CAT-04** | **Oli & Servis** | **Oli, Fluida & Servis Berkala** | • 4.1. Oli Mesin (Engine Oil)<br>• 4.2. Oli Transmisi & Gardan<br>• 4.3. Filter Kendaraan<br>• 4.4. Pengapian & Sabuk Mesin<br>• 4.5. Wiper & Cairan Servis | • Oli Mesin Bensin / Diesel<br>• Oli Transmisi ATF, CVT Fluid, Oli Gardan<br>• Filter Oli, Filter Udara OEM, Filter Kabin AC, Filter Solar<br>• Busi Iridium/Laser, V-Belt<br>• Wiper Blade, Injector/Brake Cleaner | **`SPEC_BASED` (Oli) & `SPECIFIC_FIT` (Filter/Busi/Wiper)**<br>JSONB: `sae_viscosity`, `api_ilsac_grade`, `base_oil_type`, `volume_liter`, `blade_inch` | • **Fulfillment:** `ALLOW_ALL`<br>• **Garansi:** `AUTHENTICITY_DOA` *(Jaminan 100% Keaslian Produk Resmi + Garansi Instalasi Bengkel bebas rembes/bocor).* |
| **CAT-05** | **Performa** | **Performa Mesin, Intake & Exhaust** | • 5.1. Sistem Knalpot (Exhaust)<br>• 5.2. Air Intake & Filter Racing<br>• 5.3. Forced Induction & Intercooler<br>• 5.4. Engine Management & Fuel | • Muffler, Resonator, Downpipe, Frontpipe, Full Exhaust<br>• Replacement Air Filter Racing, Open Pod Intake<br>• Intercooler Kit, Piping Kit, BOV<br>• Throttle Controller, Piggyback, Ignition Coil Racing | **`SPECIFIC_FIT`**<br>JSONB: `inlet_inch`, `material_grade` (SUS304/Titanium), `installation_bolt_on` (Bool) | • **Fulfillment:** `ALLOW_ALL`<br>• **Garansi:** `FUNCTIONAL_PERIODIC` (Garansi cacat las/kebocoran/modul elektronik 6–24 Bulan). |
| **CAT-06** | **Pendingin & AC** | **Pendingin Mesin & Sistem AC** | • 6.1. Pendingin Mesin (Radiator)<br>• 6.2. Komponen AC Mobil<br>• 6.3. Coolant & Refrigeran | • Radiator OEM/Aluminium, Extra Fan, Water Pump, Thermostat, Selang Silikon<br>• Kompresor AC, Kondensor, Evaporator, Expansion Valve<br>• Radiator Coolant, Freon | **`SPECIFIC_FIT`** *(Coolant: `UNIVERSAL`)*<br>JSONB: `ply_rows`, `transmission_type`, `coolant_premix_ratio` | • **Fulfillment:** `ALLOW_ALL`<br>• **Garansi:** `FUNCTIONAL_PERIODIC` (Wajib pasang di Bengkel Rekanan untuk validitas garansi bocor/kompresor). |
| **CAT-07** | **Transmisi** | **Transmisi, Kopling & Drivetrain** | • 7.1. Paket Kopling (Clutch)<br>• 7.2. Penggerak Roda (Driveshaft)<br>• 7.3. Differential & Mounting | • Kampas Kopling, Clutch Cover, Release Bearing, Flywheel<br>• CV Joint, As Roda, Boot As Roda<br>• Engine Mounting, Trans Mounting, LSD | **`SPECIFIC_FIT`**<br>JSONB: `transmission_compat`, `clutch_stage`, `mounting_position` | • **Fulfillment:** `ALLOW_ALL`<br>• **Garansi:** `FUNCTIONAL_PERIODIC` & `AUTHENTICITY_DOA`. |
| **CAT-08** | **Kelistrikan** | **Lampu, Aki, Dashcam & Elektrikal** | • 8.1. Lampu & Retrofit Projektor<br>• 8.2. Dashcam & Kamera Parkir<br>• 8.3. Aki Mobil (Battery)<br>• 8.4. Klakson & Modul Elektrik | • Bohlam LED, Projektor Bi-LED, Foglamp Projektor, Stoplamp<br>• Dashcam (1Ch/2Ch/3Ch + Hardwire), Kamera 360<br>• Aki MF/EFB/AGM, Aki Auxiliary EV<br>• Klakson, Modul Otomatis | **`SPEC_BASED` & `SPECIFIC_FIT`**<br>JSONB Lampu: `socket_code`, `wattage`, `kelvin`<br>JSONB Aki: `jis_din_code`, `ah_capacity`, `cca_rating` | • **Fulfillment:** `ALLOW_ALL` *(Retrofit Bi-LED & Kamera 360 dapat diset `MANDATORY_WORKSHOP`)*<br>• **Garansi:** `FUNCTIONAL_PERIODIC` (6–24 Bulan, hangus jika instalasi sendiri). |
| **CAT-09** | **Audio Mobil** | **Audio, Head Unit & Peredam Kabin** | • 9.1. Head Unit & Layar Hiburan<br>• 9.2. Speaker, Subwoofer & DSP<br>• 9.3. Peredam Suara Kabin<br>• 9.4. Frame, Canbus & Kabel PnP | • Android Head Unit, OEM-Look Screen, Roof Monitor<br>• Speaker Split/Coaxial, Subwoofer Kolong, Amplifier, DSP<br>• Peredam Pintu/Lantai/Kap Mesin<br>• Frame + Soket Canbus PnP | **`UNIVERSAL` & `SPECIFIC_FIT`**<br>JSONB: `screen_inch`, `ram_rom_gb`, `rms_watt`, `thickness_mm`, `sheet_count` | • **Fulfillment:** `ALLOW_ALL`<br>• **Garansi:** `FUNCTIONAL_PERIODIC` (12 Bulan). *Garansi elektronik hangus jika dipasang di luar Bengkel Rekanan.* |
| **CAT-10** | **Bodi & Proteksi** | **Eksterior, Interior, Proteksi & EV** | • 10.1. Kaca Film, Coating & PPF<br>• 10.2. Bodykit, Aero & Eksterior<br>• 10.3. Interior, Jok & Karpet<br>• 10.4. EV Charger & Aksesoris EV | • Paket Kaca Film, Nano Ceramic Coating, PPF<br>• Bodykit, Spoiler, Grille, Roof Box/Rack, Towing Bar<br>• Karpet Custom, Patented Seat Cover, Jok Racing<br>• EV Home Wallbox Charger, Portable EV Charger | **`SPECIFIC_FIT` & `UNIVERSAL`**<br>JSONB: `vlt_percent`, `irr_percent`, `material_type`, `ev_power_kw`, `ev_plug_type` | • **Fulfillment:**<br>- **10.1 (Kaca Film/Coating/PPF):** `MANDATORY_WORKSHOP_ONLY`<br>- **10.2–10.4:** `ALLOW_ALL` (Mendukung `PRE_ORDER_CUSTOM`). |

---

## 5. Mode Pemenuhan Pesanan (*Fulfillment Modes*), Logika Kalender & Klausul Garansi

### 5.1. Tiga Mode Pemenuhan Pesanan (*True All-In O2O Scenarios*)

| Parameter Aturan | Mode 1: Direct Ship-to-Home *(Beli Produk Saja — Kirim ke Rumah)* | Mode 2: O2O Ready at Workshop *(Pasang di Bengkel — Stok Tersedia di Bengkel)* | Mode 3: O2O Ship-to-Workshop *(Pasang di Bengkel — Dikirim dari Gudang Brand)* |
| :--- | :--- | :--- | :--- |
| **Syarat Ketersediaan Mode** | Produk memiliki kebijakan `fulfillment_policy = ALLOW_ALL`. | Bengkel Rekanan yang dipilih memiliki `workshop_inventory.available_qty >= order_qty`. | Stok di Bengkel Rekanan `0` (atau tidak menyetok), namun stok Gudang Pusat *Brand* tersedia / `PRE_ORDER_CUSTOM`. |
| **Asal & Tujuan Barang** | Gudang *Brand* -> Alamat Rumah Konsumen. | Barang sudah berada di etalase/gudang Bengkel Rekanan terpilih. | Gudang *Brand* -> Alamat Bengkel Rekanan terpilih (u.p. Nama Konsumen + Kode Booking). |
| **Harga Produk & Jasa Pasang** | Harga Katalog Normal *(Tidak ada diskon pengurangan harga)*. | **Harga Katalog All-In (Sudah Termasuk Gratis Jasa Pasang / Rp 0)**. | **Harga Katalog All-In (Sudah Termasuk Gratis Jasa Pasang / Rp 0)**. |
| **Beban Ongkos Kirim Kurir** | **Dibayar oleh Konsumen** ($L_{\text{cust}} > 0$) saat *Checkout*. | **GRATIS (`Rp 0` — Tanpa Pengiriman Kurir)**. | **GRATIS bagi Konsumen (`Rp 0`) — 100% Ongkos Kirim ke Bengkel Ditanggung oleh Official Brand ($L_{\text{brand\_ship}}$)**. |
| **Komponen Dibayar Konsumen** | $\text{Harga Produk} + \text{Ongkir ke Rumah} + \text{Platform Fee}$ | $\text{Harga Produk} + \text{Platform Fee}$ | $\text{Harga Produk} + \text{Platform Fee}$ |
| **Rumus Penguncian Kalender Booking ($T_{\text{min}}$)** | Tidak menggunakan kalender booking. | $$T_{\text{min}} = \text{Hari Ini } (H+0) \text{ atau } (H+1)$$ *(Sesuai jam batas operasional bengkel).* | $$T_{\text{min}} = H + D_{\text{prod}} + D_{\text{ship}} + 1 \text{ Hari Buffer}$$ |
| **Status Garansi Produk** | ❌ **VOID / HANGUS** *(Khusus produk bergaransi fungsi, wajib centang Waiver).* | ✅ **AKTIF PENUH** *(Garansi Produk Brand + Garansi Instalasi Bengkel 30 Hari).* | ✅ **AKTIF PENUH** *(Garansi Produk Brand + Garansi Instalasi Bengkel 30 Hari).* |

**Keterangan Variabel Rumus Kalender Mode 3:**
* $D_{\text{prod}}$ = `production_lead_time_days` (`0` untuk barang *Ready Stock*, atau `1..14` hari untuk barang berstatus `PRE_ORDER_CUSTOM` seperti karpet custom/knalpot/jok).
* $D_{\text{ship}}$ = Estimasi hari pengiriman maksimum dari kurir yang mengantar barang dari Gudang Brand ke Bengkel Rekanan (misal: Reguler 2–3 hari -> diambil angka `3`).
* $1 \text{ Hari Buffer}$ = Jeda pengaman 1 hari kerja untuk proses verifikasi *fitment* dan *inbound check* di bengkel.

### 5.2. Protokol Hukum & Sistem: *Warranty Void Waiver* (Khusus Mode 1)
Apabila konsumen membeli produk yang memiliki jaminan garansi (`warranty_duration_months > 0`), namun memilih **Mode 1 (Beli Produk Saja / Kirim ke Rumah)**, maka sistem mengeksekusi aturan wajib berikut:
1. Menampilkan komponen visual *High-Contrast Warning Modal/Banner* di halaman *Checkout* (dilengkapi tombol *1-Click Switch* untuk beralih kembali ke *Pasang Gratis di Bengkel Rekanan*).
2. Menyediakan *checkbox* persetujuan yang **wajib dicentang** sebelum tombol pembayaran aktif:
   > *"Saya memahami bahwa harga produk sudah termasuk fasilitas gratis pemasangan dan gratis pengiriman ke Bengkel Rekanan Resmi. Karena saya memilih pengiriman langsung ke alamat rumah tanpa pemasangan di Bengkel Rekanan NewTKK, maka **jaminan Garansi Produk dinyatakan TIDAK BERLAKU (HANGUS)** sejak barang diterima, kecuali terbukti cacat fisik saat paket dibuka (unboxing DOA maksimal 1x24 jam)."*
3. Sistem menyimpan bukti audit `warranty_waiver_accepted = true`, `warranty_waiver_accepted_at` (*UTC timestamp*), `warranty_waiver_text_snapshot`, dan `customer_checkout_ip` pada tabel `orders`.

---

## 6. Strategi Kompatibilitas Kendaraan (*Hybrid Soft-Fitment & Human Verification*)

Untuk memaksimalkan konversi belanja di fase MVP sekaligus menekan angka kesalahan beli hingga **0%**, NewTKK memadukan pencarian yang tidak mengikat di hulu dengan verifikasi wajib di titik transaksi:

### 6.1. Tahap Hulu: *Frictionless Soft-Fitment Browsing*
* Konsumen **tidak diwajibkan** mengisi fitur "Garasi Saya" saat pertama kali masuk ke aplikasi.
* Katalog menyediakan filter cepat 3-level: **Merek (`Make`) -> Model (`Model`) -> Tahun (`Year`)**.
* Setiap produk memiliki indikator kecocokan: `UNIVERSAL_FIT` (cocok untuk semua mobil), `SPEC_BASED` (berdasarkan ukuran/spesifikasi seperti viskositas oli atau ukuran ban), atau `SPECIFIC_FIT` (memiliki pemetaan ke model kendaraan tertentu).

### 6.2. Tahap Hilir: *Mandatory Checkout Vehicle Declaration & Brand Verification*
Saat konsumen memilih **Mode 2 atau Mode 3 (Pasang di Bengkel)**, halaman *Checkout* mewajibkan pengisian **Data Kendaraan Pemasangan**:
1. **Merek & Model Kendaraan** (Dipilih dari Master Data, atau input manual jika varian sangat langka).
2. **Tahun Pembuatan Kendaraan** (`YYYY`).
3. **Catatan Varian / Mesin / Transmisi** (Input teks singkat, misal: *"2.4 Diesel Dakar AT"*).
4. **Nomor Polisi Kendaraan** (Wajib — divalidasi saat *check-in* fisik di bengkel dan dicetak permanen pada Sertifikat Garansi Digital).

**Protokol *Human-in-the-Loop Fitment Check* oleh Official Brand:**
* Segera setelah pembayaran berstatus `PAID_AWAITING_VALIDATION`, *Official Brand* menerima notifikasi di *Brand Portal* dan WhatsApp yang menyandingkan **Daftar SKU Pesanan** dengan **Data Kendaraan Konsumen**.
* Dalam batas **SLA 1x4 Jam Kerja**, Admin *Brand* wajib memilih salah satu keputusan:
  1. **`APPROVE_FITMENT`:** Mengonfirmasi bahwa produk cocok dengan mobil konsumen. Label pengiriman baru dapat dicetak setelah tombol ini ditekan.
  2. **`FLAG_MISMATCH_SUGGEST_ALT`:** Produk tidak cocok; membuka ruang komunikasi dengan konsumen untuk menukar ke SKU yang tepat atau membatalkan pesanan.
  3. **`REJECT_FITMENT_FULL_REFUND`:** Tidak ada produk yang cocok untuk mobil tersebut; pesanan otomatis dibatalkan dan dana konsumen dikembalikan **100% (*Full Refund*)** sebelum barang dikirim.

---

## 7. Master Order, Booking & Item-Level State Machine

### 7.1. Diagram Alur State Machine Utama (Mode 2 & Mode 3 O2O)

```text
                        [1. PENDING_PAYMENT]
                                 │
          ┌──────────────────────┴──────────────────────┐
   (Timeout 60m)                                 (Webhook Midtrans Settlement)
          ▼                                             ▼
[1B. PAYMENT_EXPIRED]                      [2. PAID_AWAITING_VALIDATION]
                                                        │
                         ┌──────────────────────────────┼──────────────────────────────┐
          (Brand Reject / Batal Awal)      (Mode 2: Stok Ready di Bengkel)   (Mode 3: Kirim dari Brand)
                         ▼                              │                              ▼
             [CANCELLED_FULL_REFUND]                    │                 [3. PROCESSING_SHIPMENT]
                                                        │                              │ (Kurir Pickup)
                                                        │                              ▼
                                                        │                 [4. IN_TRANSIT_TO_WORKSHOP]
                                                        │                              │
                                                        │                 (Bengkel Foto Segel & Terima)
                                                        ▼                              ▼
                                           [5. READY_AT_WORKSHOP_FOR_INSTALL] ◄────────┘
                                                        │
                         ┌──────────────────────────────┴──────────────────────────────┐
           (Konsumen Lewat > 14 Hari)                                    (Hari-H: Scan QR / PIN Check-in
                                                                          + Foto Kondisi Awal Mobil)
                         ▼                                                             ▼
         [STORAGE_EXPIRED_FORFEITED]                                      [6. CUSTOMER_CHECKED_IN]
                                                                                       │
                                                                         (Scan Serial Number & Mulai)
                                                                                       ▼
                                                                      [7. INSTALLATION_IN_PROGRESS]
                                                                                       │
                         ┌──────────────────────────────┬──────────────────────────────┤
        (Kondisi Mobil Rusak/Tidak Aman)     (Insiden Kerusakan oleh Mekanik)    (Selesai Pasang + Upload QC
                                                                                  + Checklist Part Lama)
                         ▼                              ▼                              ▼
       [ABORTED_BY_VEHICLE_CONDITION]    [INCIDENT_UNDER_MEDIATION]     [8. AWAITING_CUSTOMER_HANDOVER]
                                                                                       │
                                                                    (Konsumen Klik Selesai / Auto 2x24 Jam)
                                                                                       ▼
                                                                        [9. COMPLETED_AND_SETTLED]
                                                                         *(atau PARTIALLY_COMPLETED)*
```

### 7.2. Tabel Spesifikasi Transisi Status, SOP & Efek Samping Sistem

| Kode Status | Aktor Pemicu | Syarat Validasi & SOP Wajib | Efek Samping Sistem (*Automated Side Effects*) |
| :--- | :--- | :--- | :--- |
| **`PENDING_PAYMENT`** | Konsumen | Checkout valid; kuota stok di-*reserve* sementara (`reserved_qty + 1`). | Menjalankan *BullMQ delayed job* 60 menit (atau 15 menit jika slot/stok terakhir). Jika belum dibayar -> `PAYMENT_EXPIRED` & kembalikan stok. |
| **`PAID_AWAITING_VALIDATION`** | Webhook Midtrans | Signature SHA-512 valid; dana masuk ke *Escrow Ledger* (`ESCROW_HELD`). | • Kirim WA & Notifikasi ke **Brand** (SLA 1x4 jam kerja untuk *Fitment Check*).<br>• Kirim WA Magic Link ke **Bengkel** (SLA 2x4 jam kerja untuk konfirmasi Sesi Jadwal: Pagi/Siang/Sore). |
| **`RESCHEDULE_PROPOSED`** | Admin Bengkel | Sesi jadwal yang dipilih konsumen penuh; bengkel mengajukan opsi tanggal/sesi baru. | Kirim WA ke konsumen untuk klik **Setuju Jadwal Baru**, **Ajukan Tanggal Lain**, atau **Pindah Bengkel Rekanan**. |
| **`PROCESSING_SHIPMENT`** | Sistem (setelah Brand + Bengkel OK) | `brand_fitment_confirmed = true` **DAN** `workshop_schedule_confirmed = true`. | Membuka kunci tombol **"Cetak Label Pengiriman & Request Pickup Kurir"** di *Brand Portal* (khusus Mode 3, ongkir ditanggung Brand). |
| **`IN_TRANSIT_TO_WORKSHOP`** | Webhook Biteship / Kurir | Paket telah diambil oleh kurir dari gudang *Official Brand* menuju alamat Bengkel. | Menampilkan pelacakan posisi paket (*live tracking*) di halaman pesanan Konsumen, Brand, dan Bengkel. |
| **`INBOUND_DAMAGE_REPORTED`** | Admin Bengkel | Saat paket tiba di bengkel, kemasan rusak berat/pecah akibat kurir. Dilaporkan dalam <= 24 jam dengan foto/video. | Menahan jadwal booking; membuka tiket klaim asuransi logistik & memicu pengiriman unit pengganti oleh *Brand*. |
| **`READY_AT_WORKSHOP_FOR_INSTALL`** | Admin Bengkel *(Mode 3)* / Sistem *(Mode 2)* | **SOP Inbound Check (Mode 3):** Admin bengkel wajib mengunggah **minimal 1 foto paket bersegel** yang tiba di bengkel dan klik "Konfirmasi Terima Paket". | • Tanggung jawab fisik barang beralih dari Kurir ke Bengkel.<br>• Kirim WA ke konsumen: *"Produk sudah aman di Bengkel X, siap dipasang pada jadwal Anda"* beserta **QR Code Check-in**.<br>• Memulai hitung mundur **SLA Masa Simpan 14 Hari**. |
| **`CUSTOMER_CHECKED_IN`** | Admin Bengkel + Konsumen/Supir | **SOP Check-in & Pre-Inspection:**<br>1. Bengkel memindai QR Code atau input 6-Digit PIN Booking.<br>2. Mekanik wajib mengambil **minimal 2 foto kondisi awal kendaraan** (*Pre-Installation Condition Photo*: area kerja & indikator *dashboard*). | • Menghentikan *timer* masa simpan 14 hari.<br>• Mencatat `checked_in_at` dan mengunci hak pembatalan sepihak oleh konsumen. |
| **`INSTALLATION_IN_PROGRESS`** | Mekanik / Admin Bengkel | Mekanik membuka kemasan produk dan memindai/menginput **Nomor Seri (*Serial Number*)** per unit fisik produk (`order_item_units`). | Memvalidasi keaslian produk *Official Brand* dan mengikat nomor seri tersebut ke draf garansi. |
| **`AWAITING_CUSTOMER_HANDOVER`** | Mekanik / Admin Bengkel | **SOP Quality Control (QC) Akhir:**<br>1. Unggah foto produk yang sudah terpasang di mobil.<br>2. Unggah foto mobil yang memperlihatkan **Plat Nomor Polisi**.<br>3. Pilih opsi wajib **Status Komponen Lama (`Old Parts Checklist`)**: `RETURNED_TO_TRUNK` atau `DISPOSED_WITH_CONSENT`. | • Mengirim notifikasi WA & *push notification* ke pemilik akun konsumen beserta galeri foto hasil QC.<br>• Memulai *BullMQ timer* **Auto-Complete 2x24 Jam (48 Jam)**. |
| **`COMPLETED_AND_SETTLED`** *(atau `PARTIALLY_COMPLETED`)* | Pemilik Akun Konsumen / Sistem Cron 48 Jam | Konsumen menekan tombol **"Konfirmasi Pekerjaan Selesai"** (atau 48 jam berlalu tanpa laporan insiden). | 1. Mencairkan dana bersih produk ($N_{\text{brand}}$) dari *Escrow* ke *Available Balance* milik **Brand**.<br>2. Mengakui pendapatan komisi **NewTKK** & alokasi kurir.<br>3. Menerbitkan **Digital Warranty Passport**.<br>4. Membuka formulir **Dual-Rating** (Rating Produk vs. Rating Bengkel). |

*(Catatan Mode 1 - Direct Ship-to-Home: Transisi berjalan sederhana dari `PAID_AWAITING_VALIDATION` -> `PROCESSING_SHIPMENT` -> `IN_TRANSIT_TO_CUSTOMER` -> `DELIVERED_TO_CUSTOMER` -> Auto-Complete 2x24 jam -> `COMPLETED_AND_SETTLED` tanpa menerbitkan garansi fungsi produk).*

---

## 8. Protokol 16 Skenario Operasional & Edge-Case Enterprise

Bagian ini mendefinisikan aturan sistem dan penyelesaian keuangan untuk seluruh skenario operasional nyata di lapangan. **Tidak boleh ada transaksi yang menggantung (*dead-end state*) di luar protokol ini.**

### Skenario 1: Aturan Keranjang Belanja (*Cart Scoping*) & Kapabilitas Bengkel Multi-Item
* **Aturan Pembatasan Checkout:** Konsumen dapat membeli **banyak SKU produk sekaligus dalam satu transaksi (Multi-Item Order) selama berasal dari 1 Official Brand yang sama** dan dipasang di **1 Bengkel Rekanan yang sama**. Jika keranjang berisi produk dari beberapa *Brand* berbeda, sistem mengelompokkan *checkout* menjadi nomor transaksi terpisah per *Brand*.
* **Filter Irisan Kapabilitas Bengkel (*Capability Intersection*):** Karena seluruh produk sudah termasuk gratis jasa pasang dan gratis ongkir ke bengkel (Mode 2 & Mode 3), sistem cukup memfilter dan hanya menampilkan Cabang Bengkel Rekanan yang memiliki kapabilitas aktif (`workshop_branch_capabilities`) untuk **seluruh `service_type_code`** dari SKU yang ada di dalam keranjang tersebut.

### Skenario 2: SOP Verifikasi Penerimaan Barang di Bengkel (*Inbound Chain of Custody*)
* Pada **Mode 3 (Ship-to-Workshop)**, saat kurir menyerahkan paket ke Bengkel Rekanan, Admin Bengkel wajib memeriksa keutuhan segel kardus luar dan mengunggah foto penerimaan dalam waktu maksimal **1x24 jam**.
* Jika kemasan luar hancur/bocor saat diserahkan kurir, bengkel memilih `INBOUND_DAMAGE_REPORTED` agar diklaim ke asuransi pengiriman. Jika bengkel telah menekan konfirmasi `READY_AT_WORKSHOP_FOR_INSTALL`, maka tanggung jawab atas kehilangan barang di gudang bengkel sepenuhnya berada pada pihak Bengkel.

### Skenario 3: Kebijakan Pembatalan Bertingkat, Batas Reschedule & *14-Day Storage Expiry*
* **Pembatalan Sebelum Dikirim (`PAID_AWAITING_VALIDATION` / `PROCESSING_SHIPMENT`):** Konsumen berhak membatalkan dengan **Refund 100%**.
* **Pembatalan Saat di Kurir (`IN_TRANSIT_TO_WORKSHOP`):** Tombol pembatalan dikunci hingga barang tiba di bengkel.
* **Pembatalan Setelah Tiba di Bengkel (`READY_AT_WORKSHOP_FOR_INSTALL`):** Hanya dapat diajukan dengan persetujuan retur *Brand*. Dana pengembalian konsumen dipotong: **Ongkir Pergi (yang telah dikeluarkan Brand) + Ongkir Retur ke Brand + Biaya Penanganan (*Handling Fee*) 5%**.
* **Aturan Reschedule:** Konsumen maksimal melakukan perubahan jadwal sebanyak **2 kali (`max_reschedule_count = 2`)** dan tanggal baru tidak boleh melewati batas **14 hari kalender** sejak barang diterima di bengkel.
* **Eksekusi Otomatis Hari ke-15 (`STORAGE_EXPIRED_FORFEITED` / *No-Show*):**
  1. Sistem mengirim pengingat WhatsApp otomatis pada **Hari ke-3, Hari ke-7, dan Hari ke-12**.
  2. Jika pada **Hari ke-15 (pukul 00:01 waktu lokal bengkel)** konsumen belum melakukan *check-in*, status otomatis berubah menjadi `STORAGE_EXPIRED_FORFEITED`.
  3. **Penyelesaian Keuangan & Barang Otomatis:**
     * **Official Brand** menerima pencairan **100% dana bersih produk ($N_{\text{brand}}$)** karena barang telah sah terjual dan dikirim ke bengkel.
     * Konsumen diberikan waktu 7 hari kalender untuk mengambil sendiri fisik barangnya di bengkel (tanpa pasang) ATAU meminta dikirim ke rumah atas biaya ongkos kirim konsumen sendiri.
     * **Status Garansi Produk:** Ditetapkan sebagai **`VOID_DUE_TO_NO_SHOW`**.

### Skenario 4: Batal Pasang karena Kondisi Mobil Konsumen Rusak/Tidak Aman (`ABORTED_BY_VEHICLE_CONDITION`)
* **Kondisi Pemicu:** Barang yang dikirim sudah benar dan sesuai pesanan, namun saat mobil diangkat ke *lift* bengkel, mekanik menemukan kerusakan struktural pada mobil konsumen (misal: dudukan sasis keropos parah, baut patah di dalam blok dari bengkel sebelumnya, atau modifikasi las ekstrem) sehingga pemasangan tidak aman dilanjutkan.
* **Validasi Anti-Kecurangan Bengkel:**
  1. Bengkel wajib mengunggah **minimal 2 foto bukti kerusakan pada mobil konsumen** beserta catatan teknis mekanik.
  2. Pemilik akun konsumen wajib menekan tombol **"Setuju Batalkan Pemasangan & Bawa Pulang Barang"** di aplikasi (jika konsumen menolak, tiket dieskalasi ke Mediator Teknis NewTKK).
* **Penyelesaian Keuangan & Barang:**
  1. Fisik produk diserahkan kepada konsumen di bengkel untuk dibawa pulang.
  2. **Official Brand** tetap menerima **100% pencairan dana bersih produk ($N_{\text{brand}}$)** karena produk *Brand* tidak bermasalah. Garansi produk ditangguhkan sementara dan dapat diaktifkan apabila konsumen menjadwalkan pemasangan kembali di bengkel rekanan setelah kondisi mobilnya diperbaiki.

### Skenario 5: Produk Cacat Pabrik Saat Dibuka oleh Mekanik (`REPORT_DOA_DEFECT`)
* **Kondisi Pemicu:** Kardus luar tampak mulus saat *inbound check*, namun ketika dibuka oleh mekanik di hadapan konsumen pada Hari-H, komponen di dalamnya terbukti cacat produksi (*Dead on Arrival / DOA*, misal: *shockbreaker* bocor oli di dalam dus, lampu mati, atau isi kiri-kiri).
* **Protokol Resolusi:**
  1. Mekanik memilih status `REPORT_DOA_DEFECT` pada unit fisik terkait dan mengunggah video tanpa jeda yang memperlihatkan nomor seri produk dan letak cacatnya.
  2. **Official Brand** wajib mengirimkan unit pengganti ke bengkel menggunakan layanan pengiriman tercepat (*Instant / Same-Day / Next-Day Express*) dengan **100% biaya ongkos kirim pergi-pulang ditanggung oleh Brand**.
  3. Apabila mekanik sudah terlanjur membongkar mobil konsumen dan terpaksa memasang kembali komponen lama agar mobil konsumen bisa dipakai pulang sambil menunggu barang pengganti, *Official Brand* menanggung kompensasi bongkar-pasang tambahan kepada Bengkel sesuai PKS B2B mereka.

### Skenario 6: Kegagalan Operasional Bengkel pada Hari-H (`WORKSHOP_UNABLE_TO_SERVE`)
* **Kondisi Pemicu:** Barang sudah di bengkel dan konsumen siap datang, tetapi bengkel mendadak tidak dapat beroperasi (misal: mati listrik total, *hydraulic lift* rusak, banjir, atau mekanik utama sakit).
* **Protokol Resolusi:** Konsumen diberikan 2 pilihan di aplikasi:
  1. **Reschedule Prioritas:** Menggeser jadwal ke hari berikutnya di bengkel yang sama tanpa mengurangi kuota *reschedule* konsumen, ditambah kompensasi *Voucher Diskon* dari platform.
  2. **Reroute Instan ke Bengkel Rekanan Terdekat:** Tim Operasional NewTKK mengatur pengiriman instan (*GoSend/GrabExpress/Lalamove*) untuk memindahkan fisik barang dari Bengkel A ke Bengkel Rekanan B terdekat yang berafiliasi dengan *Brand* tersebut.

### Skenario 7: Selisih Stok Fisik Bengkel pada Mode 2 (`WORKSHOP_STOCK_DISCREPANCY`)
* **Kondisi Pemicu:** Konsumen memesan dengan **Mode 2 (Stok Ready di Bengkel)**, namun saat admin bengkel hendak menyiapkan barang, stok fisik di bengkel ternyata kosong karena baru saja terjual ke pelanggan *walk-in*.
* **Protokol Fallback Otomatis:** Pesanan tidak langsung dibatalkan. Sistem menawarkan:
  1. Mengalihkan lokasi pemasangan ke Bengkel Rekanan lain dalam radius terdekat yang memiliki stok *ready*, **ATAU**
  2. **Konversi Otomatis ke Mode 3 (*Ship-to-Workshop*):** Barang langsung dikirim dari Gudang Pusat *Official Brand* ke bengkel tersebut dengan penyesuaian tanggal pasang (tetap **Rp 0 ongkir bagi konsumen**).

### Skenario 8: Batas Bahan Bantu Bengkel (*All-In Standard Workshop Consumables*)
* **Aturan Anti-Biaya Tersembunyi (*No Hidden Fees*):** Fasilitas gratis jasa pemasangan yang tercantum di NewTKK sudah bersifat **All-In untuk bahan bantu kerja standar (*standard shop supplies*)**, seperti: kabel ties, isolasi listrik standar, gemuk (*grease*), cairan pembersih rem standar pada area kerja, dan baut/mur standar bawaan produk.
* Setiap halaman detail produk dan tiket booking mencantumkan daftar eksplisit: **"Cakupan Pemasangan Gratis (Termasuk)"** dan **"Layanan Tambahan Opsional di Luar Paket (Tidak Termasuk, misal: Spooring 3D / Kuras Total Jalur Rem)"** agar tidak terjadi perselisihan di kasir bengkel.

### Skenario 9: Penyelesaian & Pencairan Parsial pada Pesanan Multi-Item/Multi-Unit (`PARTIAL_FULFILLMENT_AND_SETTLEMENT`)
* **Kondisi Pemicu:** Dalam 1 pesanan berisi beberapa item/unit fisik, sebagian unit berhasil dipasang sempurna sementara 1 unit batal dipasang (karena cacat DOA yang stok penggantinya kosong).
* **Arsitektur Penyelesaian Per Unit Fisik (`order_item_units` Granularity):**
  * Setiap baris unit fisik memiliki status mandiri (`INSTALLED`, `CANCELLED_REFUNDED`).
  * Pesanan induk ditutup dengan status **`PARTIALLY_COMPLETED`**.
  * Sistem *Escrow Ledger* menghitung ulang secara proporsional: mencairkan dana produk untuk unit yang terpasang ke *Official Brand*, menerbitkan *Digital Warranty Passport* khusus untuk unit yang terpasang, dan melakukan *Partial Refund* otomatis senilai harga bersih unit yang batal ke konsumen.

### Skenario 10: Delegasi Kedatangan Kendaraan (*Driver Check-In Pass & Remote Owner Approval*)
* **Kondisi Pemicu:** Pemilik akun membeli melalui aplikasi, namun menugaskan supir pribadi atau anggota keluarga untuk membawa mobil ke bengkel rekanan pada Hari-H.
* **Protokol Sistem:**
  1. **Shareable Check-In Pass:** Pemilik akun dapat menekan tombol **"Bagikan Tiket Kedatangan (QR & PIN)"** ke WhatsApp supir. Tiket ini hanya menampilkan lokasi bengkel, jadwal, daftar pekerjaan, dan QR Code Check-in (tanpa menampilkan harga atau akses ke akun utama).
  2. **Remote Handover Approval:** Saat mekanik selesai memasang produk dan mengunggah foto QC (foto produk terpasang + foto plat nomor + status komponen lama), notifikasi langsung muncul di smartphone **Pemilik Akun**. Pemilik akun dapat memverifikasi foto tersebut dari jarak jauh dan menekan **"Konfirmasi Pekerjaan Selesai"** (atau memberikan 6-digit *Completion PIN* kepada supir untuk diinput di tablet bengkel).

### Skenario 11: Insiden Kerusakan Kendaraan Akibat Kelalaian Mekanik (`INCIDENT_REPORTED_DURING_INSTALL`)
* **Kondisi Pemicu:** Selama proses pengerjaan di bengkel, terjadi kerusakan pada bagian mobil konsumen yang sebelumnya sehat (dibuktikan dengan perbandingan antara **Foto Kondisi Awal saat Check-in** vs. **Foto Kondisi Akhir**, misal: panel *dashboard* pecah terkena congkelan atau kabel sensor putus).
* **Protokol Mitigasi & Mediasi:**
  1. Konsumen menekan tombol **"Laporkan Insiden Pengerjaan"** sebelum menekan konfirmasi selesai (`INCIDENT_UNDER_MEDIATION`).
  2. **Hak Official Brand Dilindungi:** Karena produk milik *Brand* tidak bermasalah dan telah terpasang/diserahkan, pencairan dana bersih produk ke *Official Brand* tetap dapat diproses setelah diverifikasi Admin NewTKK, sementara Bengkel Rekanan wajib menyelesaikan tanggung jawab perbaikan kepada konsumen di bawah pengawasan Mediator NewTKK & *Brand* sesuai klausul PKS *Workshop Liability*.

### Skenario 12: Dukungan Produk Pre-Order / Fabrikasi Custom (`MADE_TO_ORDER_LEAD_TIME`)
* Untuk produk yang dibuat berdasarkan pesanan (seperti *Karpet Mobil Custom, Sarung Jok Paten, Downpipe/Exhaust Custom, atau Velg Forged*), *Official Brand* mengaktifkan atribut `availability_type = PRE_ORDER_CUSTOM` beserta `production_lead_time_days` (misal: `5` hari kerja).
* Sistem secara otomatis menambahkan durasi fabrikasi tersebut ke dalam SLA batas pengiriman *Brand* dan menggeser tanggal paling awal yang dapat dipilih konsumen pada kalender *booking* bengkel.

### Skenario 13: Beban Ongkos Kirim Mode 3 (Ditanggung Brand) & Guard Selisih Berat Volumetrik (`VOLUMETRIC_SHIPPING_DISCREPANCY_GUARD`)
* *Official Brand* diwajibkan mengisi berat aktual (gram) dan dimensi **setelah dikemas (*packed dimensions*: $P \times L \times T$ dalam cm)** pada setiap SKU.
* Sistem menghitung ongkos kirim berdasarkan nilai tertinggi antara Berat Aktual vs. Berat Volumetrik:
  $$W_{\text{volumetric (kg)}} = \frac{P \times L \times T}{D_{\text{divisor}}}$$
  *(Di mana $D_{\text{divisor}} = 6000$ untuk kurir reguler dan $4000$ untuk ekspedisi kargo/truk).*
* **Aturan Pemotongan Ongkir dari Pencairan Brand (`brand_borne_shipping_fee_idr`):**
  1. **Pada Mode 3 (Ship-to-Workshop):** Seluruh tagihan aktual kurir dari Gudang Brand menuju Bengkel Rekanan ($L_{\text{mode3\_actual}}$) ditanggung oleh *Official Brand* dan dipotong otomatis dari pencairan *Net Settlement* Brand ($N_{\text{brand}}$).
  2. **Pada Mode 1 (Ship-to-Home):** Ongkir estimasi awal ($L_{\text{mode1\_est}}$) dibayar oleh konsumen. Namun apabila pihak ekspedisi/Biteship menagih biaya ongkir aktual yang lebih tinggi ($L_{\text{mode1\_actual}} > L_{\text{mode1\_est}}$) akibat *Brand* salah menginput dimensi kemasan, maka selisih kurang bayar ($\Delta L_{\text{undercharge}}$) tersebut **dipotong otomatis dari pencairan saldo Official Brand**.

### Skenario 14: Penanganan Komponen Lama Konsumen (*Old Parts Handover Checklist*)
* Untuk mencegah tuduhan bahwa bengkel mengambil komponen bawaan mobil konsumen yang dilepas, formulir penyelesaian QC mekanik memiliki *radio button* wajib:
  * `RETURNED_TO_TRUNK`: Komponen lama telah dikemas dan dimasukkan ke bagasi mobil konsumen (disertai 1 foto opsional di bagasi).
  * `DISPOSED_WITH_CONSENT`: Komponen lama (seperti oli bekas, filter kotor, atau aki soak) ditinggal/dibuang oleh bengkel atas persetujuan konsumen.

### Skenario 15: Proteksi Anti-Bypass (*Off-Platform Transaction Prevention*)
* Sebelum pembayaran berstatus `PAID`, halaman pemilihan bengkel menampilkan **Nama Bengkel, Lencana Verifikasi, Rating & Ulasan, Foto Fasilitas, Nama Kecamatan/Area, dan Jarak (km)**, namun menyembunyikan nomor telepon langsung dan alamat detail nomor jalan.
* Alamat lengkap navigasi peta, nomor kontak darurat bengkel, dan **QR Code Booking** baru diterbitkan setelah pembayaran terverifikasi. Selain itu, **Digital Warranty Passport** hanya dapat diterbitkan melalui transaksi yang tercatat di sistem NewTKK.

### Skenario 16: Komunikasi Hybrid & *WhatsApp Signed Magic Links*
* Mengingat mekanik dan admin bengkel bekerja secara aktif di lapangan, setiap perubahan status kritis (seperti *Booking Masuk*, *Paket Tiba*, atau *Pengingat Jadwal H-1*) dikirimkan melalui **WhatsApp Business Cloud API** yang menyertakan tombol **Signed Magic Link** (tautan terenkripsi dengan token berbatas waktu yang langsung membuka halaman aksi spesifik di *Workshop Mobile Web Portal* tanpa mengharuskan admin bengkel mengetik ulang *username/password*).

---

## 9. Sistem Garansi Digital Satu Pintu (*Digital Warranty Passport & Single-Window Claim*)

### 9.1. Anatomi *Digital Warranty Passport*
Setiap unit fisik produk pada transaksi Mode 2 dan Mode 3 yang mencapai status `COMPLETED_AND_SETTLED` menerbitkan entitas **Digital Warranty Passport** dengan dua cakupan perlindungan terpadu:

1. **Garansi Pemasangan (*Installation Workmanship Warranty* — Tanggung Jawab Bengkel):**
   * Berlaku minimal **30 Hari Kalender** sejak tanggal penyelesaian instalasi.
   * Melindungi konsumen dari kendala teknis akibat proses pemasangan (baut kendor, bunyi tidak wajar pasca-instalasi, kebocoran paking/sil instalasi, atau kesalahan penyetelan).
2. **Garansi Produk Resmi (*Official Product Warranty* — Tanggung Jawab Brand):**
   * Terbagi menjadi 2 tipe sesuai kategori produk:
     * **`FUNCTIONAL_PERIODIC` (Misal: 6, 12, 24, atau 36 Bulan):** Menjamin perbaikan atau penggantian unit baru jika produk mengalami kegagalan fungsi atau cacat pabrik selama masa garansi.
     * **`AUTHENTICITY_AND_DOA` (Untuk Kategori Consumables seperti Oli/Filter/Kampas Rem):** Menjamin 100% keaslian produk resmi *Official Brand* serta bebas cacat awal.

### 9.2. Protokol Klaim Garansi Satu Pintu (*Anti-Pingpong 3-Way Resolution*)
Konsumen **dilarang dipingpong** antara *Brand* dan Bengkel. Seluruh klaim berjalan dalam satu alur tiket terpusat:

```text
[1. CLAIM_SUBMITTED] (Konsumen klik "Ajukan Klaim" di Garansi Digital + Upload Video/Foto + Pilih Jadwal Cek)
         │
         ▼
[2. INSPECTION_SCHEDULED] (Konsumen datang ke Bengkel Pemasang / Bengkel Rekanan Terdekat)
         │
         ▼
[3. UNDER_WORKSHOP_DIAGNOSIS] (Mekanik periksa fisik mobil, cek keutuhan segel & kesesuaian Plat Nomor)
         │
         ├──► Hasil A: Kendala Instalasi / Setelan ──► [4A. RESOLVED_BY_WORKSHOP_REWORK] (Diperbaiki Gratis Hari Itu)
         │
         ├──► Hasil B: Kerusakan Akibat Tabrakan / Modifikasi Luar ──► [4B. CLAIM_REJECTED_WITH_PROOF]
         │
         └──► Hasil C: Indikasi Cacat Produk Pabrik
                   │
                   ▼
         [4C. INTERNAL_3WAY_TRIAGE] (Ruang Diskusi Privat: Brand + Bengkel + Mediator Teknis NewTKK)
                   │ (SLA Keputusan Brand: 2x24 Jam Kerja. Jika buntu, Mediator NewTKK ketok palu)
                   ▼
         [5. REPLACEMENT_DISPATCHED] (Brand kirim unit pengganti ke Bengkel)
                   │
                   ▼
         [6. CLAIM_REPLACEMENT_INSTALLED] (Dipasang ulang di Bengkel ──► Rp 0 Biaya bagi Konsumen)
```

### 9.3. Fitur Pindah Tangan Kepemilikan Kendaraan (*Warranty Ownership Transfer*)
Untuk meningkatkan retensi ekosistem dan akuisisi pengguna baru secara organik (*viral loop*):
* Apabila konsumen menjual mobilnya kepada pihak lain saat masa **Garansi Produk (`FUNCTIONAL_PERIODIC`)** masih aktif, pemilik lama dapat menekan tombol **"Transfer Kepemilikan Garansi"** (maksimal **1 kali transfer** per sertifikat garansi).
* Pemilik lama memasukkan alamat email dan nomor WhatsApp pembeli mobil baru.
* Pembeli mobil baru menerima undangan untuk membuat akun di **NewTKK** dan mengklaim sertifikat garansi tersebut. Selama **Nomor Polisi / Nomor Rangka (VIN)** kendaraan tetap sama, sisa masa garansi tetap aktif sepenuhnya di akun pemilik baru.

---

## 10. Arsitektur Keuangan: Midtrans, Double-Entry Escrow Ledger & Perpajakan

### 10.1. Alur Dana (*Pay-In, Escrow Holding, Settlement & Iris Payout*)
1. **Pay-In (Midtrans Snap / Core API):** Konsumen membayar total tagihan ($T_{\text{customer\_pay}}$) menggunakan Virtual Account, QRIS, E-Wallet, atau Kartu Kredit/Cicilan. Dana masuk ke rekening penampungan Midtrans milik NewTKK.
2. **Escrow Holding (Internal Double-Entry Ledger):** Begitu *webhook* `settlement` / `capture` diterima dari Midtrans, sistem mencatat jurnal akuntansi berpasangan yang menahan dana di akun `SYS_ESCROW_HOLDING`.
3. **Settlement (Saat Status `COMPLETED_AND_SETTLED`):** Sistem memecah dana dari `SYS_ESCROW_HOLDING`:
   * Saldo bersih produk ($N_{\text{brand}}$) masuk ke **`BRAND_AVAILABLE_BALANCE`**.
   * Beban logistik pengiriman ($L_{\text{cust}} + L_{\text{brand\_ship}}$) dialokasikan ke **`SYS_LOGISTICS_PAYABLE`**.
   * Pendapatan komisi & *platform fee* ($N_{\text{platform}}$) masuk ke **`SYS_REVENUE_COMMISSION`** dan **`SYS_REVENUE_SERVICE_FEE`**.
4. **Disbursement / Penarikan Dana (Midtrans Iris API):**
   * *Official Brand* dapat menarik saldo `AVAILABLE_BALANCE` mereka ke rekening bank perusahaan kapan saja (dengan batas minimum penarikan Rp100.000) melalui **Midtrans Iris Payout API**.

### 10.2. Rumus Matematika Pembagian Dana (*True All-In Settlement Engine*)
Seluruh perhitungan keuangan di dalam database dan kode program **wajib menggunakan tipe data bilangan bulat (`BIGINT` dalam satuan Rupiah penuh)** dan dilarang menggunakan tipe `FLOAT`/`DOUBLE`.

#### Definisi Variabel Transaksi:
* $P_{\text{gross}}$ = Total Harga Katalog Produk (*All-In Price* sudah termasuk fasilitas gratis pasang)
* $D_{\text{brand}}$ = Diskon / Voucher yang ditanggung oleh *Official Brand*
* $P_{\text{net}} = P_{\text{gross}} - D_{\text{brand}}$ = Harga Produk Bersih yang menjadi dasar perhitungan komisi
* $L_{\text{cust}}$ = Biaya Ongkos Kirim yang dibayar oleh Konsumen saat *Checkout*:
  * Pada **Mode 2 & Mode 3 (Pasang di Bengkel Rekanan):** $L_{\text{cust}} = \mathbf{0}$ (**Gratis Ongkir ke Bengkel**).
  * Pada **Mode 1 (Beli Produk Saja / Kirim ke Rumah):** $L_{\text{cust}} = L_{\text{mode1\_est}}$ (Dibayar konsumen).
* $L_{\text{brand\_ship}}$ = Beban Ongkos Kirim yang ditanggung oleh *Official Brand*:
  * Pada **Mode 2 (Stok Ready di Bengkel):** $L_{\text{brand\_ship}} = \mathbf{0}$.
  * Pada **Mode 3 (Ship-to-Workshop):** $L_{\text{brand\_ship}} = L_{\text{mode3\_actual}}$ (**100% tagihan kurir dari Gudang Brand ke Bengkel Rekanan ditanggung oleh Brand**).
  * Pada **Mode 1 (Direct Ship-to-Home):** $L_{\text{brand\_ship}} = \max(0, L_{\text{mode1\_actual}} - L_{\text{mode1\_est}})$ (Hanya jika timbul kurang bayar akibat kesalahan input dimensi oleh Brand).
* $F_{\text{platform}}$ = Biaya Layanan Aplikasi (*Platform Service Fee* yang dibebankan ke konsumen, misal Rp5.000)
* $D_{\text{platform}}$ = Diskon / Subsidi Promo yang ditanggung oleh kas pemasaran NewTKK
* $R_{\text{brand}}$ = Persentase Komisi Platform atas Produk (misal `0.08` atau 8%, dapat diatur per *Brand* / Kategori)

#### Rumus Perhitungan:
1. **Total Tagihan yang Dibayar Konsumen ($T_{\text{customer\_pay}}$):**
   $$T_{\text{customer\_pay}} = P_{\text{net}} + L_{\text{cust}} + F_{\text{platform}} - D_{\text{platform}}$$

2. **Komisi Platform dari Produk ($C_{\text{brand}}$):**
   $$C_{\text{brand}} = \text{round}(P_{\text{net}} \times R_{\text{brand}})$$

3. **Pencairan Bersih ke Saldo Official Brand ($N_{\text{brand}}$):**
   $$N_{\text{brand}} = P_{\text{net}} - C_{\text{brand}} - L_{\text{brand\_ship}}$$

4. **Pendapatan Bersih Platform NewTKK ($N_{\text{platform}}$):**
   $$N_{\text{platform}} = C_{\text{brand}} + F_{\text{platform}} - D_{\text{platform}}$$

### 10.3. Contoh Simulasi Transaksi Nyata (Mode 3: Ship-to-Workshop)
> **Kasus:** Konsumen membeli 1 set *Shockbreaker* dengan harga *All-In* **Rp 4.000.000**, memilih pasang di Bengkel Rekanan BSD (Mode 3: barang dikirim dari Gudang Brand di Jakarta ke Bengkel BSD dengan ongkos kirim kurir sebesar **Rp 80.000** yang ditanggung Brand), dan *Platform Fee* **Rp 5.000**. Komisi platform $R_{\text{brand}} = 10\%$ (`Rp 400.000`).

| Aliran Dana | Perhitungan | Nominal (Rp) | Keterangan Waktu Pelepasan Dana |
| :--- | :--- | ---: | :--- |
| **Total Bayar Konsumen ($T_{\text{customer\_pay}}$)** | $4.000.000 + 0 \text{ (Jasa)} + 0 \text{ (Ongkir)} + 5.000$ | **Rp 4.005.000** | Masuk ke *Escrow* saat status `PAID`. |
| **Alokasi Biaya Kurir Mode 3 ($L_{\text{brand\_ship}}$)** | Ditanggung Brand & disalurkan ke Biteship | **Rp 80.000** | Dialokasikan ke `SYS_LOGISTICS_PAYABLE`. |
| **Pencairan Bersih ke Official Brand ($N_{\text{brand}}$)** | $4.000.000 - 400.000 \text{ (Komisi)} - 80.000 \text{ (Ongkir)}$ | **Rp 3.520.000** | Cair ke *Available Balance* Brand saat `COMPLETED_AND_SETTLED`. |
| **Pendapatan Bersih NewTKK ($N_{\text{platform}}$)** | $400.000 + 5.000$ | **Rp 405.000** | Diakui sebagai pendapatan platform saat `COMPLETED_AND_SETTLED`. |

### 10.4. Kepatuhan Faktur & Perpajakan (Struktur PPMSE / Marketplace)
Agar perusahaan NewTKK tidak dikenakan pajak atas omzet penjualan barang milik *Official Brand*, sistem menerbitkan **Dokumen Nota Gabungan (`Combined Invoice`)** yang secara eksplisit memisahkan:
1. **Bukti Penjualan Barang (All-In):** Diterbitkan atas nama *Official Brand Partner* kepada Konsumen senilai $P_{\text{net}}$.
2. **Bukti Biaya Layanan Platform (& Ongkir Mode 1 jika ada):** Diterbitkan atas nama *NewTKK* kepada Konsumen senilai $F_{\text{platform}}$ (serta Bukti Potong Komisi & Reimburse Logistik dari NewTKK kepada *Official Brand* senilai $C_{\text{brand}}$ dan $L_{\text{brand\_ship}}$).

---

## 11. Tabel Konfigurasi Sistem Dinamis (`platform_system_configs`)

Untuk memastikan parameter operasional dapat disesuaikan sewaktu-waktu oleh Super Admin melalui *Admin Backoffice* tanpa mengubah kode program (*zero-redeployment policy adjustment*), *AI Coding Agent* wajib menyimpan parameter berikut di dalam tabel database `platform_system_configs`:

| `config_key` | Tipe Data | Nilai Default (MVP) | Deskripsi & Penggunaan di Sistem |
| :--- | :--- | :--- | :--- |
| `PAYMENT_EXPIRY_MINUTES` | `INTEGER` | `60` | Batas waktu tunggu pembayaran Midtrans sebelum pesanan menjadi `PAYMENT_EXPIRED`. |
| `LAST_SLOT_PAYMENT_EXPIRY_MINUTES` | `INTEGER` | `15` | Batas waktu tunggu pembayaran dipercepat jika kuota sesi/stok tinggal 1 unit terakhir. |
| `SLA_BRAND_FITMENT_CHECK_HOURS` | `INTEGER` | `4` | Batas jam kerja bagi Official Brand untuk mengonfirmasi kecocokan mobil konsumen. |
| `SLA_WORKSHOP_SCHEDULE_CONFIRM_HOURS` | `INTEGER` | `8` | Batas jam kerja bagi Bengkel Rekanan untuk menerima/mengajukan jadwal ulang. |
| `SLA_INBOUND_PACKAGE_CHECK_HOURS` | `INTEGER` | `24` | Batas waktu bagi Bengkel untuk melaporkan kerusakan paket kiriman dari kurir. |
| `BOOKING_CALENDAR_BUFFER_DAYS` | `INTEGER` | `1` | Tambahan hari pengaman pada kalkulasi tanggal minimal kalender Mode 3. |
| `MAX_CUSTOMER_RESCHEDULE_COUNT` | `INTEGER` | `2` | Batas maksimal konsumen diizinkan mengubah jadwal kedatangan ke bengkel. |
| `WORKSHOP_STORAGE_MAX_DAYS` | `INTEGER` | `14` | Batas hari kalender penyimpanan barang di bengkel sebelum terkena `STORAGE_EXPIRED_FORFEITED`. |
| `CANCELLATION_HANDLING_FEE_RATE` | `DECIMAL` | `0.05` | Persentase biaya administrasi pembatalan setelah barang tiba di bengkel (5%). |
| `HANDOVER_AUTO_COMPLETE_HOURS` | `INTEGER` | `48` | Waktu tunggu otomatis (2x24 jam) setelah QC diunggah sebelum pesanan menjadi `COMPLETED_AND_SETTLED`. |
| `DEFAULT_INSTALLATION_WARRANTY_DAYS` | `INTEGER` | `30` | Masa berlaku minimal Garansi Pemasangan yang ditanggung oleh Bengkel Rekanan. |
| `MAX_WARRANTY_OWNERSHIP_TRANSFERS` | `INTEGER` | `1` | Batas berapa kali sertifikat garansi digital dapat dipindahtangankan ke pemilik mobil baru. |
| `DEFAULT_PLATFORM_SERVICE_FEE_IDR` | `BIGINT` | `5000` | Biaya layanan aplikasi per transaksi yang dibebankan kepada konsumen (Rp5.000). |
| `DEFAULT_BRAND_COMMISSION_RATE` | `DECIMAL` | `0.08` | Persentase komisi standar untuk Official Brand (8%) jika tidak ada tarif khusus per Brand/Kategori. |
| `MINIMUM_PAYOUT_AMOUNT_IDR` | `BIGINT` | `100000` | Saldo minimum *Available Balance* yang dapat ditarik Official Brand melalui Midtrans Iris (Rp100.000). |
