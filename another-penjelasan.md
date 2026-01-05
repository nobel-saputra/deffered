# 📱 Laporan Implementasi Deep Link & Deferred Deep Link

**Aplikasi: CookMaster**  
**Nama: Nobel Saputra**  
**Mata Kuliah: Pemrograman Mobile**

---

## 1. Pendahuluan

Laporan ini menjelaskan implementasi **Deep Link** dan **Deferred Deep Link** pada aplikasi mobile CookMaster yang dibangun menggunakan Expo (React Native). Kedua fitur ini bertujuan untuk meningkatkan user experience dengan memungkinkan navigasi langsung ke konten spesifik melalui URL.

---

## 2. Deep Link (✅ Berhasil Diimplementasikan)

### 2.1 Definisi

Deep Link adalah URL yang membuka aplikasi mobile dan mengarahkan user langsung ke halaman tertentu, bukan halaman utama.

### 2.2 Konfigurasi

**File: `app.json`**

```json
{
  "expo": {
    "scheme": "cookmaster"
  }
}
```

Custom scheme `cookmaster://` didaftarkan agar sistem operasi mengenali link tersebut sebagai milik aplikasi CookMaster.

### 2.3 Implementasi Routing

Expo Router secara otomatis menangani deep link dengan struktur file-based routing:

- URL: `cookmaster://resep/2b56019b-...`
- Route: `app/resep/[id].tsx`
- Parameter `id` otomatis di-extract dan bisa diakses via `useLocalSearchParams()`

**File: `app/resep/[id].tsx`** (snippet)

```typescript
const { id } = useLocalSearchParams<{ id: string }>();

useEffect(() => {
  const fetchData = async () => {
    const data = await findResepById(id);
    setResep(data);
  };
  fetchData();
}, [id]);
```

### 2.4 Workflow Deep Link

```
┌─────────────────────┐
│ User klik link:     │
│ cookmaster://resep/ │
│ 2b56019b-...        │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ OS deteksi scheme   │
│ "cookmaster://"     │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Buka aplikasi       │
│ CookMaster          │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Expo Router parse   │
│ path "/resep/..."   │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Render halaman      │
│ detail resep        │
│ dengan id tersebut  │
└─────────────────────┘
```

### 2.5 Skenario Penggunaan

**Kasus 1: Share Resep via WhatsApp**

- User A membuka resep "Nasi Goreng Spesial" di aplikasi
- User A share link `cookmaster://resep/abc123` ke User B via WhatsApp
- User B klik link → Aplikasi CookMaster terbuka langsung di halaman Nasi Goreng

**Kasus 2: Notifikasi Push**

- Server mengirim push notification "Resep baru telah ditambahkan!"
- Payload berisi deep link `cookmaster://resep/xyz789`
- User tap notifikasi → Langsung ke halaman resep baru tersebut

**Kasus 3: QR Code di Restoran**

- Restoran cetak QR Code yang link ke `cookmaster://resep/menu-spesial`
- Pengunjung scan QR → Buka aplikasi → Lihat resep menu spesial

### 2.6 Testing & Validasi

**Command Testing:**

```bash
npx uri-scheme open cookmaster://resep/2b56019b-d6b6-4fd5-a250-433c9ed0799b --android
```

**Hasil:**
✅ Aplikasi terbuka  
✅ Navigasi langsung ke halaman detail resep  
✅ Data resep berhasil di-fetch dari Supabase  
✅ UI menampilkan informasi resep dengan benar

---

## 3. Deferred Deep Link (Konsep & Implementasi Teoritis)

### 3.1 Definisi

Deferred Deep Link adalah mekanisme yang mempertahankan konteks link bahkan ketika user **belum memiliki aplikasi** saat link diklik. Setelah instalasi, aplikasi akan otomatis membuka konten yang dituju.

### 3.2 Perbedaan dengan Deep Link Biasa

| Aspek            | Deep Link                | Deferred Deep Link                         |
| ---------------- | ------------------------ | ------------------------------------------ |
| **Prasyarat**    | App sudah terinstall     | App belum terinstall                       |
| **Flow**         | Link → Buka App → Konten | Link → Store → Install → Buka App → Konten |
| **Persistence**  | Tidak perlu              | Data link disimpan selama proses install   |
| **Kompleksitas** | Sederhana                | Butuh backend/SDK pihak ketiga             |

### 3.3 Workflow Deferred Deep Link

```
┌──────────────────────┐
│ User klik link       │
│ (App belum install)  │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Smart Link Service   │
│ (Branch.io/Firebase) │
│ - Deteksi app status │
│ - Simpan context     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Redirect ke Store    │
│ (Play Store/         │
│  App Store)          │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ User install app     │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ First app launch     │
│ - SDK cek context    │
│ - Ambil data link    │
└──────────┬───────────┘
           │
           ▼
┌──────────────────────┐
│ Auto-navigate ke     │
│ konten yang dituju   │
└──────────────────────┘
```

### 3.4 Skenario Penggunaan

**Kasus 1: Kampanye Marketing**

