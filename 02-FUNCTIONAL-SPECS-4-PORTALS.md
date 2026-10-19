# 📗 DOKUMEN 2: FUNCTIONAL SPECIFICATIONS, RBAC MATRIX & ACCEPTANCE CRITERIA (4 PORTALS)

* **Nama Projek:** NewTKK
* **Kode Dokumen:** `DOC-02-FUNCTIONAL-SPECS`
* **Lokasi File:** `docs/02-FUNCTIONAL-SPECS-4-PORTALS.md`
* **Versi:** `2.1.0-FINAL` *(True All-In Pricing + Free Ship-to-Workshop Borne by Brand + Pure Operational Workshop Portal)*
* **Tanggal Efektif:** 20 Oktober 2026
* **Referensi Induk:** `docs/01-MASTER-PRD-NEWTKK.md`
* **Target Pembaca:** Product Manager, System Architect, Frontend/Backend/Mobile Engineers, QA Automation Engineers, & AI Coding Agents (Claude Code, Cursor, Antigravity, OpenCode)

---

## 1. Topologi Aplikasi & Arsitektur Autentikasi (Monorepo 4 Portals)

Di dalam repositori **Turborepo (`@newtkk/*`)**, antarmuka pengguna dibagi menjadi **4 aplikasi web independen berbasis Next.js v16.4 (App Router + Turbopack)** yang berkomunikasi ke satu **Headless Backend API (NestJS v12)** melalui kontrak terpusat `@newtkk/contracts` (Zod v4.6.4):

| Kode Portal | Direktori Monorepo | Target Domain / Subdomain | Target Perangkat Utama | Strategi Rendering Utama |
| :--- | :--- | :--- | :--- | :--- |
| **Portal 1: Customer Storefront** | `apps/web-customer` | `newtkk.com` / `www.newtkk.com` | Mobile-First PWA & Desktop Browser | **SSR + ISR + Cache Components** (Optimasi SEO Maksimal untuk Katalog, Brand, & Area Bengkel) |
| **Portal 2: Official Brand Portal** | `apps/web-brand` | `brand.newtkk.com` | Desktop-First & Tablet Responsive | **CSR / Dynamic SSR** behind Auth Guard (`src/proxy.ts`) |
| **Portal 3: Workshop Partner Portal** | `apps/web-workshop` | `workshop.newtkk.com` | **Mobile-First PWA** (HP/Tablet Lapangan Bengkel) & Desktop Kasir | **CSR / Dynamic SSR** + *WebRTC Camera & QR Scanner API* |
| **Portal 4: Super Admin Backoffice** | `apps/web-admin` | `admin-internal.newtkk.com` | Desktop Wide-Screen | **CSR / Dynamic SSR** + Strict IP/2FA Guard |

### Standar Autentikasi & Manajemen Sesi Lintas Portal
1. **Customer Auth:** Mendukung *Passwordless OTP via WhatsApp Business API*, *Google OAuth 2.0*, dan *Email + Password*. Menggunakan `HttpOnly, Secure, SameSite=Lax` JWT Access Token (15 menit) + Refresh Token Rotation (30 hari) yang disimpan di Redis.
2. **Partner & Admin Auth (Brand, Workshop, Backoffice):** Menggunakan *Email/No. HP + Password* dengan kewajiban **2FA / OTP WhatsApp** untuk aksi sensitif (perubahan nomor rekening bank dan penarikan dana *Midtrans Iris Payout* pada Portal Brand).
3. **Signed Magic Link & Mechanic PIN Token (Khusus Operasional Bengkel):**
   * Untuk mempercepat respons di lapangan, pesan WhatsApp ke Admin Bengkel menyertakan **Signed Magic Link** (`/m/[short_token]`) yang berlaku selama 24 jam dan terikat pada aksi pesanan spesifik.
   * Untuk eksekusi mekanik di lantai bengkel tanpa mengekspos akun manajemen bengkel, sistem menyediakan **Task QR / 4-Digit Mechanic PIN Session** yang membatasi layar hanya pada pengunggahan foto QC dan pemindaian nomor seri produk.

---

## 2. Matriks Role-Based Access Control (RBAC) & Permission Guards

Sistem mengimplementasikan **10 Peran Pengguna (`UserRole`)** dengan pemisahan hak akses ketat di level *API Guard* (NestJS `@RequirePermissions()`) dan *Route Proxy* (Next.js 16.4 `src/proxy.ts`).

*(Catatan Arsitektur v2.1.0: Karena biaya jasa pasang sudah termasuk di dalam Harga Produk All-In dan diselesaikan secara B2B antara Official Brand dan Bengkel Rekanan di luar sistem pembayaran NewTKK, maka Portal Bengkel tidak memiliki menu penarikan dana/wallet; seluruh pencairan dana mitra terpusat di Portal Official Brand).*

