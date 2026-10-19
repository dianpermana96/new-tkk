# 📓 DOKUMEN 5: UI/UX INFORMATION ARCHITECTURE, COMPONENT WIREFRAME SPECS, CONVERSION SAFEGUARDS & PHASE 2/3 ROADMAP

* **Nama Projek:** NewTKK
* **Kode Dokumen:** `DOC-05-UI-UX-AND-ROADMAP`
* **Lokasi File:** `docs/05-UI-UX-SPECS-AND-PHASE2-ROADMAP.md`
* **Package Monorepo Target:** `packages/ui` (`@newtkk/ui`) & `apps/web-*`
* **Versi:** `2.1.0-FINAL` *(True All-In Bundled Pricing + Free Ship-to-Workshop by Brand + Zero Vehicle Tiers)*
* **Tanggal Efektif:** 20 Oktober 2026
* **Referensi Induk:**
  * `docs/01-MASTER-PRD-NEWTKK.md`
  * `docs/02-FUNCTIONAL-SPECS-4-PORTALS.md`
  * `docs/03-DATABASE-ARCHITECTURE-AND-LEDGER-ERD.md`
  * `docs/04-SYSTEM-DESIGN-API-AND-INTEGRATIONS.md`
* **Target Pembaca:** UI/UX Designer, Frontend/Mobile Engineers, Product Manager, QA Engineers, & AI Coding Agents (Claude Code, Cursor, Antigravity, v0, OpenCode)

---

## 1. Prinsip Desain & Design System Tokens (`@newtkk/ui` — Tailwind CSS v4 + shadcn/ui)

Paket `@newtkk/ui` menerapkan **Adaptive Ergonomic Design System** yang disesuaikan dengan kondisi lingkungan fisik masing-masing pengguna di keempat portal:

### 1.1. Matriks Ergonomi & Tema Visual per Portal

| Portal Aplikasi | Karakteristik Lingkungan Pengguna | Standar Ergonomi & Ukuran Komponen (`Tailwind v4 Tokens`) |
| :--- | :--- | :--- |
| **Portal 1: Customer Storefront** (`apps/web-customer`) | Digunakan satu tangan di smartphone saat mencari suku cadang, mengecek kecocokan mobil, & memilih bengkel. | • **Trust & Zero-Friction Visuals:** Dominasi warna *Clean Slate/White* (`#F8FAFC`), *Automotive Navy* (`#0F172A`), *Verified Emerald* (`#10B981`), dan *Action Orange* (`#EA580C`).<br>• **Prominent All-In Badges:** Setiap kartu produk dan PDP menonjolkan lencana hijau: `✨ Gratis Pasang + Gratis Ongkir ke Bengkel`.<br>• **Zero-Truncation Category Grid:** Wajib merender `ui_short_label` (maks 14 karakter) pada grid ikon 2x5 dengan ukuran `text-xs font-semibold leading-tight`. |
| **Portal 2: Official Brand Portal** (`apps/web-brand`) | Digunakan di layar Desktop/Laptop untuk memproses banyak pesanan, mencetak label pengiriman Mode 3, & mengelola ratusan SKU. | • **High-Density Data Tables:** Tabel dengan *sticky header*, *keyboard shortcuts* untuk 1-Klik Validasi Fitment, dan tampilan *Side-by-Side Split View* tanpa perlu berpindah halaman. |
| **Portal 3: Workshop Partner Portal** (`apps/web-workshop`) | Digunakan di lantai bengkel (tangan berminyak, terkena sinar matahari langsung di *stall*, atau di bawah kolong mobil). | • **High-Contrast Daylight Mode:** Kontras rasio minimal **7:1 (WCAG AAA)**.<br>• **Glove/Grease-Friendly Touch Targets:** Seluruh tombol aksi utama mekanik (*Scan QR*, *Ambil Foto QC*, *Pilih Checklist Part Lama*) wajib memiliki tinggi minimal **`h-14` (56px)** dengan teks `text-base font-bold`.<br>• **Pure Operational Wizard:** Bebas dari menu keuangan/harga yang membingungkan mekanik; fokus murni pada 3 langkah eksekusi kerja. |
| **Portal 4: Super Admin Backoffice** (`apps/web-admin`) | Digunakan di layar lebar untuk audit buku besar keuangan, kurasi MMY, & mediasi klaim 3 arah. | • **Audit-Grade Clarity:** Angka nominal uang wajib menggunakan font `font-mono tabular-nums` agar digit ribuan/jutaan sejajar vertikal saat rekonsiliasi Debit vs Kredit. |