- Influencer share link promo `https://cookmaster.app/promo/diskon50`
- Follower (belum punya app) klik link
- Redirect ke Play Store → Install
- App terbuka pertama kali → Langsung ke halaman klaim diskon

**Kasus 2: User Referral**

- User A invite User B dengan link referral
- User B install app
- Setelah registrasi, otomatis masuk halaman "Terima kasih dari [User A]" dengan bonus poin

**Kasus 3: Content Discovery**

- Artikel blog embed link resep
- Pembaca baru klik link → Install app
- Langsung bisa baca resep yang dimaksud tanpa perlu cari manual

### 3.5 Teknologi yang Dibutuhkan

#### Opsi 1: Branch.io (Recommended untuk Produksi)

- **Kelebihan**:
  - Attribution tracking lengkap
  - Support iOS & Android
  - Analytics dashboard
  - Gratis untuk basic features
- **Kekurangan**:
  - Butuh setup SDK
  - Perlu account Branch.io

#### Opsi 2: Firebase Dynamic Links

- **Kelebihan**:
  - Terintegrasi dengan Firebase ecosystem
  - Gratis
- **Kekurangan**:
  - Deprecated (akan dihapus 2025)
  - Migrasi ke Firebase App Links

#### Opsi 3: Manual Implementation (Clipboard/Cookie)

- **Kelebihan**:
  - Tidak butuh SDK eksternal
  - Cocok untuk demo/prototype
- **Kekurangan**:
  - Tidak reliable di production
  - Tidak ada attribution tracking

### 3.6 Implementasi Teoritis dengan Clipboard

Untuk keperluan demo dan pemahaman konsep, simulasi Deferred Deep Link bisa dilakukan dengan:

1. **Web Landing Page** menyimpan recipe ID ke clipboard
2. **Aplikasi** mengecek clipboard saat pertama dibuka
3. Jika ada magic string `COOKMASTER_DEFER:[id]`, navigate ke halaman tersebut

**Pseudocode:**

```typescript
// Di app/_layout.tsx
useEffect(() => {
  const checkDeferredLink = async () => {
    const clipboard = await Clipboard.getStringAsync();

    if (clipboard.startsWith("COOKMASTER_DEFER:")) {
      const recipeId = clipboard.split(":")[1];
      await Clipboard.setStringAsync(""); // Clear
      router.push(`/resep/${recipeId}`);
    }
  };

  if (isFirstLaunch) {
    checkDeferredLink();
  }
}, []);
```

**Catatan:** Metode ini hanya untuk demonstrasi konsep. Untuk production, wajib gunakan Branch.io atau Firebase App Links.

---

## 4. Tantangan Implementasi

### 4.1 Deep Link

- ✅ **Solved**: Routing conflict dengan auth guard
  - **Solusi**: Kondisional redirect di `_layout.tsx` hanya untuk root path

### 4.2 Deferred Deep Link

- ⚠️ **Challenge**: Membutuhkan native build yang stabil
  - NDK corruption pada development environment
  - Gradle build failures
- ⚠️ **Challenge**: Dependency pada layanan pihak ketiga
  - Branch.io/Firebase memerlukan account & setup
- ⚠️ **Challenge**: Testing memerlukan uninstall-reinstall flow
  - Tidak praktis untuk rapid development

### 4.3 Lessons Learned

1. Deep Link sebaiknya diimplementasikan sejak awal arsitektur app
2. Deferred Deep Link memerlukan planning yang matang terkait analytics
3. Testing deep link memerlukan device fisik atau emulator dengan Google Play Services

---

## 5. Kesimpulan

### Ringkasan Implementasi

| Fitur                  | Status                      | Coverage |
| ---------------------- | --------------------------- | -------- |
| **Deep Link**          | ✅ Fully Implemented        | 100%     |
| **Deferred Deep Link** | 📚 Conceptual Understanding | Teoritis |

### Pencapaian

1. ✅ Deep Link berhasil diimplementasikan dan diuji
2. ✅ Routing dinamis dengan parameter berfungsi sempurna
3. ✅ Integrasi dengan Supabase untuk fetch data real-time
4. ✅ Pemahaman mendalam tentang workflow Deferred Deep Link
5. ✅ Dokumentasi lengkap untuk referensi masa depan

### Rekomendasi untuk Pengembangan Lanjutan

Jika aplikasi akan dipublikasikan ke production:

1. **Implementasi Branch.io** untuk Deferred Deep Link
2. **Setup Firebase Analytics** untuk tracking deep link performance
3. **Create Universal Links** (iOS) dan **App Links** (Android) untuk web fallback
4. **A/B Testing** berbagai strategi deep link untuk optimasi konversi
5. **Monitoring** deep link usage dengan dashboard analytics

---

## 6. Referensi

- [Expo Deep Linking Guide](https://docs.expo.dev/guides/deep-linking/)
- [Branch.io Documentation](https://help.branch.io/)
- [Android App Links](https://developer.android.com/training/app-links)
- [iOS Universal Links](https://developer.apple.com/ios/universal-links/)

---

**Tanggal Laporan:** 5 Januari 2026
