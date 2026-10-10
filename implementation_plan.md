# Implementation Plan: Scraping Data & Cetak Biru Web-Based MotaApp
## Lengkap dengan Seluruh Diagram Mermaid (Arsitektur, ERD, dan 7 Sequence Diagram)

> Dokumen ini memuat **seluruh hasil scraping data aplikasi Android MotaApp** dan rencana pengembangannya ke platform **Web-Based**, dilengkapi dengan **10 Diagram Mermaid lengkap** (Arsitektur, ERD, Flowchart, dan Sequence Diagram untuk setiap alur bisnis).

---

## User Review Required

> [!IMPORTANT]
> **Daftar Seluruh Diagram Mermaid di Dokumen Ini:**
> 1. **Diagram 1:** Arsitektur Sistem & Hardware Android vs Web-Based
> 2. **Diagram 2:** Navigation Map & Role-Based Access Control
> 3. **Diagram 3:** Database Schema ERD (10 Tabel Relasional Lengkap)
> 4. **Diagram 4 (Sequence):** Alur Login & Session Management
> 5. **Diagram 5 (Sequence):** Alur Transaksi Kasir POS (Scan, Keranjang, Multi-Payment, Database)
> 6. **Diagram 6 (Sequence):** Alur Pembayaran Piutang & Cicilan Jatuh Tempo Grosir
> 7. **Diagram 7 (Sequence):** Alur Mutasi Stok Antar-Cabang (Surat Jalan & Approval)
> 8. **Diagram 8 (Sequence):** Alur Pengeluaran Kas & Refund Transaksi
> 9. **Diagram 9 (Sequence):** Alur Closing Dana Shift Kasir Harian
> 10. **Diagram 10 (Sequence):** Alur Cetak Struk Thermal Bluetooth ESC/POS

---

# SECTION 1: ARSITEKTUR & DIAGRAM MERMAID LENGKAP

---

### Diagram 1: Arsitektur Sistem & Integrasi Hardware

```mermaid
graph TB
    subgraph Web_Client["Frontend Client (Browser / Tablet Kasir)"]
        UI["Next.js 14 Web UI<br/>(React, Tailwind CSS, Shadcn/UI)"]
        Store["Zustand Store<br/>(Cart State, Session, Active Filter)"]
        HW_Layer["Hardware Layer:<br/>- Barcode Scanner USB (Keyboard Listener)<br/>- Webcam Scanner (html5-qrcode)<br/>- Thermal Printer Bluetooth (Web Bluetooth API)<br/>- PDF Generator (jsPDF / html2canvas)"]
    end

    subgraph Server_API["Backend Service (Next.js API / Node.js)"]
        AuthGuard["Auth Guard (JWT Bearer Token)<br/>Role Permission (toko, grosir, gudang)"]
        BusinessLogic["Business Logic & Validation:<br/>- Re-calculate Subtotal & Discount<br/>- Auto-deduct Stock on Checkout<br/>- Mutasi Stok on Surat Jalan Approval<br/>- Real-time Cash Balance Closing"]
        PrismaORM["Prisma ORM Client"]
    end

    subgraph Database_Storage["Data & Storage Layer"]
        Postgres[(PostgreSQL Relational DB)]
        CDN_Storage[("File Storage CDN<br/>(/bukti_bayar, /artikel_image, /user_logo)")]
    end

    UI --> Store
    UI --> HW_Layer
    Store --> AuthGuard
    AuthGuard --> BusinessLogic
    BusinessLogic --> PrismaORM
    PrismaORM --> Postgres
    BusinessLogic --> CDN_Storage
```

---

### Diagram 2: Navigation Map & Role-Based Access