### 1.2. Sistem Lencana Kepercayaan & Nilai All-In (*Semantic Badges*)
1. 🟢 **`ALL_IN_INCLUDED` (`bg-emerald-50 text-emerald-800 border-emerald-300`):** *"Gratis Jasa Pasang & Gratis Ongkir ke Bengkel Rekanan"* atau *"Cocok untuk [Mobil Anda]"*.
2. 🔵 **`UNIVERSAL_OR_SPEC` (`bg-sky-50 text-sky-800 border-sky-300`):** *"Universal / Sesuai Spesifikasi Ukuran"*.
3. 🟠 **`ACTION_OR_TRADE_OFF` (`bg-amber-50 text-amber-900 border-amber-400`):** Peringatan edukatif *Warranty Void Waiver* (Mode 1) atau status menunggu konfirmasi jadwal.
4. 🔴 **`CRITICAL_OR_VOID` (`bg-rose-50 text-rose-900 border-rose-300`):** *"Tidak Cocok dengan Mobil Anda"*, *"Garansi Hangus"*, atau *"Insiden / DOA Dilaporkan"*.

---

## 2. Information Architecture & Peta Rute Lengkap Next.js v16.4 (`App Router`)

Seluruh portal wajib mematuhi standar **Next.js v16.4** (menggunakan `src/proxy.ts` sebagai pengganti `middleware.ts`):

### 2.1. Peta Rute Portal 1: `apps/web-customer/src/app`
```text
apps/web-customer/src/
├── proxy.ts                                             # Auth & Route Guard Next.js 16.4 (pengganti middleware.ts)
└── app/
    ├── (public-storefront)/
    │   ├── page.tsx                                     # Homepage: Vehicle Bar, 10 Dual-Label Categories, Featured All-In Products
    │   ├── c/[categorySlug]/page.tsx                    # Halaman Kategori L1/L2/L3 + Dynamic JSONB Faceted Filters
    │   ├── brand/[brandSlug]/page.tsx                   # Official Brand Storefront + Daftar Bengkel Rekanan Brand
    │   ├── p/[productSlug]/page.tsx                     # PDP + Widget Instant O2O Workshop Preview (Gratis Pasang & Ongkir)
    │   └── pass/[shareToken]/page.tsx                   # Public Shareable Driver Check-In Pass (Skenario 10 - Tanpa Login)
    ├── (authenticated-checkout)/
    │   ├── cart/page.tsx                                # Multi-Brand Grouped Cart + Pre-Checkout Workshop Selector
    │   └── checkout/[brandId]/page.tsx                  # Checkout 3-Mode, Vehicle Declaration, Slot Lock 10m, & Waiver
    └── (customer-account)/
        ├── garage/page.tsx                              # My Garage (Daftar Mobil & Riwayat Pemasangan per Plat Nomor)
        ├── orders/
        │   ├── page.tsx                                 # Daftar Pesanan & Status Tracking
        │   └── [orderId]/
        │       ├── page.tsx                             # Detail Pesanan, QR Booking Pass, Reschedule (Maks 2x)
        │       ├── handover-review/page.tsx             # Galeri Foto QC Bengkel -> Tombol Konfirmasi Selesai / Lapor Insiden
        │       └── review/page.tsx                      # Formulir Dual-Rating (Rating Produk Brand vs Rating Bengkel)
        ├── warranties/
        │   ├── page.tsx                                 # Digital Warranty Passport List (Countdown Garansi Produk & Pasang)
        │   └── [warrantyId]/
        │       ├── page.tsx                             # Sertifikat Detail + Tombol Transfer Kepemilikan (Maks 1x)
        │       └── claim/page.tsx                       # Form Pengajuan Klaim Garansi Satu Pintu
        └── refund-wallet/page.tsx                       # Saldo Pengembalian VA/QRIS + Penarikan Instan Midtrans Iris
```

