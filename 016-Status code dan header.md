Halo teman-teman!

Di balik setiap halaman web yang kamu buka atau API yang kamu panggil, ada dua komponen tak terlihat tapi sangat krusial: **status code** dan **header HTTP**. Status code memberi tahu klien apakah permintaannya sukses, gagal, atau perlu diarahkan ke tempat lain. Sementara header mengirim metadata—seperti tipe konten, izin akses, cache control, dan banyak lagi.

Di Node.js native (tanpa Express), kamu punya kendali penuh atas keduanya. Dan memahami cara mengatur status code dan header dengan benar adalah **kunci membuat API yang profesional, aman, dan mudah dipakai**.

Mari kita pelajari ini **step-by-step dari nol**, lengkap dengan kode, cara menjalankan, dan uji langsung!

---

### 🔧 **Langkah 1: Buat Folder & File Proyek**

Buka terminal, lalu buat folder baru:

```bash
mkdir status-header-demo
cd status-header-demo
```

Buat file server:

```bash
# macOS/Linux:
touch server.js

# Windows (Command Prompt):
type nul > server.js
```

---

### 📄 **Langkah 2: Tulis Kode Server dengan Berbagai Status Code & Header**

Buka `server.js` di editor kode, lalu salin kode berikut:

```js
const http = require('http');

const server = http.createServer((req, res) => {
  const { url, method } = req;

  // Rute: /sukses → 200 OK
  if (url === '/sukses' && method === 'GET') {
    res.writeHead(200, {
      'Content-Type': 'application/json',
      'X-Custom-Header': 'Ini header kustom!',
      'Cache-Control': 'public, max-age=60' // cache 60 detik
    });
    res.end(JSON.stringify({ pesan: 'Operasi berhasil!' }));
  }

  // Rute: /dibuat → 201 Created
  else if (url === '/dibuat' && method === 'POST') {
    res.writeHead(201, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ id: 123, pesan: 'Sumber daya berhasil dibuat' }));
  }

  // Rute: /pindah → 302 Found (redirect sementara)
  else if (url === '/pindah' && method === 'GET') {
    res.writeHead(302, { Location: '/sukses' }); // Arahkan ke /sukses
    res.end();
  }

  // Rute: /tidak_diizinkan → 403 Forbidden
  else if (url === '/rahasia' && method === 'GET') {
    res.writeHead(403, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Akses ditolak!' }));
  }

  // Rute: /tidak_ditemukan → 404 Not Found
  else if (url === '/apa-saja' && method === 'GET') {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Halaman tidak ditemukan' }));
  }

  // Rute: /error_internal → 500 Internal Server Error
  else if (url === '/error' && method === 'GET') {
    res.writeHead(500, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Terjadi kesalahan di server' }));
  }

  // Rute utama: panduan
  else if (url === '/' && method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end(`
Status Code & Header Demo

Coba akses:
- GET  /sukses        → 200 OK
- POST /dibuat         → 201 Created
- GET  /pindah         → 302 Redirect ke /sukses
- GET  /rahasia        → 403 Forbidden
- GET  /apa-saja       → 404 Not Found
- GET  /error          → 500 Internal Error
    `);
  }

  // Jika method tidak didukung
  else {
    res.writeHead(405, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Method tidak diizinkan' }));
  }
});

// Jalankan server
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`✅ Server berjalan di http://localhost:${PORT}`);
  console.log(`Buka browser atau gunakan curl untuk uji berbagai status code!`);
});
```

---

### ▶️ **Langkah 3: Jalankan Server**

Di terminal, pastikan kamu di folder `status-header-demo`, lalu jalankan:

```bash
node server.js
```

Jika berhasil, muncul:
```
✅ Server berjalan di http://localhost:3000
Buka browser atau gunakan curl untuk uji berbagai status code!
```

---

### 🧪 **Langkah 4: Uji Setiap Status Code & Header**

#### ✅ **Uji 1: 200 OK + Header Kustom**
```bash
curl -i http://localhost:3000/sukses
```
> `-i` menampilkan **header respons**  
> Kamu akan lihat:
> ```
> HTTP/1.1 200 OK
> Content-Type: application/json
> X-Custom-Header: Ini header kustom!
> Cache-Control: public, max-age=60
> ```

#### ✅ **Uji 2: 201 Created (POST)**
```bash
curl -i -X POST http://localhost:3000/dibuat
```

#### ✅ **Uji 3: 302 Redirect**
```bash
curl -i http://localhost:3000/pindah
```
> Lihat header `Location: /sukses`

#### ✅ **Uji 4: 403 Forbidden**
```bash
curl -i http://localhost:3000/rahasia
```

#### ✅ **Uji 5: 404 Not Found**
```bash
curl -i http://localhost:3000/apa-saja
```

#### ✅ **Uji 6: 500 Internal Error**
```bash
curl -i http://localhost:3000/error
```

#### ✅ **Uji 7: Halaman Panduan**
Buka di browser:  
`http://localhost:3000`  
→ Akan muncul teks panduan.

---

### 🔍 **Penjelasan Inti: Status Code & Header**

#### **Status Code Umum:**
- `200 OK` → Sukses.
- `201 Created` → Sumber daya baru dibuat (biasanya POST).
- `302 Found` → Redirect sementara (gunakan `Location` header).
- `403 Forbidden` → Akses ditolak (otentikasi valid, tapi tidak punya izin).
- `404 Not Found` → Rute tidak ada.
- `405 Method Not Allowed` → Method (GET/POST) tidak didukung di rute itu.
- `500 Internal Server Error` → Error di sisi server.

#### **Header Penting:**
- `Content-Type` → Wajib! Beri tahu klien jenis data yang dikirim.
- `Location` → Untuk redirect (3xx).
- `Cache-Control` → Atur caching di browser/proxy.
- `X-*` → Header kustom (misal `X-Request-ID` untuk logging).

> 💡 **Catatan**:  
> - Gunakan `res.writeHead(statusCode, headers)` untuk mengatur keduanya sekaligus.  
> - Atau gunakan `res.statusCode = 404` + `res.setHeader()` secara terpisah.

---

### ⚠️ **Best Practice**
- Selalu kirim **status code yang tepat** — jangan pakai 200 untuk error!
- Sertakan `Content-Type` yang sesuai (`application/json`, `text/html`, dll).
- Untuk API, hindari mengirim detail error sensitif di produksi (misal stack trace).
- Gunakan header seperti `X-Content-Type-Options: nosniff` untuk keamanan tambahan.

---

**Kesimpulannya**, status code dan header bukan sekadar formalitas—mereka adalah **bahasa protokol HTTP yang sesungguhnya**. Dengan mengatur keduanya dengan benar, kamu memberi klien (browser, app, developer lain) informasi yang akurat, aman, dan berguna. Dan di Node.js native, kamu punya kebebasan penuh untuk mengendalikannya—tanpa abstraksi tersembunyi. Karena API yang baik bukan hanya yang berfungsi… tapi yang **berbicara dengan jelas**. Sekarang, server-mu tidak hanya merespons—tapi juga **berkomunikasi**.