```mermaid
graph TD
    Start["Buka Aplikasi"] --> CheckSession{Cek Sesi Login}
    
    CheckSession -->|Belum Login| AuthRoutes["Auth Stack Navigator"]
    AuthRoutes --> Login["LoginScreen (/login)"]
    AuthRoutes --> ResetPass["ResetPassword (/forgot-password)"]
    AuthRoutes --> Signup["SignupScreen (/signup)"]

    CheckSession -->|Sudah Login| RoleCheck{Cek Role User: data_user.type}

    RoleCheck -->|Role: 'toko' / Retail| RetailMenu["Drawer Menu Retail:"]
    RetailMenu --> M1["1. Kasir / Transaksi (SaleScreen)"]
    RetailMenu --> M2["2. Pengeluaran Lain (PengeluaranScreen)"]
    RetailMenu --> M3["3. Stock Opname & Cek Cabang (StockOpnameScreen)"]
    RetailMenu --> M4["4. Closing Dana Shift Harian (ClosingDanaScreen)"]
    RetailMenu --> M5["5. Profil Toko & Logo (UserScreens)"]

    RoleCheck -->|Role: 'grosir' / 'gudang'| GrosirMenu["Drawer Menu Grosir / Gudang:"]
    GrosirMenu --> G1["1. Kasir / Transaksi (SaleScreen)"]
    GrosirMenu --> G2["2. Pengeluaran Lain (PengeluaranScreen)"]
    GrosirMenu --> G3["3. Stock Opname & Surat Jalan (StockOpnameScreen)"]
    GrosirMenu --> G4["4. Closing Dana Shift Harian (ClosingDanaScreen)"]
    GrosirMenu --> G5["5. Pembayaran Jatuh Tempo / Piutang (TrxGrosirScreen)"]
    GrosirMenu --> G6["6. Profil Toko & Logo (UserScreens)"]
```

---

### Diagram 3: Database Schema ERD (10 Tabel Relasional)

```mermaid
erDiagram
    USER ||--o{ ARTIKEL : "memiliki"
    USER ||--o{ TRANSAKSI : "membuat"
    TRANSAKSI ||--|{ TRANSAKSI_DETAIL : "berisi"
    ARTIKEL ||--o{ TRANSAKSI_DETAIL : "dijual"
    TRANSAKSI ||--o{ PEMBAYARAN_GROSIR : "dicicil"
    USER ||--o{ PENGELUARAN : "mencatat"
    TRANSAKSI ||--o{ PENGELUARAN : "direfund"
    USER ||--o{ SURAT_JALAN : "pengirim"
    USER ||--o{ SURAT_JALAN : "penerima"
    SURAT_JALAN ||--|{ SURAT_JALAN_DETAIL : "berisi"
    ARTIKEL ||--o{ SURAT_JALAN_DETAIL : "dimutasi"
    USER ||--o{ CLOSING_DANA : "menutup"
    CLOSING_DANA ||--|{ CLOSING_DANA_DETAIL : "rincian"

    USER {
        int id PK
        varchar username UK
        varchar password
        varchar name
        varchar alias
        varchar email
        enum role "toko | grosir | gudang"
        varchar hp
        text address
        varchar image
        longtext image_base64
        timestamp created_at
    }

    ARTIKEL {
        int id PK
        varchar kode UK
        varchar nama_artikel
        decimal harga_jual
        int stok
        varchar image
        int user_id FK
        timestamp created_at
    }

    TRANSAKSI {
        int id PK
        varchar kode_struk UK
        int user_id FK
        timestamp tanggal
        tinyint tipe "1:Retail | 2:Grosir"
        tinyint pembayaran "1:Cash | 2:Transfer"
        tinyint pembayaran_grosir "0:Normal | 3:Piutang | 4:Keep"
        decimal total_harga
        decimal total_bayar
        decimal total_diskon
        tinyint status "1:Belum Lunas | 2:Lunas | 3:Batal"
        date jatuh_tempo
        json data_transfer
        varchar bukti_bayar
        text keterangan
    }

    TRANSAKSI_DETAIL {
        int id PK
        int transaksi_id FK
        int artikel_id FK
        string kode_artikel
        string nama_artikel
        decimal harga
        int qty
        decimal diskon
        text keterangan
    }

    PEMBAYARAN_GROSIR {
        int id PK
        int transaksi_id FK
        timestamp tanggal
        decimal nominal
        tinyint pembayaran "1:Cash | 2:Transfer"
        varchar bukti_bayar
        json data_transfer
    }

    PENGELUARAN {
        int id PK
        int user_id FK
        varchar nama
        tinyint tipe "1:Harian | 2:Refund | 3:Lainnya | 4:Transfer | 5:Setoran"
        decimal nominal
        int transaksi_id FK
        text keterangan
        timestamp tanggal
    }

    SURAT_JALAN {
        int id PK
        varchar invoice UK
        int user_dari_id FK
        int user_untuk_id FK
        varchar address_dari
        varchar address_untuk
        tinyint status "1:Menunggu | 2:Ditolak | 3:Diterima"
        timestamp tanggal
    }

    SURAT_JALAN_DETAIL {
        int id PK
        int surat_jalan_id FK
        int artikel_id FK
        varchar nama_artikel
        int qty
    }

    CLOSING_DANA {
        int id PK
        int user_id FK
        date tanggal UK
        decimal kas_awal
        decimal kas_akhir
        timestamp created_at
    }

    CLOSING_DANA_DETAIL {
        int id PK
        int closing_dana_id FK
        varchar detail
        decimal nominal
    }
```