### 2.2. Peta Rute Portal 2: `apps/web-brand/src/app`
```text
apps/web-brand/src/
├── proxy.ts                                             # RBAC Guard (BRAND_OWNER, BRAND_CATALOG_ADMIN, BRAND_ORDER_ADMIN)
└── app/
    ├── (auth)/login/page.tsx
    └── (dashboard)/
        ├── page.tsx                                     # Overview: SLA Timer Alerts, Antrean Validasi Fitment, Omzet Bersih
        ├── fitment-queue/page.tsx                       # Antrean Prioritas 1-Klik Fitment Verification (SLA 4 Jam Kerja)
        ├── orders/
        │   ├── page.tsx                                 # Manajemen Pesanan (Mode 1, Mode 2, Mode 3)
        │   └── [orderId]/dispatch/page.tsx              # Pilih Kurir Biteship (Ongkir Mode 3 Ditanggung Brand) & Cetak Label QR
        ├── catalog/
        │   ├── page.tsx                                 # Daftar SKU (Harga All-In), Stok Multi-Gudang, & Status Pre-Order
        │   ├── new/page.tsx                             # Form SKU + Simulator Margin All-In + Packed Dimensions + Clone Fitment
        │   └── bulk-import/page.tsx                     # CSV/Excel Importer & Smart Fitment Matcher
        ├── workshops/page.tsx                           # Manajemen Afiliasi Bengkel Rekanan & Laporan Rekap B2B
        ├── warranty-claims/[claimId]/page.tsx           # Ruang Diskusi Internal 3-Way Triage & Pengiriman Unit Pengganti
        ├── vouchers/page.tsx                            # Manajemen Voucher Toko Brand (D_brand)
        ├── warehouses/page.tsx                          # Pengaturan Multi-Gudang (PostGIS), Jam Kerja, & Timezone
        └── finance/page.tsx                             # [BRAND_OWNER ONLY] Escrow Ledger, Potongan Ongkir Mode 3, & Iris Payout
```

### 2.3. Peta Rute Portal 3: `apps/web-workshop/src/app`
```text
apps/web-workshop/src/
├── proxy.ts                                             # RBAC & Mechanic PIN Guard
└── app/
    ├── m/[magicToken]/page.tsx                          # Halaman Konfirmasi Cepat via WhatsApp Signed Magic Link (Read-Only GET)
    ├── mechanic-pin/page.tsx                            # Layar Input 4-Digit PIN Mekanik (Mode Lantai Bengkel)
    └── (portal)/
        ├── page.tsx                                     # Antrean Mobil Hari Ini (Pagi/Siang/Sore) & Paket Kurir Masuk
        ├── inbound/page.tsx                             # Scan QR Paket Kurir -> Foto Segel -> Aktifkan Timer 14 Hari
        ├── execution/[bookingId]/
        │   ├── step-1-checkin/page.tsx                  # Scan QR Konsumen/Supir + Live Camera 2 Foto Kondisi Awal Mobil
        │   ├── step-2-serial/page.tsx                   # Scan Serial Number per Unit Fisik (atau Lapor DOA / Kondisi Mobil Rusak)
        │   └── step-3-qc-handover/page.tsx              # Live Camera Foto Terpasang + Plat Nomor + Checklist Part Lama
        ├── warranty-inspection/[claimId]/page.tsx       # Form Diagnosa Fisik Klaim Garansi oleh Mekanik
        ├── inventory/page.tsx                           # Manajemen Stok Lokal Bengkel (Mode 2 Instant Booking)
        ├── b2b-reports/page.tsx                         # [WORKSHOP_OWNER ONLY] Ekspor Rekap Pekerjaan Selesai untuk Klaim B2B ke Brand
        └── settings/
            ├── branches-and-quotas/page.tsx             # Kapasitas Sesi (Pagi/Siang/Sore), Jam Kerja, & Libur Cabang
            ├── capabilities/page.tsx                    # Checklist Kapabilitas Jenis Pekerjaan Cabang (Tanpa Tabel Harga Tier)
            └── affiliations/page.tsx                    # Daftar Afiliasi Official Brand
```

