# Analisis Encoding - Case 2 Product/Inventory API

## 1. Format Encoding

API Product/Inventory ini menggunakan format **JSON (JavaScript Object Notation)** sebagai satu-satunya format encoding untuk request body maupun response body. Hal ini terlihat dari header `Content-Type: application/json; charset=utf-8` yang konsisten muncul pada seluruh response dari server.

JSON dipilih karena beberapa alasan yang relevan dengan konteks API ini:
- **Mudah dibaca manusia** — struktur key-value plaintext memudahkan debugging langsung dari terminal atau Postman.
- **Ringan** — tidak ada tag pembungkus seperti XML, sehingga ukuran payload lebih kecil.
- **Didukung luas** — hampir semua bahasa pemrograman dan HTTP client (termasuk Postman, curl, browser) mendukung parsing JSON secara native.
- **Cocok untuk data semi-terstruktur** — produk memiliki field dengan tipe data yang beragam (string, number) yang terwakili langsung dalam JSON.

---

## 2. Struktur Data per Response

### 2.1 GET /api/products — Daftar Produk

**Response body (200 OK):**
```json
{
  "count": 3,
  "data": [
    {
      "id": 1,
      "name": "Laptop Pro 14",
      "category": "computer",
      "price": 14500000,
      "stock": 5
    },
    {
      "id": 2,
      "name": "Temperature Sensor DHT22",
      "category": "iot",
      "price": 85000,
      "stock": 30
    },
    {
      "id": 3,
      "name": "Wireless Router AC1200",
      "category": "networking",
      "price": 475000,
      "stock": 10
    }
  ]
}
```

**Struktur root** adalah sebuah JSON object dengan dua field:
- `count` (number) — jumlah total produk dalam respons, berguna untuk pagination atau validasi di sisi client.
- `data` (array of object) — daftar produk. Setiap object di dalam array memiliki field:
  - `id` (number) — identifikasi unik produk.
  - `name` (string) — nama produk.
  - `category` (string) — kategori produk.
  - `price` (number) — harga produk dalam satuan rupiah (integer, bukan float).
  - `stock` (number) — jumlah stok tersedia.

---

### 2.2 GET /api/products/{id} — Detail Produk

**Response body (200 OK):**
```json
{
  "id": 2,
  "name": "Temperature Sensor DHT22",
  "category": "iot",
  "price": 85000,
  "stock": 30
}
```

**Struktur root** adalah sebuah object tunggal (bukan dibungkus array). Field-fieldnya identik dengan satu elemen di dalam `data` pada response daftar.

**Perbedaan dengan response daftar:** Response detail tidak memiliki wrapper `count` dan `data` — langsung berupa object produk. Hal ini masuk akal karena client sudah menentukan id produk yang diminta, sehingga tidak ada ambiguitas jumlah data yang dikembalikan. Membungkusnya dalam array justru akan menambah kompleksitas parsing tanpa manfaat.

---

### 2.3 POST /api/products — Tambah Produk (body valid)

**Request body yang dikirim:**
```json
{
  "name": "Mechanical Keyboard K1",
  "category": "accessory",
  "price": 650000,
  "stock": 15
}
```

**Response body (201 Created):**
```json
{
  "id": 4,
  "name": "Mechanical Keyboard K1",
  "category": "accessory",
  "price": 650000,
  "stock": 15,
  "createdAt": "2026-06-22T10:13:58+0000"
}
```

**Field tambahan yang muncul di response (tidak ada di request body):**
- `id` (number) — ID baru yang digenerate server secara otomatis (auto-increment dari data yang sudah ada).
- `createdAt` (string, format ISO 8601) — timestamp pembuatan produk, digenerate server pada saat record dibuat.

Pola ini umum di REST API: client hanya mengirim data bisnis, sedangkan metadata sistem (`id`, `createdAt`) ditambahkan sepenuhnya oleh server.

**Header tambahan di response:**
- `Location: /api/products/4` — memberitahu client di mana resource baru bisa diakses. Ini adalah praktik standar HTTP untuk response `201 Created`.

---

### 2.4 POST /api/products — Uji Validasi Input (body tidak lengkap)

**Request body yang dikirim (field `stock` sengaja dihilangkan):**
```json
{
  "name": "Produk Tanpa Stok",
  "category": "accessory",
  "price": 99000
}
```

**Response body (400 Bad Request):**
```json
{
  "error": "Missing required fields",
  "missing": ["stock"]
}
```

**Struktur pesan error:**
- `error` (string) — pesan singkat yang menjelaskan jenis kesalahan.
- `missing` (array of string) — daftar nama field yang wajib diisi namun tidak ditemukan di request body. Dengan format array, server bisa melaporkan lebih dari satu field yang kurang sekaligus dalam satu response.

---

## 3. Analisis HTTP Method

**Mengapa GET dipakai untuk membaca data?**
GET adalah method HTTP yang semantiknya adalah "ambil/baca resource tanpa mengubah state server." GET bersifat *safe* (tidak ada perubahan data) dan *idempotent* (memanggil 10 kali hasilnya sama). Karena membaca daftar produk atau detail produk tidak mengubah apapun di server, GET adalah pilihan yang tepat dan sesuai standar REST.