---

### Diagram 4: Sequence Diagram — Alur Login & Autentikasi

```mermaid
sequenceDiagram
    autonumber
    actor Kasir as Kasir / User
    participant Web as Web Frontend (Login Page)
    participant API as Backend Service (/api/auth)
    participant DB as Database (USER Table)
    participant Session as LocalStorage / Cookie

    Kasir->>Web: Input Username & Password -> Klik Login
    Web->>API: POST /api/auth/login { username, password }
    API->>DB: SELECT * FROM USER WHERE username = ?
    DB-->>API: Data User & Password Hash

    alt Password Valid
        API->>API: Generate JWT Bearer Token
        API-->>Web: Response 200 OK + { token, user: { id, username, name, role, ... } }
        Web->>Session: Simpan JWT Token & Data Profil User
        Web-->>Kasir: Toast: "Login Berhasil" -> Redirect ke Dashboard POS
    else Password / User Salah
        API-->>Web: Response 401 / 404 { message: "Username atau password salah" }
        Web-->>Kasir: Alert Error: "Login Gagal"
    end
```

---

### Diagram 5: Sequence Diagram — Alur Transaksi Kasir POS (Scan, Keranjang, Multi-Payment)

```mermaid
sequenceDiagram
    autonumber
    actor Kasir as Kasir Toko
    participant POS as Layar POS (SaleScreen)
    participant Cart as Zustand Cart Store
    participant API as Backend Service (/api/transaksi)
    participant DB as Database (TRANSAKSI, DETAIL, ARTIKEL)

    Note over Kasir,POS: TAHAP 1: INPUT PRODUK KE KERANJANG
    Kasir->>POS: Scan Barcode (USB Scanner / Kamera)
    POS->>API: POST /api/stockopname/list { user, kode: barcode, limit: 1 }
    API->>DB: Query ARTIKEL by kode
    DB-->>API: Data produk (nama, harga_jual, stok)
    API-->>POS: Response produk

    alt Stok Habis (stok <= 0)
        POS-->>Kasir: Alert: "Stok Tidak Tersedia"
    else Stok Tersedia
        POS->>POS: Play Sound Beep (scan.mp3)
        POS->>Cart: Tambah Item { kode, nama, harga_jual, qty: 1 }
        Cart->>POS: Update UI Keranjang (Subtotal otomatis terhitung)
    end

    Note over Kasir,Cart: TAHAP 2: MANIPULASI ITEM KERANJANG
    Kasir->>Cart: Ubah Qty (+/-), Ubah Harga Satuan (Custom), Tambah Diskon Item, Diskon Nota
    Cart->>Cart: Hitung: Total Tagihan = Subtotal - Total Diskon

    Note over Kasir,POS: TAHAP 3: PEMILIHAN METODE PEMBAYARAN
    Kasir->>POS: Klik "Lanjutkan Pembayaran"
    Kasir->>POS: Pilih Metode (Cash / Transfer / Piutang Grosir / Keep)

    alt Pembayaran Cash
        Kasir->>POS: Input Uang Tunai Diterima -> Hitung Kembalian = Uang - Total Tagihan
    else Pembayaran Transfer
        Kasir->>POS: Input Nama Bank, No. Rekening, Atas Nama & Upload Bukti Transfer
    else Pembayaran Piutang (Grosir)
        Kasir->>POS: Pilih DP / Non-DP & Set Tanggal Jatuh Tempo
    end

    Note over Kasir,DB: TAHAP 4: SUBMIT TRANSAKSI KE DATABASE
    Kasir->>POS: Klik "Selesaikan Transaksi"
    POS->>API: POST /api/transaksi/save_trx (FormData)
    API->>API: Validasi Ulang Perhitungan di Server (Subtotal - Diskon)
    API->>DB: 1. INSERT INTO TRANSAKSI (Header Nota)
    API->>DB: 2. LOOP INSERT INTO TRANSAKSI_DETAIL (Snapshot Items)
    API->>DB: 3. UPDATE ARTIKEL SET stok = stok - qty
    DB-->>API: Transaksi Berhasil Disimpan (ID & Kode Struk)
    API-->>POS: Response 200 OK + { kode_struk }
    POS-->>Kasir: Alert: "Transaksi Berhasil" -> Buka Layar Cetak Struk
```

