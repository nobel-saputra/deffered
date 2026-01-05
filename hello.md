# Laporan Implementasi Deep Link & Deferred Deep Link

**Mata Kuliah: Pemrograman Mobile**

## 1. Deskripsi Tugas

Mahasiswa diminta untuk mengimplementasikan fitur **Deep Link** dan **Deferred Deep Link** pada aplikasi Expo, serta menjelaskan alur kerja dan skenario penggunaannya.

---

## 2. Implementasi Deep Link (Berhasil)

### Alur Kerja

Fitur ini memungkinkan aplikasi dibuka langsung ke halaman spesifik melalui URI scheme `cookmaster://`.

1.  **Konfigurasi Scheme**:
    Pada `app.json`, scheme diset menjadi `cookmaster`.
    ```json
    { "scheme": "cookmaster" }
    ```
2.  **Routing Handler**:
    `Expo Router` secara otomatis menangani parsing URL.
    - URL `cookmaster://resep/2b56...` akan otomatis dipetakan ke route `app/resep/[id].tsx`.
    - Parameter `id` diambil dan digunakan untuk fetch data ke Supabase.

### Skenario Penggunaan

- **Kasus**: Seorang pengguna ingin membagikan resep "Nasi Goreng" kepada temannya via WhatsApp.
- **Aksi**: Teman mengklik link `cookmaster://resep/123`.
- **Hasil**: Aplikasi CookMaster teman langsung terbuka dan menampilkan halaman detail Nasi Goreng (bukan halaman Beranda).

### Bukti Pengujian

Perintah yang dijalankan pada emulator:

```bash
npx uri-scheme open cookmaster://resep/2b56019b-d6b6-4fd5-a250-433c9ed0799b --android
```

**Hasil**: Aplikasi berhasil terbuka langsung di halaman detail resep yang dimaksud.

---

## 3. Konsep Deferred Deep Link

### Penjelasan & Alur Kerja

Deferred Deep Link adalah mekanisme untuk mempertahankan konteks link meskipun pengguna **belum menginstal aplikasi** saat link diklik.

1.  **Klik Link**: Pengguna mengklik link promo resep di web/sosmed.
2.  **Redirect Store**: Karena aplikasi belum ada, pengguna diarahkan ke Play Store/App Store.
3.  **Instalasi**: Pengguna mengunduh dan menginstal aplikasi.
4.  **Context Preservation**: Layanan Deep Link (seperti Branch.io/Firebase) atau mekanisme Clipboard menyimpan data link tersebut sementara.
5.  **First Open**: Saat aplikasi pertama kali dibuka setelah instalasi, aplikasi mengecek "Context" tersebut.
6.  **Navigasi Otomatis**: Aplikasi langsung mengarahkan pengguna ke resep yang tadi diklik di web, melewati halaman onboarding/home biasa.

### Skenario Penggunaan

- **Kasus**: Influencer membagikan link referral "Diskon Spesial Pengguna Baru" di Instagram Story.
- **Aksi**: Follower (yang belum punya app) mengklik link -> Install App.
- **Hasil**: Begitu app dibuka, follower langsung melihat halaman klaim diskon tersebut, tanpa perlu mencari-cari lagi link-nya.

_(Catatan: Implementasi penuh membutuhkan layanan pihak ketiga seperti Branch/AppsFlyer, simulasi dapat dilakukan menggunakan Clipboard API namun untuk tugas ini difokuskan pada pemahaman konsep)._