**Mengapa POST dipakai untuk menambah produk?**
POST digunakan untuk membuat resource baru di server. Berbeda dengan GET, POST menyertakan request body berisi data yang akan disimpan, dan setiap request POST yang berhasil menghasilkan record baru dengan ID berbeda — artinya POST tidak bersifat idempotent. Setelah POST berhasil, server merespons dengan `201 Created` (bukan `200 OK`) untuk membedakan antara "berhasil membaca" dan "berhasil membuat resource baru."

**Perbedaan POST (create) vs PATCH/PUT (update):**
- `POST /api/products` → membuat produk baru, ID ditentukan server.
- `PUT /api/products/{id}` → menggantikan seluruh data produk yang sudah ada (replace).
- `PATCH /api/products/{id}` → memperbarui sebagian field produk (partial update, misalnya hanya `stock`).

---

## 4. Analisis Status Code

| Request | Status Code | Arti | Mengapa server memberikan status ini |
|---|---|---|---|
| GET /api/products | 200 OK | Permintaan berhasil | Server berhasil menemukan dan mengembalikan data daftar produk |
| GET /api/products/2 | 200 OK | Permintaan berhasil | Server berhasil menemukan produk dengan id=2 dan mengembalikan detailnya |
| POST /api/products (valid) | 201 Created | Resource baru berhasil dibuat | Server menerima data produk baru, menyimpannya, dan meng-generate ID serta timestamp |
| POST /api/products (tidak lengkap) | 400 Bad Request | Request dari client tidak valid | Server mendeteksi bahwa field wajib (`stock`) tidak dikirimkan, sehingga request ditolak sebelum data disimpan |

Status code tambahan yang relevan untuk case ini (meskipun tidak muncul dalam 4 request utama):
- **404 Not Found** — akan muncul jika `GET /api/products/999` dipanggil dengan ID yang tidak ada.
- **500 Internal Server Error** — akan muncul jika terjadi kesalahan tak terduga di sisi server.

---

## 5. Analisis Header Penting

**`Content-Type: application/json; charset=utf-8`**
Header ini hadir di setiap response server dan berfungsi memberitahu client tentang format isi body. Nilai `application/json` artinya body harus diparsing sebagai JSON, sedangkan `charset=utf-8` artinya teks dikodekan dalam UTF-8 (mendukung karakter multibahasa termasuk bahasa Indonesia). Di sisi request, header ini juga wajib dikirim client pada request POST agar server mengetahui cara membaca body.

**`Location: /api/products/4`** (hanya muncul di response 201)
Header ini menunjuk ke URL resource yang baru saja dibuat. Manfaatnya: client tidak perlu menebak atau melakukan GET tambahan untuk mengetahui URL produk baru — cukup baca header `Location` dan gunakan langsung untuk request berikutnya.

**`Content-Length: [angka]`**
Memberitahu client ukuran body response dalam byte. Berguna agar client tahu kapan pembacaan response selesai tanpa perlu menunggu koneksi ditutup.

**`Access-Control-Allow-Origin: *`**
Header ini adalah bagian dari mekanisme CORS (Cross-Origin Resource Sharing). Nilai `*` berarti API ini mengizinkan request dari domain manapun — umum pada demo app atau API publik yang ingin bisa dipanggil dari browser domain apapun.

**`Access-Control-Allow-Methods: GET, POST, PUT, PATCH, DELETE, OPTIONS`**
Mendefinisikan method HTTP apa saja yang diizinkan server. Browser menggunakan header ini untuk memvalidasi apakah request yang akan dikirim diizinkan.

---

## 6. Validasi Input & Error Handling

**Field yang divalidasi sebagai wajib:** `name`, `category`, `price`, dan `stock`. Jika salah satu atau lebih field ini tidak dikirimkan dalam body POST, server akan menolak request dengan status `400 Bad Request`.

**Cara API ini memberi tahu client:** Respons error berisi JSON dengan dua field — `error` (deskripsi singkat jenis error) dan `missing` (array berisi nama field yang belum diisi). Format ini sangat informatif karena client bisa langsung mengetahui field mana yang perlu dilengkapi tanpa harus menebak.

**Apakah sudah cukup informatif?** Ya, untuk API internal atau demo. Namun untuk API produksi, biasanya ditambahkan kode error spesifik (misalnya `"code": "VALIDATION_ERROR"`) dan pesan yang lebih ramah pengguna agar bisa ditampilkan langsung ke UI.

---

## 7. Kesimpulan

> **Catatan:** Bagian ini perlu disesuaikan dengan pengalaman pribadi kamu saat menjalankan request — termasuk kendala spesifik yang kamu temui dan pembelajaran yang kamu rasakan sendiri.

Dari pengerjaan Case 2 ini, terlihat jelas bagaimana REST API menggunakan kombinasi **HTTP method + status code + JSON encoding** sebagai kontrak komunikasi antara client dan server. Method GET dan POST masing-masing memiliki semantik yang berbeda dan tidak saling menggantikan. Status code bukan sekadar angka — masing-masing membawa makna spesifik tentang hasil operasi. Format JSON memudahkan representasi data terstruktur sekaligus tetap mudah dibaca manusia.

Hal menarik yang ditemukan: server menambahkan field `id` dan `createdAt` secara otomatis pada response POST, memisahkan tanggung jawab antara data bisnis (dikirim client) dan metadata sistem (dikelola server). Ini adalah pola yang sangat umum dan penting dipahami dalam desain REST API.

Kendala yang ditemui: [isi sendiri berdasarkan pengalaman kamu]

Solusi: [isi sendiri]