---

### Diagram 6: Sequence Diagram — Alur Pelunasan Piutang & Jatuh Tempo Grosir

```mermaid
sequenceDiagram
    autonumber
    actor Kasir as Kasir Grosir
    participant Web as Layar Jatuh Tempo (TrxGrosirScreen)
    participant API as Backend Service (/api/transaksi)
    participant DB as Database (TRANSAKSI, PEMBAYARAN_GROSIR)

    Kasir->>Web: Buka Menu "Pembayaran Jatuh Tempo"
    Web->>API: POST /api/transaksi/list_transaksi_grosir { user, limit, offset }
    API->>DB: SELECT * FROM TRANSAKSI WHERE status = 1 (Belum Lunas)
    DB-->>API: Daftar Nota Piutang Aktif
    API-->>Web: Tampilkan List Nota & Sisa Tagihan
    
    Kasir->>Web: Klik "Bayar" pada Nota Tertentu
    Web->>API: GET /api/transaksi/detail_pembayaran_grosir?id=...
    API->>DB: SELECT * FROM PEMBAYARAN_GROSIR WHERE transaksi_id = ?
    DB-->>API: Riwayat Cicilan / DP Sebelumnya
    API-->>Web: Tampilkan Riwayat Pembayaran & Sisa Tagihan

    Kasir->>Web: Input Nominal Angsuran + Metode (Cash/Transfer) + Bukti Transfer
    Kasir->>Web: Klik "Selesaikan Pembayaran"
    Web->>API: POST /api/transaksi/pelunasan_grosir (FormData)
    API->>DB: INSERT INTO PEMBAYARAN_GROSIR (nominal, tanggal, bukti_bayar)
    
    alt Sisa Tagihan = 0 (Lunas Penuh)
        API->>DB: UPDATE TRANSAKSI SET status = 2 (Lunas)
    else Sisa Tagihan > 0 (Masih Ada Sisa)
        API->>DB: Tetap status = 1 (Belum Lunas)
    end

    DB-->>API: Pelunasan Berhasil
    API-->>Web: Response 200 OK
    Web-->>Kasir: Toast: "Pembayaran Angsuran Berhasil Disimpan"
```

