Halo teman-teman!

Bayangkan kamu sedang membangun aplikasi backend yang harus menangani **file besar**—misalnya video 2GB, dataset CSV 500MB, atau log harian yang terus bertambah. Jika kamu mencoba membaca atau menulis file seperti ini dengan metode biasa seperti `fs.readFile` atau `fs.writeFile`, **aplikasimu bisa langsung crash**. Kenapa? Karena metode tersebut memuat **seluruh isi file ke dalam memori RAM sekaligus**. Dan kalau RAM-mu cuma 1GB, sementara file-nya 2GB? Game over.

Untungnya, Node.js punya solusi elegan untuk ini: **Stream**.

Stream adalah cara membaca atau menulis data **secara bertahap**, potongan demi potongan (disebut *chunk*), tanpa perlu memuat semuanya ke memori. Ini seperti menonton film di YouTube—kamu tidak perlu download dulu seluruh videonya, tapi langsung nonton sambil data mengalir masuk. Di dunia backend, stream adalah **senjata utama** untuk menangani file besar, transfer data real-time, atau proses transformasi data skala besar.

Di Node.js, modul `fs` menyediakan `fs.createReadStream()` untuk membaca dan `fs.createWriteStream()` untuk menulis. Mari kita lihat contoh nyata: **menyalin file besar tanpa membebani memori**.

```js
const fs = require('fs');

// Buat stream baca dari file asal
const readStream = fs.createReadStream('video-asli.mp4');
// Buat stream tulis ke file baru
const writeStream = fs.createWriteStream('video-salinan.mp4');

// Alirkan data dari pembaca ke penulis
readStream.pipe(writeStream);

// Tangani saat selesai
writeStream.on('finish', () => {
  console.log('File berhasil disalin!');
});

// Tangani error
readStream.on('error', (err) => {
  console.error('Error saat membaca:', err.message);
});
writeStream.on('error', (err) => {
  console.error('Error saat menulis:', err.message);
});
```

Perhatikan fungsi `.pipe()`—ini adalah jantung stream di Node.js. Ia menghubungkan output dari satu stream ke input stream lain, secara otomatis mengatur aliran data, backpressure, dan buffer. Yang paling keren? **Memori yang dipakai tetap stabil**, bahkan untuk file ratusan gigabyte.

Tapi stream bukan cuma untuk salin-menyalin. Kamu juga bisa **membaca file besar baris per baris** (misalnya untuk proses log), atau **mengompresi data saat ditransfer**. Contoh: membaca file CSV besar dan mengirim isinya sebagai respons HTTP:

```js
const fs = require('fs');
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.url === '/data.csv') {
    res.writeHead(200, { 'Content-Type': 'text/csv' });
    const stream = fs.createReadStream('data-besar.csv');
    stream.pipe(res); // Kirim langsung ke client, tanpa load ke memori!
  }
});

server.listen(3000);
```

Di sini, server tidak pernah menyimpan seluruh file di memori—ia hanya “meneruskan” data dari disk ke klien, chunk demi chunk. Ini membuat aplikasi tetap ringan, cepat, dan scalable.

Namun, ada beberapa hal penting yang harus kamu ingat:
1. **Stream bersifat event-driven**—gunakan event seperti `'data'`, `'end'`, `'error'`, dan `'finish'` untuk kontrol penuh.
2. **Jangan campur stream dengan async/await biasa**—stream punya mekanisme aliran sendiri. Tapi jika perlu, kamu bisa wrap-nya ke dalam Promise dengan hati-hati.
3. **Selalu tangani error di kedua sisi stream**—baik reader maupun writer—karena error di salah satu akan menghentikan seluruh proses.

**Kesimpulannya**, stream adalah fitur inti Node.js yang membedakan aplikasi “main-main” dari sistem produksi yang tangguh. Dengan stream, kamu bisa menangani file sebesar apa pun—bahkan yang lebih besar dari RAM-mu—tanpa khawatir crash atau lambat. Ini bukan sekadar teknik optimisasi, tapi **prinsip dasar arsitektur backend modern**: jangan tahan data, alirkan saja. Karena di dunia yang penuh data besar, **yang menang bukan yang punya memori paling besar… tapi yang paling pandai mengalirkan**.
