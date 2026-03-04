Halo teman-teman!

Di dunia backend modern, **mengirim respons dalam format JSON** bukan lagi pilihan—tapi standar wajib. API, aplikasi mobile, bahkan website dinamis semua berkomunikasi lewat JSON karena ringkas, terstruktur, dan mudah diproses di sisi klien. Nah, di Node.js native (tanpa Express), mengirim JSON itu sangat sederhana—tapi ada **dua hal krusial** yang harus kamu lakukan:  
1. Mengatur header `Content-Type: application/json`,  
2. Mengonversi objek JavaScript ke string dengan `JSON.stringify()`.

Jangan khawatir—kita akan praktikkan ini **step-by-step dari nol**, sampai kamu bisa menjalankan server sendiri dan melihat respons JSON-nya langsung di browser!

---

### 🔧 **Langkah 1: Buat Folder & File Proyek**

Buka terminal (Command Prompt, PowerShell, atau VS Code Terminal), lalu buat folder baru:

```bash
mkdir json-response-demo
cd json-response-demo
```

Buat file server:

```bash
# Di macOS/Linux:
touch server.js

# Di Windows (Command Prompt):
type nul > server.js
```

---

### 📄 **Langkah 2: Tulis Kode Server yang Mengirim JSON**

Buka `server.js` di editor kode, lalu salin kode berikut:

```js
const http = require('http');

// Buat server HTTP
const server = http.createServer((req, res) => {
  // Langkah 1: Atur header — WAJIB untuk JSON!
  res.setHeader('Content-Type', 'application/json');

  // Langkah 2: Siapkan data dalam bentuk objek JavaScript
  const responseData = {
    pesan: "Halo! Ini adalah respons JSON dari server native Node.js.",
    waktu: new Date().toISOString(),
    status: "sukses",
    fitur: ["routing manual", "parsing URL", "respons JSON"]
  };

  // Langkah 3: Kirim respons dengan status 200 (OK)
  res.writeHead(200);

  // Langkah 4: Konversi objek ke string JSON, lalu kirim
  res.end(JSON.stringify(responseData));
});

// Jalankan server di port 3000
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`✅ Server berjalan di http://localhost:${PORT}`);
  console.log(`Buka browser dan kunjungi: http://localhost:${PORT}`);
});
```

> 💡 **Catatan penting**:  
> - `res.setHeader('Content-Type', 'application/json')` memberi tahu browser/klien: “Ini bukan HTML biasa, tapi data JSON!”  
> - `JSON.stringify()` mengubah objek `{}` menjadi string yang valid seperti `"{\"pesan\":\"Halo!\"}"`.  
> - `res.end()` **harus menerima string atau buffer**—tidak bisa objek mentah!

---

### ▶️ **Langkah 3: Jalankan Server**

Kembali ke terminal, pastikan kamu berada di folder `json-response-demo`, lalu jalankan:

```bash
node server.js
```

Jika berhasil, kamu akan melihat:
```
✅ Server berjalan di http://localhost:3000
Buka browser dan kunjungi: http://localhost:3000
```

---

### 🌐 **Langkah 4: Uji Respons JSON di Browser**

Buka browser (Chrome, Firefox, Edge), lalu kunjungi:

```
http://localhost:3000
```

Kamu akan melihat teks berwarna (biasanya hitam/putih) seperti ini:

```json
{
  "pesan": "Halo! Ini adalah respons JSON dari server native Node.js.",
  "waktu": "2026-03-04T10:15:30.123Z",
  "status": "sukses",
  "fitur": ["routing manual", "parsing URL", "respons JSON"]
}
```

> 🔍 **Tips**:  
> - Di Chrome, instal ekstensi seperti **JSON Formatter** agar tampilan JSON lebih rapi (dengan indentasi dan warna).  
> - Jika kamu lihat kode sebagai teks biasa (bukan JSON terformat), itu artinya header `Content-Type` tidak diatur dengan benar!

---

### ⚠️ **Apa yang Terjadi Jika Kamu Lupa Header atau stringify?**

1. **Lupa `Content-Type`**:  
   Browser akan anggap ini teks biasa → tidak diformat sebagai JSON.

2. **Lupa `JSON.stringify()`**:  
   ```js
   res.end(responseData); // ❌ ERROR!
   ```
   Akan muncul error:  
   `TypeError [ERR_INVALID_ARG_TYPE]: The "chunk" argument must be of type string or an instance of Buffer...`

3. **Kirim objek tanpa header + stringify**:  
   Server crash atau klien bingung—karena protokol HTTP hanya menerima **string atau buffer**.

---

### 🧪 **Bonus: Kirim Error dalam Format JSON**

Kamu juga bisa mengirim respons error dengan status code 4xx/5xx:

```js
// Contoh: rute tidak ditemukan
if (req.url === '/rahasia') {
  res.writeHead(403);
  res.end(JSON.stringify({ error: 'Akses ditolak!', kode: 403 }));
}
```

---

**Kesimpulannya**, mengirim respons JSON di Node.js native hanyalah kombinasi tiga langkah:  
1. Set header `Content-Type: application/json`,  
2. Siapkan data sebagai objek JavaScript,  
3. Konversi ke string dengan `JSON.stringify()` sebelum kirim via `res.end()`.  

Tidak ada sihir. Tidak ada framework. Hanya logika bersih dan protokol HTTP yang dipahami dengan benar. Dan dengan mempraktikkannya dari nol seperti ini, kamu tidak hanya tahu *cara* mengirim JSON—tapi juga *mengapa* setiap baris kode itu wajib ada. Karena di dunia backend, **data yang dikirim harus selalu jelas, terstruktur, dan sesuai standar—bukan asal jalan**. Sekarang, server-mu sudah siap bicara bahasa universal: JSON! 🚀