### 2.4. Peta Rute Portal 4: `apps/web-admin/src/app`
```text
apps/web-admin/src/
├── proxy.ts                                             # Strict Admin Guard (SUPER_ADMIN, OPS_AND_MEDIATOR, FINANCE_AUDITOR)
└── app/
    └── (backoffice)/
        ├── partners-kyc/page.tsx                        # White-Glove Onboarding Wizard & Verifikasi KYC + Midtrans Iris Inquiry
        ├── vehicles-master/page.tsx                     # CRUD Master Kendaraan (MMY + Chassis Code) & Antrean Crowdsourced
        ├── taxonomy-and-services/page.tsx               # 10 Kategori Dual-Label, JSONB Schema Builder, & Master Service Types
        ├── mediation-center/
        │   ├── warranty-triage/page.tsx                 # Mediasi Klaim Garansi 3 Arah & Tie-Breaker Decision
        │   └── workshop-incidents/page.tsx              # Mediasi Insiden Pengerjaan (Skenario 11)
        ├── finance-ledger/page.tsx                      # Double-Entry Ledger Explorer, Rekonsiliasi Webhook, & Pajak PPMSE
        └── system-configs/page.tsx                      # [SUPER_ADMIN ONLY] Panel Parameter Dinamis & Kalender Libur Nasional
```

---

## 3. Spesifikasi Wireframe & Logika Interaksi Komponen UI Paling Kritis (`v2.1.0-FINAL`)

### 3.1. Komponen 1: Widget *Instant O2O Workshop Preview* di Halaman Produk (PDP)
Tujuan: Menegaskan bahwa harga yang tertera **sudah termasuk Gratis Jasa Pasang & Gratis Ongkir ke Bengkel Rekanan**, serta memperlihatkan bengkel terdekat sebelum konsumen menekan *Beli*.