| Portal | Kode Peran (`Role`) | Deskripsi Aktor | Hak Akses Diizinkan (`Allowed Permissions`) | Pembatasan Keras (`Strict Denials`) |
| :--- | :--- | :--- | :--- | :--- |
| **Portal 1 (Customer)** | `CUSTOMER_GUEST` | Pengunjung belum *login*. | Browse katalog, filter MMY, lihat *O2O Workshop Preview* (kecamatan, jarak, & status gratis pasang/ongkir). | Tidak bisa *Checkout*, melihat alamat jalan/telepon bengkel, atau *Chat*. |
| | `CUSTOMER_MEMBER` | Pemilik kendaraan terdaftar. | Semua akses Guest + *Checkout* Mode 1/2/3, kelola *My Garage*, bagikan *Driver Check-In Pass*, kelola *Digital Warranty Passport* (klaim & 1x transfer), *Dual-Rating*, tarik *Refund Balance*. | Hanya dapat mengakses data transaksi & garansi miliknya sendiri. |
| **Portal 2 (Brand)** | `BRAND_OWNER` | Pemilik / Direktur Prinsipal Brand. | Akses penuh Portal Brand: kelola profil & multi-gudang, kelola staf, afiliasi bengkel, katalog, pesanan, voucher Brand, serta **lihat saldo bersih ($N_{\text{brand}}$) & eksekusi Iris Payout / ubah rekening bank**. | Perubahan rekening bank wajib lulus validasi nama Midtrans Iris + OTP WA Owner. |
| | `BRAND_CATALOG_ADMIN` | Staf Katalog & Merchandising Brand. | Buat/ubah SKU (Harga All-In), *Bulk CSV Import*, *Clone Fitment*, atur stok per gudang, atur dimensi kemasan (`packed_dimensions`). | **Dilarang** mengakses menu Keuangan/Saldo, Payout, dan pengaturan komisi/rekening. |
| | `BRAND_ORDER_ADMIN` | Staf Operasional & CS Pesanan Brand. | Eksekusi **1-Klik Validasi Fitment**, pilih kurir & cetak label pengiriman (termasuk pengiriman Mode 3 ke bengkel yang ditanggung Brand), tangani komplain DOA & diskusi klaim garansi 3 arah, balas *chat* produk. | **Dilarang** mengubah harga SKU, mengubah stok master tanpa log, atau mengakses menu Keuangan/Payout. |
| **Portal 3 (Workshop)** | `WORKSHOP_OWNER` | Pemilik Bengkel Rekanan (Akun Induk). | Akses penuh Portal Bengkel: kelola multi-cabang (`workshop_branches`), kapabilitas layanan (`workshop_branch_capabilities`), kuota sesi, afiliasi Brand, staf, serta **unduh laporan rekap pekerjaan selesai (Excel/PDF) untuk penagihan B2B ke Brand**. | Tidak memiliki akses ke pengaturan keuangan/saldo milik Brand. |
| | `WORKSHOP_FRONT_DESK` | Service Advisor (SA) / Admin Kasir Cabang. | Konfirmasi/ajukan *reschedule* jadwal booking cabang, *Inbound Package Check* (foto segel paket tiba), *Scan QR Check-in* konsumen, kelola stok lokal bengkel (Mode 2), cetak *Work Order*, serta eksekusi penuh form QC. | **Dilarang** mengubah pengaturan cabang induk, menghapus afiliasi Brand, atau melihat data cabang lain. |
| | `WORKSHOP_MECHANIC` | Mekanik Lapangan (Login via PIN 4-Digit / Task QR). | Melihat antrean pengerjaan cabang hari ini, ambil **Foto Kondisi Awal Mobil**, scan/input **Serial Number Produk**, ambil **Foto QC Terpasang + Plat Nomor**, isi **Checklist Part Lama**, lapor kendala `ABORTED_BY_VEHICLE_CONDITION` / `REPORT_DOA_DEFECT`. | **Buta Finansial (*Financial Blindness*):** Tidak dapat melihat harga barang maupun data nomor HP pribadi konsumen. |
| **Portal 4 (Backoffice)** | `SUPER_ADMIN` | Eksekutif / System Owner NewTKK. | Akses penuh seluruh modul Backoffice, konfigurasi `platform_system_configs`, manajemen akun admin, & *override* status darurat. | Setiap perubahan nilai konfigurasi atau *override* wajib mengisi alasan dan tercatat di `immutable_audit_logs`. |
| | `OPS_AND_MEDIATOR` | Tim Operasional, Onboarding & Teknis NewTKK. | *White-Glove Partner Onboarding*, verifikasi KYC Brand/Bengkel, kelola Master Data Kendaraan (MMY), kelola 10 Kategori & JSONB Schema, mediasi klaim garansi 3 arah & insiden bengkel. | **Dilarang** mengeksekusi pencairan dana manual tanpa persetujuan `FINANCE_AUDITOR` / `SUPER_ADMIN`. |
| | `FINANCE_AUDITOR` | Tim Keuangan & Akuntansi NewTKK. | Pantau *Double-Entry Escrow Ledger*, rekonsiliasi *webhook* Midtrans & pemotongan ongkir Mode 3 / selisih volumetrik Biteship, setujui *payout* bernilai besar, unduh laporan pajak/faktur PPMSE. | **Dilarang** mengubah katalog produk, memodifikasi ulasan, atau mengubah matriks teknis kendaraan. |

---

## 3. Spesifikasi Fungsional Portal 1: Customer Storefront (`apps/web-customer`)

### Modul 1.1: Discovery, Pencarian Instan & *Frictionless Soft-Fitment*
* **Deskripsi Fungsional:** Memungkinkan konsumen mencari produk secara bebas tanpa dipaksa mendaftar atau mengisi garasi terlebih dahulu, dengan dukungan filter kompatibilitas kendaraan 3-level yang cepat.
* **Spesifikasi Fitur:**
  1. **Homepage 10 Root Categories Grid:** Menampilkan 10 kategori utama menggunakan atribut `ui_short_label` (maksimal 14 karakter: *Kaki-Kaki, Pengereman, Velg & Ban, Oli & Servis, Performa, Pendingin & AC, Transmisi, Kelistrikan, Audio Mobil, Bodi & Proteksi*) agar rapi dalam grid 2x5 di layar mobile tanpa teks terpotong.
  2. **Typesense Instant Search Bar:** Pencarian *real-time* (< 30ms) dengan dukungan toleransi salah ketik (*typo-tolerance*) untuk kombinasi nama produk + merek + nama mobil (contoh: ketik `"kampas rem innova zenix"` langsung menyaring kategori Pengereman dan model Kijang Innova Zenix).
  3. **Sticky Vehicle Selector Bar (Opsional / Soft-Fitment):**
     * Konsumen dapat memilih **Merek (`Make`) -> Model (`Model`) -> Tahun (`Year`)** di bagian atas layar.
     * Jika dipilih, pilihan disimpan di `localStorage` (untuk Guest) atau tabel `user_garage_vehicles` (untuk Member), dan setiap kartu produk menampilkan lencana kontekstual:
       * 🟢 **`Cocok untuk [Nama Mobil Anda]`** (jika SKU terpemeta pada MMY tersebut).
       * 🔵 **`Universal / Sesuai Ukuran Spesifikasi`** (jika produk bertipe `UNIVERSAL_FIT` atau `SPEC_BASED`).
       * ⚪ **Tanpa Pemblokiran:** Konsumen tetap diizinkan membuka produk lain dan mengubah pilihan mobil saat *Checkout*.

