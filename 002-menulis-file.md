Halo teman-teman!

Setelah kita memahami cara membaca file di Node.js, sekarang saatnya melangkah ke operasi yang tak kalah penting: **menulis file**. Di backend, kamu sering perlu menyimpan data—bisa berupa log aktivitas pengguna, hasil ekspor laporan, cache sementara, atau bahkan file konfigurasi yang dibuat dinamis. Untungnya, Node.js menyediakan kemampuan ini lewat modul inti `fs` (File System), dengan fleksibilitas yang sama seperti saat membaca: kamu bisa menulis file secara **synchronous**, **asynchronous**, atau dengan gaya modern **Promise + async/await**.

Pertama, mari kenali metode dasarnya: `fs.writeFile`. Fungsi ini akan **menimpa seluruh isi file** jika file tersebut sudah ada, atau **membuat file baru** jika belum ada. Ini sangat berguna saat kamu ingin menyimpan data utuh dalam satu operasi—misalnya menyimpan respons API ke file JSON, atau menulis template HTML hasil render.

Contoh menulis file secara asinkron (dengan callback):
```js
const fs = require('fs');

const data = JSON.stringify({ pesan: "Halo dari backend!", waktu: new Date() });

fs.writeFile('output.json', data, 'utf8', (err) => {
  if (err) {
    console.error('Gagal menulis file:', err.message);
    return;
  }
  console.log('File berhasil disimpan!');
});
```

Perhatikan tiga argumen utama:  
1. Nama file (termasuk path jika perlu),  
2. Data yang akan ditulis (harus berupa string atau buffer),  
3. Encoding (biasanya `'utf8'` untuk teks biasa).  

Jika kamu lupa encoding, Node.js akan menyimpan data sebagai **buffer biner**—dan file-nya tidak bisa dibaca sebagai teks biasa.

Untuk kode yang lebih rapi dan mudah di-debug, gunakan pendekatan **async/await** lewat `fs.promises`:
```js
const fs = require('fs').promises;

async function simpanLaporan() {
  try {
    const laporan = { status: 'sukses', timestamp: Date.now() };
    await fs.writeFile('laporan.json', JSON.stringify(laporan, null, 2), 'utf8');
    console.log('Laporan tersimpan!');
  } catch (err) {
    console.error('Error saat menyimpan:', err.message);
  }
}

simpanLaporan();
```

Di sini, kita juga memformat JSON dengan `null, 2` agar hasilnya rapi (ada indentasi)—sangat berguna untuk file konfigurasi atau log yang perlu dibaca manusia.

Namun, apa yang terjadi jika kamu **tidak ingin menimpa file**, tapi justru **menambahkan (append) data** ke akhir file? Misalnya, saat mencatat log aktivitas harian. Untuk itu, Node.js menyediakan `fs.appendFile`:
```js
const fs = require('fs').promises;

async function catatAktivitas(pesan) {
  const log = `[${new Date().toISOString()}] ${pesan}\n`;
  await fs.appendFile('aktivitas.log', log, 'utf8');
}

// Panggil berkali-kali → data bertambah, tidak ditimpa
catatAktivitas("User login");
catatAktivitas("Data diunduh");
```

Dan seperti biasa—**error handling wajib ada**. Saat menulis file, error bisa muncul karena:  
- Direktori tujuan tidak ada,  
- Izin tulis ditolak oleh sistem operasi,  
- Disk penuh,  
- Atau nama file mengandung karakter ilegal.

Tanpa penanganan error, aplikasimu bisa crash diam-diam—dan kamu bahkan tidak tahu data penting gagal tersimpan.

**Kesimpulannya**, menulis file di Node.js adalah keterampilan esensial yang harus dikuasai dengan penuh kesadaran konteks:  
- Gunakan `writeFile` untuk menyimpan data utuh (replace/create),  
- Gunakan `appendFile` untuk menambahkan data (seperti log),  
- Pilih gaya **async/await** untuk kode backend yang bersih dan non-blocking,  
- Selalu konversi data ke string (misalnya dengan `JSON.stringify`),  
- Dan **jangan pernah lupa tangani error**.  

Dengan pendekatan yang tepat, kamu tidak hanya “menyimpan file”—tapi membangun sistem yang andal, transparan, dan siap produksi. Karena di dunia backend, **data yang tidak tersimpan dengan benar… sama saja dengan data yang hilang**.
