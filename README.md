# Aulaa Payment Integration

Integrasi pembayaran minimalis menggunakan **Aulaa API**, **Supabase Database**, dan **Supabase Edge Functions**. Repository ini ditujukan untuk mengelola transaksi dengan satu tabel pembayaran dan fungsi backend seperlunya.

> **Ruang lingkup:** integrasi API Call Aulaa (API Non-SNAP v1) dengan Hosted Checkout. Pelanggan menyelesaikan pembayaran di halaman Aulaa; aplikasi menerima konfirmasi melalui webhook.

## Komponen

- **Aulaa API** — membuat invoice dan menyediakan payment link.
- **Supabase Edge Functions** — menjadi backend untuk komunikasi dengan Aulaa dan penerimaan webhook.
- **Supabase Database** — menyimpan referensi transaksi dan status pembayaran.

## Alur Pembayaran

1. Frontend meminta backend membuat pembayaran dengan `order_id` dan `amount`.
2. Edge Function memanggil Aulaa: `POST https://api.aulaa.co/v1/payments`.
3. Backend menyimpan ID invoice Aulaa dan data transaksi yang diperlukan.
4. Frontend mengarahkan pelanggan ke `payment_link` dari respons Aulaa.
5. Aulaa mengirim webhook ketika status pembayaran berubah.
6. Edge Function memverifikasi signature webhook dan mencocokkan transaksi sebelum memperbarui status di database.

```text
Frontend
   |
   v
Supabase Edge Function
   |             |
   v             v
Aulaa API    Supabase Database
   |
   v
Aulaa Hosted Checkout
   |
   v
Aulaa Webhook
   |
   v
Supabase Edge Function
   |
   v
Update payment status
```

## Aulaa API

**Base URL:** `https://api.aulaa.co`

| Kegunaan | Method | Endpoint |
|---|---|---|
| Membuat pembayaran | `POST` | `/v1/payments` |
| Cek status pembayaran | `GET` | `/v1/payments/{id}` |
| Membatalkan pembayaran | `POST` | `/v1/payments/{id}/cancel` |

Autentikasi menggunakan Bearer Token:

```http
Authorization: Bearer pgw_id_your_api_key
Content-Type: application/json
```

Contoh request membuat pembayaran:

```json
{
  "order_id": "ORDER-001",
  "amount": 50000
}
```

- `order_id` wajib unik per project Aulaa. Penggunaan ulang menghasilkan `409 Conflict`, bukan invoice baru.
- `amount` berupa integer Rupiah, dengan batas Rp1.000–Rp500.000.000.
- `payment_method` opsional. Jika tidak dikirim, pelanggan dapat memilih metode di halaman pembayaran Aulaa.
- Respons pembuatan pembayaran menyediakan `id`, `status`, dan `payment_link`. Gunakan `payment_link` untuk mengarahkan pelanggan ke Hosted Checkout.
- ID invoice Aulaa (`id`) berbeda dari ID pesanan merchant (`order_id`).

## Webhook

Aulaa mengirim HTTP `POST` ke Callback URL project saat status pembayaran berubah. Payload webhook berbentuk flat dan memuat antara lain `order_id`, `amount`, `currency`, `status`, serta `payment_method`.

Validasi keamanan yang wajib dilakukan:

- Baca **raw request body** sebelum parsing JSON.
- Verifikasi header `X-Webhook-Signature` menggunakan HMAC-SHA256 dan Webhook Secret project.
- Signature berupa hexadecimal tanpa prefix.
- Cocokkan `order_id` dan `amount` dengan data transaksi lokal sebelum menandai pembayaran berhasil.
- Buat handler idempoten karena Aulaa dapat mengirim ulang webhook.
- Kembalikan respons HTTP `2xx` setelah notifikasi berhasil diproses.

Aulaa menyebutkan retry hingga 4 percobaan total (percobaan awal + 3 retry), dengan jeda 1, 5, dan 30 menit.

**Jangan menggunakan redirect pelanggan sebagai bukti pembayaran.** Webhook terverifikasi merupakan notifikasi utama; endpoint cek status digunakan untuk rekonsiliasi atau verifikasi berkala.

## Status Pembayaran

Status yang didokumentasikan Aulaa:

| Status | Arti |
|---|---|
| `pending` | Menunggu pembayaran |
| `paid` | Pembayaran berhasil |
| `expired` | Tagihan kedaluwarsa |
| `cancelled` | Tagihan dibatalkan |
| `failed` | Pembayaran gagal |
| `refunded` | Dana dikembalikan |

## Konfigurasi Supabase

Simpan kredensial sebagai **Supabase Edge Function Secrets**, bukan di frontend atau repository:

```bash
supabase secrets set AULAA_API_KEY=your_aulaa_api_key
supabase secrets set AULAA_WEBHOOK_SECRET=your_aulaa_webhook_secret
```

Gunakan project Aulaa bertipe **API** dan daftarkan URL Edge Function penerima webhook sebagai Callback URL di Dashboard Aulaa.

## Keamanan dan Integritas Data

- Jangan pernah mengekspos API Key atau Webhook Secret ke browser.
- Jangan mempercayai status pembayaran dari frontend atau URL redirect.
- Verifikasi signature webhook dari raw body.
- Cocokkan ID pesanan dan nominal dengan database sebelum memperbarui status.
- Terapkan idempotensi pada pembuatan pembayaran dan pemrosesan webhook.
- Jangan membuat `order_id` baru untuk mengulang request dari pesanan yang sama; tangani respons `409` dengan mengambil `payment_id` yang dikembalikan Aulaa.

## Pengujian

Gunakan mode **Sandbox** Aulaa sebelum Go Live. Sandbox menyediakan kode pembayaran dummy dan halaman simulasi untuk menguji skenario berhasil maupun gagal, termasuk pengiriman webhook.

Halaman simulasi: https://api.aulaa.co/payment-simulation

## Dokumentasi Resmi

- [Dokumentasi Aulaa](https://aulaa.co/dokumentasi)
- [Pengenalan dan tipe project](https://aulaa.co/dokumentasi/mulai/pengenalan)
- [Akun dan API Key](https://aulaa.co/dokumentasi/api-v1/akun-dan-api-key)
- [Buat Pembayaran](https://aulaa.co/dokumentasi/api-v1/buat-pembayaran)
- [Cek Status Pembayaran](https://aulaa.co/dokumentasi/api-v1/cek-status)
- [Batalkan Pembayaran](https://aulaa.co/dokumentasi/api-v1/batalkan-pembayaran)
- [Webhook](https://aulaa.co/dokumentasi/api-v1/webhook)
- [Kode Error](https://aulaa.co/dokumentasi/api-v1/kode-error)
- [Supabase Edge Functions](https://supabase.com/docs/guides/functions)

## License

Belum ditentukan.