### Modul 1.2: Halaman Detail Produk (PDP) & *Instant O2O Workshop Preview Widget*
* **Deskripsi Fungsional:** Memberikan kepastian bahwa harga produk adalah **Harga All-In (Gratis Pasang + Gratis Ongkir ke Bengkel Rekanan)** serta menampilkan ketersediaan bengkel rekanan terdekat **sebelum** konsumen menekan tombol "Tambah ke Keranjang".
* **Spesifikasi Komponen PDP:**
  1. **Informasi Harga All-In, Keaslian & Garansi:**
     * Lencana **100% Official Store [Nama Brand]**.
     * **Lencana Nilai Utama (*True All-In Value Banner*):**
       * `✨ GRATIS JASA PASANG DI BENGKEL REKANAN (Rp 0)`
       * `🚚 GRATIS ONGKIR PENGIRIMAN KE BENGKEL REKANAN (Rp 0 — Ditanggung Brand)`
     * Informasi Garansi jelas: *"Garansi Resmi Produk [X] Bulan + Garansi Pemasangan 30 Hari (Wajib pasang di Bengkel Rekanan NewTKK)"*.
     * Indikator ketersediaan: `Ready Stock` atau `Pre-Order / Custom ([N] Hari Kerja)`.
  2. **Widget *Instant O2O Workshop Preview* (Anti-Bypass Compliant):**
     * Mendeteksi kota/kabupaten konsumen (via IP Geolocation / pilihan kota manual / GPS Browser).
     * Menampilkan ringkasan jaringan pemasangan untuk produk tersebut:
       > *"Tersedia pemasangan gratis di **[N] Bengkel Rekanan Resmi** di sekitar **[Kota Anda]** (Terdekat **2.3 km** di Kec. Serpong)."*
     * Menampilkan daftar kartu ringkas bengkel rekanan yang mencakup:
       * **Nama Bengkel, Rating Bintang & Jumlah Ulasan, Kecamatan/Kota, Jarak (km), dan Foto Fasilitas**.
       * **Status Ketersediaan Barang & Jadwal:**
         * 🟢 **`Mode 2 (Stok Ready di Bengkel):`** Bisa langsung pasang Hari Ini / Besok ($H+0 / H+1$) • Jasa Pasang & Ongkir: **GRATIS (Rp 0)**.
         * 📦 **`Mode 3 (Dikirim dari Gudang Brand ke Bengkel):`** Estimasi tiba di bengkel & siap pasang mulai $H + \text{Lead Time}$ • Jasa Pasang & Ongkir ke Bengkel: **GRATIS (Rp 0)**.
       * *(Catatan Keamanan Skenario 15: Alamat jalan lengkap dan nomor telepon bengkel disembunyikan sebelum transaksi berstatus `PAID`).*
     * **Rincian Cakupan Pemasangan Gratis (*All-In Scope* — Skenario 8):** Menampilkan daftar *bullet points* apa yang sudah termasuk dalam fasilitas pemasangan gratis (termasuk bahan bantu standar bengkel) dan apa yang opsional di luar paket (misal: *Spooring 3D*).

### Modul 1.3: Keranjang Belanja (*Cart Scoping*) & Alur Checkout 3 Mode
* **Aturan Keranjang Belanja (`Cart Grouping Rules`):**
  1. Item di keranjang otomatis dikelompokkan per **Official Brand** (`1 Checkout = 1 Official Brand + 1 Lokasi Pengiriman/Bengkel`).
  2. **Aturan Dominasi `MANDATORY_WORKSHOP_ONLY`:** Jika dalam grup *checkout* terdapat minimal 1 SKU dengan kebijakan `MANDATORY_WORKSHOP_ONLY` (seperti Kaca Film / Coating), maka pilihan **Mode 1 (Beli Produk Saja / Kirim ke Rumah)** otomatis dikunci/dinonaktifkan dengan penjelasan pada layar.
* **Spesifikasi Layar Checkout:**
  * **Langkah 1 — Pilih Mode Pemenuhan (`Fulfillment Mode`):**
    * **Opsi A: Pasang di Bengkel Rekanan Resmi (Direkomendasikan — Gratis Jasa Pasang, Gratis Ongkir ke Bengkel, & Garansi Aktif Penuh)**
    * **Opsi B: Beli Produk Saja / Kirim ke Alamat Rumah (Bayar Ongkir ke Rumah & Garansi Produk Hangus)**
  * **Langkah 2A (Jika Pilih Opsi A — Pasang di Bengkel Rekanan / Mode 2 & 3):**
    1. **Deklarasi Data Kendaraan Wajib:**
       * Pilih **Merek, Model, & Tahun** dari Master Data (atau gunakan *1-Tap Autofill* dari "Garasi Saya"). Jika varian sangat langka dan tidak ditemukan di daftar, konsumen dapat klik *"Mobil saya tidak ada di daftar"* -> ketik nama mobil & tahun secara manual.
       * Input **Nomor Polisi Kendaraan** (Wajib, format tervalidasi regex plat Indonesia misal `B 1234 ABC`).
       * Input **Catatan Varian/Mesin/Transmisi** (misal: `2.4 Dakar Diesel AT`).
    2. **Pemilihan Bengkel Rekanan (*Capability Intersection Filter*):**
       * Sistem memfilter dan hanya menampilkan cabang Bengkel Rekanan yang berafiliasi aktif dengan *Brand* tersebut **DAN** memiliki kapabilitas aktif (`workshop_branch_capabilities`) untuk **seluruh `service_type_code`** dari produk yang ada di *checkout* tersebut.
       * Sistem otomatis menentukan apakah pesanan berjalan dengan **Mode 2 (Stok Ready di Bengkel)** atau **Mode 3 (Dikirim Gratis dari Gudang Brand ke Bengkel)**. Pada kedua mode ini, konsumen dikenakan biaya jasa pasang **`Rp 0`** dan biaya ongkos kirim **`Rp 0`**.
    3. **Pemilihan Tanggal & Sesi Waktu Pemasangan (`Smart Booking Calendar`):**
       * Untuk **Mode 2**, tanggal paling awal adalah $H+0$ (jika sebelum jam *cut-off* cabang) atau $H+1$.
       * Untuk **Mode 3**, sistem memanggil estimasi durasi pengiriman kurir di latar belakang dan mengunci seluruh tanggal sebelum $T_{\text{min}} = H + D_{\text{prod}} + D_{\text{ship}} + 1 \text{ Hari Buffer}$.
       * Konsumen memilih tanggal tersedia dan **Sesi Waktu**:
         * `MORNING` (Pagi: 09:00 – 12:00)
         * `AFTERNOON` (Siang: 12:00 – 15:00)
         * `EVENING` (Sore: 15:00 – 17:30)
       * Sesi yang kuota kapasitas hariannya sudah penuh (`booked_count >= max_capacity`) ditampilkan sebagai `Penuh (Disabled)`.
  * **Langkah 2B (Jika Pilih Opsi B — Kirim ke Rumah / Mode 1):**
    1. Harga produk tetap sama rata (tidak ada pengurangan harga).
    2. Konsumen memilih alamat rumah/kantor dan **wajib memilih serta membayar layanan ekspedisi via Biteship ($L_{\text{mode1\_est}}$)**.
    3. **Komponen Wajib *Educational Warranty Void Waiver*:** Muncul kotak peringatan kontras tinggi (dilengkapi tombol cepat *"Ubah ke Pasang Gratis di Bengkel Rekanan"*) dan *checkbox* wajib centang bahwa konsumen menyetujui **Garansi Produk Hangus** karena tidak dipasang di Bengkel Rekanan. Tombol "Bayar" tidak aktif (`disabled`) sebelum dicentang.
  * **Langkah 3 — Voucher & Pembayaran (Midtrans Snap / Core API):**
    * Konsumen dapat memasukkan kode **Voucher Brand** ($D_{\text{brand}}$) dan/atau **Voucher Promo Platform** ($D_{\text{platform}}$), serta menggunakan **Saldo Pengembalian (`Refundable Balance`)** jika ada.
    * Klik **"Bayar Sekarang"** -> mengunci kuota sesi sementara (*Soft-Lock*) dan memunculkan *Midtrans Payment Modal* (VA BCA/Mandiri/BRI/BNI/Permata, QRIS, GoPay, ShopeePay, Kartu Kredit Full/Cicilan).