---

### Diagram 7: Sequence Diagram — Alur Mutasi Stok Antar-Cabang (Surat Jalan & Approval)

```mermaid
sequenceDiagram
    autonumber
    actor TokoA as Toko Peminta (Outlet A)
    participant WebA as Web App Toko A
    participant API as Backend Service (/api/stockopname)
    participant DB as Database (SURAT_JALAN, ARTIKEL)
    participant WebB as Web App Toko B
    actor TokoB as Toko Pemilik Stok (Outlet B)

    Note over TokoA,WebA: TAHAP 1: CEK STOK & BUAT SURAT JALAN
    TokoA->>WebA: Buka "Stock Opname" -> Cek "Stok Toko Lain"
    WebA->>API: POST /api/stockopname/list_stok_toko { kode, user }
    API->>DB: SELECT * FROM ARTIKEL WHERE kode = ? GROUP BY user_id
    DB-->>API: List Outlet yang Memiliki Stok
    API-->>WebA: Tampilkan: Toko B memiliki stok 10 pcs
    
    TokoA->>WebA: Pilih Toko B & Input Qty Permintaan: 3 pcs
    TokoA->>WebA: Klik "Lanjutkan Permintaan Stok"
    WebA->>API: POST /api/stockopname/permintaan_stok { user: TokoA, toko: TokoB, items }
    API->>DB: INSERT INTO SURAT_JALAN (status = 1 / Menunggu)
    API->>DB: INSERT INTO SURAT_JALAN_DETAIL (artikel_id, qty: 3)
    DB-->>API: Surat Jalan Dibuat (No. Invoice)
    API-->>WebA: Berhasil Mengirim Permintaan ke Toko B

    Note over TokoB,WebB: TAHAP 2: APPROVAL OLEH TOKO B
    TokoB->>WebB: Buka Menu "List Surat Jalan" (Tipe 2 / Permintaan Masuk)
    WebB->>API: POST /api/stockopname/list_bagi_stok { user: TokoB, tipe: 2 }
    API->>DB: SELECT * FROM SURAT_JALAN WHERE user_untuk_id = TokoB
    DB-->>API: Daftar Permintaan Masuk dari Toko A
    API-->>WebB: Tampilkan Daftar Permintaan

    alt Toko B Menyetujui (Approve)
        TokoB->>WebB: Klik "Terima Permintaan"
        WebB->>API: POST /api/stockopname/acc_bagi_stok { id, status: 3 }
        API->>DB: 1. UPDATE SURAT_JALAN SET status = 3 (Diterima)
        API->>DB: 2. UPDATE ARTIKEL Toko B (stok = stok - 3)
        API->>DB: 3. UPDATE ARTIKEL Toko A (stok = stok + 3)
        DB-->>API: Mutasi Stok Berhasil
        API-->>WebB: Toast: "Permintaan Stok Diterima & Stok Berpindah"
    else Toko B Menolak (Reject)
        TokoB->>WebB: Klik "Tolak Permintaan"
        WebB->>API: POST /api/stockopname/acc_bagi_stok { id, status: 2 }
        API->>DB: UPDATE SURAT_JALAN SET status = 2 (Ditolak)
        DB-->>API: Status Diperbarui
        API-->>WebB: Toast: "Permintaan Ditolak"
    end
```

---

### Diagram 8: Sequence Diagram — Alur Pengeluaran Kas & Refund Transaksi

