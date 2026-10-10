# MOTAAPP DESIGN SYSTEM & STYLE GUIDE
## Editorial Lookbook & Ledger Atelier (Anti-Slop Craftsmanship Standard)
**Panduan Identitas Visual, Tipografi, Palet Warna, dan Komponen Bebas AI Slop untuk MotaApp**

---

### 1. Brand Essence & Karakter Desain

MotaApp adalah platform ERP operasional untuk konveksi pakaian, pergudangan kain, dan jaringan ritel/grosir tekstil Tanah Abang milik **Pak Abrar**.

Mengacu pada prinsip **Anti-Slop (R-01 s/d R-38)**, desain MotaApp menolak keras template visual generik bawaan AI (tanpa neon purple-blue gradient, tanpa bola cahaya radial glow, dan tanpa excessive frosted glassmorphism).

Prinsip desain utama:
* **The Sartorial Atelier:** Tampilan yang berwibawa, bersih, dan elegan seperti katalog busana (*fashion lookbook*) dan lembar kerja pemotongan pola kain fisik.
* **The Merchant Ledger:** Struktur tabel, angka, dan kartu mengadopsi ketegasan buku kas besar (*accounting ledger book*) dengan garis batas presisi (*hairline border 1px*).
* **Tactile Authenticity:** Elemen metadata ditampilkan menyerupai label pakaian fisik (*fabric woven labels & garment spec tags*), bukan kapsul warna-warni mengambang.

---

### 2. Palet Warna (Color Palette)

| Kategori Warna | Kode Hex | Nama Nuansa | Karakter & Kegunaan |
| :--- | :---: | :--- | :--- |
| **Page Canvas** | `#F6F5F0` | Archival Linen | Latar belakang panggung utama; hangat dan nyaman di mata (anti-silau). |
| **Surface Card** | `#FFFFFF` | Cotton Crisp | Permukaan kartu, lembar slide, dan modul data fisik solid (tanpa blur). |
| **Primary Ink** | `#1C1917` | Deep Tailor Charcoal | Teks utama, judul tebal, dan elemen hierarki tertinggi dengan kontras tajam. |
| **Secondary Text**| `#57534E` | Architectural Stone | Teks keterangan, paragraf deskriptif, dan label sekunder. |
| **Hairline Border**| `#E7E5E4` | Tailor Seam | Garis batas tipis 1px pemisah kartu dan kolom tabel (tegas dan terukur). |
| **Stitch Ochre** | `#B45309` | Tailor Gold / Thread | Aksen utama benang jahit, kartu procurement, dan sorotan harga turnkey. |
| **Ledger Forest** | `#15803D` | Bookkeeper Emerald | Indikator kesehatan finansial, gross margin riil, dan status pelunasan kasir. |
| **Denim Navy** | `#1E3A8A` | Indanthrene Indigo | Aksen alur dokumen legal (PR/PO) dan serah terima hak cipta source code. |
| **Stuck Crimson** | `#BE123C` | Alert Red | Alarm visual khusus untuk PO konveksi macet (>14 hari) dan stok kritis. |

---

### 3. Tipografi (Typography)

* **Headings & Judul:** `Plus Jakarta Sans` (Weight 700–900, tight tracking `-0.02em`, tegas tanpa gradien neon).
* **Body Text & Keterangan:** `Plus Jakarta Sans` / `Inter` (Weight 400–500, line-height proporsional untuk kenyamanan membaca).
* **Kutipan & Konteks Naratif:** `Newsreader` (Serif italic untuk memberikan sentuhan *editorial craftsmanship* pada kutipan masalah lapangan).
* **Angka Moneter & Barcode:** `JetBrains Mono` (Font monospace untuk angka rupiah, kuantitas pack, nomor PO, dan HPP).

---

### 4. Hirarki Komponen Visual Bebas Slop

1. **Specimen Tags (Pengganti Pill Badge AI):**  
   Menggunakan bentuk label kotak bersudut minimal (`border-radius: 4px`), border hairline, font monospace, dan latar warna pastel lembut:
   * Ochre: `bg-[#FEF3C7] text-[#92400E] border-[#FDE68A]`
   * Forest: `bg-[#DCFCE7] text-[#14532D] border-[#BBF7D0]`
   * Crimson: `bg-[#FFE4E6] text-[#9F1239] border-[#FECDD3]`

2. **Editorial Cards (Pengganti Glassmorphism):**  
   Permukaan padat (*solid `#FFFFFF`*) dengan border tipis `1px solid #E7E5E4`, aksen warna pada garis tepi kiri (*`border-l-4`*), dan bayangan mikro `0 1px 2px rgba(28,25,23,0.03)`. Menghindari efek tembus pandang atau blur yang membuat mata cepat lelah.

3. **Ledger Table & Heatmap:**  
   Header kolom tegas dengan teks kapital kecil, baris pemisah horizontal tipis, dan sel kalender berwarna solid pastel terukur tanpa efek glow pudar.
