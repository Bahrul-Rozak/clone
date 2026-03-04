Halo teman-teman!

Bayangkan kamu ingin membangun server web—tapi tanpa framework, tanpa Express, tanpa dependensi eksternal. Hanya Node.js murni. Mungkin terdengar menantang, tapi sebenarnya **Node.js sudah menyediakan semua yang kamu butuhkan** lewat modul inti `http`. Ya, kamu bisa membuat HTTP server yang berjalan penuh—menerima request, mengirim response, bahkan menangani rute dasar—hanya dengan satu baris `require('http')`.

Mengapa penting belajar ini? Karena memahami cara kerja server HTTP dari nol memberimu **pemahaman mendalam tentang bagaimana web sebenarnya bekerja**. Express, Fastify, atau Koa hanyalah lapisan di atas fondasi ini. Dan saat kamu tahu fondasinya, kamu tidak akan pernah takut lagi saat harus debug, optimasi, atau bahkan membangun framework-mu sendiri.

Mari kita mulai dari yang paling dasar: **membuat server HTTP yang merespons “Halo Dunia!”**.

```js
const http = require('http');

// Buat server
const server = http.createServer((req, res) => {
  // Set header respons: status 200 + tipe konten teks
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  
  // Kirim isi respons
  res.end('Halo Dunia dari Server HTTP Native!');
});

// Jalankan server di port 3000
server.listen(3000, () => {
  console.log('✅ Server berjalan di http://localhost:3000');
});
```

Jalankan file ini dengan `node server.js`, lalu buka browser ke `http://localhost:3000`—dan voilà! Kamu punya server web hidup.

Sekarang, mari selami apa yang terjadi di balik layar:

1. **`http.createServer()`** menerima sebuah *callback function* yang akan dipanggil setiap kali ada request masuk.
2. Callback ini menerima dua argumen:
   - `req` (**request**): objek yang berisi data dari klien (URL, method, header, body, dll).
   - `res` (**response**): objek yang digunakan untuk mengirim balasan ke klien.
3. **`res.writeHead()`** digunakan untuk mengatur status code dan header HTTP.
4. **`res.end()`** mengakhiri respons dan mengirim data ke klien. Ini wajib dipanggil—kalau tidak, browser akan terus menunggu.

Kita bisa perluas ini untuk menangani **rute berbeda** berdasarkan URL dan method:

```js
const http = require('http');
const url = require('url');

const server = http.createServer((req, res) => {
  const parsedUrl = url.parse(req.url, true);
  const path = parsedUrl.pathname;
  const method = req.method;

  if (path === '/' && method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>Halaman Utama</h1>');
  } 
  else if (path === '/about' && method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ message: 'Tentang kami' }));
  } 
  else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('404 - Halaman tidak ditemukan');
  }
});

server.listen(3000, () => {
  console.log('✅ Server dengan rute dasar berjalan di port 3000');
});
```

Perhatikan bahwa kita menggunakan modul `url` untuk mem-parse URL agar lebih mudah diakses bagian-bagiannya (seperti `pathname`).

Beberapa hal penting yang harus kamu ingat:
- **Setiap request harus diakhiri dengan `res.end()`**—tidak peduli sukses atau error.
- **Header hanya bisa dikirim sebelum `res.end()`**.
- **Server ini bersifat stateless**: setiap request diproses secara independen.
- **Tidak ada middleware otomatis**—semua logika harus kamu tulis manual.

Meski terlihat “ribet” dibanding Express, inilah **inti sejati dari web server**: permintaan masuk → proses → kirim balasan. Tidak ada sihir. Hanya logika murni.

**Kesimpulannya**, membuat HTTP server dari nol dengan modul `http` bukan sekadar latihan akademis—ini adalah jendela ke dalam mesin web itu sendiri. Dengan memahami alur request-response, struktur header, dan siklus hidup koneksi, kamu tidak hanya jadi developer yang lebih kompeten, tapi juga lebih percaya diri saat bekerja dengan framework apa pun. Karena pada akhirnya, **semua server web—sekompleks apa pun—masih berdiri di atas prinsip sederhana ini: dengarkan, proses, dan balas**. Dan sekarang, kamu tahu cara membangunnya dari nol.
