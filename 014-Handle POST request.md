Halo teman-teman!

Sampai sekarang, kita sudah belajar membuat server HTTP native, routing manual, parsing URL, dan mengirim respons JSON. Tapi semuanya masih fokus pada **request GET**—yang datanya dikirim lewat URL. Sekarang, saatnya naik level: **menangani POST request**, di mana data dikirim lewat **body request**, seperti saat pengguna mengisi form, mengirim login, atau mengunggah data ke API.

Di Node.js native (tanpa Express), menangani POST request sedikit lebih rumit karena **body tidak langsung tersedia**—kamu harus "mendengarkan" aliran data yang masuk secara bertahap. Tapi jangan khawatir! Kita akan bahas ini **step-by-step dari nol**, lengkap dengan cara menjalankannya dan mengujinya—jadi kamu bisa ikut praktik langsung.

---

### 🔧 **Langkah 1: Buat Folder & File Proyek**

Buka terminal, lalu buat folder baru:

```bash
mkdir post-handler-demo
cd post-handler-demo
```

Buat file server:

```bash
# macOS/Linux:
touch server.js

# Windows (Command Prompt):
type nul > server.js
```

---

### 📄 **Langkah 2: Tulis Kode untuk Handle POST Request**

Buka `server.js` di editor kode, lalu salin kode berikut:

```js
const http = require('http');

const server = http.createServer((req, res) => {
  // Atur header CORS & tipe konten default
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Content-Type', 'application/json');

  // Tangani preflight CORS (opsional, tapi membantu saat uji dari browser)
  if (req.method === 'OPTIONS') {
    res.writeHead(204);
    return res.end();
  }

  // Hanya tangani POST ke /submit
  if (req.url === '/submit' && req.method === 'POST') {
    let body = '';

    // Dengarkan data yang masuk (chunk by chunk)
    req.on('data', (chunk) => {
      body += chunk.toString(); // Tambahkan potongan data ke body
    });

    // Saat semua data selesai diterima
    req.on('end', () => {
      try {
        // Parsing body sebagai JSON
        const data = JSON.parse(body);

        // Validasi minimal
        if (!data.nama || !data.email) {
          res.writeHead(400);
          return res.end(JSON.stringify({ error: 'Nama dan email wajib diisi!' }));
        }

        // Simpan atau proses data (di sini hanya simulasi)
        console.log('📩 Data diterima:', data);

        // Kirim respons sukses
        res.writeHead(201);
        res.end(JSON.stringify({
          pesan: 'Data berhasil diterima!',
          data_yang_dikirim: data
        }));
      } catch (err) {
        // Jika JSON tidak valid
        res.writeHead(400);
        res.end(JSON.stringify({ error: 'Format JSON tidak valid!' }));
      }
    });
  }
  else if (req.method === 'GET' && req.url === '/') {
    res.writeHead(200);
    res.end(JSON.stringify({
      message: 'Server siap terima POST ke /submit (kirim JSON: { "nama": "...", "email": "..." })'
    }));
  }
  else {
    res.writeHead(404);
    res.end(JSON.stringify({ error: 'Rute atau method tidak didukung' }));
  }
});

// Jalankan server
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`✅ Server berjalan di http://localhost:${PORT}`);
  console.log(`Kirim POST ke: http://localhost:${PORT}/submit`);
});
```

> 💡 **Penjelasan Inti**:  
> - `req` adalah stream readable → data masuk **bertahap** (dalam *chunk*).  
> - Event `'data'`: dipanggil setiap ada potongan data masuk.  
> - Event `'end'`: dipanggil saat seluruh body selesai diterima.  
> - Baru di `'end'` kamu bisa parsing dan proses datanya.

---

### ▶️ **Langkah 3: Jalankan Server**

Kembali ke terminal, pastikan di folder `post-handler-demo`, lalu jalankan:

```bash
node server.js
```

Jika berhasil, muncul:
```
✅ Server berjalan di http://localhost:3000
Kirim POST ke: http://localhost:3000/submit
```

---

### 🧪 **Langkah 4: Uji POST Request**

Karena browser tidak bisa kirim POST JSON langsung dari address bar, kamu butuh tools eksternal. Berikut 3 cara:

---

#### ✅ **Cara 1: Gunakan `curl` (di terminal)**

Buka **terminal baru** (jangan tutup server!), lalu jalankan:

```bash
curl -X POST http://localhost:3000/submit \
  -H "Content-Type: application/json" \
  -d '{"nama":"Andi","email":"andi@example.com"}'
```

> Di Windows PowerShell, ganti tanda kutip jadi:
> ```powershell
> curl -X POST http://localhost:3000/submit -H "Content-Type: application/json" -d '{\"nama\":\"Andi\",\"email\":\"andi@example.com\"}'
> ```

**Respons yang muncul:**
```json
{"pesan":"Data berhasil diterima!","data_yang_dikirim":{"nama":"Andi","email":"andi@example.com"}}
```

Dan di terminal **server**, kamu akan lihat:
```
📩 Data diterima: { nama: 'Andi', email: 'andi@example.com' }
```

---

#### ✅ **Cara 2: Gunakan Postman / Insomnia (GUI)**

1. Buka Postman.
2. Pilih method **POST**.
3. Masukkan URL: `http://localhost:3000/submit`
4. Klik tab **Body** → pilih **raw** → pilih **JSON**.
5. Masukkan:
   ```json
   { "nama": "Budi", "email": "budi@test.com" }
   ```
6. Klik **Send**.

---

#### ✅ **Cara 3: Uji Error (JSON tidak valid)**

Kirim data salah:
```bash
curl -X POST http://localhost:3000/submit -H "Content-Type: application/json" -d "ini bukan json"
```

→ Akan muncul error: `{"error":"Format JSON tidak valid!"}`

---

### ⚠️ **Hal Penting yang Harus Diingat**

1. **Selalu dengarkan `data` dan `end`** — jangan asumsikan `req.body` ada (itu fitur Express, bukan Node.js native).
2. **Pastikan client kirim header `Content-Type: application/json`** — agar kamu tahu cara mem-parsing-nya.
3. **Tangani error parsing** — jika JSON rusak, `JSON.parse()` akan throw error.
4. **Jangan lupa `res.end()`** — kalau tidak, koneksi akan hang!

---

**Kesimpulannya**, menangani POST request di Node.js native mengajarkanmu **cara kerja dasar HTTP**: data dikirim sebagai aliran byte, dan tugasmu adalah merangkainya kembali menjadi informasi yang bermakna. Ini mungkin terasa “ribet” dibanding `req.body` di Express, tapi justru di sinilah kamu belajar **bagaimana request benar-benar bekerja di balik layar**. Dan ketika kamu paham ini, kamu tidak lagi takut pada error “body undefined” atau “stream already consumed”—karena kamu tahu persis bagaimana data mengalir dari klien ke server. Sekarang, server-mu tidak hanya bisa *membaca*—tapi juga *menerima*. Dan itu adalah langkah besar menuju backend yang sesungguhnya.
