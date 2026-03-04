Halo teman-teman!

Bayangkan kamu ingin membuat **REST API lengkap**—dengan operasi dasar: lihat semua user, tambah user, edit user, dan hapus user—tapi **tanpa Express, tanpa framework, hanya pakai Node.js murni**. Mungkin terdengar menantang, tapi sebenarnya ini adalah cara terbaik untuk benar-benar memahami bagaimana REST API bekerja di balik layar.

Kita akan bangun semuanya dari nol:  
- Routing manual,  
- Parsing URL (termasuk parameter `:id`),  
- Parsing body JSON,  
- Mengelola data dalam memori (simulasi database),  
- Dan mengirim respons dengan status code yang tepat.

Dan yang terpenting: **kita akan lakukan ini step-by-step**, lengkap dengan cara menjalankan dan mengujinya—jadi kamu bisa ikut praktik langsung!

---

### 🔧 **Langkah 1: Buat Folder & File Proyek**

Buka terminal, lalu buat folder baru:

```bash
mkdir rest-api-native
cd rest-api-native
```

Buat file server:

```bash
# macOS/Linux:
touch server.js

# Windows (Command Prompt):
type nul > server.js
```

---

### 📄 **Langkah 2: Tulis Kode REST API Lengkap**

Buka `server.js` di editor kode, lalu salin kode berikut:

```js
const http = require('http');
const url = require('url');

// Simulasi "database" di memori
let users = [
  { id: 1, nama: 'Andi', email: 'andi@example.com' },
  { id: 2, nama: 'Budi', email: 'budi@example.com' }
];
let nextId = 3;

// Helper: kirim JSON response
function sendJSON(res, statusCode, data) {
  res.writeHead(statusCode, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify(data));
}

// Helper: parse body JSON
function parseBody(req) {
  return new Promise((resolve, reject) => {
    let body = '';
    req.on('data', chunk => body += chunk.toString());
    req.on('end', () => {
      try {
        resolve(JSON.parse(body));
      } catch (err) {
        reject(new Error('Invalid JSON'));
      }
    });
  });
}

// Buat server
const server = http.createServer(async (req, res) => {
  // Set CORS agar bisa diuji dari browser/tools eksternal
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');

  // Handle preflight CORS
  if (req.method === 'OPTIONS') {
    return res.end();
  }

  const parsedUrl = url.parse(req.url, true);
  const path = parsedUrl.pathname;
  const method = req.method;

  // --- Routing: GET /users ---
  if (path === '/users' && method === 'GET') {
    sendJSON(res, 200, users);
  }

  // --- Routing: GET /users/:id ---
  else if (path.startsWith('/users/') && method === 'GET') {
    const id = parseInt(path.split('/')[2]);
    const user = users.find(u => u.id === id);
    if (user) {
      sendJSON(res, 200, user);
    } else {
      sendJSON(res, 404, { error: 'User tidak ditemukan' });
    }
  }

  // --- Routing: POST /users ---
  else if (path === '/users' && method === 'POST') {
    try {
      const data = await parseBody(req);
      if (!data.nama || !data.email) {
        return sendJSON(res, 400, { error: 'Nama dan email wajib diisi' });
      }
      const newUser = { id: nextId++, ...data };
      users.push(newUser);
      sendJSON(res, 201, newUser);
    } catch (err) {
      sendJSON(res, 400, { error: 'Body harus berupa JSON valid' });
    }
  }

  // --- Routing: PUT /users/:id ---
  else if (path.startsWith('/users/') && method === 'PUT') {
    const id = parseInt(path.split('/')[2]);
    const userIndex = users.findIndex(u => u.id === id);

    if (userIndex === -1) {
      return sendJSON(res, 404, { error: 'User tidak ditemukan' });
    }

    try {
      const data = await parseBody(req);
      if (!data.nama || !data.email) {
        return sendJSON(res, 400, { error: 'Nama dan email wajib diisi' });
      }
      users[userIndex] = { id, ...data };
      sendJSON(res, 200, users[userIndex]);
    } catch (err) {
      sendJSON(res, 400, { error: 'Body harus berupa JSON valid' });
    }
  }

  // --- Routing: DELETE /users/:id ---
  else if (path.startsWith('/users/') && method === 'DELETE') {
    const id = parseInt(path.split('/')[2]);
    const initialLength = users.length;
    users = users.filter(u => u.id !== id);

    if (users.length < initialLength) {
      sendJSON(res, 200, { message: 'User berhasil dihapus' });
    } else {
      sendJSON(res, 404, { error: 'User tidak ditemukan' });
    }
  }

  // --- Rute tidak dikenali ---
  else {
    sendJSON(res, 404, { error: 'Rute tidak ditemukan' });
  }
});

// Jalankan server
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`✅ REST API berjalan di http://localhost:${PORT}`);
  console.log(`\nEndpoint yang tersedia:`);
  console.log(`GET    http://localhost:${PORT}/users`);
  console.log(`GET    http://localhost:${PORT}/users/1`);
  console.log(`POST   http://localhost:${PORT}/users`);
  console.log(`PUT    http://localhost:${PORT}/users/1`);
  console.log(`DELETE http://localhost:${PORT}/users/1`);
});
```

---

### ▶️ **Langkah 3: Jalankan Server**

Di terminal, pastikan kamu di folder `rest-api-native`, lalu jalankan:

```bash
node server.js
```

Jika berhasil, muncul daftar endpoint seperti:
```
✅ REST API berjalan di http://localhost:3000