### Modul 1.4: Manajemen Pesanan, Tiket Booking, *Driver Pass* & *Remote Handover*
* **Fitur Utama pasca-Pembayaran (`PAID`):**
  1. **Pembukaan Informasi Bengkel:** Begitu status `PAID`, halaman detail pesanan membuka alamat jalan lengkap bengkel, tombol navigasi **Google Maps / Waze**, nomor telepon darurat bengkel, dan status progres pengiriman barang dari Brand ke bengkel.
  2. **Alur Persetujuan Reschedule / Alternatif Fitment:**
     * Jika bengkel mengajukan jadwal pengganti (`RESCHEDULE_PROPOSED`), konsumen dapat menekan **"Setuju Jadwal Baru"**, **"Pilih Tanggal Lain"**, atau **"Pindah ke Bengkel Rekanan Lain"** (maksimal perubahan jadwal oleh konsumen adalah 2 kali dalam rentang 14 hari sejak barang tiba di bengkel).
  3. **Tiket Kedatangan (*QR Check-In*) & *Shareable Driver Pass* (Skenario 10):**
     * Saat status mencapai `READY_AT_WORKSHOP_FOR_INSTALL`, muncul **QR Code Check-In** dan **6-Digit PIN Kedatangan**.
     * Tersedia tombol **"Kirim Tiket Kedatangan ke Supir / Perwakilan (WhatsApp)"**: menghasilkan halaman web publik terenkripsi (`/pass/[token]`) yang hanya menampilkan Nama Bengkel, Peta Lokasi, Jadwal, Nomor Polisi, Daftar Pekerjaan, dan QR Code Check-In (tanpa menampilkan harga barang atau akses ke akun pemilik).
  4. **Konfirmasi Penyelesaian Jarak Jauh (*Remote Owner Handover Approval*):**
     * Saat mekanik selesai bekerja (`AWAITING_CUSTOMER_HANDOVER`), pemilik akun menerima notifikasi WhatsApp dan *push notification*.
     * Layar pesanan menampilkan **Galeri Bukti QC Bengkel** (Foto kondisi awal vs. Foto produk terpasang + Foto plat nomor + Status komponen lama di bagasi).
     * Pemilik akun memiliki waktu **2x24 jam** untuk menekan:
       * ✅ **"Konfirmasi Pekerjaan Selesai"** (atau memberikan 6-Digit *Completion PIN* kepada supir di lokasi), **ATAU**
       * 🚨 **"Laporkan Insiden / Kendala Pengerjaan"** (jika terjadi kerusakan pada mobil sesuai Skenario 11).

### Modul 1.5: *Digital Warranty Passport*, Klaim Satu Pintu, *Dual-Rating* & Saldo Refund
1. **Menu "Garansi Saya" (*Digital Warranty Passport*):**
   * Menampilkan daftar kartu garansi aktif yang terikat pada **Nomor Polisi Kendaraan** dan **Serial Number Produk**.
   * Menampilkan 2 indikator masa aktif terpisah pada setiap kartu:
     * **Garansi Pemasangan Bengkel** (Countdown 30 Hari).
     * **Garansi Resmi Produk Brand** (Countdown Bulan/Tahun atau *Badge Authenticity Guaranteed*).
   * **Tombol "Ajukan Klaim Garansi":** Membuka form satu pintu (pilih keluhan, unggah foto/video, pilih jadwal kunjungan inspeksi ke bengkel rekanan).
   * **Tombol "Transfer Kepemilikan Garansi (Maks. 1x)":** Memungkinkan pemilik mobil memindahkan kartu garansi aktif ke akun pembeli mobil barunya melalui input Email/No. WhatsApp pemilik baru + verifikasi OTP pemilik lama.
2. **Formulir *Dual-Rating* (Ulasan Terpisah):**
   * Setelah status `COMPLETED_AND_SETTLED`, konsumen mengisi 2 penilaian independen dalam satu layar:
     * **Bagian 1:** Rating Bintang (1–5), Tag Cepat, & Ulasan Foto untuk **Kualitas Produk & Official Brand**.
     * **Bagian 2:** Rating Bintang (1–5), Tag Cepat (*Mekanik Rapi, Tepat Waktu, Ruang Tunggu Nyaman*), & Ulasan untuk **Layanan Bengkel Rekanan**.
3. **Dompet Pengembalian Dana (`Customer Refundable Balance`):**
   * Menampilkan dana *refund* dari transaksi Virtual Account / QRIS yang batal atau *partial refund*.
   * Tombol **"Tarik Instan ke Rekening Bank"** (terintegrasi dengan **Midtrans Iris Payout API**).

### Acceptance Criteria Kritis — Portal 1 (Customer Storefront)
* **[AC-CUST-01]** **GIVEN** konsumen berada di halaman *Checkout* dan memilih *Mode 2 (Stok Ready di Bengkel)* atau *Mode 3 (Ship-to-Workshop)*, **WHEN** rincian tagihan dikalkulasi oleh sistem, **THEN** baris *Jasa Pemasangan* wajib bernilai `Rp 0 (Gratis)` DAN baris *Ongkos Kirim ke Bengkel* wajib bernilai `Rp 0 (Ditanggung Official Brand)`.
* **[AC-CUST-02]** **GIVEN** konsumen berada di halaman *Checkout* dan memilih *Mode 1 (Kirim ke Rumah)* untuk produk bergaransi, **WHEN** konsumen belum mencentang *checkbox Warranty Void Waiver*, **THEN** tombol *"Bayar Sekarang"* wajib berstatus `disabled`, konsumen wajib dikenakan biaya *Ongkos Kirim ke Rumah* ($L_{\text{mode1\_est}}$), dan saat dicentang lalu dibayar, sistem wajib menyimpan `warranty_waiver_accepted_at` beserta IP Address.
* **[AC-CUST-03]** **GIVEN** konsumen membeli produk dengan *Mode 3 (Ship-to-Workshop)* yang memiliki `production_lead_time_days = 3` dan estimasi kurir ke bengkel `2–4 hari`, **WHEN** konsumen membuka kalender *booking* bengkel, **THEN** seluruh tanggal sebelum $H + 3 + 4 + 1 = H+8$ wajib terkunci (`disabled`).
* **[AC-CUST-04]** **GIVEN** konsumen memasukkan 2 produk dari 1 Brand ke keranjang di mana Produk A membutuhkan `SVC_OIL` dan Produk B membutuhkan `SVC_TINT`, **WHEN** sistem memuat daftar pilihan bengkel saat *checkout*, **THEN** sistem hanya boleh menampilkan bengkel rekanan yang memiliki kapabilitas aktif untuk **kedua** layanan tersebut sekaligus.

---

## 4. Spesifikasi Fungsional Portal 2: Official Brand Partner Dashboard (`apps/web-brand`)

