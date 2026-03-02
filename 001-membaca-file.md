Halo teman-teman!

Di dunia backend, kemampuan untuk berinteraksi dengan file sistem adalah salah satu keterampilan paling mendasar—dan di Node.js, semua itu dimungkinkan lewat **modul inti `fs` (File System)**. Modul ini memungkinkan kamu membaca, menulis, menghapus, bahkan memantau perubahan pada file dan folder, **tanpa perlu instalasi pustaka tambahan**, karena `fs` sudah termasuk dalam instalasi Node.js sejak awal.

Kita mulai dari hal paling umum: **membaca file**. Di lingkungan backend, kamu sering perlu membaca file konfigurasi (misalnya `.env` atau `config.json`), log aplikasi, template HTML, atau data statis seperti CSV. Node.js menyediakan dua pendekatan utama untuk membaca file: **synchronous (sinkron)** dan **asynchronous (asinkron)**.  

Pendekatan **synchronous** (misalnya `fs.readFileSync`) akan **memblokir seluruh eksekusi program** sampai file selesai dibaca. Ini praktis untuk skrip CLI sederhana atau inisialisasi awal aplikasi, tapi **sangat tidak disarankan di server web**, karena bisa membuat seluruh aplikasi “beku” saat menangani banyak pengguna.

Contoh baca file secara sinkron:
```js
const fs = require('fs');

try {
  const data = fs.readFileSync('catatan.txt', 'utf8');
  console.log('Isi file:', data);
} catch (err) {
  console.error('Gagal membaca file:', err.message);
}
```

Namun, di lingkungan backend yang menangani banyak permintaan sekaligus—seperti API atau web server—kamu **harus menggunakan pendekatan asinkron**. Dengan `fs.readFile`, operasi pembacaan file berjalan di latar belakang, dan program tetap merespons permintaan lain. Kamu cukup memberikan *callback function* yang akan dipanggil saat data siap.

Contoh baca file secara asinkron:
```js
const fs = require('fs');

fs.readFile('catatan.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Error membaca file:', err.message);
    return;
  }
  console.log('Isi file:', data);
});
```

Sejak ES2015, Node.js juga mendukung gaya **Promise-based** melalui `fs.promises`, yang memungkinkanmu menggunakan `async/await`—cara modern yang jauh lebih mudah dibaca dan ditangani error-nya.

Contoh dengan `async/await`:
```js
const fs = require('fs').promises;

async function bacaFile() {
  try {
    const data = await fs.readFile('catatan.txt', 'utf8');
    console.log('Isi file:', data);
  } catch (err) {
    console.error('Gagal membaca file:', err.message);
  }
}

bacaFile();
```

Perhatikan bahwa kita selalu menentukan encoding `'utf8'` agar hasilnya berupa string (bukan buffer biner). Dan yang tak kalah penting: **selalu tangani error**. File bisa saja tidak ditemukan, izin akses ditolak, atau sedang digunakan proses lain—dan tanpa penanganan error, aplikasimu bisa crash tanpa peringatan.

**Kesimpulannya**, membaca file di Node.js bukan sekadar soal “ambil isi file”, tapi tentang **memilih strategi yang tepat sesuai konteks**:  
- Gunakan `readFileSync` hanya untuk inisialisasi awal atau skrip non-server.  
- Gunakan `readFile` dengan callback atau `fs.promises` + `async/await` untuk aplikasi backend yang responsif.  
Dan selalu ingat: **jangan pernah abaikan error handling**—karena di dunia nyata, file tidak selalu ada, dan sistem tidak selalu ramah. Dengan pendekatan yang benar, kamu tidak hanya membaca file… tapi membangun sistem yang tangguh.