```mermaid
sequenceDiagram
    autonumber
    actor Kasir as Kasir Toko
    participant Web as Layar Pengeluaran (PengeluaranScreen)
    participant API as Backend Service (/api/pengeluaran)
    participant DB as Database (PENGELUARAN, TRANSAKSI)

    Kasir->>Web: Buka Menu "Pengeluaran Lain"
    Kasir->>Web: Pilih Kategori Pengeluaran

    alt Kategori 1, 3, 4, 5 (Biaya Harian / Lainnya / Transfer / Setoran)
        Kasir->>Web: Input Nama Pengeluaran ("Beli Plastik") + Nominal (Rp 25.000)
    else Kategori 2 (Refund Transaksi)
        Kasir->>Web: Cari Nomor Struk Transaksi Asli
        Web->>API: GET /api/transaksi/list?kode=...
        API->>DB: SELECT * FROM TRANSAKSI WHERE kode_struk = ?
        DB-->>API: Data Transaksi Asli
        API-->>Web: Tampilkan Rincian Barang yang Diretur
        Kasir->>Web: Input Nominal Refund & Alasan Keterangan
    end

    Kasir->>Web: Klik "Simpan Pengeluaran"
    Web->>API: POST /api/pengeluaran/store_pengeluaran { user, nama, tipe, nominal, transaksi, keterangan }
    API->>DB: INSERT INTO PENGELUARAN (user_id, nama, tipe, nominal, transaksi_id, tanggal)
    DB-->>API: Pengeluaran Tersimpan
    API-->>Web: Response 200 OK
    Web-->>Kasir: Toast: "Pengeluaran Berhasil Dicatat" (Otomatis masuk pengurang Closing Dana)
```

---

### Diagram 9: Sequence Diagram — Alur Closing Dana Shift Kasir Harian

```mermaid
sequenceDiagram
    autonumber
    actor Kasir as Kasir Toko
    participant Web as Layar Closing (ClosingDanaScreen)
    participant API as Backend Service (/api/closingdana)
    participant DB as Database (CLOSING_DANA, TRANSAKSI, PENGELUARAN)

    Kasir->>Web: Buka Menu "Closing Dana"
    Web->>API: GET /api/closingdana/closing_now?user=...
    API->>DB: Cek apakah tanggal hari ini sudah ditutup di CLOSING_DANA
    
    alt Toko Sudah Closing Hari Ini
        DB-->>API: Ada record closing hari ini
        API-->>Web: Response 404 (Toko Sudah Tutup)
        Web-->>Kasir: Tombol Nonaktif: "Sudah Closing Dana Hari Ini"
    else Belum Closing
        DB-->>API: Belum ada closing hari ini
        API-->>Web: Response 200 OK (Boleh Closing)
        
        Web->>API: GET /api/closingdana/detail?user=...&tanggal=TODAY
        API->>DB: 1. Ambil Kas Awal
        API->>DB: 2. Hitung Omset (Retail + Tempo + Keep Hari Ini)
        API->>DB: 3. Ambil Total Pengeluaran Hari Ini
        DB-->>API: Data Ringkasan Kas
        API->>API: Hitung Kas Akhir = Kas Awal + Omset - Total Pengeluaran
        API-->>Web: Tampilkan Rincian Buku Kas Harian & Kas Akhir
        
        Kasir->>Web: Hitung Fisik Uang di Laci Kasir -> Klik "Closing Dana"
        Web->>Web: Dialog Konfirmasi: "Apakah anda yakin ingin menutup dana ini?"
        Kasir->>Web: Konfirmasi "Ya"
        Web->>API: POST /api/closingdana/store { user, kas_awal, kas_akhir }
        API->>DB: INSERT INTO CLOSING_DANA (user_id, tanggal, kas_awal, kas_akhir)
        API->>DB: INSERT INTO CLOSING_DANA_DETAIL (line item rincian pendapatan & pengeluaran)
        DB-->>API: Closing Sukses
        API-->>Web: Response 200 OK
        Web-->>Kasir: Toast: "Closing Dana Berhasil" -> Pilihan Export Laporan PDF
    end
```

---

### Diagram 10: Sequence Diagram — Alur Cetak Struk Thermal Bluetooth ESC/POS

