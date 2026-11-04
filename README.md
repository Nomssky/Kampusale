# Kampusale

> Barang bekas kating, hemat buat kamu.

**Kampusale** (kampus + sale) adalah marketplace barang bekas khusus mahasiswa. Barang yang masih layak pakai berpindah dari satu mahasiswa ke mahasiswa lain **di kampus yang sama**, dengan serah terima langsung di area kampus atau kos.

Status: **MVP dalam pengembangan**.

---

## Latar belakang

Mahasiswa sering kesulitan menemukan barang bekas layak pakai dari sesama mahasiswa, karena informasinya tersebar di grup WhatsApp, Instagram Story, dan lingkaran pertemanan. Di sisi lain, banyak mahasiswa (terutama tingkat akhir atau yang akan pindah kos) punya barang yang sudah tidak terpakai tetapi masih layak dijual.

Kampusale menyatukan keduanya dalam satu platform per kampus.

## Fitur MVP

- **Autentikasi** dengan email dan password. Saat daftar, pengguna memilih kampus, dan hanya melihat barang dari kampusnya sendiri.
- **Unggah barang**: judul, deskripsi, harga (IDR), kategori, kondisi, lokasi serah terima, dan 1-5 foto (maks. 2 MB per foto; jpg, png, webp).
- **Katalog** dengan pencarian judul, filter kategori dan rentang harga, serta urutan terbaru atau termurah.
- **Detail barang**: foto, harga, kondisi, deskripsi, lokasi serah terima, dan nama penjual.
- **Permintaan pesanan**: pembeli mengajukan permintaan (dengan catatan opsional), penjual menerima, menolak, atau menandai selesai. Pembeli dapat membatalkan selama masih menunggu.
- **Kontak penjual**: nomor WhatsApp penjual baru terlihat oleh pembeli **setelah permintaan diterima** (tautan `wa.me`).

### Di luar cakupan MVP

Pembayaran, escrow, komisi, chat dalam aplikasi, notifikasi push, rating dan ulasan, dashboard admin, aplikasi mobile, dan dukungan multi-bahasa.

## Teknologi

| Lapisan | Teknologi |
|---|---|
| Framework | Next.js (App Router), TypeScript |
| Tampilan | Tailwind CSS |
| Database | PostgreSQL (Supabase) dengan Prisma ORM |
| Autentikasi | Supabase Auth |
| Penyimpanan foto | Supabase Storage |
| Validasi | Zod |
| Pengujian | Vitest |
| Kualitas kode | ESLint, Prettier, GitHub Actions |

## Struktur proyek

```
kampusale/
├── app/            # route dan halaman (Next.js App Router)
├── components/     # komponen UI
├── lib/            # klien Supabase, Prisma, validator Zod
├── server/         # logika bisnis (items.service.ts, orders.service.ts)
├── prisma/         # schema.prisma dan seed.ts
├── tests/          # pengujian Vitest
└── .github/        # workflow CI
```

Logika bisnis ditempatkan di `server/`, bukan di dalam komponen, supaya mudah diuji dan dirawat.

## Model data

- **Campus**: id, name, slug
- **User**: id, authId, name, whatsapp, campusId
- **Item**: id, sellerId, campusId, title, description, priceIdr, category, condition, pickupLocation, status
- **ItemImage**: id, itemId, url, position
- **Order**: id, itemId, buyerId, status, note

Enum utama:

| Enum | Nilai |
|---|---|
| Kategori | `BUKU_PERLENGKAPAN`, `ELEKTRONIK`, `PERLENGKAPAN_KOS`, `PAKAIAN_SEPATU_TAS` |
| Kondisi | `SEPERTI_BARU`, `BAIK`, `CUKUP` |
| Status barang | `AVAILABLE`, `RESERVED`, `SOLD` |
| Status pesanan | `PENDING`, `ACCEPTED`, `REJECTED`, `COMPLETED`, `CANCELLED` |

## Aturan bisnis

Aturan ini dijalankan di server, bukan hanya di antarmuka.