### Modul 2.1: Onboarding, Profil Toko Resmi & Manajemen Multi-Gudang
1. **Aktivasi & KYC:** Mendukung alur aktivasi dari *White-Glove Admin Onboarding* maupun pendaftaran mandiri (unggah NIB, NPWP, Surat Prinsipal/Distributor Resmi, dan validasi rekening bank via Midtrans Iris).
2. **Manajemen Multi-Gudang (`Origin Warehouses`):**
   * *Brand* dapat mendaftarkan $\ge 1$ titik gudang pengiriman lengkap dengan titik koordinat peta (`lat/lng`), kode pos Biteship, zona waktu IANA (`WIB/WITA/WIT`), serta jadwal jam kerja operasional dan tanggal libur gudang.
   * Saat konsumen memesan barang (Mode 1 atau Mode 3), sistem otomatis memilih gudang *Brand* terdekat yang memiliki stok tersedia untuk meminimalkan ongkos kirim dan waktu tempuh.

### Modul 2.2: Manajemen Katalog (Harga All-In), Simulator Margin & Produktivitas Fitment Massal
1. **Pembuatan & Pengelolaan SKU:**
   * Input Nama SKU, Nomor Part Resmi (*OEM/Aftermarket Part Number*), Kategori 3-Level (otomatis memuat form atribut `JSONB` sesuai kategori), **Harga Katalog All-In (`catalog_price_idr` — sudah termasuk fasilitas pasang di bengkel rekanan)**, Stok per Gudang, dan Status Ketersediaan (`READY_STOCK` vs `PRE_ORDER_CUSTOM` + `production_lead_time_days`).
   * **Input Wajib Dimensi Kemasan (`Packed Dimensions` — Skenario 13):** Wajib mengisi Berat Kemasan (gram) serta Panjang $\times$ Lebar $\times$ Tinggi (cm) **setelah dikemas kardus/kayu**.
   * **Widget Simulator Margin Bersih Brand:** Menampilkan simulasi langsung di layar form SKU: `Harga Jual All-In` dikurangi `Komisi Platform (%)` dan `Estimasi Ongkir Mode 3 ke Bengkel` sehingga Admin Brand dapat menetapkan harga jual dengan margin yang sehat.
   * **Konfigurasi Layanan & Garansi per SKU:**
     * Pilih Kebijakan Pemenuhan: `ALLOW_ALL` atau `MANDATORY_WORKSHOP_ONLY`.
     * Pilih Kode Pekerjaan Bengkel (`service_type_code`) yang dibutuhkan untuk memasang produk ini.
     * Pilih Tipe Garansi (`FUNCTIONAL_PERIODIC` + durasi bulan, atau `AUTHENTICITY_AND_DOA`).
2. **Fitur Produktivitas Pemetaan Kendaraan (*High-Speed Fitment Mapping*):**
   * **Opsi 1 — *Clone Fitment from Existing SKU*:** Menyalin seluruh daftar mobil yang cocok dari SKU lain milik *Brand* tersebut dalam 1 klik.
   * **Opsi 2 — *Vehicle Platform Presets*:** Memilih kelompok sasis/platform mobil populer sekaligus (misal: centang grup *"Platform Toyota IMV: Innova / Fortuner / Hilux"*).
   * **Opsi 3 — *Bulk CSV/Excel Catalog & Fitment Importer*:** Mengunggah ratusan SKU beserta rentang tahun mobil menggunakan template Excel terstandarisasi dengan pratinjau validasi error sebelum disimpan.

### Modul 2.3: Manajemen Jaringan Afiliasi Bengkel Rekanan
* **Direktori & Undangan Kemitraan:**
  * Mencari bengkel terdaftar di NewTKK berdasarkan kota/spesialisasi dan mengirim **Undangan Afiliasi Resmi**, atau menyetujui/menolak permohonan afiliasi yang diajukan oleh bengkel.
  * Mengatur cakupan otorisasi per bengkel (misal: Bengkel A diizinkan memasang seluruh kategori produk *Brand*, sedangkan Bengkel B hanya diizinkan untuk kategori Oli & Pengereman).
  * Memantau metrik performa bengkel rekanan (Rating Pemasangan dari konsumen, kecepatan konfirmasi jadwal, riwayat klaim, serta mengunduh laporan rekapitulasi pemasangan per bengkel untuk penyelesaian insentif B2B).

### Modul 2.4: Antrean *1-Click Fitment Check*, Pengiriman Mode 3 (Ongkir Ditanggung Brand) & DOA
1. **Antrean Prioritas `AWAITING_FITMENT_CONFIRMATION` (SLA 1x4 Jam Kerja):**
   * Menampilkan kartu perbandingan berdampingan (*Side-by-Side Verification Card*):
     * **Sisi Kiri:** Foto SKU, Nama Produk, Spesifikasi Teknis, & Daftar Fitment SKU.
     * **Sisi Kanan:** Deklarasi Kendaraan Konsumen (**Merek, Model, Tahun, Varian/Transmisi, & Nomor Polisi**).
   * **3 Tombol Aksi Cepat:**
     1. 🟢 **`[Konfirmasi Cocok — Lanjut Proses]`:** Mengubah status `brand_fitment_status = 'APPROVED'`.
     2. 🟡 **`[Kurang Tepat — Sarankan SKU Lain / Diskusi]`:** Membuka *chat* langsung ke konsumen & mengirim notifikasi WhatsApp berisi tautan SKU pengganti.
     3. 🔴 **`[Tolak & Batalkan — Refund 100%]`:** Membatalkan transaksi tanpa penalti jika tidak ada produk yang cocok.
2. **Pemilihan Kurir, Pencetakan Label & Penyerahan Kurir (Mode 1 & Mode 3):**
   * Untuk **Mode 3 (Ship-to-Workshop)**, tombol **"Pilih Kurir, Cetak Label & Request Pickup"** baru aktif setelah *Fitment* disetujui **DAN** jadwal Bengkel Rekanan terkonfirmasi (`CONFIRMED`).
   * Pada Mode 3, Admin Brand memilih layanan ekspedisi (Reguler/Kargo via Biteship) yang memenuhi target tanggal pasang konsumen. Biaya pengiriman ke bengkel tersebut ($L_{\text{mode3\_actual}}$) ditampilkan secara transparan sebagai beban logistik *Brand* yang akan dipotong dari pencairan transaksi terkait.
   * Label pengiriman Mode 3 mencetak dengan jelas: **Nama & Alamat Bengkel Rekanan Tujuan**, **Nama Konsumen & Plat Nomor Mobil**, serta **QR Code Inbound Check** untuk dipindai oleh Admin Bengkel saat paket tiba.
3. **Penanganan Komplain Paket Rusak (`INBOUND_DAMAGE`) & Cacat Pabrik (`REPORT_DOA_DEFECT`):**
   * Melihat bukti foto/video dari bengkel dan memicu pengiriman unit pengganti ekspres (`Replacement Shipment`).