```text
┌──────────────────────────────────────────────────────────────────────────────┐
│ 🛠️ FASILITAS PEMASANGAN RESMI & GARANSI PENUH (SUDAH TERMASUK DI HARGA)      │
│ ──────────────────────────────────────────────────────────────────────────── │
│ ✨ GRATIS JASA PASANG (Rp 0)   •   🚚 GRATIS ONGKIR KE BENGKEL REKANAN (Rp 0)│
│    ⓘ Termasuk bahan bantu standar bengkel (Lihat rincian cakupan kerja)      │
│                                                                              │
│ 📍 Lokasi Anda: [ Tangerang Selatan (Ubah) ] • 🚗 Mobil: [ Innova Zenix 2024 ]│
│ ✅ Tersedia di 6 Bengkel Rekanan Resmi (Terdekat 2.4 km di Kec. Serpong)     │
│                                                                              │
│ ┌──────────────────────────────────┐  ┌──────────────────────────────────┐   │
│ │ ⭐ 4.9 (142) • 2.4 km             │  │ ⭐ 4.8 (89) • 5.1 km              │   │
│ │ Nawilis Auto Service - Cab. BSD  │  │ ProAuto Undercarriage - Bintaro  │   │
│ │ Kec. Serpong, Tangerang Selatan  │  │ Kec. Pondok Aren, Tangsel        │   │
│ │ 🟢 Stok Ready di Bengkel (H+0)   │  │ 📦 Dikirim dari Brand (Tiba H+3)  │   │
│ │ 💰 Jasa Pasang & Ongkir: GRATIS  │  │ 💰 Jasa Pasang & Ongkir: GRATIS  │   │
│ │ [ Lihat Foto Fasilitas & Ulasan ]│  │ [ Lihat Foto Fasilitas & Ulasan ]│   │
│ └──────────────────────────────────┘  └──────────────────────────────────┘   │
│ 🔒 Alamat jalan lengkap, peta navigasi, & QR Booking diberikan setelah bayar │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

### 3.2. Komponen 2: Layar Checkout 3-Mode, *10-Minute Slot Soft-Lock*, & *Educational Warranty Waiver*
Tujuan: Menampilkan kontras keuntungan memilih *Pasang di Bengkel Rekanan* (Gratis Pasang + Gratis Ongkir + Garansi Aktif) vs. *Kirim ke Rumah* (Bayar Ongkir + Garansi Hangus).

```text
┌──────────────────────────────────────────────────────────────────────────────┐
│ 1. PILIH METODE PEMENUHAN PESANAN                                            │
│                                                                              │
│ (●) PASANG DI BENGKEL REKANAN RESMI [DIREKOMENDASIKAN — PALING HEMAT & AMAN] │
│     ✅ Jasa Pemasangan Profesional : GRATIS (Rp 0)                           │
│     ✅ Ongkos Kirim ke Bengkel     : GRATIS (Rp 0 — Ditanggung Brand)        │
│     🛡️ Garansi Produk 12 Bulan + Garansi Pasang 30 Hari AKTIF PENUH          │
│                                                                              │
│ ( ) BELI PRODUK SAJA (KIRIM KE ALAMAT RUMAH)                                 │
│     🚚 Ongkos Kirim ke Rumah       : Ditanggung Pembeli (+ Rp 65.000)        │
│     ⚠️ Tanpa pemasangan resmi — Garansi fungsi produk TIDAK BERLAKU (VOID)   │
├──────────────────────────────────────────────────────────────────────────────┤
│ [JIKA PILIH OPSI 2 (KIRIM KE RUMAH) -> TAMPILKAN KOTAK EDUKATIF INI:]        │
│ ┌──────────────────────────────────────────────────────────────────────────┐ │
│ │ ⚠️ INFORMASI PENTING PERLINDUNGAN GARANSI & BIAYA PENGIRIMAN             │ │
│ │ Harga produk ini sudah termasuk fasilitas pemasangan gratis di Bengkel   │ │
│ │ Rekanan Resmi. Jika dikirim langsung ke rumah, Anda dikenakan biaya      │ │
│ │ ongkir ke rumah dan Garansi Fungsi Produk (12 Bulan) dinyatakan HANGUS   │ │
│ │ untuk melindungi dari risiko salah instalasi oleh pihak luar.            │ │
│ │                                                                          │ │
│ │ [🔄 Beralih ke Pasang Gratis di Bengkel Rekanan (Gratis Ongkir & Pasang)]│ │
│ │                                                                          │ │
│ │ [✓] Saya memahami dan menyetujui bahwa jaminan Garansi Produk tidak      │ │
│ │     berlaku (Void) karena tidak dipasang di Bengkel Rekanan NewTKK.      │ │
│ └──────────────────────────────────────────────────────────────────────────┘ │
├──────────────────────────────────────────────────────────────────────────────┤
│ [JIKA PILIH OPSI 1 (PASANG DI BENGKEL) -> TAMPILKAN PANEL RINGKAS INI:]      │
│ 2. DATA KENDARAAN PEMASANGAN (Dipilih dari Garasi Anda)                      │
│    [🚗 Toyota Kijang Innova Zenix (AG10) • 2024]  [Ganti Mobil]              │
│    Nomor Polisi Wajib : [ B 1889 TKK ]   Catatan Varian : [ 2.0 Q HV CVT ]   │
│                                                                              │
│ 3. LOKASI BENGKEL & JADWAL KEDATANGAN                                        │
│    Bengkel Terpilih : Nawilis Auto Service - Cab. BSD (Kec. Serpong)         │
│    Tanggal Pasang   : [ Sabtu, 24 Okt 2026 ]  Sesi : [ Pagi (09:00-12:00) ]  │
│    ⏱️ Kuota sesi Anda dikunci sementara selama 09:42 menit untuk pembayaran. │
│                                                                              │
│ 4. RINGKASAN PEMBAYARAN                                                      │
│    • Harga Produk All-In (1 Set Shockbreaker)            Rp 4.000.000        │
│    • Jasa Pemasangan di Bengkel Rekanan                  GRATIS (Rp 0)       │
│    • Ongkos Kirim ke Bengkel Rekanan                     GRATIS (Rp 0)       │
│    • Biaya Layanan Aplikasi                              Rp     5.000        │
│    ───────────────────────────────────────────────────────────────────────── │
│    TOTAL BAYAR                                           Rp 4.005.000        │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

