# DoorHarvest

Dashboard realtime untuk monitoring gudang berbasis ESP32 dan Firebase Realtime Database.
Seluruh dashboard ada di satu file, `index.html`, tanpa build step. Buka langsung di
browser atau deploy ke Firebase Hosting.

## Konfigurasi

- `firebaseConfig` ada di bagian atas skrip dalam `index.html`.
- Dashboard tidak memakai login, jadi Rules harus mengizinkan pembacaan publik pada
  `/device/esp32`:

```json
{
  "rules": {
    "device": {
      "esp32": {
        ".read": true
      }
    }
  }
}
```

Contoh di atas hanya mengatur pembacaan dan tidak memberi akses tulis. Tambahkan `.write`
sesuai cara ESP32 mengautentikasi diri. Dengan `.read: true`, siapa pun yang tahu URL
database bisa membaca datanya.

## Peringatan: format alamat database berbeda

Alamat database ditulis berbeda di sketsa ESP32 dan di dashboard. Keduanya sering
tertukar, dan errornya tidak jelas.

| Tempat | Nama | Format | Nilai |
|---|---|---|---|
| Sketsa ESP32 | `FIREBASE_HOST` | **Tanpa** `https://` dan tanpa `/` di akhir | `doorharvest-default-rtdb.asia-southeast1.firebasedatabase.app` |
| Dashboard | `databaseURL` | **Lengkap** dengan `https://` | `https://doorharvest-default-rtdb.asia-southeast1.firebasedatabase.app` |

## Pemecahan masalah

### Pesan di serial monitor ESP32

| Pesan | Arti | Yang harus dilakukan |
|---|---|---|
| `Firebase GAGAL: -1` | Permintaan tidak sampai ke server | Periksa WiFi atau captive portal |
| `ditolak, HTTP 401` atau `ditolak, HTTP 403` | Rules menolak | Periksa Rules di Firebase Console |
| `ditolak, HTTP 404` | `FIREBASE_HOST` salah ketik | Cocokkan dengan format pada tabel di atas |
| Tidak ada baris `Upload` sama sekali | WiFi belum tersambung | Periksa SSID, kata sandi, dan jangkauan WiFi |

### Tampilan kosong di dashboard

Tiga kondisi ini mudah tertukar, padahal penyebabnya berbeda.

| Kartu status | Indikator kanan atas | Arti | Yang harus dilakukan |
|---|---|---|---|
| `MENUNGGU` | Terhubung | Database bisa dibaca tetapi kosong; alat belum pernah mengirim | Periksa serial monitor ESP32 |
| `AKSES DITOLAK` | Terhubung | Rules menolak pembacaan oleh dashboard | Izinkan `.read`, lalu muat ulang halaman |
| `TERPUTUS` | Terputus | Perangkat yang membuka dashboard tidak tersambung internet | Periksa internet perangkat; halaman menyambung ulang otomatis |

Kalau muncul `GAGAL MULAI`, baca keterangan di kartu. Penyebabnya SDK tidak bisa diunduh
dari `www.gstatic.com`, atau `databaseURL` salah format.