### Modul 2.5: Resolusi Klaim Garansi 3 Arah, Voucher Brand & Keuangan (Iris Payout)
1. **Ruang Triage Klaim Garansi (`INTERNAL_3WAY_TRIAGE`):**
   * Melihat hasil diagnosa mekanik bengkel atas klaim garansi konsumen, berdiskusi dengan Bengkel dan Mediator NewTKK, serta menekan persetujuan pengiriman unit pengganti garansi dalam SLA 2x24 jam kerja.
2. **Manajemen Voucher Toko Brand ($D_{\text{brand}}$):**
   * Membuat kode voucher diskon produk yang biayanya ditanggung oleh promosi *Brand*.
3. **Dashboard Keuangan & Midtrans Iris Payout:**
   * Menampilkan **Saldo Tertahan di Escrow (`Escrow Pending`)** vs. **Saldo Siap Tarik (`Available Balance`)**.
   * Menampilkan rincian transparan per pesanan:
     $$N_{\text{brand}} = P_{\text{net}} - C_{\text{brand}} - L_{\text{brand\_ship}}$$
   * Fitur penarikan dana instan ke rekening bank perusahaan melalui **Midtrans Iris API** dan pengunduhan laporan mutasi/faktur pajak bulanan.

### Acceptance Criteria Kritis — Portal 2 (Official Brand Portal)
* **[AC-BRAND-01]** **GIVEN** pesanan Mode 3 baru masuk dengan status `PAID_AWAITING_VALIDATION`, **WHEN** Admin Brand belum menekan tombol *Konfirmasi Cocok* ATAU Bengkel belum mengonfirmasi jadwal, **THEN** tombol *"Request Pickup Kurir / Cetak Label"* wajib terkunci (`disabled`) untuk mencegah barang terlanjur dikirim ke bengkel yang menolak jadwal.
* **[AC-BRAND-02]** **GIVEN** pesanan Mode 3 dikirim dari Gudang Brand ke Bengkel Rekanan dengan tagihan aktual Biteship sebesar Rp 75.000, **WHEN** pesanan selesai (`COMPLETED_AND_SETTLED`), **THEN** sistem wajib memotong Rp 75.000 tersebut sebagai `brand_borne_shipping_fee_idr` ($L_{\text{brand\_ship}}$) dari pencairan bersih Brand ($N_{\text{brand}}$).

---

## 5. Spesifikasi Fungsional Portal 3: Affiliated Workshop Portal (`apps/web-workshop`)

*(Dirancang sebagai **100% Pure Operational Mobile-First Web App** dengan antarmuka **High-Contrast UI** dan tombol berukuran minimal 56px agar sangat mudah dioperasikan menggunakan smartphone atau tablet di lingkungan bengkel).*

### Modul 3.1: Manajemen Multi-Cabang, Kapabilitas Layanan & Kapasitas Sesi
1. **Pengaturan Cabang & Kapasitas Sesi (`Branch & Slot Capacity`):**
   * Mengatur titik lokasi GPS akurat (PostGIS), zona waktu IANA (`WIB/WITA/WIT`), foto fasilitas *stall*/ruang tunggu, jam buka per hari, tanggal libur bengkel, serta **Kuota Maksimal Mobil per Sesi** (`Pagi`, `Siang`, `Sore`).
2. **Konfigurasi Kapabilitas Jenis Pekerjaan (`Workshop Service Capabilities`):**
   * Tanpa perlu mengisi tabel tarif rumit, Owner/Admin Bengkel cukup mencentang daftar jenis pekerjaan (`master_service_types`, misal: *Pasang Shockbreaker/Coilover, Pasang Kampas/Cakram Rem, Ganti Oli Mesin/Transmisi, Retrofit Bi-LED, Instalasi Dashcam Hardwire, Pasang Kaca Film*) yang mampu dikerjakan oleh peralatan dan mekanik di cabang tersebut.

### Modul 3.2: Konfirmasi Jadwal (*WhatsApp Signed Magic Link*) & *Inbound Package Check*
1. **Konfirmasi Jadwal Masuk (SLA 2x4 Jam Kerja):**
   * Saat ada pesanan baru, Admin/SA Bengkel menerima pesan WhatsApp berisi ringkasan mobil, jenis pekerjaan, tanggal/sesi pilihan konsumen, dan tombol **Signed Magic Link**.
   * Saat diklik, langsung membuka layar aksi cepat dengan 2 pilihan:
     * ✅ **`[Terima Jadwal Booking]`**
     * 📅 **`[Sesi Penuh — Ajukan Tanggal/Sesi Pengganti]`** (memilih 2 alternatif waktu kosong untuk dikirim balik ke konsumen).
2. **SOP Penerimaan Paket dari Kurir (`Inbound Package Verification` — Khusus Mode 3):**
   * Saat paket dari *Official Brand* tiba di bengkel, Admin Bengkel memindai QR Code pada label pengiriman paket.
   * Admin memilih kondisi fisik paket luar:
     * **Kondisi Baik & Bersegel:** Wajib memfoto paket di lokasi bengkel -> klik **"Terima & Simpan di Bengkel"** -> status berubah menjadi `READY_AT_WORKSHOP_FOR_INSTALL` dan *timer* batas simpan 14 hari dimulai.
     * **Kemasan Hancur / Bocor Parah:** Wajib memfoto/memvideokan kerusakan -> klik **"Laporkan Kerusakan Kurir (`INBOUND_DAMAGE_REPORTED`)"**.

### Modul 3.3: Eksekusi Lantai Bengkel (*Check-In, Live Camera QC & Mechanic PIN Mode*)
Admin Kasir/SA dapat menampilkan **QR Tugas Kerja** atau mekanik cukup memasukkan **4-Digit PIN Mekanik** di HP operasional bengkel untuk menjalankan 4 langkah berurutan berikut:

1. **Langkah 1 — Check-In Kedatangan & *Pre-Installation Condition Photo* (Skenario 11):**
   * Scan **QR Code Booking** (atau input 6-Digit PIN) dari HP Konsumen / Supir.
   * **Wajib Ambil Minimal 2 Foto Kondisi Awal Mobil** menggunakan **Kamera Langsung (`Live Camera Capture` dengan Server-Side Auto-Watermark Timestamp + GPS + Booking ID)**:
     * *Foto 1:* Mobil berada di *stall* bengkel (memperlihatkan Plat Nomor).
     * *Foto 2:* Kondisi awal area kerja / panel *dashboard* / indikator *speedometer* sebelum disentuh mekanik.