Endpoint yang tersedia:
GET    http://localhost:3000/users
GET    http://localhost:3000/users/1
...
```

---

### 🧪 **Langkah 4: Uji Semua Endpoint**

Gunakan **curl** di terminal baru (atau Postman/Insomnia):

#### ✅ **1. GET /users** — lihat semua user
```bash
curl http://localhost:3000/users
```

#### ✅ **2. GET /users/1** — lihat user spesifik
```bash
curl http://localhost:3000/users/1
```

#### ✅ **3. POST /users** — tambah user baru
```bash
curl -X POST http://localhost:3000/users \
  -H "Content-Type: application/json" \
  -d '{"nama":"Citra","email":"citra@test.com"}'
```

#### ✅ **4. PUT /users/1** — edit user
```bash
curl -X PUT http://localhost:3000/users/1 \
  -H "Content-Type: application/json" \
  -d '{"nama":"Andi Baru","email":"andi.baru@test.com"}'
```

#### ✅ **5. DELETE /users/1** — hapus user
```bash
curl -X DELETE http://localhost:3000/users/1
```

> 💡 **Catatan**:  
> - Data disimpan di memori → jika server direstart, data kembali ke awal.  
> - Untuk proyek nyata, ganti `users` array dengan database (SQLite, PostgreSQL, dll).

---

### 🔍 **Apa yang Kita Bangun?**

- **Routing manual** dengan `startsWith()` dan `split()` untuk ekstrak `:id`.
- **Parsing body JSON** dengan Promise agar bisa pakai `async/await`.
- **Status code RESTful**:  
  - `200` untuk sukses baca/edit,  
  - `201` untuk pembuatan,  
  - `404` jika data tidak ada,  
  - `400` untuk input salah.
- **Respons konsisten** dalam format JSON.

---

### ⚠️ **Catatan Penting**
- Ini adalah **demo edukasi** — jangan gunakan di produksi tanpa validasi, rate limiting, atau autentikasi.
- Untuk parameter dinamis lebih kompleks (misal `/users/:id/posts/:postId`), pertimbangkan library routing ringan.
- Tapi intinya tetap sama: **REST API hanyalah kombinasi HTTP method + path + data**.

---

**Kesimpulannya**, membuat REST API tanpa framework bukan tentang “menolak tools”, tapi tentang **memahami fondasi**. Dengan membangun ini dari nol, kamu tahu persis bagaimana setiap request dipetakan ke logika, bagaimana data dikirim dan diterima, dan bagaimana status code memberi makna pada setiap interaksi. Dan itulah yang membedakan developer yang “pakai” API… dengan developer yang benar-benar **menguasai** API. Sekarang, kamu punya server RESTful-mu sendiri—tanpa satu dependensi pun! 🚀
