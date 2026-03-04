Halo teman-teman!

Saat kamu menerima **POST request** di server Node.js native, data yang dikirim klien—entah itu form, JSON, atau teks—tidak langsung tersedia sebagai objek rapi seperti `req.body` di Express. Sebaliknya, data itu datang dalam bentuk **aliran byte (stream)** yang harus kamu **kumpulkan, satukan, lalu parsing manual**. Inilah yang disebut **parsing body request manual**.

Ini adalah keterampilan inti yang wajib kamu kuasai jika ingin benar-benar memahami cara kerja HTTP—bukan hanya mengandalkan framework. Dan kabar baiknya: meski terdengar teknis, prosesnya sangat logis dan bisa kamu bangun dari nol dalam hitungan menit.

Mari kita praktikkan **step-by-step**, lengkap dengan cara menjalankan dan mengujinya!

---

### 🔧 **Langkah 1: Buat Folder & File Proyek**

Buka terminal, lalu buat folder baru:

```bash
mkdir parse-body-manual
cd parse-body-manual
```

Buat file server:

```bash
# macOS/Linux:
touch server.js

# Windows (Command Prompt):
type nul > server.js
```

---

### 📄 **Langkah 2: Tulis Kode Parsing Body Manual**

Buka `server.js` di editor kode, lalu salin kode berikut:

```js
const http = require('http');

const server = http.createServer((req, res) => {
  // Atur header respons
  res.setHeader('Content-Type', 'application/json');
  res.setHeader('Access-Control-Allow-Origin', '*');

  // Hanya tangani POST ke /data
  if (req.url === '/data' && req.method === 'POST') {
    let rawBody = ''; // Penampung sementara untuk data mentah

    // Event: setiap ada potongan data masuk
    req.on('data', (chunk) => {
      rawBody += chunk.toString(); // Konversi buffer ke string, lalu gabungkan
    });

    // Event: saat seluruh body selesai diterima
    req.on('end', () => {
      console.log('📦 Raw body diterima:', rawBody);

      // Cek tipe konten dari header
      const contentType = req.headers['content-type'];

      let parsedData;
      try {
        if (contentType === 'application/json') {
          // Parsing sebagai JSON
          parsedData = JSON.parse(rawBody);
        } else if (contentType === 'text/plain') {
          // Simpan sebagai teks biasa
          parsedData = { text: rawBody };
        } else if (contentType?.startsWith('application/x-www-form-urlencoded')) {
          // Parsing form biasa: "nama=Andi&email=andi@test.com"
          const params = new URLSearchParams(rawBody);
          parsedData = Object.fromEntries(params);
        } else {
          throw new Error('Tipe konten tidak didukung');
        }

        // Kirim balik data yang sudah diparsing
        res.writeHead(200);
        res.end(JSON.stringify({
          sukses: true,
          tipe_konten: contentType,
          data_terparse: parsedData
        }));
      } catch (err) {
        console.error('❌ Error parsing body:', err.message);
        res.writeHead(400);
        res.end(JSON.serialize({ error: 'Gagal memproses body request', detail: err.message }));
      }
    });
  }
  else {
    res.writeHead(404);
    res.end(JSON.stringify({ error: 'Rute hanya menerima POST ke /data' }));
  }
});

// Jalankan server
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`✅ Server berjalan di http://localhost:${PORT}`);
  console.log(`Kirim POST ke /data dengan berbagai tipe konten!`);
});
```

> 💡 **Catatan penting**:  
> - `req` adalah stream → data masuk per *chunk*.  
> - Kita kumpulkan semua chunk di `rawBody`.  
> - Setelah `end`, baru kita parsing sesuai `Content-Type`.

---

### ▶️ **Langkah 3: Jalankan Server**

Di terminal, pastikan kamu di folder `parse-body-manual`, lalu jalankan:

```bash
node server.js
```

Jika berhasil, muncul:
```
✅ Server berjalan di http://localhost:3000
Kirim POST ke /data dengan berbagai tipe konten!
```

---

### 🧪 **Langkah 4: Uji dengan Berbagai Jenis Body**

Buka **terminal baru**, lalu uji tiga skenario umum:

---

#### ✅ **Uji 1: Kirim JSON**

```bash
curl -X POST http://localhost:3000/data \
  -H "Content-Type: application/json" \
  -d '{"nama":"Citra","usia":25}'
```

**Respons:**
```json
{
  "sukses": true,
  "tipe_konten": "application/json",
  "data_terparse": { "nama": "Citra", "usia": 25 }
}
```

---

#### ✅ **Uji 2: Kirim Form (URL-encoded)**

```bash
curl -X POST http://localhost:3000/data \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "produk=laptop&jumlah=2"
```

**Respons:**
```json
{
  "sukses": true,
  "tipe_konten": "application/x-www-form-urlencoded",
  "data_terparse": { "produk": "laptop", "jumlah": "2" }
}
```

> ⚠️ Catatan: nilai form selalu jadi **string**.

---

#### ✅ **Uji 3: Kirim Teks Biasa**

```bash
curl -X POST http://localhost:3000/data \
  -H "Content-Type: text/plain" \
  -d "Ini adalah pesan rahasia!"
```

**Respons:**
```json
{
  "sukses": true,
  "tipe_konten": "text/plain",
  "data_terparse": { "text": "Ini adalah pesan rahasia!" }
}
```

---

#### ❌ **Uji 4: Kirim JSON Rusak**

```bash
curl -X POST http://localhost:3000/data \
  -H "Content-Type: application/json" \
  -d "{ nama: Citra }"  # ❌ JSON tidak valid (kurang tanda kutip)
```

→ Akan muncul error parsing.

---

### 🔍 **Apa yang Terjadi di Balik Layar?**

1. Klien kirim data → dibagi jadi potongan-potongan (*chunks*).
2. Server terima tiap chunk via event `'data'` → simpan di `rawBody`.
3. Saat semua chunk selesai (`'end'`), cek `Content-Type` dari header.
4. Parsing sesuai tipe:
   - `application/json` → `JSON.parse()`
   - `x-www-form-urlencoded` → `URLSearchParams`
   - `text/plain` → simpan apa adanya
5. Kirim respons berisi data yang sudah terstruktur.

---

### ⚠️ **Peringatan Keamanan & Best Practice**

- **Jangan percaya input user** — selalu validasi setelah parsing.
- **Batasi ukuran body** (di contoh ini belum ada) agar tidak kena serangan DoS.
- Untuk proyek nyata, pertimbangkan gunakan library seperti `raw-body` atau middleware — tapi pahami dulu cara manual seperti ini!

---

**Kesimpulannya**, parsing body request manual bukan sekadar latihan teknis—ini adalah **jendela ke dalam arsitektur HTTP itu sendiri**. Dengan membangunnya dari nol, kamu belajar bahwa setiap byte yang dikirim klien harus ditangani dengan hati-hati, disusun ulang, lalu diubah menjadi informasi yang bermakna. Dan inilah fondasi sejati dari setiap API modern: bukan sihir, tapi aliran data yang dikendalikan dengan logika. Sekarang, kamu tidak hanya menerima data—kamu benar-benar **memahaminya**.
