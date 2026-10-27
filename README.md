# example-go

Dua contoh **Go** di atas SDK [`yespos-client-sdk-go`](https://github.com/Yes-Pos/yespos-client-sdk-go): CLI untuk melihat katalog dan membuat order, serta storefront HTTP dengan keranjang, checkout **QRIS / Virtual Account / EDC**, dan penerima callback status order.

Tanpa dependency selain SDK-nya; sisanya standard library.

## Menjalankan

```bash
export YESPOS_API_BASE_URL="https://api.yespos.id"
export YESPOS_CLIENT_KEY="…"
export YESPOS_CLIENT_SECRET="…"
export YESPOS_OUTLET_UUID="…"                      # opsional, kosong = outlet pertama
export YESPOS_NOTIFY_URL="https://…/api/notify"    # opsional, callback status order
```

### `cmd/quickstart` — CLI

Menampilkan outlet, katalog, dan metode pembayaran yang aktif; bisa sekalian membuat satu order percobaan.

```bash
go run ./cmd/quickstart                                        # lihat katalog
go run ./cmd/quickstart -order p1 -qty 2 -payment va:bca       # buat order
go run ./cmd/quickstart -order p2 -variant v1 -payment qris:qris
go run ./cmd/quickstart -status <uuid-order>                   # cek status
```

Flag lain: `-outlet`, `-timeout`. Jalankan dengan `-h` untuk daftar lengkapnya.

```
Outlet: Cabang Sudirman (o1)

# Makanan
  p1                      Rp25.000              Nasi Goreng
  p2                      Rp18.000 – Rp24.000   Kopi Susu
    varian v1             Rp18.000              Kecil

# Metode pembayaran aktif
  qris:qris   QRIS
  va:bca      BCA
```

### `cmd/storefront` — toko online

```bash
go run ./cmd/storefront          # lalu buka http://localhost:8080
ADDR=:3000 go run ./cmd/storefront
```

| Endpoint | Guna |
|---|---|
| `GET /` | Halaman toko (HTML + JS, tanpa build step) |
| `GET /api/catalog` | Outlet dan produknya |
| `GET /api/payment-methods` | Metode aktif; EDC muncul hanya bila terminalnya dikonfigurasi |
| `POST /api/checkout` | Membuat order dari keranjang |
| `GET /api/orders/{uuid}` | Status dan instruksi bayar sebuah order |
| `POST /api/notify` | Callback status order dari YesPOS |

## Pola yang dipakai

**SDK hanya hidup di server.** Browser memanggil endpoint JSON milik aplikasi ini sendiri, jadi `YESPOS_CLIENT_SECRET` tidak pernah ikut terkirim ke mana pun selain header signature.

**Harga tidak pernah datang dari browser.** `POST /api/checkout` hanya menerima `product_uuid`, `variant_uuid`, dan `quantity`; `shop.PlaceOrder` mengambil harganya dari API lalu menghitung total sendiri. Angka total yang tampil di halaman cuma perkiraan untuk pembeli.

**Status akhir datang lewat callback, bukan polling.** `POST /api/notify` mencatat perubahan status, lalu **membaca ulang order lewat SDK** sebagai sumber kebenaran — isi body callback tidak dipercaya begitu saja. Di halaman, status hanya dicek saat pembeli menekan tombol.

**Error dari YesPOS diteruskan apa adanya**, kecuali kegagalan kredensial: itu masalah server, jadi pembeli hanya melihat pesan umum sementara detailnya masuk ke log.

**Ctrl-C membatalkan request yang sedang jalan**, bukan membunuh proses di tengah panggilan HTTP — keduanya memakai `signal.NotifyContext`, dan storefront menutup diri lewat `server.Shutdown`.

## Mencoba tanpa kredensial asli

Arahkan `YESPOS_API_BASE_URL` ke server tiruan. Repo SDK punya satu di `mock_test.go` yang memverifikasi signature persis seperti backend.

## Contoh lain

| Repo | Kerangka |
|---|---|
| [`yespos-client-sdk-go`](https://github.com/Yes-Pos/yespos-client-sdk-go) | SDK Go |
| [`yespos-client-sdk-php`](https://github.com/Yes-Pos/yespos-client-sdk-php) | SDK PHP |
| [`yespos-client-sdk`](https://github.com/Yes-Pos/yespos-client-sdk) | SDK TypeScript |
| [`example-laravel`](https://github.com/Yes-Pos/example-laravel) | Laravel |

## Lisensi

MIT
