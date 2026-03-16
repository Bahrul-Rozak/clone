Halo teman-teman!

Pernahkah kamu melihat URL seperti ini?  
`http://localhost:3000/cari?kategori=elektronik&harga_max=500000`  

Bagian `?kategori=elektronik&harga_max=500000` itu disebut **query string**—dan di baliknya, ada data penting yang dikirim pengguna ke server. Tapi bagaimana cara **membaca dan memakai data itu** di server HTTP native Node.js? Jawabannya: **parsing URL**.

Di materi ini, kita akan belajar **step-by-step** cara mem-parsing URL dan query string secara manual—tanpa Express, tanpa library tambahan—hanya pakai modul bawaan Node.js. Dan yang terpenting: **kita akan praktik langsung, dari nol sampai jalan!**

---

### 🔧 **Langkah 1: Buat Folder & File Proyek**

Buka terminal, lalu buat folder baru:

```bash
mkdir url-parsing-demo
cd url-parsing-demo
```

Buat file server:

```bash
# Jika di macOS/Linux:
touch server.js

# Jika di Windows (di Command Prompt):
type nul > server.js
```

---

### 📄 **Langkah 2: Tulis Kode Parsing URL**

Buka `server.js` di editor kode, lalu salin kode berikut:

```js
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
  // Langkah 1: Parse URL lengkap dengan query string
  const parsedUrl = url.parse(req.url, true); // <-- true = parse query jadi object
  
  // Ambil bagian-bagian penting
  const pathname = parsedUrl.pathname;   // misal: "/cari"
  const query = parsedUrl.query;         // misal: { kategori: "elektronik", harga_max: "500000" }

  // Atur header respons
  res.setHeader('Content-Type', 'application/json');

  // Contoh: tangani rute /cari dengan query
  if (pathname === '/cari' && req.method === 'GET') {
    const { kategori, harga_max } = query;

    // Validasi minimal
    if (!kategori) {
      res.writeHead(400);
      return res.end(JSON.stringify({ error: 'Parameter "kategori" wajib diisi' }));
    }

    // Simulasi hasil pencarian
    const hasil = [
      { nama: 'Laptop XYZ', harga: 4500000 },
      { nama: 'HP ABC', harga: 3000000 }
    ].filter(item => {
      if (harga_max && item.harga > parseInt(harga_max)) return false;
      return item.nama.toLowerCase().includes(kategori.toLowerCase()) || 
             kategori === 'semua';
    });

    res.writeHead(200);
    res.end(JSON.stringify({
      query_diterima: query,
      jumlah_hasil: hasil.length,
      data: hasil
    }));
  }
  else if (pathname === '/') {
    res.writeHead(200);
    res.end(JSON.stringify({
      message: 'Coba akses /cari?kategori=elektronik&harga_max=5000000'
    }));
  }
  else {
    res.writeHead(404);
    res.end(JSON.stringify({ error: 'Rute tidak ditemukan' }));
  }
});

// Jalankan server
const PORT = 3000;
server.listen(PORT, () => {
  console.log(`✅ Server berjalan di http://localhost:${PORT}`);
  console.log(`Contoh URL:\n- http://localhost:${PORT}/cari?kategori=semua\n- http://localhost:${PORT}/cari?kategori=hp&harga_max=3500000`);
});
```

---

### ▶️ **Langkah 3: Jalankan Server**

Kembali ke terminal, pastikan kamu di folder `url-parsing-demo`, lalu jalankan:

```bash
node server.js
```

Jika berhasil, muncul pesan:
```
✅ Server berjalan di http://localhost:3000
Contoh URL:
- http://localhost:3000/cari?kategori=semua
- http://localhost:3000/cari?kategori=hp&harga_max=3500000
```

---

### 🌐 **Langkah 4: Uji Query String di Browser**

Buka browser, lalu coba beberapa URL berikut:

#### ✅ Contoh 1: Cari semua produk
```
http://localhost:3000/cari?kategori=semua
```
→ Akan muncul semua data simulasi.

#### ✅ Contoh 2: Cari dengan batas harga
```
http://localhost:3000/cari?kategori=hp&harga_max=3500000
```
→ Hanya HP ABC yang muncul (karena harganya 3 juta < 3,5 juta).

#### ❌ Contoh 3: Tanpa parameter wajib
```
http://localhost:3000/cari
```
→ Muncul error: `"Parameter \"kategori\" wajib diisi"`.

#### ✅ Contoh 4: Halaman utama
```
http://localhost:3000/
```
→ Panduan penggunaan.

---

### 🔍 **Penjelasan Inti: Bagaimana Parsing Ini Bekerja?**

1. **`url.parse(req.url, true)`**  
   - Argumen kedua (`true`) sangat penting!  
   - Tanpa `true`, `query` akan jadi string biasa (`"kategori=elektronik&..."`).  
   - Dengan `true`, `query` otomatis jadi **object JavaScript**:  
     ```js
     { kategori: "elektronik", harga_max: "500000" }
     ```

2. **Nilai query selalu berupa string**  
   - Jadi, jika butuh angka (seperti `harga_max`), kamu harus konversi manual:  
     ```js
     const max = parseInt(harga_max, 10);
     ```

3. **Aman untuk input dasar**  
   - Tapi tetap **validasi dan sanitasi** sebelum pakai—jangan percaya input user!

---

### ⚠️ **Catatan Penting untuk Developer Modern**

Modul `url.parse()` sebenarnya sudah *deprecated* di Node.js versi terbaru (mulai v11+). Cara modern adalah:

```js
const parsedUrl = new URL(req.url, `http://${req.headers.host}`);
const query = Object.fromEntries(parsedUrl.searchParams);
```

Tapi untuk pembelajaran dasar—dan kompatibilitas luas—`url.parse(url, true)` masih sangat valid dan mudah dipahami.

---

const hasil = [
    {nama: 'Laptop XYZ', harga: 4500000},
    {nama: 'HP ABC', harga: 3000000}
].filter(item => {
    // Filter berdasarkan kategori
    const matchKategori = kategori === 'semua' || 
                         item.nama.toLowerCase().includes(kategori.toLowerCase());
    
    // Filter berdasarkan harga_max (jika ada)
    const matchHarga = !harga_max || item.harga <= parseInt(harga_max);
    
    // Item lolos jika kedua kondisi terpenuhi
    return matchKategori && matchHarga;
});

**Kesimpulannya**, memahami cara parsing URL dan query string adalah keterampilan wajib bagi developer backend. Ini adalah fondasi bagaimana API menerima parameter dari klien—entah itu filter pencarian, pagination, atau kredensial sederhana. Dengan hanya dua baris kode (`url.parse` + destructuring), kamu sudah bisa mengubah string acak di URL menjadi data terstruktur yang siap diproses. Dan yang terpenting: **kamu tahu persis apa yang terjadi di balik layar**, bukan hanya mengandalkan “magic” dari framework. Karena di dunia backend, **data yang masuk harus selalu dipahami—bukan hanya diterima**.
