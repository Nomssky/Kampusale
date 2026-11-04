# AGENTS.md

Panduan untuk AI coding agent (GitHub Copilot, Claude Code, dan sejenisnya) serta anggota tim yang bekerja di repo **Kampusale**. Baca file ini sebelum mengubah kode. Untuk gambaran produk dan cara menjalankan proyek, lihat `README.md`.

Bahasa: dokumen ini berbahasa Indonesia. Kode, komentar, nama variabel, dan pesan commit ditulis dalam **bahasa Inggris**. Teks yang tampil di antarmuka aplikasi ditulis dalam **bahasa Indonesia**.

---

## 1. Ringkasan proyek

Kampusale adalah marketplace barang bekas khusus mahasiswa. Pengguna hanya melihat dan bertransaksi dengan barang dari **kampusnya sendiri**. Serah terima dilakukan langsung (tanpa pembayaran online) pada MVP ini.

Prinsip utama: **scope tetap sempit**. Kerjakan hanya fitur MVP di bawah ini.

**Fitur MVP:** autentikasi dan pilih kampus, unggah barang (1-5 foto), katalog dengan pencarian dan filter, detail barang, permintaan pesanan (terima/tolak/selesai/batal), dan pengungkapan nomor WhatsApp penjual setelah permintaan diterima.

**Di luar cakupan (jangan dibuat):** pembayaran, escrow, komisi, chat dalam aplikasi, notifikasi push, rating dan ulasan, dashboard admin, aplikasi mobile, multi-bahasa.

## 2. Teknologi

- Next.js (App Router), TypeScript, Tailwind CSS
- Supabase (Postgres, Auth, Storage) dengan Prisma sebagai ORM
- Zod untuk validasi, Vitest untuk pengujian
- ESLint dan Prettier, GitHub Actions untuk CI

## 3. Perintah

> Sesuaikan dengan `package.json` bila berbeda.

```bash
npm install            # pasang dependensi
npm run dev            # server pengembangan
npm run lint           # cek gaya kode
npm run typecheck      # cek tipe TypeScript
npm test               # jalankan Vitest
npx prisma migrate dev # migrasi database lokal
npx prisma db seed     # isi data contoh
```

Sebelum menyatakan pekerjaan selesai, jalankan `lint`, `typecheck`, dan `test`. Ketiganya harus lolos.

## 4. Struktur dan tempat menaruh kode

```
app/          route dan halaman
components/   komponen UI
lib/          klien Supabase, Prisma, validator Zod
server/       logika bisnis (items.service.ts, orders.service.ts)
prisma/       schema.prisma dan seed.ts
tests/        pengujian Vitest
docs/         dokumentasi produk, QA, dan teknis (dibuat bila diperlukan)
.github/      workflow CI dan template PR
```

Aturan penempatan:

- **Logika bisnis ada di `server/`**, bukan di komponen atau halaman.
- Transisi status pesanan ditulis sebagai **satu fungsi murni** agar mudah diuji.
- Validasi input dengan Zod di `lib/`, dipakai ulang oleh server dan form.

## 5. Konvensi kode

- TypeScript `strict`. Hindari `any`; bila terpaksa, beri komentar alasannya.
- Komponen kecil dan fokus. Jangan menaruh query database di komponen.
- Mobile-first: setiap halaman harus nyaman di layar HP, dan punya state loading, kosong, dan error.
- Nama berkas: komponen `PascalCase.tsx`, selain itu `kebab-case.ts`.
- Jangan menambah dependensi baru tanpa alasan jelas. Sebutkan alasannya di deskripsi PR.
- Jangan menyalin gaya atau aset dari merek lain. Gunakan desain dari Figma tim.

## 6. Aturan bisnis (tidak boleh dilanggar)

Dijalankan di **server**, bukan hanya di UI.

- Status barang: `AVAILABLE` → `RESERVED` (permintaan diterima) → `SOLD` (selesai). Jika permintaan yang diterima batal atau ditolak, barang kembali `AVAILABLE`.
- Maksimal satu pesanan `ACCEPTED` per barang.
- Penjual tidak bisa memesan barangnya sendiri. Pembeli tidak bisa mengajukan permintaan ganda untuk barang yang sama selagi aktif.
- Hanya penjual yang boleh mengubah atau menghapus barangnya serta menerima, menolak, dan menyelesaikan pesanan. Hanya pembeli yang boleh membatalkan pesanannya sendiri.
- Pengguna hanya bisa membaca dan menulis data **kampusnya sendiri**.
- Nomor WhatsApp penjual hanya dikirim ke pembeli setelah pesanan berstatus `ACCEPTED`.

## 7. Keamanan dan privasi