2. **Langkah 2 — Buka Segel & Pemindaian Nomor Seri (`Serial Number Scan`):**
   * Mekanik memindai *barcode/QR* atau memfoto + mengetik **Nomor Seri Produk** untuk setiap unit fisik (`order_item_units`) yang akan dipasang.
   * **Tombol Darurat Sebelum/Saat Bongkar:**
     * Jika saat kardus dibuka produk ternyata cacat pabrik/bocor/salah isi -> klik 🔴 **`[Laporkan Produk Cacat DOA]`** + unggah video bukti (Skenario 5).
     * Jika saat mobil diangkat ternyata sasis/komponen mobil konsumen rusak parah sehingga tidak aman dipasang -> klik 🟠 **`[Batalkan Pasang: Kondisi Mobil Tidak Aman]`** + unggah minimal 2 foto kerusakan pada mobil konsumen untuk dimintakan persetujuan di layar HP konsumen (Skenario 4).
3. **Langkah 3 — Penyelesaian Pemasangan & *Post-Installation QC Form*:**
   * Setelah komponen selesai dipasang dan diuji, mekanik wajib mengisi:
     * **Foto 1 (Wajib Live Camera):** Komponen baru yang sudah terpasang rapi di kendaraan.
     * **Foto 2 (Wajib Live Camera):** Tampak kendaraan selesai pengerjaan / indikator bebas *error*.
     * **Checklist Komponen Lama Konsumen (`Old Parts Handover` — Skenario 14):** Pilih wajib antara:
       * `[x] Komponen lama telah dikemas dan dimasukkan ke bagasi mobil konsumen` (disertai 1 foto di bagasi), **ATAU**
       * `[x] Komponen lama (oli/filter/part rusak) ditinggal di bengkel atas persetujuan konsumen`.
     * **Catatan Teknis Mekanik (Opsional):** Misal saran servis berikutnya untuk konsumen.
4. **Langkah 4 — Serah Terima Kendaraan (`Handover`):**
   * Klik **"Selesai Pengerjaan & Minta Konfirmasi Konsumen"** -> mengirimkan hasil QC ke HP pemilik akun konsumen (atau mekanik dapat memasukkan 6-Digit *Completion PIN* dari konsumen/supir agar transaksi langsung selesai detik itu juga di tempat).

### Modul 3.4: Inspeksi Klaim Garansi, Inventori Lokal (Mode 2) & Laporan Rekonsiliasi B2B
1. **Modul Diagnosa Klaim Garansi (`UNDER_WORKSHOP_DIAGNOSIS`):**
   * Saat konsumen datang membawa tiket klaim garansi, mekanik memverifikasi kesesuaian plat nomor dan nomor seri, lalu mengunggah foto/video pemeriksaan dan memilih hasil diagnosa (*A: Perbaikan Setelan/Instalasi oleh Bengkel*, *B: Indikasi Cacat Pabrik Produk -> Eskalasi ke Triage 3 Arah*, atau *C: Kerusakan Akibat Pemakaian/Tabrakan*).
2. **Manajemen Stok Lokal Bengkel (Khusus Mode 2):**
   * Mengatur jumlah stok fisik titipan/milik bengkel yang diizinkan untuk *Instant Booking* Mode 2, dilengkapi tombol darurat `WORKSHOP_STOCK_DISCREPANCY` jika terjadi selisih stok fisik.
3. **Laporan Rekapitulasi Pekerjaan Selesai (`B2B Installation Settlement Report`):**
   * Menyediakan tabel riwayat dan fitur unduh laporan **Excel/CSV & PDF Resmi NewTKK** per periode dan per *Official Brand* yang merinci seluruh pemasangan yang telah selesai (`Booking ID, Tanggal Selesai, Nama Brand, SKU, Serial Number, Plat Nomor Mobil, & Tautan Bukti Foto QC`) untuk digunakan oleh Owner Bengkel dalam rekonsiliasi B2B dengan masing-masing *Official Brand*.

### Acceptance Criteria Kritis — Portal 3 (Affiliated Workshop Portal)
* **[AC-WORK-01]** **GIVEN** pengguna berada di dalam sesi `WORKSHOP_MECHANIC` (PIN 4-Digit), **WHEN** mekanik membuka detail pesanan kerja, **THEN** sistem wajib menyembunyikan seluruh informasi harga barang dan kontak pribadi konsumen, serta hanya menampilkan spesifikasi kendaraan, daftar unit fisik yang harus dipasang, dan langkah form QC.
* **[AC-WORK-02]** **GIVEN** mekanik sedang mengisi form *Pre-Installation Photo* atau *Post-Installation QC Photo*, **WHEN** mekanik mengambil foto melalui kamera web/HP, **THEN** server wajib membakar *watermark* permanen (`Booking ID + Server Timestamp WIB/WITA/WIT + Lat/Lng`) menggunakan `sharp` sebelum disimpan ke Cloudflare R2, dan jika menggunakan unggah galeri darurat, sistem wajib menandai transaksi dengan bendera `FLAGGED_GALLERY_UPLOAD = true`.

---

## 6. Spesifikasi Fungsional Portal 4: Super Admin Backoffice (`apps/web-admin`)

### Modul 4.1: *White-Glove Partner Onboarding* & Verifikasi KYC Mitra
1. **Wizard *Admin-Assisted Onboarding*:**
   * Memungkinkan tim BD/Ops NewTKK mendaftarkan akun *Official Brand* atau *Bengkel Rekanan* secara langsung, mengunggah dokumen legalitas, memvalidasi nomor rekening bank *Official Brand* secara *real-time* melalui **Midtrans Iris Bank Account Validation API**, dan mengirimkan tautan aktivasi ke pemilik mitra.
2. **Antrean Verifikasi KYC Mandiri (`PENDING_KYC_VERIFICATION`):**
   * Meninjau dokumen pendaftaran mitra baru, menyetujui/menolak dengan catatan perbaikan, serta mengatur tarif komisi khusus per *Brand* (`custom_commission_rate`) jika berbeda dari tarif *default*.

### Modul 4.2: Manajemen Master Data Kendaraan (MMY & Crowdsourced Queue)
1. **CRUD Master Kendaraan Terpusat:**
   * Mengelola tabel `vehicle_makes`, `vehicle_models` (dengan pemisahan kode sasis `chassis_code`), dan `vehicle_generations` (tahun awal–akhir `int4range`, tipe mesin, PCD roda OEM, ukuran ban OEM).
2. **Antrean Kurasi Kendaraan Input Manual (`PENDING_MASTER_VEHICLE_REVIEW`):**
   * Menampilkan daftar kendaraan langka yang diketik manual oleh konsumen saat *checkout*. Admin dapat melakukan **1-Klik Merge** (jika ternyata sudah ada di database dengan penulisan berbeda) atau **1-Klik Promote to Master Data** agar langsung tersedia bagi seluruh pengguna berikutnya.

