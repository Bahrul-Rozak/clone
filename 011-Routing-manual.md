Halo teman-teman!

Setelah kita berhasil membuat server HTTP dasar menggunakan modul `http` bawaan Node.js, langkah berikutnya yang sangat penting—tapi sering dianggap “mudah” oleh framework seperti Express—adalah **routing manual**. Artinya: kita akan menentukan sendiri **apa yang terjadi saat pengguna mengakses `/`, `/users`, `/api/data`, atau rute lainnya**, tanpa bantuan router eksternal. Ini bukan cuma soal kontrol penuh, tapi juga cara terbaik untuk benar-benar memahami bagaimana permintaan HTTP dipetakan ke logika aplikasi.

Dan yang terpenting: **kita akan lakukan ini step-by-step, termasuk cara menjalankannya dari nol**—jadi kamu bisa ikut praktik langsung, bahkan jika ini kali pertamamu menyentuh backend.

---

### 🔧 **Langkah 1: Buat Folder Proyek & File Server**

Buka terminal (Command Prompt, PowerShell, atau Terminal di VS Code), lalu buat folder baru:

```bash
mkdir routing-manual
cd routing-manual
```

Buat file utama:

```bash
touch server.js
```

> 💡 Jika kamu pakai Windows dan perintah `touch` tidak dikenali, ganti dengan:
> ```bash
> type nul > server.js
> ```

---

### 📄 **Langkah 2: Tulis Kode Routing Manual**

Buka `server.js` di editor kode (misalnya VS Code), lalu salin kode berikut:

```js
const http = require('http');
const url = require('url');

// Buat server HTTP
const server = http.createServer((req, res) => {
  // Parse URL agar mudah diakses bagian-bagiannya
  const parsedUrl = url.parse(req.url, true);
  const path = parsedUrl.pathname; // misal: "/users"
  const method = req.method;       // misal: "GET", "POST"

  // Atur header default: izinkan CORS & tipe konten JSON
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Content-Type', 'application/json');

  // Routing manual — sesuaikan respons berdasarkan path & method
  if (path === '/' && method === 'GET') {
    res.writeHead(200);
    res.end(JSON.stringify({ message: 'Selamat datang di server manual!' }));
  }
  else if (path === '/users' && method === 'GET') {
    res.writeHead(200);
    res.end(JSON.stringify({ users: ['Andi', 'Budi', 'Citra'] }));
  }
  else if (path === '/users' && method === 'POST') {
    // Tangani body (untuk demo, kita kirim respons statis)
    res.writeHead(201);
    res.end(JSON.stringify({ message: 'User berhasil dibuat!' }));
  }
  else if (path === '/about' && method === 'GET') {
    res.writeHead(200);
    res.end(JSON.stringify({ version: '1.0.0', author: 'Kamu' }));
  }
  else {
    // Jika rute tidak dikenali
    res.writeHead(404);
    res.end(JSON.stringify({ error: 'Rute tidak ditemukan' }));
  }
});

// Jalankan server di port 3000
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`✅ Server berjalan di http://localhost:${PORT}`);
  console.log(`Coba buka:\n- http://localhost:${PORT}/\n- http://localhost:${PORT}/users\n- http://localhost:${PORT}/about`);
});
```

---

### ▶️ **Langkah 3: Jalankan Server**

Kembali ke terminal, pastikan kamu berada di folder `routing-manual`, lalu jalankan:

```bash
node server.js
```

Jika berhasil, kamu akan melihat pesan:
```
✅ Server berjalan di http://localhost:3000
Coba buka:
- http://localhost:3000/
- http://localhost:3000/users
- http://localhost:3000/about
```

---

### 🌐 **Langkah 4: Uji Rute-rutenya**

Buka browser dan kunjungi:

1. **`http://localhost:3000/`**  
   → Akan muncul: `{"message":"Selamat datang di server manual!"}`

2. **`http://localhost:3000/users`**  
   → Akan muncul daftar user dalam format JSON.

3. **`http://localhost:3000/about`**  
   → Menampilkan info versi.

4. **`http://localhost:3000/apa-saja`**  
   → Akan muncul error 404: `{"error":"Rute tidak ditemukan"}`

> 💡 Untuk menguji **POST /users**, kamu butuh tools seperti **Postman**, **Insomnia**, atau curl:
> ```bash
> curl -X POST http://localhost:3000/users
> ```

---

### 🧠 **Penjelasan Inti: Bagaimana Routing Ini Bekerja?**

- Setiap request masuk → kita **parse URL-nya** pakai `url.parse(req.url, true)`.
- Lalu kita ambil `pathname` (misal `/users`) dan `method` (`GET`, `POST`, dll).
- Dengan **if-else bersarang**, kita cocokkan kombinasi path + method.
- Setiap cabang mengatur:
  - Status code (`200`, `201`, `404`),
  - Header (kita set `Content-Type: application/json`),
  - Body (dikirim via `res.end()`).

Ini adalah **routing manual murni**: tidak ada regex, tidak ada parameter dinamis (belum), tidak ada middleware—hanya logika kondisional sederhana yang 100% transparan.

---

### ⚠️ **Catatan Penting**
- Modul `url.parse()` sebenarnya sudah *deprecated* di Node.js versi terbaru, tapi masih aman dipakai untuk pembelajaran. Di proyek nyata, kamu bisa gunakan `new URL(req.url, 'http://localhost')` sebagai pengganti modern.
- Untuk menangani **body request** (misal data dari form atau JSON), kamu perlu dengarkan event `data` dan `end` dari `req`—tapi itu akan kita bahas di materi selanjutnya.

---

**Kesimpulannya**, routing manual mungkin terlihat “primitif” dibanding Express, tapi justru di sinilah kamu belajar **bagaimana web server benar-benar bekerja**: setiap rute adalah keputusan logis berdasarkan URL dan method. Dengan membangun ini dari nol, kamu tidak hanya tahu *bagaimana* membuat API—tapi juga *mengapa* setiap bagian ada. Dan itulah fondasi sejati seorang developer backend: **tidak takut pada hal-hal yang “sudah disediakan”, karena kamu tahu cara membangunnya sendiri**.