- Jangan pernah meng-commit rahasia. Pakai `.env.local`, dan perbarui `.env.example` bila menambah variabel.
- `SUPABASE_SERVICE_ROLE_KEY` hanya boleh dipakai di kode server.
- Periksa otorisasi di setiap mutasi dan validasi semua input dengan Zod.
- Batasi unggahan foto: maksimal 2 MB, hanya jpg, png, dan webp.
- Jangan mencatat data pribadi (nomor WhatsApp, email) ke log.
- Jangan mengirim data pengguna ke layanan pihak ketiga tanpa persetujuan tim.

## 8. Definition of done

Sebuah pekerjaan dianggap selesai bila:

1. Sesuai kriteria penerimaan (acceptance criteria) dari Product Owner.
2. `lint`, `typecheck`, dan `test` lolos.
3. Aturan bisnis di bagian 6 tetap terpenuhi, dengan pengujian untuk logika yang diubah.
4. Tampilan sudah dicek di ukuran layar HP dan laptop.
5. Dokumentasi (README, `docs/`) diperbarui bila perilaku berubah.
6. Pull request sudah di-review minimal satu orang.

## 9. Alur Git dan pull request

- Branch dari `main`: `feat/nama-fitur`, `fix/nama-masalah`, `docs/nama-topik`.
- Pesan commit dalam bahasa Inggris, singkat dan jelas (misalnya `feat: add item filter by price`).
- Satu PR untuk satu perubahan yang fokus. Jelaskan apa yang berubah, alasannya, dan cara mengujinya.
- Jangan push langsung ke `main`.

## 10. Peran tim dan kepemilikan area

Tim terdiri dari 5 orang dengan aturan **1 orang 1 peran**. Setiap peran dipegang tepat satu orang, dan pemegang peran itulah yang bertanggung jawab atas area-nya serta me-review perubahan di dalamnya.

| # | Peran | Penanggung jawab | Fokus | Area utama |
|---|---|---|---|---|
| 1 | **UI/UX Designer & Product Owner** | @affelgato | Riset pengguna, user flow, wireframe hingga mockup (Figma), prioritas fitur | `docs/product/`, desain di Figma, kriteria penerimaan |
| 2 | **Frontend Developer** | @rakhadwi14 | Mengubah desain menjadi antarmuka yang responsif dan terhubung ke data backend | `app/`, `components/`, gaya Tailwind |
| 3 | **Backend Developer** | @Nomssky | API, basis data, autentikasi, dan keamanan | `server/`, `prisma/`, `lib/`, route handler, aturan akses Supabase |
| 4 | **QA Tester & Technical Writer** | @caesaria | Test case, uji sebelum rilis, dokumentasi pengguna dan teknis | `tests/`, `docs/qa/`, `README.md`, `docs/` |
| 5 | **Project Manager / Scrum Master & Fullstack** | Farhan (username GitHub menyusul, undangan masih pending) | Jadwal, daily standup, menghilangkan hambatan, membantu coding bila menumpuk | `.github/`, papan proyek |

Peran nomor 5 boleh membantu coding di area Frontend atau Backend bila ada tugas yang menumpuk, tetapi perubahannya tetap di-review oleh pemilik area tersebut (peran 2 atau 3).

### Aturan review lintas peran

| Perubahan menyentuh | Wajib di-review oleh |
|---|---|
| `prisma/schema.prisma`, migrasi, autentikasi, aturan akses data | Backend Developer |
| Tampilan, alur halaman, teks antarmuka | UI/UX Designer & Product Owner (cek kesesuaian dengan desain) |
| Komponen dan halaman | Frontend Developer |
| `tests/`, dokumentasi | QA Tester & Technical Writer |
| `.github/`, konfigurasi CI | Project Manager / Scrum Master & Fullstack |

## 11. Cara agen AI bekerja di repo ini

**Lakukan tanpa bertanya:**
- Memperbaiki bug yang jelas, menambah pengujian, merapikan kode dalam area yang sedang dikerjakan.
- Menambah dokumentasi untuk perubahan yang dibuat.

**Tanyakan dulu (buat catatan di PR atau issue):**
- Mengubah skema database, aturan bisnis di bagian 6, atau alur autentikasi.
- Menambah dependensi baru atau mengubah konfigurasi CI.
- Mengubah tampilan yang tidak ada di desain Figma, atau menambah fitur di luar daftar MVP.

**Jangan pernah:**
- Meng-commit rahasia atau menonaktifkan pemeriksaan otorisasi demi mempercepat pekerjaan.
- Menghapus atau melemahkan pengujian supaya CI lolos.
- Membuat fitur di luar cakupan (pembayaran, chat, notifikasi, dan sebagainya).
- Mengarang fakta tentang pengguna, data, atau hasil uji. Jika tidak yakin, tulis asumsi dengan jelas di PR.

Setiap PR dari agen harus menyebut: ringkasan perubahan, asumsi yang diambil, hal yang belum teruji, dan peran yang sebaiknya me-review.