### Modul 4.3: Manajemen Taksonomi 3-Level Dinamis, JSONB Schema Builder & Master Layanan
1. **Dynamic 3-Level Category Manager:**
   * Mengelola 10 Kategori Utama (mengatur `ui_short_label` dan `full_display_name`), Sub-Kategori Level 2, dan Tipe Produk Level 3 tanpa perlu *redeployment* kode.
   * **JSONB Attribute Template Builder:** Menentukan daftar *field* spesifikasi teknis untuk setiap kategori Level 3 (nama atribut, tipe input: *number/text/select/boolean*, satuan unit, dan apakah dijadikan filter di *Storefront*).
2. **Master Katalog Jenis Pekerjaan (`Service Type Catalog`):**
   * Mengelola daftar `service_type_code`, estimasi durasi pengerjaan standar, serta daftar standar *"Apa yang Termasuk (All-In Consumables)"* dan *"Apa yang Tidak Termasuk"*.

### Modul 4.4: Pusat Mediasi Sengketa 3 Arah, Insiden Bengkel & Kalender SLA
1. **Pusat Mediasi Klaim Garansi (`3-Way Warranty Triage Center`):**
   * Memantau seluruh tiket klaim garansi yang masuk ke tahap `INTERNAL_3WAY_TRIAGE`.
   * Menampilkan bukti foto pemasangan awal (QC saat transaksi selesai) berdampingan dengan foto/video diagnosa klaim saat ini.
   * **Tombol Putusan Mediator (`Tie-Breaker Decision`):** Jika *Brand* dan Bengkel tidak mencapai kesepakatan dalam 2x24 jam, Mediator NewTKK dapat menetapkan pihak yang bertanggung jawab sesuai matriks liabilitas PKS.
2. **Resolusi Insiden Pengerjaan (`INCIDENT_UNDER_MEDIATION`) & Selisih Kondisi Mobil:**
   * Memeriksa laporan kerusakan kendaraan saat instalasi (Skenario 11) atau memverifikasi pembatalan `ABORTED_BY_VEHICLE_CONDITION` jika konsumen mengajukan keberatan.
3. **Kalender Hari Libur Nasional Indonesia (`SLA Holiday Calendar`):**
   * Mengatur daftar tanggal merah nasional agar *BullMQ SLA Timer* tidak menghitung mundur jam batas respons mitra pada hari libur nasional.

### Modul 4.5: *Double-Entry Escrow Ledger*, Rekonsiliasi Midtrans/Biteship & Konfigurasi Sistem
1. **Audit Buku Besar Keuangan (`Escrow & Ledger Explorer`):**
   * Memantau keseimbangan mutlak buku besar akuntansi berpasangan ($\sum \text{Debit} = \sum \text{Kredit}$).
   * Memantau pemotongan biaya logistik Mode 3 (`brand_borne_shipping_fee_idr`) dan antrean pencairan dana **Midtrans Iris Payout** ke rekening *Official Brand* maupun *Customer Refund Wallet*, serta mengunduh laporan pajak PPMSE.
2. **Manajemen Promo Platform ($D_{\text{platform}}$) & Banner CMS:**
   * Membuat kode promo subsidi platform dan mengelola *banner* kampanye di *Storefront*.
3. **Panel Konfigurasi Parameter Sistem (`platform_system_configs` UI):**
   * Antarmuka formulir aman bagi `SUPER_ADMIN` untuk mengubah nilai parameter operasional sistem (seperti durasi *storage* 14 hari, waktu *auto-complete* 48 jam, biaya layanan aplikasi Rp5.000, dan tarif komisi *default* 8%) secara *real-time* dengan pencatatan log perubahan.

---

## 7. Matriks Notifikasi Lintas Portal & *Automated SLA Triggers* (BullMQ + WhatsApp API)

| Kode Event Pemicu | Penerima Notifikasi | Saluran (*Channel*) | Isi Pesan Utama & Aksi Cepat (*Call-to-Action*) | Timer / SLA BullMQ Terkait |
| :--- | :--- | :--- | :--- | :--- |
| `ORDER_CREATED_UNPAID` | Konsumen | WA + In-App + Email | Tagihan pembayaran Rp [Nominal] + Link bayar Midtrans. | Job `EXPIRE_UNPAID_ORDER` (60 menit / 15 menit jika slot terakhir). |
| `ORDER_PAID_ESCROW` | • **Official Brand**<br>• **Bengkel Rekanan**<br>• **Konsumen** | • WA + Brand Portal<br>• **WA Signed Magic Link**<br>• WA + In-App | • Brand: *"Pesanan baru! Segera validasi kecocokan mobil [Mobil Konsumen]."*<br>• Bengkel: *"Booking masuk untuk tanggal [Tgl/Sesi]. Klik link ini untuk Terima/Ubah."* | • Job `CHECK_BRAND_FITMENT_SLA` (4 jam kerja).<br>• Job `CHECK_WORKSHOP_CONFIRM_SLA` (8 jam kerja). |
| `SHIPMENT_DISPATCHED` | Bengkel & Konsumen | WA + In-App | Nomor resi kurir + Estimasi tanggal paket tiba di Bengkel Rekanan. | Webhook listener Biteship (`DELIVERED`). |
| `PACKAGE_ARRIVED_AT_WORKSHOP` | Konsumen | WA + In-App + Email | *"Produk Anda sudah diterima & diperiksa oleh Bengkel [Nama]. Ini QR Code Check-In Anda untuk jadwal [Tanggal]."* | Job `REMIND_STORAGE_DAY_3`, `DAY_7`, `DAY_12`, & `EXECUTE_FORFEIT_DAY_15`. |
| `BOOKING_REMINDER_H_MINUS_1` | Konsumen & Bengkel | WA + Push Notif | Pengingat kunjungan besok pukul [Sesi Waktu] + Tombol *"Bagikan Tiket ke Supir"* + Peta Lokasi Bengkel. | Dieksekusi H-1 pukul 09:00 waktu lokal. |
| `WORKSHOP_QC_SUBMITTED` | Pemilik Akun Konsumen | **WA Prioritas** + Push Notif | *"Pemasangan di mobil [Plat Nomor] selesai! Lihat foto hasil pemasangan & klik Konfirmasi Selesai."* | Job `AUTO_COMPLETE_HANDOVER` (48 jam setelah QC diunggah). |
| `ORDER_COMPLETED_SETTLED` | • **Konsumen**<br>• **Brand & Bengkel** | • WA + In-App<br>• Portal Notif + Email | • Konsumen: Link **Digital Warranty Passport** + Form **Dual-Rating**.<br>• Brand: Notifikasi dana bersih ($N_{\text{brand}}$) telah masuk ke *Available Balance*. | — |
| `WARRANTY_CLAIM_SUBMITTED` | Bengkel Pemasang, Brand, & Ops NewTKK | WA + Portal Alert | Peringatan klaim garansi satu pintu masuk beserta jadwal inspeksi kendaraan konsumen. | Job `CHECK_TRIAGE_DECISION_SLA` (2x24 jam kerja setelah diagnosa bengkel). |
