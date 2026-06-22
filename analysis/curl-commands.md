# Curl Commands - Case 2 Product/Inventory API

Base URL demo app: `http://localhost:8088`

Jalankan `curl -X POST http://localhost:8088/api/reset` lebih dulu supaya data produk dalam kondisi awal (3 produk seed) sebelum mengambil evidence.

## 1. GET - Daftar Produk
```bash
curl -i -X GET "http://localhost:8088/api/products"
```
Expected: `200 OK`, body berisi `count` dan array `data` (semua produk).

## 2. GET - Detail Produk
```bash
curl -i -X GET "http://localhost:8088/api/products/2"
```
Expected: `200 OK`, body berisi satu object produk (id, name, category, price, stock). Ganti `2` dengan id produk lain bila perlu, atau `404` jika id tidak ada.

## 3. POST - Tambah Produk (body lengkap)
```bash
curl -i -X POST "http://localhost:8088/api/products" \
 -H "Content-Type: application/json" \
 -d '{"name":"Mechanical Keyboard K1","category":"accessory","price":650000,"stock":15}'
```
Expected: `201 Created`, header `Location: /api/products/{id-baru}`, body berisi produk baru lengkap dengan `createdAt`.

## 4. POST - Uji Validasi Input (body tidak lengkap)
```bash
curl -i -X POST "http://localhost:8088/api/products" \
 -H "Content-Type: application/json" \
 -d '{"name":"Produk Tanpa Stok","category":"accessory","price":99000}'
```
Expected: `400 Bad Request`, body berisi `{"error":"Missing required fields","missing":["stock"]}`.

---

## Catatan
- Jika eksekusi dilakukan lewat Postman, gunakan tombol **Code** (`</>`) pada tiap request untuk mendapatkan snippet curl yang sama persis, lalu tempel di sini sebagai bukti command.
- Ganti nilai pada body sesuai data yang benar-benar kamu jalankan, lalu sesuaikan juga dengan response yang muncul di evidence/screenshot-mu.