### 3.3. Komponen 3: Kartu *1-Click Side-by-Side Fitment Verification* (Portal Brand)
```text
┌──────────────────────────────────────────────────────────────────────────────┐
│ ⏳ ANTREAN VALIDASI FITMENT #NTKK-20261020-99412     [Sisa SLA: 03j 42m]     │
├──────────────────────────────────────┬───────────────────────────────────────┤
│ 📦 PRODUK YANG DIBELI KONSUMEN       │ 🚗 DEKLARASI MOBIL KONSUMEN           │
│ • SKU: BIL-B6-INNOVA-SET             │ • Merek & Model:                      │
│ • Nama: Shockbreaker Bilstein B6     │   Toyota Kijang Innova Reborn (AN140) │
│   Heavy Duty Full Set (4 Pcs)        │ • Tahun Pembuatan: 2021               │
│ • Pemetaan Sistem:                   │ • Catatan Varian/Mesin:               │
│   Innova Reborn AN140 [2016 - 2026]  │   "2.4 V Luxury Diesel Matic"         │
│ • Status Match Sistem: 🟢 COCOK      │ • Plat Nomor: B 2021 KJG              │
├──────────────────────────────────────┴───────────────────────────────────────┤
│ KEPUTUSAN ADMIN OFFICIAL BRAND:                                              │
│ [ ✅ KONFIRMASI COCOK & BUKA LABEL KIRIM ]  [ ⚠️ SARANKAN SKU PENGGANTI ]    │
│                                             [ ❌ TOLAK & REFUND 100% ]       │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

### 3.4. Komponen 4: *3-Step High-Contrast Mechanic Wizard* (Portal Bengkel — Mode PIN 4-Digit)
```text
┌──────────────────────────────────────────────────────────────────────────────┐
│ 🔧 TUGAS KERJA MEKANIK • BOOKING #BKG-882910          [Mode PIN: Hendra (M)] │
│ Mobil: TOYOTA KIJANG INNOVA ZENIX (2024) • PLAT: B 1889 TKK                  │
├──────────────────────────────────────────────────────────────────────────────┤
│ [LANGKAH 1: CHECK-IN & FOTO KONDISI AWAL] ──► [2: SCAN SERI] ──► [3: QC AKHIR]│
│                                                                              │
│ 1. Ambil Foto Mobil di Stall Bengkel (Wajib terlihat Plat Nomor B 1889 TKK)  │
│    [ 📷 BUKA KAMERA LANGSUNG (AUTO-WATERMARK GPS & JAM SERVER) ]             │
│                                                                              │
│ 2. Ambil Foto Kondisi Awal Area Kerja / Indikator Dashboard Sebelum Dibongkar│
│    [ 📷 BUKA KAMERA LANGSUNG (AUTO-WATERMARK GPS & JAM SERVER) ]             │
│                                                                              │
│ ──────────────────────────────────────────────────────────────────────────── │
│ ⚠️ TOMBOL DARURAT SEBELUM PEMASANGAN:                                        │
│ [ 🔴 Lapor Produk Cacat Pabrik (DOA) ]  [ 🟠 Kondisi Mobil Rusak/Tidak Aman ]│
└──────────────────────────────────────────────────────────────────────────────┘
```

---

### 3.5. Komponen 5: *Digital Warranty Passport* & *Single-Window Claim Card* (Portal Konsumen)
```text
┌──────────────────────────────────────────────────────────────────────────────┐
│ 🛡️ SERTIFIKAT GARANSI DIGITAL RESMI NEWTKK        No: NTKK-WRT-202610-88912 │
│ ════════════════════════════════════════════════════════════════════════════ │
│ KENDARAAN TERIKAT : TOYOTA KIJANG INNOVA ZENIX (2024) • PLAT: B 1889 TKK     │
│ PRODUK TERPASANG  : Shockbreaker Tein EnduraPro Plus (Unit #1 - Depan Kiri)  │
│ NOMOR SERI RESMI  : SN-TEIN-99281726-FL                                      │
│ DIPASANG OLEH     : Nawilis Auto Service - Cabang BSD (24 Okt 2026)          │
│ ──────────────────────────────────────────────────────────────────────────── │
│ 🟢 GARANSI INSTALASI BENGKEL : AKTIF (Sisa 26 Hari • s/d 23 Nov 2026)        │
│    Mencakup: Bunyi tidak wajar pasca-pasang, kekencangan baut, & setelan.    │
│                                                                              │
│ 🟢 GARANSI PRODUK OFFICIAL BRAND : AKTIF (Sisa 361 Hari • s/d 24 Okt 2027)   │
│    Mencakup: Kebocoran oli tabung, cacat fungsi redaman, & cacat pabrik.     │
│ ──────────────────────────────────────────────────────────────────────────── │
│ [ 🛠️ AJUKAN KLAIM GARANSI SATU PINTU ]   [ 🔄 TRANSFER KE PEMILIK BARU (1x) ]│
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 4. Cetak Biru Arsitektur Ekspansi Fase 2 & Fase 3 (*Future-Proofing Spec*)

### 4.1. Fase 2.1: Fitur *"Shop the Build"* (Visual Inspiration & 1-Click All-In Bundling)
* Menyajikan galeri referensi mobil nyata yang telah selesai dibangun di bengkel rekanan NewTKK lengkap dengan titik *Interactive Image Hotspots* di atas foto mobil (`vehicle_builds` & `vehicle_build_hotspots`).
* Karena seluruh produk sudah menggunakan model **True All-In Pricing (Gratis Pasang & Gratis Ongkir ke Bengkel)**, tombol *"Beli Paket Build Ini"* menjadi sangat sederhana: konsumen langsung melihat total harga paket tanpa perlu pusing menghitung kombinasi ongkir atau tarif jasa per item!

### 4.2. Fase 2.2: Fitur *In-App Additional Work Order* (Penambahan Part Saat Mobil di Lift Bengkel)
* Ketika mobil sedang berada di *lift* bengkel dan mekanik menemukan komponen lain yang perlu diganti (dari stok lokal Mode 2 di bengkel tersebut), mekanik mengirimkan foto bukti ber-*watermark* + penawaran SKU resmi ke HP pemilik mobil.
* Pemilik mobil menyetujui dan membayar via Midtrans Snap, lalu sistem otomatis menerbitkan *Digital Warranty Passport* tambahan untuk komponen tersebut.

### 4.3. Fase 3: Ekspansi Aplikasi Mobile (`apps/mobile-customer` & `apps/mobile-workshop` — React Native + Expo)
* Menggunakan ulang **> 70% kode TypeScript, kontrak Zod v4.6.4 (`@newtkk/contracts`), dan hooks TanStack Query v5** di dalam monorepo Turborepo tanpa perubahan satu baris pun pada Backend NestJS v12.

---

## 5. Urutan Eksekusi Pembangunan 6 Sprint (`Master Implementation Checklist v2.1.0`)

| Tahap / Sprint | Fokus Eksekusi Teknis | Acuan Dokumen Utama |
| :--- | :--- | :--- |
| **Sprint 1: Monorepo Bootstrap, DB Schema (45 Tabel) & Seeder** | Inisialisasi Turborepo + pnpm v10 Catalogs (TS 7.0.2), konfigurasi Docker (`PostgreSQL 18.6 + PostGIS + Redis + Typesense`), implementasi DDL Drizzle ORM di `@newtkk/db` (dengan constraint `chk_free_workshop_shipping_for_customer`), dan eksekusi `db:seed`. | **Dokumen 3** & **Dokumen 4 (Bab 1)** |
| **Sprint 2: Shared Contracts (`@newtkk/contracts`), Auth & RBAC Guards** | Implementasi skema Zod v4.6.4 (`PublicWorkshopPreviewSchema` dengan `installationFeeForCustomerIdr: 0` & `shippingFeeToWorkshopForCustomerIdr: 0`), autentikasi JWT + WA OTP, `TenantScopeGuard`, dan Next.js 16.4 `src/proxy.ts`. | **Dokumen 2 (Bab 1–2)** & **Dokumen 4 (Bab 3–4)** |
| **Sprint 3: Portal Brand, Portal Bengkel (Setup) & Portal Admin Backoffice** | CRUD Katalog SKU Harga All-In + Simulator Margin Brand + *Packed Dimensions*, *Clone Fitment*, matriks `int4range` + sinkronisasi `Outbox -> Typesense`, manajemen afiliasi $M:N$, kapabilitas bengkel (`workshop_branch_capabilities`), & kuota sesi harian. | **Dokumen 2 (Bab 4, 5, 6)** & **Dokumen 4 (Bab 10)** |
| **Sprint 4: Customer Storefront, O2O Preview & Zero-Trust All-In Checkout** | Halaman Katalog, PDP dengan widget *Instant O2O Workshop Preview* (PostGIS + Anti-Bypass Stripping), *Cart Grouping*, logika Gratis Pasang & Gratis Ongkir Mode 2/3 vs Ongkir Mode 1, *10-Minute Slot Soft-Lock*, & pembayaran Midtrans Snap/Core. | **Dokumen 1 (Bab 3)**, **Dokumen 2 (Bab 3)**, & **Dokumen 5 (Bab 3)** |
| **Sprint 5: Order State Machine, Magic Links, Live Camera QC & SLA Workers** | Alur 1-Klik Fitment Check Brand, pengiriman Mode 3 Biteship (beban ongkir Brand), *WhatsApp Signed Magic Link* Bengkel, *Inbound Package Check*, *3-Step Mechanic Wizard* dengan *Server-Side Sharp Watermark* ke Cloudflare R2, & BullMQ SLA Workers. | **Dokumen 1 (Bab 4)** & **Dokumen 4 (Bab 7–9)** |
| **Sprint 6: Double-Entry Escrow Ledger, Midtrans Iris Payout & Digital Warranty** | Jurnal akuntansi berpasangan (`ORDER_PAID_ESCROW`, `COMPLETED_AND_SETTLED` dengan pemotongan $L_{\text{brand\_ship}}$), protokol *Two-Phase Midtrans Iris Payout* ke Brand & Refund Wallet, *Digital Warranty Passport* (Klaim 3 Arah + 1x Transfer), Laporan Rekap B2B Bengkel, & *Dual-Rating*. | **Dokumen 1 (Bab 5)**, **Dokumen 3 (Bab 4)**, & **Dokumen 4 (Bab 6)** |