- Alur status barang: `AVAILABLE` → `RESERVED` (permintaan diterima) → `SOLD` (selesai). Jika permintaan yang diterima dibatalkan atau ditolak, barang kembali `AVAILABLE`.
- Hanya satu pesanan `ACCEPTED` per barang pada satu waktu.
- Penjual tidak bisa memesan barangnya sendiri, dan pembeli tidak bisa mengajukan permintaan ganda untuk barang yang sama selagi masih aktif.
- Hanya penjual yang boleh mengubah atau menghapus barangnya, serta menerima, menolak, dan menyelesaikan pesanan. Hanya pembeli yang boleh membatalkan pesanannya sendiri.
- Pengguna hanya dapat membaca dan menulis data kampusnya sendiri.

## Memulai

### Prasyarat

- Node.js 20 atau lebih baru
- Akun [Supabase](https://supabase.com) (paket gratis cukup)

### Langkah

1. **Clone dan pasang dependensi**

   ```bash
   git clone https://github.com/<username>/kampusale.git
   cd kampusale
   npm install
   ```

2. **Buat proyek Supabase**, lalu:
   - Buat bucket Storage bernama `item-images` (publik untuk baca).
   - Aktifkan Auth dengan metode email dan password.

3. **Siapkan variabel lingkungan**

   ```bash
   cp .env.example .env.local
   ```

   Isi nilai berikut dari dashboard Supabase:

   | Variabel | Keterangan |
   |---|---|
   | `NEXT_PUBLIC_SUPABASE_URL` | URL proyek Supabase |
   | `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Kunci anon (aman untuk klien) |
   | `SUPABASE_SERVICE_ROLE_KEY` | Kunci service role (**hanya server, jangan dibocorkan**) |
   | `DATABASE_URL` | Connection string (pooled) untuk aplikasi |
   | `DIRECT_URL` | Connection string langsung untuk migrasi Prisma |

4. **Jalankan migrasi dan seed**

   ```bash
   npx prisma migrate dev
   npx prisma db seed
   ```

   Seed membuat 3 kampus, 2 pengguna demo, dan sekitar 12 barang contoh.

5. **Jalankan aplikasi**

   ```bash
   npm run dev
   ```

   Buka http://localhost:3000.

> Nama skrip di bawah mengikuti konvensi umum. Sesuaikan dengan isi `package.json` hasil generate.

### Skrip

| Perintah | Fungsi |
|---|---|
| `npm run dev` | Menjalankan server pengembangan |
| `npm run build` | Build produksi |
| `npm run lint` | Memeriksa gaya kode |
| `npm run typecheck` | Memeriksa tipe TypeScript |
| `npm test` | Menjalankan pengujian Vitest |

## Pengujian

Pengujian mencakup mesin status pesanan (fungsi murni untuk semua transisi yang valid dan tidak valid) serta logika kueri harga dan filter.

```bash
npm test
```

GitHub Actions menjalankan lint, typecheck, dan pengujian pada setiap pull request.

## Deploy

Rekomendasi untuk MVP: **Vercel** untuk aplikasi dan **Supabase** untuk database, autentikasi, dan penyimpanan. Atur semua variabel lingkungan di dashboard Vercel, dan jalankan `npx prisma migrate deploy` pada database produksi.

## Kontribusi

1. Buat branch dari `main` (`feat/nama-fitur` atau `fix/nama-masalah`).
2. Pastikan `lint`, `typecheck`, dan `test` lolos sebelum membuka pull request.
3. Setiap pull request minimal mendapat satu review sebelum di-merge.
4. Jangan pernah meng-commit rahasia. Gunakan `.env.local` dan perbarui `.env.example` bila menambah variabel.

## Rencana ke depan

Belum dikerjakan di MVP, urutan masih bisa berubah setelah validasi ke pengguna:

- Pembayaran lewat platform (dibutuhkan agar komisi transaksi bisa ditagih)
- Promosi barang berbayar
- Verifikasi pengguna (misalnya email kampus)
- Notifikasi dan rating penjual

## Lisensi

Belum ditentukan.
