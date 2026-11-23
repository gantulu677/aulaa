Minimalist Payment System

Sistem pembayaran minimalis menggunakan Aulaa API dan Supabase. Menggunakan tabel database sederhana dan Edge Functions untuk membuat pembayaran, mengelola transaksi, serta memverifikasi status pembayaran.

Tech Stack

Supabase Database

Supabase Edge Functions

Aulaa Payment API

Features

Membuat transaksi pembayaran.

Menyimpan data transaksi.

Mengelola status pembayaran.

Memverifikasi pembayaran melalui backend.

Integrasi dengan Aulaa API.

Architecture

Frontend
   |
   v
Supabase Edge Functions
   |
   v
Aulaa API
   |
   v
Payment Gateway

Supabase Database
   |
   v
Transaction Records

Environment Variables

Simpan konfigurasi API sebagai Secrets pada Supabase:

AULAA_API_KEY=your_api_key

Jangan pernah mengekspos API Key di frontend atau repository publik.

Security

Seluruh komunikasi dengan Aulaa dilakukan melalui Edge Functions.

API Key disimpan secara aman di backend.

Status pembayaran harus diverifikasi sebelum transaksi dinyatakan berhasil.

Documentation

Aulaa API Documentation

Supabase Documentation

Supabase Edge Functions

License

MIT License
