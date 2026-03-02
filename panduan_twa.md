# Panduan Lengkap Verifikasi Website TWA (Trusted Web Activity)

Agar aplikasi Android Anda (TWA) dapat percaya penuh pada website Anda `client.pass.net.id` dan **menghilangkan *address bar* (URL bar)** di bagian atas, Anda perlu mengonfigurasi Digital Asset Links (DAL).

Berikut adalah langkah-langkah lengkap dari awal hingga pengaturan untuk publikasi ke Google Play Store:

## Langkah 1: Persiapan File Konfigurasi

File konfigurasi yang bernama `assetlinks.json` sudah dibuatkan di lokasi berikut:
➔ `D:\Dari Desktop\Kiro\twa_myclientid\assetlinks.json`

File ini berisi relasi yang menyatakan bahwa aplikasi Android Anda (dengan nama paket `com.passnet.myclientid`) memiliki izin penuh untuk menangani URL dari website Anda.

Isi file saat ini:
```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.passnet.myclientid",
    "sha256_cert_fingerprints": [
      "A7:C3:03:AF:FC:E6:25:8A:BE:4F:8C:3D:1A:B2:E9:D1:23:3D:A1:6B:DD:39:0A:A5:D1:97:B9:5E:1A:79:1B:61"
    ]
  }
}]
```
> **Catatan:** Fingerprint `A7:C3:03...` ini adalah kunci versi *Debug* dari komputer Anda. Kunci ini memungkinkan Anda untuk mengetes aplikasi langsung di HP via kabel USB (Run/Debug di Android Studio) tanpa muncul address bar.

## Langkah 2: Upload File ke Hosting

Anda harus meletakkan file tersebut di server hosting website agar aplikasi Android bisa memverifikasinya.

1. Buka **cPanel**, **File Manager**, atau gunakan **FTP** untuk mengakses server web `client.pass.net.id`.
2. Masuk ke *root directory* publik website Anda (biasanya bernama `public_html`, `htdocs`, atau `www`).
3. Buat folder baru (jika belum ada) dan beri nama **`.well-known`**.
   *(Perhatikan tanda titik `.` di depan nama foldernya).*
4. Upload/salin file `assetlinks.json` ke dalam folder `.well-known` tersebut.

Struktur folder akhir di hosting Anda harus seperti ini:
```text
/public_html
 └── .well-known/
     └── assetlinks.json
```

## Langkah 3: Verifikasi Instalasi di Server

Untuk memastikan file sudah terpasang dan bisa diakses secara publik, buka browser Anda dan kunjungi URL berikut:
👉 `https://client.pass.net.id/.well-known/assetlinks.json`

Jika browser menampilkan teks kode JSON yang sama persis seperti di Langkah 1, berarti pengaturannya **berhasil**.

## Langkah 4: Pengaturan Saat Upload ke Google Play Store

Karena Anda menggunakan format `.aab` dan Google Play Console akan mengaktifkan fitur **Play App Signing** (Google akan menandatangani ulang aplikasi Anda dengan sertifikat mereka), maka Anda **harus menambahkan** satu fingerprint lagi khusus dari Google Play.

Langkahnya:
1. Setelah aplikasi berhasil di-upload ke **Google Play Console**, buka dashboard aplikasi Anda.
2. Masuk ke menu **Release** > **Setup** > **App Integrity** (atau tab **App Signing**).
3. Di bagian **App signing key certificate**, temukan dan *copy* kode pada baris **`SHA-256 certificate fingerprint`**.
4. Kembali ke *File Manager hosting* Anda, edit file `assetlinks.json` yang ada di dalam folder `.well-known`.
5. Sisipkan kode SHA-256 dari Play Store tersebut ke dalam daftar `sha256_cert_fingerprints` (gunakan tanda koma pemisah).

Contoh hasil akhirnya (ganti "`KODE_SHA256_DARI_PLAY_CONSOLE_ANDA_DI_SINI`" dengan kode asli dari Play Console):