```mermaid
sequenceDiagram
    autonumber
    actor Kasir as Kasir Toko
    participant Web as Web App (GenerateScreen / PrintData)
    participant BT as Web Bluetooth API (navigator.bluetooth)
    participant Printer as Printer Thermal ESC/POS (58mm / 80mm)

    Kasir->>Web: Selesai Transaksi -> Klik "Cetak Struk"
    Web->>BT: Request Device (Filter Service: 000018f0 / Bluetooth SPP)
    BT-->>Kasir: Browser Native Dialog: Pilih Printer Bluetooth
    Kasir->>BT: Pilih Perangkat Printer -> Connect
    BT-->>Web: GATT Server Connected & Write Characteristic Ready

    Web->>Web: Encode Raw Bytes ESC/POS Buffer:
    Note over Web: 1. ESC @ (Init Printer)<br/>2. GS v 0 (Print Raster Image Logo Base64)<br/>3. ESC a 1 (Align Center) -> Print Kop Toko<br/>4. Print Separator '===================='<br/>5. Print Invoice ID & Tanggal<br/>6. ESC a 0 (Align Left) -> Loop Print [Qty, Nama, Harga, Total]<br/>7. Print Subtotal, Diskon, Total, Bayar, Kembalian<br/>8. Print Footer Ucapan Terima Kasih<br/>9. GS V 66 0 (Paper Cut / Feed Line)

    Web->>BT: writeValue(escpos_bytes_buffer)
    BT->>Printer: Kirim Data Raw Byte Stream
    Printer-->>Kasir: Struk Fisik Kasir Tercetak
```

---

# SECTION 2: MAPPING ADOPSI TEKNIS (ANDROID $\rightarrow$ WEB-BASED)

| Fitur / Komponen | Android Existing | Solusi Web-Based Baru |
| :--- | :--- | :--- |
| **Framework** | React Native 0.76.2 | Next.js 14 App Router (React 18, TypeScript) |
| **Styling & UI** | StyleSheet native | Tailwind CSS + Shadcn UI (Responsive Dual-Panel POS) |
| **Autentikasi** | Parameter plain `user: username` di body/params | Header `Authorization: Bearer <JWT>` + Secure Cookies |
| **Validasi Kalkulasi**| 100% dihitung di aplikasi HP | Di-recalculate dan divalidasi ketat di backend server |
| **Barcode Scanner** | Kamera HP (`react-native-camera-kit`) | Scanner Fisik USB (`onKeyDown` Enter listener) & Webcam (`html5-qrcode`) |
| **Thermal Printing**| Native SPP (`react-native-bluetooth-escpos-printer`)| Web Bluetooth API & CSS `@media print` 58mm/80mm |
| **Export PDF** | Local library `react-native-html-to-pdf` | Client-side export via `jspdf` & `html2canvas` |
| **Database** | Database lama (PHP backend) | **PostgreSQL** dengan **Prisma ORM** (10 tabel terstruktur) |
| **Dashboard** | Belum ada | **Dashboard Analytics Baru:** Grafik tren omset, top 10 produk, notifikasi stok |

---

# SECTION 3: ROADMAP IMPLEMENTASI (5 SPRINT)

* **Sprint 1 (Minggu 1):** Setup Project Next.js 14, Database PostgreSQL, Prisma ORM Schema, Autentikasi JWT & Role Guard.
* **Sprint 2 (Minggu 2):** Layar Kasir POS Dual-Panel, Barcode Scanner USB, Keranjang, Multi-Payment, Cetak Struk Bluetooth.
* **Sprint 3 (Minggu 3):** Riwayat Transaksi, Pelunasan Angsuran Piutang Grosir (Jatuh Tempo/Keep), Pengeluaran & Refund.
* **Sprint 4 (Minggu 4):** Stock Opname, Cek Stok Cabang, Surat Jalan Antar-Toko (Approve/Reject), Closing Kasir & PDF.
* **Sprint 5 (Minggu 5):** Dashboard Analytics & KPI Toko, Pengujian Hardware End-to-End, UAT & Deployment Staging.