```json
[{
  "relation": ["delegate_permission/common.handle_all_urls"],
  "target": {
    "namespace": "android_app",
    "package_name": "com.passnet.myclientid",
    "sha256_cert_fingerprints": [
      "A7:C3:03:AF:FC:E6:25:8A:BE:4F:8C:3D:1A:B2:E9:D1:23:3D:A1:6B:DD:39:0A:A5:D1:97:B9:5E:1A:79:1B:61",
      "KODE_SHA256_DARI_PLAY_CONSOLE_ANDA_DI_SINI"
    ]
  }
}]
```

> **Tips:** Jangan menghapus kunci debug (`A7:C3:03...`). Biarkan keduanya agar aplikasi tetap bisa bekerja lancar baik saat Anda jalankan dari Android Studio maupun saat diunduh dari Play Store.

## Selesai!
Setelah Anda menyelesaikan langkah-langkah di atas, aplikasi TWA Anda sudah sepenuhnya bebas dari *address bar* seperti layaknya aplikasi bawaan Android *native*!

---

## 🛠️ Troubleshooting (Penyelesaian Masalah)

### 1. Aplikasi Langsung Tiba-tiba *Crash* (Tertutup Sendiri saat Dibuka)
Masalah umum yang terjadi saat pengaturan TWA baru adalah aplikasi di-build dengan **sukses tapi _crash_ saat dibuka**.
Penyebab utamanya adalah parameter `<meta-data>` pada `AndroidManifest.xml` tidak menggunakan referensi XML (*resource reference*) melainkan teks mentah.

Library **`androidbrowserhelper`** mewajibkan nilai-nilai tertentu di-fetch sebagai Array atau Resource, antara lain:
- **`ADDITIONAL_TRUSTED_ORIGINS`** (harus memanggil *string-array*)
- **`asset_statements`** (direkomendasikan memakai *string resource*)

**Solusi yang benar (dan sudah diterapkan pada project ini):**

1. Semua string spesifik ini harus disimpan dalam file `app/src/main/res/values/strings.xml`:
```xml
<resources>
    <string name="app_name">My Passnet</string>
    <string name="asset_statements">${assetStatements}</string>
    <string-array name="additional_trusted_origins">
        <item>https://client.pass.net.id</item>
    </string-array>
</resources>
```
2. Pastikan file `app/src/main/AndroidManifest.xml` menggunakan `android:resource` (bukan `android:value`):
```xml
<!-- Benar (Memakai Resource) -->
<meta-data
    android:name="asset_statements"
    android:resource="@string/asset_statements" />

<meta-data android:name="android.support.customtabs.trusted.ADDITIONAL_TRUSTED_ORIGINS"
    android:resource="@array/additional_trusted_origins" />
```
*Menggunakan `android:value` berisikan link secara langsung pada tag origin tersebut akan mengakibatkan Exception dan menghentikan (crash) aplikasi secara paksa.*

### 2. Peringatan (Warning) "Unsupported Compile SDK" saat nge-Build
Saat di build, Android Studio bisa mengeluarkan _log warning_:
`WARNING: We recommend using a newer Android Gradle plugin to use compileSdk = 35`

Ini hanya sekadar peringatan untuk _versioning_ tool Gradle, dan tidak memengaruhi kualitas/sistem APK Anda. 
**Solusi:** (Sudah diterapkan)
Cukup letakkan tambahan sintaks berikut di bagian terbawah file `gradle.properties`:
```properties
android.suppressUnsupportedCompileSdk=35
```

### 3. Aplikasi "Blank Putih" Saat Koneksi Terputus / Fallback WebView
Agar aplikasi TWA otomatis mengenali bahwa ia butuh mengakses lalu lintas browser web (saat chrome tidak siap dan fallback ke WebView lokal), _permission_ Internet mutlak diberikan secara eksplisit.
**Solusi:** (Sudah diterapkan)
Tambahkan ke dalam `AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
